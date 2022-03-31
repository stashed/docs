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

# Cross-namespaces Auto-Backup

This tutorial we will firstly demonstrate how as a cluster admin you can configure backup policy with your credentials in an isolated namespace using Stash. After that, we will demonstrate how different users of your cluster can take backup from their respective namespaces using your pre-configured backup policy.

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one using [kind](https://kind.sigs.k8s.io/docs/user/quick-start/).

- Install `Stash` in your cluster following the steps [here](/docs/setup/README.md).

- You should be familiar with the following Stash concepts:
  - [Repository](/docs/concepts/crds/repository.md)
  - [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md)
  - [BackupBlueprint](/docs/concepts/crds/backupblueprint.md)
  - [BackupSession](/docs/concepts/crds/backupsession.md)


## Configure Backup Policy

In this section we will assume that, you will configure backup policy with your credentials from `backup` namespace. 

Let's create the `backup` namespace,
```bash
$ kubectl create namespace backup
namespace/backup created
```

We will use [GCS Backend](/docs/guides/backends/gcs.md) to store our backed-up data. You can use any of the supported backend you prefer. You just have to configure StorageSecret and the `spec.backend` section of `BackupBlueprint` to match your backend. To learn which backends are supported by Stash and how to configure them, please visit [here](/docs/guides/backends/overview.md).

> For GCS backend, if the bucket does not exist, Stash needs `Storage Object Admin` role permissions to create the bucket. For more details, please check the following [guide](/docs/guides/backends/gcs.md).

**Create Storage Secret:**

At first, let's create a Storage Secret for the GCS backend in `backup` namespace,

```bash
$ echo -n 'changeit' > RESTIC_PASSWORD
$ echo -n '<your-project-id>' > GOOGLE_PROJECT_ID
$ mv downloaded-sa-key.json GOOGLE_SERVICE_ACCOUNT_JSON_KEY
$ kubectl create secret generic -n backup gcs-secret \
    --from-file=./RESTIC_PASSWORD \
    --from-file=./GOOGLE_PROJECT_ID \
    --from-file=./GOOGLE_SERVICE_ACCOUNT_JSON_KEY
secret/gcs-secret created
```
Note, if you want to keep your Repositories in `backup` namespace you need to create Secret in the same namespace also.

**Create BackupBlueprint:**

Now, we have to create a `BackupBlueprint` object with a blueprint for the `Repository` and `BackupConfiguration` object. To keep the Repositories in an isolated namespace `backup`, we need to put that namespace name into `spec.repoNamespace` section. The Repositories will authenticate users for the backend using the secret mentioned in the `spec.backend.storageSecretName` section.

Below is the YAML of the `BackupBlueprint` object that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupBlueprint
metadata:
  name: managed-backup-blueprint
spec:
  # ============== Blueprint for Repository ==========================
  repoNamespace: backup
  backend:
    gcs:
      bucket: stash-testing
      prefix: stash-auto-backup/${TARGET_NAMESPACE}/${TARGET_KIND}/${TARGET_NAME}
    storageSecretName: gcs-secret
  usagePolicy:
    allowedNamespaces:
      from: All
  # ============== Blueprint for BackupConfiguration =================
  schedule: "*/5 * * * *"
  retentionPolicy:
    name: 'keep-last-5'
    keepLast: 5
    prune: true
```

Note that, the value in the `spec.usagePolicy` section indicates that the Repositories created from this blueprint are allowed to be referred from all namespace of our cluster. If you don't specify any usagePolicy in your spec, it will only allow the Repositories to be referred from the target workload's namespace. 

Here is an example of `spec.usagePolicy` that limits referencing the Repositories only from the target workload's namespace,
```yaml
spec: 
  usagePolicy:
    allowedNamespaces:
      from: Same
```

Here is an example of `spec.usagePolicy` that allows referencing the Repositories from the `prod` and `staging` namespaces,
```yaml
spec:
  usagePolicy:
    allowedNamespaces:
      from: Selector
      selector:
        matchExpressions:
        - key: "kubernetes.io/metadata.name"
          operator: In
          values: ["prod","staging"]
```

Let's create the `BackupBlueprint` that we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/auto-backup/managed-backup/examples/backupblueprint.yaml
backupblueprint.stash.appscode.com/managed-backup-blueprint created
```

Now, automatic backup is configured for Kubernetes workloads (`Deployment`, `StatefulSet`, `DaemonSet` etc.). We just need to add some annotations into our target workloads for enabling periodic backup.

## Backup Using the Pre-configured Backup Policy

In this part of this tutorial we will assume that, different users of your cluster named user1, user2, and user3 will take backup from `user1`, `user2`, and `user3` namespaces respectively using the pre-configured backup policy.

### Backup workload from `user1` namespace

Now, we are going to backup specific directories of a Deployment from the `user1` namespace using the blueprint we have configured earlier. We will mount two ConfigMap volumes in two different directories of the Deployment. Then, we will backup those directories using Auto-backup.

Let's create the `user1` namespace,
```bash
$ kubectl create namespace user1
namespace/user1 created
```

**Create Deployment:**

Below is the YAML of the Deployment and respective ConfigMaps that we are going to create in the `user1` namespace,

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: stash-sample-data-1
  namespace: user1
data:
  file1.txt: "Data from ConfigMap 'stash-sample-data-1'"
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: stash-sample-data-2
  namespace: user1
data:
  file2.txt: "Data from ConfigMap 'stash-sample-data-2'"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: stash-demo
  name: stash-demo
  namespace: user1
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

Notice the `metadata.annotations` field. We have specified the automatic backup specific annotations to backup `/source/data-1` and `/source/data-2` directories of the `source-data-1` and `source-data-2` volumes respectively. We have also specified to use `managed-backup-blueprint` BackupBlueprint for creating `Repository` and `BackupConfiguration` for this Deployment. As the BackupBlueprint is a non-namespaced resource, we just need to specify the name of the BackupBlueprint.

Let's create the above Deployment,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/auto-backup/managed-backup/examples/deployment_user1.yaml
configmap/stash-sample-data-1 created
configmap/stash-sample-data-2 created
deployment.apps/stash-demo created
```

Stash will create a `BackupConfiguration` in the `user1` namespace and `Repository` in the `backup` namespace for this Deployment.

**Verify Repository:**

Verify that the Repository has been created in `backup` namespace by the following command,

```bash
$ kubectl get repository -n backup
NAME                      INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
user1-deploy-stash-demo                                                                6s
```

Let's view the yaml of the Repository,

```bash
$ kubectl get repository -n backup user1-deploy-stash-demo -o yaml
```

```yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: user1-deploy-stash-demo
  namespace: backup
  ...
spec:
  backend:
    gcs:
      bucket: stash-testing
      prefix: stash-auto-backup/user1/deployment/stash-demo
    storageSecretName: gcs-secret
  usagePolicy:
    allowedNamespaces:
      from: All
```

In the `spec.backend.gcs.prefix` field, we can see that the variables `${TARGET_NAMESPACE}`, `${TARGET_KIND}` and `${TARGET_NAME}` has been replaced by `user1`, `deployment` and `stash-demo` respectively.

**Verify BackupConfiguration:**

If everything goes well, Stash should create a `BackupConfiguration` for our Deployment in `user1` namespace and the phase of that `BackupConfiguration` should be `Ready`. Verify that the `BackupConfiguration` has been created by the following command,

```bash
$ kubectl get backupconfiguration -n user1
NAME                TASK   SCHEDULE      PAUSED   PHASE   AGE
deploy-stash-demo          */5 * * * *            Ready   3m22s
```

Let's check the YAML of this `BackupConfiguration`,

```bash
$ kubectl get backupconfiguration -n user1 deploy-stash-demo -o yaml
```

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: deploy-stash-demo
  namespace: user1
  ...
spec:
  driver: Restic
  repository:
    name: user1-deploy-stash-demo
    namespace: backup
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

Notice that the `spec.repository` is pointing to the `user1-deploy-stash-demo` Repository of the `backup` namespace. Also, notice that the `spec.target.paths` and `spec.target.volumeMounts` fields have been populated with the information we had provided as annotation of the Deployment.

**Wait for BackupSession to be succeded:**

Now, wait for the next backup schedule to invoke a BackupSession and to go the session into Succeeded phase. Run the following command to watch the `BackupSession` object in user1 namespace:

```bash
$ kubectl get backupsession -n user1 -w
NAME                           INVOKER-TYPE          INVOKER-NAME        PHASE       DURATION   AGE
deploy-stash-demo-1648718400   BackupConfiguration   deploy-stash-demo   Running                5s
deploy-stash-demo-1648718400   BackupConfiguration   deploy-stash-demo   Running                33s
deploy-stash-demo-1648718400   BackupConfiguration   deploy-stash-demo   Succeeded   39s        2m45s
```

>Note: Respective CronJob creates `BackupSession` object with the following label `stash.appscode.com/backup-configuration=<BackupConfiguration crd name>`. We can use this label to watch only the `BackupSession` of our desired `BackupConfiguration`.

### Backup workload from `user2` namespace

Now, we will backup a Deployment from the `user2` namespace using the blueprint we have configured earlier. We will mount two ConfigMap volume in two different directories of the Deployment as before. 

Let's create the `user2` namespace,
```bash
$ kubectl create namespace user2
namespace/user2 created
```

**Create Deployment:**

Below is the YAML of the Deployment and respective ConfigMaps that we are going to create in the `user2` namespace,

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: stash-sample-data-1
  namespace: user2
data:
  file1.txt: "Data from ConfigMap 'stash-sample-data-1'"
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: stash-sample-data-2
  namespace: user2
data:
  file2.txt: "Data from ConfigMap 'stash-sample-data-2'"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: stash-demo
  name: stash-demo
  namespace: user2
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

Let's create the above Deployment,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/auto-backup/managed-backup/examples/deployment_user2.yaml
configmap/stash-sample-data-1 created
configmap/stash-sample-data-2 created
deployment.apps/stash-demo created
```

Stash will create a `BackupConfiguration` in the `user2` namespace and `Repository` in the `backup` namespace for this Deployment.

**Verify Repository:**

Verify that the Repository for this Deployment has been created in `backup` namespace by the following command,

```bash
$ kubectl get repository -n backup
NAME                      INTEGRITY   SIZE        SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
user1-deploy-stash-demo   true        4.154 KiB   10               4m34s                    27m
user2-deploy-stash-demo                                                                     43s
```

Let's view the yaml of the Repository,

```bash
$ kubectl get repository -n backup user2-deploy-stash-demo -o yaml
```

```yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: user2-deploy-stash-demo
  namespace: backup
  ...
spec:
  backend:
    gcs:
      bucket: stash-testing
      prefix: stash-auto-backup/user2/deployment/stash-demo
    storageSecretName: gcs-secret
  usagePolicy:
    allowedNamespaces:
      from: All
```

In the `spec.backend.gcs.prefix` field, we can see that the variables `${TARGET_NAMESPACE}`, `${TARGET_KIND}` and `${TARGET_NAME}` has been replaced by `user2`, `deployment` and `stash-demo` respectively.

**Verify BackupConfiguration:**

If everything goes well, Stash should create a `BackupConfiguration` for our Deployment in `user2` namespace and the phase of that `BackupConfiguration` should be `Ready`. Verify that the `BackupConfiguration` has been created by the following command,

```bash
$ kubectl get backupconfiguration -n user2
NAME                TASK   SCHEDULE      PAUSED   PHASE   AGE
deploy-stash-demo          */5 * * * *            Ready   3m15s
```

Let's check the YAML of this `BackupConfiguration`,

```bash
$ kubectl get backupconfiguration -n user2 deploy-stash-demo -o yaml
```

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: deploy-stash-demo
  namespace: user2
  ...
spec:
  driver: Restic
  repository:
    name: user2-deploy-stash-demo
    namespace: backup
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

Notice that the `spec.repository` is pointing to the `user2-deploy-stash-demo` Repository of the `backup` namespace. Also, notice that the `spec.target.paths` and `spec.target.volumeMounts` fields have been populated with the information we had provided as annotation of the Deployment.

**Wait for BackupSession to be succeded:**

Now, wait for the next backup schedule to invoke a BackupSession and to go the session into Succeeded Phase. Run the following command to watch the `BackupSession` object in user2 namespace:

```bash
$ kubectl get backupsession -n user2 -w
NAME                           INVOKER-TYPE          INVOKER-NAME        PHASE       DURATION   AGE
deploy-stash-demo-1648719900   BackupConfiguration   deploy-stash-demo   Running                3s
deploy-stash-demo-1648719900   BackupConfiguration   deploy-stash-demo   Running                28s
deploy-stash-demo-1648719900   BackupConfiguration   deploy-stash-demo   Succeeded   36s        36s
```

### Backup workload from `user3` namespace

Now, we are going to backup a Deployment from the `user3` namespace using the blueprint. Similarly, we will mount two ConfigMap volume in two different directories of the Deployment. Then, we will backup those directories using Auto-backup.

Let's create the `user3` namespace,
```bash
$ kubectl create namespace user3
namespace/user3 created
```

**Create Deployment:**

Below is the YAML of the Deployment and respective ConfigMaps that we are going to create in the `user3` namespace,

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: stash-sample-data-1
  namespace: user3
data:
  file1.txt: "Data from ConfigMap 'stash-sample-data-1'"
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: stash-sample-data-2
  namespace: user3
data:
  file2.txt: "Data from ConfigMap 'stash-sample-data-2'"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: stash-demo
  name: stash-demo
  namespace: user3
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

Let's create the above Deployment,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/auto-backup/managed-backup/examples/deployment_user3.yaml
configmap/stash-sample-data-1 created
configmap/stash-sample-data-2 created
deployment.apps/stash-demo created
```

If everything goes well, Stash will create a `BackupConfiguration` in the `user3` namespace and `Repository` in the `backup` namespace for this Deployment.

**Verify Repository:**

Verify that the Repository has been created in `backup` namespace successfully by the following command,

```bash
$ kubectl get repository -n backup
NAME                      INTEGRITY   SIZE        SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
user1-deploy-stash-demo   true        4.154 KiB   10               5m14s                    42m
user2-deploy-stash-demo   true        4.156 KiB   6                5m14s                    16m
user3-deploy-stash-demo                                                                     85s
```

Let's view the yaml of the Repository,

```bash
$ kubectl get repository -n backup user3-deploy-stash-demo -o yaml
```

```yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: user3-deploy-stash-demo
  namespace: backup
  ...
spec:
  backend:
    gcs:
      bucket: stash-testing
      prefix: stash-auto-backup/user3/deployment/stash-demo
    storageSecretName: gcs-secret
  usagePolicy:
    allowedNamespaces:
      from: All
```


**Verify BackupConfiguration:**

If everything goes well, Stash should create a `BackupConfiguration` for our Deployment in `user3` namespace and the phase of that `BackupConfiguration` should be `Ready`. Verify that the `BackupConfiguration` has been created by the following command,

```bash
$ kubectl get backupconfiguration -n user3
NAME                TASK   SCHEDULE      PAUSED   PHASE   AGE
deploy-stash-demo          */5 * * * *            Ready   3m24s
```

Let's check the YAML of this `BackupConfiguration`,

```bash
$ kubectl get backupconfiguration -n user3 deploy-stash-demo -o yaml
```

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: deploy-stash-demo
  namespace: user3
  ...
spec:
  driver: Restic
  repository:
    name: user3-deploy-stash-demo
    namespace: backup
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

```

Notice that the `spec.repository` is pointing to the `user3-deploy-stash-demo` Repository of the `backup` namespace. Also, notice that the `spec.target.paths` and `spec.target.volumeMounts` fields have been populated with the information we had provided as annotation of the Deployment.

**Wait for BackupSession to be succeded:**

Now, wait for the next backup schedule to invoke a BackupSession and to go the session into Succeeded phase. Run the following command to watch the `BackupSession` object in ns1 namespace:

```bash
$ kubectl get backupsession -n user3 -w
NAME                           INVOKER-TYPE          INVOKER-NAME        PHASE       DURATION   AGE
deploy-stash-demo-1648720801   BackupConfiguration   deploy-stash-demo   Running                5s
deploy-stash-demo-1648720801   BackupConfiguration   deploy-stash-demo   Running                21s
deploy-stash-demo-1648720801   BackupConfiguration   deploy-stash-demo   Succeeded   36s        35s
```

## Cleanup

To cleanup the resources created by this tutorial, please run the following commands,

```bash
$ kubectl delete deployment -n user1 stash-demo
deployment.apps "stash-demo" deleted
$ kubectl delete deployment -n user2 stash-demo
deployment.apps "stash-demo" deleted
$ kubectl delete deployment -n user2 stash-demo
deployment.apps "stash-demo" deleted
$ kubectl delete repository -n backup user1-deploy-stash-demo
repository.stash.appscode.com "user1-deploy-stash-demo" deleted
$ kubectl delete repository -n backup user2-deploy-stash-demo
repository.stash.appscode.com "user2-deploy-stash-demo" deleted
$ kubectl delete repository -n backup user3-deploy-stash-demo
repository.stash.appscode.com "user3-deploy-stash-demo" deleted
$ kubectl delete backupblueprint managed-backup-blueprint
backupblueprint.stash.appscode.com "managed-backup-blueprint" deleted
$ kubectl delete secret -n backup gcs-secret
secret/gcs-secret deleted
```