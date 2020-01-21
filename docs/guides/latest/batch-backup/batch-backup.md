---
title: Batch Backup | Stash
description: A step by step guide showing how to backup application with multiple co-related components.
menu:
  product_stash_{{ .version }}:
    identifier: batch-backup
    name: Backup application with multiple co-related components
    parent: batch-backup
    weight: 20
product_name: stash
menu_name: product_stash_{{ .version }}
section_menu_id: guides
---

# Backup application with multiple co-related components using BackupBatch

This section will demonstrate how to use Stash to take backup of an application with multiple co-related components. Here, we are going to take backup of [WordPress with MySQL database](https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/).

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [kind](https://kind.sigs.k8s.io/docs/user/quick-start/).
- Install `Stash` in your cluster following the steps [here](/docs/setup/install.md).
- Install MySQL addon for Stash following the steps [here](https://stash.run/docs/v0.9.0-rc.2/addons/mysql/guides/8.0.14/mysql/).
- If you are not familiar with how Stash backup and restore MySQL databases, please check the following guide [here](https://stash.run/docs/v0.9.0-rc.2/addons/mysql/overview/).

- You should be familiar with the following `Stash` concepts:
  - [Appbinding](/docs/concepts/crds/appbinding.md)
  - [Function](/docs/concepts/crds/function.md)
  - [Task](/docs/concepts/crds/task.md)
  - [BackupBatch](/docs/concepts/crds/backupbatch.md)
  - [BackupSession](/docs/concepts/crds/backupsession.md)
  - [Repository](/docs/concepts/crds/repository.md)

To keep everything isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$ kubectl create ns demo
namespace/demo created
```

> **Note:** YAML files used in this tutorial are stored in [docs/examples/guides/latest/batch-backup](/docs/examples/guides/latest/batch-backup) directory of [stashed/docs](https://github.com/stashed/docs) repository.

## Deploy Application

At first, we are going to deploy a WordPress site with a MySQL database and generate some sample data in it. MySQL and WordPress each use a PersistentVolume to store data. Then, we are going to backup this application's data and database into a GCS bucket.

**Create database Secret :**

Now, we are going to create a secret to secure the data for MySQL Deployment.

Let's create a secret called `mysql-pass` ,

```console
$ kubectl create secret  -n demo generic mysql-pass \
      --from-literal=username=root \
      --from-literal=password=mysqlpass
secret/mysql-pass created
```

### Deploy Database

Below are the YAML of a sample MySQL Deployment with PVC and a Service that we are going to create for this tutorial:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  namespace: demo
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-mysql
  namespace: demo
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
      - image: mysql:8.0.14
        name: mysql
        args:
        - --default-authentication-plugin=mysql_native_password
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: username
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
        - name: config-volume
          mountPath: /etc/mysql/conf.d
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
      - name: config-volume
        emptyDir: {}
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  namespace: demo
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Create the above MySQL Deployment,

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/batch-backup/mysql.yaml
service/wordpress-mysql created
deployment.apps/wordpress-mysql created
persistentvolumeclaim/mysql-pv-claim created
```

Let's check if the MySQL Pod is ready to use,

```console
$ kubectl get pod -n demo
NAME                               READY   STATUS    RESTARTS   AGE
wordpress-mysql-58b865dfd7-dt6t8   1/1     Running   0          34s
```

Let's also check if the MySQL database is ready to connect,

```console
$ kubectl logs -n demo -f wordpress-mysql-58b865dfd7-dt6t8
Initializing database
....
...
2020-01-07T12:33:23.242350Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.0.14'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server - GPL.
2020-01-07T12:33:23.325223Z 0 [ERROR] [MY-011300] [Server] Plugin mysqlx reported: 'Setup of socket: '/var/run/mysqld/mysqlx.sock' failed, can't create lock file /var/run/mysqld/mysqlx.sock.lock'
2020-01-07T12:33:23.325316Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Bind-address: '::' port: 33060
```

We have created a service `wordpress-mysql` alongside the Deployment through which the MySQL database will be accessed.

Let's check if the Service is running:

```console
 $ kubectl get service -n demo  wordpress-mysql
NAME              TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
wordpress-mysql   ClusterIP   None         <none>        3306/TCP   39m
```

### Deploy WordPress

Now, we are going to deploy a WordPress site. This site will use the MySQL database through the service `wordpress-mysql` as deployed earlier. Below are the YAML of a sample WordPress Deployment with PVC and a Service that we are going to create for this tutorial:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  namespace: demo
  labels:
    app: wordpress
spec:
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
  type: LoadBalancer
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  namespace: demo
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
      - image: wordpress:5.3.2-apache
        name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
          value: wordpress-mysql
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        - name: WORDPRESS_DB_USER
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: username
        ports:
        - containerPort: 80
          name: wordpress
        volumeMounts:
        - name: wordpress-persistent-storage
          mountPath: /var/www/html
      volumes:
      - name: wordpress-persistent-storage
        persistentVolumeClaim:
          claimName: wp-pv-claim
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wp-pv-claim
  namespace: demo
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Create the above WordPress Deployment,

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/batch-backup/wordpress.yaml
service/wordpress created
deployment.apps/wordpress created
persistentvolumeclaim/wp-pv-claim created
```

Let's check if the WordPress Pod is ready to use,

```console
$ kubectl get pod -n demo
NAME                               READY   STATUS    RESTARTS   AGE
wordpress-549c4f6867-957mn         1/1     Running   0          7m41s
```

We have created a service(LoadBalancer type) `wordpress` alongside the WordPress Deployment through which the application will be exposed.

Let's check if the Service is running:

```console
$ kubectl get service -n demo  wordpress
NAME        TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
wordpress   LoadBalancer   10.104.176.125   <pending>     80:32555/TCP   20m
```

**Insert Sample Data into database :**

Now, you can access your application from outside of the cluster by using `http://<Cluster Node IP Address>:<service nodePort>` URL or any way. If you access the application by entering the URL into your browser, you should see that the WordPress set up page similar to the following screenshot:

<figure align="center">
  <img alt="WordPress application home page" src="/docs/images/guides/latest/batch-backup/wordpress-site.png">
  <figcaption align="center">Fig: WordPress application home page</figcaption>
</figure>

Now, you can insert sample data by using the WordPress application.

If you insert some sample data by using the WordPress application then you can verify it by using `exec` into the database pod with MySQL database `Secret` credentials,

```console
$ kubectl exec -it -n demo wordpress-mysql-58b865dfd7-dt6t8 -- mysql --user=root --password=mysqlpass
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 31
Server version: 8.0.14 MySQL Community Server - GPL

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| wordpress          |
+--------------------+
5 rows in set (0.00 sec)

mysql> SHOW TABLES IN wordpress;
+-----------------------+
| Tables_in_wordpress   |
+-----------------------+
| wp_commentmeta        |
| wp_comments           |
| wp_links              |
| wp_options            |
| wp_postmeta           |
| wp_posts              |
| wp_term_relationships |
| wp_term_taxonomy      |
| wp_termmeta           |
| wp_terms              |
| wp_usermeta           |
| wp_users              |
+-----------------------+
12 rows in set (0.00 sec)

mysql> exit
Bye
```

If you do the above procedure, you will see that `wordpress` database and related tables have been created.

### Backup Application

Now, we are going to backup the WordPress and MySQL database into GCS bucket.

#### Create AppBinding

For MySQL database backup, We have to create an `AppBinding` crd that holds the necessary information to connect with the MySQL database.

The following YAML shows a minimal AppBinding specification that you have to create,

```yaml
apiVersion: appcatalog.appscode.com/v1alpha1
kind: AppBinding
metadata:
  name: sample-mysql
  namespace: demo
spec:
  type: mysql
  version: 8.0.14
  clientConfig:
    service:
      name: wordpress-mysql
      port: 3306
      scheme: mysql
  secret:
    name: mysql-pass
```

Here,

- `.spec.clientConfig.service.name` specifies the name of the Service that connects to the MySQL database.
- `.spec.clientConfig.service.port` specifies the port where the target database is running.
- `.spec.secret` specifies the name of the Secret that holds the necessary credentials to access the database.
- `spec.type` specifies the MySQL app that this AppBinding is pointing to.

Create the above AppBinding,

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/batch-backup/appbinding.yaml
appbinding.appcatalog.appscode.com/sample-mysql created
```

#### Prepare Backend

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
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/batch-backup/repository.yaml
repository.stash.appscode.com/gcs-repo created
```

Now, we are ready to backup our sample data into this backend.

#### Backup

We have to create a `BackupBatch` crd targeting the respective MySQL database and WordPress Deployment that we have deployed. Stash will inject a sidecar container into WordPress Deployment and create job for MySQL database to take a periodic backup of the applications' data.

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
  schedule: "*/5 * * * *"
  members:
  - target:
      ref:
        apiVersion: apps/v1
        kind: AppBinding
        name: sample-mysql
    task:
      name: mysql-backup-8.0.14
  - target:
      ref:
        apiVersion: apps/v1
        kind: Deployment
        name: wordpress
      volumeMounts:
        - name: wordpress-persistent-storage
          mountPath: /var/www/html
      paths:
        - /var/www/html
  retentionPolicy:
    name: 'keep-last-10'
    keepLast: 10
    prune: true
```

Here,

- `spec.repository` refers to the `Repository` object `gcs-repo` that holds backend information.
- `spec.schedule` is a cron expression that indicates `BackupSession` will be created at 6 minute interval.
- `spec.members[].target.ref` refers to the `AppBinding` crd that was created for sample-MySQL database and `deploy-stash-demo` refers to the Deployment respectively.
- `spec.members[].task` specifies the name of the `Task` object that specifies the `Function` and their order of execution to perform a backup of a database.
- `spec.members[].target.volumeMounts` specifies a list of volumes and their mountPath that contain the target paths.
- `spec.members[].target.paths` specifies list of file paths to backup for target.

Let's create the `BackupBatch` crd we have shown above,

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/batch-backup/backupbatch.yaml
backupbatch.stash.appscode.com/deploy-backup-batch created
```

**Verify CronJob:**

Stash will also create a `CronJob` with the schedule specified in `spec.schedule` field of `BackupBatch` crd.

Verify that Stash has created a CronJob to trigger a periodic backup of the targeted components by the following command,

```console
$ kubectl get cronjob -n demo
NAME                  SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
deploy-backup-batch   */5 * * * *   False     0        2m51s           3m30s
```

**Wait for BackupSession:**

The `deploy-backup-batch` CronJob will trigger a backup on each scheduled slot by creating a `BackupSession` crd.
The sidecar container and the stash operator watches for the `BackupSession` crd. When they find one, they will take backup of the applications' components separately.

Wait for the next schedule for backup. Run the following command to watch `BackupSession` crd,

```console
$ watch -n 3 kubectl get backupsession -n demo
Every 3.0s: kubectl get backupsession -n demo                suaas-appscode: Sun Jan  5 14:40:51 2020

NAME                             INVOKER-TYPE   INVOKER-NAME          PHASE       AGE
deploy-backup-batch-1578458376   BackupBatch    deploy-backup-batch   Succeeded   101s
```

We can see from the above output that the backupSession has succeeded.

If you describe the `deploy-backup-batch-1578458376` BackupSession crd, you will see in the status section of the `BackupSession` that all the target specified in the `BackupBatch` have succeeded.

```console
$ kubectl describe backupsession -n demo deploy-backup-batch-1578458376
```

```yaml
Name:         deploy-backup-batch-1578458376
Namespace:    demo
Labels:       app.kubernetes.io/component=stash-backup
              app.kubernetes.io/managed-by=stash.appscode.com
              stash.appscode.com/invoker-name=deploy-backup-batch
              stash.appscode.com/invoker-type=BackupBatch
Annotations:  <none>
API Version:  stash.appscode.com/v1beta1
Kind:         BackupSession
Metadata:
  Creation Timestamp:  2020-01-08T04:39:36Z
  Generation:          1
  Owner References:
    API Version:           stash.appscode.com/v1beta1
    Block Owner Deletion:  true
    Controller:            true
    Kind:                  BackupBatch
    Name:                  deploy-backup-batch
    UID:                   f5b9a1ce-238f-432a-86ac-287e2a85ef26
  Resource Version:        7332
  Self Link:               /apis/stash.appscode.com/v1beta1/namespaces/demo/backupsessions/deploy-backup-batch-1578458376
  UID:                     4bc5607b-04cd-4aeb-8f61-7dd21483ebb4
Spec:
  Invoker:
    API Group:  stash.appscode.com
    Kind:       BackupBatch
    Name:       deploy-backup-batch
Status:
  Phase:             Succeeded
  Session Duration:  2m6.273902333s
  Targets:
    Phase:  Succeeded
    Ref:
      Kind:  AppBinding
      Name:  sample-mysql
    Stats:
      Duration:  28.449428155s
      Hostname:  host-0
      Phase:     Succeeded
      Snapshots:
        File Stats:
          Modified Files:    0
          New Files:         1
          Total Files:       1
          Unmodified Files:  0
        Name:                597602f9
        Path:                dumpfile.sql
        Processing Time:     0:04
        Uploaded:            3.407 MiB
    Total Hosts:             1
    Phase:                   Succeeded
    Ref:
      Kind:  Deployment
      Name:  wordpress
    Stats:
      Duration:  50.781377951s
      Hostname:  host-0
      Phase:     Succeeded
      Snapshots:
        File Stats:
          Modified Files:    0
          New Files:         1932
          Total Files:       1932
          Unmodified Files:  0
        Name:                ce1c2487
        Path:                /var/www/html
        Processing Time:     0:24
        Total Size:          42.702 MiB
        Uploaded:            42.645 MiB
    Total Hosts:             1
Events:
  Type    Reason                   Age    From                      Message
  ----    ------                   ----   ----                      -------
  Normal  BackupSession Running    2m18s  BackupSession Controller  Backup job has been created succesfully/sidecar is watching the BackupSession.
  Normal  BackupSession Running    2m18s  BackupSession Controller  Backup job has been created succesfully/sidecar is watching the BackupSession.
  Normal  Host Backup Succeeded    88s    Status Updater            backup succeeded for host host-0 of "Deployment"/"wordpress".
  Normal  Host Backup Succeeded    12s    Status Updater            backup succeeded for host host-0 of "AppBinding"/"sample-mysql".
  Normal  BackupSession Succeeded  12s    BackupSession Controller  Backup session completed successfully
```

Now, we are going to verify whether the backed up data has been stored in the backend.

**Verify Backup:**

When a backup session is completed, Stash will update the respective `Repository` to reflect the latest state of backed up data.

Run the following command to check if a backup snapshot has been stored in the backend,

```console
$ kubectl get repository -n demo gcs-repo
NAME       INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
gcs-repo   true               3                3s                       38m
```

From the output above, we can see that 4 snapshots have been stored in the backend specified by Repository `gcs-repo`.

Now, if we navigate to the GCS bucket, we are going to see backed up data has been stored in `/demo/data/<target-kind>/<target-name>` directory.
> Stash forms the directory by concatenating the `spec.backend.gcs.prefix` field of Repository crd and `<target-kind>/<target-name>`.

<figure align="center">
  <img alt="Backup data in GCS Bucket" src="/docs/images/guides/latest/batch-backup/apps-data.png">
  <figcaption align="center">Fig: Backup data in GCS Bucket</figcaption>
</figure>

> **Note:** Stash keeps all the backed up data encrypted. So, data in the backend will not make any sense until they are decrypted.

## Cleaning Up

To clean up the Kubernetes resources created by this tutorial, run:

```console
kubectl delete -n demo backupbatch deploy-backup-batch
kubectl delete -n demo deployment wordpress-mysql
kubectl delete -n demo deployment wordpress
kubectl delete -n demo repository gcs-repo
kubectl delete -n demo pvc --all
kubectl delete -n demo service --all
```
