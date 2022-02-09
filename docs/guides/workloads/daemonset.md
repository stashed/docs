---
title: Backup and Restore Volumes of a DaemonSet | Stash
description: A step by step guide showing how to backup and restore volumes of a DaemonSet.
menu:
  docs_{{ .version }}:
    identifier: workload-daemonset
    name: Backup & Restore Volumes of a DaemonSet
    parent: workload
    weight: 40
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: guides
---

# Backup and Restore Volumes of a DaemonSet

This guide will show you how to use Stash to backup and restore volumes of a DaemonSet.

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [kind](https://kind.sigs.k8s.io/docs/user/quick-start/).

- Install `Stash` in your cluster following the steps [here](/docs/setup/README.md).

- You should be familiar with the following `Stash` concepts:
  - [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md)
  - [BackupSession](/docs/concepts/crds/backupsession.md)
  - [RestoreSession](/docs/concepts/crds/restoresession.md)
  - [Repository](/docs/concepts/crds/repository.md)

To keep everything isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```bash
$ kubectl create ns demo
namespace/demo created
```

> **Note:** YAML files used in this tutorial are stored in [docs/examples/guides/workloads](/docs/examples/guides/workloads) directory of [stashed/docs](https://github.com/stashed/docs) repository.

## Backup Volumes of a DaemonSet

This section will show you how to use Stash to backup volumes of a DaemonSet. Here, we are going to deploy a DaemonSet and generate some sample data in it. Then, we are going to backup this sample data using Stash.

**Deploy DaemonSet:**

At first, we are going to deploy a DaemonSet. This DaemonSet will automatically generate sample data (`data.txt` file) in `/source/data` directory.

Below is the YAML of the DaemonSet that we are going to create,

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: stash-demo
  name: stash-demo
  namespace: demo
spec:
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
        hostPath:
          path: /stash/source/data
```

Let's create the DaemonSet we have shown above.

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/workloads/daemonset/daemon.yaml
daemonset.apps/stash-demo created
```

Now, wait for the pod of the DaemonSet to go into the `Running` state.

```bash
$ kubectl get pod -n demo
NAME                READY   STATUS    RESTARTS   AGE
stash-demo-c4nqw    1/1     Running   0          39s
```

Verify that the sample data has been created in `/source/data` directory using the following command,

```bash
$ kubectl exec -n demo stash-demo-c4nqw -- cat /source/data/data.txt
sample_data
```

### Prepare Backend

We are going to store our backed up data into a GCS bucket. We have to create a Secret with necessary credentials and a Repository crd to use this backend. If you want to use a different backend, please read the respective backend configuration doc from [here](/docs/guides/backends/overview.md).

> For GCS backend, if the bucket does not exist, Stash needs `Storage Object Admin` role permissions to create the bucket. For more details, please check the following [guide](/docs/guides/backends/gcs.md).

**Create Secret:**

Let's create a secret called `gcs-secret` with access credentials to our desired GCS bucket,

```bash
$ echo -n 'changeit' > RESTIC_PASSWORD
$ echo -n '<your-project-id>' > GOOGLE_PROJECT_ID
$ cat /path/to/downloaded-sa-key.json > GOOGLE_SERVICE_ACCOUNT_JSON_KEY
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
      prefix: /source/data/sample-daemonset
    storageSecretName: gcs-secret
```

Let's create the Repository we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/workloads/daemonset/repository.yaml
repository.stash.appscode.com/gcs-repo created
```

Now, we are ready to backup our sample data into this backend.

### Backup

We have to create a `BackupConfiguration` crd targeting the `stash-demo` DaemonSet that we have deployed earlier. Stash will inject a sidecar container into the target. It will also create a `CronJob` to take periodic backup of `/source/data` directory of the target.

**Create BackupConfiguration:**

Below is the YAML of the `BackupConfiguration` crd that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: dmn-backup
  namespace: demo
spec:
  repository:
    name: gcs-repo
  schedule: "*/5 * * * *"
  target:
    ref:
      apiVersion: apps/v1
      kind: DaemonSet
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
- `spec.target.ref` refers to the `stash-demo` DaemonSet.
- `spec.target.volumeMounts` specifies a list of volumes and their mountPath that contain the target paths.
- `spec.target.paths` specifies list of file paths to backup.

Let's create the `BackupConfiguration` crd we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/workloads/daemonset/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/dmn-backup created
```

**Verify Backup Setup Successful**

If everything goes well, the Phase of the `BackupConfiguration` should be `Ready`. The `Ready` Phase indicates that the backup setup is successful. Let's verify the `Phase` of the BackupConfiguration,

```bash
$ kubectl get backupconfiguration -n demo
NAME                TASK    SCHEDULE      PAUSED   PHASE      AGE
dmn-backup                  */5 * * * *            Ready      11s
```
> If the BackupConfiguration is not in `Ready` state, you need to describe that CRD for finding out the specific reason of the backup setup being unsuccessful.  Describe the BackupConfiguration by following command,
```bash
$ kubectl describe backupconfiguration -n demo dmn-backup
```

**Verify Sidecar:**

Stash will inject a sidecar container into the `stash-demo` DaemonSet to take backup of `/source/data` directory. Let’s check that the sidecar has been injected successfully,

```bash
$ kubectl get pod -n demo
NAME                READY   STATUS    RESTARTS   AGE
stash-demo-6lnbp    2/2     Running   0          10s
```

Look at the pod. It now has 2 containers. If you view the resource definition of this pod, you will see that there is a container named `stash` which is running `run-backup` command.

```yaml
$ kubectl get pod -n demo stash-demo-6lnbp -o yaml
apiVersion: v1
kind: Pod
metadata:
  name: stash-demo-6lnbp
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
    - --backup-configuration=dmn-backup
    - --secret-dir=/etc/stash/repository/secret
    - --enable-cache=true
    - --max-connections=0
    - --metrics-enabled=true
    - --pushgateway-url=http://stash-operator.kube-system.svc:56789
    - --enable-status-subresource=true
    - --use-kubeapiserver-fqdn-for-aks=true
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
    name: stash
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

```bash
$ kubectl get backupconfiguration -n  demo
NAME         TASK   SCHEDULE      PAUSED   AGE
dmn-backup          */5 * * * *            3m
```

**Wait for BackupSession:**

The `dmn-backup` CronJob will trigger a backup on each schedule by creating a `BackupSession` crd. The sidecar container watches for the `BackupSession` crd. When it finds one, it will take backup immediately.

Wait for the next schedule for backup. Run the following command to watch `BackupSession` crd,

```bash
$ watch -n 3 kubectl get backupsession -n demo
Every 3.0s: kubectl get backupsession -n demo                suaas-appscode: Wed Jun 26 16:05:26 2019

NAME                    INVOKER-TYPE          INVOKER-NAME   PHASE       AGE
dmn-backup-1561543509   BackupConfiguration   dmn-backup     Succeeded   2m20s
```

We can see from the above output that the backup session has succeeded. Now, we are going to verify whether the backed up data has been stored in the backend.

**Verify Backup:**

Once a backup is complete, Stash will update the respective `Repository` crd to reflect the backup. Check that the repository `gcs-repo` has been updated by the following command,

```bash
$ kubectl get repository -n demo
NAME       INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
gcs-repo   true        0 B    3                47s                      4m
```

Now, if we navigate to the GCS bucket, we are going to see backed up data has been stored in `source/data/sample-daemonset` directory as specified by `spec.backend.gcs.prefix` field of Repository crd.

<figure align="center">
  <img alt="Backup data in GCS Bucket" src="/docs/images/guides/workloads/gcs_bucket_dmn.png">
  <figcaption align="center">Fig: Backup data in GCS Bucket</figcaption>
</figure>

> **Note:** Stash keeps all the backed up data encrypted. So, data in the backend will not make any sense until they are decrypted.

## Restore the Backed up Data

This section will show you how to restore the backed up data from the backend we have taken in the earlier section.

**Stop Taking Backup of the Old DaemonSet:**

At first, let's stop taking any further backup of the old DaemonSet so that no backup is taken during the restore process. We are going to pause the `BackupConfiguration` that we created to backup the `stash-demo` DaemonSet. Then, Stash will stop taking any further backup for this DaemonSet. You can learn more how to pause a scheduled backup [here](/docs/guides/advanced-use-case/pause-backup.md)

Let's pause the `dmn-backup` BackupConfiguration,

```bash
$ kubectl patch backupconfiguration -n demo dmn-backup --type="merge" --patch='{"spec": {"paused": true}}'
backupconfiguration.stash.appscode.com/dmn-backup patched
```

Now, wait for a moment. Stash will pause the BackupConfiguration. Verify that the BackupConfiguration  has been paused,

```bash
$ kubectl get backupconfiguration -n demo
NAME                TASK   SCHEDULE      PAUSED   PHASE   AGE
dmn-backup                 */5 * * * *   true     Ready   26m
```

Notice the `PAUSED` column. Value `true` for this field means that the BackupConfiguration has been paused.

**Deploy DaemonSet:**

We are going to create a new DaemonSet named `stash-recovered` and restore the backed up data inside it.

Below is the YAML of the DaemonSet that we are going to create,

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: stash-recovered
  name: stash-recovered
  namespace: demo
spec:
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
        hostPath:
          path: /stash/recovered/data
```

Let's create the DaemonSet we have shown above.

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/workloads/daemonset/recovered_daemon.yaml
daemonset.apps/stash-recovered configured
```

**Create RestoreSession:**

Now, we need to create a `RestoreSession` crd targeting the `stash-recovered` DaemonSet to restore the backed up data inside it.

Below is the YAML of the `RestoreSesion` crd that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: dmn-restore
  namespace: demo
spec:
  repository:
    name: gcs-repo
  target: # target indicates where the recovered data will be stored
    ref:
      apiVersion: apps/v1
      kind: DaemonSet
      name: stash-recovered
    volumeMounts:
    - name:  source-data
      mountPath:  /source/data
    rules:
    - paths:
      - /source/data
```

Here,

- `spec.repository.name` specifies the `Repository` crd that holds the backend information where our backed up data has been stored.
- `spec.target.ref` refers to the target workload where the recovered data will be stored.
- `spec.target.volumeMounts` specifies a list of volumes and their mountPath where the data will be restored.
  - `mountPath` must be same `mountPath` as the original volume because Stash stores absolute path of the backed up files. If you use different `mountPath` for the restored volume the backed up files will not be restored into your desired volume.

Let's create the `RestoreSession` crd we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/workloads/daemonset/restoresession.yaml
restoresession.stash.appscode.com/dmn-restore created
```

Once, you have created the `RestoreSession` crd, Stash will inject `init-container` into `stash-recovered` DaemonSet. The pods of this DaemonSet will restart and the `init-container` will restore the desired data on start-up.

**Verify Init-Container:**

Wait until the `init-container` has been injected into the `stash-recovered` DaemonSet. Let's describe the DaemonSet to verify that the `init-container` has been injected successfully.

```yaml
 $ kubectl describe daemonset -n demo stash-recovered
Name:           stash-recovered
Selector:       app=stash-recovered
Node-Selector:  <none>
Labels:         app=stash-recovered
Pods Status:  1 Running / 0 Waiting / 0 Succeeded / 0 Failed
...
Pod Template:
  Labels:       app=stash-recovered
  Annotations:  stash.appscode.com/last-applied-restoresession-hash: 4703201294184533055
  Init Containers:
   stash-init:
    Image:      suaas21/stash:vs_linux_amd64
    Port:       <none>
    Host Port:  <none>
    Args:
      restore
      --restore-session=dmn-restore
      --secret-dir=/etc/stash/repository/secret
      --enable-cache=true
      --max-connections=0
      --metrics-enabled=true
      --pushgateway-url=http://stash-operator.kube-system.svc:56789
      --enable-status-subresource=true
      --use-kubeapiserver-fqdn-for-aks=true
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
    Type:          HostPath (bare host directory volume)
    Path:          /stash/recovered/data
    HostPathType:
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
...
```

Notice the `Init-Containers` section. We can see that the init-container `stash-init` has been injected which is running `restore` command.

**Wait for RestoreSession to Succeeded:**

Run the following command to watch RestoreSession phase,

```bash
$ watch -n 3 kubectl get restoresession -n demo
Every 3.0s: kubectl get restoresession -n demo               suaas-appscode: Wed Jun 26 14:28:29 2019

NAME          REPOSITORY-NAME   PHASE       AGE
dmn-restore   gcs-repo          Succeeded   3m29s
```

So, we can see from the output of the above command that the restore process succeeded.

> **Note:** If you want to restore the backed up data inside the same DaemonSet whose volumes were backed up, you have to remove the corrupted data from the DaemonSet. Then, you have to create a RestoreSession targeting the DaemonSet.

**Verify Restored Data:**

In this section, we are going to verify that the desired data has been restored successfully.

At first, check if the `stash-recovered` pods of a DaemonSet has gone into `running` state by the following command,

```bash
$ kubectl get pod -n demo
NAME                    READY   STATUS    RESTARTS   AGE
stash-recovered-dqlrb   1/1     Running   0          4m4s
```

Verify that the backed up data has been restored in `/source/data` directory of the `stash-recovered` pods of a DaemonSet using the following command,

```bash
$ kubectl exec -n demo stash-recovered-dqlrb -- cat /source/data/data.txt
sample_data
```

## Cleaning Up

To clean up the Kubernetes resources created by this tutorial, run:

```bash
kubectl delete -n demo daemonset stash-demo
kubectl delete -n demo daemonset stash-recovered
kubectl delete -n demo backupconfiguration dmn-backup
kubectl delete -n demo restoresession dmn-restore
kubectl delete -n demo repository gcs-repo
kubectl delete -n demo pvc --all
```
