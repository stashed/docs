---
title: Repository Overview
menu:
  docs_{{ .version }}:
    identifier: repository-overview
    name: Repository
    parent: crds
    weight: 5
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: concepts
---

> New to Stash? Please start [here](/docs/concepts/README.md).

# Repository

## What is Repository

A `Repository` is a Kubernetes `CustomResourceDefinition`(CRD) which represents [backend](/docs/guides/backends/overview/index.md) information in a Kubernetes native way.

You have to create a `Repository` object for each backup target. Since v1beta1 api, a `Repository` object has 1-1 mapping with a target. Thus, only one target can be backed up into one `Repository`.

## Repository CRD Specification

Like any official Kubernetes resource, a `Repostiory` object has `TypeMeta`, `ObjectMeta`, `Spec` and `Status` sections.

A sample `Repository` object that uses Google Cloud Storage(GCS) bucket as backend is shown below:

```yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: gcs-demo-repo
  namespace: demo
spec:
  backend:
    gcs:
      bucket: stash-demo-backup
      prefix: demo
    storageSecretName: gcs-secret
  usagePolicy:
    allowedNamespaces:
      from: Same
  wipeOut: false
status:
  firstBackupTime: "2019-04-15T06:08:16Z"
  integrity: true
  lastBackupTime: "2019-04-15T06:14:15Z"
  totalSize: 2.567 KiB
  snapshotCount: 5
  snapshotsRemovedOnLastCleanup: 1
```

Here, we are going to describe the various sections of the `Repository` crd.

### Repository `Spec`

`Repository` CRD has the following fields in the `.spec` section.

- **spec.backend**
`spec.backend` specifies the storage location where the backed up snapshots will be stored. To learn how to configure `Repository` crd for  various backends, please visit [here](/docs/guides/backends/overview/index.md).

- **backend prefix/subPath**
`prefix` of any backend denotes the directory inside the backend where the backed up snapshots will be stored. In case of **Local** backend, `subPath` is used for this purpose.

- **spec.wipeOut**
As the name implies, `spec.wipeOut` field indicates whether Stash operator should delete backed up files from the backend when `Repository` crd is deleted. The default value of this field is `false` which tells Stash to not delete backed up data when a user deletes a `Repository` crd.

- **spec.usagePolicy**
This lets you control which namespaces are allowed to use the Repository and which are not. If you refer to a Repository from a restricted namespace, Stash will reject creating the respective BackupConfiguration/RestoreSession from validating webhook. You can use the `usagePolicy` to allow only the same namespace, a subset of namespaces, or all the namespaces to refer to the Repository. If you don't specify any `usagePolicy`, Stash will allow referencing the Repository only from the namespace where the Repository has been created.


Here is an example of `spec.usagePolicy` that limits referencing the Repository only from the same namespace,
```yaml
spec: 
  usagePolicy:
    allowedNamespaces:
      from: Same
```

Here is an example of `spec.usagePolicy` that allows referencing it from all namespaces,
```yaml
spec: 
  usagePolicy:
    allowedNamespaces:
      from: All
```

Here is an example of `spec.usagePolicy` that allows referencing it from only `prod` and `staging` namespace,
```yaml
spec:
  usagePolicy:
    allowedNamespaces:
      from: Selector
      selector:
        matchExpressions:
        - key: "kubernetes.io/metadata.name"
          operator: In
          values: ["prod","staging"]
```

### Repository `Status`

Stash operator updates `.status` of a Repository crd every time a backup operation is completed. `Repository` crd shows the following statistics in status section:

- **status.firstBackupTime**
`status.firstBackupTime` indicates the timestamp when the first backup was taken.

- **status.lastBackupTime**
`status.lastBackupTime` indicates the timestamp when the latest backup was taken.

- **status.integrity**
Stash checks the integrity of backed up files after each backup. `status.integrity` shows the result of the integrity check.

- **status.totalSize**
`status.totalSize` shows the total size of a repository after last backup.

- **status.snapshotCount**
`status.SnapshotCount` shows the number of snapshots stored in the Repository.

- **status.snapshotsRemovedOnLastCleanup**
`status.snapshotsRemovedOnLastCleanup` shows the number of old snapshots that has been cleaned up according to retention policy on last backup session.


## Deleting Repository

Stash allows users to delete **only `Repository` crd** or **`Repository` crd along with respective backed up data**. Here, we are going to show how to perform these operations.

**Delete only `Repository` keeping backed up data :**

 You can delete only `Repository` crd by,

```bash
$ kubectl delete repository <repository-name>

# Example
$ kubectl delete repository gcs-demo-repo
repository "gcs-demo-repo" deleted
```

This will delete only `Repository` crd. It won't delete any backed up data from the backend. You can recreate the `Repository` object later to reuse existing data as long as your restic password in unchanged.

>If you delete `Repository` crd while respective stash sidecar still exists on the workload, it will fail to take further backup.

**Delete `Repository` along with backed up data :**

In order to prevent the users from accidentally deleting backed up data, Stash uses a special `wipeOut` flag in `spec` section of `Repository` crd. By default, this flag is set to `wipeOut: false`. If you want to delete respective backed up data from backend while deleting `Repository` crd, you must set this flag to `wipeOut: true`.

> Currently, Stash does not support wiping out backed up data for local backend. If you want to cleanup backed up data from local backend, you must do it manually.

Here, is an example of deleting backed up data from GCS backend,

- First, set `wipeOut: true` by patching `Repository` crd.

  ```bash
  $ kubectl patch repository gcs-demo-repo --type="merge" --patch='{"spec": {"wipeOut": true}}'
  repository "gcs-demo-repo" patched
  ```

- Finally, delete `Repository` object. It will delete backed up data from the backend.

  ```bash
  $ kubectl delete repository gcs-demo-repo
  repository "gcs-demo-repo" deleted
  ```

You can browse your backend storage bucket to verify that the backed up data has been wiped out.

## Next Steps

- Learn how to create `Repository` crd for different backends from [here](/docs/guides/backends/overview/index.md).
- Learn how Stash backup workloads data from [here](/docs/guides/workloads/overview/index.md).
- Learn how Stash backup databases from [here](/docs/guides/addons/overview/index.md).
