# Restore Volumes in PVC Template

This guide will show you how to restore backed up volumes in PVC Template using Stash.

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

>**Note:** YAML files used in this tutorial are stored in  [/docs/examples/guides/latest/advance-use-case/restore-in-pvc-template](/docs/examples/guides/latest/advance-use-case/restore-in-pvc-template) directory of [stashed/stash](https://github.com/stashed/stash) repository.

## Restore Deployment's volumes in PVC Template

Here we are going to restore a Deploymnent's volumes in PVC Template. At first, we will back up a Deployment's volumes then we will restore these volumes in PVC Template.

### Backup

Now, we will deploy a Deployment with two pvcs and generate some sample data in it. Then, we will backup these PVCs.

**Deploy Deployment:**

Below is the YAML of the Deployment and PVCs that we are going to create,

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
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: source-config
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
      - args: ["echo sample_data > /source/data/data.txt; echo config_data > /source/config/config.cfg && sleep 3000"]
        command: ["/bin/sh", "-c"]
        image: busybox
        imagePullPolicy: IfNotPresent
        name: busybox
        volumeMounts:
        - mountPath: /source/data
          name: source-data
        - mountPath: /source/config
          name: source-config
      restartPolicy: Always
      volumes:
      - name: source-data
        persistentVolumeClaim:
          claimName: source-data
      - name: source-config
        persistentVolumeClaim:
          claimName: source-config
```

The above Deployment will automatically create `data.txt` and `config.cfg` file in `/source/data` and `sourc/config` directory respectively and write some sample data in it.

Let's create the Deployment and PVCs we have shown above.

```console
$ kubectl apply -f ./docs/examples/guides/latest/advance-use-case/restore-in-pvc-template/deployment/deployment.yaml
persistentvolumeclaim/source-data created
persistentvolumeclaim/source-config created
deployment.apps/stash-demo created
```

Now, wait for pod of the Deployment to go into `Running` state.

```console
$ kubectl get pod -n demo 
NAME                          READY   STATUS    RESTARTS   AGE
stash-demo-67ccdfbbc7-z97rd   1/1     Running   0          77s
```

Verify that the sample data has been created in `/source/data` and `/source/config` directory using the following command,

```console
$ kubectl exec -n demo stash-demo-67ccdfbbc7-z97rd ls /source/data
data.txt
$ kubectl exec -n demo stash-demo-67ccdfbbc7-z97rd ls /source/config
config.cfg
```

**Create Repository:**

We are going to store our backed up data into a GCS bucket. We have to create a Secret and a Repository object with access credentials and backend information respectively.

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
$ kubectl apply -f ./docs/examples/guides/latest/advance-use-case/restore-in-pvc-template/repository.yaml
repository.stash.appscode.com/gcs-repo created
```

Now, we are ready to backup our volumes to our desired backend.

**Create BackupConfiguration:**

We have to create a `BackupConfiguration` crd targeting the `stash-demo` Deployment that we have deployed earlier. Then, Stash will inject a sidecar container into the target. It will also create a CronJob to take periodic backup of `/source/data` and `/source/config` directory of the target.

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
      - name: source-data
        mountPath: /source/data
      - name: source-config
        mountPath: /source/config
    directories:
      - /source/data
      - /source/config
  retentionPolicy:
    name: 'keep-last-5'
    keepLast: 5
    prune: true
```

Let's create the `BackupConfiguration` object that we have shown above,

```console
$ kubectl apply -f ./docs/examples/guides/latest/advance-use-case/restore-in-pvc-template/deployment/dep-backupconfiguration.yaml
backupconfiguration.stash.appscode.com/deployment-backup created
```

If everything goes well, Stash will create a `CronJob` to trigger backup periodically.

**Verify CronJob:**

Verify that Stash has created a `CronJob` to trigger a periodic backup of volumes of the Deployment by the following command,

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

We can see from the above output that the backup session has succeeded. This indicates that the volumes of the deployment have been stored in the backend successfully.

### Restore

Now, we will restore the volumes that we have backed up in the previous section into PVC Template. In order to do that, we have to create a `RestoreSession` object that specify the `volumeClaimTemplates`.

**Create RestoreSession:**

Below is the YAML of the `RestoreSession` object that we are going to create,

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
    - /source/data
    - /source/config
  target:
    volumeMounts:
    - name:  restore-data
      mountPath:  /source/data
    - name:  restore-config
      mountPath:  /source/config
    volumeClaimTemplates:
    - metadata:
        name:  restore-data
      spec:
        accessModes: [ "ReadWriteOnce" ]
        storageClassName: "standard"
        resources:
          requests:
            storage: 2Gi
    - metadata:
        name:  restore-config
      spec:
        accessModes: [ "ReadWriteOnce" ]
        storageClassName: "standard"
        resources:
          requests:
            storage: 2Gi
```

- `spec.target.volumeMounts` specifies the directory where the targeted PVC will be mounted inside the restore job.
- `spec.target.rules[*].paths` specifies the directories that will be restored from the backed up data.
- `spec.target.volumeClaimTemplates:` specifies a list of PVC templates that will be created by restoring volumes from respective backed up volumes.
  - `metadata.name` specifies the name of the restored PVC.

Let's create the `RestoreSession` object that we have shown above,

```console
$ kubectl apply -f ./docs/examples/guides/latest/advance-use-case/restore-in-pvc-template/deployment/restore-deployment.yaml
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

Once a restore process is complete, we will see that new PVCs with the name `restore-data` and `restore-config` have been created.

Verify that the PVCs have been created by the following command,

```console
$ kubectl get pvc -n demo
NAME             STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
restore-config   Bound    pvc-6aab94dc-10b2-4c36-8768-89b20a7a24ed   2Gi        RWO            standard       32s
restore-data     Bound    pvc-8296da99-b813-466a-b9f2-efff1faeee17   2Gi        RWO            standard       32s
```

Notice the STATUS field. `Bound` indicates that PVCs have been initialized from the respective backed up volumes.

**Verify Restored Data:**

We will create a new Deployment with the restored PVCs to verify whether the backed up data have been restored.

Below is the YAML of the Deployment that we are going to create,

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: stash-demo
  name: restore-demo
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
        - mountPath: /restore/data
          name: restore-data
        - mountPath: /source/config
          name: restore-config
      restartPolicy: Always
      volumes:
      - name: restore-data
        persistentVolumeClaim:
          claimName: restore-data
      - name: restore-config
        persistentVolumeClaim:
          claimName: restore-config
```

Let's create the deployment we have shown above.

```console
$ kubectl apply -f ./docs/examples/guides/latest/advance-use-case/restore-in-pvc-template/deployment/restore-deployment.yaml
deployment.apps/restore-demo created
```

Now, wait for pod of the deployment to go into the `Running` state.

```console
$ kubectl get pod -n demo
NAME                                READY   STATUS    RESTARTS   AGE
restore-demo-85fbcb5dcf-vpbt8       1/1     Running   0          2m50s
```

Verify that the sample data has been created in `/source/data` and `/source/config` directory using the following command,

```console
$ kubectl exec -n demo restore-demo-85fbcb5dcf-vpbt8 ls /restore/data
data.txt
$ kubectl exec -n demo restore-demo-85fbcb5dcf-vpbt8 ls /restore/config
config.cfg
```

## Restore SatefulSet's volumes in PVC Template

Here we are going to restore a StatefulSet’s volumes in PVC Template. At first, we will back up a StatefulSet’s volumes then we will restore these volumes in PVC Template.

### Backup

Now, we will deploy a StatefulSet and generate some sample data in it's volume. Then, we will backup these volumes.

**Deploy StatefulSet:**

Below is the YAML of the Statefulset that we are going to create,

```yaml
apiVersion: v1
kind: Service
metadata:
  name: headless
  namespace: demo
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
  namespace: demo
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
      - args: ["echo $(POD_NAME) > /source/data/data.txt; echo $(POD_NAME) > /source/config/config.cfg && sleep 3000"]
        command: ["/bin/sh", "-c"]
        env:
        - name:  POD_NAME
          valueFrom:
            fieldRef:
              fieldPath:  metadata.name
        name: busybox
        image: busybox
        ports:
        - containerPort: 80
          name: http
        volumeMounts:
        - name: source-data
          mountPath: "/source/data"
        - name: source-config
          mountPath: "/source/config"
        imagePullPolicy: IfNotPresent
  volumeClaimTemplates:
  - metadata:
      name: source-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "standard"
      resources:
        requests:
          storage: 2Gi
  - metadata:
      name: source-config
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "standard"
      resources:
        requests:
          storage: 2Gi
```

The above StatefulSet will automatically create `data.txt` and `config.cfg` file in `/source/data` and `sourc/config` directory respectively and write some sample data in it.

Let's create the Statefulset we have shown above.

```console
$ kubectl apply -f ./docs/examples/guides/latest/advance-use-case/restore-in-pvc-template/statefulset/statefulset.yaml
service/headless configured
statefulset.apps/stash-demo created
```

Now, wait for pod of the Statefulset to go into the `Running` state.

```console
$ kubectl get pod -n demo 
NAME           READY   STATUS    RESTARTS   AGE
stash-demo-0   1/1     Running   0          47s
stash-demo-1   1/1     Running   0          43s
stash-demo-2   1/1     Running   0          33s
```

Verify that the file has been created in `/source/data` and `/source/config` directory using the following command,

```console
$ kubectl exec -n demo stash-demo-0 ls /source/data
data.txt
$ kubectl exec -n demo stash-demo-0 ls /source/config
config.txt
```

**Create Repository:**

We are going to store our backed up data into a GCS bucket. We have to create a Secret and a Repository object with access credentials and backend information respectively.

Let’s create a secret called `gcs-secret` with access credentials of our desired GCS backend,

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

Let’s create the Repository object that we have shown above,

```console
$ kubectl apply -f ./docs/examples/guides/latest/advance-use-case/restore-in-pvc-template/repository.yaml
repository.stash.appscode.com/gcs-repo created
```

Now, we are ready to backup our volumes to our desired backend.

**Create BackupConfiguration:**

Now, create a `BackupConfiguration` crd to take periodic backup of the PVCs of `stash-demo` Statefulset.

Below is the YAML of the `BackupConfiguration` that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: ss-backup
  namespace: demo
spec:
  repository:
    name: gcs-repo
  schedule: "* * * * *"
  target:
    ref:
      apiVersion: apps/v1
      kind: StatefulSet
      name: stash-demo
    volumeMounts:
      - name: source-data
        mountPath: /source/data
      - name: source-config
        mountPath: /source/config
    directories:
      - /source/data
      - /source/config
  retentionPolicy:
    name: 'keep-last-5'
    keepLast: 5
    prune: true
```

Let’s create the `BackupConfiguration` object that we have shown above,

```console
$ kubectl apply -f ./docs/examples/guides/latest/advance-use-case/restore-in-pvc-template/statefulset/ss-backupconfiguration.yaml
backupconfiguration.stash.appscode.com/ss-backup created
```

If everything goes well, Stash will create a `CronJob` to trigger backup periodically.

**Verify CronJob:**

Verify that Stash has created a `CronJob` to trigger a periodic backup of the volumes of the Statefulset by the following command,

```console
$ kubectl get backupconfiguration -n demo
NAME        TASK   SCHEDULE      PAUSED   AGE
ss-backup          * * * * *              2m
```

**Wait for BackupSession:**

Now, wait for a backup schedule to appear. You can watch for `BackupSession` crd using the following command,

```console
$ watch -n 3 kubectl get backupsession -n demo
Every 3.0s: kubectl get backupsession -n demo                suaas-appscode: Tue Jul  9 17:09:43 2019

NAME                   BACKUPCONFIGURATION   PHASE       AGE
ss-backup-1562670004   ss-backup             Succeeded   9m39s
```

We can see from the above output that the backup session has succeeded. This indicates that the volumes of StatefulSet have been stored in the backend successfully.

### Restore

Now, we will restore the volumes that we have backed up in the previous section into PVC Template. In order to do that, we have to create a `RestoreSession` object that specify the `volumeClaimTemplates`.

**Create RestoreSession:**

Below is the YAML of the `RestoreSession` object that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: restore-statefulset
  namespace: demo
spec:
  repository:
    name: gcs-repo
  rules:
  - paths:
    - /source/data
    - /source/config
  target:
    replicas: 3
    volumeMounts:
    - name:  restore-data-restore-demo
      mountPath:  /source/data
    - name:  restore-config-restore-demo
      mountPath:  /source/config
    volumeClaimTemplates:
    - metadata:
        name: restore-data-restore-demo
      spec:
        accessModes: [ "ReadWriteOnce" ]
        storageClassName: "standard"
        resources:
          requests:
            storage: 2Gi
    - metadata:
        name: restore-config-restore-demo
      spec:
        accessModes: [ "ReadWriteOnce" ]
        storageClassName: "standard"
        resources:
          requests:
            storage: 2Gi
```

- `spec.target.replicas` `spec.target.replicas` specify the number of replicas of a StatefulSet whose volumes was backed up and Stash uses this field to dynamically create the desired number of PVCs and initialize them from respective Snapshots.
- `spec.target.volumeClaimTemplates:` specifies a list of PVC templates that will be created by restoring volumes from respective backed up volumes.
  - `metadata.name` Specifies the name of the restored PVC without pod ordinal. You must provide the name in the format:
  ```
  <claim name>-<statefulset name>
  ```

Let’s create the `RestoreSession` object that we have shown above,

```console
$ kubectl apply -f ./docs/examples/guides/latest/advance-use-case/restore-in-pvc-template/statefulset/restoresession.yaml
restoresession.stash.appscode.com/restore-statefulset created
```

**Wait for RestoreSession to Succeed:**

Once, you have created the `RestoreSession` crd, Stash will create a job to restore. We can watch the `RestoreSession` phase to check if the restore process is succeeded or not.

Run the following command to watch `RestoreSession` phase,

```console
$ watch -n 3 kubectl get restoresession -n demo
Every 3.0s: kubectl get restoresession -n demo               suaas-appscode: Tue Jul  9 18:14:44 2019

NAME                  REPOSITORY-NAME   PHASE       AGE
restore-statefulset   gcs-repo          Succeeded   2m2s
```

So, we can see from the output of the above command that the restore process succeeded.

**Verify Restored PVC:**

Once a restore process is complete, verify that new PVCs have been created successfully by the following command,

```console
$ kubectl get pvc -n demo
NAME                            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
restore-config-restore-demo-0   Bound    pvc-c575f88a-79c9-4d25-9aab-5f9822ced239   2Gi        RWO            standard       19s
restore-config-restore-demo-1   Bound    pvc-09d8e1dc-9b51-4a74-983e-8ef2ecde88b5   2Gi        RWO            standard       19s
restore-config-restore-demo-2   Bound    pvc-45fe4050-12dc-46cb-a797-63f8ea420e28   2Gi        RWO            standard       19s
restore-data-restore-demo-0     Bound    pvc-d27485e6-5377-4009-b2e2-9ddc2f9afaf3   2Gi        RWO            standard       19s
restore-data-restore-demo-1     Bound    pvc-ae605285-ef6c-4b02-958c-d34352972ff0   2Gi        RWO            standard       19s
restore-data-restore-demo-2     Bound    pvc-bd087508-9d9c-4ee0-955f-4cd822ab85f7   2Gi        RWO            standard       19s
```

Notice the STATUS field. `Bound` indicates that PVC has been initialized from the respective backed up volumes.

**Verify Restored Data:**

We will create a new Statefulset with the restored PVCs to verify whether the backed up data have been restored.

Below is the YAML of the StatefulSet that we are going to create,

```yaml
apiVersion: v1
kind: Service
metadata:
  name: re-headless
  namespace: demo
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
  name: restore-demo
  namespace: demo
  labels:
    app: stash-demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: stash-demo
  serviceName: re-headless
  template:
    metadata:
      labels:
        app: stash-demo
    spec:
      containers:
      - command:
        - sleep
        - '3600'
        name: busybox
        image: busybox
        ports:
        - containerPort: 80
          name: http
        volumeMounts:
        - name: restore-data
          mountPath: "/restore/data"
        - name: restore-config
          mountPath: "/restore/config"
        imagePullPolicy: IfNotPresent
  volumeClaimTemplates:
    - metadata:
        name: restore-data
      spec:
        accessModes: [ "ReadWriteOnce" ]
        storageClassName: "standard"
        resources:
          requests:
            storage: 2Gi
    - metadata:
        name: restore-config
      spec:
        accessModes: [ "ReadWriteOnce" ]
        storageClassName: "standard"
        resources:
          requests:
            storage: 2Gi
```

Let’s create the StatefulSet we have shown above.

```console
$ kubectl apply -f ./docs/examples/guides/latest/advance-use-case/restore-in-pvc-template/statefulset/restore-statefulset.yaml
service/re-headless created
statefulset.apps/restore-demo created
```

Now, wait for the pod of deployment to go into the `Running` state.

```console
$ kubectl get pod -n demo 
NAME             READY   STATUS    RESTARTS   AGE
restore-demo-0   1/1     Running   0          34s
restore-demo-1   1/1     Running   0          31s
restore-demo-2   1/1     Running   0          26s
```

Verify that the sample data has been created in `/restore/data` and `/restore/config` directory using the following command,

```console
$ kubectl exec -n demo restore-demo-0 ls /restore/data
data.txt
$ kubectl exec -n demo restore-demo-0 ls /restore/config
config.txt
```

## Cleanup

To clean up the Kubernetes resources created by this tutorial, run:

```yaml
kubectl delete -n demo deployment stash-demo
kubectl delete -n demo deployment restore-demo
kubectl delete -n demo statefulset stash-demo
kubectl delete -n demo statefulset restore-demo
kubectl delete -n demo backupconfiguration deployment-backup
kubectl delete -n demo backupconfiguration ss-backup
kubectl delete -n demo restoresession restore-deployment
kubectl delete -n demo restoresession restore-statefulset
kubectl delete -n demo repository gcs-repo
kubectl delete -n demo secret gcs-secret
kubectl delete -n demo pvc --all
```