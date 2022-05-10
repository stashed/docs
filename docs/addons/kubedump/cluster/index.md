---
title: Backup Cluster KubeDumps | Stash
description: Take backup of cluster manifests using Stash
menu:
  docs_{{ .version }}:
    identifier: stash-kubedump-helm
    name: Cluster Backup
    parent: stash-kubedump
    weight: 20
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: stash-addons
---

# Backup KubeDumps of Entire Cluster using Stash

Stash `{{< param "info.version" >}}` supports backup and restoration of Redis databases. This guide will show you how you can take a logical backup of your Redis databases and restore them using Stash.

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster.
- Install Stash Enterprise in your cluster following the steps [here](/docs/setup/install/enterprise.md).
- If you are not familiar with how Stash backup and restore Redis databases, please check the following guide [here](/docs/addons/redis/overview/index.md).

You have to be familiar with following custom resources:

- [AppBinding](/docs/concepts/crds/appbinding.md)
- [Function](/docs/concepts/crds/function.md)
- [Task](/docs/concepts/crds/task.md)
- [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md)
- [BackupSession](/docs/concepts/crds/backupsession.md)
- [RestoreSession](/docs/concepts/crds/restoresession.md)

To keep things isolated, we are going to use a separate namespace called `demo` throughout this tutorial. Create `demo` namespace if you haven't created already.

```bash
$ kubectl create ns demo
namespace/demo created
```

> Note: YAML files used in this tutorial are stored [here](https://github.com/stashed/docs/tree/{{< param "info.version" >}}/docs/addons/redis/helm/examples).

## Prepare Redis

In this section, we are going to deploy a Redis database. Then, we are going to insert some sample data into it.

### Deploy Redis

At first, let's deploy a Redis database. Here, we are going to use [bitnami/redis](https://artifacthub.io/packages/helm/bitnami/redis)  chart from [ArtifactHub](https://artifacthub.io/).

Let's deploy a Redis database named `sample-redis` using Helm as below,

```bash
# Add bitnami chart registry
$ helm repo add bitnami https://charts.bitnami.com/bitnami
# Update helm registries
$ helm repo update
# Install bitnami/redis chart into demo namespace
$ helm install sample-redis bitnami/redis -n demo
```

This chart will create the necessary StatefulSet, Secret, Service etc. for the database. You can easily view all the resources created by chart using [ketall](https://github.com/corneliusweig/ketall) `kubectl` plugin as below,

```bash
❯ kubectl get-all -n demo -l app.kubernetes.io/instance=sample-redis
NAME                                                        NAMESPACE  AGE
configmap/sample-redis-configuration                        demo       117s  
configmap/sample-redis-health                               demo       117s  
configmap/sample-redis-scripts                              demo       117s  
endpoints/sample-redis-headless                             demo       117s  
endpoints/sample-redis-master                               demo       117s  
endpoints/sample-redis-replicas                             demo       117s  
persistentvolumeclaim/redis-data-sample-redis-master-0      demo       117s  
persistentvolumeclaim/redis-data-sample-redis-replicas-0    demo       117s  
persistentvolumeclaim/redis-data-sample-redis-replicas-1    demo       79s   
persistentvolumeclaim/redis-data-sample-redis-replicas-2    demo       51s   
pod/sample-redis-master-0                                   demo       117s  
pod/sample-redis-replicas-0                                 demo       117s  
pod/sample-redis-replicas-1                                 demo       79s   
pod/sample-redis-replicas-2                                 demo       51s   
secret/sample-redis                                         demo       117s  
serviceaccount/sample-redis                                 demo       117s  
service/sample-redis-headless                               demo       117s  
service/sample-redis-master                                 demo       117s  
service/sample-redis-replicas                               demo       117s  
controllerrevision.apps/sample-redis-master-755dd8b64d      demo       117s  
controllerrevision.apps/sample-redis-replicas-7b8c7694bf    demo       117s  
statefulset.apps/sample-redis-master                        demo       117s  
statefulset.apps/sample-redis-replicas                      demo       117s  
endpointslice.discovery.k8s.io/sample-redis-headless-6bvt2  demo       117s  
endpointslice.discovery.k8s.io/sample-redis-master-78wcv    demo       117s  
endpointslice.discovery.k8s.io/sample-redis-replicas-vhc7z  demo       117s 
```

Now, wait for the database pod `sample-redis-master-0` to go into `Running` state,

```bash
❯ kubectl get pod -n demo sample-redis-master-0
NAME                    READY   STATUS    RESTARTS   AGE
sample-redis-master-0   1/1     Running   0          2m57s
```

Once the database pod is in `Running` state, verify that the database is ready to accept the connections.

```bash
❯ kubectl logs -n demo sample-redis-master-0
1:C 28 Jul 2021 13:03:28.191 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1:C 28 Jul 2021 13:03:28.191 # Redis version=6.2.5, bits=64, commit=00000000, modified=0, pid=1, just started
1:C 28 Jul 2021 13:03:28.191 # Configuration loaded
1:M 28 Jul 2021 13:03:28.192 * monotonic clock: POSIX clock_gettime
1:M 28 Jul 2021 13:03:28.192 * Running mode=standalone, port=6379.
1:M 28 Jul 2021 13:03:28.192 # Server initialized
1:M 28 Jul 2021 13:03:28.193 * Ready to accept connections
```

From the above log, we can see the database is ready to accept connections.

### Insert Sample Data

Now, we are going to exec into the database pod and create some sample data. The helm chart has created a secret with access credentials. Let's find out the credentials from the Secret,

```yaml
❯ kubectl get secret -n demo sample-redis -o yaml
apiVersion: v1
data:
  redis-password: WTFZTENrZmNpcw==
kind: Secret
metadata:
  annotations:
    meta.helm.sh/release-name: sample-redis
    meta.helm.sh/release-namespace: demo
  creationTimestamp: "2021-07-28T13:03:23Z"
  labels:
    app.kubernetes.io/instance: sample-redis
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: redis
    helm.sh/chart: redis-14.8.6
  name: sample-redis
  namespace: demo
  resourceVersion: "530037"
  uid: a48ce23a-105d-4d92-9067-c80623cbe269
type: Opaque

```

Here, we are going to use `redis-password` to authenticate and insert the sample data.

At first, let's export the password as environment variables to make further commands re-usable.

```bash
export PASSWORD=$(kubectl get secrets -n demo sample-redis -o jsonpath='{.data.\redis-password}' | base64 -d)
```

Now, let's exec into the database pod and insert some sample data,

```bash
❯ kubectl exec -it -n demo sample-redis-master-0 -- redis-cli -a $PASSWORD
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
# insert some key value pairs
127.0.0.1:6379> set key1 value1
OK
127.0.0.1:6379> set key2 value2
OK
# check the inserted data
127.0.0.1:6379> get key1
"value1"
127.0.0.1:6379> get key2
"value2"
# exit from redis-cli
127.0.0.1:6379> exit
```

We have successfully deployed a Redis database and inserted some sample data into it. In the subsequent sections, we are going to backup these data using Stash.

## Prepare for Backup

In this section, we are going to prepare the necessary resources (i.e. database connection information, backend information, etc.) before backup.

### Ensure Redis Addon

When you install Stash Enterprise version, it will automatically install all the official database addons. Make sure that Redis addon was installed properly using the following command.

```bash
❯ kubectl get tasks.stash.appscode.com | grep redis
redis-backup-6.2.5            24m
redis-restore-6.2.5           24m
```

This addon should be able to take backup of the databases with matching major versions as discussed in [Addon Version Compatibility](/docs/addons/redis/README.md#addon-version-compatibility).

### Create AppBinding

Stash needs to know how to connect with the database. An `AppBinding` exactly provides this information. It holds the Service and Secret information of the database. You have to point to the respective `AppBinding` as a target of backup instead of the database itself.

Stash expect your database Secret to have `password` keys. If your database secret does not have the expected key, the `AppBinding` can also help here. You can specify a `secretTransforms` section with the mapping between the current keys and the desired keys.

Here, is the YAML of the `AppBinding` that we are going to create for the Redis database we have deployed earlier.

```yaml
apiVersion: appcatalog.appscode.com/v1alpha1
kind: AppBinding
metadata:
  name: sample-redis
  namespace: demo
spec:
  clientConfig:
    service:
      name: sample-redis-master
      path: /
      port: 6379
      scheme: http
  secret:
    name: sample-redis
  secretTransforms:
  - renameKey:
      from: redis-password
      to: password
  type: redis
  version: 6.2.5
```

Here,

- **.spec.clientConfig.service** specifies the Service information to use to connects with the database.
- **.spec.secret** specifies the name of the Secret that holds necessary credentials to access the database. If your Redis is not using authentication, then don't provide this field.
- **.spec.secretTransforms** specifies the transformations required to achieve the desired keys from the current Secret. You can apply the following transformations here:
  - **addKey**: If your database Secret does not have an equivalent key expected by Stash, you can add the key using `addKey` transformation.
  - **renameKey**: If your database Secret does not have a key expected by Stash but it has an equivalent key that is used for the same purpose, you can use `renameKey` transformation to specify the mapping between the keys. For example, our Redis Secret didn't have `password` key but it has an equivalent `redis-password` key that contains password for the database. Hence, we are telling Stash using `renameKey` transformation that the `redis-password` should be used as `password` key.
  - **addKeysFrom**: You can also merge keys from another Secret using `addKeysFrom` transformation. You have to specify the respective Secret name and namespace as below:
    ```yaml
    addKeysFrom:
      name: <secret name>
      namespace: <secret namespace>
    ```
- `spec.type` specifies the type of the database. This is particularly helpful in auto-backup where you want to use different path prefixes for different types of database.

Let's create the `AppBinding` we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/tree/{{< param "info.version" >}}/docs/addons/redis/helm/examples/appbinding.yaml
appbinding.appcatalog.appscode.com/sample-redis created
```

>The `secretTransforms` does not modify your original database Secret. Stash just uses those transformations to obtain the desired keys from the original Secret.

### Prepare Backend

We are going to store our backed up data into a GCS bucket. So, we need to create a Secret with GCS credentials and a `Repository` object with the bucket information. If you want to use a different backend, please read the respective backend configuration doc from [here](/docs/guides/backends/overview.md).

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
      prefix: /manifests/cluster
    storageSecretName: gcs-secret
```

Let's create the `Repository` we have shown above,

```bash
$ kubectl create -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}❯/docs/addons/kubedump/cluster/examples/repository.yaml
repository.stash.appscode.com/gcs-repo created
```

Now, we are ready to backup our database into our desired backend.

```bash
❯ kubectl apply -f ./docs/addons/kubedump/cluster/examples/rbac.yaml
serviceaccount/cluster-resource-reader created
clusterrole.rbac.authorization.k8s.io/cluster-resource-reader created
clusterrolebinding.rbac.authorization.k8s.io/cluster-resource-reader created
```

### Backup

To schedule a backup, we have to create a `BackupConfiguration` object targeting the respective `AppBinding` of our desired database. Then Stash will create a CronJob to periodically backup the database.

#### Create BackupConfiguration

Below is the YAML for `BackupConfiguration` object we care going to use to backup the `sample-redis` database we have deployed earlier,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: cluster-kubedump
  namespace: demo
spec:
  schedule: "*/5 * * * *"
  task:
    name: kubedump-backup-0.1.0
  repository:
    name: gcs-repo
  retentionPolicy:
    name: keep-last-5
    keepLast: 5
    prune: true
```

Here,

- `.spec.schedule` specifies that we want to backup the database at 5 minutes intervals.
- `.spec.task.name` specifies the name of the Task object that specifies the necessary Functions and their execution order to backup a Redis database.
- `.spec.repository.name` specifies the Repository CR name we have created earlier with backend information.
- `.spec.target.ref` refers to the AppBinding object that holds the connection information of our targeted database.
- `.spec.retentionPolicy` specifies a policy indicating how we want to cleanup the old backups.

Let's create the `BackupConfiguration` object we have shown above,

```bash
$ kubectl create -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/addons/kubedump/cluster/examples/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/cluster-kubedump created
```


#### Verify Backup Setup Successful

If everything goes well, the phase of the `BackupConfiguration` should be `Ready`. The `Ready` phase indicates that the backup setup is successful. Let's verify the `Phase` of the BackupConfiguration,

```bash
$ kubectl get backupconfiguration -n demo
NAME                  TASK                       SCHEDULE      PAUSED   PHASE      AGE
sample-redis-backup   redis-backup-6.2.5         */5 * * * *            Ready      11s
```

#### Verify CronJob

Stash will create a CronJob with the schedule specified in `spec.schedule` field of `BackupConfiguration` object.

Verify that the CronJob has been created using the following command,

```bash
❯ kubectl get cronjob -n demo
NAME                               SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
stash-backup-sample-redis-backup   */5 * * * *   False     0        <none>          14s
```

#### Wait for BackupSession

The `sample-redis-backup` CronJob will trigger a backup on each scheduled slot by creating a `BackupSession` object.

Now, wait for a schedule to appear. Run the following command to watch for a `BackupSession` object,

```bash
❯ kubectl get backupsession --all-namespaces -w
NAMESPACE   NAME                                 INVOKER-TYPE          INVOKER-NAME              PHASE   DURATION   AGE
demo        cluster-kubedump-1651049400   BackupConfiguration   cluster-kubedump                      0s
demo        cluster-kubedump-1651049400   BackupConfiguration   cluster-kubedump   Pending              0s
demo        cluster-kubedump-1651049400   BackupConfiguration   cluster-kubedump   Running              0s
demo        cluster-kubedump-1651049400   BackupConfiguration   cluster-kubedump   Running              19s
demo        cluster-kubedump-1651049400   BackupConfiguration   cluster-kubedump   Running              53s
demo        cluster-kubedump-1651049400   BackupConfiguration   cluster-kubedump   Running              70s
demo        cluster-kubedump-1651049400   BackupConfiguration   cluster-kubedump   Running              70s
demo        cluster-kubedump-1651049400   BackupConfiguration   cluster-kubedump   Running              70s
demo        cluster-kubedump-1651049400   BackupConfiguration   cluster-kubedump   Running              70s
demo        cluster-kubedump-1651049400   BackupConfiguration   cluster-kubedump   Succeeded   1m10s
```

Here, the phase `Succeeded` means that the backup process has been completed successfully.

#### Verify Backup

Now, we are going to verify whether the backed up data is present in the backend or not. Once a backup is completed, Stash will update the respective `Repository` object to reflect the backup completion. Check that the repository `gcs-repo` has been updated by the following command,

```bash
❯ kubectl get repository -n demo
NAME       INTEGRITY   SIZE        SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
gcs-repo   true        2.525 MiB   1                108s                     89m

```

Now, if we navigate to the GCS bucket, we will see the backed up data has been stored in `demo/redis/sample-redis` directory as specified by `.spec.backend.gcs.prefix` field of the `Repository` object.

<figure align="center">
  <img alt="Backup data in GCS Bucket" src="/docs/addons/redis/helm/images/sample-redis-backup.png">
  <figcaption align="center">Fig: Backup data in GCS Bucket</figcaption>
</figure>

> Note: Stash keeps all the backed up data encrypted. So, data in the backend will not make any sense until they are decrypted.

## Downloads KubeDumps

```bash
❯ kubectl stash download -n demo gcs-repo  --destination=$HOME/Downloads/stash --snapshots="latest"
I0427 16:01:16.874481  698253 download.go:176] Running docker with args: [run --rm -u 1000 -v /tmp/scratch:/tmp/scratch -v /home/emruz/Downloads/stash:/tmp/destination --env HTTP_PROXY= --env HTTPS_PROXY= --env-file /tmp/scratch/config/restic-envs stashed/restic:latest --no-cache restore latest --target /tmp/destination/latest]
I0427 16:02:19.413197  698253 download.go:181] Output: restoring <Snapshot ae2de1c2 of [/tmp/manifests] at 2022-04-27 10:00:23.19484503 +0000 UTC by @host-0> to /tmp/destination/latest

I0427 16:02:19.413215  698253 download.go:137] Snapshots: [latest] of Repository demo/gcs-repo restored in path /home/emruz/Downloads/stash
❯ cd $HOME/Downloads/stash
❯ ls
latest
❯ cd latest/
❯ ls
tmp
❯ cd tmp/manifests/
❯ ls
global  namespaces
❯ ls global
APIService   ClusterRoleBinding  CSINode                   FlowSchema  MetricsConfiguration          Namespace  PodSecurityPolicy  PriorityLevelConfiguration  Task
ClusterRole  ComponentStatus     CustomResourceDefinition  Function    MutatingWebhookConfiguration  Node       PriorityClass      StorageClass                ValidatingWebhookConfiguration
❯ ls namespaces
default  demo  kube-node-lease  kube-public  kube-system  local-path-storage  stash-e2e-z1ggg4-restore
❯ ls global/ClusterRole
admin.yaml                                 system:aggregate-to-admin.yaml                                             system:controller:ephemeral-volume-controller.yaml    system:coredns.yaml
appscode:license-checker.yaml              system:aggregate-to-edit.yaml                                              system:controller:expand-controller.yaml              system:discovery.yaml
appscode:license-reader.yaml               system:aggregate-to-view.yaml                                              system:controller:generic-garbage-collector.yaml      system:heapster.yaml
appscode:metrics:edit.yaml                 system:auth-delegator.yaml                                                 system:controller:horizontal-pod-autoscaler.yaml      system:kube-aggregator.yaml
appscode:metrics:view.yaml                 system:basic-user.yaml                                                     system:controller:job-controller.yaml                 system:kube-controller-manager.yaml
appscode:stash:edit.yaml                   system:certificates.k8s.io:certificatesigningrequests:nodeclient.yaml      system:controller:namespace-controller.yaml           system:kube-dns.yaml
appscode:stash:garbage-collector.yaml      system:certificates.k8s.io:certificatesigningrequests:selfnodeclient.yaml  system:controller:node-controller.yaml                system:kubelet-api-admin.yaml
appscode:stash:view.yaml                   system:certificates.k8s.io:kube-apiserver-client-approver.yaml             system:controller:persistent-volume-binder.yaml       system:kube-scheduler.yaml
cluster-admin.yaml                         system:certificates.k8s.io:kube-apiserver-client-kubelet-approver.yaml     system:controller:pod-garbage-collector.yaml          system:monitoring.yaml
cluster-resource-reader.yaml               system:certificates.k8s.io:kubelet-serving-approver.yaml                   system:controller:pvc-protection-controller.yaml      system:node-bootstrapper.yaml
edit.yaml                                  system:certificates.k8s.io:legacy-unknown-approver.yaml                    system:controller:pv-protection-controller.yaml       system:node-problem-detector.yaml
kindnet.yaml                               system:controller:attachdetach-controller.yaml                             system:controller:replicaset-controller.yaml          system:node-proxier.yaml
kubeadm:get-nodes.yaml                     system:controller:certificate-controller.yaml                              system:controller:replication-controller.yaml         system:node.yaml
local-path-provisioner-role.yaml           system:controller:clusterrole-aggregation-controller.yaml                  system:controller:resourcequota-controller.yaml       system:persistent-volume-provisioner.yaml
stash-backup-job.yaml                      system:controller:cronjob-controller.yaml                                  system:controller:root-ca-cert-publisher.yaml         system:public-info-viewer.yaml
stash-cron-job.yaml                        system:controller:daemon-set-controller.yaml                               system:controller:route-controller.yaml               system:service-account-issuer-discovery.yaml
stash-restore-init-container.yaml          system:controller:deployment-controller.yaml                               system:controller:service-account-controller.yaml     system:volume-scheduler.yaml
stash-restore-job.yaml                     system:controller:disruption-controller.yaml                               system:controller:service-controller.yaml             view.yaml
stash-sidecar.yaml                         system:controller:endpoint-controller.yaml                                 system:controller:statefulset-controller.yaml
stash-stash-enterprise-crd-installer.yaml  system:controller:endpointslice-controller.yaml                            system:controller:ttl-after-finished-controller.yaml
stash-stash-enterprise.yaml                system:controller:endpointslicemirroring-controller.yaml                   system:controller:ttl-controller.yaml
❯ cat  global/ClusterRole/appscode:license-reader.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    helm.sh/hook: pre-install,pre-upgrade
    helm.sh/hook-delete-policy: before-hook-creation
  name: appscode:license-reader
rules:
- apiGroups:
  - apiregistration.k8s.io
  resources:
  - apiservices
  verbs:
  - get
- nonResourceURLs:
  - /appscode/license
  verbs:
  - get
❯ ls namespaces/kube-system
ConfigMap  ControllerRevision  DaemonSet  Deployment  Endpoints  EndpointSlice  Lease  Pod  ReplicaSet  Role  RoleBinding  Secret  Service  ServiceAccount
❯ ls namespaces/kube-system/ConfigMap
coredns.yaml  extension-apiserver-authentication.yaml  kubeadm-config.yaml  kubelet-config-1.21.yaml  kube-proxy.yaml  kube-root-ca.crt.yaml
❯ cat namespaces/kube-system/ConfigMap/kubeadm-config.yaml
apiVersion: v1
data:
  ClusterConfiguration: |
    apiServer:
      certSANs:
      - localhost
      - 127.0.0.1
      extraArgs:
        authorization-mode: Node,RBAC
        runtime-config: ""
      timeoutForControlPlane: 4m0s
    apiVersion: kubeadm.k8s.io/v1beta2
    certificatesDir: /etc/kubernetes/pki
    clusterName: kind
    controlPlaneEndpoint: kind-control-plane:6443
    controllerManager:
      extraArgs:
        enable-hostpath-provisioner: "true"
    dns:
      type: CoreDNS
    etcd:
      local:
        dataDir: /var/lib/etcd
    imageRepository: k8s.gcr.io
    kind: ClusterConfiguration
    kubernetesVersion: v1.21.1
    networking:
      dnsDomain: cluster.local
      podSubnet: 10.244.0.0/16
      serviceSubnet: 10.96.0.0/16
    scheduler: {}
  ClusterStatus: |
    apiEndpoints:
      kind-control-plane:
        advertiseAddress: 172.18.0.2
        bindPort: 6443
    apiVersion: kubeadm.k8s.io/v1beta2
    kind: ClusterStatus
kind: ConfigMap
metadata:
  name: kubeadm-config
  namespace: kube-system
```

## Cleanup

To cleanup the Kubernetes resources created by this tutorial, run:

```bash
kubectl delete -n demo backupconfiguration sample-redis-backup
kubectl delete -n demo restoresession sample-redis-restore
kubectl delete -n demo repository gcs-repo
# delete the database chart
helm delete sample-redis -n demo
```
