# Backup and Restore Deployment's Data

This guide will show you how to use Stash to backup and restore Deployment's data.

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [Minikube](https://github.com/kubernetes/minikube).

- Install `Stash` in your cluster following the steps [here](https://appscode.com/products/stash/0.8.3/setup/install/).

- You should be familiar with the following `Stash` concepts:
  - [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md/)
  - [BackupSession](/docs/concepts/crds/backupsession.md/)
  - [RestoreSession](/docs/concepts/crds/restoresession.md/)
  - [Repository](/docs/concepts/crds/repository.md/)

To keep everything isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$ kubectl create ns demo
namespace/demo created
```

>**Note:** YAML files used in this tutorial are stored in  [docs/examples/guides/latest/workloads](/docs/examples/guides/latest/workloads) directory of [stashed/stash](https://github.com/stashed/stash) repository.

## Backup Deployment's Data

This section will show you how to use Stash to backup Deployment's data. Here, we are going to deploy a Deployment with a PVC and generate some sample data in it. Then, we will backup this sample data using Stash.

### Deploy workload

At first, we will create a PVC then we will create a Deployment that will use this PVC.

**Create PVC:**

Below is the YAML of the sample PVC that we are going to create,

```console
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

Let's create the PVC we have shown above,

```console
$ kubectl apply -f ./pvc.yaml 
persistentvolumeclaim/stash-sample-data created
```

**Deploy Deployment:**

Now, we will deploy a Deployment that uses the above PVC. This Deployment will automatically generate sample data (`sample-file.txt` file) in /`source/data` directory where we have mounted the desired PVC.

Below is the YAML of the Deployment that we are going to create,

```console
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
      - args: ["touch source/data/sample-file.txt && sleep 3000"]
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
```

Let's create the Deployment we have shown above.

```console
$ kubectl apply -f ./deployment.yaml 
deployment.apps/stash-demo created
```

Now, wait for Deployment’s pod to go into the `Running` state.

```console
$ kubectl get pod -n demo 
NAME                         READY   STATUS    RESTARTS   AGE
stash-demo-8cfcbcc89-2z6mq   1/1     Running   0          30s
stash-demo-8cfcbcc89-j9wbc   1/1     Running   0          30s
stash-demo-8cfcbcc89-q8xfd   1/1     Running   0          30s
```

Verify that the sample data has been created in `/source/data` directory using the following command,

```console
$ kubectl exec -n demo stash-demo-8cfcbcc89-2z6mq -- ls -R /source/data
/source/data:
sample-file.txt
```

### Prepare Backend

We are going to store our backed up data into a GCS bucket. At first, we need to create a secret with GCS credentials then we need to create a Repository crd. If you want to use a different backend, please read the respective backend configuration doc from [here](/docs/guides/latest/backends/overview.md).

**Create Secret:**

At first, we need to create a secret with access credentials to our desired GCS bucket,

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

Now, we are ready to backup our workload's data to our desired backend.

**Create Repository:**

Now, create a `Respository` using this secret.

Below is the YAML of `Repository` crd we are going to create,

```console
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: gcs-repo
  namespace: demo
spec:
  backend:
    gcs:
      bucket: appscode-qa
      prefix: /source/data/sample-deployment
    storageSecretName: gcs-secret
```

Let's create the Repository we have shown above,

```console
$ kubectl apply -f ./repository.yaml
repository.stash.appscode.com/gcs-repo created
```

Now, we are ready to backup our volumes to our desired backend.

### Backup

We have to create a `BackupConfiguration` crd targeting the `stash-demo` Deployment that we have deployed earlier. Then Stash will inject a sidecar container to the target. It will also, create a `CronJob` to take periodic backup of `/source/data` directory of the target.

**Create BackupConfiguration:**

Below is the YAML of the `BackupConfiguration` crd that we are going to create,

```console
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

- `spec.repository` refers to the `gcs-repo` GCP Backend.
- `spec.schedule` is a cron expression indicates that `BackupSession` will be created at 1 minute interval.
- `spec.target.ref` refers to the target workload that was created for `stash-demo` Deploymnet.

Let's create the `BackupConfiguration` crd we have shown above,

```console
$ kubectl apply -f ./backupconfiguration.yaml
backupconfiguration.stash.appscode.com/deployment-backup created
```

**Verify Sidecar:**

If everything goes well, Stash will inject a sidecar container into the `stash-demo` deployment to take periodic backup. Let’s check that sidecar has been injected successfully,

```console
$ kubectl get pod -n demo 
NAME                          READY   STATUS        RESTARTS   AGE
stash-demo-856896bd95-4gfbh   2/2     Running       0          12s
stash-demo-856896bd95-njr8x   2/2     Running       0          17s
stash-demo-856896bd95-ttbq4   2/2     Running       0          15s

```

Look at the pod. It now has 2 containers. If you view the resource definition of this pod, you will see that there is a container named `stash` which running backup command.

```console
$ kubectl get pod -n demo stash-demo-856896bd95-4gfbh -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    stash.appscode.com/last-applied-backupconfiguration-hash: "16695904176112618119"
  creationTimestamp: "2019-06-25T05:51:18Z"
  generateName: stash-demo-856896bd95-
  labels:
    app: stash-demo
    pod-template-hash: 856896bd95
  name: stash-demo-856896bd95-4gfbh
  namespace: demo
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: stash-demo-856896bd95
    uid: b6ab0568-9704-11e9-975f-080027cababb
  resourceVersion: "16335"
  selfLink: /api/v1/namespaces/demo/pods/stash-demo-856896bd95-4gfbh
  uid: 4047c78e-970d-11e9-975f-080027cababb
spec:
  containers:
  - args:
    - touch source/data/sample-file.txt && sleep 3000
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
      name: default-token-4tzgg
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
    image: suaas21/stash:vs_linux_amd64
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
      name: default-token-4tzgg
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
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
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
  - name: default-token-4tzgg
    secret:
      defaultMode: 420
      secretName: default-token-4tzgg
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2019-06-25T05:51:18Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2019-06-25T05:51:20Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2019-06-25T05:51:20Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2019-06-25T05:51:18Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://1b849112500fe7334134f4b4a9c113302cf845cd5bfb79c66cb752b23875323e
    image: busybox:latest
    imageID: docker-pullable://busybox@sha256:c94cf1b87ccb80f2e6414ef913c748b105060debda482058d2b8d0fce39f11b9
    lastState: {}
    name: busybox
    ready: true
    restartCount: 0
    state:
      running:
        startedAt: "2019-06-25T05:51:19Z"
  - containerID: docker://da95746aa508d135b59ce13d6711c20e30f5626e2558ee3b9a068cf08be5a672
    image: suaas21/stash:vs_linux_amd64
    imageID: docker-pullable://suaas21/stash@sha256:8b6afb1f6c6cd4139f6892e94367ae9462d76dca19a028e717e53afe1944250a
    lastState: {}
    name: stash
    ready: true
    restartCount: 0
    state:
      running:
        startedAt: "2019-06-25T05:51:19Z"
  hostIP: 10.0.2.15
  phase: Running
  podIP: 172.17.0.10
  qosClass: BestEffort
  startTime: "2019-06-25T05:51:18Z"
```

**Verify CronJob:**

It will also create a `CronJob` with the schedule specified in `spec.schedule` field of `BackupConfiguration` crd.

Verify that the `CronJob` has been created using the following command,

```console
$ kubectl get cronjob -n demo
NAME                SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
deployment-backup   */1 * * * *   False     0        35s             64s
```

**Wait for BackupSession:**

The `deployment-backup` CronJob will trigger a backup on each schedule by creating a `BackpSession` crd. The sidecar container will watches for the `BackupSession` crd. Once it found one, it will take backup immediately.

Wait for a schedule to appear. Run the following command to watch `BackupSession` crd,

```console
$ watch -n 2 kubectl get backupsession -n demo 
Every 1.0s: kubectl get backupsession -n demo     suaas-appscode: Mon Jun 24 10:23:08 2019

NAME                           BACKUPCONFIGURATION   PHASE       AGE
deployment-backup-1561350065   deployment-backup     Running     30s
deployment-backup-1561350125   deployment-backup     Succeeded   63s
```

We can see above that the backup session has succeeded. Now, we will verify that the backed up data has been stored in the local backend.

**Verify Backup:**

Once a backup is complete, Stash will update the respective `Repository` crd to reflect the backup. Check that the repository `gcs-repo` has been updated by the following command,

```console
$ kubectl get repository -n demo gcs-repo
NAME       INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
gcs-repo   true        0 B    5                58s                      18m
```

Now, if we navigate to the GCS bucket, we will see backed up data has been stored in `source/data/sample-deployment` directory as specified by `spec.backend.gcs.prefix` field of Repository crd.

<figure align="center">
  <img alt="Backup data in GCS Bucket" src="/docs/images/v1beta1/backends/workloads/gcs_bucket_dep.png">
  <figcaption align="center">Fig: Backup data in GCS Bucket</figcaption>
</figure>

>**Note:** Stash keeps all the backed up data encrypted. So, data in the backend will not make any sense until they are decrypted.


## Restore Deployment's Data

This section will show you how to restore the backed up data from the backend we have taken in earlier section.

**Deploy Deployment:**

We are going to create a new Deployment named `stash-recovered` and restore the inside it.

Below is the YAML of the Deployment that we are going to create,

```console
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: demo-pvc
  namespace: demo
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 1Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: stash-demo
  name: stash-recovered
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
      - args:
        - sleep
        - "3600"
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
          claimName: demo-pvc
```

Let's create the deployment we have shown above.

```console
$ kubectl apply -f ./recovered_deployment.yaml 
persistentvolumeclaim/demo-pvc created
deployment.apps/stash-recovered created
```

Now, wait for deployment’s pod to go into the `Running` state.

```console
$ kubectl get pod -n demo 
NAME                               READY   STATUS    RESTARTS   AGE
stash-recovered-676bd87957-7bcfg   1/1     Running   0          31s
stash-recovered-676bd87957-pv4hg   1/1     Running   0          31s
stash-recovered-676bd87957-q2hbl   1/1     Running   0          31s

```

**Create RestoreSession:**

Now, we need to create a `RestoreSession` crd targeting the `stash-recovered` Deployment to restore the inside it.

Below is the YAML of the `RestoreSesion` crd that we are going to create,

```console
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
        - /source/data
  target: # target indicates where the recovered data will be stored
    ref:
      apiVersion: apps/v1
      kind: Deployment
      name: stash-recovered
    volumeMounts:
      - name:  source-data
        mountPath:  /source/data
```

Here,

`spec.repository.name` specifies the `Repository` crd that holds the backend information where our backed up data has been stored.

`spec.target.ref` refers to the target workload where the recovered data will be stored.

Let's create the `RestoreSession` crd we have shown above,

```console
$ kubectl apply -f ./restoresession.yaml
restoresession.stash.appscode.com/deployment-restore created
```

Once, you have created the `RestoreSession` crd, Stash will inject `init-container` to `stash-recovered` Deployment. The Deployment will restart and the `init-container` will recovered on start-up.

**Verify Init-Container:**

Wait until the `init-container` has been injected to the `stash-recovered` Deployment's pod, Run the following command to describe the `stash-recovered` Deployment's pod,

```console
$ kubectl describe pod -n demo stash-recovered-867688ddd5-rfsw4 
Name:               stash-recovered-867688ddd5-rfsw4
Namespace:          demo
Priority:           0
PriorityClassName:  <none>
Node:               minikube/10.0.2.15
Start Time:         Tue, 25 Jun 2019 12:35:49 +0600
Labels:             app=stash-demo
                    pod-template-hash=867688ddd5
Annotations:        stash.appscode.com/last-applied-restoresession-hash: 18120714276586222772
Status:             Running
IP:                 172.17.0.9
Controlled By:      ReplicaSet/stash-recovered-867688ddd5
Init Containers:
  stash-init:
    Container ID:  docker://5676ef7ba4d1accb1f53a8ae81d7ce523ac08d4cdc53bf66e573be8cb1e405ec
    Image:         suaas21/stash:vs_linux_amd64
    Image ID:      docker-pullable://suaas21/stash@sha256:8b6afb1f6c6cd4139f6892e94367ae9462d76dca19a028e717e53afe1944250a
    Port:          <none>
    Host Port:     <none>
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
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Tue, 25 Jun 2019 12:35:50 +0600
      Finished:     Tue, 25 Jun 2019 12:36:06 +0600
    Ready:          True
    Restart Count:  0
    Environment:
      NODE_NAME:   (v1:spec.nodeName)
      POD_NAME:   stash-recovered-867688ddd5-rfsw4 (v1:metadata.name)
    Mounts:
      /etc/stash/repository/secret from stash-secret-volume (rw)
      /source/data from source-data (rw)
      /tmp from tmp-dir (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-4tzgg (ro)
Containers:
  busybox:
    Container ID:  docker://40c19ff4c31933815a068edb6016cf73f95a0f1ac909b0eeb4eb2138d459f237
    Image:         busybox
    Image ID:      docker-pullable://busybox@sha256:c94cf1b87ccb80f2e6414ef913c748b105060debda482058d2b8d0fce39f11b9
    Port:          <none>
    Host Port:     <none>
    Args:
      sleep
      3600
    State:          Running
      Started:      Tue, 25 Jun 2019 12:36:06 +0600
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /source/data from source-data (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-4tzgg (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  source-data:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  demo-pvc
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
  default-token-4tzgg:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-4tzgg
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  54s   default-scheduler  Successfully assigned demo/stash-recovered-867688ddd5-rfsw4 to minikube
  Normal  Pulled     53s   kubelet, minikube  Container image "suaas21/stash:vs_linux_amd64" already present on machine
  Normal  Created    53s   kubelet, minikube  Created container stash-init
  Normal  Started    53s   kubelet, minikube  Started container stash-init
  Normal  Pulled     37s   kubelet, minikube  Container image "busybox" already present on machine
  Normal  Created    37s   kubelet, minikube  Created container busybox
  Normal  Started    37s   kubelet, minikube  Started container busybox
```

You will see that `stash-init` container has been injected.

**Wait for RestoreSession to Succeeded:**

Run the following command to watch RestoreSession phase,

```console
$ watch -n 2 kubectl get restoresession -n demo
Every 5.0s: kubectl get restoresession -n demo           suaas-appscode: Mon Jun 24 10:33:57 2019

NAME                 REPOSITORY-NAME   PHASE       AGE
deployment-restore   gcs-repo          Running     30s
deployment-restore   gcs-repo          Succeeded   2m56s
```

So, we can see from the output of the above command that the restore process succeeded.

**Verify Restored Data:**

In this section, we will verify that the desired data has been restored successfully.

At first, check if the `stash-recovered` Deployment's pod has gone into Running state after successful `stash-init` container injection by the following command,

```console
$ kubectl get pod -n demo
NAME                               READY   STATUS    RESTARTS   AGE
stash-recovered-867688ddd5-67xr8   1/1     Running   0          21m
stash-recovered-867688ddd5-rfsw4   1/1     Running   0          21m
stash-recovered-867688ddd5-zswhs   1/1     Running   0          22m
```

Verify that the sample data has been restored in `/source/data` directory of the `stash-recovered` deployment's pod using the following command,

```console
$ kubectl exec -n demo stash-recovered-867688ddd5-67xr8 -- ls -R /source/data
/source/data:
sample-file.txt
```

# Cleaning Up

To clean up the Kubernetes resources created by this tutorial, run:

```console
kubectl delete -n demo deployment stash-demo
kubectl delete -n demo deployment stash-recovered
kubectl delete -n demo backupconfiguration deployment-backup
kubectl delete -n demo restoresession deployment-restore
kubectl delete -n demo repository gcs-repo
kubectl delete -n demo pvc --all
```