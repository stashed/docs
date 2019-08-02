# Pause Backup

This guide will show you how to stop Stash from taking backup volumes of a Kubernetes workload.

## Before You Begin

- At first, you need to have a Kubernetes cluster. If you do not already have a cluster, you can create one by using [Minikube](https://github.com/kubernetes/minikube).

- Install `Stash` in your cluster following the steps [here](https://appscode.com/products/stash/0.8.3/setup/install/).
- You should be familiar with the following `Stash` concepts:
  - [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md/)
  - [BackupSession](/docs/concepts/crds/backupsession.md/)
  - [Repository](/docs/concepts/crds/repository.md/)

To keep everything isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$ kubectl create ns demo
namespace/demo created
```

> **Note:** YAML files used in this tutorial are stored in [docs/examples/guides/latest/advanced-use-case/pause-backup](/docs/examples/guides/latest/advanced-use-case/pause-backup) directory of [stashed/doc](https://github.com/stashed/doc) repository.

## Run Backup

Here, we are going to deploy a Deployment with a PVC. This Deployment will automatically generate some sample data into the PVC. Then, we are going to backup this sample data using Stash.

**Deploy Deployment:**

Below are the YAMLs of the Deployment and PVC that we are going to create,

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: source-pvc
  namespace: demo
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
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
          claimName: source-pvc
```

The above Deployment will automatically create a `data.txt` file in `/source/data` directory and write some sample data in it.

Let's create the Deployment and PVC we have shown above.

```console
$ kubectl apply -f ./docs/examples/guides/latest/advanced-use-case/pause-backup/deployment.yaml
persistentvolumeclaim/source-pvc created
deployment.apps/stash-demo created
```

Now, wait for the pods of the Deployment to go into the `Running` state.

```console
kubectl get pod -n demo
NAME                          READY   STATUS    RESTARTS   AGE
stash-demo-69f9ffbbf7-bww24   1/1     Running   0          100s
stash-demo-69f9ffbbf7-r8wgh   1/1     Running   0          100s
stash-demo-69f9ffbbf7-rsj55   1/1     Running   0          100s
```

To verify that the sample data has been created in `/source/data` directory, use the following command:

```console
$ kubectl exec -n demo stash-demo-69f9ffbbf7-bww24 -- cat /source/data/data.txt
sample_data
```

**Create Secret and Repository:**

We are going to store our backed up data into a [GCS bucket](https://cloud.google.com/storage/). We have to create a Secret and a Repository object with access credentials and backend information respectively.

Let’s create a secret called ` gcs-secret` with access credentials to our desired GCS bucket,

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

Now, create a `Respository` using this secret. Below is the YAML of `Repository` crd we are going to create,

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
      prefix: /sample/data
    storageSecretName: gcs-secret
```

Let's create the Repository we have shown above,

```console
kubectl apply -f ./docs/examples/guides/latest/advanced-use-case/pause-backup/repository.yaml
repository.stash.appscode.com/gcs-repo created
```

Now, we are ready to backup our sample data into this backend.

**Create BackupConfiguration:**

We have to create a `BackupConfiguration` crd targeting the `stash-demo` Deployment that we have deployed earlier. Stash will inject a sidecar container into the target. It will also create a `CronJob` to take a periodic backup of `/source/data` directory of the target.

Below is the YAML of the `BackupConfiguration` crd that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: pause-backup
  namespace: demo
spec:
  repository:
    name: gcs-repo
  schedule: "*/1 * * * *"
  target:
    ref:
      apiVersion: apps/v1
      kind: Deployment
      name: stash-demo
    volumeMounts:
    - name: source-data
      mountPath: /source/data
    directories:
    - /source/data
  retentionPolicy:
    name: 'keep-last-5'
    keepLast: 5
    prune: true
```

Let's create the `BackupConfiguration` crd we have shown above,

```console
$ kubectl apply -f ./docs/examples/guides/latest/advanced-use-case/pause-backup/backupconfiguration.yaml 
backupconfiguration.stash.appscode.com/pause-backup created
```

**Verify Sidecar:**

If everything goes well, Stash will inject a sidecar container into the `stash-demo` Deployment to take backup of `/source/data` directory. Let’s check that the sidecar has been injected successfully,

```console
$ kubectl get pod -n demo
NAME                          READY   STATUS    RESTARTS   AGE
stash-demo-7489fcb7f5-jctj4   2/2     Running   0          117s
stash-demo-7489fcb7f5-mfqps   2/2     Running   0          112s
stash-demo-7489fcb7f5-zp8c5   2/2     Running   0          115s
```

Look at the pod. It now has 2 containers. If you view the resource definition of this pod, you will see that there is a container named `stash` which is running `run-backup` command.

**Verify CronJob:**

It will also create a `CronJob` with the schedule specified in `spec.schedule` field of `BackupConfiguration` crd.

Verify that the `CronJob` has been created using the following command,

```console
Every 3.0s: kubectl get backupconfiguration --all-namespaces                      suaas-appscode: Thu Aug  1 17:08:08 2019

NAMESPACE   NAME           TASK   SCHEDULE      PAUSED   AGE
demo        pause-backup          */1 * * * *            27s
```

**Wait for BackupSession Succeeded:**

Wait for the next schedule for backup. Run the following command to watch `BackupSession` crd,

```console
Every 3.0s: kubectl get backupssession --all-namespaces                      suaas-appscode: Thu Aug  1 17:43:57 2019

NAMESPACE   NAME                      BACKUPCONFIGURATION   PHASE       AGE
demo        pause-backup-1564659789   pause-backup          Succeeded   49s
```

We can see from the above output that the backup session has succeeded. This indicates that the volumes of the Deployment have been backed up in the backend successfully.

## Pause Running Backup

To stop Stash from taking backup volumes of the deployment, you can do the following things:

**Patch BackupConfiguration:**

- Set `spec.paused`: true in `BackupConfiguration` yaml and then apply the update. This happens:
  - The running backup process will stop immediately and paused `BackupConfiguration` CRD will not be applicable for the targeted workload.
  - Stash sidecar containers will not be removed from existing workloads but the sidecar will stop taking backup.

You can do the things by using the following command,

```console
$ kubectl patch backupconfiguration -n demo stash-demo --type="merge" --patch='{"spec": {"paused": true}}'
backupconfiguration.stash.appscode.com/pause-backup patched
```

Now, Describe the `BackupConfiguration` crd by using the following command,

```yaml
$ kubectl describe backupconfiguration -n demo pause-backup
Name:         pause-backup
Namespace:    demo
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"stash.appscode.com/v1beta1","kind":"BackupConfiguration","metadata":{"annotations":{},"name":"pause-backup","namespace":"de...
API Version:  stash.appscode.com/v1beta1
Kind:         BackupConfiguration
Metadata:
  Creation Timestamp:  2019-08-01T11:42:22Z
  Finalizers:
    stash.appscode.com
  Generation:        2
  Resource Version:  14465
  Self Link:         /apis/stash.appscode.com/v1beta1/namespaces/demo/backupconfigurations/pause-backup
  UID:               5e1abe4b-c3d2-438a-ab8d-67ed7e663546
Spec:
  Paused:  true
  Repository:
    Name:  gcs-repo
  Retention Policy:
    Keep Last:  5
    Name:       keep-last-5
    Prune:      true
  Schedule:     */1 * * * *
  Target:
    Directories:
      /source/data
    Ref:
      API Version:  apps/v1
      Kind:         Deployment
      Name:         stash-demo
    Volume Mounts:
      Mount Path:  /source/data
      Name:        source-data
Events:
  Type    Reason          Age   From                       Message
  ----    ------          ----  ----                       -------
  Normal  Backup Skipped  25s   Backup Triggering CronJob  Skipping creating BackupSession. Reason: Backup Configuration demo/pause-backup is paused.
```

Look at the `Events` of the `BackupConfiguration` crd. The event shows that the creation of `BackupSession` has paused. This means the hole backup process has stopped.

**Skipped Instant Backup:**

If we try to create a `BackupSession` for instant backup  the `BackupSession` will be `Skipped`. Now, we are going to create a `BackupSession` crd for instant backup. Below is the YAML of the `BackupSession` that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupSession
metadata:
  labels:
    stash.appscode.com/backup-configuration: pause-backup
  name: instant-backupsession
  namespace: demo
spec:
  backupConfiguration:
    name: pause-backup
```

let's create the `BackupSession` we have shown above.

```console
kubectl apply -f ./docs/examples/guides/latest/advanced-use-case/pause-backup/backupsession.yaml
backupsession.stash.appscode.com/instant-backupsession created
```

Wait for next schedule for backup. Run the following command to watch `BackupSession` crd,

```console
Every 3.0s: kubectl get backupsession -n demo                      suaas-appscode: Thu Aug  1 19:10:39 2019

NAMESPACE   NAME                       BACKUPCONFIGURATION   PHASE       AGE
demo        instant-backupsession      pause-backup          Skipped     1m2s
demo        pause-backup-1564659789    pause-backup          Succeeded   2m
```

So, we can see from the output of the above command that the restore process has Skipped. If you describe the `BackupSession` object you will see that there is a warning for skipping BackupSession.

```yaml
$ kubectl describe backupsession -n demo deployment-backupsession
Name:         deployment-backupsession
Namespace:    demo
Labels:       stash.appscode.com/backup-configuration=pause-backup
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"stash.appscode.com/v1beta1","kind":"BackupSession","metadata":{"annotations":{},"labels":{"stash.appscode.com/backup-config...
API Version:  stash.appscode.com/v1beta1
Kind:         BackupSession
Metadata:
  Creation Timestamp:  2019-08-01T11:53:56Z
  Generation:          1
  Resource Version:    15745
  Self Link:           /apis/stash.appscode.com/v1beta1/namespaces/demo/backupsessions/deployment-backupsession
  UID:                 6125ea45-6ef5-41d4-8ae2-66ec71376f28
Spec:
  Backup Configuration:
    Name:  pause-backup
Status:
  Phase:  Skipped
Events:
  Type     Reason                Age   From                      Message
  ----     ------                ----  ----                      -------
  Warning  BackupSessionSkipped  49s   BackupSession Controller  Backup Configuration is paused
  Warning  BackupSessionSkipped  49s   BackupSession Controller  Backup Configuration is paused
```

## Cleaning Up

To clean up the Kubernetes resources created by this tutorial, run:

```console
kubectl delete -n demo deployment stash-demo
kubectl delete -n demo backupconfiguration pause-backup
kubectl delete -n demo repository gce-repo
kubectl delete -n demo backupsession deployment-backupsession
kubectl delete -n demo secret gce-secret
kubectl delete -n demo pvc --all
```