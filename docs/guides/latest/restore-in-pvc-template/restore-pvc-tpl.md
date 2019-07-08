# Restore Data in PVC Template

This guide will show you how to restore backed up data in PVC Template using Stash. Here we are going to backup Deploymnen's and StatefulSet's data then we will restore them in PVC Template.

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [Minikube](https://github.com/kubernetes/minikube).

- Install `Stash` in your cluster following the steps [here](https://appscode.com/products/stash/0.8.3/setup/install/).

- You should be familiar with the following `Stash` concepts:
  - [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md/)
  - [BackupSession](/docs/concepts/crds/backupsession.md/)
  - [Repository](/docs/concepts/crds/repository.md/)
  - [RestoreSession](/docs/concepts/crds/restoresession.md/)
  
To keep everything isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$ kubectl create ns demo
namespace/demo created
```

>**Note:** YAML files used in this tutorial are stored in  [docs/examples/guides/latest/volumes](/docs/examples/guides/latest/restore-in-pvc-template) directory of [stashed/stash](https://github.com/stashed/stash) repository.

## Restore Deployment's data in PVC Template

Here, we are going to backup a Deploymnet's data then we will restore that data in PVC template.

### Backup

We will deploy a Deployment with two PVCs. This Deployment will automatically generate sample data in `/source/data-1` and `sourc/data-2` directory where we have mounted the PVCs. Then, we will take backup of those PVCs using Stash.

Below is the YAML of the Deployment and PVC's that we are going to create,

**Deploy Deployment:**

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: source-pvc-1
  namespace: demo
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 6Gi
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: source-pvc-2
  namespace: demo
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 6Gi
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
      - args: ["touch /source/data-1/file1.txt /source/data-2/file2.txt && sleep 3000"]
        command: ["/bin/sh", "-c"]
        image: busybox
        imagePullPolicy: IfNotPresent
        name: busybox
        volumeMounts:
        - mountPath: /source/data-1
          name: source-data-1
        - mountPath: /source/data-2
          name: source-data-2
      restartPolicy: Always
      volumes:
      - name: source-data-1
        persistentVolumeClaim:
          claimName: source-pvc-1
      - name: source-data-2
        persistentVolumeClaim:
          claimName: source-pvc-2
```

Let's create the Deployment and PVCs we have shown above.

```console
$ kubectl apply -f ./deployment.yaml
persistentvolumeclaim/source-pvc-1 created
persistentvolumeclaim/source-pvc-2 created
deployment.apps/stash-demo created
```

Now, wait for Deployment’s pod to go into the Running state.

```console
$ kubectl get pod -n demo 
NAME                          READY   STATUS    RESTARTS   AGE
stash-demo-5459cd4485-zp6v6   1/1     Running   0          57s
```

Verify that the sample data has been created in `/source/data-1` and `/source/data-2` directory using the following command,

```console
$ kubectl exec -n demo stash-demo-5459cd4485-zp6v6 ls source/data-1
file1.txt
$ kubectl exec -n demo stash-demo-5459cd4485-zp6v6 ls source/data-2
file2.txt
```

**Create Repository:**

We are going to store our backed up data into a GCS bucket using Stash. We have to create a Secret and a Repository object with access credentials and backend information respectively.

Let's create a secret called `gcs-secret` with access credentials of our desired GCS backend,

```console
$ echo -n 'changeit' > RESTIC_PASSWORD
$ echo -n '<your-project-id>' > GOOGLE_PROJECT_ID
$ cat /path/to/downloaded/sa_key_file.json > GOOGLE_SERVICE_ACCOUNT_JSON_KEY
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
      prefix: /source/data/restore-pvc-in-temp
    storageSecretName: gcs-secret
```

Let's create the `Repository` object that we have shown above,

```console
$ kubectl apply -f ./repository.yaml
repository.stash.appscode.com/gcs-repo created
```

Now, we are ready to backup our volumes to our desired backend.

**Create BackupConfiguration:**

We have to create a `BackupConfiguration` crd targeting the `stash-demo` Deployment that we have deployed earlier. Then, Stash will inject a sidecar container into the target. It will also create a CronJob to take periodic backup of `/source/data-1` and `/source/data-2` directory of the target.

Below is the YAML of the `BackupConfiguration` object that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: deployment-backup
  namespace: demo
spec:
  repository:
    name: gcs-repo
  schedule: "* * * * *"
  target:
    ref:
      apiVersion: apps/v1
      kind: Deployment
      name: stash-demo
    volumeMounts:
      - name: source-data-1
        mountPath: /source/data-1
      - name: source-data-2
        mountPath: /source/data-2
    directories:
      - /source/data-1
      - /source/data-2
  retentionPolicy:
    name: 'keep-last-5'
    keepLast: 5
    prune: true
```

Let's create the `BackupConfiguration` object that we have shown above,

```console
$ kubectl apply -f ./dep-backupconfiguration.yaml
backupconfiguration.stash.appscode.com/deployment-backup created
```

If everything goes well, Stash will create a CronJob to trigger backup periodically.

**Verify CronJob:**

Verify that Stash has created a CronJob to trigger a periodic backup of the Deployment's volume by the following command,

```console
$ kubectl get backupconfiguration -n demo
NAME                TASK   SCHEDULE    PAUSED   AGE
deployment-backup          * * * * *            36s
```

**Wait for BackupSession:**

Now, wait for a backup schedule to appear. You can watch for `BackupSession` crd using the following command,

```console
$ watch -n 3 kubectl get backupconfiguration -n demo
Every 3.0s: kubectl get backupconfiguration --all-namespaces                  suaas-appscode: Mon Jul  8 18:20:47 2019

NAMESPACE   NAME                           BACKUPCONFIGURATION   PHASE       AGE
demo        deployment-backup-1562588351   deployment-backup     Succeeded   96s
```

We can see from the above output that the backup session has succeeded. This indicates that deployment's data have been stored in the backend successfully.

### Restore

This section will show you how to restore the backed up data in PVC Template using stash. Here, we will restore the data we have backed up in the previous section.

**Create RestoreSession:**

Now, we will create a `RestoreSession` object to restore the backed up data into the PVC Template. Below is the YAML of the `RestoreSession` object that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: restore-deployment
  namespace: demo
spec:
  repository:
    name: gcs-repo
  rules:
  - paths:
    - /source/data-1
    - /source/data-2
  target:
    volumeMounts:
    - name:  restore-data-1
      mountPath:  /source/data-1
    - name:  restore-data-2
      mountPath:  /source/data-2
    volumeClaimTemplates:
    - metadata:
        name:  restore-data-1
      spec:
        accessModes: [ "ReadWriteOnce" ]
        storageClassName: "standard"
        resources:
          requests:
            storage: 1Gi
    - metadata:
        name:  restore-data-2
      spec:
        accessModes: [ "ReadWriteOnce" ]
        storageClassName: "standard"
        resources:
          requests:
            storage: 1Gi
```

- `spec.target.volumeMounts` specifies the directory where the targeted PVC will be mounted inside the restore job.
- `spec.target.rules[*].paths` specifies the directories that will be restored from the backed up data.

Let's create the `RestoreSession` object that we have shown above,

```console
$ kubectl apply -f ./restore-deployment.yaml
restoresession.stash.appscode.com/restore-deployment created
```

**Wait for RestoreSession to Succeed:**

Once, you have created the `RestoreSession` crd, Stash will create a job to restore. We can watch the `RestoreSession` phase to check if the restore process is succeeded or not.

Run the following command to watch RestoreSession phase,

```console
$ watch -n 3 kubectl get restoresession -n demo

Every 3.0s: kubectl get restoresession -n demo               suaas-appscode: Mon Jul  8 18:39:58 2019

NAME                 REPOSITORY-NAME   PHASE       AGE
restore-deployment   gcs-repo          Succeeded   3m27s
```

So, we can see from the output of the above command that the restore process succeeded.

**Verify Restored PVC:**

Once a restore process is complete, we will see that new PVCs with the name `restore-data-1` and `restore-data-2` has been created.

Verify that the PVCs has been created by the following command,

```console
$ kubectl get pvc -n demo
NAME             STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
restore-data-1   Bound    pvc-505a80a1-37a3-4ce8-b632-de870666b498   1Gi        RWO            standard       5m36s
restore-data-2   Bound    pvc-c2c21e8e-3466-4098-b8eb-231f4bfea2e5   1Gi        RWO            standard       5m36s
```

Notice the STATUS field. `Bound` indicates that PVC has been initialized from the respective snapshot.

**Verify Restored Data:**

We will create a new Deployment with the restored PVCs to verify whether the backed up data has been restored.

Below is the YAML of the Deployment that we are going to create,

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: stash-demo
  name: restore-stash-demo
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
      - args:
        - sleep
        - "3600"
        image: busybox
        imagePullPolicy: IfNotPresent
        name: busybox
        volumeMounts:
        - mountPath: /source/data-1
          name: restore-data-1
        - mountPath: /source/data-2
          name: restore-data-2
      restartPolicy: Always
      volumes:
      - name: restore-data-1
        persistentVolumeClaim:
          claimName: restore-data-1
      - name: restore-data-2
        persistentVolumeClaim:
          claimName: restore-data-2
```

Let's create the deployment we have shown above.

```console
$ kubectl apply -f ./restore-deployment.yaml
deployment.apps/restore-stash-demo created
```

Now, wait for deployment’s pod to go into the `Running` state.

```console
$ kubectl get pod -n demo
NAME                                READY   STATUS    RESTARTS   AGE
restore-stash-demo-d9d54b8f-xnddz   1/1     Running   0          2m50s
```

Verify that the sample data has been created in `/source/data-1` and `/source/data-2` directory using the following command,

```console
 $ kubectl exec -n demo restore-stash-demo-d9d54b8f-xnddz ls /source/data-1
file1.txt
$ kubectl exec -n demo restore-stash-demo-d9d54b8f-xnddz ls /source/data-2
file2.txt
```

## Restore SatefulSet's data in PVC Template

Now, wait for the restore process to complete. You can watch the `RestoreSession` phase using the following command,

```console
```

From the output of the above command, we can see that restore has been completed successfully.



### Backup

**Deploy StatefulSet:**

**Create Repository:**

**Create BackupConfiguration:**

### Restore

**Create RestoreSession:**

**Verify Restored PVC:**

**Verify Restored Data:**

## Cleanup