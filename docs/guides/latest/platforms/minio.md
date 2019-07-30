---
title: Minio | Stash
description: Using Stash with TLS secured Minio Server
menu:
  product_stash_0.8.3:
    identifier: platforms-minio
    name: Minio
    parent: platforms
    weight: 40
product_name: stash
menu_name: product_stash_0.8.3
section_menu_id: guides
---

# Using Stash with TLS secured Minio Server

Minio is an open-source object storage server compatible with [AWS S3](https://aws.amazon.com/s3/) cloud storage service. This guide will show you how to use Stash to backup and restore volumes of a Kubernetes workload in [Minio](https://min.io/) backend. Here, we are going to backup a volume of a Deployment into a Minio bucket. Then, we are going to show how to restore this backed up data into a volume of another Deployment.

## Before You Begin

- At first, you need to have a Kubernetes cluster. If you don't already have a cluster, you can create one by using [Minikube](https://github.com/kubernetes/minikube).

- Install `Stash` in your cluster following the steps [here](https://appscode.com/products/stash/0.8.3/setup/install/).

- You should be familiar with the following `Stash` concepts:
  - [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md/)
  - [BackupSession](/docs/concepts/crds/backupsession.md/)
  - [RestoreSession](/docs/concepts/crds/restoresession.md/)
  - [Repository](/docs/concepts/crds/repository.md/)
- You will need a TLS secured Minio server to store backed up data. If you already do not have a Minio server running, deploy one following the tutorial from [here](https://github.com/appscode/third-party-tools/blob/master/storage/minio/README.md). For this tutorial, we have deployed Minio server in `storage` namespace and it is accessible through `minio.storage.svc` dns.

To keep everything isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$ kubectl create ns demo
namespace/demo created
```

> **Note:** YAML files used in this tutorial are stored in [docs/examples/guides/latest/platforms/minio](/docs/examples/guides/latest/platforms/minio) directory of [stashed/doc](https://github.com/stashed/doc) repository.

## Backup the Volume of a Deployment

Here, we are going to deploy a Deployment with a PVC. This Deployment will automatically generate some sample data into the PVC. Then, we are going to backup this sample data using Stash.

### Prepare Workload

At first, let's deploy the workload whose volumes we are going to backup. Here, we are going create a PVC and deploy a Deployment with this PVC.

**Create PVC:**

Below is the YAML of the sample PVC that we are going to create,

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: source-pvc
  namespace: demo
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Let's create the PVC we have shown above,

```console
$ kubectl apply -f ./docs/examples/guides/latest/platforms/minio/pvc.yaml
persistentvolumeclaim/source-pvc created
```

**Deploy Deployment:**

Now, we are going to deploy a Deployment that uses the above PVC. This Deployment will automatically generate sample data (`data.txt` file) in `/source/data` directory where we have mounted the PVC.

Below is the YAML of the Deployment that we are going to create,

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: stash-demo
  name: stash-demo
  namespace: demo
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
            claimName: source-pvc
```

Let's create the Deployment we have shown above.

```console
$ kubectl apply -f ./docs/examples/guides/latest/platforms/minio/deployment.yaml
deployment.apps/stash-demo created
```

Now, wait for the pods of the Deployment to go into the `Running` state.

```console
$ kubectl get pod -n demo
NAME                          READY   STATUS    RESTARTS   AGE
stash-demo-69f9ffbbf7-6wwtr   1/1     Running   0          60s
stash-demo-69f9ffbbf7-88kgj   1/1     Running   0          60s
stash-demo-69f9ffbbf7-q8qld   1/1     Running   0          60s
```

To verify that the sample data has been created in `/source/data` directory, use the following command:

```console
$ kubectl exec -n demo stash-demo-69f9ffbbf7-6wwtr -- cat /source/data/data.txt
sample_data
```

### Prepare Backend

We are going to store our backed up data into an [Minio Bucket](https://min.io/). At first, we need to create a secret with the access credentials to our Minio bucket. Then, we have to create a `Repository` crd that will hold the information about our backend storage. If you want to use a different backend, please read the respective backend configuration doc from [here](/docs/guides/latest/backends/overview.md).

**Create Secret:**

Let's create a secret called `minio-secret` with access credentials to our desired [Minio bucket](https://min.io/),

```console
$ echo -n 'changeit' > RESTIC_PASSWORD
$ echo -n '<your-minio-access-key-id-here>' > AWS_ACCESS_KEY_ID
$ echo -n '<your-minio-secret-access-key-here>' > AWS_SECRET_ACCESS_KEY
$ cat ./directory/of/root/certificate/ca.crt > CA_CERT_DATA
$ kubectl create secret generic -n demo minio-secret \
    --from-file=./RESTIC_PASSWORD \
    --from-file=./AWS_ACCESS_KEY_ID \
    --from-file=./AWS_SECRET_ACCESS_KEY \
    --from-file=./CA_CERT_DATA
secret/minio-secret created
```

Verify that the secret has been created successfully,

```console
$ kubectl get secret -n demo minio-secret -o yaml
```

```yaml
apiVersion: v1
data:
  AWS_ACCESS_KEY_ID: YWRtaW4=
  AWS_SECRET_ACCESS_KEY: Y2hhbmdlaXQ=
  CA_CERT_DATA: <base64 endoded ca.crt data>
  RESTIC_PASSWORD: Y2hhbmdlaXQ=
kind: Secret
metadata:
  creationTimestamp: "2019-07-23T05:32:40Z"
  name: minio-secret
  namespace: demo
  resourceVersion: "7824"
  selfLink: /api/v1/namespaces/demo/secrets/minio-secret
  uid: 31c9310d-dc31-42a3-bc42-f9318290b091
type: Opaque
```

**Create Repository:**

Now, let's create a `Respository` with the information of our desired Minio bucket. Below is the YAML of `Repository` crd we are going to create,

```yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: minio-repo
  namespace: demo
spec:
  backend:
    s3:
      endpoint: "https://minio.storage.svc"
      bucket: minio-bucket
      prefix: /source/data
    storageSecretName: minio-secret
```

Let's create the `Repository` we have shown above,

```console
$ kubectl apply -f ./docs/examples/guides/latest/platforms/minio/repository.yaml
repository.stash.appscode.com/minio-repo created
```

Now, we are ready to backup our sample data into this backend.

### Backup

We have to create a `BackupConfiguration` crd targeting the `stash-demo` Deployment that we have deployed earlier. Stash will inject a sidecar container into the target. It will also create a `CronJob` to take a periodic backup of `/source/data` directory of the target.

**Create BackupConfiguration:**

Below is the YAML of the `BackupConfiguration` crd that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: deployment-backup
  namespace: demo
spec:
  repository:
    name: minio-repo
  schedule: "*/1 * * * *"
  target:
    ref:
      apiVersion: apps/v1
      kind: Deployment
      name: stash-demo
    volumeMounts:
      - name: source-data
        mountPath: /source/data
    directories:
      - /source/data
  retentionPolicy:
    name: "keep-last-5"
    keepLast: 5
    prune: true
```

Here,

- `spec.repository` refers to the `Repository` object `minio-repo` that holds backend [Minio bucket](https://min.io/) information.
- `spec.target.ref` refers to the `stash-demo` Deployment for backup target.
- `spec.target.volumeMounts` specifies a list of volumes and their mountPath that contain the target directories.
- `spec.target.directories` specifies list of directories to backup.

Let's create the `BackupConfiguration` crd we have shown above,

```console
$ kubectl apply -f ./docs/examples/guides/latest/platforms/minio/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/deployment-backup created
```

**Verify Sidecar:**

If everything goes well, Stash will inject a sidecar container into the `stash-demo` Deployment to take backup of `/source/data` directory. Let’s check that the sidecar has been injected successfully,

```console
$ kubectl get pod -n demo
NAME                         READY   STATUS    RESTARTS   AGE
stash-demo-6548cf5cc-7qx9d   2/2     Running   0          42s
stash-demo-6548cf5cc-d26sx   2/2     Running   0          37s
stash-demo-6548cf5cc-f2xbl   2/2     Running   0          40s
```

Look at the pod. It now has 2 containers. If you view the resource definition of this pod, you will see that there is a container named `stash` which is running `run-backup` command.

```yaml
$ kubectl get pod -n demo stash-demo-6548cf5cc-7qx9d -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    stash.appscode.com/last-applied-backupconfiguration-hash: "17185792379662025862"
  creationTimestamp: "2019-07-23T08:52:42Z"
  generateName: stash-demo-6548cf5cc-
  labels:
    app: stash-demo
    pod-template-hash: 6548cf5cc
  name: stash-demo-6548cf5cc-7qx9d
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
      name: default-token-95xvp
      readOnly: true
  - args:
    - run-backup
    - --backup-configuration=deployment-backup
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
    image: suaas21/stash:volumeTemp_linux_amd64
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
      name: default-token-95xvp
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: minikube
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  volumes:
  - name: source-data
    persistentVolumeClaim:
      claimName: source-pvc
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
      secretName: minio-secret
  - name: default-token-95xvp
    secret:
      defaultMode: 420
      secretName: default-token-95xvp
  ...
...
```

**Verify CronJob:**

It will also create a `CronJob` with the schedule specified in `spec.schedule` field of `BackupConfiguration` crd.

Verify that the `CronJob` has been created using the following command,

```console
$ kubectl get cronjob -n demo
NAME                SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
deployment-backup   */1 * * * *   False     0        13s             1m50s
```

**Wait for BackupSession:**

The `deployment-backup` CronJob will trigger a backup on each schedule by creating a `BackpSession` crd. The sidecar container will watch for the `BackupSession` crd. When it finds one, it will take backup immediately.

Wait for the next schedule for backup. Run the following command to watch `BackupSession` crd,

```console
$ watch -n 3 kubectl get backupsession -n demo
Every 3.0s: kubectl get backupsession --all-namespaces                      suaas-appscode: Mon Jul 22 15:01:21 2019

NAMESPACE   NAME                           BACKUPCONFIGURATION   PHASE       AGE
demo        deployment-backup-1563786061   deployment-backup     Succeeded   18s
```

We can see from the above output that the backup session has succeeded. Now, we are going to verify whether the backed up data has been stored in the backend.

**Verify Backup:**

Once a backup is complete, Stash will update the respective `Repository` crd to reflect the backup. Check that the repository `minio-repo` has been updated by the following command,

```console
$ kubectl get repository -n demo
NAME         INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
minio-repo   true        32 B   2                3m40s                    5m18s
```

Now, if we navigate to the Minio Bucket, we are going to see backed up data has been stored in `<bucket name>/source/data` directory as specified by `spec.backend.s3.prefix` field of `Repository` crd.

<figure align="center">
  <img alt="Backup data in Minio Bucket" src="/docs/images/guides/latest/platforms/minio.png">
  <figcaption align="center">Fig: Backup data in Minio Bucket</figcaption>
</figure>

> **Note:** Stash keeps all the backed up data encrypted. So, data in the backend will not make any sense until they are decrypted.

## Restore the Backed up Data

This section will show you how to restore the backed up data from [Minio bucket](https://min.io/) we have taken in earlier section.

**Deploy Deployment:**

We are going to create a new Deployment named `stash-recovered` with a new PVC and restore the backed up data inside it.

Below are the YAMLs of the Deployment and PVC that we are going to create,

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: restore-pvc
  namespace: demo
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: stash-recovered
  name: stash-recovered
  namespace: demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: stash-recovered
  template:
    metadata:
      labels:
        app: stash-recovered
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
            - mountPath: /restore/data
              name: restore-data
      restartPolicy: Always
      volumes:
        - name: restore-data
          persistentVolumeClaim:
            claimName: restore-pvc
```

Let's create the Deployment and PVC we have shown above.

```console
$ kubectl apply -f ./docs/examples/guides/latest/platforms/minio/recovered_deployment.yaml
persistentvolumeclaim/restore-pvc created
deployment.apps/stash-recovered created
```

**Create RestoreSession:**

Now, we need to create a `RestoreSession` crd targeting the `stash-recovered` Deployment.

Below is the YAML of the `RestoreSesion` crd that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: deployment-restore
  namespace: demo
spec:
  repository:
    name: minio-repo
  rules:
    - paths:
        - /source/data/
  target: # target indicates where the recovered data will be stored
    ref:
      apiVersion: apps/v1
      kind: Deployment
      name: stash-recovered
    volumeMounts:
      - name: restore-data
        mountPath: /source/data
```

Here,

- `spec.repository.name` specifies the `Repository` crd that holds the backend information where our backed up data has been stored.
- `spec.target.ref` refers to the target workload where the recovered data will be stored.
- `spec.target.volumeMounts` specifies a list of volumes and their mountPath where the data will be restored.

Let's create the `RestoreSession` crd we have shown above,

```console
$ kubectl apply -f ./docs/examples/guides/latest/platforms/minio/restoresession.yaml
restoresession.stash.appscode.com/deployment-restore created
```

Once, you have created the `RestoreSession` crd, Stash will inject `init-container` into `stash-recovered` Deployment. The Deployment will restart and the `init-container` will restore the desired data on start-up.

**Verify Init-Container:**

Wait until the `init-container` has been injected into the `stash-recovered` Deployment. Let’s describe the Deployment to verify that `init-container` has been injected successfully.

```yaml
$ kubectl describe deployment -n demo stash-recovered
Name:                   stash-recovered
Namespace:              demo
CreationTimestamp:      Tue, 23 Jul 2019 15:05:11 +0600
Labels:                 app=stash-recovered
Selector:               app=stash-recovered
Replicas:               3 desired | 2 updated | 4 total | 3 available | 1 unavailable
StrategyType:           RollingUpdate
...
Pod Template:
  Labels:       app=stash-recovered
  Annotations:  stash.appscode.com/last-applied-restoresession-hash: 9375001205249667775
  Init Containers:
   stash-init:
    Image:      suaas21/stash:volumeTemp_linux_amd64
    Port:       <none>
    Host Port:  <none>
    Args:
      restore
      --restore-session=deployment-restore
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
      /source/data from restore-data (rw)
      /tmp from tmp-dir (rw)
  Containers:
   busybox:
    Image:      busybox
    Port:       <none>
    Host Port:  <none>
    Args:
      sleep
      3600
    Environment:  <none>
    Mounts:
      /restore/data from restore-data (rw)
  Volumes:
   restore-data:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  restore-pvc
    ReadOnly:   false
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
    SecretName:  minio-secret
    Optional:    false
...
```

Notice the `Init-Containers` section. We can see that the init-container `stash-init` has been injected which is running `restore` command.

**Wait for RestoreSession to Succeeded:**

Now, wait for the restore process to complete. You can watch the `RestoreSession` phase using the following command,

```console
$ watch -n 2 kubectl get restoresession -n demo
Every 3.0s: kubectl get restoresession --all-namespaces                 suaas-appscode: Mon Jul 22 18:17:26 2019

NAMESPACE   NAME                 REPOSITORY-NAME   PHASE       AGE
demo        deployment-restore   minio-repo        Succeeded   110s
```

So, we can see from the output of the above command that the restore process has succeeded.

> **Note:** If you want to restore the backed up data inside the same Deployment whose volumes were backed up, you have to remove the corrupted data from the Deployment. Then, you have to create a RestoreSession targeting the Deployment.

**Verify Restored Data:**

In this section, we are going to verify that the desired data has been restored successfully. At first, check if the `stash-recovered` pod of the Deployment has gone into `Running` state by the following command,

```console
$ kubectl get pod -n demo
NAME                               READY   STATUS    RESTARTS   AGE
stash-recovered-6f5c46fdbf-lfthv   1/1     Running   0          2m39s
stash-recovered-6f5c46fdbf-s7rrq   1/1     Running   0          2m15s
stash-recovered-6f5c46fdbf-vn2z8   1/1     Running   0          2m35s
```

Verify that the sample data has been restored in `/restore/data` directory of the `stash-recovered` pod of the Deployment using the following command,

```console
$ kubectl exec -n demo stash-recovered-6f5c46fdbf-s7rrq -- cat /restore/data/data.txt
sample_data
```

## Cleaning Up

To clean up the Kubernetes resources created by this tutorial, run:

```console
kubectl delete -n demo deployment stash-demo
kubectl delete -n demo deployment stash-recovered
kubectl delete -n demo backupconfiguration deployment-backup
kubectl delete -n demo restoresession deployment-restore
kubectl delete -n demo repository minio-repo
kubectl delete -n demo secret minio-secret
kubectl delete -n demo pvc --all
```
