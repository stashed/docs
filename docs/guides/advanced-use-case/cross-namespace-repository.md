---
title: Cross Namespce Repository | Stash
description: A guide on how to  use cross-namespace repository to take backup and restore using Stash.
menu:
  docs_{{ .version }}:
    identifier: advance-use-case-cross-namespace-repo
    name: Cross-Namespace Backup and Restore
    parent: advance-use-case
    weight: 10
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: guides
---

# Cross-Namespace Backup and Restore

This guide will show you how to use cross-namespace repository to take backup and restore in Stash.

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [kind](https://kind.sigs.k8s.io/docs/user/quick-start/).

- Install `Stash` in your cluster following the steps [here](/docs/setup/README.md).

- You should be familiar with the following `Stash` concepts:
  - [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md)
  - [BackupSession](/docs/concepts/crds/backupsession.md)
  - [RestoreSession](/docs/concepts/crds/restoresession.md)
  - [Repository](/docs/concepts/crds/repository.md)

To demonstrate the cross-namespace capability, we are going to use `dev` namespace for taking backup, `staging` namespace for restoring backup, and crete `Repository` in a separate namespace called `stash` in this tutorial.

```bash
$ kubectl create ns dev
namespace/dev created

$ kubectl create ns staging
namespace/staging created

$ kubectl create ns stash
namespace/stash created
```

>**Note:** YAML files used in this tutorial are stored in [docs/examples/guides/advanced-use-case/cross-namespace-repo](/docs/examples/guides/advanced-use-case/cross-namespace-repo) directory of [stashed/docs](https://github.com/stashed/docs) repository.


## Configure Backup

Here, we are going to configure backup for volumes of a StatefulSet in the `dev` namespace. We will deploy a StatefulSet with 3 replicas and a service. We will generate some sample data in it. Then, we are going to configure backup for this StatefulSet using Stash referencing a Repository from `stash` namespace.


### Deploy StatefulSet

Let's deploy a StatefulSet in `dev` namespace at the begining. This StatefulSet will automatically generate sample data in `/source/data` directory.

Here is the YAML of the StatefulSet that we are going to create,

```yaml
apiVersion: v1
kind: Service
metadata:
  name: headless
  namespace: dev
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
  namespace: dev
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

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/advanced-use-case/cross-namespace-repo/statefulset.yaml
service/headless created
statefulset.apps/stash-demo created
```

Now, wait for the pods of the StatefulSet to go into the `Running` state. You can verify the Status of the pods by ececuting the following command,

```bash
$ kubectl get pod -n dev
NAME           READY   STATUS    RESTARTS   AGE
stash-demo-0   1/1     Running   0          42s
stash-demo-1   1/1     Running   0          40s
stash-demo-2   1/1     Running   0          36s
```

Verify that the sample data has been generated in `/source/data` directory for `stash-demo-0` , `stash-demo-1` and `stash-demo-2` pod respectively using the following commands,

```bash
$ kubectl exec -n dev stash-demo-0 -- cat /source/data/data.txt
stash-demo-0
$ kubectl exec -n dev stash-demo-1 -- cat /source/data/data.txt
stash-demo-1
$ kubectl exec -n dev stash-demo-2 -- cat /source/data/data.txt
stash-demo-2
```

### Prepare Backend

We are going to store our backed up data into a GCS bucket. We have to create a Secret with necessary credentials and a Repository CRD to use this backend. We will create the Secret and Repository in `stash` namespace. With the cross-namespace Repository support, Stash has the ability to backup and restore data in a different namespace.

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

Let's create the Repository we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/advanced-use-case/cross-namespace-repo/repository.yaml
repository.stash.appscode.com/gcs-repo created
```

> Please see the `spec.usagePolicy` section of the Repository here. From the `allowedNamespaces` section we have allowed all the namespaces to refer this Repository. Also note that, we need to keep the `Repository` and the `Secret` in the same namespace

Now, we are ready to backup our sample data into this backend.

### Backup

We are going to create a `BackupConfiguration` crd in `dev` namespace targeting the `stash-demo` StatefulSet that we have deployed earlier. This `BackupConfiguration` will refer to the `gcs-repo` repository of `stash` namespace. Stash will inject a sidecar container into the target. It will also create a `CronJob` to take periodic backup of `/source/data` directory of the target.

**Create BackupConfiguration:**

Below is the YAML of the `BackupConfiguration` crd that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: ss-backup
  namespace: dev
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

- `spec.repository` refers to the `Repository` object `gcs-repo` that holds backend information.
- `spec.repository.namespace` refers to the namespace of the `Repository` object `gcs-repo`.
- `spec.schedule` is a cron expression that indicates `BackupSession` will be created at 5 minute interval.
- `spec.target.ref` refers to the `stash-demo` StatefulSet.
- `spec.target.volumeMounts` specifies a list of volumes and their mountPath that contain the target paths.
- `spec.target.paths` specifies list of file paths to backup.

Let's create the `BackupConfiguration` crd we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/workloads/statefulset/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/ss-backup created
```

**Verify Sidecar:**

If everything goes well, Stash will inject a sidecar container into the `stash-demo` StatefulSet to take backup of `/source/data` directory. Let’s check that the sidecar has been injected successfully,

```bash
$ kubectl get pod -n dev
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
    controller-revision-hash: stash-demo-6cf96db445
    statefulset.kubernetes.io/pod-name: stash-demo-0
  name: stash-demo-0
  namespace: dev
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
      name: kube-api-access-fpt7b
      readOnly: true
  - args:
    - run-backup
    - --invoker-name=ss-backup
    - --invoker-kind=BackupConfiguration
    - --target-name=stash-demo
    - --target-kind=StatefulSet
    - --enable-cache=true
    - --max-connections=0
    - --metrics-enabled=true
    - --pushgateway-url=http://stash-stash-enterprise.kube-system.svc:56789
    - --use-kubeapiserver-fqdn-for-aks=true
    - --logtostderr=true
    - --alsologtostderr=false
    - --v=3
    - --stderrthreshold=2
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
    volumeMounts:
    - mountPath: /tmp
      name: tmp-dir
    - mountPath: /source/data
      name: source-data
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-fpt7b
      readOnly: true
  hostname: stash-demo-0
  nodeName: kind-control-plane
   volumes:
  - name: source-data
    persistentVolumeClaim:
      claimName: source-data-stash-demo-0
  - emptyDir: {}
    name: tmp-dir
  - name: kube-api-access-fpt7b
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
  ...
...
```

**Verify CronJob:**

If everything goes well, Stash will also create a `CronJob` with the schedule specified in `spec.schedule` field of the `BackupConfiguration` crd.

Verify that the `CronJob` has been created using the following command,

```bash
$ kubectl get cronjob -n  dev
NAME                      SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
stash-trigger-ss-backup   */5 * * * *   False     0        4m55s           3m14s
```

**Wait for BackupSession:**

The `stash-trigger-ss-backup` CronJob will trigger a backup on each scheduled slot by creating a `BackupSession` crd. The sidecar container watches for the `BackupSession` crd. When it finds one, it will take backup immediately.

Wait for the next schedule for backup. Run the following command to watch `BackupSession` crd,

```bash
$ kubectl get backupsession -n dev -w

NAME                   INVOKER-TYPE          INVOKER-NAME    PHASE      DURATION   AGE
ss-backup-1644562803   BackupConfiguration   ss-backup      Running                18s
ss-backup-1644562803   BackupConfiguration   ss-backup      Running                34s
ss-backup-1644562803   BackupConfiguration   ss-backup      Succeeded   1m21s      81s
```

We can see from the above output that the backup session has succeeded. Now, we are going to verify whether the backed up data has been stored in the backend.

**Verify Backup:**

Once a backup is complete, Stash will update the respective `Repository` crd of the `BackupConfiguration` to reflect the backup. Check that the repository `gcs-repo` of `stash` namespace has been updated by the following command,

```bash
$ kubectl get repository -n stash
NAME       INTEGRITY   SIZE        SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
gcs-repo   true        3.161 KiB   15               3m31s                    31m
```

Now, if we navigate to the GCS bucket, we should see that backed up data has been stored in `/cross-namespace/data/sample-statefulset` directory as specified by `spec.backend.gcs.prefix` field of Repository crd.

<figure align="center">
  <img alt="Backup data in GCS Bucket" src="/docs/images/guides/advanced-use-case/cross_namespace.png">
  <figcaption align="center">Fig: Backup data in GCS Bucket</figcaption>
</figure>

> **Note:** Stash keeps all the backed up data encrypted. So, data in the backend will not make any sense until they are decrypted.



## Restore Volumes of a StatefulSet

This section will show you how to restore the backed up data in a different namespace from the backend we have taken previously. We will demonstrate restoring volumes in `staging` namespace using the Repository of the `stash` namespace created earlier.

**Stop Taking Backup of the Old StatefulSet:**

At first, let's stop taking any further backup of the old StatefulSet so that no backup is taken during the restore process. We are going to pause the `BackupConfiguration` that we created to backup the `stash-demo` StatefulSet. Then, Stash will stop taking any further backup for this StatefulSet. You can learn more how to pause a scheduled backup [here](/docs/guides/advanced-use-case/pause-backup.md)

Let's pause the `ss-backup` BackupConfiguration,

```bash
$ kubectl patch backupconfiguration -n dev ss-backup --type="merge" --patch='{"spec": {"paused": true}}'
backupconfiguration.stash.appscode.com/ss-backup patched
```

Verify that the BackupConfiguration  has been paused,

```bash
$ kubectl get backupconfiguration -n dev
NAME        TASK   SCHEDULE      PAUSED   PHASE   AGE
ss-backup          */5 * * * *   true     Ready   53m
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
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/advanced-use-case/cross-namespace-repo/statefulset_recovered.yaml
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

- `spec.repository.name` specifies the `Repository` crd that holds the backend information where our backed up data has been stored.
- `spec.repository.namespace` refers to the namespace of the `Repository` object `gcs-repo`.
- `spec.target.ref` refers to the target workload where the recovered data will be stored.
- `spec.target.volumeMounts` specifies a list of volumes and their mountPath where the data will be restored.
  - `mountPath` must be same `mountPath` as the original volume because Stash stores absolute path of the backed up files. If you use different `mountPath` for the restored volume the backed up files will not be restored into your desired volume.

Let's create the `RestoreSession` crd we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/workloads/statefulset/restoresession.yaml
restoresession.stash.appscode.com/ss-restore created
```

Once, you have created the `RestoreSession` crd, Stash will inject `init-container` into `stash-recovered` StatefulSet. The StatefulSet will restart and the `init-container` will restore the desired data on start-up.

**Verify Init-Container:**

Wait until the `init-container` has been injected into the `stash-recovered` StatefulSet. Let’s describe the StatefulSet to verify that the `init-container` has been injected successfully.

```yaml
$ kubectl describe statefulset -n staging stash-recovered
Name:               stash-recovered
Namespace:          demo
Selector:           app=stash-recovered
Labels:             app=stash-recovered
Replicas:           3 desired | 3 total
Update Strategy:    RollingUpdate
Partition:        0
Pods Status:        3 Running / 0 Waiting / 0 Succeeded / 0 Failed

...
 stash-init:
    Image:      appscodeci/stash-enterprise:v0.17.0-11-g3f50cb35_linux_amd64
    Port:       <none>
    Host Port:  <none>
    Args:
      restore
      --invoker-kind=RestoreSession
      --invoker-name=ss-restore
      --target-name=stash-recovered
      --target-kind=StatefulSet
      --enable-cache=true
      --max-connections=0
      --metrics-enabled=true
      --pushgateway-url=http://stash-stash-enterprise.kube-system.svc:56789
      --use-kubeapiserver-fqdn-for-aks=true
      --logtostderr=true
      --alsologtostderr=false
      --v=3
      --stderrthreshold=2
    Environment:
      NODE_NAME:   (v1:spec.nodeName)
      POD_NAME:    (v1:metadata.name)
    Mounts:
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

```bash
$ kubectl get restoresession -n staging -w

NAME         REPOSITORY   PHASE       DURATION   AGE
ss-restore   gcs-repo     Succeeded   3m6s       4m
```

So, we can see from the output of the above command that the restore process succeeded.


**Verify Restored Data:**

In this section, we are going to verify that the desired data has been restored successfully.

At first, check if the `stash-recovered` pods of a StatefulSet has gone into `Running` state by the following commands,

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

Throughout the above sections, we have demonstrated how we can use cross-namespace backup and restore feature. We have taken backup from `dev` namespace and restored the backup data into `staging` namespace while the Repository was in `stash` namespace.

## Cleaning Up

To clean up the Kubernetes resources created by this tutorial, run:

```bash
kubectl delete -n dev statefulset stash-demo
kubectl delete -n staging statefulset stash-recovered
kubectl delete -n dev backupconfiguration ss-backup
kubectl delete -n staging restoresession ss-restore
kubectl delete -n stash repository gcs-repo
kubectl delete -n stash secret gcs-secret
kubectl delete -n dev pvc --all
kubectl delete -n staging pvc --all
```
