---
title: Cross Namespce Repository | Stash
description: A guide on how to  use cross-namespace repository to take backup and restore using Stash.
menu:
  docs_{{ .version }}:
    identifier: advance-use-case-cross-namespace-repo
    name: Cross-Namespace Backup and Restore
    parent: advance-use-case
    weight: 10
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: guides
---

# Cross-Namespace Backup and Restore

This guide will show you how to use cross-namespace repository to take backup and restore in Stash.

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [kind](https://kind.sigs.k8s.io/docs/user/quick-start/).

- Install `Stash` in your cluster following the steps [here](/docs/setup/README.md).

- You should be familiar with the following `Stash` concepts:
  - [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md)
  - [BackupSession](/docs/concepts/crds/backupsession.md)
  - [RestoreSession](/docs/concepts/crds/restoresession.md)
  - [Repository](/docs/concepts/crds/repository.md)

To demonstrate the cross-namespace capability, we are going to use `dev` namespace for taking backup, `staging` namespace for restoring backup, and crete `Repository` in a separate namespace called `stash` in this tutorial.

```bash
$ kubectl create ns dev
namespace/dev created

$ kubectl create ns staging
namespace/staging created

$ kubectl create ns stash
namespace/stash created
```

>**Note:** YAML files used in this tutorial are stored in [docs/examples/guides/advanced-use-case/cross-namespace-repo](/docs/examples/guides/advanced-use-case/cross-namespace-repo) directory of [stashed/docs](https://github.com/stashed/docs) repository.


## Configure Backup

Here, we are going to configure backup for volumes of a StatefulSet in the `dev` namespace. We will deploy a StatefulSet with 3 replicas and a service. We will generate some sample data in it. Then, we are going to configure backup for this StatefulSet using Stash referencing a Repository from `stash` namespace.


### Deploy StatefulSet

Let's deploy a StatefulSet in `dev` namespace at the begining. This StatefulSet will automatically generate sample data in `/source/data` directory.

Here is the YAML of the StatefulSet that we are going to create,

```yaml
apiVersion: v1
kind: Service
metadata:
  name: headless
  namespace: dev
spec:
  ports:
  - name: http
    port: 80
    targetPort: 0
  selector:
    app: stash-demo
  clusterIP: None
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: stash-demo
  namespace: dev
  labels:
    app: stash-demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: stash-demo
  serviceName: headless
  template:
    metadata:
      labels:
        app: stash-demo
    spec:
      containers:
      - name: busybox
        image: busybox
        command: ["/bin/sh", "-c","echo $(POD_NAME) > /source/data/data.txt && sleep 3000"]
        env:
        - name:  POD_NAME
          valueFrom:
            fieldRef:
              fieldPath:  metadata.name
        volumeMounts:
        - name: source-data
          mountPath: "/source/data"
        imagePullPolicy: IfNotPresent
  volumeClaimTemplates:
  - metadata:
      name: source-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "standard"
      resources:
        requests:
          storage: 1Gi
```

Let's create the StatefulSet we have shown above.

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/advanced-use-case/cross-namespace-repo/statefulset.yaml
service/headless created
statefulset.apps/stash-demo created
```

Now, wait for the pods of the StatefulSet to go into the `Running` state. You can verify the Status of the pods by ececuting the following command,

```bash
$ kubectl get pod -n dev
NAME           READY   STATUS    RESTARTS   AGE
stash-demo-0   1/1     Running   0          42s
stash-demo-1   1/1     Running   0          40s
stash-demo-2   1/1     Running   0          36s
```

Verify that the sample data has been generated in `/source/data` directory for `stash-demo-0` , `stash-demo-1` and `stash-demo-2` pod respectively using the following commands,

```bash
$ kubectl exec -n dev stash-demo-0 -- cat /source/data/data.txt
stash-demo-0
$ kubectl exec -n dev stash-demo-1 -- cat /source/data/data.txt
stash-demo-1
$ kubectl exec -n dev stash-demo-2 -- cat /source/data/data.txt
stash-demo-2
```

### Prepare Backend

We are going to store our backed up data into a GCS bucket. We have to create a Secret with necessary credentials and a Repository CRD to use this backend. We will create the Secret and Repository in `stash` namespace. With the cross-namespace Repository support, Stash has the ability to backup and restore data in a different namespace.

If you want to use a different backend, please read the respective backend configuration doc from [here](/docs/guides/backends/overview.md).

> For GCS backend, if the bucket does not exist, Stash needs `Storage Object Admin` role permissions to create the bucket. For more details, please check the following [guide](/docs/guides/backends/gcs.md).

**Create Secret:**

Let's create a secret called `gcs-secret` with access credentials to our desired GCS bucket,

```bash
$ echo -n 'changeit' > RESTIC_PASSWORD
$ echo -n '<your-project-id>' > GOOGLE_PROJECT_ID
$ cat /path/to/downloaded-sa-key.json > GOOGLE_SERVICE_ACCOUNT_JSON_KEY
$ kubectl create secret generic -n stash gcs-secret \
    --from-file=./RESTIC_PASSWORD \
    --from-file=./GOOGLE_PROJECT_ID \
    --from-file=./GOOGLE_SERVICE_ACCOUNT_JSON_KEY
secret/gcs-secret created
```

Now, we are ready to backup our workload's data to our desired backend.

**Create Repository:**

Now, create a `Repository` using this secret. Below is the YAML of `Repository` crd we are going to create,

```yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: gcs-repo
  namespace: stash
spec:
  backend:
    gcs:
      bucket: stash-testing
      prefix: /cross-namespace/data/sample-statefulset
    storageSecretName: gcs-secret
```

Let's create the Repository we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/advanced-use-case/cross-namespace-repo/repository.yaml
repository.stash.appscode.com/gcs-repo created
```

Now, we are ready to backup our sample data into this backend.
