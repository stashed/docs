---
title: Simultaneous Backup volumes of Workloads and Standalone PVC | Stash
description: A step by step guide showing how to backup volumes of Workloads and Standalone PVC Simultaneously.
menu:
  product_stash_{{ .version }}:
    identifier: volumes-backup
    name: Backup Volumes of Workloads and Standalone PVC simultaneously
    parent: backup-multiple-target
    weight: 30
product_name: stash
menu_name: product_stash_{{ .version }}
section_menu_id: guides
---

# Simultaneous Backup volumes of workloads and standalone PVCs

Stash gives you backup volumes of workloads and standalone PVCs support simultaneously. This guide will show you how to use Stash to take backup volumes of workload and standalone PVCs simultaneously.

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [kind](https://kind.sigs.k8s.io/docs/user/quick-start/).

- Install `Stash` in your cluster following the steps [here](/docs/setup/install.md).

- You should be familiar with the following `Stash` concepts:
  - [BackupBatch](/docs/concepts/crds/backupbatch.md)
  - [BackupSession](/docs/concepts/crds/backupsession.md)
  - [Repository](/docs/concepts/crds/repository.md)
  - [Function](/docs/concepts/crds/function.md)
  - [Task](/docs/concepts/crds/task.md)

To keep everything isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$ kubectl create ns demo
namespace/demo created
```

**Verify necessary Function and Task:**

Stash uses a `Function-Task` model to backup stand-alone volume. When you install Stash, it automatically creates the necessary `Function` and `Task` to backup a stand-alone volume.

Let's verify that Stash has created the necessary `Function` to backup/restore PVC by the following command,

```console
$ kubectl get function
NAME            AGE
pvc-backup      117m
pvc-restore     117m
update-status   117m
```

Also, verify that the necessary `Task` has been created,

```console
$ kubectl get task
NAME          AGE
pvc-backup    118m
pvc-restore   118m
```

> **Note:** YAML files used in this tutorial are stored in [docs/examples/guides/latest/backup-multiple-target](/docs/examples/guides/latest/workloads) directory of [stashed/docs](https://github.com/stashed/docs) repository.

## Backup Volumes of multiple Workload and Standalone PVCs

This section will show you how to use Stash to take backup volumes of Deployment and standalone PVCs together. Here, we are going to create two PVCs and mount it with pods and generate some sample data in it. We are also going to deploy a Deployment with a PVC and also generate some sample data in it. Then, we are going to backup the PVC's and Deployment's data simultaneously.

### Prepare Volumes

**Create PVCs and Pods :**

Below are the YAML of the PVCs and Pods that we are going to create,

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: demo-pod
  namespace: demo
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["/bin/sh", "-c","echo 'hello from pod' > /sample/data/hello.txt && sleep 3000"]
    volumeMounts:
    - name: my-volume
      mountPath: /sample/data
  volumes:
  - name: my-volume
    persistentVolumeClaim:
      claimName: demo-pvc
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: demo-pvc
  namespace: demo
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
kind: Pod
apiVersion: v1
metadata:
  name: config-pod
  namespace: demo
spec:
  containers:
    - name: busybox
      image: busybox
      command: ["/bin/sh", "-c","echo 'hello from pod' > /sample/data/hello.txt && sleep 3000"]
      volumeMounts:
        - name: my-volume
          mountPath: /sample/data
  volumes:
    - name: my-volume
      persistentVolumeClaim:
        claimName: config-pvc
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: config-pvc
  namespace: demo
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

The above Pods will automatically generate `hello from pod` data in `hello.txt` file in `/sample/data` directory.

Let's create the PVCs and Pods we have shown above,

```console
$ kubectl apply  -f ./docs/examples/guides/latest/backup-multiple-target/volumes/pvcs.yaml
pod/demo-pod created
persistentvolumeclaim/demo-pvc created
pod/config-pod created
persistentvolumeclaim/config-pvc created
```

Now, wait for the pods to go into the `Running` state.

```console
$ kubectl get pods -n demo
NAME         READY   STATUS    RESTARTS   AGE
config-pod   1/1     Running   0          79s
demo-pod     1/1     Running   0          79s
```

Verify that the sample data has been created in `/sample/data` directory using the following command,

```console
$ kubectl exec -n demo config-pod -- cat /sample/data/hello.txt
hello from pod
 $ kubectl exec -n demo demo-pod -- cat /sample/data/hello.txt
hello from pod
```

**Deploy a Deployment with a PVC :**

Below are the YAML of the Deployment and PVC that we are going to create,

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: stash-demo
  name: stash-demo
  namespace: demo
spec:
  replicas: 2
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
---
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

The above Deployment will automatically create `smaple_data` data in `data.txt` file in `/source/data` directory.

Let's create the Deployment we have shown above,

```console
$ kubectl apply  -f ./docs/examples/guides/latest/backup-multiple-target/volumes/workload.yaml
deployment.apps/stash-demo created
persistentvolumeclaim/stash-sample-data created
```

Now, wait for the pods of the Deployment to go into the `Running` state.

```console
$ kubectl get pods -n demo
NAME                          READY   STATUS    RESTARTS   AGE
...
stash-demo-6f5dc86955-7znb8   1/1     Running   0          33s
stash-demo-6f5dc86955-wrb4x   1/1     Running   0          33s
```

Verify that the sample data has been created in `/source/data` directory using the following command,

```console
$ kubectl exec -n demo stash-demo-6f5dc86955-wrb4x -- cat /source/data/data.txt
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
      bucket: appscode-testing
      prefix: /demo/volumes
    storageSecretName: gcs-secret
```

Let's create the Repository we have shown above,

```console
$ kubectl apply -f ./docs/examples/guides/latest/backup-multiple-target/volumes/repository.yaml
repository.stash.appscode.com/gcs-repo created
```

Now, we are ready to backup our sample data into this backend.

### Backup

We have to create a `BackupBatch` crd targeting the `config-pvc`, `demo-pvc` PVC and `stash-demo` Deployment respectively that we have deployed earlier. Stash will inject a sidecar container into Deployment target. It will also create a `CronJob` to take periodic backup of `/sample/data`(for PVC) and `/source/data`(for Deployment) directory of the target respectively.

**Create BackupBatch:**

Below is the YAML of the `BackupBatch` crd that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupBatch
metadata:
  name: volume-backup-batch
  namespace: demo
spec:
  repository:
    name: gcs-repo
  schedule: "*/5 * * * *"
  backupConfigurationTemplates:
  - spec:
      task:
        name: pvc-backup
      target:
        ref:
          apiVersion: v1
          kind: PersistentVolumeClaim
          name: demo-pvc
  - spec:
      task:
        name: pvc-backup
      target:
        ref:
          apiVersion: v1
          kind: PersistentVolumeClaim
          name: config-pvc
  - spec:
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
- `spec.schedule` is a cron expression that indicates `BackupSession` will be created at 6 minute interval.
- `spec.backupConfigurationTemplates[].spec.target.ref` refers to the `demo-pvc`, `config-pvc` PVC and `deploy-stash-demo` Deployment respectively.
- `spec.backupConfigurationTemplates[].spec.task` specifies the name of the `Task` object that specifies the `Function` and their order of execution to perform a backup of a stand-alone PVC.
- `spec.backupConfigurationTemplates[].spec.target.volumeMounts` specifies a list of volumes and their mountPath that contain the target paths.
- `spec.backupConfigurationTemplates[].spec.target.paths` specifies list of file paths to backup for target.

Let's create the `BackupBatch` crd we have shown above,

```console
$ kubectl apply -f ./docs/examples/guides/latest/backup-multiple-target/volumes/backupbatch.yaml
backupbatch.stash.appscode.com/deploy-backup-batch created
```

**Verify Sidecar:**

If everything goes well, Stash will inject a sidecar container into the `stash-demo` Deployment to take backup of `/source/data` directory. Let’s check that the sidecar has been injected successfully and wait for all the pods to go into the `Running` state,

```console
$ kubectl get pod -n demo
NAME                         READY   STATUS    RESTARTS   AGE
...
stash-demo-95c8c78b9-5f6mv   2/2     Running   0          44s
stash-demo-95c8c78b9-7cfhr   2/2     Running   0          74s
```

Look at the pod. It now has 2 containers. If you view the resource definition of the pod, you will see that there is a container named `stash` which is running `run-backup` command.

```console
$ kubectl get pod -n demo stash-demo-95c8c78b9-5f6mv -o yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  ...
  creationTimestamp: "2019-12-13T10:06:49Z"
  generateName: stash-demo-95c8c78b9-
  labels:
    app: stash-demo
    pod-template-hash: 95c8c78b9
  name: stash-demo-95c8c78b9-5f6mv
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
      name: default-token-t5rkw
      readOnly: true
  - args:
    - run-backup
    - --invokername=volume-backup-batch
    - --invokertype=BackupBatch
    - --targetname=stash-demo
    - --targetkind=Deployment
    - --secret-dir=/etc/stash/repository/secret
    - --enable-cache=true
    - --max-connections=0
    - --metrics-enabled=true
    - --pushgateway-url=http://stash-operator.kube-system.svc:56789
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
    image: suaas21/stash:backup-batch_linux_amd64
    imagePullPolicy: IfNotPresent
    name: stash
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
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
      name: default-token-t5rkw
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: kind-worker2
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext:
    fsGroup: 65535
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
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
  - name: default-token-t5rkw
    secret:
      defaultMode: 420
      secretName: default-token-t5rkw
  ...
...
```

**Verify CronJob:**

Stash will also create a `CronJob` with the schedule specified in `spec.schedule` field of `BackupBatch` crd.

Verify that Stash has created a CronJob to trigger a periodic backup of the targeted PVC and Deployment by the following command,

```console
$ kubectl get cronjob -n demo
NAME                  SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
volume-backup-batch   */5 * * * *   False     0        75s             4m56s
```

**Wait for BackupSession:**

The `volume-backup-batch` CronJob will trigger a backup on each scheduled slot by creating a `BackpSession` crd. 
The sidecar container and the stash operator watches for the `BackupSession` crd. When they finds one, they will take backup of the Deployment and PVC separately.

Wait for the next schedule for backup. Run the following command to watch `BackupSession` crd,

```console
$ watch -n 2 kubectl get backupsession -n demo
Every 2.0s: kubectl get backupsession --all-namespaces           suaas-appscode: Fri Dec 13 16:12:31 2019

NAMESPACE   NAME                             INVOKER-TYPE   INVOKER-NAME          PHASE       AGE
demo        volume-backup-batch-1576231804   BackupBatch    volume-backup-batch   Succeeded   2m27s
```

We can see from the above output that the backupSession has succeeded.

If you describe the `volume-backup-batch-1576231804` BackupSession crd, you will see in the status section of the `BackupSession` that all the target specified in the `BackupBatch` have succeeded.

```console
$ kubectl describe backupsession -n demo volume-backup-batch-1576231804
```

```yaml
Name:         volume-backup-batch-1576231804
Namespace:    demo
Labels:       app.kubernetes.io/component=stash-backup
              app.kubernetes.io/managed-by=stash.appscode.com
              stash.appscode.com/invoker-name=volume-backup-batch
              stash.appscode.com/invoker-type=BackupBatch
Annotations:  <none>
API Version:  stash.appscode.com/v1beta1
Kind:         BackupSession
Metadata:
  Creation Timestamp:  2019-12-13T10:10:04Z
  Generation:          1
  Owner References:
    API Version:           stash.appscode.com/v1beta1
    Block Owner Deletion:  false
    Kind:                  BackupBatch
    Name:                  volume-backup-batch
    UID:                   a7a455ce-7db1-4e62-a027-a2efbfdeb010
  Resource Version:        43585
  Self Link:               /apis/stash.appscode.com/v1beta1/namespaces/demo/backupsessions/volume-backup-batch-1576231804
  UID:                     6443758c-b9c1-495e-a26f-b0bf3dd8110d
Spec:
  Invoker:
    API Group:  stash.appscode.com
    Kind:       BackupBatch
    Name:       volume-backup-batch
Status:
  Phase:             Succeeded
  Session Duration:  40.486551206s
  Targets:
    Ref:
      Kind:  Deployment
      Name:  stash-demo
    Stats:
      Duration:  29.417982691s
      Hostname:  host-0
      Phase:     Succeeded
      Snapshots:
        File Stats:
          Modified Files:    0
          New Files:         1
          Total Files:       1
          Unmodified Files:  0
        Name:                d6a4485e
        Path:                /source/data
        Processing Time:     0:03
        Total Size:          12 B
        Uploaded:            717 B
    Target Phase:            Succeeded
    Total Hosts:             1
    Ref:
      Kind:  PersistentVolumeClaim
      Name:  demo-pvc
    Stats:
      Duration:  26.782035107s
      Hostname:  host-0
      Phase:     Succeeded
      Snapshots:
        File Stats:
          Modified Files:    0
          New Files:         1
          Total Files:       1
          Unmodified Files:  0
        Name:                44807c2c
        Path:                /stash-data
        Processing Time:     0:03
        Total Size:          15 B
        Uploaded:            373 B
    Target Phase:            Succeeded
    Total Hosts:             1
    Ref:
      Kind:  PersistentVolumeClaim
      Name:  config-pvc
    Stats:
      Duration:  30.449523597s
      Hostname:  host-0
      Phase:     Succeeded
      Snapshots:
        File Stats:
          Modified Files:    0
          New Files:         1
          Total Files:       1
          Unmodified Files:  0
        Name:                2ccf9a3d
        Path:                /stash-data
        Processing Time:     0:03
        Total Size:          15 B
        Uploaded:            373 B
    Target Phase:            Succeeded
    Total Hosts:             1
Events:
  Type    Reason                   Age    From                      Message
  ----    ------                   ----   ----                      -------
  Normal  BackupSession Running    5m15s  BackupSession Controller  Backup job has been created succesfully/sidecar is watching the BackupSession.
  Normal  BackupSession Running    5m14s  BackupSession Controller  Backup job has been created succesfully/sidecar is watching the BackupSession.
  Normal  BackupSession Running    5m14s  BackupSession Controller  Backup job has been created succesfully/sidecar is watching the BackupSession.
  Normal  Host Backup Succeeded    4m45s  Status Updater            backup succeeded for host host-0 of "Deployment"/"stash-demo".
  Normal  Host Backup Succeeded    4m40s  Status Updater            backup succeeded for host host-0 of "PersistentVolumeClaim"/"demo-pvc".
  Normal  Host Backup Succeeded    4m35s  Status Updater            backup succeeded for host host-0 of "PersistentVolumeClaim"/"config-pvc".
  Normal  BackupSession Succeeded  4m35s  BackupSession Controller  Backup session completed successfully
```

```yaml
```

Now, we are going to verify whether the backed up data has been stored in the backend.

**Verify Backup:**

When backup session is completed, Stash will update the respective `Repository` to reflect the latest state of backed up data.

Run the following command to check if a backup snapshot has been stored in the backend,

```console
$ kubectl get repository -n demo gcs-repo
NAME       INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
gcs-repo   true               1                3m58s                    10m
```

From the output above, we can see that 1 snapshot has been stored in the backend specified by Repository `gcs-repo`.

Now, if we navigate to the GCS bucket, we are going to see backed up data has been stored in `/demo/volumes/<target-kind>/<target-name>` directory.
> Stash forms the directory by concatenating the `spec.backend.gcs.prefix` field of Repository crd and `<target-kind>/<target-name>`.

<figure align="center">
  <img alt="Backup data in GCS Bucket" src="/docs/images/guides/latest/backup-multiple-target/volumes.png">
  <figcaption align="center">Fig: Backup data in GCS Bucket</figcaption>
</figure>

> **Note:** Stash keeps all the backed up data encrypted. So, data in the backend will not make any sense until they are decrypted.

## Cleaning Up

To clean up the Kubernetes resources created by this tutorial, run:

```console
kubectl delete -n demo backupbatch volume-backup-batch
kubectl delete -n demo deployment stash-demo
kubectl delete -n demo pod --all
kubectl delete -n demo repository gcs-repo
kubectl delete -n demo pvc --all
```