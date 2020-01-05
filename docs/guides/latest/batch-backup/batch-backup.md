---
title: Batch Backup | Stash
description: A step by step guide showing how to backup StatefulSet Application data.
menu:
  product_stash_{{ .version }}:
    identifier: batch-backup
    name: Backup StatefulSet Application data
    parent: batch-backup
    weight: 20
product_name: stash
menu_name: product_stash_{{ .version }}
section_menu_id: guides
---

# Backup StatefulSet Application Data

Stash supports batch backup volumes of workloads, standalone PVCs and databases simultaneously. This guide will show you how to use Stash to take backup of StatefulSet applications like WordPress with MySQL database.

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

> **Note:** YAML files used in this tutorial are stored in [docs/examples/guides/latest/batch-backup](/docs/examples/guides/latest/batch-backup) directory of [stashed/docs](https://github.com/stashed/docs) repository.

## Batch backup of WordPress application with MySQL database

This section will demonstrate how to use Stash to take batch backup of [WordPress and MySQL database](https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/). Here, we are going to deploy a WordPress site with a MySQL database and generate some sample data in it. MySQL and WordPress each requires a PersistentVolume to store data. Their PVCs will be created at the deployment step. Then, we are going to backup this application's data and database into a GCS bucket.

**Create Secret for MySQL database and WordPress application :**

At first, we will create a secret to secure the data for WordPress and MySQL Deployment.

Let's create a secret called `mysql-pass` to secure application's data,

```console
$ kubectl create secret  -n demo generic mysql-pass --from-literal=password=mysqlpass
secret/mysql-pass created
```

### Deploy MySQL database

Now, we are going to deploy a MySQL database. Below is the YAML of a sample MySQL Deployment that we are going to create for this tutorial:

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
  clusterIP: None
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
      initContainers:
      - image: busybox
        name: busybox
        command:
        - rm
        - -rf
        - /var/lib/mysql/lost+found
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      containers:
      - image: mysql:5.7
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
        - name: config-volume
          mountPath: /etc/mysql/
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
      storage: 20Gi
```

Create the above MySQL Deployment,

```console
$ kubectl apply -f ./docs/examples/guides/latest/batch-backup/mysql-deployment.yaml
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
kubectl logs -n demo wordpress-mysql-58b865dfd7-dt6t8
....
....
2020-01-05T07:03:46.156073Z 0 [Warning] Insecure configuration for --pid-file: Location '/var/lib/mysql' in the path is accessible to all OS users. Consider choosing a different directory.
2020-01-05T07:03:46.163686Z 0 [Note] Event Scheduler: Loaded 0 events
2020-01-05T07:03:46.163879Z 0 [Note] mysqld: ready for connections.
Version: '5.7.28'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server (GPL)
```

We have created a service `wordpress-mysql` alongside the Deployment through which the MySQL database will be accessed.

Let's check if the Service is running:

```console
 $ kubectl get service -n demo  wordpress-mysql
NAME              TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
wordpress-mysql   ClusterIP   None         <none>        3306/TCP   39m
```

### Deploy WordPress application

Now we are going to deploy a WordPress site. This site will use the MySQL database through the service `wordpress-mysql`. Below is the YAML of a sample WordPress site Deployment that we are going to create for this tutorial:

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
      - image: wordpress:4.8-apache
        name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
          value: wordpress-mysql
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
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
      storage: 20Gi
```

Create the above WordPres Deployment,

```console
$ kubectl apply -f ./docs/examples/guides/latest/batch-backup/wordpress-deployment.yaml
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

Now, you can access your application from outside of the cluster by using `http://<Cluster Node IP Address>:<port>` URl.

Let's access the application by entering the URL into your browser ,

```console
http://172.17.0.4:32555
```

Here,

- `172.17.0.4` is the `Kind` Cluster Node IP address
- `32555` is the port of the service

You should see the WordPress set up page similar to the following screenshot:

<figure align="center">
  <img alt="WordPress Application HomePage" src="/docs/images/guides/latest/batch-backup/wordpress-site.png">
  <figcaption align="center">Fig: WordPress Application HomePage</figcaption>
</figure>

The application is ready to use. Let's create some sample data using this WordPress application and then backup the data in GCS Bucket.

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
$ kubectl apply -f ./docs/examples/guides/latest/batch-backup/repository.yaml
repository.stash.appscode.com/gcs-repo created
```

Now, we are ready to backup our sample data into this backend.

### Backup

We have to create a `BackupBatch` crd targeting the respective MySQL and WordPress Deployment that we have deployed. Stash will inject a sidecar container into Deployments' target. It will also create a `CronJob` to take periodic backup of the application's volume simultaneously.

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
        kind: Deployment
        name: wordpress-mysql
      volumeMounts:
      - name: mysql-persistent-storage
        mountPath: /var/lib/mysql
      paths:
      - /var/lib/mysql
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
$ kubectl apply -f ./docs/examples/guides/latest/batch-backup/backupbatch.yaml
backupbatch.stash.appscode.com/deploy-backup-batch created
```

**Verify Sidecar:**

If everything goes well, Stash will inject a sidecar container into the `wordpress-mysql` and `wordpress` Deployment to take backup of `/var/lib/mysql` and `/var/www/html` directory respectively. Let’s check that the sidecar has been injected successfully and wait for all the pods to go into the `Running` state,

```console
$ kubectl get pod -n demo
NAME                              READY   STATUS    RESTARTS   AGE
wordpress-66954d8645-mbc2p        2/2     Running   0          10s
wordpress-mysql-69fcf4664-64jgm   2/2     Running   0          8s
```

Look at the pod. It now has 2 containers. If you view the resource definition of the `wordpress-mysql-69fcf4664-64jgm` pod, you will see that there is a container named `stash` which is running `run-backup` command.

```console
$ kubectl get pod -n demo wordpress-mysql-69fcf4664-64jgm  -o yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    stash.appscode.com/last-applied-backup-invoker-hash: "347177349119174663"
  creationTimestamp: "2020-01-05T08:35:25Z"
  generateName: wordpress-mysql-69fcf4664-
  labels:
    app: wordpress
    pod-template-hash: 69fcf4664
    tier: mysql
  name: wordpress-mysql-69fcf4664-64jgm
  namespace: demo
  ...
  resourceVersion: "15814"
  selfLink: /api/v1/namespaces/demo/pods/wordpress-mysql-69fcf4664-64jgm
  uid: b470971c-9455-4857-add1-b7d935fb0419
spec:
  containers:
  - env:
    - name: MYSQL_ROOT_PASSWORD
      valueFrom:
        secretKeyRef:
          key: password
          name: mysql-pass
    image: mysql:5.7
    imagePullPolicy: IfNotPresent
    name: mysql
    ports:
    - containerPort: 3306
      name: mysql
      protocol: TCP
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/lib/mysql
      name: mysql-persistent-storage
    - mountPath: /etc/mysql/
      name: config-volume
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-5xmtv
      readOnly: true
  - args:
    - run-backup
    - --invoker-name=deploy-backup-batch
    - --invoker-type=BackupBatch
    - --target-name=wordpress-mysql
    - --target-kind=Deployment
    - --secret-dir=/etc/stash/repository/secret
    - --enable-cache=true
    - --max-connections=0
    - --metrics-enabled=true
    - --pushgateway-url=http://stash.kube-system.svc:56789
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
    image: appscodeci/stash:fix-backup-batch_linux_amd64
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
    - mountPath: /var/lib/mysql
      name: mysql-persistent-storage
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-5xmtv
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  initContainers:
  - command:
    - rm
    - -rf
    - /var/lib/mysql/lost+found
    image: busybox
    imagePullPolicy: Always
    name: busybox
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/lib/mysql
      name: mysql-persistent-storage
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-5xmtv
      readOnly: true
  nodeName: kind-worker
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext:
    fsGroup: 65535
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  ...
  volumes:
  - name: mysql-persistent-storage
    persistentVolumeClaim:
      claimName: mysql-pv-claim
  - emptyDir: {}
    name: config-volume
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
  - name: default-token-5xmtv
    secret:
      defaultMode: 420
      secretName: default-token-5xmtv
...
```

**Verify CronJob:**

Stash will also create a `CronJob` with the schedule specified in `spec.schedule` field of `BackupBatch` crd.

Verify that Stash has created a CronJob to trigger a periodic backup of the targeted Deployments by the following command,

```console
$ kubectl get cronjob -n demo
NAME                  SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
deploy-backup-batch   */5 * * * *   False     0        2m51s           3m30s
```

**Wait for BackupSession:**

The `deploy-backup-batch` CronJob will trigger a backup on each scheduled slot by creating a `BackupSession` crd.
The sidecar container and the stash operator watches for the `BackupSession` crd. When they find one, they will take backup of the application volume separately.

Wait for the next schedule for backup. Run the following command to watch `BackupSession` crd,

```console
$ watch -n 3 kubectl get backupsession -n demo
Every 3.0s: kubectl get backupsession -n demo                suaas-appscode: Sun Jan  5 14:40:51 2020

NAME                             INVOKER-TYPE   INVOKER-NAME          PHASE       AGE
deploy-backup-batch-1578213550   BackupBatch    deploy-backup-batch   Succeeded   101s
```

We can see from the above output that the backupSession has succeeded.

If you describe the `deploy-backup-batch-1578213550` BackupSession crd, you will see in the status section of the `BackupSession` that all the target specified in the `BackupBatch` have succeeded.

```console
$ kubectl describe backupsession -n demo deploy-backup-batch-1578213550
```

```yaml
Name:         deploy-backup-batch-1578213550
Namespace:    demo
Labels:       app.kubernetes.io/component=stash-backup
              app.kubernetes.io/managed-by=stash.appscode.com
              stash.appscode.com/invoker-name=deploy-backup-batch
              stash.appscode.com/invoker-type=BackupBatch
Annotations:  <none>
API Version:  stash.appscode.com/v1beta1
Kind:         BackupSession
Metadata:
  Creation Timestamp:  2020-01-05T08:39:10Z
  Generation:          1
  ...
  Resource Version:        16610
  Self Link:               /apis/stash.appscode.com/v1beta1/namespaces/demo/backupsessions/deploy-backup-batch-1578213550
  UID:                     f13ed6ff-564f-4194-a068-990886169412
Spec:
  Invoker:
    API Group:  stash.appscode.com
    Kind:       BackupBatch
    Name:       deploy-backup-batch
Status:
  Phase:             Succeeded
  Session Duration:  24.29772244s
  Targets:
    Phase:  Succeeded
    Ref:
      Kind:  Deployment
      Name:  wordpress-mysql
    Stats:
      Duration:  23.58662455s
      Hostname:  host-0
      Phase:     Succeeded
      Snapshots:
        File Stats:
          Modified Files:    0
          New Files:         0
          Total Files:       4
          Unmodified Files:  4
        Name:                ce208676
        Path:                /var/lib/mysql
        Processing Time:     0:02
        Total Size:          184.011 MiB
        Uploaded:            0 B
    Total Hosts:             1
    Phase:                   Succeeded
    Ref:
      Kind:  Deployment
      Name:  wordpress
    Stats:
      Duration:  23.654072068s
      Hostname:  host-0
      Phase:     Succeeded
      Snapshots:
        File Stats:
          Modified Files:    0
          New Files:         0
          Total Files:       1488
          Unmodified Files:  1488
        Name:                42f59bbd
        Path:                /var/www/html
        Processing Time:     0:02
        Total Size:          22.695 MiB
        Uploaded:            0 B
    Total Hosts:             1
Events:
  Type    Reason                   Age    From                      Message
  ----    ------                   ----   ----                      -------
  Normal  BackupSession Running    3m8s   BackupSession Controller  Backup job has been created succesfully/sidecar is watching the BackupSession.
  Normal  BackupSession Running    3m8s   BackupSession Controller  Backup job has been created succesfully/sidecar is watching the BackupSession.
  Normal  Host Backup Succeeded    2m44s  Status Updater            backup succeeded for host host-0 of "Deployment"/"wordpress-mysql".
  Normal  Host Backup Succeeded    2m44s  Status Updater            backup succeeded for host host-0 of "Deployment"/"wordpress".
  Normal  BackupSession Succeeded  2m44s  BackupSession Controller  Backup session completed successfully
```

Now, we are going to verify whether the backed up data has been stored in the backend.

**Verify Backup:**

When backup session is completed, Stash will update the respective `Repository` to reflect the latest state of backed up data.

Run the following command to check if a backup snapshot has been stored in the backend,

```console
$ kubectl get repository -n demo gcs-repo
NAME       INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
gcs-repo   true               4                67s                      28m
```

From the output above, we can see that 4 snapshot has been stored in the backend specified by Repository `gcs-repo`.

Now, if we navigate to the GCS bucket, we are going to see backed up data has been stored in `/demo/volumes/<target-kind>/<target-name>` directory.
> Stash forms the directory by concatenating the `spec.backend.gcs.prefix` field of Repository crd and `<target-kind>/<target-name>`.

<figure align="center">
  <img alt="Backup data in GCS Bucket" src="/docs/images/guides/latest/batch-backup/wordpress-data.png">
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