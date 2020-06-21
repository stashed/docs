---
title: Clone Data Volumes | Stash
description: An step by step guide on how to clone data volumes using Stash.
menu:
  docs_{{ .version }}:
    identifier: advance-use-case-clone-pvc
    name: Clone Data Volumes
    parent: advance-use-case
    weight: 30
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: guides
---

# Clone Data Volumes using Stash

Using Stash you can clone data volumes of a workload into a different namespace in a cluster. You can provide a template for PVC in `RestoreSession`. Stash will create PVCs according to the template, then it will restore data in that PVC. Then you can use the cloned PVCs to deploy your desired workload. This guide will show you how to clone backed up data into new PVCs using Stash.

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [kind](https://kind.sigs.k8s.io/docs/user/quick-start/).

- Install `Stash` in your cluster following the steps [here](/docs/setup/install.md).

- You should be familiar with the following `Stash` concepts:
  - [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md)
  - [BackupSession](/docs/concepts/crds/backupsession.md)
  - [Repository](/docs/concepts/crds/repository.md)
  - [RestoreSession](/docs/concepts/crds/restoresession.md)

To keep everything isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$ kubectl create ns demo
namespace/demo created
```

> **Note:** YAML files used in this tutorial are stored in [/docs/examples/guides/latest/advanced-use-case/clone-pvc](/docs/examples/guides/latest/advanced-use-case/clone-pvc) directory of [stashed/docs](https://github.com/stashed/docs) repository.

## Clone the Volumes of a Deployment

Here we are going to clone the volumes of a Deployment. At first, we are going to back up the volumes of a Deployment, then we are going to restore these volumes into new PVCs.

### Backup

We are going to deploy a Deployment with two PVCs and generate some sample data in it. Then, we are going to back up these PVCs.

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

The above Deployment will automatically create `data.txt` and `config.cfg` file in `/source/data` and `/source/config` directory respectively and write some sample data in it.

Let's create the Deployment and PVCs we have shown above.

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/advanced-use-case/clone-pvc/deployment/deployment.yaml
persistentvolumeclaim/source-data created
persistentvolumeclaim/source-config created
deployment.apps/stash-demo created
```

Now, wait for the pod of the Deployment to go into `Running` state.

```console
$ kubectl get pod -n demo
NAME                          READY   STATUS    RESTARTS   AGE
stash-demo-67ccdfbbc7-z97rd   1/1     Running   0          77s
```

Verify that the sample data has been created in `/source/data` and `/source/config` directory using the following commands,

```console
$ kubectl exec -n demo stash-demo-67ccdfbbc7-z97rd -- cat /source/data/data.txt
sample_data
$ kubectl exec -n demo stash-demo-67ccdfbbc7-z97rd -- cat /source/config/config.cfg
config_data
```

**Create Repository:**

We are going to store our backed up data into a GCS bucket. We have to create a Secret and a Repository object with access credentials and backend information respectively.

> For GCS backend, if the bucket does not exist, Stash needs `Storage Object Admin` role permissions to create the bucket. For more details, please check the following [guide](/docs/guides/latest/backends/gcs.md).

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
      prefix: /source/data/restore-pvc-in-temp
    storageSecretName: gcs-secret
```

Let's create the `Repository` object that we have shown above,

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/advanced-use-case/clone-pvc/repository.yaml
repository.stash.appscode.com/gcs-repo created
```

Now, we are ready to backup our volumes to our desired backend.

**Create BackupConfiguration:**

Now, create a `BackupConfiguration` crd to take periodic backup of the PVCs of `stash-demo` Deployment.

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
    paths:
    - /source/data
    - /source/config
  retentionPolicy:
    name: 'keep-last-5'
    keepLast: 5
    prune: true
```

Let's create the `BackupConfiguration` object that we have shown above,

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/advanced-use-case/clone-pvc/deployment/dep-backupconfiguration.yaml
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

Now, wait for the next backup schedule. You can watch for `BackupSession` crd using the following command,

```console
$ watch -n 3 kubectl get backupconfiguration -n demo
Every 3.0s: kubectl get backupconfiguration -n demo                  suaas-appscode: Mon Jul  8 18:20:47 2019

NAME                           INVOKER-TYPE          INVOKER-NAME        PHASE       AGE
deployment-backup-1562588351   BackupConfiguration   deployment-backup   Succeeded   96s
```

We can see from the above output that the backup session has succeeded. This indicates that the volumes of the Deployment have been backed up in the backend successfully.

### Restore

Now, we are going to clone the volumes that we have backed up in the previous section. To do that, we have to create a `RestoreSession` object with `volumeClaimTemplates`.

**Stop Taking Backup of the Old Deployment:**

At first, let's pause the scheduled backup of the old Deployment so that no backup is taken during the restore process. To pause the `deployment-backup` BackupConfiguration, run:

```console
$ kubectl patch backupconfiguration -n demo deployment-backup --type="merge" --patch='{"spec": {"paused": true}}'
backupconfiguration.stash.appscode.com/deployment-backup patched
```

Now, wait for a moment. Stash will pause the BackupConfiguration. Verify that the BackupConfiguration  has been paused,

```console
$ kubectl get backupconfiguration -n demo
NAME                TASK   SCHEDULE      PAUSED   AGE
deployment-backup          */1 * * * *   true     26m
```

Notice the `PAUSED` column. Value `true` for this field means that the BackupConfiguration has been paused.

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

Here,

- `spec.target.volumeMounts` specifies the directory where the newly created PVC will be mounted inside the restore job.
- `spec.rules[*].paths` specifies the file paths that will be restored from the backed up data.
- `spec.target.volumeClaimTemplates:` a list of PVC templates that will be created by Stash to restore the respective backed up data.
  - `name` specifies the name of the volume mountPath. This name must be the same as the volumeClaimTemplate name.
  - `mountPath` must be same `mountPath` as the original volume because Stash stores absolute path of the backed up files. If you use different `mountPath` for the restored volume the backed up files will not be restored into your desired volume.

Let's create the `RestoreSession` object that we have shown above,

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/advanced-use-case/clone-pvc/deployment/restoresession.yaml
restoresession.stash.appscode.com/restore-deployment created
```

**Wait for RestoreSession to Succeed:**

Once, you have created the `RestoreSession` crd, Stash will create a job to restore PVCs. We can watch the `RestoreSession` phase to check if the restore process has succeeded or not.

Run the following command to watch RestoreSession phase,

```console
$ watch -n 3 kubectl get restoresession -n demo
Every 3.0s: kubectl get restoresession -n demo               suaas-appscode: Mon Jul  8 18:39:58 2019

NAME                 REPOSITORY-NAME   PHASE       AGE
restore-deployment   gcs-repo          Succeeded   3m27s
```

So, we can see from the output of the above command that the restore process has completed successfully.

**Verify Restored PVC:**

Once the restore process is complete, we are going to see that new PVCs with the name `restore-data` and `restore-config` have been created.

Verify that the PVCs have been created by the following command,

```console
$ kubectl get pvc -n demo
NAME             STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
restore-config   Bound    pvc-6aab94dc-10b2-4c36-8768-89b20a7a24ed   2Gi        RWO            standard       32s
restore-data     Bound    pvc-8296da99-b813-466a-b9f2-efff1faeee17   2Gi        RWO            standard       32s
```

**Verify Restored Data:**

Let's create a new Deployment with the restored PVCs to verify whether the backed up data have been restored.

Below is the YAML of the Deployment that we are going to create,

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: restore-demo
  name: restore-demo
  namespace: demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: restore-demo
  template:
    metadata:
      labels:
        app: restore-demo
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
        - mountPath: /restore/config
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

Create the deployment we have shown above.

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/advanced-use-case/clone-pvc/deployment/restore-deployment.yaml
deployment.apps/restore-demo created
```

Now, wait for the pod of the Deployment to go into the `Running` state.

```console
$ kubectl get pod -n demo
NAME                                READY   STATUS    RESTARTS   AGE
restore-demo-85fbcb5dcf-vpbt8       1/1     Running   0          2m50s
```

Verify that the backed up data has been restored in `/source/data` and `/source/config` directory using the following command,

```console
$ kubectl exec -n demo restore-demo-85fbcb5dcf-vpbt8 -- cat /restore/data/data.txt
sample_data
$ kubectl exec -n demo restore-demo-85fbcb5dcf-vpbt8 -- cat /restore/config/config.cfg
config_data
```

## Clone the Volumes of a SatefulSet

Here we are going to clone the volumes of a StatefulSet. At first, we are going to back up the volumes of a StatefulSet, then we are going to restore these volumes into new PVCs.

### Backup

Now, we are going to deploy a StatefulSet and generate some sample data in its volume. Then, we are going to back up these volumes.

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

The above StatefulSet will automatically create `data.txt` and `config.cfg` file in `/source/data` and `/source/config` directory respectively and write some sample data in it.

Let's create the Statefulset we have shown above.

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/advanced-use-case/clone-pvc/statefulset/statefulset.yaml
service/headless configured
statefulset.apps/stash-demo created
```

Now, wait for the pod of the Statefulset to go into the `Running` state.

```console
$ kubectl get pod -n demo
NAME           READY   STATUS    RESTARTS   AGE
stash-demo-0   1/1     Running   0          47s
stash-demo-1   1/1     Running   0          43s
stash-demo-2   1/1     Running   0          33s
```

Verify that the sample data has been created in `/source/data` and `/source/config` directory using the following command,

```console
$ kubectl exec -n demo stash-demo-0 -- cat /source/data/data.txt
stash-demo-0
$ kubectl exec -n demo stash-demo-0 -- cat /source/config/config.cfg
stash-demo-0
$ kubectl exec -n demo stash-demo-1 -- cat /source/data/data.txt
stash-demo-1
$ kubectl exec -n demo stash-demo-1 -- cat /source/config/config.cfg
stash-demo-1
$ kubectl exec -n demo stash-demo-2 -- cat /source/data/data.txt
stash-demo-2
$ kubectl exec -n demo stash-demo-2 -- cat /source/config/config.cfg
stash-demo-2
```

**Create Repository:**

We are going to store our backed up data into a GCS bucket. Let’s create a secret called `gcs-secret` with access credentials of our desired GCS backend,

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
      prefix: /source/data/restore-pvc-in-temp
    storageSecretName: gcs-secret
```

Let’s create the Repository object that we have shown above,

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/advanced-use-case/clone-pvc/repository.yaml
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
  schedule: "*/3 * * * *"
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
    paths:
    - /source/data
    - /source/config
  retentionPolicy:
    name: 'keep-last-5'
    keepLast: 5
    prune: true
```

Let’s create the `BackupConfiguration` object that we have shown above,

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/advanced-use-case/clone-pvc/statefulset/ss-backupconfiguration.yaml
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

Now, wait for the next backup schedule. You can watch for `BackupSession` crd using the following command,

```console
$ watch -n 3 kubectl get backupsession -n demo
Every 3.0s: kubectl get backupsession -n demo                suaas-appscode: Tue Jul  9 17:09:43 2019

NAME                   INVOKER-TYPE          INVOKER-NAME   PHASE       AGE
ss-backup-1562670004   BackupConfiguration   ss-backup      Succeeded   9m39s
```

We can see from the above output that the backup session has succeeded. This indicates that the volumes of the Deployment have been backed up in the backend successfully.

### Restore

Now, we are going to restore the volumes that we have backed up in the previous section. To do that, we have to create a `RestoreSession` object with `volumeClaimTemplates`.

**Stop Taking Backup of the Old StatefulSet:**

At first, let's pause the scheduled backup of the old StatefulSet so that no backup is taken during the restore process. To pause the `ss-backup` BackupConfiguration, run:

```console
$ kubectl patch backupconfiguration -n demo ss-backup --type="merge" --patch='{"spec": {"paused": true}}'
backupconfiguration.stash.appscode.com/ss-backup patched
```

Now, wait for a moment. Stash will pause the BackupConfiguration. Verify that the BackupConfiguration  has been paused,

```console
$ kubectl get backupconfiguration -n demo
NAME                TASK   SCHEDULE      PAUSED   AGE
ss-backup                  */1 * * * *   true     26m
```

Notice the `PAUSED` column. Value `true` for this field means that the `BackupConfiguration` has been paused.

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
        name: restore-data-restore-demo-${POD_ORDINAL}
      spec:
        accessModes: [ "ReadWriteOnce" ]
        storageClassName: "standard"
        resources:
          requests:
            storage: 2Gi
    - metadata:
        name: restore-config-restore-demo-${POD_ORDINAL}
      spec:
        accessModes: [ "ReadWriteOnce" ]
        storageClassName: "standard"
        resources:
          requests:
            storage: 2Gi
```

- `spec.target.replicas` `spec.target.replicas` specify the number of replicas of a StatefulSet whose volumes were backed up and Stash uses this field to dynamically create the desired number of PVCs and initialize them from respective Volumes.
- `spec.target.volumeMounts`  specifies a list of volumes and their mountPath where the data will be restored.
  - `name` specifies the name of the volume mountPath. This name must be the same as the volumeClaimTemplate name without the `POD_ORDINAL` part.
  - `mountPath` must be same `mountPath` as the original volume because Stash stores absolute path of the backed up files. If you use different `mountPath` for the restored volume the backed up files will not be restored into your desired volume.
- `spec.target.volumeClaimTemplates:` a list of PVC templates that will be created by Stash to restore the respective backed up data.
  - `metadata.name` is a template for the name of the restored PVC that will be created by Stash. You have to provide this named template to match with your desired StatefulSet's PVC. For example, if you want to deploy a StatefulSet named `stash-demo` with `volumeClaimTemplate` name `my-volume`, your StatefulSet's PVC will be`my-volume-stash-demo-0`, `my-volume-stash-demo-1` and so on. In this case, you have to provide `volumeClaimTemplate` name in RestoreSession in the following format:

    ```console
    <pvc name>-<statefulset name>-${POD_ORDINAL}
    ```

    So for the above example, `volumeClaimTemplate` name for `RestoreSession` will be `my-volume-restore-demo-${POD_ORDINAL}`.
    The `${POD_ORDINAL}` variable is resolved by Stash.

Let’s create the `RestoreSession` object that we have shown above,

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/advanced-use-case/clone-pvc/statefulset/restoresession.yaml
restoresession.stash.appscode.com/restore-statefulset created
```

**Wait for RestoreSession to Succeed:**

Once, you have created the `RestoreSession` crd, Stash will create a job to restore. We can watch the `RestoreSession` phase to check if the restore process has succeeded or not.

Run the following command to watch `RestoreSession` phase,

```console
$ watch -n 3 kubectl get restoresession -n demo
Every 3.0s: kubectl get restoresession -n demo               suaas-appscode: Tue Jul  9 18:14:44 2019

NAME                  REPOSITORY-NAME   PHASE       AGE
restore-statefulset   gcs-repo          Succeeded   2m2s
```

So, we can see from the output of the above command that the restore process succeeded.

**Verify Restored PVC:**

Once the restore process is complete, verify that new PVCs have been created successfully by the following command,

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

**Verify Restored Data:**

Let's create a new Statefulset with the restored PVCs to verify whether the backed up data have been restored.

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
    app: restore-demo
  clusterIP: None
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: restore-demo
  namespace: demo
  labels:
    app: restore-demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: restore-demo
  serviceName: re-headless
  template:
    metadata:
      labels:
        app: restore-demo
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

Create the StatefulSet we have shown above.

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/advanced-use-case/clone-pvc/statefulset/restore-statefulset.yaml
service/re-headless created
statefulset.apps/restore-demo created
```

Now, wait for the pod of the StatefulSet to go into the `Running` state.

```console
$ kubectl get pod -n demo
NAME             READY   STATUS    RESTARTS   AGE
restore-demo-0   1/1     Running   0          34s
restore-demo-1   1/1     Running   0          31s
restore-demo-2   1/1     Running   0          26s
```

Verify that the backed up data has been restored in `/restore/data` and `/restore/config` directory using the following command,

```console
$ kubectl exec -n demo restore-demo-0 -- cat /restore/data/data.txt
stash-demo-0
$ kubectl exec -n demo restore-demo-0 -- cat /restore/config/config.cfg
stash-demo-0
$ kubectl exec -n demo restore-demo-1 -- cat /restore/data/data.txt
stash-demo-1
$ kubectl exec -n demo restore-demo-1 -- cat /restore/config/config.cfg
stash-demo-1
$ kubectl exec -n demo restore-demo-2 -- cat /restore/data/data.txt
stash-demo-2
$ kubectl exec -n demo restore-demo-2 -- cat /restore/config/config.cfg
stash-demo-2
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
