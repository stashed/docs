---
title: Backup and Restore Volumes of a StatefulSet | Stash
description: A step by step guide showing how to backup and restore volumes of a StatefulSet.
menu:
  docs_{{ .version }}:
    identifier: workload-statefulset
    name: Backup & Restore Volumes of a StatefulSet
    parent: workload
    weight: 30
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: guides
---

# Backup and Restore Volumes of a StatefulSet

This guide will show you how to use Stash to backup and restore volumes of a StatefulSet.

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

## Backup Volumes of a StatefulSet

This section will show you how to use Stash to backup volumes of a StatefulSet. Here, we are going to deploy a StatefulSet with a PVC and generate some sample data in it. Then, we are going to backup this sample data using Stash.

**Deploy StatefulSet:**

At first, We are going to deploy a StatefulSet. This StatefulSet will automatically generate sample data in `/source/data` directory.

Below is the YAML of the StatefulSet that we are going to create,

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

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/workloads/statefulset/statefulset.yaml
service/headless created
statefulset.apps/stash-demo created
```

Now, wait for the pods of the StatefulSet to go into the `Running` state.

```console
$ kubectl get pod -n demo
NAME           READY   STATUS    RESTARTS   AGE
stash-demo-0   1/1     Running   0          42s
stash-demo-1   1/1     Running   0          40s
stash-demo-2   1/1     Running   0          36s
```

Verify that the sample data has been generated in `/source/data` directory for `stash-demo-0` , `stash-demo-1` and `stash-demo-2` pod respectively using the following commands,

```console
$ kubectl exec -n demo stash-demo-0 -- cat /source/data/data.txt
stash-demo-0
$ kubectl exec -n demo stash-demo-1 -- cat /source/data/data.txt
stash-demo-1
$ kubectl exec -n demo stash-demo-2 -- cat /source/data/data.txt
stash-demo-2
```

### Prepare Backend

We are going to store our backed up data into a GCS bucket. We have to create a Secret with necessary credentials and a Repository crd to use this backend. If you want to use a different backend, please read the respective backend configuration doc from [here](/docs/guides/latest/backends/overview.md).

> For GCS backend, if the bucket does not exist, Stash needs `Storage Object Admin` role permissions to create the bucket. For more details, please check the following [guide](/docs/guides/latest/backends/gcs.md).

**Create Secret:**

Let's create a secret called `gcs-secret` with access credentials to our desired GCS bucket,

```console
$ echo -n 'changeit' > RESTIC_PASSWORD
$ echo -n '<your-project-id>' > GOOGLE_PROJECT_ID
$ cat /path/to/downloaded-sa-json.key > GOOGLE_SERVICE_ACCOUNT_JSON_KEY
$ kubectl create secret generic -n demo gcs-secret \
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
  namespace: demo
spec:
  backend:
    gcs:
      bucket: appscode-qa
      prefix: /source/data/sample-statefulset
    storageSecretName: gcs-secret
```

Let's create the Repository we have shown above,

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/workloads/statefulset/repository.yaml
repository.stash.appscode.com/gcs-repo created
```

Now, we are ready to backup our sample data into this backend.

### Backup

We have to create a `BackupConfiguration` crd targeting the `stash-demo` StatefulSet that we have deployed earlier. Stash will inject a sidecar container into the target. It will also create a `CronJob` to take periodic backup of `/source/data` directory of the target.

**Create BackupConfiguration:**

Below is the YAML of the `BackupConfiguration` crd that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: ss-backup
  namespace: demo
spec:
  repository:
    name: gcs-repo
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

- `spec.repository` refers to the `Repository` object `gcs-repo` that holds backend information.
- `spec.schedule` is a cron expression that indicates `BackupSession` will be created at 5 minute interval.
- `spec.target.ref` refers to the `stash-demo` StatefulSet.
- `spec.target.volumeMounts` specifies a list of volumes and their mountPath that contain the target paths.
- `spec.target.paths` specifies list of file paths to backup.

Let's create the `BackupConfiguration` crd we have shown above,

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/workloads/statefulset/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/ss-backup created
```

**Verify Sidecar:**

If everything goes well, Stash will inject a sidecar container into the `stash-demo` StatefulSet to take backup of `/source/data` directory. Let’s check that the sidecar has been injected successfully,

```console
$ kubectl get pod -n demo
NAME           READY   STATUS    RESTARTS   AGE
stash-demo-0   2/2     Running   0          5s
stash-demo-1   2/2     Running   0          42s
stash-demo-2   2/2     Running   0          76s
```

Look at the pod. It now has 2 containers. If you view the resource definition of this pod, you will see that there is a container named `stash` which is running `run-backup` command.

```yaml
$ kubectl get pod -n demo stash-demo-0 -o yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: stash-demo
    controller-revision-hash: stash-demo-6d887c7b6f
    statefulset.kubernetes.io/pod-name: stash-demo-0
  name: stash-demo-0
  namespace: demo
  ...
spec:
  containers:
  - command:
    - /bin/sh
    - -c
    - echo $(POD_NAME) > /source/data/data.txt && sleep 3000
    env:
    - name: POD_NAME
      valueFrom:
        fieldRef:
          apiVersion: v1
          fieldPath: metadata.name
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
    - --backup-configuration=ss-backup
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
    imagePullPolicy: IfNotPresent
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
  hostname: stash-demo-0
  volumes:
  - name: source-data
    persistentVolumeClaim:
      claimName: source-data-stash-demo-0
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
$ kubectl get backupconfiguration -n  demo
NAME        TASK   SCHEDULE      PAUSED   AGE
ss-backup          */1 * * * *            3m41s
```

**Wait for BackupSession:**

The `ss-backup` CronJob will trigger a backup on each scheduled slot by creating a `BackupSession` crd. The sidecar container watches for the `BackupSession` crd. When it finds one, it will take backup immediately.

Wait for the next schedule for backup. Run the following command to watch `BackupSession` crd,

```console
$ watch -n 2 kubectl get backupsession -n demo
Every 5.0s: kubectl get bs -n demo                               suaas-appscode: Tue Jun 25 17:54:41 2019

NAME                   INVOKER-TYPE          INVOKER-NAME    PHASE       AGE
ss-backup-1561463408   BackupConfiguration   ss-backup       Succeeded   36s
```

We can see from the above output that the backup session has succeeded. Now, we are going to verify whether the backed up data has been stored in the backend.

**Verify Backup:**

Once a backup is complete, Stash will update the respective `Repository` crd to reflect the backup. Check that the repository `gcs-repo` has been updated by the following command,

```console
$ kubectl get repository -n demo
NAME       INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
gcs-repo   true        0 B    3                103s                     5m
```

Now, if we navigate to the GCS bucket, we are going to see backed up data has been stored in `source/data/sample-statefulset` directory as specified by `spec.backend.gcs.prefix` field of Repository crd.

<figure align="center">
  <img alt="Backup data in GCS Bucket" src="/docs/images/guides/latest/workloads/gcs_bucket_ss.png">
  <figcaption align="center">Fig: Backup data in GCS Bucket</figcaption>
</figure>

> **Note:** Stash keeps all the backed up data encrypted. So, data in the backend will not make any sense until they are decrypted.

## Restore Volumes of a StatefulSet

This section will show you how to restore the backed up data from the backend we have taken in the earlier section.

**Stop Taking Backup of the Old StatefulSet:**

At first, let's stop taking any further backup of the old StatefulSet so that no backup is taken during the restore process. We are going to pause the `BackupConfiguration` that we created to backup the `stash-demo` StatefulSet. Then, Stash will stop taking any further backup for this StatefulSet. You can learn more how to pause a scheduled backup [here](/docs/guides/latest/advanced-use-case/pause-backup.md)

Let's pause the `ss-backup` BackupConfiguration,

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

Notice the `PAUSED` column. Value `true` for this field means that the BackupConfiguration has been paused.

**Deploy StatefulSet:**

We are going to create a new StatefulSet named `stash-recovered` and restore the backed up data inside it.

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
    app: stash-recovered
  clusterIP: None
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: stash-recovered
  namespace: demo
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

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/workloads/statefulset/recovered_statefulset.yaml
service/re-headless created
statefulset.apps/stash-recovered created
```

**Create RestoreSession:**

Now, we need to create a `RestoreSession` crd targeting the `stash-recovered` StatefulSet to restore the backed up data inside it.

Below is the YAML of the `RestoreSesion` crd that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: ss-restore
  namespace: demo
spec:
  repository:
    name: gcs-repo
  rules:
  - paths:
    - /source/data
  target:
    ref:
      apiVersion: apps/v1
      kind: StatefulSet
      name: stash-recovered
    volumeMounts:
    - name:  source-data
      mountPath:  /source/data
```

Here,

- `spec.repository.name` specifies the `Repository` crd that holds the backend information where our backed up data has been stored.
- `spec.target.ref` refers to the target workload where the recovered data will be stored.
- `spec.target.volumeMounts` specifies a list of volumes and their mountPath where the data will be restored.
  - `mountPath` must be same `mountPath` as the original volume because Stash stores absolute path of the backed up files. If you use different `mountPath` for the restored volume the backed up files will not be restored into your desired volume.

Let's create the `RestoreSession` crd we have shown above,

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/workloads/statefulset/restoresession.yaml
restoresession.stash.appscode.com/ss-restore created
```

Once, you have created the `RestoreSession` crd, Stash will inject `init-container` into `stash-recovered` StatefulSet. The StatefulSet will restart and the `init-container` will restore the desired data on start-up.

**Verify Init-Container:**

Wait until the `init-container` has been injected into the `stash-recovered` StatefulSet. Let’s describe the StatefulSet to verify that the `init-container` has been injected successfully.

```yaml
$ kubectl describe statefulset -n demo stash-recovered
Name:               stash-recovered
Namespace:          demo
Selector:           app=stash-recovered
Labels:             app=stash-recovered
Replicas:           3 desired | 3 total
Pods Status:        3 Running / 0 Waiting / 0 Succeeded / 0 Failed
...
Pod Template:
  Labels:       app=stash-recovered
  Annotations:  stash.appscode.com/last-applied-restoresession-hash: 10309464337907785627
  Init Containers:
   stash-init:
    Image:      suaas21/stash:vs_linux_amd64
    Port:       <none>
    Host Port:  <none>
    Args:
      restore
      --restore-session=ss-restore
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
    Command:
      sleep
      3600
    Environment:  <none>
    Mounts:
      /source/data from source-data (rw)
  Volumes:
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
Volume Claims:
  Name:          source-data
  StorageClass:  standard
  Labels:        <none>
  Annotations:   <none>
  Capacity:      1Gi
  Access Modes:  [ReadWriteOnce]
...
```

Notice the `Init-Containers` section. We can see that the init-container `stash-init` has been injected which is running `restore` command.

**Wait for RestoreSession to Succeeded:**

Run the following command to watch RestoreSession phase,

```console
$ watch -n 3 kubectl get restoresession -n demo
Every 5.0s: kubectl get restoresession -n demo               suaas-appscode: Tue Jun 25 18:27:30 2019

NAME         REPOSITORY-NAME   PHASE       AGE
ss-restore   gcs-repo          Succeeded   4m31s
```

So, we can see from the output of the above command that the restore process succeeded.

> **Note:** If you want to restore the backed up data inside the same StatefulSet whose volumes were backed up, you have to remove the corrupted data from the StatefulSet. Then, you have to create a RestoreSession targeting the StatefulSet.

**Verify Restored Data:**

In this section, we are going to verify that the desired data has been restored successfully.

At first, check if the `stash-recovered` pods of a StatefulSet has gone into `Running` state by the following commands,

```console
$ kubectl get pod -n demo
NAME                READY   STATUS    RESTARTS   AGE
stash-recovered-0   1/1     Running   0          10m
stash-recovered-1   1/1     Running   0          11m
stash-recovered-2   1/1     Running   0          12m
```

Verify that the backed up data has been restored in `/source/data` directory of the `stash-recovered` pods of a StatefulSet using the following commands,

```console
$ kubectl exec -n demo stash-recovered-0 -- cat /source/data/data.txt
stash-demo-0
$ kubectl exec -n demo stash-recovered-1 -- cat /source/data/data.txt
stash-demo-1
$ kubectl exec -n demo stash-recovered-2 -- cat /source/data/data.txt
stash-demo-2
```

### Customize Restore Process

Generally, Stash restores data in individual replicas from a backup of the respective replica of the original StatefulSet. That means, backed up data of `pod-0` of original StatefulSet will be restored in `pod-0` of new StatefulSet and so on. However, you can customize this behavior through the `spec.rules` section of RestoreSession object. This is particularly helpful when your restored StatefulSet has a different number of replicas than the original StatefulSet. You can control which data will be restored in the additional replicas.

**Stop Taking Backup of the Old StatefulSet:**

At first, let's stop taking any further backup of the old StatefulSet so that no backup is taken during the restore process. We are going to pause the `BackupConfiguration` that we created to backup the `stash-demo` StatefulSet. Then, Stash will stop taking any further backup for this StatefulSet. You can learn more how to pause a scheduled backup [here](/docs/guides/latest/advanced-use-case/pause-backup.md)

Let's pause the `deployment-backup` BackupConfiguration,

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

Notice the `PAUSED` column. Value `true` for this field means that the BackupConfiguration has been paused.

**Deploy StatefulSet:**

We are going to create a new StatefulSet named `stash-recovered-adv` with `spec.replica` 5 and restore the backed up data inside it.

Below is the YAML of the StatefulSet that we are going to create,

```yaml
apiVersion: v1
kind: Service
metadata:
  name: adv-headless
  namespace: demo
spec:
  ports:
  - name: http
    port: 80
    targetPort: 0
  selector:
    app: stash-recovered-adv
  clusterIP: None
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: stash-recovered-adv
  namespace: demo
  labels:
    app: stash-recovered-adv
spec:
  replicas: 5
  selector:
    matchLabels:
      app: stash-recovered-adv
  serviceName: adv-headless
  template:
    metadata:
      labels:
        app: stash-recovered-adv
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

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/workloads/statefulset/adv_statefulset.yaml
service/adv-headless created
statefulset.apps/stash-recovered-adv created
```

**Create RestoreSession:**

Now, we are going to create a `RestoreSession` crd targeting the `stash-recovered` StatefulSet to restore the backed up data inside it.

Below is the YAML of the `RestoreSesion` crd that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: ss-restore
  namespace: demo
spec:
  driver: Restic
  repository:
    name: gcs-repo
  target:
    ref:
      apiVersion: apps/v1
      kind: StatefulSet
      name: stash-recovered-adv
    volumeMounts:
    - mountPath: /source/data
      name: source-data
  rules:
  - targetHosts: ["host-3","host-4"]
    sourceHost: "host-1"
    paths:
    - /source/data
  - targetHosts: []
    sourceHost: ""
    paths:
    - /source/data
```

Here,

- `spec.rules`: `spec.rules` specify how Stash should restore data for each host.
  - `targetHosts` the first rule specify that backed up data of `host-1`(old pods of a StatefulSet-1) will be restored into targetHosts `host-3`(new pod-3 of the StatefulSet) and `host-4`(new pod-4 of the StatefulSet) and the second rule specify that data from a similar backup host will be restored on the respective restore host. That means, backed up data of `host-0` will be restored into `host-0`, backed up data of `host-1` will be restored into `host-1` and so on.
  - `sourceHost` specifies the name of the host whose backed up data will be restored.

Let's create the `RestoreSession` crd we have shown above,

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/workloads/statefulset/adv_restoresession.yaml
restoresession.stash.appscode.com/ss-restore created
```

Once, you have created the `RestoreSession` crd, Stash will inject `init-container` into `stash-recovered` StatefulSet. The StatefulSet will restart and the `init-container` will restore the desired data on start-up.

**Verify Init-Container:**

Wait until the `init-container` has been injected into the `stash-recovered` StatefulSet. Let’s describe the StatefulSet to verify that the `init-container` has been injected successfully.

```yaml
$ kubectl describe statefulset -n demo stash-recovered-adv
Name:               stash-recovered-adv
Namespace:          demo
Selector:           app=stash-recovered-adv
Labels:             app=stash-recovered-adv
Replicas:           5 desired | 5 total
Pods Status:        5 Running / 0 Waiting / 0 Succeeded / 0 Failed
...
Pod Template:
  Labels:       app=stash-recovered-adv
  Annotations:  stash.appscode.com/last-applied-restoresession-hash: 4338322130475899419
  Init Containers:
   stash-init:
    Image:      suaas21/stash:vs_linux_amd64
    Port:       <none>
    Host Port:  <none>
    Args:
      restore
      --restore-session=ss-restore
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
    Command:
      sleep
      3600
    Environment:  <none>
    Mounts:
      /source/data from source-data (rw)
  Volumes:
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
Volume Claims:
  Name:          source-data
  StorageClass:  standard
  Labels:        <none>
  Annotations:   <none>
  Capacity:      1Gi
  Access Modes:  [ReadWriteOnce]
...
```

Notice the `Init-Containers` section. We can see that the init-container `stash-init` has been injected which is running `restore` command.

**Wait for RestoreSession to Succeeded:**

Run the following command to watch RestoreSession phase,

```console
$ watch -n 3 kubectl get restoresession -n demo
Every 5.0s: kubectl get restoresession -n demo               suaas-appscode: Tue Jun 25 18:27:30 2019

NAME         REPOSITORY-NAME   PHASE       AGE
ss-restore   gcs-repo          Succeeded   8m21s
```

So, we can see from the output of the above command that the restore process succeeded.

**Verify Restored Data:**

In this section, we are going to verify that the desired data has been restored successfully.

At first, check if the `stash-recovered` pods of the StatefulSet has gone into `Running` state by the following commands,

```console
$ kubectl get pod -n demo
NAME                    READY   STATUS    RESTARTS   AGE
stash-recovered-adv-0   1/1     Running   0          3m30s
stash-recovered-adv-1   1/1     Running   0          4m50s
stash-recovered-adv-2   1/1     Running   0          6m
stash-recovered-adv-3   1/1     Running   0          7m10s
stash-recovered-adv-4   1/1     Running   0          8m1s
```

Verify that the sample data has been restored in `/source/data` directory of the `stash-recovered` pods of the StatefulSet using the following commands,

```console
$ kubectl exec -n demo stash-recovered-adv-0 -- cat /source/data/data.txt
stash-demo-0
$ kubectl exec -n demo stash-recovered-adv-1 -- cat /source/data/data.txt
stash-demo-1
$ kubectl exec -n demo stash-recovered-adv-2 -- cat /source/data/data.txt
stash-demo-2
$ kubectl exec -n demo stash-recovered-adv-3 -- cat /source/data/data.txt
stash-demo-1
$ kubectl exec -n demo stash-recovered-adv-4 -- cat /source/data/data.txt
stash-demo-1
```

We can see from the above output that backup data of `host-1` has been restored into `host-3` and `host-4` successfully.

## Cleaning Up

To clean up the Kubernetes resources created by this tutorial, run:

```console
kubectl delete -n demo statefulset stash-demo
kubectl delete -n demo statefulset stash-recovered
kubectl delete -n demo backupconfiguration ss-backup
kubectl delete -n demo restoresession ss-restore
kubectl delete -n demo repository gcs-repo
kubectl delete -n demo pvc --all
```
