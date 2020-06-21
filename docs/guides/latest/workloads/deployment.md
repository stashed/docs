---
title: Backup and Restore Volumes of a Deployment | Stash
description: A step by step guide showing how to backup and restore volumes of a Deployment.
menu:
  docs_{{ .version }}:
    identifier: workload-deployment
    name: Backup & Restore Volumes of a Deployment
    parent: workload
    weight: 20
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: guides
---

# Backup and Restore Volumes of a Deployment

This guide will show you how to use Stash to backup and restore volumes of a Deployment.

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [kind](https://kind.sigs.k8s.io/docs/user/quick-start/).

- Install `Stash` in your cluster following the steps [here](/docs/setup/install.md).

- You should be familiar with the following `Stash` concepts:
  - [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md)
  - [BackupSession](/docs/concepts/crds/backupsession.md)
  - [RestoreSession](/docs/concepts/crds/restoresession.md)
  - [Repository](/docs/concepts/crds/repository.md)

To keep everything isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$ kubectl create ns demo
namespace/demo created
```

> **Note:** YAML files used in this tutorial are stored in [docs/examples/guides/latest/workloads](/docs/examples/guides/latest/workloads) directory of [stashed/docs](https://github.com/stashed/docs) repository.

## Backup Volumes of a Deployment

This section will show you how to use Stash to backup volumes of a Deployment. Here, we are going to deploy a Deployment with a PVC and generate some sample data in it. Then, we are going to backup this sample data using Stash.

### Prepare Workload

At first, we are going to create a PVC then we are going to create a Deployment that will use this PVC.

**Create PVC:**

Below is the YAML of the sample PVC that we are going to create,

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
```

Let's create the PVC we have shown above,

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/workloads/deployment/pvc.yaml
persistentvolumeclaim/stash-sample-data created
```

**Deploy Deployment:**

Now, we are going to deploy a Deployment that uses the above PVC. This Deployment will automatically generate sample data (`data.txt` file) in `/source/data` directory where we have mounted the PVC.

Below is the YAML of the Deployment that we are going to create,

```yaml
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

Let's create the Deployment we have shown above.

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/workloads/deployment/deployment.yaml
deployment.apps/stash-demo created
```

Now, wait for the pods of the Deployment to go into the `Running` state.

```console
$ kubectl get pod -n demo
NAME                         READY   STATUS    RESTARTS   AGE
stash-demo-8cfcbcc89-2z6mq   1/1     Running   0          30s
stash-demo-8cfcbcc89-j9wbc   1/1     Running   0          30s
stash-demo-8cfcbcc89-q8xfd   1/1     Running   0          30s
```

Verify that the sample data has been created in `/source/data` directory using the following command,

```console
$ kubectl exec -n demo stash-demo-8cfcbcc89-2z6mq -- cat /source/data/data.txt
sample_data
```

### Prepare Backend

We are going to store our backed up data into a GCS bucket. We have to create a Secret with necessary credentials and a Repository crd to use this backend. If you want to use a different backend, please read the respective backend configuration doc from [here](/docs/guides/latest/backends/overview.md).

> For GCS backend, if the bucket does not exist, Stash needs `Storage Object Admin` role permissions to create the bucket. For more details, please check the following [guide](/docs/guides/latest/backends/gcs.md).

**Create Secret:**

Let's create a secret called `gcs-secret` with access credentials to our desired GCS bucket,

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

**Create Repository:**

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
      prefix: /source/data/sample-deployment
    storageSecretName: gcs-secret
```

Let's create the Repository we have shown above,

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/workloads/deployment/repository.yaml
repository.stash.appscode.com/gcs-repo created
```

Now, we are ready to backup our sample data into this backend.

### Backup

We have to create a `BackupConfiguration` crd targeting the `stash-demo` Deployment that we have deployed earlier. Stash will inject a sidecar container into the target. It will also create a `CronJob` to take periodic backup of `/source/data` directory of the target.

**Create BackupConfiguration:**

Below is the YAML of the `BackupConfiguration` crd that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: deployment-backup
  namespace: demo
spec:
  repository:
    name: gcs-repo
  schedule: "*/5 * * * *"
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

Here,

- `spec.repository` refers to the `Repository` object `gcs-repo` that holds backend information.
- `spec.schedule` is a cron expression that indicates `BackupSession` will be created at 5 minute interval.
- `spec.target.ref` refers to the `stash-demo` Deployment.
- `spec.target.volumeMounts` specifies a list of volumes and their mountPath that contain the target paths.
- `spec.target.paths` specifies list of file paths to backup.

Let's create the `BackupConfiguration` crd we have shown above,

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/workloads/deployment/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/deployment-backup created
```

**Verify Sidecar:**

If everything goes well, Stash will inject a sidecar container into the `stash-demo` Deployment to take backup of `/source/data` directory. Let’s check that the sidecar has been injected successfully,

```console
$ kubectl get pod -n demo
NAME                          READY   STATUS        RESTARTS   AGE
stash-demo-856896bd95-4gfbh   2/2     Running       0          12s
stash-demo-856896bd95-njr8x   2/2     Running       0          17s
stash-demo-856896bd95-ttbq4   2/2     Running       0          15s
```

Look at the pod. It now has 2 containers. If you view the resource definition of this pod, you will see that there is a container named `stash` which is running `run-backup` command.

```yaml
$ kubectl get pod -n demo stash-demo-856896bd95-4gfbh -o yaml
apiVersion: v1
kind: Pod
metadata:
  name: stash-demo-856896bd95-4gfbh
  namespace: demo
 ...
spec:
  containers:
  - args:
    - echo sample_data > /source/data/data.txt && sleep 3000
    command:
    - /bin/sh
    - -c
    image: busybox
    imagePullPolicy: IfNotPresent
    name: busybox
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /source/data
      name: source-data
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-4tzgg
      readOnly: true
  - args:
    - run-backup
    - --backup-configuration=deployment-backup
    - --secret-dir=/etc/stash/repository/secret
    - --enable-cache=true
    - --max-connections=0
    - --metrics-enabled=true
    - --pushgateway-url=http://stash-operator.kube-system.svc:56789
    - --enable-status-subresource=true
    - --use-kubeapiserver-fqdn-for-aks=true
    - --enable-analytics=true
    - --logtostderr=true
    - --alsologtostderr=false
    - --v=3
    - --stderrthreshold=0
    env:
    - name: NODE_NAME
      valueFrom:
        fieldRef:
          apiVersion: v1
          fieldPath: spec.nodeName
    - name: POD_NAME
      valueFrom:
        fieldRef:
          apiVersion: v1
          fieldPath: metadata.name
    volumeMounts:
    - mountPath: /etc/stash
      name: stash-podinfo
    - mountPath: /etc/stash/repository/secret
      name: stash-secret-volume
    - mountPath: /tmp
      name: tmp-dir
    - mountPath: /source/data
      name: source-data
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-4tzgg
      readOnly: true
  volumes:
  - name: source-data
    persistentVolumeClaim:
      claimName: stash-sample-data
  - emptyDir: {}
    name: tmp-dir
  - downwardAPI:
      defaultMode: 420
      items:
      - fieldRef:
          apiVersion: v1
          fieldPath: metadata.labels
        path: labels
    name: stash-podinfo
  - name: stash-secret-volume
    secret:
      defaultMode: 420
      secretName: gcs-secret
  - name: default-token-4tzgg
    secret:
      defaultMode: 420
      secretName: default-token-4tzgg
  ...
...
```

**Verify CronJob:**

It will also create a `CronJob` with the schedule specified in `spec.schedule` field of `BackupConfiguration` crd.

Verify that the `CronJob` has been created using the following command,

```console
$ kubectl get cronjob -n demo
NAME                SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
deployment-backup   */1 * * * *   False     0        35s             64s
```

**Wait for BackupSession:**

The `deployment-backup` CronJob will trigger a backup on each scheduled slot by creating a `BackupSession` crd. The sidecar container watches for the `BackupSession` crd. When it finds one, it will take backup immediately.

Wait for the next schedule for backup. Run the following command to watch `BackupSession` crd,

```console
$ watch -n 2 kubectl get backupsession -n demo
Every 1.0s: kubectl get backupsession -n demo     suaas-appscode: Mon Jun 24 10:23:08 2019

NAME                           INVOKER-TYPE          INVOKER-NAME        PHASE       AGE
deployment-backup-1561350125   BackupConfiguration   deployment-backup   Succeeded   63s
```

We can see from the above output that the backup session has succeeded. Now, we are going to verify whether the backed up data has been stored in the backend.

**Verify Backup:**

Once a backup is complete, Stash will update the respective `Repository` crd to reflect the backup. Check that the repository `gcs-repo` has been updated by the following command,

```console
$ kubectl get repository -n demo gcs-repo
NAME       INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
gcs-repo   true        0 B    5                58s                      18m
```

Now, if we navigate to the GCS bucket, we are going to see backed up data has been stored in `source/data/sample-deployment` directory as specified by `spec.backend.gcs.prefix` field of Repository crd.

<figure align="center">
  <img alt="Backup data in GCS Bucket" src="/docs/images/guides/latest/workloads/gcs_bucket_dep.png">
  <figcaption align="center">Fig: Backup data in GCS Bucket</figcaption>
</figure>

> **Note:** Stash keeps all the backed up data encrypted. So, data in the backend will not make any sense until they are decrypted.

## Restore the Backed up Data

This section will show you how to restore the backed up data from the backend we have taken in the earlier section.

**Stop Taking Backup of the Old Deployment:**

At first, let's stop taking any further backup of the old Deployment so that no backup is taken during the restore process. We are going to pause the `BackupConfiguration` that we created to backup the `stash-demo` Deployment. Then, Stash will stop taking any further backup for this Deployment. You can learn more how to pause a scheduled backup [here](/docs/guides/latest/advanced-use-case/pause-backup.md)

Let's pause the `deployment-backup` BackupConfiguration,

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

**Deploy Deployment:**

We are going to create a new Deployment named `stash-recovered` and restore the backed up data inside it.

Below are the YAMLs of the Deployment and PVC that we are going to create,

```yaml
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
    app: stash-recovered
  name: stash-recovered
  namespace: demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: stash-recovered
  template:
    metadata:
      labels:
        app: stash-recovered
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

Let's create the Deployment and PVC we have shown above.

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/workloads/deployment/recovered_deployment.yaml
persistentvolumeclaim/demo-pvc created
deployment.apps/stash-recovered created
```

**Create RestoreSession:**

Now, we need to create a `RestoreSession` crd targeting the `stash-recovered` Deployment to restore the backed up data inside it.

Below is the YAML of the `RestoreSesion` crd that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: deployment-restore
  namespace: demo
spec:
  repository:
    name: gcs-repo
  rules:
  - paths:
    - /source/data/
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

- `spec.repository.name` specifies the `Repository` crd that holds the backend information where our backed up data has been stored.
- `spec.target.ref` refers to the target workload where the recovered data will be stored.
- `spec.target.volumeMounts`  specifies a list of volumes and their mountPath where the data will be restored.
  - `mountPath` must be same `mountPath` as the original volume because Stash stores absolute path of the backed up files. If you use different `mountPath` for the restored volume the backed up files will not be restored into your desired volume.

Let's create the `RestoreSession` crd we have shown above,

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/workloads/deployment/restoresession.yaml
restoresession.stash.appscode.com/deployment-restore created
```

Once, you have created the `RestoreSession` crd, Stash will inject `init-container` into `stash-recovered` Deployment. Deployment will restart and the `init-container` will restore the desired data on start-up.

**Verify Init-Container:**

Wait until the `init-container` has been injected into the `stash-recovered` Deployment. Let’s describe the Deployment to verify that `init-container` has been injected successfully.

```yaml
$ kubectl describe deployment -n demo stash-recovered
Name:                   stash-recovered
Namespace:              demo
Labels:                 app=stash-recovered
Selector:               app=stash-recovered
Replicas:               3 desired | 3 updated | 3 total | 3 available |
...
Pod Template:
  Labels:       app=stash-recovered
  Annotations:  stash.appscode.com/last-applied-restoresession-hash: 14443247646000846167
  Init Containers:
   stash-init:
    Image:      suaas21/stash:vs_linux_amd64
    Port:       <none>
    Host Port:  <none>
    Args:
      restore
      --restore-session=deployment-restore
      --secret-dir=/etc/stash/repository/secret
      --enable-cache=true
      --max-connections=0
      --metrics-enabled=true
      --pushgateway-url=http://stash-operator.kube-system.svc:56789
      --enable-status-subresource=true
      --use-kubeapiserver-fqdn-for-aks=true
      --enable-analytics=true
      --logtostderr=true
      --alsologtostderr=false
      --v=3
      --stderrthreshold=0
    Environment:
      NODE_NAME:   (v1:spec.nodeName)
      POD_NAME:    (v1:metadata.name)
    Mounts:
      /etc/stash/repository/secret from stash-secret-volume (rw)
      /source/data from source-data (rw)
      /tmp from tmp-dir (rw)
  Containers:
   busybox:
    Image:      busybox
    Port:       <none>
    Host Port:  <none>
    Args:
      sleep
      3600
    Environment:  <none>
    Mounts:
      /source/data from source-data (rw)
  Volumes:
   source-data:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  demo-pvc
    ReadOnly:   false
   tmp-dir:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:  <unset>
   stash-podinfo:
    Type:  DownwardAPI (a volume populated by information about the pod)
    Items:
      metadata.labels -> labels
   stash-secret-volume:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  gcs-secret
    Optional:    false
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  stash-recovered-7478988f57 (3/3 replicas created)
NewReplicaSet:   <none>
...
```

Notice the `Init-Containers` section. We can see that the init-container `stash-init` has been injected which is running `restore` command.

**Wait for RestoreSession to Succeeded:**

Run the following command to watch RestoreSession phase,

```console
$ watch -n 2 kubectl get restoresession -n demo
Every 5.0s: kubectl get restoresession -n demo           suaas-appscode: Mon Jun 24 10:33:57 2019

NAME                 REPOSITORY-NAME   PHASE       AGE
deployment-restore   gcs-repo          Succeeded   2m56s
```

So, we can see from the output of the above command that the restore process succeeded.

> **Note:** If you want to restore the backed up data inside the same Deployment whose volumes were backed up, you have to remove the corrupted data from the Deployment. Then, you have to create a RestoreSession targeting the Deployment.

**Verify Restored Data:**

In this section, we are going to verify that the desired data has been restored successfully.

At first, check if the `stash-recovered` pods of the Deployment has gone into `Running` state by the following command,

```console
$ kubectl get pod -n demo
NAME                               READY   STATUS    RESTARTS   AGE
stash-recovered-867688ddd5-67xr8   1/1     Running   0          21m
stash-recovered-867688ddd5-rfsw4   1/1     Running   0          21m
stash-recovered-867688ddd5-zswhs   1/1     Running   0          22m
```

Verify that the sample data has been restored in `/source/data` directory of the `stash-recovered` pods of the Deployment using the following command,

```console
$ kubectl exec -n demo stash-recovered-867688ddd5-67xr8 -- cat /source/data/data.txt
sample_data
```

## Cleaning Up

To clean up the Kubernetes resources created by this tutorial, run:

```console
kubectl delete -n demo deployment stash-demo
kubectl delete -n demo deployment stash-recovered
kubectl delete -n demo backupconfiguration deployment-backup
kubectl delete -n demo restoresession deployment-restore
kubectl delete -n demo repository gcs-repo
kubectl delete -n demo pvc --all
```
