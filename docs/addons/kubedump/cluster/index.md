---
title: Backup resource YAMLs of entire cluster | Stash
description: Take backup of cluster resources YAMLs using Stash
menu:
  docs_{{ .version }}:
    identifier: stash-kubedump-cluster
    name: Cluster Backup
    parent: stash-kubedump
    weight: 20
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: stash-addons
---

# Backup resource YAMLs of entire cluster using Stash

Stash `{{< param "info.version" >}}` supports taking backup of the resource YAMLs using `kubedump` plugin. This guide will show you how you can take a backup of the resource YAMLs of your entire cluster using Stash.

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster.
- Install Stash Enterprise in your cluster following the steps [here](/docs/setup/install/enterprise.md).
- Install Stash `kubectl` plugin in your local machine following the steps [here](/docs/setup/install/kubectl_plugin.md).
- If you are not familiar with how Stash backup the resource YAMLs, please check the following guide [here](/docs/addons/kubedump/overview/index.md).

You have to be familiar with the following custom resources:

- [AppBinding](/docs/concepts/crds/appbinding/index.md)
- [Function](/docs/concepts/crds/function/index.md)
- [Task](/docs/concepts/crds/task/index.md)
- [BackupConfiguration](/docs/concepts/crds/backupconfiguration/index.md)
- [BackupSession](/docs/concepts/crds/backupsession/index.md)

To keep things isolated, we are going to use a separate namespace called `demo` throughout this tutorial. Create the `demo` namespace if you haven't created it already.

```bash
$ kubectl create ns demo
namespace/demo created
```

> Note: YAML files used in this tutorial are stored [here](https://github.com/stashed/docs/tree/{{< param "info.version" >}}/docs/addons/kubedump/cluster/examples).

### Prepare for Backup

In this section, we are going to configure a backup for all the resource YAMLs of our cluster.

#### Ensure `kubedump` Addon

When you install the Stash Enterprise version, it will automatically install all the official addons. Make sure that `kubedump` addon was installed properly using the following command.

```bash
❯ kubectl get tasks.stash.appscode.com | grep kubedump
kubedump-backup-0.1.0          23s
```

#### Prepare Backend

We are going to store our backed-up data into a GCS bucket. So, we need to create a Secret with GCS credentials and a `Repository` object with the bucket information. If you want to use a different backend, please read the respective backend configuration doc from [here](/docs/guides/backends/overview/index.md).

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
  name: cluster-resource-storage
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
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/addons/kubedump/cluster/examples/repository.yaml
repository.stash.appscode.com/cluster-resource-storage created
```

#### Create RBAC

The `kubedump` plugin requires read permission for all the cluster resources. By default, Stash does not grant such cluster-wide permissions. We have to provide the necessary permissions manually.

Here, is the YAML of the `ServiceAccount`, `ClusterRole`, and `ClusterRoleBinding` that we are going to use for granting the necessary permissions.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cluster-resource-reader
  namespace: demo
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-resource-reader
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["get","list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-resource-reader
subjects:
- kind: ServiceAccount
  name: cluster-resource-reader
  namespace: demo
roleRef:
  kind: ClusterRole
  name: cluster-resource-reader
  apiGroup: rbac.authorization.k8s.io
```

Let's create the RBAC resources we have shown above,

```bash
❯ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/addons/kubedump/cluster/examples/rbac.yaml
serviceaccount/cluster-resource-reader created
clusterrole.rbac.authorization.k8s.io/cluster-resource-reader created
clusterrolebinding.rbac.authorization.k8s.io/cluster-resource-reader created
```

Now, we are ready for backup. In the next section, we are going to schedule a backup for our cluster resources.

### Backup

To schedule a backup, we have to create a `BackupConfiguration` object. Then Stash will create a CronJob to periodically backup the database.

#### Create BackupConfiguration

Below is the YAML for `BackupConfiguration` object we care going to use to backup the YAMLs of the cluster resources,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: cluster-resources-backup
  namespace: demo
spec:
  schedule: "*/5 * * * *"
  task:
    name: kubedump-backup-0.1.0
  repository:
    name: cluster-resource-storage
  runtimeSettings:
    pod:
      serviceAccountName: cluster-resource-reader
  retentionPolicy:
    name: keep-last-5
    keepLast: 5
    prune: true
```

Here,

- `.spec.schedule` specifies that we want to backup the cluster resources at 5 minutes intervals.
- `.spec.task.name` specifies the name of the Task object that specifies the necessary Functions and their execution order to backup the resource YAMLs.
- `.spec.repository.name` specifies the Repository CR name we have created earlier with backend information.
- `.spec.runtimeSettings.pod.serviceAccountName` specifies the ServiceAccount name that we have created earlier with cluster-wide resource reading permission.
- `.spec.retentionPolicy` specifies a policy indicating how we want to cleanup the old backups.

Let's create the `BackupConfiguration` object we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/addons/kubedump/cluster/examples/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/cluster-resources-backup created
```

#### Verify Backup Setup Successful

If everything goes well, the phase of the `BackupConfiguration` should be `Ready`. The `Ready` phase indicates that the backup setup is successful. Let's verify the `Phase` of the BackupConfiguration,

```bash
❯ kubectl get backupconfiguration -n demo
NAME                       TASK                    SCHEDULE      PAUSED   PHASE   AGE
cluster-resources-backup   kubedump-backup-0.1.0   */5 * * * *            Ready   18s
```

#### Verify CronJob

Stash will create a CronJob with the schedule specified in `spec.schedule` field of `BackupConfiguration` object.

Verify that the CronJob has been created using the following command,

```bash
❯ kubectl get cronjob -n demo
NAME                                     SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
stash-trigger-cluster-resources-backup   */5 * * * *   False     0        <none>          49s
```

#### Wait for BackupSession

The `stash-trigger-cluster-resources-backup` CronJob will trigger a backup on each scheduled slot by creating a `BackupSession` object.

Now, wait for a schedule to appear. Run the following command to watch for a `BackupSession` object,

```bash
❯ kubectl get backupsession -n demo -w
NAME                                  INVOKER-TYPE          INVOKER-NAME               PHASE   DURATION   AGE
cluster-resources-backup-1652164800   BackupConfiguration   cluster-resources-backup                      0s
cluster-resources-backup-1652164800   BackupConfiguration   cluster-resources-backup   Pending              0s
cluster-resources-backup-1652164800   BackupConfiguration   cluster-resources-backup   Running              0s
cluster-resources-backup-1652164800   BackupConfiguration   cluster-resources-backup   Running              69s
cluster-resources-backup-1652164800   BackupConfiguration   cluster-resources-backup   Succeeded   1m9s       69s
```

Here, the phase `Succeeded` means that the backup process has been completed successfully.

#### Verify Backup

Now, we are going to verify whether the backed-up data is present in the backend or not. Once a backup is completed, Stash will update the respective `Repository` object to reflect the backup completion. Check that the repository `cluster-resource-storage` has been updated by the following command,

```bash
❯ kubectl get repository -n demo
NAME                       INTEGRITY   SIZE        SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
cluster-resource-storage   true        2.324 MiB   1                70s                      54m

```

Now, if we navigate to the GCS bucket, we will see the backed up data has been stored in `/manifest/cluster` directory as specified by `.spec.backend.gcs.prefix` field of the `Repository` object.

<figure align="center">
  <img alt="Backup data in GCS Bucket" src="/docs/addons/kubedump/cluster/images/cluster_manifests_backup.png">
  <figcaption align="center">Fig: Backup data in GCS Bucket</figcaption>
</figure>

> Note: Stash keeps all the backed-up data encrypted. So, data in the backend will not make any sense until they are decrypted.

## Restore

Stash does not provide any automatic mechanism to restore the cluster resources from the backed-up YAMLs. Your application might be managed by Helm or by an operator. In such cases, just applying the YAMLs is not enough to restore the application. Furthermore, there might be an order issue. Some resources must be applied before others. It is difficult to generalize and codify various application-specific logic.

Therefore, it is the user's responsibility to download the backed-up YAMLs and take the necessary steps based on his application to restore it properly.

### Download the YAMLs

Stash provides a [kubectl plugin](https://stash.run/docs/v2022.05.12/guides/cli/cli/#download-snapshots) for making it easy to download a snapshot locally.

Now, let's download the latest Snapshot from our backed-up data into the `$HOME/Downloads/stash` folder of our local machine.

```bash
❯ kubectl stash download -n demo cluster-resource-storage  --destination=$HOME/Downloads/stash --snapshots="latest"
```

Now, lets use [tree](https://linux.die.net/man/1/tree) command to inspect downloaded YAMLs files.

```bash
❯ tree $HOME/Downloads/stash
/home/emruz/Downloads/stash
└── latest
    └── tmp
        └── resources
            ├── global
            │   ├── APIService
            │   │   ├── v1.admissionregistration.k8s.io.yaml
            │   │   ├── v1alpha1.admission.stash.appscode.com.yaml
            │   │   ├── v1alpha1.metrics.appscode.com.yaml
            │   │   ├── v1alpha1.repositories.stash.appscode.com.yaml
            │   ├── ClusterRole
            │   │   ├── admin.yaml
            │   │   ├── system:heapster.yaml
            │   │   └── view.yaml
            │   ├── ClusterRoleBinding
            │   │   ├── appscode:license-reader-demo-cluster-resource-reader.yaml
            │   │   ├── appscode:stash:garbage-collector.yaml
            │   │   ├── cluster-admin.yaml
            │   │   ├── cluster-resource-reader.yaml
            │   │   ├── kindnet.yaml
            │   ├── ComponentStatus
            │   │   ├── controller-manager.yaml
            │   │   ├── etcd-0.yaml
            │   │   └── scheduler.yaml
            │   ├── CSINode
            │   │   └── kind-control-plane.yaml
            │   ├── CustomResourceDefinition
            │   │   ├── appbindings.appcatalog.appscode.com.yaml
            │   │   ├── backupbatches.stash.appscode.com.yaml
            │   │   ├── backupblueprints.stash.appscode.com.yaml
            │   │   ├── backupconfigurations.stash.appscode.com.yaml
            │   ├── FlowSchema
            │   │   ├── catch-all.yaml
            │   │   ├── exempt.yaml
            │   │   ├── global-default.yaml
            │   │   ├── kube-controller-manager.yaml
            │   │   ├── kube-scheduler.yaml
            │   ├── Function
            │   │   ├── elasticsearch-backup-5.6.4.yaml
            │   │   ├── mongodb-restore-5.0.3.yaml
            │   │   ├── mysql-backup-5.7.25.yaml
            │   │   └── update-status.yaml
            │   ├── MetricsConfiguration
            │   │   ├── stash-appscode-com-backupconfiguration.yaml
            │   ├── MutatingWebhookConfiguration
            │   │   └── admission.stash.appscode.com.yaml
            │   ├── Namespace
            │   │   ├── default.yaml
            │   │   ├── demo.yaml
            │   │   ├── kube-node-lease.yaml
            │   │   ├── kube-public.yaml
            │   │   ├── kube-system.yaml
            │   │   └── local-path-storage.yaml
            │   ├── Node
            │   │   └── kind-control-plane.yaml
            │   ├── StorageClass
            │   │   └── standard.yaml
            │   ├── Task
            │   │   ├── elasticsearch-backup-5.6.4.yaml
            │   │   ├── redis-restore-5.0.13.yaml
            │   │   └── redis-restore-6.2.5.yaml
            │   └── ValidatingWebhookConfiguration
            │       └── admission.stash.appscode.com.yaml
            └── namespaces
                ├── default
                │   ├── ConfigMap
                │   │   └── kube-root-ca.crt.yaml
                │   ├── Endpoints
                │   │   └── kubernetes.yaml
                │   ├── EndpointSlice
                │   │   └── kubernetes.yaml
                │   ├── Secret
                │   │   └── default-token-7lpm6.yaml
                │   ├── Service
                │   │   └── kubernetes.yaml
                │   └── ServiceAccount
                │       └── default.yaml
                ├── demo
                │   ├── BackupConfiguration
                │   │   └── cluster-resources-backup.yaml
                │   ├── BackupSession
                │   │   ├── cluster-resources-backup-1652176500.yaml
                │   │   └── cluster-resources-backup-1652176800.yaml
                │   ├── ConfigMap
                │   │   └── kube-root-ca.crt.yaml
                │   ├── CronJob
                │   │   └── stash-trigger-cluster-resources-backup.yaml
                │   ├── Event
                │   │   ├── cluster-resources-backup-1652173200.16edb2d9e21556ba.yaml
                │   ├── Job
                │   │   ├── stash-backup-cluster-resources-backup-1652176500-0.yaml
                │   │   └── stash-backup-cluster-resources-backup-1652176800-0.yaml
                │   ├── Pod
                │   │   ├── stash-backup-cluster-resources-backup-1652176500-0-vmx94.yaml
                │   │   └── stash-backup-cluster-resources-backup-1652176800-0-5v4k9.yaml
                │   ├── Repository
                │   │   └── cluster-resource-storage.yaml
                │   ├── RoleBinding
                │   │   ├── backupconfiguration-cluster-resources-backup-0.yaml
                │   │   └── stash-trigger-cluster-resources-backup.yaml
                │   ├── Secret
                │   │   ├── cluster-resource-reader-token-rqf6n.yaml
                │   │   ├── default-token-rbrqt.yaml
                │   │   └── gcs-secret.yaml
                │   ├── ServiceAccount
                │   │   ├── cluster-resource-reader.yaml
                │   │   └── default.yaml
                │   └── Snapshot
                │       ├── cluster-resource-storage-5085e316.yaml
                │       ├── cluster-resource-storage-6034bdcf.yaml
                │       ├── cluster-resource-storage-706e8f46.yaml
                ├── kube-node-lease
                │   ├── ConfigMap
                │   │   └── kube-root-ca.crt.yaml
                │   ├── Lease
                │   │   └── kind-control-plane.yaml
                │   ├── Secret
                │   │   └── default-token-69n9s.yaml
                │   └── ServiceAccount
                │       └── default.yaml
                ├── kube-public
                │   ├── ConfigMap
                │   │   ├── cluster-info.yaml
                │   │   └── kube-root-ca.crt.yaml
                │   ├── Role
                │   │   ├── kubeadm:bootstrap-signer-clusterinfo.yaml
                │   │   └── system:controller:bootstrap-signer.yaml
                │   ├── RoleBinding
                │   │   ├── kubeadm:bootstrap-signer-clusterinfo.yaml
                │   │   └── system:controller:bootstrap-signer.yaml
                │   ├── Secret
                │   │   └── default-token-jqv4n.yaml
                │   └── ServiceAccount
                │       └── default.yaml
                ├── kube-system
                │   ├── ConfigMap
                │   │   ├── coredns.yaml
                │   │   ├── extension-apiserver-authentication.yaml
                │   │   ├── kubeadm-config.yaml
                │   │   ├── kubelet-config-1.21.yaml
                │   │   ├── kube-proxy.yaml
                │   │   └── kube-root-ca.crt.yaml
                │   ├── ControllerRevision
                │   │   ├── kindnet-5b547684d9.yaml
                │   │   └── kube-proxy-6bc6858f58.yaml
                │   ├── DaemonSet
                │   │   ├── kindnet.yaml
                │   │   └── kube-proxy.yaml
                │   ├── Deployment
                │   │   ├── coredns.yaml
                │   │   └── stash-stash-enterprise.yaml
                │   ├── Endpoints
                │   │   ├── kube-dns.yaml
                │   │   └── stash-stash-enterprise.yaml
                │   ├── EndpointSlice
                │   │   ├── kube-dns-m2s5c.yaml
                │   │   └── stash-stash-enterprise-k28h6.yaml
                │   ├── Lease
                │   │   ├── kube-controller-manager.yaml
                │   │   └── kube-scheduler.yaml
                │   ├── Pod
                │   │   ├── coredns-558bd4d5db-hdsw9.yaml
                │   │   ├── coredns-558bd4d5db-wk9tx.yaml
                │   │   ├── etcd-kind-control-plane.yaml
                │   │   └── stash-stash-enterprise-567dd95f5b-6xtxg.yaml
                │   ├── ReplicaSet
                │   │   ├── coredns-558bd4d5db.yaml
                │   │   └── stash-stash-enterprise-567dd95f5b.yaml
                │   ├── Role
                │   │   ├── extension-apiserver-authentication-reader.yaml
                │   │   ├── kubeadm:kubelet-config-1.21.yaml
                │   │   └── system::leader-locking-kube-scheduler.yaml
                │   ├── RoleBinding
                │   │   ├── kubeadm:kubelet-config-1.21.yaml
                │   │   ├── kubeadm:nodes-kubeadm-config.yaml
                │   │   ├── kube-proxy.yaml
                │   ├── Service
                │   │   ├── kube-dns.yaml
                │   │   └── stash-stash-enterprise.yaml
                │   └── ServiceAccount
                │       ├── attachdetach-controller.yaml
                │       ├── bootstrap-signer.yaml
                └── local-path-storage
                    ├── ConfigMap
                    │   ├── kube-root-ca.crt.yaml
                    │   └── local-path-config.yaml
                    ├── Deployment
                    │   └── local-path-provisioner.yaml
                    ├── Endpoints
                    │   └── rancher.io-local-path.yaml
                    ├── Pod
                    │   └── local-path-provisioner-547f784dff-jb9tq.yaml
                    ├── ReplicaSet
                    │   └── local-path-provisioner-547f784dff.yaml
                    ├── Secret
                    │   ├── default-token-bnk6x.yaml
                    │   └── local-path-provisioner-service-account-token-fvkxj.yaml
                    └── ServiceAccount
                        ├── default.yaml
                        └── local-path-provisioner-service-account.yaml

77 directories, 776 files
```

Here, the non-namespaced resources have been grouped under the `global` directory and the namespaced resources have been grouped inside the namespace specific folder under the `namespaces` directory.

Let's inspect the YAML of `kubeadm-config.yaml` file under `kube-system` namespace.

```yaml
❯ cat $HOME/Downloads/stash/latest/tmp/resources/namespaces/kube-system/ConfigMap/kubeadm-config.yaml
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

Now, you can use these YAML files to re-create your desired application.

## Cleanup

To cleanup the Kubernetes resources created by this tutorial, run:

```bash
kubectl delete -n demo backupconfiguration cluster-resources-backup
kubectl delete -n demo repository cluster-resource-storage
kubectl delete -n demo serviceaccount cluster-resource-reader
kubectl delete -n demo clusterrole cluster-resource-reader
kubectl delete -n demo clusterrolebinding cluster-resource-reader
```
