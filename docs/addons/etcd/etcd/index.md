---
title: Logical Backup & Restore Etcd | Stash
description: Take logical backup of Etcd database using Stash
menu:
  docs_{{ .version }}:
    identifier: stash-etcd
    name: Etcd
    parent: stash-etcd
    weight: 20
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: stash-addons
---

# Take a logical backup of the Etcd database using Stash

Stash `{{< param "info.version" >}}` supports backup and restoration of Etcd databases. This guide will show you how you can take a logical backup of your Etcd databases and restore them using Stash.

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster.
- Install Stash Enterprise in your cluster following the steps [here](/docs/setup/install/enterprise.md).
- If you are not familiar with how Stash backup and restore Etcd databases, please check the following guide [here](/docs/addons/etcd/overview/index.md).

You have to be familiar with following custom resources:

- [AppBinding](/docs/concepts/crds/appbinding.md)
- [Function](/docs/concepts/crds/function.md)
- [Task](/docs/concepts/crds/task.md)
- [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md)
- [BackupSession](/docs/concepts/crds/backupsession.md)
- [RestoreSession](/docs/concepts/crds/restoresession.md)

To keep things isolated, we are going to use a separate namespace called `demo` throughout this tutorial. Create `demo` namespace if you haven't created that already.

```bash
$ kubectl create ns demo
namespace/demo created
```

> Note: YAML files used in this tutorial are stored [here](https://github.com/stashed/docs/tree/{{< param "info.version" >}}/docs/addons/etcd/etcd/examples).

## Prepare Etcd

In this section, we are going to deploy an Etcd database cluster. Then, we will insert some sample data into it.

### Deploy Etcd

At first, let's deploy an Etcd database cluster. Here, we will use a statefulset and a service for deploying an Etcd database cluster consisting of three members. The service is used for handling peer communications and client requests.

Let's deploy an Etcd database cluster named `etcd` using a statefulset and a service  from the YAML manifest as below,

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: demo
  name: etcd
spec:
  clusterIP: None
  ports:
    - port: 2379
      name: client
    - port: 2380
      name: peer
  selector:
    app: etcd
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: etcd
  namespace: demo
  labels:
    app: etcd
spec:
  serviceName: etcd
  replicas: 3
  selector:
    matchLabels:
      app: etcd
  template:
    metadata:
      name: etcd
      labels:
        app: etcd
    spec:
      containers:
        - name: etcd
          image: gcr.io/etcd-development/etcd:v3.5.0
          ports:
            - containerPort: 2379
              name: client
            - containerPort: 2380
              name: peer
          volumeMounts:
            - name: data
              mountPath: /var/run/etcd
          command:
            - /bin/sh
            - -c
            - |
              PEERS="etcd-0=http://etcd-0.etcd:2380,etcd-1=http://etcd-1.etcd:2380,etcd-2=http://etcd-2.etcd:2380"
              exec etcd --name ${HOSTNAME} \
                --listen-peer-urls http://0.0.0.0:2380 \
                --listen-client-urls http://0.0.0.0:2379 \
                --initial-cluster etcd-0=http://etcd-0.etcd:2380,etcd-1=http://etcd-1.etcd:2380,etcd-2=http://etcd-2.etcd:2380 \
                --initial-cluster-token etcd-cluster-1 \
                --advertise-client-urls http://${HOSTNAME}.etcd:2379 \
                --initial-advertise-peer-urls http://${HOSTNAME}.etcd:2380 \
                --data-dir /var/run/etcd
  volumeClaimTemplates:
    - metadata:
        name: data
        namespace: demo
      spec:
        storageClassName: standard
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 1Gi
```

Let's create the `Etcd database cluster` we have shown above,
```bash
$ kubectl apply -f https://github.com/stashed/docs/tree/{{< param "info.version" >}}/docs/addons/etcd/etcd/examples/etcd.yaml
service/etcd created
statefulset.apps/etcd created
```
This YAML will create the necessary Statefulset, Service, PVCs for the Etcd database Cluster. 

Now, wait for the database pods `etcd-0`, `etcd-1`, and `etcd-2` to go into `Running` state,

```bash
❯ kc get pods -n demo --selector=app=etcd

NAME     READY   STATUS    RESTARTS   AGE
etcd-0   1/1     Running   0          11s
etcd-1   1/1     Running   0          10s
etcd-2   1/1     Running   0          9s
```

Once the database pod is in `Running` state, verify that the database is ready to accept the connections. For that, we have to exec into any one of the Etcd database cluster pods and run the `endpoint health` command. If the command returns a healthy state, then we can conclude that the database is ready to accept connections. To exec into a database pod and check endpoint health, run the following commands,

```bash
❯ kubectl exec -it -n demo etcd-0 -- /bin/sh
127.0.0.1:2379> etcdctl endpoint health
127.0.0.1:2379 is healthy: successfully committed proposal: took = 1.258639ms
```
From the above log, we can see the Etcd database is ready to accept connections.

### Insert Sample Data

Now, we are going to exec into the database pod and create some sample data. To use basic-authentication from Stash, we need to create a secret containing access credentials. Here, we are going to use `root` as user and `qwe` as password to authenticate and insert sample data into our Etcd database. We have to encode our username and password in base64 encoding into the secret. Let's create the secret containing the username and password of Etcd database from the following YAML,

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: etcd-stash-auth
  namespace: demo
type: Opaque
data:
  username: cm9vdA==
  password: cXdl
```

Let's create the `etcd-stash-auth` secret we have shown above,
```bash
$ kubectl apply -f https://github.com/stashed/docs/tree/{{< param "info.version" >}}/docs/addons/etcd/etcd/examples/etcd-secret.yaml
secret/etcd-stash-auth created
```

Now, let's exec into anyone of the database pod and insert some sample data,
```bash
❯ kubectl exec -it -n demo etcd-0 -- /bin/sh
127.0.0.1:2379> etcdctl --user root:qwe put foo bar
OK
127.0.0.1:2379> etcdctl --user root:qwe put foo2 bar2
OK
127.0.0.1:2379> etcdctl --user root:qwe put foo3 bar3
OK

# Verify that the data has been inserted successfully
127.0.0.1:2379> etcdctl --user root:qwe get --prefix foo
foo
bar
foo2
bar2
foo3
bar3
127.0.0.1:2379> exit
```

We have successfully deployed an Etcd database cluster and inserted some sample data into it. In the subsequent sections, we are going to backup these data using Stash.

## Prepare for Backup

In this section, we are going to prepare the necessary resources (i.e. database connection information, backend information, etc.) before backup.

### Ensure Etcd Addon

When you install Stash Enterprise version, it will automatically install all the official database addons. Make sure that Etcd addon was installed properly using the following command.

```bash
❯ kubectl get tasks.stash.appscode.com | grep etcd
etcd-backup-3.5.0             18m
etcd-restore-3.5.0            18m
```

This addon should be able to take backup of the databases with matching major versions as discussed in [Addon Version Compatibility](/docs/addons/etcd/README.md#addon-version-compatibility).

### Create AppBinding

Stash needs to know how to connect with the Etcd databse cluster. An `AppBinding` exactly provides this information. It holds the Service and Secret information of the Etcd database cluster. You have to point to the respective `AppBinding` as a target of backup instead of the Etcd database cluster itself.

Here, is the YAML of the `AppBinding` that we are going to create for the Etcd database cluster we have deployed earlier.

```yaml
apiVersion: appcatalog.appscode.com/v1alpha1
kind: AppBinding
metadata:
  name: etcd-appbinding
  namespace: demo
spec:
  clientConfig:
    service:
      name: etcd
      port: 2379
      scheme: http
  secret:
    name: etcd-stash-auth
  type: etcd
  version: 3.5.0
```

Here,

- **.spec.clientConfig.service** specifies the Service information to use to connects with the database.
- **.spec.secret** specifies the name of the Secret that holds necessary credentials to access the database. If your Etcd database is not using authentication, then don't provide this field.
- `spec.type` specifies the type of the database.

Let's create the `AppBinding` we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/tree/{{< param "info.version" >}}/docs/addons/etcd/etcd/examples/appbinding.yaml
appbinding.appcatalog.appscode.com/etcd-appbinding created
```

### Prepare Backend

We are going to store our backed up data into a GCS bucket. So, we need to create a Secret with GCS credentials and a `Repository` object with the bucket information. If you want to use a different backend, please read the respective backend configuration doc from [here](/docs/guides/latest/backends/overview.md).

**Create Storage Secret:**

At first, let's create a secret called `gcs-secret` with access credentials to our desired GCS bucket,

```bash
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

Now, crete a `Repository` object with the information of your desired bucket. Below is the YAML of `Repository` object we are going to create,

```yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: gcs-repo
  namespace: demo
spec:
  backend:
    gcs:
      bucket: stash-testing
      prefix: /demo/etcd/sample-etcd
    storageSecretName: gcs-secret
```

Let's create the `Repository` we have shown above,

```bash
$ kubectl create -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/addons/etcd/etcd/examples/repository.yaml
repository.stash.appscode.com/gcs-repo created
```

Now, we are ready to backup our database into our desired backend.

### Backup

To schedule a backup, we have to create a `BackupConfiguration` object targeting the respective `AppBinding` of our desired database. Then Stash will create a CronJob to periodically backup the database.

#### Create BackupConfiguration

Below is the YAML for `BackupConfiguration` object we are going to use to backup the `etcd` database cluster we have deployed earlier,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: etcd-backup
  namespace: demo
spec:
  schedule: "*/5 * * * *"
  task:
    name: etcd-backup-3.5.0
  repository:
    name: gcs-repo
  target:
    ref:
      apiVersion: appcatalog.appscode.com/v1alpha1
      kind: AppBinding
      name: etcd-appbinding 
  retentionPolicy:
    name: keep-last-5
    keepLast: 5
    prune: true
```

Here,

- `.spec.schedule` specifies that we want to backup the database at 5 minutes intervals.
- `.spec.task.name` specifies the name of the Task object that specifies the necessary Functions and their execution order to backup a Etcd database.
- `.spec.repository.name` specifies the Repository CR name we have created earlier with backend information.
- `.spec.target.ref` refers to the AppBinding object that holds the connection information of our targeted database.
- `.spec.retentionPolicy` specifies a policy indicating how we want to cleanup the old backups.

Let's create the `BackupConfiguration` object we have shown above,

```bash
$ kubectl create -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/addons/etcd/etcd/examples/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/etcd-backup created
```

#### Verify CronJob

If everything goes well, Stash will create a CronJob with the schedule specified in `spec.schedule` field of `BackupConfiguration` object.

Verify that the CronJob has been created using the following command,

```bash
❯ kubectl get cronjob -n demo
NAME                               SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
stash-trigger-etcd-backup          */5 * * * *   False     0        <none>          14s
```

#### Wait for BackupSession

The `stash-trigger-etcd-backup` CronJob will trigger a backup on each scheduled slot by creating a `BackupSession` object.

Now, wait for a schedule to appear. Run the following command to watch for a `BackupSession` object,

```bash
❯ kubectl get backupsession -n demo -w
NAME                             INVOKER-TYPE          INVOKER-NAME          PHASE       DURATION          AGE
etcd-backup-1634538902           BackupConfiguration   etcd-backup                       0s
etcd-backup-1634538902           BackupConfiguration   etcd-backup           Running                       0s
etcd-backup-1634538902           BackupConfiguration   etcd-backup           Succeeded   39s               39s
```

Here, the phase `Succeeded` means that the backup process has been completed successfully.

#### Verify Backup

Now, we are going to verify whether the backed up data is present in the backend or not. Once a backup is completed, Stash will update the respective `Repository` object to reflect the backup completion. Check that the repository `gcs-repo` has been updated by the following command,

```bash
❯ kubectl get repository -n demo
NAME       INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
gcs-repo   true        93 B   1                2m1s                     24m
```

Now, if we navigate to the GCS bucket, we will see the backed up data has been stored in `/demo/etcd/sample-etcd` directory as specified by `.spec.backend.gcs.prefix` field of the `Repository` object.

<figure align="center">
  <img alt="Backup data in GCS Bucket" src="/docs/addons/etcd/etcd/images/sample-etcd-backup.png">
  <figcaption align="center">Fig: Backup data in GCS Bucket</figcaption>
</figure>

> Note: Stash keeps all the backed up data encrypted. So, data in the backend will not make any sense until they are decrypted.

## Restore Etcd

If you have followed the previous sections properly, you should have a successful logical backup of your Etcd database. Now, we are going to show how you can restore the database from the backed up data.

### Restore Into the Same Database

You can restore your data into the same database you have backed up from or into a different database in the same cluster or a different cluster. In this section, we are going to show you how to restore in the same database which may be necessary when you have accidentally deleted any data from the running database.

#### Temporarily Pause Backup

At first, let's stop taking any further backup of the database so that no backup runs after we delete the sample data. We are going to pause the `BackupConfiguration` object. Stash will stop taking any further backup when the `BackupConfiguration` is paused.

Let's pause the `etcd-backup` BackupConfiguration,

```bash
$ kubectl patch backupconfiguration -n demo etcd-backup --type="merge" --patch='{"spec": {"paused": true}}'
backupconfiguration.stash.appscode.com/etcd-backup patched
```

Verify that the `BackupConfiguration` has been paused,

```bash
❯ kubectl get backupconfiguration -n demo etcd-backup
NAME                  TASK                 SCHEDULE      PAUSED   AGE
etcd-backup           etcd-backup-3.5.0    */5 * * * *   true     22m
```

Notice the `PAUSED` column. Value `true` for this field means that the `BackupConfiguration` has been paused.

Stash will also suspend the respective CronJob.

```bash
❯ kubectl get cronjob -n demo
NAME                               SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
stash-trigger-etcd-backup          */5 * * * *   True      0        6m15s           23m
```

#### Simulate Disaster

Now, let's simulate an accidental deletion scenario. Here, we are going to exec into the database pod and delete the sample data we have inserted earlier.

```bash
❯ kubectl exec -it -n demo etcd-0 -- /bin/sh
127.0.0.1:2379> etcdctl --user root:qwe del foo
1
127.0.0.1:2379> etcdctl --user root:qwe del foo2
1
127.0.0.1:2379> etcdctl --user root:qwe del foo3
1
# verify that the sample data has been deleted
127.0.0.1:2379> etcdctl --user root:qwe get --prefix foo
(nil)
127.0.0.1:2379> exit
```

#### Create RestoreSession

To restore the database, you have to create a `RestoreSession` object pointing to the `AppBinding` of the targeted database.

Here, is the YAML of the `RestoreSession` object that we are going to use for restoring our `etcd` database.

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: etcd-restore
  namespace: demo
spec:
  task:
    name: etcd-restore-3.5.0 
    params:
      - name: initialCluster
        value:  "etcd-0=http://etcd-0.etcd:2380,etcd-1=http://etcd-1.etcd:2380,etcd-2=http://etcd-2.etcd:2380"
      - name: initialClusterToken
        value: "etcd-cluster-1"
      - name: dataDir
        value: "/var/run/etcd"
      - name: workloadKind
        value: "StatefulSet"
      - name: workloadName
        value: "etcd"
  repository:
    name: gcs-repo
  target:
    ref:
      apiVersion: appcatalog.appscode.com/v1alpha1
      kind: AppBinding
      name: etcd-appbinding 
  runtimeSettings:
    container:
      securityContext:
        runAsUser: 0
        runAsGroup: 0
  rules:
  - snapshots: [latest]
```

Here,

- `.spec.task.name` specifies the name of the Task object that specifies the necessary Functions and their execution order to restore a Etcd database.
- `.spec.task.params` refers to the names and values of the Params objects specifying necessary parameters and their values for restoring backupdata into an Etcd database cluster.
- `.spec.repository.name` specifies the Repository object that holds the backend information where our backed up data has been stored.
- `.spec.target.ref` refers to the respective AppBinding of the `etcd` database.
- `.spec.rules` specifies that we are restoring data from the latest backup snapshot of the database.

Let's create the `RestoreSession` object object we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/addons/etcd/etcd/examples/restoresession.yaml
restoresession.stash.appscode.com/etcd-restore created
```

Once, you have created the `RestoreSession` object, Stash will create a restore Job. Run the following command to watch the phase of the `RestoreSession` object,

```bash
❯ kubectl get restoresession -n demo -w
NAME                   REPOSITORY   PHASE     DURATION          AGE
etcd-restore           gcs-repo     Running                     9s
etcd-restore           gcs-repo     Running                     13s
etcd-restore           gcs-repo     Running                     72s
etcd-restore           gcs-repo     Succeeded                   72s
etcd-restore           gcs-repo     Succeeded  1m13s            72s
```

The `Succeeded` phase means that the restore process has been completed successfully.

#### Verify Restored Data

Now, let's exec into the database pod and verify whether data actual data has been restored or not,

```bash
❯ kubectl exec -it -n demo etcd-0 -- /bin/sh

127.0.0.1:2379> etcdctl --user root:qwe get --prefix foo
foo
bar
foo2
bar2
foo3
bar3
127.0.0.1:2379> exit
```

Hence, we can see from the above output that the deleted data has been restored successfully from the backup.

#### Resume Backup

Since our data has been restored successfully we can now resume our usual backup process. Resume the `BackupConfiguration` using following command,

```bash
❯ kubectl patch backupconfiguration -n demo etcd-backup --type="merge" --patch='{"spec": {"paused": false}}'
backupconfiguration.stash.appscode.com/etcd-backup patched
```

Verify that the `BackupConfiguration` has been resumed,
```bash
❯ kubectl get backupconfiguration -n demo etcd-backup
NAME                  TASK                 SCHEDULE      PAUSED   AGE
etcd-backup           etcd-backup-3.5.0    */5 * * * *   false    4h54m
```

Here,  `false` in the `PAUSED` column means the backup has been resume successfully. The CronJob also should be resumed now.

```bash
❯ kubectl get cronjob -n demo
NAME                               SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
stash-trigger-etcd-backup          */5 * * * *   False     0        3m24s           4h54m
```

Here, `False` in the `SUSPEND` column means the CronJob is no longer suspended and will trigger in the next schedule.

### Restore Into Different Database of the Same Namespace

If you want to restore the backed up data into a different database of the same namespace, you have to create another `AppBinding` pointing to the desired database. Then, you have to create the `RestoreSession` pointing to the new `AppBinding`.

### Restore Into Different Namespace

If you want to restore into a different namespace of the same cluster, you have to create the Repository, backend Secret, AppBinding, in the desired namespace. You can use [Stash kubectl plugin](https://stash.run/docs/latest/guides/latest/cli/cli/) to easily copy the resources into a new namespace. Then, you have to create the `RestoreSession` object in the desired namespace pointing to the Repository, AppBinding of that namespace.

### Restore Into Different Cluster

If you want to restore into a different cluster, you have to install Stash in the desired cluster. Then, you have to install Stash Etcd addon in that cluster too. Then, you have to create the Repository, backend Secret, AppBinding, in the desired cluster. Finally, you have to create the `RestoreSession` object in the desired cluster pointing to the Repository, AppBinding of that cluster.

## Cleanup

To cleanup the Kubernetes resources created by this tutorial, run:

```bash
kubectl delete -n demo backupconfiguration etcd-backup
kubectl delete -n demo restoresession etcd-restore
kubectl delete -n demo repository gcs-repo
# delete the database, service, and PVCs
kubectl delete -f https://github.com/stashed/docs/tree/{{< param "info.version" >}}/docs/addons/etcd/etcd/examples/etcd.yaml
kubectl delete pvc -n demo data-etcd-0 data-etcd-1 data-etcd-2
```
