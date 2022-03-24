---
title: Managed Backup | Stash
description: An step by step guide on how you can manage your backup accross namespaces.
menu:
  docs_{{ .version }}:
    identifier: auto-backup-managed-backup
    name: Managed Backup
    parent: auto-backup
    weight: 50
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: guides
---

{{< notice type="warning" message="This is an Enterprise-only feature. Please install [Stash Enterprise Edition](/docs/setup/install/enterprise.md) to try this feature." >}}

# Managed Backup Across Namespaces

This tutorial will show you how to manage cross-namespace backup for Kubernetes workloads using the Auto-Backup feature of Stash. We are going to keep our StorageSecret in the `demo` namespace and take backup of your workloads from `ns1`, `ns2`, and `ns3` namespaces. 


## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one using [kind](https://kind.sigs.k8s.io/docs/user/quick-start/).

- Install `Stash` in your cluster following the steps [here](/docs/setup/README.md).

- You should be familiar with the following Stash concepts:
  - [Repository](/docs/concepts/crds/repository.md)
  - [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md)
  - [BackupBlueprint](/docs/concepts/crds/backupblueprint.md)
  - [BackupSession](/docs/concepts/crds/backupsession.md)

To demonstrate the cross-namespace backup management, we are going to create 4 namespaces called `demo`, `ns1`, `ns2`, and `ns3`  throughout this tutorial.

```bash
$ kubectl create namespace demo
namespace/demo created
$ kubectl create namespace ns1
namespace/ns1 created
$ kubectl create namespace ns2
namespace/ns2 created
$ kubectl create namespace ns3
namespace/ns3 created
```

## Prepare Backup Blueprint

We are going to use [GCS Backend](/docs/guides/backends/gcs.md) to store the backed up data. You can use any supported backend you prefer. You just have to configure StorageSecret and the` spec.backend` section of `BackupBlueprint` to match your backend. To learn which backends are supported by Stash and how to configure them, please visit [here](/docs/guides/backends/overview.md).

> For GCS backend, if the bucket does not exist, Stash needs `Storage Object Admin` role permissions to create the bucket. For more details, please check the following [guide](/docs/guides/backends/gcs.md).

**Create Storage Secret:**

At first, let's create a Storage Secret for the GCS backend in `demo` namespace,

```bash
$ echo -n 'changeit' > RESTIC_PASSWORD
$ echo -n '<your-project-id>' > GOOGLE_PROJECT_ID
$ mv downloaded-sa-json.key GOOGLE_SERVICE_ACCOUNT_JSON_KEY
$ kubectl create secret generic -n demo gcs-secret \
    --from-file=./RESTIC_PASSWORD \
    --from-file=./GOOGLE_PROJECT_ID \
    --from-file=./GOOGLE_SERVICE_ACCOUNT_JSON_KEY
secret/gcs-secret created
```

**Create BackupBlueprint:**

Now, we have to create a `BackupBlueprint` crd with a blueprint for the `Repository` and `BackupConfiguration` object. To keep our repository in an isolated namespace `demo` we have to put that into `spec.repoNamespace` section.

Below is the YAML of the `BackupBlueprint` object that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupBlueprint
metadata:
  name: managed-backup-blueprint
spec:
  # ============== Blueprint for Repository ==========================
  repoNamespace: demo
  backend:
    gcs:
      bucket: stash-testing
      prefix: stash-auto-backup/${TARGET_NAMESPACE}/${TARGET_KIND}/${TARGET_NAME}
    storageSecretName: gcs-secret
  # ============== Blueprint for BackupConfiguration =================
  schedule: "*/5 * * * *"
  retentionPolicy:
    name: 'keep-last-5'
    keepLast: 5
    prune: true
```
0
Note that if we keep the `spec.repoNamespace` field empty, the repository will be created in the workload's namespace by default. 

Let's create the `BackupBlueprint` that we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/auto-backup/managed-backup/examples/backupblueprint.yaml
backupblueprint.stash.appscode.com/managed-backup-blueprint created
```

Now, automatic backup is configured for Kubernetes workloads (`Deployment`, `StatefulSet`, `DaemonSet` etc.). We just need to add some annotations to the targeted workload from different or same namespaces to enable periodic backup.


## Backup Deployment from `ns1` namespace

Now, we are going to backup a Deployment from the `ns1` namespace using the blueprint we have configured earlier. We are going to mount two ConfigMap volume in two different directories of the Deployment. Then, we will backup those directories using Auto-backup.

**Create Deployment:**

Below is the YAML of the Deployment and respective ConfigMaps that we are going to create,

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: stash-sample-data-1
  namespace: ns1
data:
  file1.txt: "Data from ConfigMap 'stash-sample-data-1'"
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: stash-sample-data-2
  namespace: ns1
data:
  file2.txt: "Data from ConfigMap 'stash-sample-data-2'"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: stash-demo
  name: stash-demo
  namespace: ns1
  annotations:
    stash.appscode.com/backup-blueprint: managed-backup-blueprint
    stash.appscode.com/target-paths: "/source/data-1,/source/data-2"
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

Notice the `metadata.annotations` field. We have specified the automatic backup specific annotations to backup `/source/data-1` and `/source/data-2` directories of the `source-data-1` and `source-data-2` volumes respectively. We have also specified to use `managed-backup-blueprint` BackupBlueprint for creating `Repository` and `BackupConfiguration` for this Deployment. BackupBlueprint is a non-namespaced resource, so we just need to specify the name of the blueprint.

Let's create the above Deployment,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/auto-backup/managed-backup/examples/deployment_ns1.yaml
configmap/stash-sample-data-1 created
configmap/stash-sample-data-2 created
deployment.apps/stash-demo created
```

If everything goes well, Stash will create a `BackupConfiguration` in the `ns1` namespace and `Repository` in the `demo` namespace.

**Verify Repository:**

Verify that the Repository has been created in `demo` successfully by the following command,

```bash
$ kubectl get repository -n demo
NAME                     INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
demo-deploy-stash-demo                                                                29s
```

Let's view the yaml of the Repository,

```bash
$ kubectl get repository -n demo demo-deploy-stash-demo -o yaml
```

```yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: demo-deploy-stash-demo
  namespace: demo
  ...
spec:
  backend:
    gcs:
      bucket: stash-testing
      prefix: stash-auto-backup/ns1/deployment/stash-demo
    storageSecretName: gcs-secret
  usagePolicy:
    allowedNamespaces:
      from: Selector
      selector:
        matchLabels:
          kubernetes.io/metadata.name: ns1
```

In the `spec.backend.gcs.prefix` field, we can see that the variables `${TARGET_NAMESPACE}`, `${TARGET_KIND}` and `${TARGET_NAME}` has been replaced by `ns1`, `deployment` and `stash-demo` respectively.

**Verify BackupConfiguratoin:**

If everything goes well, Stash should create a `BackupConfiguration` for our Deployment and the phase of that `BackupConfiguration` should be `Ready`. Verify that the `BackupConfiguration` has been created by the following command,

```bash
$ kubectl get backupconfiguration -n ns1
NAME                TASK   SCHEDULE      PAUSED   PHASE   AGE
deploy-stash-demo          */5 * * * *            Ready   41s
```

Let's check the YAML of this `BackupConfiguration`,

```bash
$ kubectl get backupconfiguration -n ns1 deploy-stash-demo -o yaml
```

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: deploy-stash-demo
  namespace: ns1
  ...
spec:
  driver: Restic
  repository:
    name: demo-deploy-stash-demo
    namespace: demo
  retentionPolicy:
    keepLast: 5
    name: keep-last-5
    prune: true
  runtimeSettings: {}
  schedule: '*/5 * * * *'
  target:
    paths:
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

Notice that the `spec.repository` is pointing to the `demo-deploy-stash-demo` Repository of the `demo` namespace. Also, notice that the `spec.target.paths` and `spec.target.volumeMounts` fields have been populated with the information we had provided as annotation of the Deployment.

**Wait for BackupSession:**

Now, wait for the next backup schedule. Run the following command to watch `BackupSession` crd:

```bash
$ kubectl get backupsession -n ns1 -w

NAME                           INVOKER-TYPE          INVOKER-NAME        PHASE     DURATION   AGE
deploy-stash-demo-1648116301   BackupConfiguration   deploy-stash-demo   Running              27s
deploy-stash-demo-1648116301   BackupConfiguration   deploy-stash-demo   Running              37s
deploy-stash-demo-1648116301   BackupConfiguration   deploy-stash-demo   Succeeded   38s        37s
```

>Note: Respective CronJob creates `BackupSession` crd with the following label `stash.appscode.com/backup-configuration=<BackupConfiguration crd name>`. We can use this label to watch only the `BackupSession` of our desired `BackupConfiguration`.

## Backup StatefulSet from `ns2` namespace

Now, we are going to backup a StatefulSet from the `ns2` namespace with the same blueprint we have used to backup Deployment in the previous section.

**Create StatefulSet:**

We are going to create a StatefulSet with 3 replicas. StatefulSet is configured to generate sample data in each replica.

Below is the YAML of the StatefulSet that we are going to create,

```yaml
apiVersion: v1
kind: Service
metadata:
  name: headless
  namespace: ns2
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
  namespace: ns2
  labels:
    app: stash-demo
  annotations:
    stash.appscode.com/backup-blueprint: managed-backup-blueprint
    stash.appscode.com/target-paths: "/source/data-1,/source/data-2"
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

Let's create the StatefulSet we have created above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/auto-backup/managed-backup/examples/statefulset_ns2.yaml
service/headless created
statefulset.apps/sts-demo created
```

**Verify Repository:**

Verify that a Repository has been created for this StatefulSet using the following command,

```bash
$ kubectl get repository -n demo
NAME                     INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
demo-deploy-stash-demo                                                                35m
demo-sts-sts-demo                                                                     48s
```

Here, `demo-sts-sts-demo` Repository has been created for our `sts-demo` StatefulSet.

Let's view the YAML of the Repository,

```bash
$ kubectl get repository -n demo demo-sts-sts-demo -o yaml
```

```yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: demo-sts-sts-demo
  namespace: demo
  ...
spec:
  backend:
    gcs:
      bucket: stash-testing
      prefix: stash-auto-backup/ns2/statefulset/sts-demo
    storageSecretName: gcs-secret
  usagePolicy:
    allowedNamespaces:
      from: Selector
      selector:
        matchLabels:
          kubernetes.io/metadata.name: ns2
```

Notice that the variables of the `spec.backend.gcs.prefix` field of `BackupBlueprint` is now replaced with `demo`, `statefulset` and `sts-demo` respectively.

**Verify BackupConfiguratoin:**

Verify that a `BackupConfiguration` has been created in the `ns2` namespace and it is in the `Ready` Phase for this StatefulSet using the following command,

```bash
$ kubectl get backupconfiguration -n ns2
NAME           TASK   SCHEDULE      PAUSED   PHASE   AGE
sts-sts-demo          */5 * * * *            Ready   3m49s
```

Here, `sts-sts-demo` has been created for the StatefulSet `sts-demo`. You can check the YAML of this `BackupConfiguration` to see that it is pointing to the repository of the `demo` namespace.

```bash
$ kubectl get backupconfiguration -n ns2 sts-sts-demo -o yaml
```

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: statefulset-sts-demo
  namespace: demo
  ...
spec:
  driver: Restic
  repository:
    name: demo-sts-sts-demo
    namespace: demo
  retentionPolicy:
    keepLast: 5
    name: keep-last-5
    prune: true
  runtimeSettings: {}
  schedule: '*/5 * * * *'
  target:
    paths:
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

Now, wait for the next backup schedule. Watch the `BackupSession` of the BackupConfiguration `statefulset-sts-demo` using the following command to see if it's succeeded,

```bash
$ kubectl get backupsession -n ns2 -w

NAME                      INVOKER-TYPE          INVOKER-NAME   PHASE       DURATION   AGE
sts-sts-demo-1648118701   BackupConfiguration   sts-sts-demo   Running                35s
sts-sts-demo-1648118701   BackupConfiguration   sts-sts-demo   Running                95s
sts-sts-demo-1648118701   BackupConfiguration   sts-sts-demo   Succeeded   1m36s      95s
```

## Backup DaemonSet from `ns3` namespace

Now, we are going to use the same blueprint to backup a DaemonSet from the `ns3` namespace. We are going to mount a ConfigMap in `/etc/config` directory. Then, we will backup this directory using automatic backup.

**Create DaemonSet:**

Below is the YAML of the DaemonSet that we are going to create,

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-daemon-config
  namespace: ns3
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
  namespace: ns3
  annotations:
    stash.appscode.com/backup-blueprint: managed-backup-blueprint
    stash.appscode.com/target-paths: "/etc/config"
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

Notice the `metadata.annotations` field. We have specified automatic backup-specific annotations to backup our desired file path.

Let's create the DaemonSet we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/auto-backup/managed-backup/examples/daemonset_ns3.yaml
configmap/my-daemon-config created
daemonset.apps/dmn-demo created
```

**Verify Repository:**

Verify that a `Repository` has been created for this DaemonSet  in `demo` namespace using the following command,

```bash
$ kubectl get repository -n demo
NAME                     INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
demo-deploy-stash-demo                                                                61m
demo-ds-dmn-demo                                                                      94s
demo-sts-sts-demo                                                                     26m
```

Here, `demo-ds-dmn-demo` Repository has been created for our `dmn-demo` DaemonSet.

Let's view the YAML of the Repository,

```bash
$ kubectl get repository -n demo demo-ds-dmn-demo -o yaml
```

```yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: demo-ds-dmn-demo
  namespace: demo
  ...
spec:
  backend:
    gcs:
      bucket: stash-testing
      prefix: stash-auto-backup/ns3/daemonset/dmn-demo
    storageSecretName: gcs-secret
  usagePolicy:
    allowedNamespaces:
      from: Selector
      selector:
        matchLabels:
          kubernetes.io/metadata.name: ns3
```

**Verify BackupConfiguratoin:**
If everything goes well, Stash should create a `BackupConfiguration` in the `ns3` namespace for our DaemonSet and the phase of that `BackupConfiguration` should be `Ready`. Verify the `BackupConfiguration` crd by the following command,

```bash
$ kubectl get backupconfiguration -n ns3
NAME          TASK   SCHEDULE      PAUSED   PHASE   AGE
ds-dmn-demo          */5 * * * *            Ready   8m36s
```

Here, `ds-dmn-demo` backupconfiguration has been created for the DaemonSet `dmn-demo`. You can check the YAML of this `BackupConfiguration` to see that the target field is pointing to this DaemonSet.

```bash
$ kubectl get backupconfiguration -n ns3 ds-dmn-demo -o yaml
```

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: ds-dmn-demo
  namespace: ns3
  ...
spec:
  driver: Restic
  repository:
    name: demo-ds-dmn-demo
    namespace: demo
  retentionPolicy:
    keepLast: 5
    name: keep-last-5
    prune: true
  runtimeSettings: {}
  schedule: '*/5 * * * *'
  target:
    paths:
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

Notice that the `spec.repository` is pointing to the `demo-ds-dmn-demo` Repository of the `demo` namespace.

**Wait for BackupSession to be succeeded:**

Now, wait for the next backup schedule. Watch the `BackupSession` of the BackupConfiguration `ds-dmn-demo` using the following command,

```bash
$ kubectl get backupsession -n ns3 -w

NAME                     INVOKER-TYPE          INVOKER-NAME   PHASE       DURATION   AGE
ds-dmn-demo-1648121403   BackupConfiguration   ds-dmn-demo    Running                6s
ds-dmn-demo-1648121403   BackupConfiguration   ds-dmn-demo    Running                18s
ds-dmn-demo-1648121403   BackupConfiguration   ds-dmn-demo    Succeeded   30s        30s
```