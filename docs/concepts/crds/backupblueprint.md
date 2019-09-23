---
title: BackupBlueprint Overview
menu:
  product_stash_{{ .version }}:
    identifier: backupblueprint-overview
    name: BackupBlueprint
    parent: crds
    weight: 40
product_name: stash
menu_name: product_stash_{{ .version }}
section_menu_id: concepts
---

> New to Stash? Please start [here](/docs/concepts/README.md).

# BackupBlueprint

## What is BackupBlueprint

Stash uses 1-1 mapping among `Repository`, `BackupConfiguration` and the target. So, whenever you want to backup a target(workload/PV/PVC/database), you have to create a `Repository` and `BackupConfiguration` object. This could become tiresome when you are trying to backup similar types of target and the `Repository` and `BackupConfiguration` has only slight difference. To mitigate this problem, Stash provides a way to specify a blueprint for these two objects via `BackupBlueprint` crd.

A `BackupBlueprint` is a Kubernetes `CustomResourceDefinition`(CRD) which specifies a blueprint for `Repository` and `BackupConfiguration` in a Kubernetes native way.

You have to create only one  `BackupBlueprint` for all similar types of workloads (i.e. Deployment, DaemonSet, StatefulSet etc.). Then, you just need to add some annotations in the target workload. Stash will automatically create respective `Repository`, `BackupConfiguration` object using the blueprint. In Stash parlance, we call this process as **auto backup**.

## BackupBlueprint CRD Specification

Like any official Kubernetes resource, a `BackupBlueprint` has `TypeMeta`, `ObjectMeta` and `Spec` sections. However, unlike other Kubernetes resources, it does not have a `Status` section.

A sample `BackupBlueprint` object to auto backup the volumes of a Deployment is shown below,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupBlueprint
metadata:
  name: workload-backup-blueprint
spec:
  # ============== Blueprint for Repository ==========================
  backend:
    gcs:
      bucket: stash-backup
      prefix: stash/${TARGET_NAMESPACE}/${TARGET_KIND}/${TARGET_NAME}
    storageSecretName: gcs-secret
  wipeOut: false
  # ============== Blueprint for BackupConfiguration =================
  schedule: "* * * * *"
  backupHistoryLimit: 3
  # task: # no task section is required for workload data backup
  #   name: workload-backup
  runtimeSettings:
    container:
      securityContext:
        runAsUser: 2000
        runAsGroup: 2000
  tempDir:
    medium: "Memory"
    size:  "1Gi"
    disableCache: false
  retentionPolicy:
    name: 'keep-last-5'
    keepLast: 5
    prune: true
```

The sample `BackupBlueprint` that has been shown above can be used to backup Deployments, DaemonSets, StatefulSets, ReplicaSets and ReplicationControllers. You only have to add some annotations to these workloads. For more details on how auto backup works in Stash, please visit [here](/docs/guides/latest/auto-backup/overview.md).

Here, we are going to describe the various sections of `BackupBlueprint` crd.

### BackupBlueprint `Spec`

We can divide BackupBlueprint's `.spec` section into two parts. One part specifies a blueprint for `Repository` object and other specifies a blueprint for `BackupConfiguration` object.

#### Repository Blueprint

You can configure `Repository` blueprint using `spec.backend` field and `spec.wipeOut` field.

- **spec.backend :** `spec.backend` field is backend specification similar to [spec.backend](/docs/concepts/crds/repository.md#specbackend) field of a `Repository` crd. There is only one difference. You can now templatize `prefix` section (`subPath` for local volume) of the backend to store backed up data of different workloads at different directory. You can use the following variables to templatize `spec.backend` field:

|       Variable        |                                                              Usage                                                              |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `TARGET_API_VERSION`  | API version of the target                                                                                                       |
| `TARGET_KIND`         | Resource kind of the target                                                                                                     |
| `TARGET_NAMESPACE`    | Namespace of the target                                                                                                         |
| `TARGET_NAME`         | Name of the target                                                                                                              |
| `TARGET_RESOURCE`     | Plural form of the target kind. i.e. `deployments`, `statefulsets` etc.                                                         |

The following variables are available only for database backup.

|       Variable        |                                                              Usage                                                              |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `TARGET_APP_GROUP`    | Represents the application group where the respective app belongs (i.e: `kubedb.com`).                                          |
| `TARGET_APP_RESOURCE` | Represents the resource kind under that application group that the respective app works with (i.e: `postgres`).                 |
| `TARGET_APP_TYPE`     | Represents the total types of the application. It's simply `TARGET_APP_GROUP/TARGET_APP_RESOURCE` (i.e: `kubedb.com/postgres`). |

  If you use the sample `BackupBlueprint` that has been shown above to backup a Deployment named `my-deploy` of `test` namespace, the backed up data will be stored in `stash/test/deployment/my-deploy` directory of the `stash-backup` bucket. Similarly, if you want to backup a StatefulSet with name `my-sts` of same namespace, the backed up data will be stored in `/stash/test/statefulset/my-sts` directory of the backend.

- **spec.backend.\<backend type>.storageSecretName:** specifies the name of the secret that holds the access credentials to the backend.

>Note: `BackupBlueprint` is a non-namespaced crd. So, you can use a `BackupBlueprint` to backup targets in multiple namespaces. However, Storage Secret is a namespaced object. So, you have to manually create the secret in each namespace where you have a target for backup.

- **spec.wipeOut :** `spec.wipeOut` indicates whether Stash should delete the respective backed up data from the backend if a user deletes a `Repository` crd. For more details, please visit [here](/docs/concepts/crds/repository.md#specwipeout).

#### BackupConfiguration Blueprint

You can provide a blueprint for the `BackupConfiguration` object that will be created for respective target using the following fields:

- **spec.schedule :** `spec.schedule` is the schedule that will be used to create `BackupConfiguration` for respective target. For more details, please visit [here](/docs/concepts/crds/backupconfiguration.md#specschedule).

- **spec.backupHistoryLimit :** `spec.backupHistoryLimit` specifies a limit for backup history to keep for debugging purpose. For more details, please visit [here](/docs/concepts/crds/backupconfiguration.md#specbackuphistroylimit).

- **spec.task :** `spec.task` specifies the name and the parameters of [Task](/docs/concepts/crds/task.md) to use to backup the target. You can template the name field with `TARGET_APP_VERSION` variable for database backup. Stash will replace this variable with respective database version. This will allow you to backup multiple database versions with the same `BackupBlueprint`. For more details, please check the following [guide](/docs/guides/latest/auto-backup/database.md).

- **spec.runtimeSettings :** `spec.runtimeSettings` allows to configure runtime environment for backup sidecar or job. For more details, please visit [here](/docs/concepts/crds/backupconfiguration.md#specruntimesettings).

- **spec.tempDir :** `spec.tempDir` specifies the temporary volume setting that will be used to create respective `BackupConfiguration` object. For more details, please visit [here](/docs/concepts/crds/backupconfiguration.md#spectempdir).

- **spec.interimVolumeTemplate :** `spec.interimVolumeTemplate` specifies a PVC template for holding data temporarily before uploading to the backend. For more details, please visit [here](/docs/concepts/crds/backupconfiguration.md#specinterimvolumetemplate).

- **spec.retentionPolicy :** `spec.retentionPolicy` specifies the retention policies that will be used to create respective `BackupConfiguration` object. For more details, please visit [here](/docs/concepts/crds/backupconfiguration.md#specretentionpolicy).

## Next Steps

- Learn how to use `BackupBlueprint` for auto backup of workloads data from [here](/docs/guides/latest/auto-backup/workload.md).
- Learn how to use `BackupBlueprint` for auto backup of database from [here](/docs/guides/latest/auto-backup/database.md).
- Learn how to use `BackupBlueprint` for auto backup of stand-alone PVC from [here](/docs/guides/latest/auto-backup/pvc.md).
