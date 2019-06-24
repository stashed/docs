# Backup and Restore Deployment's Volumes

This guide will show you how to use Stash to backup and restore Deployment's volumes.

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [Minikube](https://github.com/kubernetes/minikube).

- Install `Stash` in your cluster following the steps [here](https://appscode.com/products/stash/0.8.3/setup/install/).

- You should be familiar with the following `Stash` concepts:
  - [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md/)
  - [BackupSession](/docs/concepts/crds/backupsession.md/)
  - [RestoreSession](/docs/concepts/crds/restoresession.md/)
  - [Repository](/docs/concepts/crds/repository.md/)

To keep everything isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$ kubectl create ns demo
namespace/demo created
```

>Note: YAML files used in this tutorial are stored in [/docs/examples/volume-snapshot](/docs/examples/volume-snapshot) directory of [appscode/stash](https://github.com/stashed/stash) repository.

This section will show you how to use Stash to backup Deployment's volumes. 

## Backup Deployment's Volumes

Here, we are going to deploy a Deployment with PVC and generate some sample data on it. Then, we will take backup of those PVC using Stash.

### Deploy workload

At first, we will create a PVC then we will create a Deployment for backup.

**Create PVC:**

let's create a sample PVC. We will mount this PVC in our targeted Deployment.

Below is the YAML of the sample PVC,

```console
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
```

Create the PVC we have shown above.

```console
$ kubectl apply -f ./pvc.yaml 
persistentvolumeclaim/stash-sample-data created
```

**Deploy Deployment:**

Now, we will deploy a Deployment that uses the above PVC. This Deployment will automatically generate sample data (sample-file.txt file) in /source/data directory where we have mounted the desired PVC.

Below is the YAML of the Deployment that we are going to create,

```console
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
      - args: ["touch source/data/sample-file.txt && sleep 3000"]
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

Let's create the deployment we have shown above.

```console
$ kubectl apply -f ./deployment.yaml 
deployment.apps/stash-demo created
```

Now, wait for deployment’s pod to go into the `Running` state.

```console
$ kubectl get pod -n demo 
NAME                         READY   STATUS    RESTARTS   AGE
stash-demo-8cfcbcc89-2z6mq   1/1     Running   0          30s
stash-demo-8cfcbcc89-j9wbc   1/1     Running   0          30s
stash-demo-8cfcbcc89-q8xfd   1/1     Running   0          30s
```

Verify that the sample data has been created in /source/data directory using the following command,

```console
$ kubectl exec -n demo stash-demo-8cfcbcc89-2z6mq -- ls -R /source/data
/source/data:
sample-file.txt
```

### Prepare Backend

We are going to store our back up data into our local backend. At first, we need to create a secret then we need to create a `Repository` crd. If you want to use a different backend, please read the respective backend configuration doc from [here](https://appscode.com/products/stash/0.8.3/guides/backends/overview/).

**Create Secret:**

Let's create a secret called `local-secret` for local backend,

```console
$ echo -n 'changeit' > RESTIC_PASSWORD
$ kubectl create secret generic -n demo local-secret \
    --from-file=./RESTIC_PASSWORD
secret/local-secret created
```

Verify that the secret has been created successfully,

```console
$ kubectl get secret -n demo local-secret -o yaml
```

**Create Repository:**

Now, create a `Respository` using this secret. 

Below is the YAML of `Repository` crd we are going to create,

```console
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: repo-pvc
  namespace: demo
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: local-repo
  namespace: demo
spec:
  backend:
    local:
      mountPath: /safe/data
      persistentVolumeClaim:
        claimName: repo-pvc
    storageSecretName: local-secret
```

Let's create the Repository we have shown above,

```console
$ kubectl apply -f ./local_repository.yaml 
persistentvolumeclaim/repo-pvc created
repository.stash.appscode.com/local-repo created
```

Now, we are ready to backup our volumes to our desired backend.

### Backup

We have to create a `BackupConfiguration` crd. Then Stash will create a `CronJob` to take periodically backup of the PVC of `stash-demo` Deployment.

**Create BackupConfiguration:**

Below is the YAML of the `BackupConfiguration` crd that we are going to create,

```console
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: deployment-backup
  namespace: demo
spec:
  repository:
    name: local-repo
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

Here,

- `spec.repository` refers to the `laocal-repo` local Backend.
- `spec.schedule` is a cron expression indicates that `BackupSession` will be created at 1 minute interval.
- `spec.target.ref` refers to the target workload that was created for `stash-demo` Deploymnet.

Let's create the `BackupConfiguration` crd we have shown above,

```console
 $ kubectl apply -f ./backupconfiguration.yaml
backupconfiguration.stash.appscode.com/deployment-backup created
```

**Verify CronJob:**

If everything goes well, Stash will create a `CronJob` with the schedule specified in `spec.schedule` field of `BackupConfiguration` crd.

Verify that the `CronJob` has been created using the following command,

```console
$ kubectl get cronjob -n demo
NAME                SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
deployment-backup   */1 * * * *   False     0        35s             64s
```

**Wait for BackupSession:**

The `deployment-backup` CronJob will trigger a backup on each schedule by creating a `BackpSession` crd.

Wait for a schedule to appear. Run the following command to watch `BackupSession` crd,

```console
$ watch -n 2 kubectl get backupsession -n demo 
Every 1.0s: kubectl get backupsession -n demo     suaas-appscode: Mon Jun 24 10:23:08 2019

NAME                           BACKUPCONFIGURATION   PHASE       AGE
deployment-backup-1561350065   deployment-backup     Running     30s
deployment-backup-1561350125   deployment-backup     Succeeded   63s
```

We can see above that the backup session has succeeded. Now, we will verify that the backed up data has been stored in the local backend.

**Verify Backup:**

Once a backup is complete, Stash will update the respective `Repository` crd to reflect the backup. Check that the repository `local-repo` has been updated by the following command,

```console
 $ kubectl get repository -n demo local-repo
NAME         INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
local-repo   true        0 B    1                27s                      1m4s
```

## Restore Deployment's Volumes

This section will show you how to restore the backed up data from the backend we have taken in earlier section.

**Deploy Deployment:**

Now, we have to deploy a new Deployment whether the backed up data has been restored.

Below is the YAML of the Deployment that we are going to create,

```console
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: demo-pvc
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
  name: stash-recovered
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
      - args:
        - sleep
        - "3600"
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
          claimName: demo-pvc
```

Let's create the deployment we have shown above.

```console
$ kubectl apply -f ./recovered_deployment.yaml 
persistentvolumeclaim/demo-pvc created
deployment.apps/stash-recovered created
```

Now, wait for deployment’s pod to go into the `Running` state.

```console
$ kubectl get pod -n demo 
NAME                               READY   STATUS    RESTARTS   AGE
stash-recovered-676bd87957-7bcfg   1/1     Running   0          31s
stash-recovered-676bd87957-pv4hg   1/1     Running   0          31s
stash-recovered-676bd87957-q2hbl   1/1     Running   0          31s

```

**Create RestoreSession:**

We need to create a `RestoreSession` crd to restore the backed up data in the target volume from backend.

Below is the YAML of the `RestoreSesion` crd that we are going to create,

```console
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: deployment-restore
  namespace: demo
spec:
  repository:
    name: local-repo
  rules:
    - paths:
        - /source/data
  target: # target indicates where the recovered data will be stored
    ref:
      apiVersion: apps/v1
      kind: Deployment
      name: stash-recovered
    volumeMounts:
      - name:  source-data
        mountPath:  /source/data
```

Here,

`spec.repository.name` specifies the `Repository` crd that holds the backend information where our backed up data has been stored.

`spec.target.ref` refers to the target workload where the recovered data will be stored.

Let's create the `RestoreSession` crd we have shown above,

```console
$ kubectl apply -f ./restoresession.yaml
restoresession.stash.appscode.com/deployment-restore created
```

Once, you have created the `RestoreSession` crd, Stash will create a job to restore. We can watch the `RestoreSession` phase to check if the restore process is `succeeded` or not.

Run the following command to watch RestoreSession phase,

```console
$ watch -n 2 kubectl get restore -n demo
Every 5.0s: kubectl get restore -n demo           suaas-appscode: Mon Jun 24 10:33:57 2019

NAME                 REPOSITORY-NAME   PHASE       AGE
deployment-restore   local-repo        Succeeded   2m56s
```

So, we can see from the output of the above command that the restore process succeeded.

**Verify Restored Data:**

In this section, we will verify that the desired data has been restored successfully.

Verify that the sample data has been restored in `/source/data` directory of the `stash-recovered` deployment's pod using the following command,

```console
$ kubectl exec -n demo stash-recovered-7dc79fc894-csgtn -- ls -R /source/data
/source/data:
sample-file.txt
```

# Cleaning Up

To clean up the Kubernetes resources created by this tutorial, run:

```console
kubectl delete -n demo deployment stash-demo
kubectl delete -n demo deployment stash-recovered
kubectl delete -n demo backupconfiguration deployment-backup
kubectl delete -n demo restoresession deployment-restore
kubectl delete -n demo repository local-repo
kubectl delete -n demo pvc --all
```