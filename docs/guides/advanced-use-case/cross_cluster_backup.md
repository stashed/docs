---
title: Cross-Cluster Backup and Restore | Stash
description: A guide on how to use backup and restore across clusters using Stash.
menu:
  docs_{{ .version }}:
    identifier: advance-use-case-cross-cluster-backup
    name: Cross-Cluster Backup and Restore
    parent: advance-use-case
    weight: 20
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: guides
---

# Cross-Namespace Backup and Restore

This guide will show you how to take backup and restore across clusters using Stash.

## Before You Begin

- At first, you need to have running Kubernetes clusters, and the `kubectl` command-line tool must be configured to communicate with your clusters0. If you do not already have clusters, you can create one by using [kind](https://kind.sigs.k8s.io/docs/user/quick-start/).


- You should be familiar with the following `Stash` concepts:
  - [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md)
  - [BackupSession](/docs/concepts/crds/backupsession.md)
  - [RestoreSession](/docs/concepts/crds/restoresession.md)
  - [Repository](/docs/concepts/crds/repository.md)

To demonstrate the cross-clusters capability, we are going to use the `prod` cluster for taking backup. Then, we will restore the backup in the `staging` cluster.

Let's create a cluster named `prod`,
```bash
$ kubectl create ns prod
Creating cluster "prod" ...
 ✓ Ensuring node image (kindest/node:v1.23.0) 🖼
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-prod"
You can now use your cluster with:

kubectl cluster-info --context kind-prod

Have a nice day! 👋
```

We are going to create a namespace named `demo` in `prod` cluster,
```
$ kubectl create ns demo
namespace/demo created
```

>**Note:** YAML files used in this tutorial are stored in [docs/examples/guides/advanced-use-case/cross-cluster-backup](/docs/examples/guides/advanced-use-case/cross-cluster-backup) directory of [stashed/docs](https://github.com/stashed/docs) repository.


Install `Stash` in your `prod` cluster following the steps [here](/docs/setup/README.md).

## Configure Backup

Now, we are going to configure backup for volumes of a Deployment in the `prod` cluster. We will deploy a Deployment with 3 replicas. We will generate some sample data in it. Then, we are going to configure backup for this Deployment using Stash.


### Deploy Deployment

Let's deploy a Deployment in the `prod` cluster at the beginning. This Deployment will automatically generate sample data in the `/source/data` directory.

Below are the YAMLs of the Deployment and PVC that we are going to create,

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: stash-sample-data
  namespace: demo
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: stash-demo
  name: stash-demo
  namespace: demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: stash-demo
  template:
    metadata:
      labels:
        app: stash-demo
      name: busybox
    spec:
      containers:
      - args: ["echo sample_data > /source/data/data.txt && sleep 3000"]
        command: ["/bin/sh", "-c"]
        image: busybox
        imagePullPolicy: IfNotPresent
        name: busybox
        volumeMounts:
        - mountPath: /source/data
          name: source-data
      restartPolicy: Always
      volumes:
      - name: source-data
        persistentVolumeClaim:
          claimName: stash-sample-data
```

Let's create the Deployment and PVC we have shown above.

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/advanced-use-case/cross-cluster-backup/deployment_prod.yaml

persistentvolumeclaim/stash-sample-data created
deployment.apps/stash-demo created
```

Now, wait for the pods of the Deployment to go into the `Running` state. You can verify the `Status` of the pods by executing the following command,

```bash
$ kubectl get pods -n demo
NAME                          READY   STATUS    RESTARTS   AGE
stash-demo-7678679bcb-2s927   1/1     Running   0          29s
stash-demo-7678679bcb-b849l   1/1     Running   0          29s
stash-demo-7678679bcb-p62vn   1/1     Running   0          29s
```

Verify that the sample data has been created in `/source/data` directory using the following command,

```bash
$ kubectl exec -n demo stash-demo-7678679bcb-2s927 -- cat /source/data/data.txt

sample_data
```

### Prepare Backend

We are going to store our backed-up data into a GCS bucket. We have to create a Secret with the necessary credentials and a Repository CRD to use this backend. We will create the Secret and Repository in the `stash` namespace. With the cross-namespace Repository support, Stash has the ability to backup and restore data in a different namespace.

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
  usagePolicy:
    allowedNamespaces:
      from: All
```

Notice the `spec.usagePolicy` section. Here, we are allowing all namespaces to refer to this repository. You can restrict this capability to a particular namespace. Please, check the following guide from [here](/docs/concepts/crds/repository.md) for more details.

Let's create the Repository we have shown above,
```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/advanced-use-case/cross-namespace-backup/repository.yaml
repository.stash.appscode.com/gcs-repo created
```
Now, we are ready to backup our sample data into this backend.

### Backup

We are going to create a `BackupConfiguration` crd in the `prod` namespace targeting the `stash-demo` StatefulSet that we have deployed earlier. This `BackupConfiguration` will refer to the `gcs-repo` repository of the `stash` namespace. Stash will inject a sidecar container into the target. It will also create a `CronJob` to take periodic backup of the `/source/data` directory of the target.

**Create BackupConfiguration:**

Below is the YAML of the `BackupConfiguration` crd that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: ss-backup
  namespace: prod
spec:
  repository:
    name: gcs-repo
    namespace: stash
  schedule: "*/5 * * * *"
  target:
    ref:
      apiVersion: apps/v1
      kind: StatefulSet
      name: stash-demo
    volumeMounts:
    - name: source-data
      mountPath: /source/data
    paths:
    - /source/data
  retentionPolicy:
    name: 'keep-last-5'
    keepLast: 5
    prune: true
```

Here,

- `spec.repository.name` refers to the Repository object that holds backend information.
- `spec.repository.namespace` refers to the namespace of the Repository object.
- `spec.schedule` is a cron expression that indicates BackupSession will be created at 5 minutes intervals.
- `spec.target.ref` refers to the `stash-demo` StatefulSet.
- `spec.target.volumeMounts` specifies a list of volumes and their mountPath that contain the target paths.
- `spec.target.paths` specifies list of file paths to backup.

Let's create the `BackupConfiguration` crd we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/workloads/statefulset/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/ss-backup created
```

### Verify Backup Setup

**Verify BackupConfiguration Ready:**

If everything goes well, Stash should create a `stash-demo` StatefulSet in the `prod` namespace and the phase of that `BackupConfiguration` should be `Ready`. Verify the `BackupConfiguration` crd by the following command,

```bash
❯ kubectl get backupconfiguration -n prod
NAME        TASK   SCHEDULE      PAUSED   PHASE   AGE
ss-backup          */5 * * * *            Ready   13s
```

**Verify CronJob:**

Stash will also create a `CronJob` with the schedule specified in the `spec.schedule` field of the `BackupConfiguration` crd.

Verify that the `CronJob` has been created using the following command,

```bash
$ kubectl get cronjob -n  prod
NAME                      SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
stash-trigger-ss-backup   */5 * * * *   False     0        4m55s           3m14s
```

### Verify Backup

**Verify BackupSession Succeeded:**

The `stash-trigger-ss-backup` CronJob will trigger a backup on each scheduled slot by creating a `BackupSession` crd. The sidecar container watches for the `BackupSession` crd. When it finds one, it will take backup immediately.

Wait for the next schedule for the backup. Run the following command to watch `BackupSession` crd,

```bash
$ kubectl get backupsession -n prod -w

NAME                   INVOKER-TYPE          INVOKER-NAME    PHASE      DURATION   AGE
ss-backup-1644562803   BackupConfiguration   ss-backup      Running                18s
ss-backup-1644562803   BackupConfiguration   ss-backup      Running                34s
ss-backup-1644562803   BackupConfiguration   ss-backup      Succeeded   1m21s      81s
```

We can see from the above output that the backup session has succeeded. Now, we are going to verify whether the backed-up data has been stored in the backend.


## Restore

This section will show you how to restore the backed-up data in a different namespace. We will demonstrate restoring volumes in the `staging` namespace using Stash.

**Stop Taking Backup of the Old StatefulSet:**

At first, let's stop taking any further backup of the old StatefulSet so that no backup is taken during the restore process. We are going to pause the `BackupConfiguration` that we created earlier. Then, Stash will stop taking any further backup for this StatefulSet. You can learn more how to pause a scheduled backup [here](/docs/guides/advanced-use-case/pause-backup.md)

Let's pause the `ss-backup` BackupConfiguration,

```bash
$ kubectl patch backupconfiguration -n prod ss-backup --type="merge" --patch='{"spec": {"paused": true}}'
backupconfiguration.stash.appscode.com/ss-backup patched
```

Verify that the BackupConfiguration has been paused,

```bash
$ kubectl get backupconfiguration -n prod
NAME        TASK   SCHEDULE      PAUSED   PHASE   AGE
ss-backup          */5 * * * *   true     Ready   53m
```

Notice the `PAUSED` column. Value `true` indicates that the BackupConfiguration has been paused.

**Deploy StatefulSet:**

We are going to create a new StatefulSet named `stash-recovered` and restore the backed-up data inside it.

Below is the YAML of the StatefulSet that we are going to create,

```yaml
apiVersion: v1
kind: Service
metadata:
  name: re-headless
  namespace: staging
spec:
  ports:
  - name: http
    port: 80
    targetPort: 0
  selector:
    app: stash-recovered
  clusterIP: None
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: stash-recovered
  namespace: staging
  labels:
    app: stash-recovered
spec:
  replicas: 3
  selector:
    matchLabels:
      app: stash-recovered
  serviceName: re-headless
  template:
    metadata:
      labels:
        app: stash-recovered
    spec:
      containers:
      - name: busybox
        image: busybox
        command:
        - sleep
        - '3600'
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
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/advanced-use-case/cross-namespace-backup/statefulset_recovered.yaml
service/re-headless created
statefulset.apps/stash-recovered created
```

**Create RestoreSession:**

Now, we need to create a `RestoreSession` crd targeting the `stash-recovered` StatefulSet to restore the backed-up data inside it.

Below is the YAML of the `RestoreSesion` crd that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: ss-restore
  namespace: staging
spec:
  repository:
    name: gcs-repo
    namespace: stash
  target:
    ref:
      apiVersion: apps/v1
      kind: StatefulSet
      name: stash-recovered
    volumeMounts:
    - name:  source-data
      mountPath:  /source/data
    rules:
    - paths:
      - /source/data
```

Here,

- `spec.repository.name` specifies the name of the Repository.
- `spec.repository.namespace` refers to the namespace of the Repository.
- `spec.target.ref` refers to the target workload where the recovered data will be stored.
- `spec.target.volumeMounts` specifies a list of volumes and their mountPath where the data will be restored.
  - `mountPath` must be the same mountPath as the original volume because Stash stores the absolute path of the backed-up files. If you use a different mountPath for the restored volume the backed up files will not be restored into your desired volume.

Let's create the `RestoreSession` crd we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/workloads/statefulset/restoresession.yaml
restoresession.stash.appscode.com/ss-restore created
```

Once, you have created the `RestoreSession` crd, Stash will inject `init-container` into the `stash-recovered` StatefulSet. The StatefulSet will restart and the `init-container` will restore the desired data on start-up.

### Verify Restore

**Wait for RestoreSession to Succeeded:**

Run the following command to watch the RestoreSession phase,

```bash
$ kubectl get restoresession -n staging -w

NAME         REPOSITORY   PHASE       DURATION   AGE
ss-restore   gcs-repo     Succeeded   3m6s       4m
```

So, we can see from the output of the above command that the restore process succeeded.


**Verify Restored Data:**

In this section, we are going to verify that the desired data has been restored successfully.

At first, check if the `stash-recovered` pods of a StatefulSet has gone into the `Running` state by the following commands,

```bash
$ kubectl get pod -n staging
NAME                READY   STATUS    RESTARTS   AGE
stash-recovered-0   1/1     Running   0          10m
stash-recovered-1   1/1     Running   0          11m
stash-recovered-2   1/1     Running   0          12m
```

Verify that the backed up data has been restored in `/source/data` directory of the `stash-recovered` pods of a StatefulSet using the following commands,

```bash
$ kubectl exec -n staging stash-recovered-0 -- cat /source/data/data.txt
stash-demo-0
$ kubectl exec -n staging stash-recovered-1 -- cat /source/data/data.txt
stash-demo-1
$ kubectl exec -n staging stash-recovered-2 -- cat /source/data/data.txt
stash-demo-2
```

## Cleaning Up

To clean up the Kubernetes resources created by this tutorial, run:

```bash
kubectl delete -n prod statefulset stash-demo
kubectl delete -n prod backupconfiguration ss-backup
kubectl delete -n prod pvc --all
kubectl delete -n staging statefulset stash-recovered
kubectl delete -n staging restoresession ss-restore
kubectl delete -n staging pvc --all
kubectl delete -n stash repository gcs-repo
kubectl delete -n stash secret gcs-secret
```
