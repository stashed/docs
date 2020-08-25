---
title: Local Backend | Stash
description: Configure Stash to Use Local Backend.
menu:
  docs_{{ .version }}:
    identifier: backend-local
    name: Kubernetes Volumes
    parent: backend
    weight: 20
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: guides
---

{{< notice type="warning" message="Local backend is an enterprise feature. You must install Stash Enterprise operator to use local backend." >}}

# Local Backend

`Local` backend refers to a local path inside `stash` sidecar container. Any Kubernetes supported [persistent volume](https://kubernetes.io/docs/concepts/storage/volumes/) such as [PersistentVolumeClaim](https://kubernetes.io/docs/concepts/storage/volumes/#persistentvolumeclaim), [HostPath](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath), [EmptyDir](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir) (for testing only), [NFS](https://kubernetes.io/docs/concepts/storage/volumes/#nfs),  [gcePersistentDisk](https://kubernetes.io/docs/concepts/storage/volumes/#gcepersistentdisk) etc. can be used as local backend.

In order to use Kubernetes volumes as backend, you have to create a `Secret` and a `Repository` object pointing to the desired volume.

### Create Storage Secret

To configure storage secret for local backend, following secret keys are needed:

|        Key        |    Type    |                        Description                         |
| ----------------- | ---------- | ---------------------------------------------------------- |
| `RESTIC_PASSWORD` | `Required` | Password that will be used to encrypt the backup snapshots |

Create storage secret as below,

```bash
$ echo -n 'changeit' > RESTIC_PASSWORD
$ kubectl create secret generic -n demo local-secret --from-file=./RESTIC_PASSWORD
secret/local-secret created
```

### Create Repository

Now, you have to create a `Repository` crd that uses Kubernetes volume as a backend. You have to provide the storage secret that we have created earlier in `spec.backend.storageSecretName` field.

Following parameters are available for `Local` backend.

|      Parameter       |    Type    |                                              Description                                               |
| -------------------- | ---------- | ------------------------------------------------------------------------------------------------------ |
| `local.mountPath`    | `Required` | Path where this volume will be mounted inside the sidecar container. Example: `/safe/data`. <br> <strong>We have put `stash` binary  in the root directory. Hence, you can not use `/stash` or `/stash/*` as `local.mountPath` </strong>
| `local.subPath`      | `Optional` | Sub-path inside the referenced volume where the backed up snapshot will be stored instead of its root. |
| `local.VolumeSource` | `Required` | Any Kubernetes volume. Can be specified inlined. Example: `hostPath`.                                  |

Here, we are going to show some sample `Repository` crds that uses different Kubernetes volume as a backend.

##### HostPath volume as Backend

Below, the YAML of a sample `Repository` crd that uses a `hostPath` volume as a backend.

```yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: local-repo-with-hostpath
  namespace: demo
spec:
  backend:
    local:
      mountPath: /safe/data
      hostPath:
        path: /data/stash-test/repo
    storageSecretName: local-secret
```

Create the `Repository` we have shown above using the following command,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/backends/local_hostPath.yaml
repository/local-repo-with-hostpath created
```

>Note that by default, Stash runs as `non-root` user. `hostPath` volume is writable only for `root` user. So, in order to use `hostPath` volume as backend, either you have to run Stash as `root` user using securityContext or you have to change the permission of the `hostPath` to make it writable for `non-root` users.

##### PersistentVolumeClaim as Backend

Below, the YAML of a sample `Repository` crd that uses a `PersistentVolumeClaim` as a backend.

```yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: local-repo-with-pvc
  namespace: demo
spec:
  backend:
    local:
      mountPath: /safe/data
      persistentVolumeClaim:
        claimName: repo-pvc
    storageSecretName: local-secret
```

Create the `Repository` we have shown above using the following command,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/backends/local_pvc.yaml
repository/local-repo-with-pvc created
```

##### NFS volume as Backend

Below, the YAML of a sample `Repository` crd that uses an `NFS` volume as a backend.

```yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: local-repo-with-nfs
  namespace: demo
spec:
  backend:
    local:
      mountPath: /safe/data
      nfs:
        server: "nfs-service.storage.svc.cluster.local" # use you own NFS server address
        path: "/" # this path is relative to "/exports" path of NFS server
    storageSecretName: local-secret
```

Create the `Repository` we have shown above using the following command,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/backends/local_nfs.yaml
repository/local-repo-with-nfs created
```

>For NFS backend, Stash may have to run the network volume accessor deployments in privileged mode to provide Snapshot listing facility. In this case, please configure network volume accessors by following the instruction [here](/docs/setup/install.md#configuring-network-volume-accessor).

##### GCE PersitentDisk as Backend

Below, the YAML of a sample `Repository` crd that uses a [gcePersistentDisk](https://kubernetes.io/docs/concepts/storage/volumes/#gcepersistentdisk) as a backend.

```yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: local-repo-with-gcepersistentdisk
  namespace: demo
spec:
  backend:
    local:
      mountPath: /safe/data
      gcePersistentDisk:
        pdName: stash-repo
        fsType: ext4
    storageSecretName: local-secret
```

Create the `Repository` we have shown above using the following command,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/backends/local_gcePersistentDisk.yaml
repository/local-repo-with-gcepersistentdisk created
```

>In order to use `gcePersistentDisk` volume as backend, the node where stash container is running must be a GCE VM and the VM must be in same GCE project and zone as the Persistent Disk.

##### AWS EBS volume as Backend

Below, the YAML of a sample `Repository` crd that uses an [awsElasticBlockStore](https://kubernetes.io/docs/concepts/storage/volumes/#awselasticblockstore) as a backend.

```yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: local-repo-with-awsebs
  namespace: demo
spec:
  backend:
    local:
      mountPath: /safe/data
      awsElasticBlockStore: # This AWS EBS volume must already exist.
        volumeID: <volume-id>
        fsType: ext4
    storageSecretName: local-secret
```

Create the `Repository` we have shown above using the following command,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/backends/local_awsElasticBlockStore.yaml
repository/local-repo-with-awsebs created
```

>In order to use `awsElasticBlockStore` volume as backend, the pod where stash container is running must be running on an AWS EC2 instance and the instance must be in the same region and availability-zone as the EBS volume.

##### Azure Disk as Backend

Below, the YAML of a sample `Repository` crd that uses an [azureDisk](https://kubernetes.io/docs/concepts/storage/volumes/#azuredisk) as a backend.

```yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: local-repo-with-azuredisk
  namespace: demo
spec:
  backend:
    local:
      mountPath: /safe/data
      azureDisk:
        diskName: stash.vhd
        diskURI: https://someaccount.blob.microsoft.net/vhds/stash.vhd
    storageSecretName: local-secret
```

Create the `Repository` we have shown above using the following command,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/backends/local_azureDisk.yaml
repository/local-repo-with-azuredisk created
```

##### StorageOS as Backend

Below, the YAML of a sample `Repository` crd that uses a [storageOS](https://kubernetes.io/docs/concepts/storage/volumes/#storageos) volume as a backend.

```yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: local-repo-with-storageos
  namespace: demo
spec:
  backend:
    local:
      mountPath: /safe/data
      storageos:
        volumeName: stash-vol01 # The `stash-vol01` volume must already exist within StorageOS in the `demo` namespace.
        fsType: ext4
    storageSecretName: local-secret
```

Create the `Repository` we have shown above using the following command,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/backends/local_storageOS.yaml
repository/local-repo-with-storageos created
```

##### EmptyDir volume as Backend

Below, the YAML of a sample `Repository` crd that uses an [emptyDir](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir) as a backend.

```yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: local-repo-with-emptydir
  namespace: demo
spec:
  backend:
    local:
      mountPath: /safe/data
      emptyDir: {}
    storageSecretName: local-secret
```

Create the `Repository` we have shown above using the following command,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/backends/emptyDir.yaml
repository/local-repo-with-emptydir created
```

>**Warning:** Data of an `emptyDir` volume is not persistent. If you delete the pod that runs the respective stash container, you will lose all the backed up data. You should use this kind of volumes only to test backup process.

## Next Steps

- Learn how to use Stash to backup workloads data from [here](/docs/guides/latest/workloads/overview.md).
- Learn how to use Stash to backup databases from [here](/docs/guides/latest/addons/overview.md).
- Learn how to use Stash to backup stand-alone PVC from [here](/docs/guides/latest/volumes/overview.md).
