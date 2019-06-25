# Backup and Restore Statefulset's Data

This guide will show you how to use Stash to backup and restore Statefulset's data.

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

## Backup Statefulset's Data

This section will show you how to use Stash to backup Statefulset's data. Here, we are going to deploy a Statefulset with a PVC and generate some sample data in it. Then, we will backup this sample data using Stash.

**Deploy Statefulset:**

Now, We will deploy a Statefulset. This Statefulset will automatically generate sample data in `/source/data` directory.

Below is the YAML of the Deployment that we are going to create,

```console
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
  name: stash-demo
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
        command: ["/bin/sh", "-c","touch /source/data/$(POD_NAME).txt && sleep 3000"]
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

Let's create the Statefulset we have shown above.

```console
$ kubectl apply -f ./statefulset.yaml 
service/headless created
statefulset.apps/stash-demo created
```

Now, wait for Statefulset’s pod to go into the `Running` state.

```console
$ kubectl get pod -n demo
NAME           READY   STATUS    RESTARTS   AGE
stash-demo-0   1/1     Running   0          42s
stash-demo-1   1/1     Running   0          40s
stash-demo-2   1/1     Running   0          36s
```

Verify that the sample data has been generated in `/source/data` directory for `stash-demo-0` , `stash-demo-1` and `stash-demo-2` pod respectively using the following command,

```console
$ kubectl exec -n demo stash-demo-0 -- ls -R /source/data
/source/data:
stash-demo-0.txt
$ kubectl exec -n demo stash-demo-1 -- ls -R /source/data
/source/data:
stash-demo-1.txt
$ kubectl exec -n demo stash-demo-1 -- ls -R /source/data
/source/data:
stash-demo-1.txt
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
      prefix: /source/data/sample-statefulset
    storageSecretName: gcs-secret
```

Let's create the Repository we have shown above,

```console
$ kubectl apply -f ./repository.yaml
repository.stash.appscode.com/gcs-repo created
```

Now, we are ready to backup our volumes to our desired backend.

### Backup

We have to create a `BackupConfiguration` crd targeting the `stash-demo` Statefulset that we have deployed earlier. Then Stash will inject a sidecar container to the target. It will also, create a `CronJob` to take periodic backup of `/source/data` directory of the target.

**Create BackupConfiguration:**

Below is the YAML of the `BackupConfiguration` crd that we are going to create,

```console
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: ss-backup
  namespace: demo
spec:
  repository:
    name: gcs-repo
  schedule: "*/1 * * * *"
  target:
    ref:
      apiVersion: apps/v1
      kind: StatefulSet
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
- `spec.target.ref` refers to the target workload that was created for `stash-demo` Statefulset.

Let's create the `BackupConfiguration` crd we have shown above,

```console
$ kubectl apply -f ./backupconfiguration.yaml
backupconfiguration.stash.appscode.com/ss-backup created
```

**Verify Sidecar:**

If everything goes well, Stash will inject a sidecar container into the `stash-demo` Statefulset to take periodic backup. Let’s check that sidecar has been injected successfully,

```console
$ kubectl get pod -n demo
NAME           READY   STATUS    RESTARTS   AGE
stash-demo-0   2/2     Running   0          5s
stash-demo-1   2/2     Running   0          42s
stash-demo-2   2/2     Running   0          76s
```

Look at the pod. It now has 2 containers. If you view the resource definition of this pod, you will see that there is a container named `stash` which running backup command.

```console
$ kubectl get pod -n demo stash-demo-0 -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    stash.appscode.com/last-applied-backupconfiguration-hash: "9136633043451830730"
  creationTimestamp: "2019-06-25T11:50:49Z"
  generateName: stash-demo-
  labels:
    app: stash-demo
    controller-revision-hash: stash-demo-6d887c7b6f
    statefulset.kubernetes.io/pod-name: stash-demo-0
  name: stash-demo-0
  namespace: demo
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: StatefulSet
    name: stash-demo
    uid: b1697215-973c-11e9-975f-080027cababb
  resourceVersion: "45602"
  selfLink: /api/v1/namespaces/demo/pods/stash-demo-0
  uid: 7a22614a-973f-11e9-975f-080027cababb
spec:
  containers:
  - command:
    - /bin/sh
    - -c
    - touch /source/data/$(POD_NAME).txt && sleep 3000
    env:
    - name: POD_NAME
      valueFrom:
        fieldRef:
          apiVersion: v1
          fieldPath: metadata.name
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
    - --backup-configuration=ss-backup
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
  hostname: stash-demo-0
  nodeName: minikube
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  subdomain: headless
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
      claimName: source-data-stash-demo-0
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
    lastTransitionTime: "2019-06-25T11:50:50Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2019-06-25T11:50:52Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2019-06-25T11:50:52Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2019-06-25T11:50:49Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://689851a16d318d475bd82d2f4d1118c4b4f40555de6768b4bbb6ed4dabe29377
    image: busybox:latest
    imageID: docker-pullable://busybox@sha256:c94cf1b87ccb80f2e6414ef913c748b105060debda482058d2b8d0fce39f11b9
    lastState: {}
    name: busybox
    ready: true
    restartCount: 0
    state:
      running:
        startedAt: "2019-06-25T11:50:51Z"
  - containerID: docker://aaca98dd37831d86dd30cffafbab72605030187a2cfe15134c3f1be54bb58086
    image: suaas21/stash:vs_linux_amd64
    imageID: docker-pullable://suaas21/stash@sha256:8b6afb1f6c6cd4139f6892e94367ae9462d76dca19a028e717e53afe1944250a
    lastState: {}
    name: stash
    ready: true
    restartCount: 0
    state:
      running:
        startedAt: "2019-06-25T11:50:51Z"
  hostIP: 10.0.2.15
  phase: Running
  podIP: 172.17.0.4
  qosClass: BestEffort
  startTime: "2019-06-25T11:50:50Z"
```

**Verify CronJob:**

It will also create a `CronJob` with the schedule specified in `spec.schedule` field of `BackupConfiguration` crd.

Verify that the `CronJob` has been created using the following command,

```console
$ kubectl get backupconfiguration -n  demo
NAME        TASK   SCHEDULE      PAUSED   AGE
ss-backup          */1 * * * *            3m41s
```

**Wait for BackupSession:**

The `ss-backup` CronJob will trigger a backup on each schedule by creating a `BackpSession` crd. The sidecar container will watches for the `BackupSession` crd. Once it found one, it will take backup immediately.

Wait for a schedule to appear. Run the following command to watch `BackupSession` crd,

```console
$ watch -n 2 kubectl get backupsession -n demo
Every 5.0s: kubectl get bs -n demo                               suaas-appscode: Tue Jun 25 17:54:41 2019

NAME                   BACKUPCONFIGURATION   PHASE       AGE
ss-backup-1561463528   ss-backup             Running     2m33s
ss-backup-1561463408   ss-backup             Succeeded   4m33s
```

We can see above that the backup session has succeeded. Now, we will verify that the backed up data has been stored in the local backend.

**Verify Backup:**

Once a backup is complete, Stash will update the respective `Repository` crd to reflect the backup. Check that the repository `gcs-repo` has been updated by the following command,

```console
$ kubectl get repository -n demo 
NAME       INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
gcs-repo   true        0 B    12              103s                     22m
```

Now, if we navigate to the GCS bucket, we will see backed up data has been stored in `source/data/sample-statefulset` directory as specified by `spec.backend.gcs.prefix` field of Repository crd.

<figure align="center">
  <img alt="Backup data in GCS Bucket" src="/docs/images/v1beta1/backends/workloads/gcs_bucket_ss.png">
  <figcaption align="center">Fig: Backup data in GCS Bucket</figcaption>
</figure>

>**Note:** Stash keeps all the backed up data encrypted. So, data in the backend will not make any sense until they are decrypted.

## Restore Statefulset's Data

This section will show you how to restore the backed up data from the backend we have taken in earlier section.

**Deploy Statefulset:**

We are going to create a new Statefulset named `stash-recovered` and restore the inside it.

Below is the YAML of the Statefulset that we are going to create,

```console
apiVersion: v1
kind: Service
metadata:
  name: re_headless
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
  name: stash-recovered
  namespace: demo
  labels:
    app: stash-demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: stash-demo
  serviceName: re_headless
  template:
    metadata:
      labels:
        app: stash-demo
    spec:
      containers:
        - name: busybox
          image: busybox
          command:
            - sleep
            - '3600'
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

Let's create the Statefulset we have shown above.

```console
$ kubectl apply -f ./recovered_statefulset.yaml 
service/re-headless created
statefulset.apps/stash-recovered created
```

Now, wait for Statefulset’s pod to go into the `Running` state.

```console
$ kubectl get pod -n demo
NAME                READY   STATUS    RESTARTS   AGE
stash-recovered-0   1/1     Running   0          37s
stash-recovered-1   1/1     Running   0          35s
stash-recovered-2   1/1     Running   0          32s
```

**Create RestoreSession:**

Now, we need to create a `RestoreSession` crd targeting the `stash-recovered` Statefulset to restore the inside it.

Below is the YAML of the `RestoreSesion` crd that we are going to create,

```console
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: ss-restore
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
restoresession.stash.appscode.com/ss-restore created
```

Once, you have created the `RestoreSession` crd, Stash will inject `init-container` to `stash-recovered` Statefulset. The Statefulset will restart and the `init-container` will recovered on start-up.

**Verify Init-Container:**

Wait until the `init-container` has been injected to the `stash-recovered` Statefulset's pod, Run the following command to describe the `stash-recovered` statefulset's pod,

```console
$ kubectl describe pod -n demo stash-recovered-0
Name:               stash-recovered-0
Namespace:          demo
Priority:           0
PriorityClassName:  <none>
Node:               minikube/10.0.2.15
Start Time:         Tue, 25 Jun 2019 18:25:24 +0600
Labels:             app=stash-demo
                    controller-revision-hash=stash-recovered-6858d6fb9
                    statefulset.kubernetes.io/pod-name=stash-recovered-0
Annotations:        stash.appscode.com/last-applied-restoresession-hash: 10309464337907785627
Status:             Running
IP:                 172.17.0.8
Controlled By:      StatefulSet/stash-recovered
Init Containers:
  stash-init:
    Container ID:  docker://bc76efd070277f0cec25bcc0f7f9c516008d279da3ec67e4fa45b50a86d3059b
    Image:         suaas21/stash:vs_linux_amd64
    Image ID:      docker-pullable://suaas21/stash@sha256:8b6afb1f6c6cd4139f6892e94367ae9462d76dca19a028e717e53afe1944250a
    Port:          <none>
    Host Port:     <none>
    Args:
      restore
      --restore-session=ss-restore
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
      Started:      Tue, 25 Jun 2019 18:25:26 +0600
      Finished:     Tue, 25 Jun 2019 18:25:52 +0600
    Ready:          True
    Restart Count:  0
    Environment:
      NODE_NAME:   (v1:spec.nodeName)
      POD_NAME:   stash-recovered-0 (v1:metadata.name)
    Mounts:
      /etc/stash/repository/secret from stash-secret-volume (rw)
      /source/data from source-data (rw)
      /tmp from tmp-dir (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-4tzgg (ro)
Containers:
  busybox:
    Container ID:  docker://ea8841e0f2074d5d67d0c35b72163c665fcde1f7a238e9c72636cfd5e8f58ac6
    Image:         busybox
    Image ID:      docker-pullable://busybox@sha256:c94cf1b87ccb80f2e6414ef913c748b105060debda482058d2b8d0fce39f11b9
    Port:          <none>
    Host Port:     <none>
    Command:
      sleep
      3600
    State:          Running
      Started:      Tue, 25 Jun 2019 18:25:53 +0600
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
    ClaimName:  source-data-stash-recovered-0
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
  Normal  Scheduled  51s   default-scheduler  Successfully assigned demo/stash-recovered-0 to minikube
  Normal  Pulled     50s   kubelet, minikube  Container image "suaas21/stash:vs_linux_amd64" already present on machine
  Normal  Created    50s   kubelet, minikube  Created container stash-init
  Normal  Started    49s   kubelet, minikube  Started container stash-init
  Normal  Pulled     22s   kubelet, minikube  Container image "busybox" already present on machine
  Normal  Created    22s   kubelet, minikube  Created container busybox
```

You will see that `stash-init` container has been injected.

**Wait for RestoreSession to Succeeded:**

Run the following command to watch RestoreSession phase,

```console
watch -n 5 kubectl get restoresession -n demo
Every 5.0s: kubectl get restoresession -n demo               suaas-appscode: Tue Jun 25 18:27:30 2019

NAME         REPOSITORY-NAME   PHASE       AGE
ss-restore   gcs-repo          Running     2m15s
ss-restore   gcs-repo          Succeeded   4m31s
```

So, we can see from the output of the above command that the restore process succeeded.

**Verify Restored Data:**

In this section, we will verify that the desired data has been restored successfully.

At first, check if the `stash-recovered` statefulset's pod has gone into Running state after successful `stash-init` container injection by the following command,

```console
$ kubectl get pod -n demo
NAME                READY   STATUS    RESTARTS   AGE
stash-recovered-0   1/1     Running   0          10m
stash-recovered-1   1/1     Running   0          11m
stash-recovered-2   1/1     Running   0          12m
```

Verify that the sample data has been restored in `/source/data` directory of the `stash-recovered` Statefulset's pod using the following command,

```console
$ kubectl exec -n demo stash-recovered-0 -- ls -R /source/data
/source/data:
stash-demo-0.txt
$ kubectl exec -n demo stash-recovered-1 -- ls -R /source/data
/source/data:
stash-demo-1.txt
$ kubectl exec -n demo stash-recovered-2 -- ls -R /source/data
/source/data:
stash-demo-2.txt
```

### Advanced

**Create RestoreSession:**

Now, we need to create a `RestoreSession` crd targeting the `stash-recovered` Statefulset to restore the inside it.

Below is the YAML of the `RestoreSesion` crd that we are going to create,

```console
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: ss-restore
  namespace: demo
spec:
  driver: Restic
  repository:
    name: gcs-repo
  # task:
  #   name: workload-restore # task field is not required for workload data restore but it is necessary for database restore.
  target:
    ref:
      apiVersion: apps/v1
      kind: StatefulSet
      name: stash-recovered
    volumeMounts:
      - mountPath: /source/data
        name: source-data
  rules:
    - targetHosts: ["host-1","host-2"] # "host-3" and "host-4" will have restored data of backed up host "host-1"
      sourceHost: "host-1" # source host
      paths:
        - /source/data
    - targetHosts: [] # empty host match all hosts
      sourceHost: "" # no source host indicates that the host is pod itself
      paths:
        - /source/data
```

Here,
............................................................

```console
$ kubectl apply -f ./adv_restoresession.yaml
restoresession.stash.appscode.com/ss-restore created
```

Once, you have created the `RestoreSession` crd, Stash will inject `init-container` to `stash-recovered` Statefulset. The Statefulset will restart and the `init-container` will recovered on start-up.

**Verify Init-Container:**

Wait until the `init-container` has been injected to the `stash-recovered` Statefulset's pod, Run the following command to describe the `stash-recovered` statefulset's pod,

```console
$ kubectl describe pod -n demo stash-recovered-0
Name:               stash-recovered-0
Namespace:          demo
Priority:           0
PriorityClassName:  <none>
Node:               minikube/10.0.2.15
Start Time:         Tue, 25 Jun 2019 18:59:20 +0600
Labels:             app=stash-demo
                    controller-revision-hash=stash-recovered-779667f4cc
                    statefulset.kubernetes.io/pod-name=stash-recovered-0
Annotations:        stash.appscode.com/last-applied-restoresession-hash: 2761075000118428355
Status:             Running
IP:                 172.17.0.8
Controlled By:      StatefulSet/stash-recovered
Init Containers:
  stash-init:
    Container ID:  docker://7ada5c7c00ee4e4bfc5730e8f8944cd925926ef64012af8fbda007dbb91ea263
    Image:         suaas21/stash:vs_linux_amd64
    Image ID:      docker-pullable://suaas21/stash@sha256:8b6afb1f6c6cd4139f6892e94367ae9462d76dca19a028e717e53afe1944250a
    Port:          <none>
    Host Port:     <none>
    Args:
      restore
      --restore-session=ss-restore
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
      Started:      Tue, 25 Jun 2019 18:59:21 +0600
      Finished:     Tue, 25 Jun 2019 18:59:47 +0600
    Ready:          True
    Restart Count:  0
    Environment:
      NODE_NAME:   (v1:spec.nodeName)
      POD_NAME:   stash-recovered-0 (v1:metadata.name)
    Mounts:
      /etc/stash/repository/secret from stash-secret-volume (rw)
      /source/data from source-data (rw)
      /tmp from tmp-dir (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-4tzgg (ro)
Containers:
  busybox:
    Container ID:  docker://366c264277ae57980aafd83baec6462a0876e5f20dff918a4504fcedb3943f11
    Image:         busybox
    Image ID:      docker-pullable://busybox@sha256:c94cf1b87ccb80f2e6414ef913c748b105060debda482058d2b8d0fce39f11b9
    Port:          <none>
    Host Port:     <none>
    Command:
      sleep
      3600
    State:          Running
      Started:      Tue, 25 Jun 2019 18:59:48 +0600
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
    ClaimName:  source-data-stash-recovered-0
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
  Normal  Scheduled  33s   default-scheduler  Successfully assigned demo/stash-recovered-0 to minikube
  Normal  Pulled     32s   kubelet, minikube  Container image "suaas21/stash:vs_linux_amd64" already present on machine
  Normal  Created    32s   kubelet, minikube  Created container stash-init
  Normal  Started    32s   kubelet, minikube  Started container stash-init
  Normal  Pulled     6s    kubelet, minikube  Container image "busybox" already present on machine
  Normal  Created    6s    kubelet, minikube  Created container busybox
  Normal  Started    5s    kubelet, minikube  Started container busybox
```

You will see that `stash-init` container has been injected.

**Wait for RestoreSession to Succeeded:**

Run the following command to watch RestoreSession phase,

```console
watch -n 5 kubectl get restoresession -n demo
Every 5.0s: kubectl get restoresession -n demo               suaas-appscode: Tue Jun 25 18:27:30 2019

NAME         REPOSITORY-NAME   PHASE       AGE
ss-restore   gcs-repo          Running     2m
ss-restore   gcs-repo          Succeeded   4m21s
```

So, we can see from the output of the above command that the restore process succeeded.

**Verify Restored Data:**

Verify that the sample data has been restored in `/source/data` directory of the `stash-recovered` Statefulset's pod using the following command,

```console
$ kubectl exec -n demo stash-recovered-0 -- ls -R /source/data
/source/data:
stash-demo-0.txt
$ kubectl exec -n demo stash-recovered-1 -- ls -R /source/data
/source/data:
stash-demo-1.txt
$ kubectl exec -n demo stash-recovered-2 -- ls -R /source/data
/source/data:
stash-demo-1.txt
stash-demo-2.txt
```

# Cleaning Up

To clean up the Kubernetes resources created by this tutorial, run:

```console
kubectl delete -n demo statefulset stash-demo
kubectl delete -n demo statefulset stash-recovered
kubectl delete -n demo backupconfiguration ss-backup
kubectl delete -n demo restoresession ss-restore
kubectl delete -n demo repository gcs-repo
kubectl delete -n demo pvc --all
```