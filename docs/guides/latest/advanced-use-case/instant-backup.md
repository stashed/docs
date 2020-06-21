---
title: Instant Backup | Stash
description: An step by step guide on how to take instant backup using Stash.
menu:
  docs_{{ .version }}:
    identifier: advance-use-case-instant-backup
    name: Instant Backup
    parent: advance-use-case
    weight: 10
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: guides
---

# Instant Backup

This guide will show you how to take an instant backup in Stash.

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [kind](https://kind.sigs.k8s.io/docs/user/quick-start/).

- Install `Stash` in your cluster following the steps [here](/docs/setup/install.md).

- You should be familiar with the following `Stash` concepts:
  - [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md)
  - [BackupSession](/docs/concepts/crds/backupsession.md)
  - [Repository](/docs/concepts/crds/repository.md)

To keep everything isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$ kubectl create ns demo
namespace/demo created
```

>**Note:** YAML files used in this tutorial are stored in  [docs/examples/guides/latest/advanced-use-case/instant-backup](/docs/examples/guides/latest/advanced-use-case/instant-backup) directory of [stashed/docs](https://github.com/stashed/docs) repository.

## Configure Backup

Here, we are going to configure backup for a Deployment. At first, we are going to deploy a Deployment with a PVC and generate some sample data in it. Then, we are going to configure backup for the Deployment using Stash.

**Deploy Deployment:**

Below is the YAML of the Deployment and PVC that we are going to create,

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: source-data
  namespace: demo
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 2Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: stash-demo
  name: stash-demo
  namespace: demo
spec:
  replicas: 1
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
          claimName: source-data
```

The above Deployment will automatically create a `data.txt` file in `/source/data` directory and write some sample data in it.

Let’s create the Deployment and PVC we have shown above.

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/advanced-use-case/instant-backup/deployment.yaml
persistentvolumeclaim/source-data created
deployment.apps/stash-demo created
```

Now, wait for the pod of the Deployment to go into `Running` state.

```console
$ kubectl get pod -n demo
NAME                          READY   STATUS    RESTARTS   AGE
stash-demo-859d96f6bd-fxr7l   1/1     Running   0          81s
```

Verify that the sample data has been created in `/source/data` directory using the following command,

```console
$ kubectl exec -n demo stash-demo-859d96f6bd-fxr7l -- cat /source/data/data.txt
sample_data
```

**Create Secret and Repository:**

We are going to store our backed up data into a GCS bucket. We have to create a Secret and a Repository object with access credentials and backend information respectively.

> For GCS backend, if the bucket does not exist, Stash needs `Storage Object Admin` role permissions to create the bucket. For more details, please check the following [guide](/docs/guides/latest/backends/gcs.md).

Let’s create a secret called `gcs-secret` with access credentials of our desired GCS backend,

```console
$ echo -n 'changeit' > RESTIC_PASSWORD
$ echo -n '<your-project-id>' > GOOGLE_PROJECT_ID
$ cat downloaded-sa-json.key > GOOGLE_SERVICE_ACCOUNT_JSON_KEY
$ kubectl create secret generic -n demo gcs-secret \
    --from-file=./RESTIC_PASSWORD \
    --from-file=./GOOGLE_PROJECT_ID \
    --from-file=./GOOGLE_SERVICE_ACCOUNT_JSON_KEY
secret/gcs-secret created
```

Now, create a `Repository` using this secret. Below is the YAML of `Repository` crd we are going to create,

```yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: gcs-repo
  namespace: demo
spec:
  backend:
    gcs:
      bucket: appscode-qa
      prefix: /stash/instant/deployment
    storageSecretName: gcs-secret
```

Let’s create the `Repository` object that we have shown above,

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/advanced-use-case/instant-backup/repository.yaml
repository.stash.appscode.com/gcs-repo created
```

Now, we are ready to backup our sample data into this backend.

**Create BackupConfiguration:**

We have to create a `BackupConfiguration` crd targeting the `stash-demo` Deployment that we have deployed earlier. Then, Stash will inject a sidecar container into the target. It will also create a `CronJob` to take periodic backup of `/source/data` directory of the target.

Below is the YAML of the `BackupConfiguration` object that we are going to create and specify that this `BackupConfiguration` will take backup in every 40 minutes.

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: deployment-backup
  namespace: demo
spec:
  repository:
    name: gcs-repo
  schedule: "*/40 * * * *"
  target:
    ref:
      apiVersion: apps/v1
      kind: Deployment
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

Let’s create the `BackupConfiguration` object that we have shown above,

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/advanced-use-case/instant-backup/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/deployment-backup created
```

**Verify Sidecar:**

If everything goes well, Stash will inject a sidecar container into the `stash-demo` Deployment to take backup of `/source/data` directory. Let’s check that the sidecar has been injected successfully,

```console
$ kubectl get pod -n demo
NAME                         READY   STATUS    RESTARTS   AGE
stash-demo-9bff9fd4f-xvt77   2/2     Running   0          57s
```

Look at the pod. It now has 2 containers. If you view the resource definition of this pod, you will see that there is a container named `stash` which is running `run-backup` command.

**Verify CronJob:**

It will also create a `CronJob` with the schedule specified in `spec.schedule` field of `BackupConfiguration` crd.

Verify that the `CronJob` has been created using the following command,

```console
$ kubectl get backupconfiguration -n demo
NAME                TASK   SCHEDULE       PAUSED   AGE
deployment-backup          */40 * * * *            6m41s
```

## Trigger Instant Backup

Now, we are going to trigger an instant backup of the volumes of the Deployment that we have configured in the previous section. To do that, we have to create a `BackupSession` object pointing to the respective `BackupConfiguration` object.

**Create BackupSession:**

Below is the YAML of the `BackupSession` crd that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupSession
metadata:
  labels:
    stash.appscode.com/backup-configuration: deployment-backup
  name: deployment-backupsession
  namespace: demo
spec:
  invoker:
    apiGroup: stash.appscode.com
    kind: BackupConfiguration
    name: deployment-backup
```

- `metadata.labels` holds the respective `BackupConfiguration` name as a label. Stash backup sidecar container use this label to watch only the BackupSessions of that `BackupConfiguration`. You must provide the label in the following format:
  ```
  stash.appscode.com/backup-configuration: <BackupConfiguration name>
  ```
- `spec.invoker` section indicates the `BackupConfiguration` object whose target will be backed up instantly for this `BackupSession`.

Let's create the `BackupSession` object that we have have shown above,

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/advanced-use-case/instant-backup/backupsession.yaml
backupsession.stash.appscode.com/deployment-backupsession created
```

If everything goes well, the stash sidecar inside the Deployment will take a backup instantly.

**Wait for BackupSession to Succeed:**

Run the following command to watch `BackupSession` phase,

```console
$ watch -n 3 kubectl get backupsession -n demo
Every 3.0s: kubectl get backupsession -n demo                               suaas-appscode: Wed Jul 10 17:18:52 2019

NAME                       INVOKER-TYPE          INVOKER-NAME        PHASE       AGE
deployment-backupsession   BackupConfiguration   deployment-backup   Succeeded   21s
```

We can see from the above output that the instant backup session has succeeded. Now, we are going to verify that the backed up data has been stored in the backend.

**Verify Backup:**

Once a backup is complete, Stash will update the respective `Repository` crd to reflect the backup. Check that the repository `gcs-repo` has been updated by the following command,

```console
$ kubectl get repository -n demo gcs-repo
NAME       INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
gcs-repo   true        24 B   1                116s                     10m
```

Now, if we navigate to the GCS bucket, we are going to see backed up data has been stored in `stash/instant/deployment` directory as specified by `spec.backend.gcs.prefix` field of `Repository` crd.

<figure align="center">
  <img alt="Backup data in GCS Bucket" src="/docs/images/guides/latest/advanced-use-case/instant.png">
  <figcaption align="center">Fig: Backup data in GCS Bucket</figcaption>
</figure>

>**Note:** Stash keeps all the backed up data encrypted. So, data in the backend will not make any sense until they are decrypted.

## Cleanup

```yaml
kubectl delete -n demo deployment stash-demo
kubectl delete -n demo backupconfiguration deployment-backup
kubectl delete -n demo backupsession deployment-backupsession
kubectl delete -n demo repository gcs-secret
kubectl delete -n demo secret gcs-secret
kubectl delete -n demo pvc --all
```
