---
title: RestoreSession Overview
menu:
  product_stash_0.8.3:
    identifier: restoresession-overview
    name: RestoreSession
    parent: crds
    weight: 25
product_name: stash
menu_name: product_stash_0.8.3
section_menu_id: concepts
---

> New to Stash? Please start [here](/docs/concepts/README.md).

# RestoreSession

## What is RestoreSession

A `RestoreSession` is a Kubernetes `CustomResourceDefinition`(CRD) which specifies a target to restore and the source of data that will be restored in a Kubernetes native way.

You have to create a `RestoreSession` object whenever you want to restore. When a `RestoreSession` object is created, Stash injects an `init-container` into the target workload and restarts it. The `init-container` restores the desired data. If the target is a database or a stand-alone PVC, Stash launches a job to perform the restore process.

## RestoreSession CRD Specification

Like any official Kubernetes resource, a `RestoreSession` has `TypeMeta`, `ObjectMeta`, `Spec` and `Status` sections.

A sample `RestoreSession` object to restore backed up data of a StatefulSet is shown below:

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: statefulset-restore
  namespace: demo
spec:
  driver: Restic
  repository:
    name: local-repo
  # task:
  #   name: workload-restore # task field is not required for workload data restore but it is necessary for database restore.
  target:
    ref:
      apiVersion: apps/v1
      kind: StatefulSet
      name: recovered-statefulset
    volumeMounts:
    - mountPath: /source/data
      name: source-data
  rules:
  - targetHosts: ["host-3","host-4"] # "host-3" and "host-4" will have restored data of backed up host "host-1"
    sourceHost: "host-1" # source host
    paths:
    - /source/data
  - targetHosts: [] # empty host match all hosts
    sourceHost: "" # no source host indicates that the host is pod itself
    paths:
    - /source/data
  runtimeSettings:
    container:
      resources:
        limits:
          memory: 256M
        requests:
          memory: 256M
      securityContext:
        runAsGroup: 2000
        runAsUser: 2000
      ionice:
        class: 2
        classData: 4
      nice:
        adjustment: 5
    pod:
      imagePullSecrets:
      - name: my-private-registry-secret
      serviceAccountName: my-restore-svc
  tempDir:
    disableCache: false
    medium: Memory
    size: 2Gi
status:
  totalHosts: 5
  phase: Succeeded
  sessionDuration: 4.148288404s
  stats:
  - duration: 884.431745ms
    hostname: host-1
    phase: Succeeded
  - duration: 769.924342ms
    hostname: host-2
    phase: Succeeded
  - duration: 868.694738ms
    hostname: host-3
    phase: Succeeded
  - duration: 792.097784ms
    hostname: host-4
    phase: Succeeded
  - duration: 833.139795ms
    hostname: host-0
    phase: Succeeded
```

Here, we are going to describe the various sections of a `RestoreSession` object.

### RestoreSession `Spec`

A `RestoreSession` object has the following fields in the `spec` section.

#### spec.driver

`spec.driver` indicates the mechanism used to restore a target. Currently, Stash supports `Restic` and `VolumeSnapshotter` as drivers. The default value of this field is `Restic`.

| Driver              | Usage                                                                                                                                                                                                    |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Restic`            | Used to restore workload data, persistent volumes data and databases. It uses [restic](https://restic.net) to restore the target.                                                                                                                                                            |
| `VolumeSnapshotter` | Used to initialize PersistentVolumeClaim from VolumeSnapshot. It leverages Kubernetes [VolumeSnapshot](https://kubernetes.io/docs/concepts/storage/volume-snapshots/) crd and CSI driver to initialize the target PVCs from respective snapshots. Currently, it can restore only in new PVC. |

#### spec.target

`spec.target` field indicates the target where data will be restored. This section consists of the following fields:

- **spec.target.ref :** `spec.target.ref` refers to the restore target. You have to specify `apiVersion`, `kind` and `name` of the target. Stash will use this information to inject an `init-container` or to create a restore job.

- **spec.target.volumeMounts :** `spec.target.volumeMounts` specifies a list of volumes and their `mountPath` where the data will be restored. Stash will mount these volumes inside the `init-container` or restore job.

> Note: Stash stores absolute path of the backed up files. Hence, your restored volume must be mounted on the same `mountPath` as the original volume. Otherwise, the backed up files will not be restored into your desired volume.

- **spec.target.volumeClaimTemplates :** `spec.target.volumeClaimTemplates` specifies a list of PVC templates that will be created by restoring data from respective VolumeSnapshots. You have to set `spec.dataSource` section to the respective VolumeSnapshot. You can templatize `spec.dataSource.name` section. Stash will resolve the template and dynamically creates respective PVCs and initialize them from respective VolumeSnapshots. Use this field only if `spec.driver` is set to `VolumeSnapshotter`. For more details on how to restore PVCs from VolumeSnapshot, please visit [here](/docs/guides/latest/volume-snapshot/restore.md).

- **spec.target.replicas :** `spec.target.replicas` used to specify the number of replicas of a StatefulSet whose PVCs was snapshotted by `VolumeSnapshotter`. Stash uses this field to dynamically create the desired number of PVCs and initialize them from respective VolumeSnapshots. Use this field only if `spec.driver` is set to `VolumeSnapshotter` and `spec.target.volumeClaimTemplates` specified PVC template of a StatefulSet.

#### spec.repository

`spec.repository.name` indicates the `Repository` crd name that holds necessary backend information from where data will be restored.

#### spec.task

`spec.task` specifies the name and parameters of the [Task](/docs/concepts/crds/task.md) crd to use to restore the target data.

- **spec.task.name:** `spec.task.name` indicates the name of the `Task` template to use for this restore process.
- **spec.task.params:** `spec.task.params` is an array of custom parameters to use to configure the task.

> `spec.task` section is not necessary for restoring workload data (i.e. Deployment, DaemonSet, StatefulSet etc.). However, it is necessary for restoring database and stand-alone PVC.

#### spec.rules

`spec.rules` is an array of restore rules that specify how Stash should restore data for each host. For example, Stash runs restore process in all pod's of a StatefulSet. You can configure this `spec.rules` section to control what data will be restored into which pod.

Each restore rule has the following fields:

- **targetHosts :** `targetHosts` field contains a list of host names which are subject to this rule. If `targetHosts` field is empty, this rule applies to all hosts for which there is no specific rule. In the sample `RestoreSession` given above, the first rule applies to only `host-3` and `host-4` and the second rule is applicable to all hosts.
- **sourceHost :** `sourceHost` specifies the name of host whose backed up data will be restored by this rule. In the sample `RestoreSession`, the first rule specify that backed up data of `host-0` (i.e. `pod-0` of old StatefulSet) will be restored into `host-3` and `host-4` (i.e. `pod-3` and `pod-4` of new StatefulSet). If you keep `sourceHost` field empty as the second rule of the above example, data from a similar restore host will be restored on the respective restore host. That means, backed up data of `host-0` will be restored into `host-0`, backed up data of `host-1` will be restored into `host-1` and so on.
- **paths :** `paths` specifies a list of file paths that will be restored into the hosts who are subject to this rule.
- **snapshots :** `snapshots` specifies the list of snapshots that will be restored into the hosts who are subject to this rule. If you don't specify snapshot field, latest snasphot of the file paths specified in `paths` section will be restored.

Restore rules comply with the following conditions:

- There could be at most one rule with empty `targetHosts` field.
- No two rules with non-emtpy `targetHosts` can't be matched for a single host.
- Stash restore only one file path in a single snapshot. So, if you specify `snapshots` field in a rule, you can't specify `paths` field as it may cause restore failure if a file path wasn't backed up in the snapshot specified in the `snapshots` field.
- If no rule matches for a host, no data will be restored on that host.
- The order of the rules does not have any effect on the restore process.

#### spec.runtimeSettings

`spec.runtimeSettings` allows to configure runtime environment for restore `init-container` or job. You can specify runtime settings in both pod level and container level.

- **spec.runtimeSettings.container**

  `spec.runtimeSettings.container` is used to configure restore init-container/job in container level. You can configure the following container level parameters,

|       Field       |                                                                                                              Usage                                                                                                               |
| :---------------: | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `resources`       | Compute resources required by restore init-container or restore job. To know how to manage resources for containers, please visit [here](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/). |
| `livenessProbe`   | Periodic probe of restore init-container/job's container liveness. Container will be restarted if the probe fails.                                                                                                               |
| `readinessProbe`  | Periodic probe of restore init-container/job's container readiness. Container will be removed from service endpoints if the probe fails.                                                                                         |
| `lifecycle`       | Actions that the management system should take in response to container lifecycle events.                                                                                                                                        |
| `securityContext` | Security options that restore init-container/job's container should run with. For more details, please visit [here](https://kubernetes.io/docs/concepts/policy/security-context/).                                               |
| `nice`            | Set CPU scheduling priority for the restore process. For more details about `nice`, please visit [here](https://www.askapache.com/optimize/optimize-nice-ionice/#nice).                                                          |
| `ionice`          | Set I/O scheduling class and priority for the restore process. For more details about `ionice`, please visit [here](https://www.askapache.com/optimize/optimize-nice-ionice/#ionice).                                            |
| `env`             | A list of the environment variables to set in the restore init-container/job's container.                                                                                                                         |
| `envFrom`         | This allows to set environment variables to the the restore init-container/job's container from a Secret or ConfigMap.                                                                                               |

- **spec.runtimeSettings.pod**

  `spec.runtimeSettings.pod` is used to configure restore job in pod level. You can configure following pod level parameters,

|             Field              |                                                                                                                  Usage                                                                                                                   |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `serviceAccountName`           | Name of the `ServiceAccount` to use for restore job. Stash init-container will use the same `ServiceAccount` as the target.                                                                                                              |
| `nodeSelector`                 | Selector which must be true for restore job pod to fit on a node.                                                                                                                                                                        |
| `automountServiceAccountToken` | Indicates whether a service account token should be automatically mounted into the restore job's pod.                                                                                                                                    |
| `nodeName`                     | NodeName is used to request to schedule restore job's pod onto a specific node.                                                                                                                                                          |
| `securityContext`              | Security options that restore job's pod should run with. For more details, please visit [here](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/).                                                              |
| `imagePullSecrets`             | A list of secret names in the same namespace that will be used to pull image from private docker registry. For more details, please visit [here](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/). |
| `affinity`                     | Affinity and anti-affinity to schedule restore job's pod in the desired node. For more details, please visit [here](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity).                      |
| `schedulerName`                | Name of the scheduler that should dispatch the restore job.                                                                                                                                                                              |
| `tolerations`                  | Taints and Tolerations to ensure that restore job's pod is not scheduled in inappropriate nodes. For more details about `toleration`, please visit [here](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/).      |
| `priorityClassName`            | Indicates the restore job pod's priority class. For more details, please visit [here](https://kubernetes.io/docs/concepts/configuration/pod-priority-preemption/).                                                                       |
| `priority`                     | Indicates the restore job pod's priority value.                                                                                                                                                                                          |
| `readinessGates`               | Specifies additional conditions to be evaluated for Pod readiness. For more details, please visit [here](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-readiness-gate).                                          |
| `runtimeClassName`             | RuntimeClass is used for selecting the container runtime configuration. For more details, please visit [here](https://kubernetes.io/docs/concepts/containers/runtime-class/)                                                             |
| `enableServiceLinks`           | EnableServiceLinks indicates whether information about services should be injected into pod's environment variables.                                                                                                                     |

#### spec.tempDir

Stash mounts an `emptyDir` for holding temporary files. It is also used for `caching` for faster restore performance. You can configure the `emptyDir` using `spec.tempDir` section. You can also disable `caching` using this field. The following fields are configurable in `spec.tempDir` section:

- **spec.tempDir.medium :** Specifies the type of storage medium should back this file path.
- **spec.tempDir.sizeLimit :** Maximum limit of storage for this volume.
- **spec.tempDir.disableCaching :** Disable caching while restoring. This may negatively impact restore performance. This field is set to `false` by default.

### RestoreSession `Status`

`.status` section of `RestoreSession` shows progress, stats and overall phase of the restore process. The restore init-container or job adds its respective stats in `.status` section after it completes its task. `.status` section consists of the following fields:

#### status.totalHosts

Not every pods or replica of the target will run restore process. Thus, we refer those entities that runs restore process as a host. `status.totalHosts` specifies the total number of hosts that will run restore process for this RestoreSession. For more details on how many hosts will run restore process for which types of workload, please visit [here](#hosts-of-a-restore-process).

#### status.phase

`status.phase` indicates the overall phase of the restore process for this RestoreSession. `status.phase` will be `Succeeded` only if the phase of all hosts are `Succeeded`. If any of the hosts fail to complete restore, `status.phase` will be `Failed`.

#### status.sessionDuration

`status.sessionDuration` indicates the total time taken to complete the restoration of all hosts.

#### status.stats

`status.stats` section is an array of restore statistics of individual hosts. Each host adds their statistics in this array after completing their restore process. This field is only available for the `Restic` driver but not available for the `VolumeSnapshotter` driver. The default value of the driver is `Restic`.

Individual host stats entry consists of the following fields:

- **hostname :** `hostname` indicates the name of the host.
- **phase :** `phase` indicates the restore phase of this host.
- **duration :** `duration` indicates the total time taken to complete restore process for this host.
- **error :** `error` shows the reason for failure if the restore process fails for this host.

### Hosts of a restore process

Stash uses two different models for restoring backed up data depending on the target type. It uses **init-container model** for Kubernetes workloads and  **job model** for rest of the targets. In the init-container model, Stash injects an init-container inside the targeted workload and the init-container is responsible for restoring the desired data on workload restart. In the job model, Stash launches a job to restore the desired data.

Stash uses an identifier called **host** to identify the entity where the restore process will be run. This host identification process depends on the restore model and the target types. The restore strategy and host identification strategy for different types of  target is explained below.

**Kubernetes Workloads:**

Stash uses init-container model to restore Kubernetes workloads. However, not every init-container will run restore process. How many init-containers will run restore process depends on the type of the workload. We can divide them into the following categories:

- **Deployment, ReplicaSet and ReplicationController:** For these types of workload, all the replicas mounts the same volumes. So, restoring into only one replica is enough. In this case, Stash uses leader election to elect the leader pod. Only the init-container inside the leader pod runs restore process. This leader pod is identified as **host-0**. The total number of hosts for these types of workload is 1.
- **StatefulSet:** Every replica of a StatefulSet mounts different volumes. So, restoring into each replica is necessary. In this case, init-container inside each replica runs restore process. Stash identifies **pod-0** as **host-0**, **pod-1** as **host-1**, **pod-2** as **host-2** and so on. Hence, the total number of hosts for a StatefulSet is the number of replicas.
- **DaemonSet:** Daemon replicas on every node may contain different data. So, restoring into each daemon pod is necessary. In this case, init-container inside each daemon pod runs restore process. Stash considers the individual daemon pod as a separate host and the **node name** where the daemon pod is running act as their **host** identifier. The total number of hosts for a DaemonSet is the number of daemon pod running in the cluster.

**Stand-alone PVC:**

Stash uses job model to restore a stand-alone PVC. Stash launches a job to restore into the targeted PVC. This job is identified as **host-0**. In this case, the total number of host is 1.

**Databases:**

Stash uses job model to restore a database. Stash launches a job to restore into the targeted database. In this case, the number of hosts depends on database types.

- **Stand-alone database:** For stand-alone database, the restore target is identified as **host-0** and the total number of host is 1.
- **Replicated cluster:** For replicated clustered database such as MongoDB ReplicaSet, all the replicas contain the same data. In this case, Stash restores same data into each replica. Thus, the total number of host is 1 and it is identified as **host-0**.
- **Sharded cluster:** For sharded database cluster, Stash restores the backed up data of individual shard into the respective shard. Hence, the number of hosts for a sharded database is the number of shards and they are identified as **host-0**, **host-1**, **host-2**, etc. However, the number of hosts may increase based on the database type.

**VolumeSnapshot:**

Stash uses job model for restoring volume from VolumeSnapshot. Each volume is considered as different hosts and they are identified by their name. Hence, the number of total hosts is the number of targeted volumes to be restored.

**Restore using volumeClaimTemplates:**

If `volumeClaimTemplates` is specified in a RestoreSession, Stash creates the PVCs according to the template then it launches one job for each replica specified by `spec.target.replicas` field. In this case, the total number of hosts is number of replicas specified by `spec.target.replicas` field. If this field is not provided then the total number of hosts is 1.

## Next Steps

- Learn how restore of workloads data works from [here](/docs/guides/latest/workloads/overview.md).
- Learn how restore of databases works from [here](/docs/guides/latest/databases/overview.md).
- Learn how restore stand-alone PVC works from [here](/docs/guides/latest/volumes/overview.md).
