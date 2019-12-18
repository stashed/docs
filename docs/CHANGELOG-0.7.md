---
title: Changelog | Stash
description: Changelog
menu:
  docs_{{ .version }}:
    identifier: changelog-stash-0.7
    name: Changelog-0.7
    parent: welcome
    weight: 70
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: welcome
url: /docs/{{ .version }}/welcome/changelog-0.7/
aliases:
  - /docs/{{ .version }}/CHANGELOG-0.7/
---
# Change Log

## [0.7.0](https://github.com/appscode/stash/tree/0.7.0) (2018-05-29)
[Full Changelog](https://github.com/appscode/stash/compare/0.7.0-rc.5...0.7.0)

**Implemented enhancements:**

- Support custom CA cert with backend [\#288](https://github.com/appscode/stash/issues/288)

**Fixed bugs:**

- Pod restart after each backup when Mutating Webhook enabled [\#396](https://github.com/appscode/stash/issues/396)
- Sidecar RoleBinding is not being created when Mutating Webhook is enabled  [\#395](https://github.com/appscode/stash/issues/395)
- Recovery to PVC restores data in subdirectory instead of root directory [\#392](https://github.com/appscode/stash/issues/392)
- Forget panics in 0.7.0-rc.0 [\#373](https://github.com/appscode/stash/issues/373)

**Closed issues:**

- Resource type "snapshot" not registered [\#499](https://github.com/appscode/stash/issues/499)
- Support Repository deletion [\#416](https://github.com/appscode/stash/issues/416)
- Docs TODO [\#414](https://github.com/appscode/stash/issues/414)
- Convert Initializer to MutationWebhook [\#326](https://github.com/appscode/stash/issues/326)
- Use informer factory for backup scheduler [\#321](https://github.com/appscode/stash/issues/321)
- Show repository snapshot list [\#319](https://github.com/appscode/stash/issues/319)
- Verbosity \(--v\) flag not inherited to backup sidecars [\#282](https://github.com/appscode/stash/issues/282)
- Double Deployment patch when deleting a Restic CRD? [\#281](https://github.com/appscode/stash/issues/281)
- Consider a simple 'enabled' switch for Restic CRD [\#279](https://github.com/appscode/stash/issues/279)
- offline backup is not supported for workload kind `Deployment`, `Replicaset` and `ReplicationController` with `replicas \> 1` [\#244](https://github.com/appscode/stash/issues/244)
- Recover specific snapshot ID [\#215](https://github.com/appscode/stash/issues/215)

**Merged pull requests:**

- Prepare docs for 0.7.0 release. [\#502](https://github.com/appscode/stash/pull/502) ([tamalsaha](https://github.com/tamalsaha))
- Set RollingUpdate for DaemonSet [\#349](https://github.com/appscode/stash/pull/349) ([tamalsaha](https://github.com/tamalsaha))

## [0.7.0-rc.5](https://github.com/appscode/stash/tree/0.7.0-rc.5) (2018-05-23)
[Full Changelog](https://github.com/appscode/stash/compare/0.7.0-rc.4...0.7.0-rc.5)

**Fixed bugs:**

- Fix storage implementation for snapshots [\#497](https://github.com/appscode/stash/pull/497) ([tamalsaha](https://github.com/tamalsaha))

**Merged pull requests:**

- Prepare docs for 0.7.0-rc.5 [\#498](https://github.com/appscode/stash/pull/498) ([tamalsaha](https://github.com/tamalsaha))
- Update changelog [\#495](https://github.com/appscode/stash/pull/495) ([tamalsaha](https://github.com/tamalsaha))

## [0.7.0-rc.4](https://github.com/appscode/stash/tree/0.7.0-rc.4) (2018-05-22)
[Full Changelog](https://github.com/appscode/stash/compare/0.7.0-rc.3...0.7.0-rc.4)

**Fixed bugs:**

- Restic sidecar not properly working because of image tag error [\#443](https://github.com/appscode/stash/issues/443)
- Removed owner reference from repo-reader role-binding [\#484](https://github.com/appscode/stash/pull/484) ([hossainemruz](https://github.com/hossainemruz))
- Permit stash operator to perform pods/exec [\#433](https://github.com/appscode/stash/pull/433) ([tamalsaha](https://github.com/tamalsaha))
- Add missing batch jobs get RBAC permission [\#419](https://github.com/appscode/stash/pull/419) ([galexrt](https://github.com/galexrt))

**Closed issues:**

- Stash restore pod fails with istio sidecar [\#475](https://github.com/appscode/stash/issues/475)
- Stash stores GCS credentials in /tmp with 644 permissions [\#470](https://github.com/appscode/stash/issues/470)
- Update minio doc for 1.10? [\#467](https://github.com/appscode/stash/issues/467)
- Fix docs for StatefulSet [\#444](https://github.com/appscode/stash/issues/444)

**Merged pull requests:**

- Delete user roles on purge. [\#494](https://github.com/appscode/stash/pull/494) ([tamalsaha](https://github.com/tamalsaha))
- Add app: stash label to user roles. [\#493](https://github.com/appscode/stash/pull/493) ([tamalsaha](https://github.com/tamalsaha))
- Use post-install hooks to install admission controller in chart [\#492](https://github.com/appscode/stash/pull/492) ([tamalsaha](https://github.com/tamalsaha))
- Update changelog [\#491](https://github.com/appscode/stash/pull/491) ([tamalsaha](https://github.com/tamalsaha))
- Avoid creating apiservice when webhooks are not used. [\#490](https://github.com/appscode/stash/pull/490) ([tamalsaha](https://github.com/tamalsaha))
- Install correct version of stash chart [\#489](https://github.com/appscode/stash/pull/489) ([tamalsaha](https://github.com/tamalsaha))
- Use wait-until instead of fixed delay  [\#488](https://github.com/appscode/stash/pull/488) ([hossainemruz](https://github.com/hossainemruz))
- Concourse [\#486](https://github.com/appscode/stash/pull/486) ([tahsinrahman](https://github.com/tahsinrahman))
- Prepare docs for 0.7.0-rc.4 [\#483](https://github.com/appscode/stash/pull/483) ([tamalsaha](https://github.com/tamalsaha))
- Revendor [\#481](https://github.com/appscode/stash/pull/481) ([tamalsaha](https://github.com/tamalsaha))
- Fix enableRBAC  flag for sidecar [\#480](https://github.com/appscode/stash/pull/480) ([hossainemruz](https://github.com/hossainemruz))
- Typo \(`Weclome` → `Welcome`\) in page title [\#479](https://github.com/appscode/stash/pull/479) ([eliasp](https://github.com/eliasp))
- Add support for initial backoff to the apiserver call on recover [\#476](https://github.com/appscode/stash/pull/476) ([farcaller](https://github.com/farcaller))
- Support recovering from repository in different namespace [\#474](https://github.com/appscode/stash/pull/474) ([tamalsaha](https://github.com/tamalsaha))
- Update docs \(run minio in v1.9.4+ cluster and add example yaml files in respective backends\) [\#473](https://github.com/appscode/stash/pull/473) ([hossainemruz](https://github.com/hossainemruz))
- Limit the GCS file permissions to owner only [\#472](https://github.com/appscode/stash/pull/472) ([farcaller](https://github.com/farcaller))
- Fix a typo [\#471](https://github.com/appscode/stash/pull/471) ([farcaller](https://github.com/farcaller))
- Don't panic if admission options is nil [\#469](https://github.com/appscode/stash/pull/469) ([tamalsaha](https://github.com/tamalsaha))
- Disable admission controllers for webhook server [\#468](https://github.com/appscode/stash/pull/468) ([tamalsaha](https://github.com/tamalsaha))
- Use new UpdateRecoveryStatus method [\#466](https://github.com/appscode/stash/pull/466) ([tamalsaha](https://github.com/tamalsaha))
- Add Update\*\*\*Status helpers [\#465](https://github.com/appscode/stash/pull/465) ([tamalsaha](https://github.com/tamalsaha))
- Added SSL support for deleting restic repository from Minio backend [\#464](https://github.com/appscode/stash/pull/464) ([hossainemruz](https://github.com/hossainemruz))
- Update client-go to 7.0.0 [\#463](https://github.com/appscode/stash/pull/463) ([tamalsaha](https://github.com/tamalsaha))
- Rename webhook files in chart [\#460](https://github.com/appscode/stash/pull/460) ([tamalsaha](https://github.com/tamalsaha))
- Update workload api [\#459](https://github.com/appscode/stash/pull/459) ([tamalsaha](https://github.com/tamalsaha))
- Remove stash crds before uninstalling operator [\#458](https://github.com/appscode/stash/pull/458) ([tamalsaha](https://github.com/tamalsaha))
- Export kube-ca only if required [\#457](https://github.com/appscode/stash/pull/457) ([tamalsaha](https://github.com/tamalsaha))
- Improve installer [\#456](https://github.com/appscode/stash/pull/456) ([tamalsaha](https://github.com/tamalsaha))
- Update changelog [\#455](https://github.com/appscode/stash/pull/455) ([tamalsaha](https://github.com/tamalsaha))
- Various installer fixes [\#454](https://github.com/appscode/stash/pull/454) ([tamalsaha](https://github.com/tamalsaha))
- Update workload client [\#453](https://github.com/appscode/stash/pull/453) ([tamalsaha](https://github.com/tamalsaha))
- Update workload client [\#452](https://github.com/appscode/stash/pull/452) ([tamalsaha](https://github.com/tamalsaha))
- Revendor workload client [\#451](https://github.com/appscode/stash/pull/451) ([tamalsaha](https://github.com/tamalsaha))
- Update workload api [\#450](https://github.com/appscode/stash/pull/450) ([tamalsaha](https://github.com/tamalsaha))
- Fixes RBAC permission for scaledownCronJob [\#449](https://github.com/appscode/stash/pull/449) ([hossainemruz](https://github.com/hossainemruz))
- Used Snapshot  to verify successful backup [\#447](https://github.com/appscode/stash/pull/447) ([hossainemruz](https://github.com/hossainemruz))
- Some cleanup [\#446](https://github.com/appscode/stash/pull/446) ([tamalsaha](https://github.com/tamalsaha))
- Update StatefulSet doc [\#445](https://github.com/appscode/stash/pull/445) ([hossainemruz](https://github.com/hossainemruz))
- pkg/util: fix error found by vet [\#442](https://github.com/appscode/stash/pull/442) ([functionary](https://github.com/functionary))
- Move Stash swagger.json to top level folder [\#441](https://github.com/appscode/stash/pull/441) ([tamalsaha](https://github.com/tamalsaha))
- Fix go\_vet error [\#440](https://github.com/appscode/stash/pull/440) ([hossainemruz](https://github.com/hossainemruz))
- Delete restic repository from backend if Repository CRD is deleted [\#438](https://github.com/appscode/stash/pull/438) ([hossainemruz](https://github.com/hossainemruz))
- Recover specific snapshot [\#437](https://github.com/appscode/stash/pull/437) ([hossainemruz](https://github.com/hossainemruz))
- Use Repository data in Recovery CRD [\#436](https://github.com/appscode/stash/pull/436) ([hossainemruz](https://github.com/hossainemruz))
- Increase qps and burst limits [\#435](https://github.com/appscode/stash/pull/435) ([tamalsaha](https://github.com/tamalsaha))
- Add RBAC instructions for GKE cluster [\#432](https://github.com/appscode/stash/pull/432) ([tamalsaha](https://github.com/tamalsaha))
- Update charts location [\#431](https://github.com/appscode/stash/pull/431) ([tamalsaha](https://github.com/tamalsaha))
- Add docs for GKE and Rook [\#430](https://github.com/appscode/stash/pull/430) ([hossainemruz](https://github.com/hossainemruz))
- concourse configs [\#429](https://github.com/appscode/stash/pull/429) ([tahsinrahman](https://github.com/tahsinrahman))
- Skip lock while listing snapshots [\#428](https://github.com/appscode/stash/pull/428) ([hossainemruz](https://github.com/hossainemruz))
- Purge repository objects in installer [\#427](https://github.com/appscode/stash/pull/427) ([tamalsaha](https://github.com/tamalsaha))
- Support installing from local installer scripts [\#426](https://github.com/appscode/stash/pull/426) ([tamalsaha](https://github.com/tamalsaha))
- Fixed Repository yaml in doc [\#425](https://github.com/appscode/stash/pull/425) ([hossainemruz](https://github.com/hossainemruz))
- Add delete method for snapshots to swagger.json [\#424](https://github.com/appscode/stash/pull/424) ([tamalsaha](https://github.com/tamalsaha))
- Generate swagger.json [\#423](https://github.com/appscode/stash/pull/423) ([tamalsaha](https://github.com/tamalsaha))
- Add install pkg for stash crds [\#422](https://github.com/appscode/stash/pull/422) ([tamalsaha](https://github.com/tamalsaha))
- Fix openapi spec for stash crds [\#421](https://github.com/appscode/stash/pull/421) ([tamalsaha](https://github.com/tamalsaha))
- Expose swagger.json [\#420](https://github.com/appscode/stash/pull/420) ([tamalsaha](https://github.com/tamalsaha))
- Show repository snapshot list [\#417](https://github.com/appscode/stash/pull/417) ([hossainemruz](https://github.com/hossainemruz))
- Add registry skeleton for snapshots [\#415](https://github.com/appscode/stash/pull/415) ([tamalsaha](https://github.com/tamalsaha))
- Update chart readme [\#413](https://github.com/appscode/stash/pull/413) ([tamalsaha](https://github.com/tamalsaha))

## [0.7.0-rc.3](https://github.com/appscode/stash/tree/0.7.0-rc.3) (2018-04-03)
[Full Changelog](https://github.com/appscode/stash/compare/0.7.0-rc.2...0.7.0-rc.3)

**Fixed bugs:**

- Use separate registry key for docker images [\#410](https://github.com/appscode/stash/pull/410) ([tamalsaha](https://github.com/tamalsaha))
- Revendor webhook util and jsonpatch fixes [\#400](https://github.com/appscode/stash/pull/400) ([tamalsaha](https://github.com/tamalsaha))

**Closed issues:**

- hack/deploy/stash.sh: $? check does not work with set -e [\#403](https://github.com/appscode/stash/issues/403)

**Merged pull requests:**

- Add frontmatter for repository crd [\#412](https://github.com/appscode/stash/pull/412) ([tamalsaha](https://github.com/tamalsaha))
- Prepare docs for 0.7.0-rc.3 [\#411](https://github.com/appscode/stash/pull/411) ([tamalsaha](https://github.com/tamalsaha))
- Add test for recovery [\#409](https://github.com/appscode/stash/pull/409) ([hossainemruz](https://github.com/hossainemruz))
- Skip setting ListKind [\#407](https://github.com/appscode/stash/pull/407) ([tamalsaha](https://github.com/tamalsaha))
- Add CRD Validation [\#406](https://github.com/appscode/stash/pull/406) ([tamalsaha](https://github.com/tamalsaha))
- Generate openapi spec for stash api [\#405](https://github.com/appscode/stash/pull/405) ([tamalsaha](https://github.com/tamalsaha))
- Fix install script for minikube 0.24.x \(Kube 1.8.0\) [\#404](https://github.com/appscode/stash/pull/404) ([tamalsaha](https://github.com/tamalsaha))
- Skip downloading onessl if already installed [\#401](https://github.com/appscode/stash/pull/401) ([tamalsaha](https://github.com/tamalsaha))
- Use Restic spec hash instead of resource version to restart pods [\#399](https://github.com/appscode/stash/pull/399) ([tamalsaha](https://github.com/tamalsaha))
- Check for valid owner object [\#397](https://github.com/appscode/stash/pull/397) ([tamalsaha](https://github.com/tamalsaha))
- Create repository crd for each Restic repository [\#394](https://github.com/appscode/stash/pull/394) ([hossainemruz](https://github.com/hossainemruz))
- Revendor webhook library [\#393](https://github.com/appscode/stash/pull/393) ([tamalsaha](https://github.com/tamalsaha))

## [0.7.0-rc.2](https://github.com/appscode/stash/tree/0.7.0-rc.2) (2018-03-24)
[Full Changelog](https://github.com/appscode/stash/compare/0.7.0-rc.1...0.7.0-rc.2)

**Fixed bugs:**

- Fix --enable-analytics flag [\#387](https://github.com/appscode/stash/pull/387) ([tamalsaha](https://github.com/tamalsaha))
- Fix flag parsing in tests [\#386](https://github.com/appscode/stash/pull/386) ([tamalsaha](https://github.com/tamalsaha))

**Merged pull requests:**

- Prepare docs for 0.7.0-rc.2 [\#391](https://github.com/appscode/stash/pull/391) ([tamalsaha](https://github.com/tamalsaha))
- Add variable for dockerRegistry [\#390](https://github.com/appscode/stash/pull/390) ([tamalsaha](https://github.com/tamalsaha))
- Reorg objects deleted in uninstall command [\#389](https://github.com/appscode/stash/pull/389) ([tamalsaha](https://github.com/tamalsaha))
- Fix Statefulset Example [\#385](https://github.com/appscode/stash/pull/385) ([rzcastilho](https://github.com/rzcastilho))
- Rename --analytics to --enable-analytics [\#384](https://github.com/appscode/stash/pull/384) ([tamalsaha](https://github.com/tamalsaha))
- Use separated appscode/kubernetes-webhook-util package [\#383](https://github.com/appscode/stash/pull/383) ([tamalsaha](https://github.com/tamalsaha))

## [0.7.0-rc.1](https://github.com/appscode/stash/tree/0.7.0-rc.1) (2018-03-21)
[Full Changelog](https://github.com/appscode/stash/compare/0.7.0-rc.0...0.7.0-rc.1)

**Fixed bugs:**

- Don't enable mutator for StatefulSet updates [\#381](https://github.com/appscode/stash/pull/381) ([tamalsaha](https://github.com/tamalsaha))
- Stop using field selectors for CRDs [\#379](https://github.com/appscode/stash/pull/379) ([tamalsaha](https://github.com/tamalsaha))

**Closed issues:**

- "DeprecatedServiceAccount not present in src" while converting unversioned StatefulSet to v1beta1.StatefulSet  [\#371](https://github.com/appscode/stash/issues/371)
- \[0.6.x\] Helm chart broken due to undocumented '--docker-registry' and other arguments [\#354](https://github.com/appscode/stash/issues/354)
- \[0.7.0-rc.0\] Fails on start-up with 'cluster doesn't provide requestheader-client-ca-file' [\#353](https://github.com/appscode/stash/issues/353)
- Ability to backup volumes with ReadWriteOnce access mode [\#350](https://github.com/appscode/stash/issues/350)
- Recovery not working! [\#303](https://github.com/appscode/stash/issues/303)

**Merged pull requests:**

- Update the image tag in operator.yaml [\#382](https://github.com/appscode/stash/pull/382) ([tamalsaha](https://github.com/tamalsaha))
- Update docs to 0.7.0-rc.1 [\#380](https://github.com/appscode/stash/pull/380) ([tamalsaha](https://github.com/tamalsaha))
-  Add types for Repository apigroup [\#377](https://github.com/appscode/stash/pull/377) ([tamalsaha](https://github.com/tamalsaha))
- Add missing front matter [\#376](https://github.com/appscode/stash/pull/376) ([tamalsaha](https://github.com/tamalsaha))
- Check for check job before creating it [\#375](https://github.com/appscode/stash/pull/375) ([galexrt](https://github.com/galexrt))
- Add travis.yaml [\#370](https://github.com/appscode/stash/pull/370) ([tamalsaha](https://github.com/tamalsaha))
- Add --purge flag [\#369](https://github.com/appscode/stash/pull/369) ([tamalsaha](https://github.com/tamalsaha))
- Make it clear that installer is a single command [\#365](https://github.com/appscode/stash/pull/365) ([tamalsaha](https://github.com/tamalsaha))
- Update installer [\#364](https://github.com/appscode/stash/pull/364) ([tamalsaha](https://github.com/tamalsaha))
- Replace initializers with mutation webhook for workloads [\#363](https://github.com/appscode/stash/pull/363) ([hossainemruz](https://github.com/hossainemruz))
- Update chart to match RBAC best practices for charts [\#362](https://github.com/appscode/stash/pull/362) ([tamalsaha](https://github.com/tamalsaha))
- Add checks to installer script [\#361](https://github.com/appscode/stash/pull/361) ([tamalsaha](https://github.com/tamalsaha))
- Use admission hook helpers from kutil [\#360](https://github.com/appscode/stash/pull/360) ([tamalsaha](https://github.com/tamalsaha))
- Fix admission webhook flag [\#359](https://github.com/appscode/stash/pull/359) ([tamalsaha](https://github.com/tamalsaha))
- Support --enable-admission-webhook=false [\#358](https://github.com/appscode/stash/pull/358) ([tamalsaha](https://github.com/tamalsaha))
- Support multiple webhooks of same apiversion [\#357](https://github.com/appscode/stash/pull/357) ([tamalsaha](https://github.com/tamalsaha))
- Sync chart to stable charts repo [\#356](https://github.com/appscode/stash/pull/356) ([tamalsaha](https://github.com/tamalsaha))
- Use restic 0.8.3 [\#355](https://github.com/appscode/stash/pull/355) ([tamalsaha](https://github.com/tamalsaha))
- Update README.md [\#352](https://github.com/appscode/stash/pull/352) ([tamalsaha](https://github.com/tamalsaha))

## [0.7.0-rc.0](https://github.com/appscode/stash/tree/0.7.0-rc.0) (2018-02-20)
[Full Changelog](https://github.com/appscode/stash/compare/0.6.4...0.7.0-rc.0)

**Merged pull requests:**

- Document user roles [\#348](https://github.com/appscode/stash/pull/348) ([tamalsaha](https://github.com/tamalsaha))
- Add changelog for 0.7.0-rc.0 [\#347](https://github.com/appscode/stash/pull/347) ([tamalsaha](https://github.com/tamalsaha))
- Add a parameter to allow disabling initializers [\#346](https://github.com/appscode/stash/pull/346) ([mcanevet](https://github.com/mcanevet))
- Update readme to point to 0.6.4 [\#345](https://github.com/appscode/stash/pull/345) ([tamalsaha](https://github.com/tamalsaha))
- Implement offline backup for multiple replica [\#335](https://github.com/appscode/stash/pull/335) ([hossainemruz](https://github.com/hossainemruz))

