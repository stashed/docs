---
title: Changelog | Stash
description: Changelog
menu:
  product_stash_{{ .version }}:
    identifier: changelog-stash-0.9
    name: Changelog-0.9
    parent: welcome
    weight: 90
product_name: stash
menu_name: product_stash_{{ .version }}
section_menu_id: welcome
url: /products/stash/{{ .version }}/welcome/changelog-0.9/
aliases:
  - /products/stash/{{ .version }}/CHANGELOG-0.9/
---
# Change Log

## [v0.9.0-rc.2](https://github.com/stashed/stash/tree/v0.9.0-rc.2)

[Full Changelog](https://github.com/stashed/stash/compare/v0.9.0-rc.1...v0.9.0-rc.2)

Stash `v0.9.0-rc.2` comes with some bug fixes and improvements since `v0.9.0-rc.1` release. Please, try it and give us your valuable feedback. To install, please follow the guide [here](https://appscode.com/products/stash/v0.9.0-rc.2/setup/install/).

## Significant Changes

- **Retention Policy for VolumeSnapshot:** We have implemented retention policy for VolumeSnapshot. Now, your old VolumeSnapshots will automatically cleaned according to retention policy. Ref: [stashed/stash#926](https://github.com/stashed/stash/pull/926).

- **Fix Argument Passing:** We have fixed argument passing to database backup/restore commands. Ref: [stashed/mongodb#5](https://github.com/stashed/mongodb/pull/5), [stashed/postgres#17](https://github.com/stashed/postgres/pull/17), [stashed/elasticsearch#8](https://github.com/stashed/elasticsearch/pull/8), [stashed/mysql#9](https://github.com/stashed/mysql/pull/9).

## Issue Fixed

- Implement retention policy for VolumeSnapshot [#808](https://github.com/stashed/stash/issues/808)
- Support elasticsearch backups [#894](https://github.com/stashed/stash/issues/)
- Chart "stash-postgres" not found in appscode index [#898](https://github.com/stashed/stash/issues/894)
- Support mysql backups [#892](https://github.com/stashed/stash/issues/892)
- Support mongodb backups [#893](https://github.com/stashed/stash/issues/893)
- RestoreSession stuck running [#895](https://github.com/stashed/stash/issues/895)
- Stash restore job fails [#914](https://github.com/stashed/stash/issues/914)

## Pull Request Merged

**[stashed/stash](https://github.com/stashed/stash):**

- Add release checklist issue template [#934](https://github.com/stashed/stash/pull/934)
- Volume e2e test github action [#933](https://github.com/stashed/stash/pull/933)
- Fix BackupSession additional print column [#932](https://github.com/stashed/stash/pull/932)
- Remove spec.backupConfiguration from BackupSession [#931](https://github.com/stashed/stash/pull/931)
- Update e2e tests to run on GitHub actions [#917](https://github.com/stashed/stash/pull/917)
- Enable make ci [#928](https://github.com/stashed/stash/pull/928)
- Don't restart workload on backup pause+Use ImagePullPolicy: "Always" for Function-Task model [#929](https://github.com/stashed/stash/pull/929)
- Add status.observedGeneration [#927](https://github.com/stashed/stash/pull/927)
- Implement RetentionPolicy for VolumeSnapshot [#926](https://github.com/stashed/stash/pull/926)
- Verify generated files are up to date [#925](https://github.com/stashed/stash/pull/925)
- Update and rename go.yml to ci.yml [#922](https://github.com/stashed/stash/pull/922)
- Add probe and BackupBatch api [#890](https://github.com/stashed/stash/pull/890)
- Use core package from k8s.io/api repo [#921](https://github.com/stashed/stash/pull/921)

**[stashed/docs](https://github.com/stashed/docs):**

- Use `spec.invoker` instead of `spec.backupConfiguration` in BackupSession [stashed/docs#44](https://github.com/stashed/docs/pull/44)

**[stashed/installer](https://github.com/stashed/installer):**

- Prepare for release v0.9.0-rc.2  [stashed/installer#16](https://github.com/stashed/installer/pull/16)
- Remove --enable-status-subresource flag [stashed/installer#15](https://github.com/stashed/installer/pull/15)
- Add delete permission to VS ClusterRole [stashed/installer#13](https://github.com/stashed/installer/pull/13)
- Ignore error while delete resources from uninstaller script [stashed/installer#14](https://github.com/stashed/installer/pull/14)

**[stashed/catalog](https://github.com/stashed/catalog):**

- Fix catalog installer (when helm is not installed) [stashed/catalog#10](https://github.com/stashed/catalog/pull/10)

**[stashed/postgres](https://github.com/stashed/postgres):**

- Fix BackupSession's Print Columns [stashed/postgres#20](https://github.com/stashed/postgres/pull/20)
- Enable make ci [stashed/postgres#19](https://github.com/stashed/postgres/pull/19)
- Remove `--enable-status-subresource` flag  [stashed/postgres#18](https://github.com/stashed/postgres/pull/18)
- Fix arguments passing [stashed/postgres#17](https://github.com/stashed/postgres/pull/17)

**[stashed/mongodb](https://github.com/stashed/mongodb):**

- Fix BackupSession's Print Columns [stashed/mongodb#9](https://github.com/stashed/mongodb/pull/9)
- Enable make ci [stashed/mongodb#8](https://github.com/stashed/mongodb/pull/8)
- Remove `--enable-status-subresource` flag [stashed/mongodb#7](https://github.com/stashed/mongodb/pull/7)
- Update totalHosts from backup/restore process [stashed/mongodb#6](https://github.com/stashed/mongodb/pull/6)
- Fix argument passing [stashed/mongodb#5](https://github.com/stashed/mongodb/pull/5)

**[stashed/mysql](https://github.com/stashed/mysql):**

- Fix BackupSession's Print Columns [stashed/mysql#12](https://github.com/stashed/mysql/pull/12)
- Enable make ci [stashed/mysql#11](https://github.com/stashed/mysql/pull/11)
- Remove `--enable-status-subresource` flag  [stashed/mysql#10](https://github.com/stashed/mysql/pull/10)
- Fix argument passing [stashed/mysql#9](https://github.com/stashed/mysql/pull/9)

**[stashed/elasticsearch](https://github.com/stashed/elasticsearch):**

- Fix BackupSession's Print Columns [stashed/elasticsearch#11](https://github.com/stashed/elasticsearch/pull/11)
- Enable make ci [stashed/elasticsearch#10](https://github.com/stashed/elasticsearch/pull/10)
- Remove --enable-status-subresource flag [stashed/elasticsearch#9](https://github.com/stashed/elasticsearch/pull/9)
- Fix argument passing [stashed/elasticsearch#8](https://github.com/stashed/elasticsearch/pull/8)

## [v0.9.0-rc.1](https://github.com/stashed/stash/tree/v0.9.0-rc.1)

[Full Changelog](https://github.com/stashed/stash/compare/v0.9.0-rc.0...v0.9.0-rc.1)

Stash `v0.9.0-rc.1` comes with documentation and respective charts for addons to backup databases. It also comes with some bug fixes since `v0.9.0-rc.0` release. We have also introduced a `kubectl` plugin for Stash in this release. Please, try it and give us your valuable feedback.  To install, please follow the guide [here](https://appscode.com/products/stash/v0.9.0-rc.1/setup/install/).

## Significant Changes

- **Addon Support:** Stash addons enable users to extend it's functionality to backup/restore any type of custom workloads. This release includes respective documentations and charts for `PostgreSQL`, `MySQL`, `MongoDB`, and `Elasticsearch` databases. This enables Stash to backup various [KubeDB](https://kubedb.com/) supported databases. You can also write your own addon to backup your custom target. We will include a guide on how to build addon for Stash in a future release.

- **Stash `kubectl` Plugin (Alpha):** We have introduced an experimental Stash `kubectl` plugin. This will help you to unlock repository, trigger instant backup and copy various Stash resources between namespace and more.

- **Introduce BackupHistoryLimit:** Now, you can configure how many `BackupSession` and associate resources (Job, PVC etc) to keep for each backup target. By default, Stash will keep only the history of last backup for debugging purpose. You can configure it using `spec.backupHistoryLimit` field of `BackupConfiguration`.

- **Introduce InterimVolumeTemplate:** We have introduced a new field called `InterimVolumeTemplate` that can be used to pass a PVC template for storing temporary backup/restore data before uploading to backup or injecting into target. For more details, please visit [here](https://appscode.com/products/stash/v0.9.0-rc.1/concepts/crds/backupconfiguration/).

- **Allow `subPath` in Auto-Backup Annotation:** Now, you can provide `subPath` in auto-backup annotation. For more details, please check [here](https://appscode.com/products/stash/v0.9.0-rc.1/guides/latest/auto-backup/workload/).

- **Use Kubebuilder to Generate CRDs:** We have moved to [kubernetes-sigs/kubebuilder](https://github.com/kubernetes-sigs/kubebuilder) for generating CRDs for Stash.

## Issues Fixed

- subPath stash autobackup annotation [#868](https://github.com/stashed/stash/issues/868)
- GKE Alpha cluster need to run as "fsGroup: 65535" [#873](https://github.com/stashed/stash/issues/873)
- Can't evaluate field version in AppBinding [#876](https://github.com/stashed/stash/issues/876)
- Neither --kubeconfig nor --master was specified [#878](https://github.com/stashed/stash/issues/878)
- Prometheus metrics missing source of the backup [#869](https://github.com/stashed/stash/issues/869)
- Waiting until stash apiservice is available on GKE with private nodes [#875](https://github.com/stashed/stash/issues/875)
- nice` and `ionice` is missing in distroless image [#882](https://github.com/stashed/stash/issues/882)
- BackupBlueprint not creating Repository or BackupConfiguration [#877](https://github.com/stashed/stash/issues/877)
- mysql stash backup error: unexpected end of JSON input [#880](https://github.com/stashed/stash/issues/880)
- How do I override the folder name stash recovers from [#861](https://github.com/stashed/stash/issues/861)
- Cannot pull image stashed/stash-mysql:8.0.14 [#879](https://github.com/stashed/stash/issues/879)
- Nil pointer check is missing in metric handling [#881](https://github.com/stashed/stash/issues/881)
- [0.9.0-rc.0] Cannot list snapshots [#901](https://github.com/stashed/stash/issues/901)
- Support interim data volume in BackupConfiguration [#906](https://github.com/stashed/stash/issues/906)
- Stash not backing up all files [#899](https://github.com/stashed/stash/issues/899)
- [feature] clean up objects [#867](https://github.com/stashed/stash/issues/867)
- Stash addon doc organization [#856](https://github.com/stashed/stash/issues/856)
- Generate swagger.json for v1beta1 [#896](https://github.com/stashed/stash/issues/896)
- Stash restore job fails [#914](https://github.com/stashed/stash/issues/914)
- RestoreSession stuck running [#895](https://github.com/stashed/stash/issues/895)
- Support mongodb backups [#893](https://github.com/stashed/stash/issues/893)
- Support mysql backups [#892](https://github.com/stashed/stash/issues/892)

## Pull Request Merged

- Bring back dependency to github.com/golang/protobuf@v1.2.0 [#918](https://github.com/stashed/stash/pull/918)
- Run e2e tests using GitHub actions [#909](https://github.com/stashed/stash/pull/909)
- Generate swagger.json for v1beta1 [#916](https://github.com/stashed/stash/pull/916)
- Add release pipeline [#915](https://github.com/stashed/stash/pull/915)
- Fix Restore PVC using Task Function method [#897](https://github.com/stashed/stash/pull/897)
- Fix VolumeSnapshot Issues [#912](https://github.com/stashed/stash/pull/912)
- Add Kubebuilder annotations [#911](https://github.com/stashed/stash/pull/911)
- Use kubebuilder to generate crd yamls [#843](https://github.com/stashed/stash/pull/843)
- Introduce InterimVolumeTemplate and BackupHistoryLimit field. [#907](https://github.com/stashed/stash/pull/907)
- Make RetentionPolicy required in v1beta1 api [#905](https://github.com/stashed/stash/pull/905)
- Enable GitHub actions [#903](https://github.com/stashed/stash/pull/903)
- Fix snapshot listing for local backend [#902](https://github.com/stashed/stash/pull/902)
- Improve Error Handling + Code refactor [#891](https://github.com/stashed/stash/pull/891)
- Add `NICE` and `IONICE` into distroless image [#883](https://github.com/stashed/stash/pull/883)
- Improve metrics handling + make update status uniform [#872](https://github.com/stashed/stash/pull/872)
- Use default fsGroup: 65535 [#874](https://github.com/stashed/stash/pull/874)
- Add DumpEnv func to Restic Config file [#870](https://github.com/stashed/stash/pull/870)
- Allow to specify subPath in auto-backup annotation + use default mounts and target paths for PVC [#871](https://github.com/stashed/stash/pull/871)

## [v0.9.0-rc.0](https://github.com/stashed/stash/tree/v0.9.0-rc.0)

[Full Changelog](https://github.com/stashed/stash/compare/0.8.3...v0.9.0-rc.0)

We are very excited to announce Stash `v0.9.0-rc.0`. This release introduces `v1beta1` API and a design overhaul. The new API and design enable Stash to support the use cases that were not possible before. This makes Stash more powerful, transparent, extensible and customizable. We are expecting that this new API will graduate to GA after some maturity. Check out the new architecture from [here](https://github.com/stashed/docs/blob/v0.9.0-rc.0/docs/concepts/what-is-stash/architecture.md).

### What's New

This release introduces lots of new features and changes. A summary of these new features is given below:

#### New Custom Resources

The following custom resources have been introduced in this release:

- [BackupConfiguration](https://github.com/stashed/docs/blob/v0.9.0-rc.0/docs/concepts/crds/backupconfiguration.md).
- [BackupSession](https://github.com/stashed/docs/blob/v0.9.0-rc.0/docs/concepts/crds/backupsession.md).
- [RestoreSession](https://github.com/stashed/docs/blob/v0.9.0-rc.0/docs/concepts/crds/restoresession.md).
- [Function](https://github.com/stashed/docs/blob/v0.9.0-rc.0/docs/concepts/crds/function.md).
- [Task](https://github.com/stashed/docs/blob/v0.9.0-rc.0/docs/concepts/crds/task.md).
- [BackupBlueprint](https://github.com/stashed/docs/blob/v0.9.0-rc.0/docs/concepts/crds/backupblueprint.md).
- [AppBinding](https://github.com/stashed/docs/blob/v0.9.0-rc.0/docs/concepts/crds/appbinding.md).

#### New Features

In addition to improving existing features, this release introduces the following new features:

- **Backup & Restore Stand-alone PVC :**
  Stash now supports taking backup of a stand-alone PVC. To learn more about how Stash takes backup of a stand-alone PVC, please visit [here](https://github.com/stashed/docs/blob/v0.9.0-rc.0/docs/guides/latest/volumes/overview.md).

- **Backup & Restore Databases :**
  Stash now can backup PostgreSQL, MongoDB, Elasticsearch and MySQL databases in both stand-alone and clustered mode. To learn more about how Stash takes backup of a database, please visit [here](https://github.com/stashed/docs/blob/v0.9.0-rc.0/docs/guides/latest/databases/overview.md).

- **VolumeSnapshot :**
  Now, you can take a scheduled snapshot of the volumes of a workload using Kubernetes [VolumeSnapshot](https://kubernetes.io/docs/concepts/storage/volume-snapshots/) API. Check out how volume snapshotting works in Stash from [here](https://github.com/stashed/docs/blob/v0.9.0-rc.0/docs/guides/latest/volumesnapshot/overview.md).

- **Instant Backup :**
  You can now trigger a backup instantly. To learn how, please visit [here](https://github.com/stashed/docs/blob/v0.9.0-rc.0/docs/guides/latest/volumesnapshot/overview.md).

- **Auto Backup :**
  Now, Stash will let you configure a common template to backup similar types of target. You will require to add just a few annotations to the targeted workload to enable backup for it. Want to know how? Please visit [here](https://github.com/stashed/docs/blob/v0.9.0-rc.0/docs/guides/latest/auto-backup/overview.md).

- **Support PSP Enabled Cluster :**
  Stash now supports PSP enabled cluster.

- **Improved Prometheus Metrics :**
  We have improved Prometheus metrics in this release. Check out the new metrics from [here](https://github.com/stashed/docs/blob/v0.9.0-rc.0/docs/guides/latest/monitoring/overview.md).

- **Support REST Server as Backend :**
  Stash now supports REST server as backend. To learn how to configure REST backend, please visit [here](https://github.com/stashed/docs/blob/v0.9.0-rc.0/docs/guides/latest/backends/rest.md).

- **KubeDB Integration :**
  Stash now seemingly integrates with [KubeDB](https://kubedb.com/). It is now recommended tool to backup & restore KubeDB supported databases.

For a complete feature list of this release, please visit [here](https://github.com/stashed/docs/blob/v0.9.0-rc.0/docs/concepts/what-is-stash/overview.md).

### Significant Changes

- Stash now has been moved from [appscode](https://github.com/appscode) Github organization to its own organization. New home for Stash is [stashed](https://github.com/stashed) organization.
- We have split the original `appscode/stash` repository into multiple repositories ([stashed/stash](https://github.com/stashed/stash), [stashed/installer](https://github.com/stashed/installer), [stashed/docs](https://github.com/stashed/docs)). This enables us to push emergency fixes to installer and docs without requiring to cut a new release.
- Now Stash uses a more industry-standard `Makefile` based build process. This makes building Stash from source code simple and easy ([\#800](https://github.com/stashed/stash/pull/800)).
- We have added ARM architecture support in this release. Thanks to  @carlosedp ([\#802](https://github.com/stashed/stash/pull/802)).
- We now use [Docker manifest](https://docs.docker.com/engine/reference/commandline/manifest/) to build Stash docker images. Hence, Stash docker images are now platform aware. You don't have to worry about your platform architecture. Docker will take care of it ([\#802](https://github.com/stashed/stash/pull/802)).
- We have updated [restic](https://restic.net/) version from `0.8.3` to `0.9.5` ([\#789](https://github.com/stashed/stash/pull/789)).
- We have upgraded Kubernetes client to `v1.14.0` ([\#775](https://github.com/stashed/stash/pull/775)).
- Now Stash uses `failurePolicy: Ignore` in webhooks for Kubernetes official resources. So, Stash will no longer cause any problem for creating new Kubernetes resources when it is not ready ([\#726](https://github.com/stashed/stash/pull/726)).
- As RBAC is now default in most of the Kubenetes cluster and creating RBAC resources in an RBAC disabled cluster does not cause any problem, we have removed `--rbac` flag. Now, Stash will always start in RBAC enabled mode ([\#761](https://github.com/stashed/stash/pull/761)).
- We have moved to [go mod](https://github.com/golang/go/wiki/Modules) from [glide](https://github.com/Masterminds/glide) for dependency management ([\#775](https://github.com/stashed/stash/pull/775)).
- We have changed Stash package path to `stash.appscode.dev/stash` ([\#776](https://github.com/stashed/stash/pull/776)).

### Upgrading from 0.8.3

If you are upgrading Stash from `0.8.3` to this version, pay attention to the following things:

**What will work:**
- Exiting scheduled backup will continue to work.
- Scheduling new backup using Restic crd will work.
- Restoring the already backed up data using Recovery crd will work.

**What will not work:**
- Restoring the data that was backed up using old API (`Restic`) with the new API (`RestoreSession`) will not work.
- Restoring the data that was backed up using new API (`BackupConfiguration`) with the old API (`Recovery`) will not work.
- Using new API (`BackupConfiguration` ) to backup into already existing Repository will not work. Stash will upload all targeted data again into the backend. Old snapshots will not be usable any more.
- Old Grafana dashboard will not work with new metrics.

### Issues Fixed

- Sidecar RoleBinding is not being created when Mutating Webhook is enabled [\#395](https://github.com/stashed/stash/issues/395)
- Cannot deploy stash with helm@3 [\#822](https://github.com/stashed/stash/issues/822)
- Restore PVCs from templates using Restic [\#784](https://github.com/stashed/stash/issues/784)
- Handle restored files permission properly [\#733](https://github.com/stashed/stash/issues/733)
- Use restic 0.9.5 [\#781](https://github.com/stashed/stash/issues/781)
- Support PSP or SCC (openshift) enabled clusters [\#462](https://github.com/stashed/stash/issues/462)
- Configure environment variables (or proxy settings) on restic and recovery CRDs [#621](https://github.com/stashed/stash/issues/621)
- Proposal: Move cluster backup part from Kubed to Stash [\#601](https://github.com/stashed/stash/issues/601)
- error: unable to retrieve the complete list of server APIs: admission.stash.appscode.com/v1alpha1: the server is currently unable to handle the request, repositories.stash.appscode.com/v1alpha1: the server is currently unable to handle the request [\#785](https://github.com/stashed/stash/issues/785)
- Internal error occurred: failed calling webhook "deployment.admission.stash.appscode.com": the server is currently unable to handle the request [\#692](https://github.com/stashed/stash/issues/692)
- MutatingWebhooks must be without side-effect [\#758](https://github.com/stashed/stash/issues/758)
- Targeted Workload stuck in terminating state after deleted a Restic CRD [\#672](https://github.com/stashed/stash/issues/672)
- OSM config and file permission issue [\#766](https://github.com/stashed/stash/issues/766)
- Remove `--rbac` flag [\#705](https://github.com/stashed/stash/issues/705)
- Use FailurePolicy ignore for K8s resource webhooks? [\#709](https://github.com/stashed/stash/issues/709)
- FR: Add possibility to change settings of Restic [\#545](https://github.com/stashed/stash/issues/545)
- Allow setting `nice` and `ionice` for backup command [\#366](https://github.com/stashed/stash/issues/366)
- Cleanup old cache after backup [\#703](https://github.com/stashed/stash/issues/703)
- Support Restic Rest server as backend. [\#126](https://github.com/stashed/stash/issues/126)

### Pull Request Summary

- Remove the `bs` short name for BackupSession [\#859](https://github.com/stashed/stash/pull/859) ([tamalsaha](https://github.com/tamalsaha))
- Use github.com/golang/protobuf@v1.2.0 [\#855](https://github.com/stashed/stash/pull/855) ([tamalsaha](https://github.com/tamalsaha))
- Fix resolving Task when volumeClaimTemplate is specified in RestoreSession [\#852](https://github.com/stashed/stash/pull/852) ([hossainemruz](https://github.com/hossainemruz))
- Use POD_ORDINAL env var to restore using PVC template [\#849](https://github.com/stashed/stash/pull/849) ([suaas21](https://github.com/suaas21))
- Pass replicas from RestoreSession to Function [\#848](https://github.com/stashed/stash/pull/848) ([hossainemruz](https://github.com/hossainemruz))
- Rename BackupConfigurationTemplate to BackupBlueprint [\#847](https://github.com/stashed/stash/pull/847) ([hossainemruz](https://github.com/hossainemruz))
- Use variable for version in BackupConfigurationTemplate name [\#846](https://github.com/stashed/stash/pull/846) ([hossainemruz](https://github.com/hossainemruz))
- New variable from type field of AppBinding + Fix RoleBinding name conflict with KubeDB [\#845](https://github.com/stashed/stash/pull/845) ([hossainemruz](https://github.com/hossainemruz))
- Fix Platforms Issue [\#844](https://github.com/stashed/stash/pull/844) ([suaas21](https://github.com/suaas21))
- Add support to restore using volumeClaimTemplate in Function-Task model [\#841](https://github.com/stashed/stash/pull/841) ([hossainemruz](https://github.com/hossainemruz))
- Add GetSnapshotSize() function [\#839](https://github.com/stashed/stash/pull/839) ([hossainemruz](https://github.com/hossainemruz))
- Fix travis build [\#837](https://github.com/stashed/stash/pull/837) ([tamalsaha](https://github.com/tamalsaha))
- Update azure-sdk-for-go dependencies [\#836](https://github.com/stashed/stash/pull/836) ([tamalsaha](https://github.com/tamalsaha))
- Fix RestoreSession replicas logic [\#835](https://github.com/stashed/stash/pull/835) ([suaas21](https://github.com/suaas21))
- Use robfig/cron@v3 [\#834](https://github.com/stashed/stash/pull/834) ([tamalsaha](https://github.com/tamalsaha))
- Add support for parallel backup & restore [\#833](https://github.com/stashed/stash/pull/833) ([hossainemruz](https://github.com/hossainemruz))
- Fix restore Job parallel execution [\#832](https://github.com/stashed/stash/pull/832) ([suaas21](https://github.com/suaas21))
- Remove unused code [\#829](https://github.com/stashed/stash/pull/829) ([tamalsaha](https://github.com/tamalsaha))
- Generate docs files inside docs repo [\#828](https://github.com/stashed/stash/pull/828) ([tamalsaha](https://github.com/tamalsaha))
- Add License notice to makefile [\#825](https://github.com/stashed/stash/pull/825) ([tamalsaha]https://github.com/tamalsaha())
-  Create default Functions and Tasks from operator [\#824](https://github.com/stashed/stash/pull/824) ([hossainemruz](https://github.com/hossainemruz))
- Fix default securityContext passing to restore init-container/job [\#823](https://github.com/stashed/stash/pull/823) ([hossainemruz](https://github.com/hossainemruz))
- Fix restore job RBAC [\#821](https://github.com/stashed/stash/pull/821) ([hossainemruz](https://github.com/hossainemruz))
- Fixed volumeSnapshot Error Issue [\#819](https://github.com/stashed/stash/pull/819) ([suaas21](https://github.com/suaas21))
- Always attempt to pull a newer image for `make container` [\#818](https://github.com/stashed/stash/pull/818) ([tamalsaha](https://github.com/tamalsaha))
- Make --image-tag a required flag. [\#817](https://github.com/stashed/stash/pull/817 ) ([tamalsaha](https://github.com/tamalsaha))
- Add PusgatewayURL input for Functions [\#816](https://github.com/stashed/stash/pull/816) ([hossainemruz](https://github.com/hossainemruz))
- Add TARGET_RESOURCE variable for BackupConfigurationTemplate [\#814](https://github.com/stashed/stash/pull/814) ([hossainemruz](https://github.com/hossainemruz))
- Add make install, uninstall, purge commands [\#813](https://github.com/stashed/stash/pull/813) ([tamalsaha](https://github.com/tamalsaha))
- Move chart & deploy scripts to github.com/stashed/installer [\#811](https://github.com/stashed/stash/pull/811) ([tamalsaha](https://github.com/tamalsaha))
- Move docs to github.com/stashed/docs repo [\#810](https://github.com/stashed/stash/pull/810) ([tamalsaha](https://github.com/tamalsaha))
- Restore PVCs from templates using Restic [\#809](https://github.com/stashed/stash/pull/809) ([hossainemruz](https://github.com/hossainemruz))
- Move HandleResticError to util package [\#806](https://github.com/stashed/stash/pull/806) ([tamalsaha](https://github.com/tamalsaha))
- Remove canary support [\#805](https://github.com/stashed/stash/pull/805) ([tamalsaha]https://github.com/tamalsaha())
- Fix travis build [\#804](https://github.com/stashed/stash/pull/804) ([tamalsaha](https://github.com/tamalsaha))
- Update Version.go [\#803](https://github.com/stashed/stash/pull/803) ([tamalsaha](https://github.com/tamalsaha))
- Added ARM64 support to the install script and manifest [\#802](https://github.com/stashed/stash/pull/802) ([tamalsaha](https://github.com/tamalsaha))
- Pass labels to offshoot + add generic offshoot labels [\#801](https://github.com/stashed/stash/pull/801) ([hossainemruz](https://github.com/hossainemruz))
- Add Makefile [\#800](https://github.com/stashed/stash/pull/800) ([tamalsaha](https://github.com/tamalsaha))
- Skip BackupSession creation if target does not exist + use timestamp … [\#797](https://github.com/stashed/stash/pull/797) ([hossainemruz](https://github.com/hossainemruz))
- Use absolute path as aliases for reference docs. [\#796](https://github.com/stashed/stash/pull/796) ([tamalsaha](https://github.com/tamalsaha)) 
- Remove importance of order of rule in RestoreSession [\#795](https://github.com/stashed/stash/pull/795) ([hossainemruz](https://github.com/hossainemruz))
- Use restic 0.9.5 [\#789](https://github.com/stashed/stash/pull/789) ([hossainemruz](https://github.com/hossainemruz))
- VolumeSnapshot [\#787](https://github.com/stashed/stash/pull/787) ([suaas21](https://github.com/suaas21))
- Fix: User and group creation of stash for mongodb and mysql [\#786](https://github.com/stashed/stash/pull/786) ([the-redback](https://github.com/the-redback))
- Update backup manager [\#782](https://github.com/stashed/stash/pull/782) ([tamalsaha]https://github.com/tamalsaha())
- Configure Env variables for Functions [\#780](https://github.com/stashed/stash/pull/780) ([tamalsaha](https://github.com/tamalsaha))
- Fix rest backend for workloads + add more authentication method for swift backend [\#778](https://github.com/stashed/stash/pull/778) ([hossainemruz](https://github.com/hossainemruz))
- Update package path to stash.appscode.dev/stash [\#776](https://github.com/stashed/stash/pull/776) ([tamalsaha](https://github.com/tamalsaha))
- Update to k8s 1.14.0 client libraries using go.mod [\#775](https://github.com/stashed/stash/pull/775) ([tamalsaha](https://github.com/tamalsaha))
- MutatingWebhooks must be without side-effect [\#773](https://github.com/stashed/stash/pull/773) ([suaas21](https://github.com/suaas21))
- Introduce VolumeSnapshot APIs [\#772](https://github.com/stashed/stash/pull/772) ([hossainemruz](https://github.com/hossainemruz))
- Use osm pkg from kmodules/objectstore-api [\#770](https://github.com/stashed/stash/pull/770) ([tamalsaha](https://github.com/tamalsaha))
- Remove --rbac flag [\#761](https://github.com/stashed/stash/pull/761) ([suaas21](https://github.com/suaas21))
Skip creating/processing backup-session when backup-config is paused [\#759](https://github.com/stashed/stash/pull/759) ([diptadas](https://github.com/diptadas))
- Add "Supported Backends" doc to new guides [\#756](https://github.com/stashed/stash/pull/756) ([hossainemruz](https://github.com/hossainemruz))
- Add guides template for new design [\#755](https://github.com/stashed/stash/pull/755) ([hossainemruz](https://github.com/hossainemruz))
- Run restic commands using docker [\#754](https://github.com/stashed/stash/pull/754) ([diptadas](https://github.com/diptadas))
- Stash v1beta1 E2E test for PVC [\#753](https://github.com/stashed/stash/pull/753) ([suaas21](https://github.com/suaas21))
- Update Kubernetes client libraries to 1.13.5 [\#752](https://github.com/stashed/stash/pull/752) [https://github.com/stashed/stash/pull/752] ([tamalsaha](https://github.com/tamalsaha))
- Enable pipefail and update restore yamls [\#750](https://github.com/stashed/stash/pull/750) ([diptadas](https://github.com/diptadas))
- Implement snapshots for v1beta1 api [\#749](https://github.com/stashed/stash/pull/749) ([diptadas](https://github.com/diptadas))
- Stash v1beta1 E2E test for ReplicaSet [\#747](https://github.com/stashed/stash/pull/747) ([suaas21](https://github.com/suaas21))
- Apply nice/ionice settings from env [\#746](https://github.com/stashed/stash/pull/746) ([diptadas](https://github.com/diptadas))
- Fixed scratch-dir, output-dir and hostname in functions/tasks yamls [\#744](https://github.com/stashed/stash/pull/744) ([diptadas](https://github.com/diptadas))
- Stash v1beta1 E2E test for ReplicationController [#742](https://github.com/stashed/stash/pull/742) ([suaas21](https://github.com/suaas21))
- Stash v1beta1 E2E test for DaemonSet [\#741](https://github.com/stashed/stash/pull/741) ([suaas21](https://github.com/suaas21))
- Update concept doc [\#739](https://github.com/stashed/stash/pull/739) ([hossainemruz](https://github.com/hossainemruz))
- Stash V1beta1 E2E test for StatefulSet [\#737](https://github.com/stashed/stash/pull/737) ([suaas21](https://github.com/suaas21))
- Attach volume for local backend [\#736](https://github.com/stashed/stash/pull/736) ([diptadas](https://github.com/diptadas))
- Add Stash CLI [\#734](https://github.com/stashed/stash/pull/734) ([diptadas](https://github.com/diptadas))
- Fix openapi path prefixes for validators and mutators [\#732](https://github.com/stashed/stash/pull/732) ([tamalsaha](https://github.com/tamalsaha))
- Add max-connections for GCS, Azure, B2 backend [\#730](https://github.com/stashed/stash/pull/730) ([diptadas](https://github.com/diptadas))
- Support PSP enabled cluster [\#729](https://github.com/stashed/stash/pull/729) ([hossainemruz](https://github.com/hossainemruz))
- Use FailurePolicy ignore for K8s resource webhooks [\#726](https://github.com/stashed/stash/pull/726) ([diptadas](https://github.com/diptadas))
- Rename admission webhooks to avoid name collision [\#725](https://github.com/stashed/stash/pull/725) ([diptadas](https://github.com/diptadas))
- Don't write secret data inside temp dir [\#724](https://github.com/stashed/stash/pull/724) ([diptadas](https://github.com/diptadas))
- Add support for backup cluster resources YAML [\#721](https://github.com/stashed/stash/pull/721) ([hossainemruz](https://github.com/hossainemruz))
- Add TempDir and PSP settings for Function [\#720](https://github.com/stashed/stash/pull/720) ([tamalsaha](https://github.com/tamalsaha))
- Apply EmptyDir settings to TmpDir [\#719](https://github.com/stashed/stash/pull/719) ([diptadas](https://github.com/diptadas))
- Use cleanup-cache flag [\#717](https://github.com/stashed/stash/pull/717) ([diptadas](https://github.com/diptadas))
- Use ionice and nice with Restic CMD [\#716](https://github.com/stashed/stash/pull/716) ([diptadas](https://github.com/diptadas))
- Add support for OpenShift DeploymentConfig [\#714](https://github.com/stashed/stash/pull/714) ([hossainemruz](https://github.com/hossainemruz))
- Add support for rest backend [\#713](https://github.com/stashed/stash/pull/713) ([diptadas](https://github.com/diptadas))
- Stash V1beta1 E2E test for Deployment [\#710](https://github.com/stashed/stash/pull/710) ([suaas21](https://github.com/suaas21))
- Backup and restore Elasticsearch [\#702](https://github.com/stashed/stash/pull/702) ([diptadas](https://github.com/diptadas))
- Add BackupSession Controller for Sidecar [\#701](https://github.com/stashed/stash/pull/701) ([suaas21](https://github.com/suaas21))
- Backup and restore Mongo DB [\#699](https://github.com/stashed/stash/pull/699) ([diptadas](https://github.com/diptadas))
- Backup and restore MySQL DB [\#696](https://github.com/stashed/stash/pull/696) ([diptadas](https://github.com/diptadas))
- Backup and restore Postgres DB [\#695](https://github.com/stashed/stash/pull/695) ([diptadas](https://github.com/diptadas))
- Backup from stdin and dump to stdout [\#694](https://github.com/stashed/stash/pull/694) ([diptadas](https://github.com/diptadas))
- Post backup/restore status update [\#691](https://github.com/stashed/stash/pull/691) ([diptadas](https://github.com/diptadas))
- Use ContainerRuntimeSettings in Function spec [\#689](https://github.com/stashed/stash/pull/689) ([diptadas](https://github.com/diptadas))
- Fix v1beta1 api for BackupConfigurationTemplate [\#688](https://github.com/stashed/stash/pull/688) ([hossainemruz](https://github.com/hossainemruz))
- Update Kubernetes client libraries to 1.13.0 [\#687](https://github.com/stashed/stash/pull/687) ([tamalsaha](https://github.com/tamalsaha))
- Backup and restore PVC [\#676](https://github.com/stashed/stash/pull/676) ([diptadas](https://github.com/diptadas))
- Update workload controller for new design [\#675](https://github.com/stashed/stash/pull/675) ([hossainemruz](https://github.com/hossainemruz))
- Resolve tasks for backup/restore sessions [\#674](https://github.com/stashed/stash/pull/674) ([diptadas](https://github.com/diptadas))
- Add restic wrapper library [\#673](https://github.com/stashed/stash/pull/673) ([hossainemruz](https://github.com/hossainemruz))
- Add BackupConfiguration Controller [\#671](https://github.com/stashed/stash/pull/671) ([suaas21](https://github.com/suaas21))
- Introduce v1beta1 api [\#647](https://github.com/stashed/stash/pull/647) ([hossainemruz](https://github.com/hossainemruz))
