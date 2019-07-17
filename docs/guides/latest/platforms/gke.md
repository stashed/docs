---
title: GKE | Stash
description: Using Stash in Google Kubernetes Engine
menu:
  product_stash_0.8.3:
    identifier: platforms-gke
    name: GKE
    parent: platforms
    weight: 30
product_name: stash
menu_name: product_stash_0.8.3
section_menu_id: guides
---

# Using Stash with Google Kubernetes Engine (GKE)

This guide will show you how to use Stash to backup and restore volumes of a Kubernetes workload in [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/). Here, we are going to backup a volume of a Deployment into [GCS Bucket](https://cloud.google.com/storage/). Then, we are going to show how to restore this backed up data into a volume of another Deployment.

## Before You Begin

- At first, you need to have a Kubernetes cluster in the Google Cloud Platform. If you don't already have a cluster, create one from [here](https://console.cloud.google.com/kubernetes/).

- Install `Stash` in your cluster following the steps [here](https://appscode.com/products/stash/0.8.3/setup/install/).

- You should be familiar with the following `Stash` concepts:
  - [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md/)
  - [BackupSession](/docs/concepts/crds/backupsession.md/)
  - [RestoreSession](/docs/concepts/crds/restoresession.md/)
  - [Repository](/docs/concepts/crds/repository.md/)
- You will need a [GCS Bucket](https://console.cloud.google.com/storage/) and [GCE persistent disk](https://console.cloud.google.com/compute/disks). GCE persistent disk must be in the same GCE project and zone as the cluster.

To keep everything isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$ kubectl create ns demo
namespace/demo created
```

>**Note:** YAML files used in this tutorial are stored in  [docs/examples/guides/latest/platforms/gke](/docs/examples/guides/latest/platforms/gke) directory of [stashed/doc](https://github.com/stashed/doc) repository.

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
$ kubectl apply -f ./docs/examples/guides/latest/platforms/gke/pvc.yaml
persistentvolumeclaim/source-pvc created
```

**Deploy Deployment:**

Now, we will deploy a Deployment that uses the above PVC. This Deployment will automatically generate sample data (`data.txt` file) in `/source/data` directory where we have mounted the PVC.

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
  replicas: 1
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
$ kubectl apply -f ./docs/examples/guides/latest/platforms/gke/deployment.yaml
deployment.apps/stash-demo created
```

Now, wait for the pod of the Deployment to go into the `Running` state.

```console
$ kubectl get pod -n demo
NAME                          READY   STATUS    RESTARTS   AGE
stash-demo-798987998b-qz6bt   1/1     Running   0          45s
```

Verify that the sample data has been created in `/source/data` directory using the following command,

```console
$ kubectl exec -n demo stash-demo-798987998b-qz6bt -- cat /source/data/data.txt
sample_data
```

### Prepare Backend

We are going to store our backed up data into an [GCS bucket](https://cloud.google.com/storage/). At first, we need to create a secret with the access credentials to our GCS bucket. Then, we have to create a `Repository` crd that will hold the information about our backend storage. If you want to use a different backend, please read the respective backend configuration doc from [here](https://appscode.com/products/stash/0.8.3/guides/backends/overview/).

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
secret "gcs-secret" created
```

Verify that the secret has been created successfully,

```console
$ kubectl get secret -n demo gcs-secret -o yaml
```

```yaml
apiVersion: v1
data:
  GOOGLE_PROJECT_ID: <base64 encoded google project id>
  GOOGLE_SERVICE_ACCOUNT_JSON_KEY: <base64 encoded google service account json key>
  RESTIC_PASSWORD: Y2hhbmdlaXQ=
kind: Secret
metadata:
  creationTimestamp: "2019-07-22T08:49:20Z"
  name: gcs-secret
  namespace: demo
  resourceVersion: "33237"
  selfLink: /api/v1/namespaces/demo/secrets/gcs-secret
  uid: 98f12a14-ac5d-11e9-8128-42010a800069
type: Opaque
```

**Create Repository:**

Now, let's create a `Respository` with the information of our desired GCS bucket. Below is the YAML of `Repository` crd we are going to create,

```yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: gcs-repo
  namespace: demo
spec:
  backend:
    gcs:
      bucket: appscode-qa
      prefix: /source/data
    storageSecretName: gcs-secret
```

Now, we are ready to backup our sample data into this backend.

```console
$ kubectl apply -f ./docs/examples/guides/latest/platforms/gke/repository.yaml
repository.stash.appscode.com/gcs-repo created
```

Now, we are ready to backup our volumes to the desired backend.

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
    name: gcs-repo
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
    name: 'keep-last-5'
    keepLast: 5
    prune: true
```

Here,

- `spec.repository` refers to the `Repository` object `gcs-repo` that holds backend [GCS bucket](https://cloud.google.com/storage/) information.
- `spec.target.ref` refers to the `stash-demo` Deployment for backup target.
- `spec.target.volumeMounts` specifies a list of volumes and their mountPath that contain the target directories.
- `spec.target.directories`  specifies list of directories to backup.

Let's create the `BackupConfiguration` crd we have shown above,

```console
$ kubectl apply -f ./docs/examples/guides/latest/platforms/gke/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/deployment-backup created
```

**Verify Sidecar:**

If everything goes well, Stash will inject a sidecar container into the `stash-demo` Deployment to take backup of `/source/data` directory. Let’s check that the sidecar has been injected successfully,

```console
$ kubectl get pod -n demo
NAME                         READY   STATUS    RESTARTS   AGE
stash-demo-998db88b7-gqnzq   2/2     Running   0          48s
```

Look at the pod. It now has 2 containers. If you view the resource definition of this pod, you will see that there is a container named `stash` which is running `run-backup` command.

```yaml
$ kubectl get pod -n demo stash-demo-998db88b7-gqnzq -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    stash.appscode.com/last-applied-backupconfiguration-hash: "16695904176112618119"
  creationTimestamp: "2019-07-22T09:00:31Z"
  generateName: stash-demo-998db88b7-
  labels:
    app: stash-demo
    pod-template-hash: 998db88b7
  name: stash-demo-998db88b7-gqnzq
  namespace: demo
  resourceVersion: "35168"
  selfLink: /api/v1/namespaces/demo/pods/stash-demo-998db88b7-gqnzq
  uid: 28a95f65-ac5f-11e9-8128-42010a800069
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
      name: default-token-sszw7
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
      name: default-token-sszw7
      readOnly: true
  dnsPolicy: ClusterFirst
  nodeName: gke-stash-gke-default-pool-9e2692ba-2hb9
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
      secretName: gcs-secret
  - name: default-token-sszw7
    secret:
      defaultMode: 420
      secretName: default-token-sszw7
  ...
...
```

**Verify CronJob:**

It will also create a `CronJob` with the schedule specified in `spec.schedule` field of `BackupConfiguration` crd.

Verify that the `CronJob` has been created using the following command,

```console
$ kubectl get cronjob -n demo
NAME                SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
deployment-backup   */1 * * * *   False     0        13s             2m42s
```

**Wait for BackupSession:**

The `deployment-backup` CronJob will trigger a backup on each schedule by creating a `BackpSession` crd. The sidecar container will watch for the `BackupSession` crd. When it finds one, it will take backup immediately.

Wait for the next schedule for backup. Run the following command to watch `BackupSession` crd,

```console
$ watch -n 3 kubectl get backupsession -n demo
Every 3.0s: kubectl get backupsession --all-namespaces                      suaas-appscode: Mon Jul 22 15:01:21 2019

NAMESPACE   NAME                           BACKUPCONFIGURATION   PHASE       AGE
demo        deployment-backup-1763886021   deployment-backup     Succeeded   13s
```

We can see from the above output that the backup session has succeeded. Now, we are going to verify whether the backed up data has been stored in the backend.

**Verify Backup:**

Once a backup is complete, Stash will update the respective `Repository` crd to reflect the backup. Check that the repository `gcs-repo` has been updated by the following command,

```console
$ kubectl get repository -n demo
NAME       INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
gcs-repo   true        8 B    1                3s                       1m15s
```

Now, if we navigate to the GCS Bucket, we will see backed up data has been stored in `<bucket name>/source/data` directory as specified by `spec.backend.gcs.prefix` field of `Repository` crd.

<figure align="center">
  <img alt="Backup data in GCS Bucket" src="/docs/images/guides/latest/platforms/gke.png">
  <figcaption align="center">Fig: Backup data in GCS Bucket</figcaption>
</figure>

>**Note:** Stash keeps all the backed up data encrypted. So, data in the backend will not make any sense until they are decrypted.

## Restore the Backed Up Data

This section will show you how to restore the backed up data from [GCS bucket](https://cloud.google.com/storage/) we have taken in the earlier section.

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
  replicas: 1
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
$ kubectl apply -f ./docs/examples/guides/latest/platforms/gke/recovered_deployment.yaml
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
    name: gcs-repo
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
$ kubectl apply -f ./docs/examples/guides/latest/platforms/gke/restoresession.yaml
restoresession.stash.appscode.com/deployment-restore created
```

Once, you have created the `RestoreSession` crd, Stash will inject `init-container` into `stash-recovered` Deployment. The Deployment will restart and the `init-container` will restore the desired data on start-up.

**Verify Init-Container:**

Wait until the `init-container` has been injected into the `stash-recovered` Deployment. Let’s describe the Deployment to verify that `init-container` has been injected successfully.

```yaml
$ kubectl describe deployment -n demo stash-recovered
Name:                   stash-recovered
Namespace:              demo
Labels:                 app=stash-recovered
Selector:               app=stash-recovered
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
...
Pod Template:
  Labels:       app=stash-recovered
  Annotations:  stash.appscode.com/last-applied-restoresession-hash: 11756285575718611872
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
    SecretName:  gcs-secret
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
demo        deployment-restore   gcs-repo          Succeeded   4m
```

So, we can see from the output of the above command that the restore process has succeeded.

> **Note:** If you want to restore the backed up data inside the same Deployment whose volumes were backed up, you have to remove the corrupted data from the Deployment. Then, you have to create a RestoreSession targeting the Deployment.

**Verify Restored Data:**

In this section, we will verify that the desired data has been restored successfully. At first, check if the `stash-recovered` pod of the Deployment has gone into `Running` state by the following command,

```console
$ kubectl get pod -n demo 
NAME                               READY   STATUS    RESTARTS   AGE
stash-recovered-7876d7bbbb-w6t8f   1/1     Running   0          4m58s
```

Verify that the sample data has been restored in `/restore/data` directory of the `stash-recovered` pod of the Deployment using the following command,

```console
$ kubectl exec -n demo stash-recovered-7876d7bbbb-w6t8f -- cat /restore/data/data.txt
sample_data
```

# Cleaning Up

To clean up the Kubernetes resources created by this tutorial, run:

```console
kubectl delete -n demo deployment stash-demo
kubectl delete -n demo deployment stash-recovered
kubectl delete -n demo backupconfiguration deployment-backup
kubectl delete -n demo restoresession deployment-restore
kubectl delete -n demo repository gcs-repo
kubectl delete -n demo secret gcs-secret
kubectl delete -n demo pvc --all
```