---
title: Backup resource YAMLs of a Namespace | Stash
description: Take backup of resources YAMLs of a Namespace using Stash
menu:
  docs_{{ .version }}:
    identifier: stash-kubedump-namespace
    name: Namespace Backup
    parent: stash-kubedump
    weight: 30
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: stash-addons
---

# Backup resource YAMLs of a Namespace using Stash

Stash `{{< param "info.version" >}}` supports taking backup of the resource YAMLs using `kubedump` plugin. This guide will show you how you can take a backup of the resource YAMLs of a particular namespace using Stash.

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster.
- Install Stash Enterprise in your cluster following the steps [here](/docs/setup/install/enterprise/index.md).
- Install Stash `kubectl` plugin in your local machine following the steps [here](/docs/setup/install/kubectl-plugin/index.md).
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

> Note: YAML files used in this tutorial are stored [here](https://github.com/stashed/docs/tree/{{< param "info.version" >}}/docs/addons/kubedump/namespace/examples).

### Prepare for Backup

In this section, we are going to configure a backup for all the resource YAMLs of `kube-system` namespace.

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
  name: namespace-resource-storage
  namespace: demo
spec:
  backend:
    gcs:
      bucket: stash-testing
      prefix: /manifests/namespace/kube-system
    storageSecretName: gcs-secret
```

Let's create the `Repository` we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/addons/kubedump/namespace/examples/repository.yaml
repository.stash.appscode.com/namespace-resource-storage created
```

#### Create RBAC

The `kubedump` plugin requires read permission for all the resources of the desired namespace. By default, Stash does not grant such permissions. We have to provide the necessary permissions manually.

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

Here, we have give permission to read all the cluster resources. You can restrict this permission to a particular namespace only.

Let's create the RBAC resources we have shown above,

```bash
❯ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/addons/kubedump/namespace/examples/rbac.yaml
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
  name: kube-system-backup
  namespace: demo
spec:
  schedule: "*/5 * * * *"
  task:
    name: kubedump-backup-0.1.0
  repository:
    name: namespace-resource-storage
  target:
    ref:
      apiVersion: v1
      kind: Namespace
      name: kube-system
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
- `.spec.target` specifies the targeted Namespace that we are going to backup.
- `.spec.runtimeSettings.pod.serviceAccountName` specifies the ServiceAccount name that we have created earlier with cluster-wide resource reading permission.
- `.spec.retentionPolicy` specifies a policy indicating how we want to cleanup the old backups.

Let's create the `BackupConfiguration` object we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/addons/kubedump/namespace/examples/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/kube-system-backup created
```

#### Verify Backup Setup Successful

If everything goes well, the phase of the `BackupConfiguration` should be `Ready`. The `Ready` phase indicates that the backup setup is successful. Let's verify the `Phase` of the BackupConfiguration,

```bash
❯ kubectl get backupconfiguration -n demo
NAME                 TASK                    SCHEDULE      PAUSED   PHASE   AGE
kube-system-backup   kubedump-backup-0.1.0   */5 * * * *            Ready   8s
```

#### Verify CronJob

Stash will create a CronJob with the schedule specified in `spec.schedule` field of `BackupConfiguration` object.

Verify that the CronJob has been created using the following command,

```bash
❯ kubectl get cronjob -n demo
NAME                               SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
stash-trigger-kube-system-backup   */5 * * * *   False     0        <none>          25s
```

#### Wait for BackupSession

The `stash-trigger-kube-system-backup` CronJob will trigger a backup on each scheduled slot by creating a `BackupSession` object.

Now, wait for a schedule to appear. Run the following command to watch for a `BackupSession` object,

```bash
❯ kubectl get backupsession -n demo -w
NAME                            INVOKER-TYPE          INVOKER-NAME         PHASE   DURATION   AGE
kube-system-backup-1652247300   BackupConfiguration   kube-system-backup                      0s
kube-system-backup-1652247300   BackupConfiguration   kube-system-backup   Pending              0s
kube-system-backup-1652247300   BackupConfiguration   kube-system-backup   Running              0s
kube-system-backup-1652247300   BackupConfiguration   kube-system-backup   Running              17s
kube-system-backup-1652247300   BackupConfiguration   kube-system-backup   Succeeded   1m11s      71s
```

Here, the phase `Succeeded` means that the backup process has been completed successfully.

#### Verify Backup

Now, we are going to verify whether the backed-up data is present in the backend or not. Once a backup is completed, Stash will update the respective `Repository` object to reflect the backup completion. Check that the repository `namespace-resource-storage` has been updated by the following command,

```bash
❯ kubectl get repository -n demo
NAME                         INTEGRITY   SIZE          SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
namespace-resource-storage   true        407.949 KiB   1                95s                      13m
```

Now, if we navigate to the GCS bucket, we will see the backed up data has been stored in `/manifest/namespace/kube-system` directory as specified by `.spec.backend.gcs.prefix` field of the `Repository` object.

<figure align="center">
  <img alt="Backup data in GCS Bucket" src="/docs/addons/kubedump/namespace/images/namespace_manifests_backup.png">
  <figcaption align="center">Fig: Backup data in GCS Bucket</figcaption>
</figure>

> Note: Stash keeps all the backed-up data encrypted. So, data in the backend will not make any sense until they are decrypted.

## Restore

Stash does not provide any automatic mechanism to restore the cluster resources from the backed-up YAMLs. Your application might be managed by Helm or by an operator. In such cases, just applying the YAMLs is not enough to restore the application. Furthermore, there might be an order issue. Some resources must be applied before others. It is difficult to generalize and codify various application-specific logic.

Therefore, it is the user's responsibility to download the backed-up YAMLs and take the necessary steps based on his application to restore it properly.

### Download the YAMLs

Stash provides a [kubectl plugin](https://stash.run/docs/v2022.05.12/guides/cli/cli/#download-snapshots) for making it easy to download a snapshot locally.

Now, let's download the latest Snapshot from our backed-up data into the `$HOME/Downloads/stash/namespace/kube-system` folder of our local machine.

```bash
❯ kubectl stash download -n demo namespace-resource-storage  --destination=$HOME/Downloads/stash/namespace/kube-system --snapshots="latest"
```

Now, lets use [tree](https://linux.die.net/man/1/tree) command to inspect downloaded YAMLs files.

```bash
❯ tree $HOME/Downloads/stash/namespace/kube-system
/home/emruz/Downloads/stash/namespace/kube-system
└── latest
    └── tmp
        └── resources
            ├── ConfigMap
            │   ├── coredns.yaml
            │   ├── extension-apiserver-authentication.yaml
            │   ├── kubeadm-config.yaml
            │   ├── kubelet-config-1.21.yaml
            │   ├── kube-proxy.yaml
            │   └── kube-root-ca.crt.yaml
            ├── ControllerRevision
            │   ├── kindnet-5b547684d9.yaml
            │   └── kube-proxy-6bc6858f58.yaml
            ├── DaemonSet
            │   ├── kindnet.yaml
            │   └── kube-proxy.yaml
            ├── Deployment
            │   ├── coredns.yaml
            │   └── stash-stash-enterprise.yaml
            ├── Endpoints
            │   ├── kube-dns.yaml
            │   └── stash-stash-enterprise.yaml
            ├── EndpointSlice
            │   ├── kube-dns-m2s5c.yaml
            │   └── stash-stash-enterprise-k28h6.yaml
            ├── Lease
            │   ├── kube-controller-manager.yaml
            │   └── kube-scheduler.yaml
            ├── Pod
            │   ├── coredns-558bd4d5db-hdsw9.yaml
            │   ├── coredns-558bd4d5db-wk9tx.yaml
            │   ├── etcd-kind-control-plane.yaml
            │   ├── kindnet-69whw.yaml
            │   ├── kube-apiserver-kind-control-plane.yaml
            │   ├── kube-controller-manager-kind-control-plane.yaml
            │   ├── kube-proxy-p7j9f.yaml
            │   ├── kube-scheduler-kind-control-plane.yaml
            │   └── stash-stash-enterprise-567dd95f5b-6xtxg.yaml
            ├── ReplicaSet
            │   ├── coredns-558bd4d5db.yaml
            │   └── stash-stash-enterprise-567dd95f5b.yaml
            ├── Role
            │   ├── extension-apiserver-authentication-reader.yaml
            │   ├── kubeadm:kubelet-config-1.21.yaml
            │   ├── kubeadm:nodes-kubeadm-config.yaml
            │   ├── kube-proxy.yaml
            ├── RoleBinding
            │   ├── kubeadm:kubelet-config-1.21.yaml
            │   ├── kubeadm:nodes-kubeadm-config.yaml
            │   ├── kube-proxy.yaml
            ├── Secret
            │   ├── attachdetach-controller-token-w68zv.yaml
            │   ├── bootstrap-signer-token-j6q2c.yaml
            │   ├── certificate-controller-token-d5dvw.yaml
            │   ├── clusterrole-aggregation-controller-token-77x8n.yaml
            ├── Service
            │   ├── kube-dns.yaml
            │   └── stash-stash-enterprise.yaml
            └── ServiceAccount
                ├── attachdetach-controller.yaml
                ├── bootstrap-signer.yaml
                ├── certificate-controller.yaml
                ├── clusterrole-aggregation-controller.yaml
                ├── coredns.yaml
                ├── cronjob-controller.yaml
                ├── daemon-set-controller.yaml
                ├── default.yaml
                ├── deployment-controller.yaml

17 directories, 131 files
```

Here, the resources has been grouped under the respective Kind folder.

Let's inspect the YAML of `coredns.yaml` file under ConfigMap folder,

```yaml
❯ cat $HOME/Downloads/stash/namespace/kube-system/latest/tmp/resources/ConfigMap/coredns.yaml
apiVersion: v1
data:
  Corefile: |
    .:53 {
        errors
        health {
           lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
           ttl 30
        }
        prometheus :9153
        forward . /etc/resolv.conf {
           max_concurrent 1000
        }
        cache 30
        loop
        reload
        loadbalance
    }
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system

```

Now, you can use these YAML files to re-create your desired application.

## Cleanup

To cleanup the Kubernetes resources created by this tutorial, run:

```bash
kubectl delete -n demo backupconfiguration kube-system-backup
kubectl delete -n demo repository namespace-resource-storage
kubectl delete -n demo serviceaccount cluster-resource-reader
kubectl delete -n demo clusterrole cluster-resource-reader
kubectl delete -n demo clusterrolebinding cluster-resource-reader
```
