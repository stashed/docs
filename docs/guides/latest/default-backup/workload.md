---
title: Default Backup Workload | Stash
description: An step by step guide on how to configure default backup for workloads.
menu:
  product_stash_0.8.3:
    identifier: default-backup-workload
    name: Default Backup for Workloads
    parent: default-backup
    weight: 20
product_name: stash
menu_name: product_stash_0.8.3
section_menu_id: guides
---

# Default Backup for Workloads

This tutorial will show you how to configure default backup for Kubernetes workloads. Here, we are going to show a demo on how we can backup Deployment, StatefulSet and DaemonSet using a common template.

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [Minikube](https://github.com/kubernetes/minikube).

- Install `Stash` in your cluster following the steps [here](/docs/setup/install.md).

- You should be familiar with the following Stash concepts:
  - [Repository](/docs/concepts/crds/repository.md)
  - [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md)
  - [BackupConfigurationTemplate](/docs/concepts/crds/backupconfiguration_template.md)
  - [BackupSession](/docs/concepts/crds/backupsession.md)

To keep things isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$ kubectl create namespace demo
namespace/demo created
```

## Prepare Template

We are going to use [GCS Backend](/docs/guides/latest/backends/gcs.md) to store the backed up data. You can use any supported backend you prefer. You have to configure Storage Secret and `spec.backend` section of `BackupConfigurationTemplate` to match your backend. To know which backed is supported by Stash and how to configure them, visit [here](/docs/guides/latest/backends/overview.md).

**Create Storage Secret:**

At first, let's create a Storage Secret for GCS backend,

```console
$ echo -n 'changeit' > RESTIC_PASSWORD
$ echo -n '<your-project-id>' > GOOGLE_PROJECT_ID
$ mv downloaded-sa-json.key > GOOGLE_SERVICE_ACCOUNT_JSON_KEY
$ kubectl create secret generic -n demo gcs-secret \
    --from-file=./RESTIC_PASSWORD \
    --from-file=./GOOGLE_PROJECT_ID \
    --from-file=./GOOGLE_SERVICE_ACCOUNT_JSON_KEY
secret/gcs-secret created
```

**Create BackupConfigurationTemplate:**

Now, we have to create a `BackupConfigurationTemplate` crd with template for `Repository` and `BackupConfiguration` object.

Below is the YAML of the `BackupConfigurationTemplate` object that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfigurationTemplate
metadata:
  name: workload-backup-template
spec:
  # ============== Template for Repository ==========================
  backend:
    gcs:
      bucket: appscode-qa
      prefix: stash-backup/${TARGET_NAMESPACE}/${TARGET_KIND}/${TARGET_NAME}
    storageSecretName: gcs-secret
  # ============== Template for BackupConfiguration =================
  schedule: "*/5 * * * *"
  retentionPolicy:
    name: 'keep-last-5'
    keepLast: 5
    prune: true
```

Note that we have used some variables (format: `${<variable name>}`) in `backend.gcs.prefix` field. Stash will substitute these variables with valuse from respective target. Since the resolved prefix will be different for different workload, the backed up data will be stored in different directory inside the bucket. To know which variable you can use in this `prefix` field, please visit [here](/docs/concepts/crds/backupconfiguration_template.md#repository-template).

Let's create the `BackupConfigurationTemplate` that we have shown above,

```console
$ kubectl apply -f ./docs/examples/guides/latest/default-backup/backupconfiguration_template.yaml
backupconfigurationtemplate.stash.appscode.com/workload-backup-template created
```

Now, default backup is configured for Kubernetes workloads (`Deployment`, `StatefulSet`, `DaemonSet` etc.). We just have to add some annotations to the targeted workload.

**Required Annotations for Workloads:**

You have to add following 3 annotations to a targetted workload to enable backup for it:

1. Name of the BackupConfigurationTemplate object where a template for Repository and BackupConfiguration has been defined.

```yaml
stash.appscode.com/backup-template: <BackupConfigurationTemplate name>
```

2. List of directories that will be backed up. Use comma (`,`) to separate multiple directories. For example, `"/my/target/dir-1,/my/target/dir-2"`.

```yaml
stash.appscode.com/target-directories: "<directories to backup>"
```

3. List of Volumes and their MountPath where the target directories are located. Use `"<volumenName>:<mountPath>"` format to specify the volumes. Use comma (`,`) to specify multiple volumes and mount path. For example, `"vol-1:/mount/path-1,vol-2:/mount/path-2"`.

```yaml
stash.appscode.com/volume-mounts: "<volume>:<mountPath>"
```

## Backup Deployment

**Create Deployment:**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: stash-sample-data-1
  namespace: demo
data:
  file1.txt: "Data from ConfigMap 'stash-sample-data-1'"
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: stash-sample-data-2
  namespace: demo
data:
  file2.txt: "Data from ConfigMap 'stash-sample-data-2'"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: stash-demo
  name: stash-demo
  namespace: demo
  annotations:
    stash.appscode.com/backup-template: workload-backup-template
    stash.appscode.com/target-directories: "/source/data-1,/source/data-2"
    stash.appscode.com/volume-mounts: "source-data-1:/source/data-1,source-data-2:/source/data-2"
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
        - mountPath: /source/data-1
          name: source-data-1
        - mountPath: /source/data-2
          name: source-data-2
      restartPolicy: Always
      volumes:
      - name: source-data-1
        configMap:
          name: stash-sample-data-1
      - name: source-data-2
        configMap:
          name: stash-sample-data-2
```

```console
$ kubectl apply -f ./docs/examples/guides/latest/default-backup/deployment.yaml
configmap/stash-sample-data-1 created
configmap/stash-sample-data-2 created
deployment.apps/stash-demo created
```

**Verify Repository:**

```console
$ kubectl get repository -n demo
NAME                    INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
deployment-stash-demo                                                                9s
```

```console
kubectl get repository -n demo deployment-stash-demo -o yaml
```

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: Repository
metadata:
  creationTimestamp: "2019-06-26T06:16:21Z"
  finalizers:
  - stash
  generation: 1
  name: deployment-stash-demo
  namespace: demo
  resourceVersion: "8788"
  selfLink: /apis/stash.appscode.com/v1beta1/namespaces/demo/repositories/deployment-stash-demo
  uid: eab6a2b5-97d9-11e9-b72d-0800275860ad
spec:
  backend:
    gcs:
      bucket: appscode-qa
      prefix: stash-backup/demo/deployment/stash-demo
    storageSecretName: gcs-secret
```

**Verify BackupConfiguratoin:**

```console
$ kubectl get backupconfiguration -n demo
NAME                    TASK   SCHEDULE      PAUSED   AGE
deployment-stash-demo          */5 * * * *            19s
```

```console
kubectl get backupconfiguration -n demo deployment-stash-demo -o yaml
```

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  creationTimestamp: "2019-06-26T06:16:21Z"
  finalizers:
  - stash.appscode.com
  generation: 1
  name: deployment-stash-demo
  namespace: demo
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: false
    kind: Deployment
    name: stash-demo
    uid: eab310c4-97d9-11e9-b72d-0800275860ad
  resourceVersion: "8798"
  selfLink: /apis/stash.appscode.com/v1beta1/namespaces/demo/backupconfigurations/deployment-stash-demo
  uid: eac2328b-97d9-11e9-b72d-0800275860ad
spec:
  repository:
    name: deployment-stash-demo
  retentionPolicy:
    keepLast: 5
    name: keep-last-5
    prune: true
  runtimeSettings: {}
  schedule: '*/5 * * * *'
  target:
    directories:
    - /source/data-1
    - /source/data-2
    ref:
      apiVersion: apps/v1
      kind: Deployment
      name: stash-demo
    volumeMounts:
    - mountPath: /source/data-1
      name: source-data-1
    - mountPath: /source/data-2
      name: source-data-2
  task: {}
  tempDir: {}
```

**Wait for BackupSession:**

```console
$ watch -n 1 kubectl get backupsession -n demo -l=stash.appscode.com/backup-configuration=deployment-stash-demo
Every 1.0s: kubectl get backupsession -n demo                                   workstation: Wed Jun 26 12:20:31 2019

NAME                               BACKUPCONFIGURATION     PHASE       AGE
deployment-stash-demo-1561530008   deployment-stash-demo   Running     24s
deployment-stash-demo-1561530008   deployment-stash-demo   Succeeded   61s
```

**Verify Backup:**

```console
$ kubectl get repository -n demo deployment-stash-demo
NAME                    INTEGRITY   SIZE    SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
deployment-stash-demo   true        246 B   6                70s                      15m
```

<figure>Figure Here</figure>

## Backup StatefulSet

**Create StatefulSet:**

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
  name: sts-demo
  namespace: demo
  labels:
    app: stash-demo
  annotations:
    stash.appscode.com/backup-template: workload-backup-template
    stash.appscode.com/target-directories: "/source/data-1,/source/data-2"
    stash.appscode.com/volume-mounts: "source-data-1:/source/data-1,source-data-2:/source/data-2"
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
        command: ["/bin/sh", "-c"]
        args: ["touch /source/data-1/sample-file-1.txt && touch /source/data-2/sample-file-2.txt && sleep 3000"]
        imagePullPolicy: IfNotPresent
        volumeMounts:
        - mountPath: /source/data-1
          name: source-data-1
        - mountPath: /source/data-2
          name:  source-data-2
  volumeClaimTemplates:
  - metadata:
      name: source-data-1
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "standard"
      resources:
        requests:
          storage: 1Gi
  - metadata:
      name: source-data-2
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "standard"
      resources:
        requests:
          storage: 1Gi
```

```console
$ kubectl apply -f ./docs/examples/guides/latest/default-backup/statefulset.yaml
service/headless created
statefulset.apps/sts-demo created
```

**Verify Repository:**

```console
$ kubectl get repository -n demo
NAME                    INTEGRITY   SIZE    SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
deployment-stash-demo   true        410 B   10               14s                      39m
statefulset-sts-demo                                                                  31s
```

```console
kubectl get repository -n demo statefulset-sts-demo -o yaml
```

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: Repository
metadata:
  creationTimestamp: "2019-06-26T06:55:23Z"
  finalizers:
  - stash
  generation: 1
  name: statefulset-sts-demo
  namespace: demo
  resourceVersion: "13149"
  selfLink: /apis/stash.appscode.com/v1beta1/namespaces/demo/repositories/statefulset-sts-demo
  uid: 5e9d8f34-97df-11e9-b72d-0800275860ad
spec:
  backend:
    gcs:
      bucket: appscode-qa
      prefix: stash-backup/demo/statefulset/sts-demo
    storageSecretName: gcs-secret
```

**Verify BackupConfiguratoin:**

```console
$ kubectl get backupconfiguration -n demo
NAME                    TASK   SCHEDULE      PAUSED   AGE
deployment-stash-demo          */5 * * * *            40m
statefulset-sts-demo           */5 * * * *            105s
```

```console
kubectl get backupconfiguration -n demo statefulset-sts-demo -o yaml
```

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  creationTimestamp: "2019-06-26T06:55:23Z"
  finalizers:
  - stash.appscode.com
  generation: 1
  name: statefulset-sts-demo
  namespace: demo
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: false
    kind: StatefulSet
    name: sts-demo
    uid: 5e9671e3-97df-11e9-b72d-0800275860ad
  resourceVersion: "13152"
  selfLink: /apis/stash.appscode.com/v1beta1/namespaces/demo/backupconfigurations/statefulset-sts-demo
  uid: 5ea175de-97df-11e9-b72d-0800275860ad
spec:
  repository:
    name: statefulset-sts-demo
  retentionPolicy:
    keepLast: 5
    name: keep-last-5
    prune: true
  runtimeSettings: {}
  schedule: '*/5 * * * *'
  target:
    directories:
    - /source/data-1
    - /source/data-2
    ref:
      apiVersion: apps/v1
      kind: StatefulSet
      name: sts-demo
    volumeMounts:
    - mountPath: /source/data-1
      name: source-data-1
    - mountPath: /source/data-2
      name: source-data-2
  task: {}
  tempDir: {}
```

**Wait for BackupSession:**

```console
$ watch -n 1 kubectl get backupsession -n demo -l=stash.appscode.com/backup-configuration=statefulset-sts-demo
Every 1.0s: kubectl get backupsession -n demo -l=stash.appscode.com/backup-...  workstation: Wed Jun 26 13:01:22 2019

NAME                              BACKUPCONFIGURATION    PHASE     AGE
statefulset-sts-demo-1561532403   statefulset-sts-demo   Running   79s
statefulset-sts-demo-1561532403   statefulset-sts-demo   Succeeded   2m21s
```

**Verify Backup:**

```console
$ kubectl get repository -n demo statefulset-sts-demo
NAME                   INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
statefulset-sts-demo   true        0 B    6                32s                      7m29s
```

<figure>Figure Here</figure>

## Backup DaemonSet

**Create DaemonSet:**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-daemon-config
  namespace: demo
data:
  config-file-1.txt: "This is first config file"
  config-file-2.txt: "This is second config file"
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: stash-demo
  name: dmn-demo
  namespace: demo
  annotations:
    stash.appscode.com/backup-template: workload-backup-template
    stash.appscode.com/target-directories: "/etc/config"
    stash.appscode.com/volume-mounts: "dmn-config:/etc/config"
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
      - name: busybox
        args:
        - sleep
        - "3600"
        image: busybox
        imagePullPolicy: IfNotPresent
        volumeMounts:
        - mountPath: /etc/config
          name: dmn-config
      restartPolicy: Always
      volumes:
      - name: dmn-config
        configMap:
          name: my-daemon-config
```

```console
$ kubectl apply -f ./docs/examples/guides/latest/default-backup/daemonset.yaml
configmap/my-daemon-config created
daemonset.apps/dmn-demo created
```

**Verify Repository:**

```console
$ kubectl get repository -n demo
NAME                    INTEGRITY   SIZE    SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
daemonset-dmn-demo                                                                    28s
deployment-stash-demo   true        410 B   10               6m3s                     70m
statefulset-sts-demo    true        0 B     26               6m6s                     31m
```

```console
kubectl get repository -n demo daemonset-dmn-demo -o yaml
```

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: Repository
metadata:
  creationTimestamp: "2019-06-26T07:26:16Z"
  finalizers:
  - stash
  generation: 1
  name: daemonset-dmn-demo
  namespace: demo
  resourceVersion: "17351"
  selfLink: /apis/stash.appscode.com/v1beta1/namespaces/demo/repositories/daemonset-dmn-demo
  uid: af7d22f6-97e3-11e9-b72d-0800275860ad
spec:
  backend:
    gcs:
      bucket: appscode-qa
      prefix: stash-backup/demo/daemonset/dmn-demo
    storageSecretName: gcs-secret
```

**Verify BackupConfiguratoin:**

```console
$  kubectl get backupconfiguration -n demo
NAME                    TASK   SCHEDULE      PAUSED   AGE
daemonset-dmn-demo             */5 * * * *            90s
deployment-stash-demo          */5 * * * *            71m
statefulset-sts-demo           */5 * * * *            32m
```

```console
kubectl get backupconfiguration -n demo daemonset-dmn-demo -o yaml
```

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  creationTimestamp: "2019-06-26T07:26:16Z"
  finalizers:
  - stash.appscode.com
  generation: 1
  name: daemonset-dmn-demo
  namespace: demo
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: false
    kind: DaemonSet
    name: dmn-demo
    uid: af74eb56-97e3-11e9-b72d-0800275860ad
  resourceVersion: "17355"
  selfLink: /apis/stash.appscode.com/v1beta1/namespaces/demo/backupconfigurations/daemonset-dmn-demo
  uid: af85d9b9-97e3-11e9-b72d-0800275860ad
spec:
  repository:
    name: daemonset-dmn-demo
  retentionPolicy:
    keepLast: 5
    name: keep-last-5
    prune: true
  runtimeSettings: {}
  schedule: '*/5 * * * *'
  target:
    directories:
    - /etc/config
    ref:
      apiVersion: apps/v1
      kind: DaemonSet
      name: dmn-demo
    volumeMounts:
    - mountPath: /etc/config
      name: dmn-config
  task: {}
  tempDir: {}
```

**Wait for BackupSession:**

```console
$ watch -n 1 kubectl get backupsession -n demo -l=stash.appscode.com/backup-configuration=daemonset-dmn-demo

Every 1.0s: kubectl get backupsession -n demo -l=stash.appscode.com/backup-...  workstation: Wed Jun 26 13:30:14 2019

NAME                            BACKUPCONFIGURATION   PHASE     AGE
daemonset-dmn-demo-1561534208   daemonset-dmn-demo    Running   6s
daemonset-dmn-demo-1561534208   daemonset-dmn-demo    Succeeded   45s
```

**Verify Backup:**

```console
$ kubectl get repository -n demo daemonset-dmn-demo
NAME                 INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
daemonset-dmn-demo   true        51 B   1                5s                       4m27s
```

<figure>Figure Here</figure>

## Cleanup

```console
kubectl delete -n demo deployment/stash-demo
kubectl delete -n demo statefulset/sts-demo
kubectl delete -n demo daemonset/dmn-demo

kubectl delete -n demo repository --all
kubectl delete -n demo secret/gcs-secret
```
