---
title: RestoreSession Overview
menu:
  docs_{{ .version }}:
    identifier: restoresession-overview
    name: RestoreSession
    parent: crds
    weight: 25
product_name: stash
menu_name: docs_{{ .version }}
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
    name: minio-repo
    namespace: demo
  # task:
  #   name: workload-restore # task field is not required for workload data backup but it is necessary for database backup.
  target:
    alias: my-sts
    ref:
      apiVersion: apps/v1
      kind: StatefulSet
      name: recovered-statefulset
    volumeMounts:
    - mountPath: /source/data
      name: source-data
    rules:
    - targetHosts: ["my-sts-3","my-sts-4"] # "pod-3" and "pod-4" will have restored data of backed up host "pod-1"
      sourceHost: "my-sts-1" # source host
      paths:
      - /source/data
      include:
      - /source/data/*.json
    - targetHosts: [] # empty host match all hosts
      sourceHost: "" # no source host indicates that the host is pod itself
      paths:
      - /source/data
      exclude:
      - /source/data/tmp.json
      - /source/data/*.txt
  hooks:
    preRestore:
      exec:
        command:
          - /bin/sh
          - -c
          - echo "Sample PreRestore hook demo"
      containerName: stash-init
    postRestore:
      executionPolicy: Always
      exec:
        command:
          - /bin/sh
          - -c
          - echo "Sample PostRestore hook demo"
      containerName: stash-init
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
      serviceAccountName: my-backup-sa
  tempDir:
    disableCaching: false
    medium: Memory
    sizeLimit: 2Gi
  timeOut: 30m
status:
  totalHosts: 5
  phase: Succeeded
  sessionDuration: 2m40.595857548s
  conditions:
  - lastTransitionTime: "2020-07-25T17:55:56Z"
    message: Repository demo/minio-repo exist.
    reason: RepositoryAvailable
    status: "True"
    type: RepositoryFound
  - lastTransitionTime: "2020-07-25T17:55:56Z"
    message: Backend Secret demo/minio-secret exist.
    reason: BackendSecretAvailable
    status: "True"
    type: BackendSecretFound
  - lastTransitionTime: "2020-07-25T17:55:56Z"
    message: Restore target apps/v1 statefulset/recovered-statefulset found.
    reason: TargetAvailable
    status: "True"
    type: RestoreTargetFound
  - lastTransitionTime: "2020-07-25T17:55:56Z"
    message: Successfully injected stash init-container.
    reason: InitContainerInjectionSucceeded
    status: "True"
    type: StashInitContainerInjected
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

| Driver              | Usage                                                                                                                                                                                                                                                                                            |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `Restic`            | Used to restore workload data, persistent volumes data and databases. It uses [restic](https://restic.net) to restore the target.                                                                                                                                                                |
| `VolumeSnapshotter` | Used to initialize PersistentVolumeClaim from VolumeSnapshot. It leverages Kubernetes [VolumeSnapshot](https://kubernetes.io/docs/concepts/storage/volume-snapshots/) crd and CSI driver to initialize the target PVCs from respective snapshots. Currently, it can restore only in the new PVC. |

#### spec.target

`spec.target` field indicates the target where data will be restored. This section consists of the following fields:

- **spec.target.alias :** `spec.target.alias` field specifies the alias that was used as the identifier of the backed up data. It must match with the alias used during backup.

- **spec.target.ref :** `spec.target.ref` refers to the restore target. You have to specify `apiVersion`, `kind`, and `name` of the target. Stash will use this information to inject an `init-container` or to create a restore job.

- **spec.target.volumeMounts :** `spec.target.volumeMounts` specifies a list of volumes and their `mountPath` where the data will be restored. Stash will mount these volumes inside the `init-container` or restore job.

> Note: Stash stores the absolute path of the backed up files. Hence, your restored volume must be mounted on the same `mountPath` as the original volume. Otherwise, the backed up files will not be restored into your desired volume.

- **spec.target.volumeClaimTemplates :** You can specify a list of PVC template using `spec.target.volumeClaimTemplates` field. Stash will create those PVCs then it will restore the desired data into them. Then, you can use those PVCs to deploy your desired workload.

- **spec.target.replicas :** If you want to restore the volumes of a StatefulSet through `spec.target.volumeClaimTemplate` field, you can specify the number of replicas of the StatefulSet using `spec.target.replicas`. In this case, you have to use `${POD_ORDINAL}` variable suffix in the claim name. Stash will replace that variable with respective ordinal and it will create the volumes for each replica. For more details, please visit [here](/docs/guides/use-cases/clone-pvc/index.md#clone-the-volumes-of-a-satefulset).

- **spec.target.rules :** `spec.target.rules` is an array of restore rules that specify how Stash should restore data for each host. For example, Stash runs the restore process in all pods of a StatefulSet. You can configure this `spec.target.rules` section to control what data will be restored into which pod. Each restore rule has the following fields:

  - **targetHosts :** `targetHosts` field contains a list of host names that are subject to this rule. If `targetHosts` field is empty, this rule applies to all hosts for which there is no specific rule. In the sample `RestoreSession` given above, the first rule applies to only `pod-3` and `pod-4` and the second rule is applicable to all hosts.
  - **sourceHost :** `sourceHost` specifies the name of host whose backed up data will be restored by this rule. In the sample `RestoreSession`, the first rule specifies that backed up data of `pod-0` will be restored into `pod-3` and `pod-4`. If you keep `sourceHost` field empty as the second rule of the above example, data from a similar backup host will be restored on the respective restore host. That means, backed up data of `pod-0` will be restored into `pod-0`, backed up data of `pod-1` will be restored into `pod-1` and so on.
  - **paths :** `paths` specifies a list of file paths that will be restored into the hosts who are subject to this rule.
  - **snapshots :** `snapshots` specifies the list of snapshots that will be restored into the hosts who are subject to this rule. If you don't specify the snapshot field, the latest snapshot of the file paths specified in the `paths` section will be restored.
  - **include :** `include` field specifies a list of patterns for the files that should be restored. Stash only restore the files that match these patterns.
  - **exclude :** `exclude` field specifies a list of patterns for the files that should be ignored during. Stash will not restore the files that match these patterns.

  Restore rules comply with the following conditions:

  - There could be at most one rule with an empty `targetHosts` field.
  - No two rules with non-empty `targetHosts` can't be matched for a single host.
  - Stash restore only one file path in a single snapshot. So, if you specify `snapshots` field in a rule, you can't specify `paths` field as it may cause restore failure if a file path wasn't backed up in the snapshot specified in the `snapshots` field.
  - If no rule matches for a host, no data will be restored on that host.
  - The order of the rules does not have any effect on the restoration process.

#### spec.repository

`spec.repository` specifies the name and namespace of the Repository CR that holds the necessary backend information where the backed up data has been stored.

- `spec.repository.name` specifies the name of the Repository CR.
- `spec.repository.namespace` specifies the namespace of the Repository. If you don't provide this field, Stash will look for the Repository CR in the same namespace as the RestoreSession.

#### spec.task

`spec.task` specifies the name and parameters of the [Task](/docs/concepts/crds/task/index.md) crd to use to restore the target data.

- **spec.task.name:** `spec.task.name` indicates the name of the `Task` template to use for this restore process.
- **spec.task.params:** `spec.task.params` is an array of custom parameters to use to configure the task.

> `spec.task` section is not necessary for restoring workload data (i.e. Deployment, DaemonSet, StatefulSet, etc.). However, it is necessary for restoring the database and stand-alone PVC.

#### spec.hooks

`spec.hooks` allows performing some actions before and after the restoration process. You can send HTTP requests to a remote server via `httpGet` or `httpPost` hooks. You can check whether a TCP socket is open using `tcpSocket` hook. You can also execute some commands into your application pod using `exec` hook.

- **spec.hooks.preRestore:** `spec.hooks.preRestore` hooks are executed before the restore process.
- **spec.hooks.postRestore:** `spec.hooks.postRestore` hooks are executed after the restore process. Unlike the `preRestore` hook, `postRestore` hook has an extra field named `executionPolicy` which let you execute hook based on the restore status. Currently, it support the following values:
  - `Always`: The hook will be executed after the restore process no matter the restore has failed or succeeded. This is the default behavior.
  - `OnSuccess`: The hook will be executed after the restore process only if the restore has succeeded.
  - `OnFailure`: The hook will be executed after the restore process only if the restore has failed.

For more details on how hooks work in Stash and how to configure different types of hook, please visit [here](/docs/guides/hooks/overview/index.md).

#### spec.runtimeSettings

`spec.runtimeSettings` allows to configure runtime environment for restore `init-container` or job. You can specify runtime settings in both the pod level and container level.

- **spec.runtimeSettings.container**

  `spec.runtimeSettings.container` is used to configure restore init-container/job in container level. You can configure the following container level parameters,

|       Field       | Usage                                                                                                                                                                                                                            |
| :---------------: | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|    `resources`    | Compute resources required by restore init-container or restore job. To know how to manage resources for containers, please visit [here](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/). |
|  `livenessProbe`  | Periodic probe of restore init-container/job's container liveness. The container will be restarted if the probe fails.                                                                                                           |
| `readinessProbe`  | Periodic probe of restore init-container/job's container readiness. The container will be removed from service endpoints if the probe fails.                                                                                     |
|    `lifecycle`    | Actions that the management system should take in response to container lifecycle events.                                                                                                                                        |
| `securityContext` | Security options that restore init-container/job's container should run with. For more details, please visit [here](https://kubernetes.io/docs/concepts/policy/security-context/).                                               |
|      `nice`       | Set CPU scheduling priority for the restore process. For more details about `nice`, please visit [here](https://www.askapache.com/optimize/optimize-nice-ionice/#nice).                                                          |
|     `ionice`      | Set I/O scheduling class and priority for the restore process. For more details about `ionice`, please visit [here](https://www.askapache.com/optimize/optimize-nice-ionice/#ionice).                                            |
|       `env`       | A list of the environment variables to set in the restore init-container/job's container.                                                                                                                                        |
|     `envFrom`     | This allows to set environment variables to the restore init-container/job's container from a Secret or ConfigMap.                                                                                                               |

- **spec.runtimeSettings.pod**

  `spec.runtimeSettings.pod` is used to configure restore job in pod level. You can configure following pod level parameters,

| Field                          | Usage                                                                                                                                                                                                                                         |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `serviceAccountName`           | Name of the `ServiceAccount` to use for restore job. Stash init-container will use the same `ServiceAccount` as the target.                                                                                                                   |
| `nodeSelector`                 | Selector which must be true for restore job pod to fit on a node.                                                                                                                                                                             |
| `automountServiceAccountToken` | Indicates whether a service account token should be automatically mounted into the restore job's pod.                                                                                                                                         |
| `nodeName`                     | NodeName is used to request to schedule restore job's pod onto a specific node.                                                                                                                                                               |
| `securityContext`              | Security options that restore job's pod should run with. For more details, please visit [here](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/).                                                                   |
| `imagePullSecrets`             | A list of secret names in the same namespace that will be used to pull images from the private docker registry. For more details, please visit [here](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/). |
| `affinity`                     | Affinity and anti-affinity to schedule restore job's pod in the desired node. For more details, please visit [here](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity).                           |
| `schedulerName`                | Name of the scheduler that should dispatch the restore job.                                                                                                                                                                                   |
| `tolerations`                  | Taints and Tolerations to ensure that restore job's pod is not scheduled in inappropriate nodes. For more details about `toleration`, please visit [here](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/).           |
| `priorityClassName`            | Indicates the restore job pod's priority class. For more details, please visit [here](https://kubernetes.io/docs/concepts/configuration/pod-priority-preemption/).                                                                            |
| `priority`                     | Indicates the restore job pod's priority value.                                                                                                                                                                                               |
| `readinessGates`               | Specifies additional conditions to be evaluated for Pod readiness. For more details, please visit [here](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-readiness-gate).                                               |
| `runtimeClassName`             | RuntimeClass is used for selecting the container runtime configuration. For more details, please visit [here](https://kubernetes.io/docs/concepts/containers/runtime-class/)                                                                  |
| `enableServiceLinks`           | EnableServiceLinks indicates whether information about services should be injected into pod's environment variables.                                                                                                                          |

#### spec.tempDir

Stash mounts an `emptyDir` for holding temporary files. It is also used for `caching` for faster restore performance. You can configure the `emptyDir` using `spec.tempDir` section. You can also disable `caching` using this field. The following fields are configurable in `spec.tempDir` section:

- **spec.tempDir.medium :** Specifies the type of storage medium should back this file path.
- **spec.tempDir.sizeLimit :** Maximum limit of storage for this volume.
- **spec.tempDir.disableCaching :** Disable caching while restoring. This may negatively impact restore performance. This field is set to `false` by default.

#### spec.timeOut

`spec.timeOut` specifies the maximum amount of time to wait for the restore to complete. If the restore doesn't complete within this time limit, Stash will mark the respective RestoreSession as `Failed`. You can specify the timeout in the following format:

- Seconds `30s`
- Minutes `10m`
- Hours  `1h`
- Combination of seconds, minutes hour 10m30s, `1h30m` etc.

Stash does not support providing days (`d`) in the `timeOut` field. Use the equivalent hours instead.

#### spec.interimVolumeTemplate

For some targets (i.e. some databases), Stash can't directly pipe the restored data into the target. In this case, it has to store the restored data temporarily before injecting into the target. `spec.interimVolumeTemplate` specifies a PVC template for holding those data temporarily. Stash will create a PVC according to the template and use it to store the data temporarily. This PVC will be deleted automatically if you delete the `RestoreSession`.

>Note that the usage of this field is different from `spec.tempDir` which is used for caching purpose. Stash has introduced this field because the `emptyDir` volume that is used for `spec.tempDir` does not play nice with large databases( i.e. 100Gi database). Also, it provides debugging capability as Stash keeps it until you delete the `RestoreSession`.

### RestoreSession `Status`

`.status` section of `RestoreSession` shows progress, stats, and overall phase of the restore process. The restore init-container or job adds its respective stats in `.status` section after it completes its task. `.status` section consists of the following fields:

#### status.totalHosts

Not every pod or replica of the target will run the restore process. Thus, we refer those entities that run the restore process as a host. `status.totalHosts` specifies the total number of hosts that will run the restore process for this RestoreSession. For more details on how many hosts will run restore process for which types of workload, please visit [here](#hosts-of-a-restore-process).

#### status.phase

`status.phase` indicates the overall phase of the restore process for this RestoreSession. `status.phase` will be `Succeeded` only if the phase of all hosts are `Succeeded`. If any of the hosts fail to complete restore, `status.phase` will be `Failed`.

#### status.sessionDuration

`status.sessionDuration` indicates the total time taken to complete the restoration of all hosts.

#### status.sessionDeadline

`status.sessionDeadline` indicates the the deadline of the restore process. `RestoreSession` will be considered `Failed` if the restore does not complete within this deadline.


#### status.conditions

`status.conditions` shows the conditions of various steps of the restore process. Stash sets the following conditions for a RestoreSession:

| Condition Type               | Usage                                                                                                                                          |
| ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `RepositoryFound`            | Indicates whether the respective Repository object was found or not.                                                                           |
| `BackendSecretFound`         | Indicates whether the respective backend secret was found or not.                                                                              |
| `RestoreTargetFound`         | Indicates whether the restore target was found or not.                                                                                         |
| `StashInitContainerInjected` | Indicates whether stash init-container was injected into the targeted workload or not. This condition is applicable only in the sidecar model. |
| `RestorerEnsured`          | Indicates whether the Restorer job/init-container was created or not.                                      |
| `ValidationPassed`   | Indicates whether the resource has passed validation checks or not. |
| ` MetricsPushed`     | Indicates whether the metrics have been pushed to the Pushgateway or not. |
| `DeadlineExceeded`   | Indicates whether the session deadline was exceeded or not.|

#### status.stats

`status.stats` section is an array of restore statistics of individual hosts. Each host adds their statistics in this array after completing their restore process. This field is only available for the `Restic` driver but not available for the `VolumeSnapshotter` driver. The default value of the driver is `Restic`.

Individual host stats entry consists of the following fields:

- **hostname :** `hostname` indicates the name of the host. Usually, it is the `alias` or `alias-<workload-specific-suffix>`. For more details, please visit [here](/docs/concepts/crds/backupsession/index.md#hosts-of-a-backup-process).
- **phase :** `phase` indicates the restore phase of this host.
- **duration :** `duration` indicates the total time taken to complete the restore process for this host.
- **error :** `error` shows the reason for failure if the restore process fails for this host.

## Next Steps

- Learn how restore of workloads data works from [here](/docs/guides/workloads/overview/index.md).
- Learn how restore of databases works from [here](/docs/guides/addons/overview/index.md).
- Learn how restore stand-alone PVC works from [here](/docs/guides/volumes/overview/index.md).
