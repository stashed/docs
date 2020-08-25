---
title: Snapshot Overview
menu:
  docs_{{ .version }}:
    identifier: snapshot-overview
    name: Snapshot
    parent: crds
    weight: 50
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: concepts
---
> New to Stash? Please start [here](/docs/concepts/README.md).

# Snapshot

## What is Snapshot

A `Snapshot` is a representation of backup snapshot in a Kubernetes native way. Stash uses an [Aggregated API Server](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/api-machinery/aggregated-api-servers.md) to provide `get` and `list` capabilities for snapshots from the backend.

This enables you to view some useful information such as `creationTimestamp`, `snapshot id`, `backed up path` etc of a snapshot. This also provides the capability to restore a specific snapshot.

## Snapshot structure

Like other official Kuberentes resources, a `Snapshot` has `TypeMeta`, `ObjectMeta` and `Status` sections. However, unlike other Kubernetes resources, it does not have a `Spec` section.

A sample `Snapshot` object is shown below,

```yaml
apiVersion: repositories.stash.appscode.com/v1alpha1
kind: Snapshot
metadata:
  creationTimestamp: "2020-07-25T17:41:31Z"
  labels:
    hostname: app
    repository: minio-repo
  name: minio-repo-b54ee4a0
  namespace: demo
  uid: b54ee4a0e9c9084696dc976f125c4fd0e6b1a31abfd82cfc857b3bc9e559fa2f
status:
  gid: 0
  hostname: app
  paths:
  - /var/lib/html
  repository: minio-repo
  tree: 11527d99281bf3725d58cd637d1f3c19ab9d397d6cff1887a1cd1f9c8c5ebb80
  uid: 0
  username: ""
```

Here, we are going to describe the various sections of a `Snapshot` object.

### Snapshot `Metadata`

- **metadata.name**

  `metadata.name` specifies the name of the `Snapshot` object. It follows the following pattern, `<Repository crd name>-<first 8 digits of snapshot id>`.

- **metadata.uid**

  `metadata.uid` specifies the complete id of the respective restic snapshot in the backend.

- **metadata.creationTimestamp**

  `metadata.creationTimestamp` represents the time when the snapshot was created.

- **metadata.labels**

  A `Snapshot` object holds `repository` and `hostname` as a label in `metadata.labels` section. This helps a user to query the snapshots of a particular repository and/or a particular host.

### Snapshot `Status`

`Snapshot` object has the following fields in `.status` section:

- **status.gid**
`status.gid` indicates the group identifier of the user who took this backup.

- **status.hostname**
`status.hostname` indicates the host identifier whose data has been backed up in this snapshot. In order to know how this host identifier are generated, please visit [here](/docs/concepts/crds/backupsession.md#hosts-of-a-backup-process).

- **status.paths**
`status.paths` indicates the paths that have been backed up in this snapshot.

- **status.repository**
`status.repository` indicates the name of the Repository crd where this Snapshot came from.

- **status.tree**
`status.tree` indicates `tree` of the restic snapshot. For more details, please visit [here](https://restic.readthedocs.io/en/stable/100_references.html#trees-and-data).

- **status.uid**
`status.uid` indicates `uid` of the user who took this backup. For `root` user it is 0.

- **status.username**
`status.username` indicates the name of the user who runs the backup process that took the backup.

- **status.tags**
`status.tags` indicates the tags of the snapshot.

## Working with Snapshot

In this section, we are going to show different types of operations you can perform on the Snapshots.

**Listing Snapshots:**

Stash lists Snapshots directly from the backend. This operation sometime takes more time than the default request timeout. So, we are going to increase the request timeout through `--request-timeout` flag in the list commands.

```bash
# List Snapshots of all Repositories in the current namespace
$ kubectl get snapshot --request-timeout=300s

# List Snapshots of all Repositories of all namespaces
$ kubectl get snapshot --all-namespaces --request-timeout=300s

# List Snapshots of all Repositories of a particular namespace
$ kubectl get snapshot -n demo --request-timeout=300s

# List Snapshots of a particular Repository
$ kubectl get snapshot -l repository=local-repo --request-timeout=300s

# List Snapshots from multiple Repositories
$ kubectl get snapshot -l 'repository in (local-repo,gcs-repo)' --request-timeout=300s

# List Snapshots of a particular host
$ kubectl get snapshot -l hostname=db --request-timeout=300s

# List Snapshots of a particular Repository and particular host
$ kubectl get snapshot -l repository=local-repo,hostname=db --request-timeout=300s
```

**Viewing information of a particular Snapshot:**

```bash
$ kubectl get snapshot [-n <namespace>] <snapshot name> -o yaml

# Example:
$ kubectl get snapshot -n demo local-repo-02b0ed42 -o yaml
```

## Preconditions for Snapshot

1. Stash provides `Snapshots` listing facility with the help of an Aggregated API Server. Your cluster must support Aggregated API Server. Otherwise, you won't be able to perform `get` or `list` operation on `Snapshot`.

2. If you are using [local](/docs/guides/latest/backends/local.md) backend, the respective pod that took the backup must be in `Running` state. It is not necessary if you use cloud backends.

## Next Steps

- Learn how to configure `BackupConfiguration` to backup workloads data from [here](/docs/guides/latest/workloads/overview.md).
- Learn how to configure `BackupConfiguration` to backup databases from [here](/docs/guides/latest/addons/overview.md).
- Learn how to configure `BackupConfiguration` to backup stand-alone PVC from [here](/docs/guides/latest/volumes/overview.md).
