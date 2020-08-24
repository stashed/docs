---
title: RestoreBatch Overview
  docs_{{ .version }}:
    identifier: restorebatch-overview
    name: RestoreBatch
    parent: crds
    weight: 27
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: concepts
---

{{< notice type="warning" message="RestoreBatch is an enterprise feature. You must install Stash Enterprise operator to use RestoreBatch." >}}

# RestoreBatch

## What is RestoreBatch

A `RestoreBatch` is a Kubernetes `CustomResourceDefinition`(CRD) which allows you to restore multiple co-related components( eg, workloads, databases, etc.) together that were backed up via a `BackupBatch`.

## RestoreBatch CRD Specification

Like any official Kubernetes resource, a `RestoreBatch` has `TypeMeta`, `ObjectMeta`, `Spec` and `Status` sections. A sample `RestoreBatch` object to restore multiple co-related components is shown below:

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: RestoreBatch
metadata:
  name: batch-restore
  namespace: demo
spec:
  driver: Restic
  repository:
    name: minio-repo
  executionOrder: Parallel
  members:
  - target:
      alias: app
      ref:
        apiVersion: apps/v1
        kind: Deployment
        name: wordpress
      rules:
      - paths:
        - /var/www/html
      volumeMounts:
      - mountPath: /var/www/html
        name: wordpress-persistent-storage
  - target:
      alias: db
      ref:
        apiVersion: appcatalog.appscode.com/v1alpha1
        kind: AppBinding
        name: wordpress-mysql
      rules:
      - snapshots:
        - latest
    task:
      name: mysql-restore-8.0.14
status:
  phase: Succeeded
  sessionDuration: 15.145032437s
  conditions:
  - lastTransitionTime: "2020-07-25T17:41:52Z"
    message: Repository demo/minio-repo exist.
    reason: RepositoryAvailable
    status: "True"
    type: RepositoryFound
  - lastTransitionTime: "2020-07-25T17:41:52Z"
    message: Backend Secret demo/minio-secret exist.
    reason: BackendSecretAvailable
    status: "True"
    type: BackendSecretFound
  members:
  - conditions:
    - lastTransitionTime: "2020-07-25T17:41:52Z"
      message: Restore target apps/v1 deployment/wordpress
        found.
      reason: TargetAvailable
      status: "True"
      type: RestoreTargetFound
    - lastTransitionTime: "2020-07-25T17:41:52Z"
      message: Successfully injected stash init-container.
      reason: InitContainerInjectionSucceeded
      status: "True"
      type: StashInitContainerInjected
    phase: Succeeded
    ref:
      apiVersion: apps/v1
      kind: Deployment
      name: wordpress
    stats:
    - duration: 822.861322ms
      hostname: app
      phase: Succeeded
    totalHosts: 1
  - conditions:
    - lastTransitionTime: "2020-07-25T17:41:52Z"
      message: Restore target appcatalog.appscode.com/v1alpha1 appbinding/wordpress-mysql
        found.
      reason: TargetAvailable
      status: "True"
      type: RestoreTargetFound
    - lastTransitionTime: "2020-07-25T17:41:52Z"
      message: Successfully created restore job.
      reason: RestoreJobCreationSucceeded
      status: "True"
      type: RestoreJobCreated
    phase: Succeeded
    ref:
      apiVersion: appcatalog.appscode.com/v1alpha1
      kind: AppBinding
      name: wordpress-mysql
    stats:
    - duration: 4.597812876s
      hostname: db
      phase: Succeeded
    totalHosts: 1
```

Here, we are going to describe the various sections of `RestoreBatch` crd.

### RestoreBatch `Spec`

A `RestoreBatch` object has the following fields in the `spec` section.

#### spec.driver

`spec.driver` indicates the mechanism used to restore. Currently, Stash supports `Restic` and `VolumeSnapshotter` as drivers. The default value of this field is `Restic`. For more details, please see [here](/docs/concepts/crds/restoresession.md#specdriver).

#### spec.repository

`spec.repository.name` indicates the `Repository` crd name that holds necessary backend information from where data will be restored.

#### spec.executionOrder

`spec.executionOrder` specifies whether Stash should restore the targets sequentially or parallelly. If `spec.executionOrder` is set to `Parallel`, Stash will start to restore of all the targets simultaneously. If it is set to `Sequential`, Stash will not start restoring a target until all the previous members have completed their restore process. The default value of this field is `Parallel`.

#### spec.members

`spec.members` field specifies a list of targets to restore. Each member consists of the following fields:

- **target :** Each member has a target specification. The target specification of a member is the same as the target specification of a `RestoreSession` explained [here](/docs/concepts/crds/restoresession.md#spectarget).

- **task :** `task` specifies the name and parameters of the [Task](/docs/concepts/crds/task.md) crd to use to restore the member. For more details, please see [here](/docs/concepts/crds/restoresession.md#spectask).

- **runtimeSettings :** `runtimeSettings` allows to configure runtime environment for the restore init-container or job. You can specify runtime settings at both pod level and container level. For more details, please see [here](/docs/concepts/crds/restoresession.md#specruntimesettings).

- **tempDir :** Stash mounts an `emptyDir` for holding temporary files. It is also used for `caching` for faster restore performance. You can configure the `emptyDir` using `tempDir` section. You can also disable `caching` using this field. For more details, please see [here](/docs/concepts/crds/restoresession.md#spectempdir).

- **interimVolumeTemplate :** For some targets (i.e. some databases), Stash can't directly pipe the restored data into the target. In this case, it has to store the restored data temporarily before injecting into the target. `spec.interimVolumeTemplate` specifies a PVC template for holding those data temporarily. Stash will create a PVC according to the template and use it to store the data temporarily. This PVC will be deleted automatically if you delete the `RestoreBatch`.

- **hooks :** Each member has it's own `hooks` field which allows you to execute member-specific pre-restore or post-restore hooks. For more details about hooks, please visit [here](/docs/concepts/crds/restoresession.md#spechooks).

#### spec.hooks

`spec.hooks` allows performing some global actions before and after the restoration process. You can send HTTP requests to a remote server via `httpGet` or `httpPost`. You can check whether a TCP port is open using `tcpSocket` hooks. You can also execute some commands using `exec` hook.

- **spec.hooks.preRestore:** `spec.hooks.preRestore` hooks are executed before restoring any of the members.
- **spec.hooks.postRestore:** `spec.hooks.postRestore` hooks are executed after restoring all the members.

For more details on how hooks work in Stash and how to configure different types of hook, please visit [here](/docs/guides/latest/hooks/overview.md).

### RestoreBatch `Status`

A `RestoreBatch` object has the following fields in the `status` section.

- **phase :** `phase` shows the overall phase of the restore process. It will be `Succeeded` only if all the targets are restored successfully.

- **sessionDuration :** `sessionDuration` shows the total time taken to complete the entire restoration process.

- **conditions :** The `conditions` field shows conditions of different steps of the restore process. The following conditions are set by the Stash operator for a RestoreBatch:

| Condition Type                   | Usage                                                                         |
| -------------------------------- | ----------------------------------------------------------------------------- |
| `RepositoryFound`                | Indicates whether the respective Repository object was found or not.          |
| `BackendSecretFound`             | Indicates whether the respective backend secret was found or not.             |
| `GlobalPreRestoreHookSucceeded`  | Indicates whether the global PreRestoreHook was executed successfully or not. |
| `GlobalPostRestoreHookSucceeded` | Indicates whether the global PostRestoreHook was executed successfully or no. |

- **members :** `members` section shows the restore status of the individual members. Each entry has the following fields:
  - **ref :** `ref` points to the respective target whose status is shown here.
  - **phase :** `phase` shows the restore phase of the member.
  - **conditions:** `conditions` shows the conditions of different steps of restoring this member. Stash set the following conditions for each restore members.

    | Condition Type               | Usage                                                                                                                                    |
    | ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
    | `RestoreTargetFound`         | Indicates whether the restore target was found or not.                                                                                   |
    | `StashInitContainerInjected` | Indicates whether stash init-container was injected into the targeted workload. This condition is applicable only for the sidecar model. |
    | `RestoreJobCreated`          | Indicates whether the restore job was created or not. This condition is only applicable in the job model.                                |

  - **totalHosts :** `totalHosts` field specifies the total number of hosts that will be restored for this member.
  - **stats :** `stats` section is an array of restore statistics of individual hosts. Individual host stats entry consists of the following fields:
    - **hostname :** `hostname` indicates the name of the host. Usually it is the `alias` or `alias-<workload-specific-suffix>`. For more details, please visit [here](/docs/concepts/crds/backupsession.md#hosts-of-a-backup-process).
    - **phase :** `phase` indicates the restore phase of this host.
    - **duration :** `duration` indicates the total time taken to complete the restore process for this host.
    - **error :** `error` shows the reason for failure if the restore process fails for this host.
