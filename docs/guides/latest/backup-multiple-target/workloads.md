---
title: Backup volumes of multiple Workload | Stash
description: A step by step guide showing how to backup volumes of multiple Workload.
menu:
  product_stash_{{ .version }}:
    identifier: workloads-backup
    name: Backup Volumes of multiple workload
    parent: backup-multiple-target
    weight: 20
product_name: stash
menu_name: product_stash_{{ .version }}
section_menu_id: guides
---

# Backup Volumes of Multiple workload

Stash gives you backup volumes of multiple workload support simultaneously. This guide will show you how to use Stash to backup multiple workload volumes.

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [kind](https://kind.sigs.k8s.io/docs/user/quick-start/).

- Install `Stash` in your cluster following the steps [here](/docs/setup/install.md).

- You should be familiar with the following `Stash` concepts:
  - [BackupBatch](/docs/concepts/crds/backupbatch.md)
  - [BackupSession](/docs/concepts/crds/backupsession.md)
  - [Repository](/docs/concepts/crds/repository.md)

To keep everything isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$ kubectl create ns demo
namespace/demo created
```

> **Note:** YAML files used in this tutorial are stored in [docs/examples/guides/latest/backup-multiple-target](/docs/examples/guides/latest/workloads) directory of [stashed/docs](https://github.com/stashed/docs) repository.

## Backup Volumes of multiple Workload

This section will show you how to use Stash to backup volumes of multiple workload. Here, we are going to deploy a DaemonSet, a StatefulSet and a Deployment with a PVC and generate some sample data in it. Then, we are going to backup this sample data using Stash.

### Prepare Workloads

Below are the YAML of the workloads and PVC that we are going to create,

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: stash-demo
  name: deploy-stash-demo
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
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: stash-demo
  name: dmn-stash-demo
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
---
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
  name: sts-stash-demo
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

The above Workloads will automatically create `smaple_data`(for Deployment and DaemonSet) and `$(POD_NAME)`(for StatefulSet) data in `data.txt` file in `/source/data` directory.

Let's create the Workloads we have shown above,

```console
kubectl apply  -f ./docs/examples/guides/latest/backup-multiple-target/workloads/workloads.yaml
deployment.apps/deploy-stash-demo created
persistentvolumeclaim/stash-sample-data created
daemonset.apps/dmn-stash-demo created
service/headless created
statefulset.apps/sts-stash-demo created
```

Now, wait for the pods of the Workloads(Deployment, DaemonSet and StatefulSet) to go into the `Running` state.

```console
$ kubectl get pod -n demo
NAME                                 READY   STATUS    RESTARTS   AGE
deploy-stash-demo-6f5dc86955-fwx5p   1/1     Running   0          19s
deploy-stash-demo-6f5dc86955-r8hm2   1/1     Running   0          19s
dmn-stash-demo-g67xk                 1/1     Running   0          19s
dmn-stash-demo-gv28k                 1/1     Running   0          19s
sts-stash-demo-0                     1/1     Running   0          19s
sts-stash-demo-1                     1/1     Running   0          14s
sts-stash-demo-2                     1/1     Running   0          9s
```

Verify that the sample data has been created in `/source/data` directory using the following command,

```console
 $ kubectl exec -n demo deploy-stash-demo-6f5dc86955-fwx5p -- cat /source/data/data.txt
sample_data
$ kubectl exec -n demo dmn-stash-demo-g67xk -- cat /source/data/data.txt
sample_data
$ kubectl exec -n demo sts-stash-demo-0 -- cat /source/data/data.txt
sts-stash-demo-0
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
      prefix: /demo/data
    storageSecretName: gcs-secret
```

Let's create the Repository we have shown above,

```console
$ kubectl apply -f ./docs/examples/guides/latest/backup-multiple-target/workloads/repository.yaml
repository.stash.appscode.com/gcs-repo created
```

Now, we are ready to backup our sample data into this backend.

### Backup

We have to create a `BackupBatch` crd targeting the `deploy-stash-demo` Deployment, `dmn-stash-demo` DaemonSet and `sts-stash-demo` StatefulSet respectively that we have deployed earlier. Stash will inject a sidecar container into all target. It will also create a `CronJob` to take periodic backup of `/source/data` directory of all target.

**Create BackupBatch:**

Below is the YAML of the `BackupBatch` crd that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupBatch
metadata:
  name: deploy-backup-batch
  namespace: demo
spec:
  repository:
    name: gcs-repo
  schedule: "*/6 * * * *"
  backupConfigurationTemplates:
  - spec:
      target:
        ref:
          apiVersion: apps/v1
          kind: Deployment
          name: deploy-stash-demo
        volumeMounts:
        - name: source-data
          mountPath: /source/data
        paths:
        - /source/data
  - spec:
      target:
        ref:
          apiVersion: apps/v1
          kind: DaemonSet
          name: dmn-stash-demo
      volumeMounts:
      - name: source-data
        mountPath: /source/data
      paths:
      - /source/data
  - spec:
      target:
        ref:
          apiVersion: apps/v1
          kind: StatefulSet
          name: sts-stash-demo
        volumeMounts:
        - name: source-data
          mountPath: /source/data
        paths:
        - /source/data
  retentionPolicy:
    name: 'keep-last-10'
    keepLast: 10
    prune: true
```

Here,

- `spec.repository` refers to the `Repository` object `gcs-repo` that holds backend information.
- `spec.schedule` is a cron expression that indicates `BackupSession` will be created at 6 minute interval.
- `spec.backupConfigurationTemplates[].spec.target.ref` refers to the `deploy-stash-demo` Deployment, `dmn-stash-demo` DaemonSet and `sts-stash-demo` StatefulSet respectively.
- `spec.backupConfigurationTemplates[].spec.target.volumeMounts` specifies a list of volumes and their mountPath that contain the target paths.
- `spec.backupConfigurationTemplates[].spec.target.paths` specifies list of file paths to backup for target.

Let's create the `BackupBatch` crd we have shown above,

```console
$ kubectl apply -f ./docs/examples/guides/latest/backup-multiple-target/workloads/backupbatch.yaml
backupbatch.stash.appscode.com/deploy-backup-batch created
```

**Verify Sidecar:**

If everything goes well, Stash will inject a sidecar container into the `deploy-stash-demo` Deployment, `dmn-stash-demo` DaemonSet and `sts-stash-demo` StatefulSet to take backup of `/source/data` directory respectively. Let’s check that the sidecar has been injected successfully and wait for all the pods to go into the `Running` state,

```console
$ kubectl get pod -n demo
NAME                               READY   STATUS    RESTARTS   AGE
deploy-stash-demo-ccc4b559-lq27s   2/2     Running   0          90s
deploy-stash-demo-ccc4b559-sfv6n   2/2     Running   0          2m
dmn-stash-demo-6bhv8               2/2     Running   0          84s
dmn-stash-demo-9rwl2               2/2     Running   0          44s
sts-stash-demo-0                   2/2     Running   0          4s
sts-stash-demo-1                   2/2     Running   0          44s
sts-stash-demo-2                   2/2     Running   0          84s
```

Look at the pods. They now have 2 containers. If you view the resource definition of any of the pod, you will see that there is a container named `stash` which is running `run-backup` command.

```console
$ kubectl get pod -n demo deploy-stash-demo-ccc4b559-sfv6n -o yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2019-12-12T06:02:04Z"
  generateName: deploy-stash-demo-ccc4b559-
  labels:
    app: stash-demo
    pod-template-hash: ccc4b559
  name: deploy-stash-demo-ccc4b559-sfv6n
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
      name: default-token-hb7vr
      readOnly: true
  - args:
    - run-backup
    - --invokername=deploy-backup-batch
    - --invokertype=BackupBatch
    - --targetname=deploy-stash-demo
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
  ...
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: kind-worker
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext:
    fsGroup: 65535
  serviceAccount: default
  serviceAccountName: default
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
  - name: default-token-hb7vr
    secret:
      defaultMode: 420
      secretName: default-token-hb7vr
  ...
...
```

**Verify CronJob:**

It will also create a `CronJob` with the schedule specified in `spec.schedule` field of `BackupBatch` crd.

Verify that the `CronJob` has been created using the following command,

```console
$ kubectl get cronjob -n demo
NAME                  SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
deploy-backup-batch   */5 * * * *   False     0        2m26s           15m
```

**Wait for BackupSession:**

The `deploy-backup-batch` CronJob will trigger a backup on each scheduled slot by creating a `BackpSession` crd. The sidecar container watches for the `BackupSession` crd. When it finds one, it will take backup immediately.

Wait for the next schedule for backup. Run the following command to watch `BackupSession` crd,

```console
Every 5.0s: kubectl get backupsession --all-namespaces           suaas-appscode: Thu Dec 12 12:08:51 2019

NAMESPACE   NAME                             INVOKER-TYPE   INVOKER-NAME          PHASE       AGE
demo        deploy-backup-batch-1576130711   BackupBatch    deploy-backup-batch   Succeeded   3m40s
```

We can see from the above output that the backup session has succeeded. Now, we are going to verify whether the backed up data has been stored in the backend.

**Verify Backup:**

Once a backup is complete, Stash will update the respective `Repository` crd to reflect the backup. Check that the repository `gcs-repo` has been updated by the following command,

```console
$ kubectl get repository -n demo gcs-repo
kubectl get repository -n demo gcs-repo
NAME       INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
gcs-repo   true               3                3m24s                    32m
```

Now, if we navigate to the GCS bucket, we are going to see backed up data has been stored in `/demo/data/<target-kind>/<target-name>` directory.
> Stash forms the directory by concatenating the `spec.backend.gcs.prefix` field of Repository crd and `<target-kind>/<target-name>`.

<figure align="center">
  <img alt="Backup data in GCS Bucket" src="/docs/images/guides/latest/backup-multiple-target/workloads.png">
  <figcaption align="center">Fig: Backup data in GCS Bucket</figcaption>
</figure>

> **Note:** Stash keeps all the backed up data encrypted. So, data in the backend will not make any sense until they are decrypted.

## Cleaning Up

To clean up the Kubernetes resources created by this tutorial, run:

```console
kubectl delete -n demo backupbatch deploy-backup-batch
kubectl delete -n demo deployment deploy-stash-demo
kubectl delete -n demo daemonset dmn-stash-demo
kubectl delete -n demo statefulset sts-stash-demo
kubectl delete -n demo repository gcs-repo
kubectl delete -n demo pvc --all
kubectl delete service -n demo headless
```