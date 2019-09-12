---
title: Changelog | Stash
description: Changelog
menu:
  product_stash_{{ .Version }}:
    identifier: changelog-stash-0.6
    name: Changelog-0.6
    parent: welcome
    weight: 60
product_name: stash
menu_name: product_stash_{{ .Version }}
section_menu_id: welcome
url: /products/stash/{{ .Version }}/welcome/changelog-0.6/
aliases:
  - /products/stash/{{ .Version }}/CHANGELOG-0.6/
---
# Change Log

## [0.6.4](https://github.com/appscode/stash/tree/0.6.4) (2018-02-20)
[Full Changelog](https://github.com/appscode/stash/compare/0.6.3...0.6.4)

**Fixed bugs:**

- Backup count rises even when backup/init fails [\#293](https://github.com/appscode/stash/issues/293)

**Closed issues:**

- Document HTTP endpoints [\#111](https://github.com/appscode/stash/issues/111)
- Support updating version of resitc side-car [\#72](https://github.com/appscode/stash/issues/72)

**Merged pull requests:**

- Update docs for 0.6.4 [\#344](https://github.com/appscode/stash/pull/344) ([tamalsaha](https://github.com/tamalsaha))
- Don't block deletion of owner by default [\#343](https://github.com/appscode/stash/pull/343) ([tamalsaha](https://github.com/tamalsaha))
- Don't block deletion of owner by default [\#342](https://github.com/appscode/stash/pull/342) ([tamalsaha](https://github.com/tamalsaha))
- Skip generating UpdateStatus method [\#341](https://github.com/appscode/stash/pull/341) ([tamalsaha](https://github.com/tamalsaha))
- Remove internal types [\#340](https://github.com/appscode/stash/pull/340) ([tamalsaha](https://github.com/tamalsaha))
- Use rbac/v1 apis [\#339](https://github.com/appscode/stash/pull/339) ([tamalsaha](https://github.com/tamalsaha))
- Add user roles [\#338](https://github.com/appscode/stash/pull/338) ([tamalsaha](https://github.com/tamalsaha))
- Use restic 0.8.2 [\#337](https://github.com/appscode/stash/pull/337) ([tamalsaha](https://github.com/tamalsaha))
- Use official code generator scripts [\#336](https://github.com/appscode/stash/pull/336) ([tamalsaha](https://github.com/tamalsaha))
- Update charts to support api registration [\#334](https://github.com/appscode/stash/pull/334) ([tamalsaha](https://github.com/tamalsaha))
- Fix e2e tests after webhook merger [\#333](https://github.com/appscode/stash/pull/333) ([tamalsaha](https://github.com/tamalsaha))
- Ensure stash can be run locally [\#332](https://github.com/appscode/stash/pull/332) ([tamalsaha](https://github.com/tamalsaha))
- Vendor client-go auth pkg [\#331](https://github.com/appscode/stash/pull/331) ([tamalsaha](https://github.com/tamalsaha))
- Update Grafana dashboard [\#330](https://github.com/appscode/stash/pull/330) ([galexrt](https://github.com/galexrt))
- Merge admission webhook and operator into one binary [\#329](https://github.com/appscode/stash/pull/329) ([tamalsaha](https://github.com/tamalsaha))
- Merge uninstall script into the stash.sh script [\#328](https://github.com/appscode/stash/pull/328) ([tamalsaha](https://github.com/tamalsaha))
- Implement informer factory for backup scheduler [\#325](https://github.com/appscode/stash/pull/325) ([hossainemruz](https://github.com/hossainemruz))
- Fixed abnormal pod recreation when Restic is deleted [\#322](https://github.com/appscode/stash/pull/322) ([hossainemruz](https://github.com/hossainemruz))
- Copy generic-admission-server into pkg [\#318](https://github.com/appscode/stash/pull/318) ([tamalsaha](https://github.com/tamalsaha))
- Use shared infromer factory [\#317](https://github.com/appscode/stash/pull/317) ([tamalsaha](https://github.com/tamalsaha))
- Use GetBaseVersion method from kutil [\#316](https://github.com/appscode/stash/pull/316) ([tamalsaha](https://github.com/tamalsaha))
- Implement Pause Restic [\#315](https://github.com/appscode/stash/pull/315) ([hossainemruz](https://github.com/hossainemruz))
- Fix webhook command description [\#314](https://github.com/appscode/stash/pull/314) ([tamalsaha](https://github.com/tamalsaha))
- Use rbac/v1beta1 api. [\#313](https://github.com/appscode/stash/pull/313) ([tamalsaha](https://github.com/tamalsaha))
- Support Create & Update operations in admission webhook [\#312](https://github.com/appscode/stash/pull/312) ([tamalsaha](https://github.com/tamalsaha))
- Merge webhook plugins into one. [\#311](https://github.com/appscode/stash/pull/311) ([tamalsaha](https://github.com/tamalsaha))
- Support private docker registry in installer [\#310](https://github.com/appscode/stash/pull/310) ([tamalsaha](https://github.com/tamalsaha))
- Compress go binaries [\#309](https://github.com/appscode/stash/pull/309) ([tamalsaha](https://github.com/tamalsaha))
- Rename --initializer flag to --enable-initializer [\#308](https://github.com/appscode/stash/pull/308) ([tamalsaha](https://github.com/tamalsaha))
- Remove STASH\_ROLE\_TYPE from installer scripts [\#307](https://github.com/appscode/stash/pull/307) ([tamalsaha](https://github.com/tamalsaha))
- Use rbac/v1 api [\#306](https://github.com/appscode/stash/pull/306) ([tamalsaha](https://github.com/tamalsaha))
- Use kubectl auth reconcile [\#305](https://github.com/appscode/stash/pull/305) ([tamalsaha](https://github.com/tamalsaha))
- Add --initializer flag to installer [\#304](https://github.com/appscode/stash/pull/304) ([tamalsaha](https://github.com/tamalsaha))
- Prepare docs for 0.7.0-alpha.0 [\#302](https://github.com/appscode/stash/pull/302) ([tamalsaha](https://github.com/tamalsaha))
- Change installer script [\#301](https://github.com/appscode/stash/pull/301) ([tamalsaha](https://github.com/tamalsaha))
- Added support for private docker registry [\#300](https://github.com/appscode/stash/pull/300) ([diptadas](https://github.com/diptadas))
- Add ValidatingAdmissionWebhook for Stash CRDs [\#299](https://github.com/appscode/stash/pull/299) ([tamalsaha](https://github.com/tamalsaha))
- Remove TPR to CRD migrator [\#298](https://github.com/appscode/stash/pull/298) ([tamalsaha](https://github.com/tamalsaha))
- Update dependencies to Kubernetes 1.9 [\#297](https://github.com/appscode/stash/pull/297) ([tamalsaha](https://github.com/tamalsaha))
- Write restic stderror in error events [\#296](https://github.com/appscode/stash/pull/296) ([diptadas](https://github.com/diptadas))
- Fixed backup count [\#295](https://github.com/appscode/stash/pull/295) ([diptadas](https://github.com/diptadas))
- Support self-signed ca cert for backends [\#294](https://github.com/appscode/stash/pull/294) ([hossainemruz](https://github.com/hossainemruz))

## [0.6.3](https://github.com/appscode/stash/tree/0.6.3) (2018-01-18)
[Full Changelog](https://github.com/appscode/stash/compare/0.6.2...0.6.3)

**Implemented enhancements:**

- Add Stash Backup Grafana dashboard to monitoring docs [\#285](https://github.com/appscode/stash/issues/285)
- Added Grafana Stash overview dashboard [\#286](https://github.com/appscode/stash/pull/286) ([galexrt](https://github.com/galexrt))

**Fixed bugs:**

- PushGateURL not given to sidecar container [\#283](https://github.com/appscode/stash/issues/283)
- Fix inline volumeSource marshalling for LocalSpec [\#289](https://github.com/appscode/stash/pull/289) ([tamalsaha](https://github.com/tamalsaha))

**Closed issues:**

- Test Failed: Invalid argument error in sidecar container [\#290](https://github.com/appscode/stash/issues/290)

**Merged pull requests:**

- Cleanup headless service [\#292](https://github.com/appscode/stash/pull/292) ([diptadas](https://github.com/diptadas))
- Fixed parsing argument error [\#291](https://github.com/appscode/stash/pull/291) ([diptadas](https://github.com/diptadas))
- Pass through logger flags [\#287](https://github.com/appscode/stash/pull/287) ([tamalsaha](https://github.com/tamalsaha))
- Pass --pushgateway-url for injected containers. [\#284](https://github.com/appscode/stash/pull/284) ([tamalsaha](https://github.com/tamalsaha))

## [0.6.2](https://github.com/appscode/stash/tree/0.6.2) (2018-01-05)
[Full Changelog](https://github.com/appscode/stash/compare/0.6.1...0.6.2)

**Fixed bugs:**

- Created stash-sidecar clusterrole is missing statefulsets permission [\#272](https://github.com/appscode/stash/issues/272)
- Garbage collect s/a and rolebindings for \*Jobs [\#271](https://github.com/appscode/stash/issues/271)
- Fix RBAC roles in chart [\#276](https://github.com/appscode/stash/pull/276) ([tamalsaha](https://github.com/tamalsaha))
- Garbage collect service-accounts and role-bindings for jobs [\#275](https://github.com/appscode/stash/pull/275) ([diptadas](https://github.com/diptadas))
- Fix new restic format in upgrade docs [\#274](https://github.com/appscode/stash/pull/274) ([tamalsaha](https://github.com/tamalsaha))
- Add statefulsets to stash-sidecar ClusterRole creation [\#273](https://github.com/appscode/stash/pull/273) ([galexrt](https://github.com/galexrt))

**Closed issues:**

- Image kubectl not found because of Kubernetes version [\#266](https://github.com/appscode/stash/issues/266)

**Merged pull requests:**

- Prepare docs for 0.6.2 release [\#278](https://github.com/appscode/stash/pull/278) ([tamalsaha](https://github.com/tamalsaha))
- Update Helm chart to use newer 'fullname' template that avoids duplicate \(e.g. 'stash-stash-...'\) resource names [\#277](https://github.com/appscode/stash/pull/277) ([whereisaaron](https://github.com/whereisaaron))
- Reduce operator permissions for service accounts [\#270](https://github.com/appscode/stash/pull/270) ([tamalsaha](https://github.com/tamalsaha))
- Fix formatting of uninstall.md [\#269](https://github.com/appscode/stash/pull/269) ([tamalsaha](https://github.com/tamalsaha))

## [0.6.1](https://github.com/appscode/stash/tree/0.6.1) (2018-01-03)
[Full Changelog](https://github.com/appscode/stash/compare/0.6.0...0.6.1)

**Fixed bugs:**

- Error while running restic [\#256](https://github.com/appscode/stash/issues/256)

**Closed issues:**

- Unable to use non-aws S3 backend [\#226](https://github.com/appscode/stash/issues/226)

**Merged pull requests:**

- Prepare docs for 0.6.1 [\#268](https://github.com/appscode/stash/pull/268) ([tamalsaha](https://github.com/tamalsaha))

## [0.6.0](https://github.com/appscode/stash/tree/0.6.0) (2018-01-03)
[Full Changelog](https://github.com/appscode/stash/compare/0.4.2...0.6.0)

**Implemented enhancements:**

- Feature: Support offline consistent backups [\#225](https://github.com/appscode/stash/issues/225)
- Collect ideas on how to improve recovery process [\#131](https://github.com/appscode/stash/issues/131)
- Use log.LEVEL\(\) instead of fmt.Printf\(\) [\#252](https://github.com/appscode/stash/pull/252) ([galexrt](https://github.com/galexrt))

**Fixed bugs:**

- Fix ConfigMap Name in Leader Election [\#227](https://github.com/appscode/stash/issues/227)
- StatefulSet: Forbidden: pod updates may not add or remove containers [\#191](https://github.com/appscode/stash/issues/191)
- Events are not recording for Recovery [\#219](https://github.com/appscode/stash/issues/219)
- \[0.5.0\] Record backup event on kubernetes failure [\#212](https://github.com/appscode/stash/issues/212)
- Fix kubectl version parsing generation in GKE [\#267](https://github.com/appscode/stash/pull/267) ([tamalsaha](https://github.com/tamalsaha))

**Closed issues:**

- Replace fmt.Print\* with log statements [\#248](https://github.com/appscode/stash/issues/248)
- Dynamically create stash-sidecar ClusterRole in operator [\#220](https://github.com/appscode/stash/issues/220)
- LeaderElection part -2  [\#218](https://github.com/appscode/stash/issues/218)
- Reimplement CheckRecoveryJob using Job watcher [\#216](https://github.com/appscode/stash/issues/216)
- Enable --cache-dir [\#238](https://github.com/appscode/stash/issues/238)
- Upgrade procedure for 0.5.1 -\> 0.6.0 [\#237](https://github.com/appscode/stash/issues/237)
- Test RBAC setup [\#224](https://github.com/appscode/stash/issues/224)
- Record recovery status for individual FileGroup [\#213](https://github.com/appscode/stash/issues/213)
- Periodically run restic check [\#195](https://github.com/appscode/stash/issues/195)
- Handle Deployment etc with replicas \> 1 [\#140](https://github.com/appscode/stash/issues/140)
- Support Backblaze B2 as backend [\#125](https://github.com/appscode/stash/issues/125)
- Turn Stash operator into an Initializer [\#5](https://github.com/appscode/stash/issues/5)

**Merged pull requests:**

- Detect analytics client id using env vars [\#265](https://github.com/appscode/stash/pull/265) ([tamalsaha](https://github.com/tamalsaha))
- Repare docs for 0.6.0 release [\#264](https://github.com/appscode/stash/pull/264) ([tamalsaha](https://github.com/tamalsaha))
- Reorganize docs [\#263](https://github.com/appscode/stash/pull/263) ([tamalsaha](https://github.com/tamalsaha))
- Add support for B2 [\#262](https://github.com/appscode/stash/pull/262) ([tamalsaha](https://github.com/tamalsaha))
- Update restic website link [\#261](https://github.com/appscode/stash/pull/261) ([tamalsaha](https://github.com/tamalsaha))
- Update docs for unified LocalSpec [\#260](https://github.com/appscode/stash/pull/260) ([diptadas](https://github.com/diptadas))
- Unify LocalSpec and RecoveredVolume [\#259](https://github.com/appscode/stash/pull/259) ([diptadas](https://github.com/diptadas))
- Remove restic-dependency from recovery [\#258](https://github.com/appscode/stash/pull/258) ([diptadas](https://github.com/diptadas))
- Update restic version to 0.8.1 [\#257](https://github.com/appscode/stash/pull/257) ([tamalsaha](https://github.com/tamalsaha))
- Use cmp methods from kutil [\#255](https://github.com/appscode/stash/pull/255) ([tamalsaha](https://github.com/tamalsaha))
- Remove TryPatch methods [\#254](https://github.com/appscode/stash/pull/254) ([tamalsaha](https://github.com/tamalsaha))
- Log operator version on start [\#253](https://github.com/appscode/stash/pull/253) ([galexrt](https://github.com/galexrt))
- Use verb type for mutation [\#251](https://github.com/appscode/stash/pull/251) ([tamalsaha](https://github.com/tamalsaha))
- Use CreateOrPatchCronJob from kutil [\#250](https://github.com/appscode/stash/pull/250) ([tamalsaha](https://github.com/tamalsaha))
- Indicate mutation in PATCH helper method return [\#249](https://github.com/appscode/stash/pull/249) ([tamalsaha](https://github.com/tamalsaha))
- Simplify clientID generation for analytics [\#247](https://github.com/appscode/stash/pull/247) ([tamalsaha](https://github.com/tamalsaha))
- Set analytics clientID [\#246](https://github.com/appscode/stash/pull/246) ([tamalsaha](https://github.com/tamalsaha))
- Reorganize docs [\#245](https://github.com/appscode/stash/pull/245) ([tamalsaha](https://github.com/tamalsaha))
- Upgrade procedure for 0.5.1 to 0.6.0 [\#243](https://github.com/appscode/stash/pull/243) ([diptadas](https://github.com/diptadas))
- Fix retentionPolicyName not found error [\#242](https://github.com/appscode/stash/pull/242) ([diptadas](https://github.com/diptadas))
- Enable Restic cahce-dir flag [\#241](https://github.com/appscode/stash/pull/241) ([diptadas](https://github.com/diptadas))
- Use lower case workload.kind in prefix [\#240](https://github.com/appscode/stash/pull/240) ([diptadas](https://github.com/diptadas))
- Use RegisterCRDs helper [\#239](https://github.com/appscode/stash/pull/239) ([tamalsaha](https://github.com/tamalsaha))
- Update docs [\#236](https://github.com/appscode/stash/pull/236) ([diptadas](https://github.com/diptadas))
- Change left\_menu -\> menu\_name [\#235](https://github.com/appscode/stash/pull/235) ([sajibcse68](https://github.com/sajibcse68))
- Revendor dependencies [\#234](https://github.com/appscode/stash/pull/234) ([tamalsaha](https://github.com/tamalsaha))
- Add aliases for README file in front matter [\#233](https://github.com/appscode/stash/pull/233) ([sajibcse68](https://github.com/sajibcse68))
- Update bundles restic to 0.8.0 [\#232](https://github.com/appscode/stash/pull/232) ([tamalsaha](https://github.com/tamalsaha))
- Add Docs Front Matter for 0.5.1 [\#231](https://github.com/appscode/stash/pull/231) ([sajibcse68](https://github.com/sajibcse68))
- Revendor kutil [\#230](https://github.com/appscode/stash/pull/230) ([tamalsaha](https://github.com/tamalsaha))
- Implement offline backup [\#229](https://github.com/appscode/stash/pull/229) ([diptadas](https://github.com/diptadas))
- Fix Configmap Name in Leader Election [\#228](https://github.com/appscode/stash/pull/228) ([diptadas](https://github.com/diptadas))
- Run `restic check` once every 3 days [\#223](https://github.com/appscode/stash/pull/223) ([tamalsaha](https://github.com/tamalsaha))
- Record recovery status for individual FileGroup [\#222](https://github.com/appscode/stash/pull/222) ([tamalsaha](https://github.com/tamalsaha))
- Dynamically create stash-sidecar ClusterRole in operator [\#221](https://github.com/appscode/stash/pull/221) ([tamalsaha](https://github.com/tamalsaha))
- Make stash chart namespaced [\#210](https://github.com/appscode/stash/pull/210) ([tamalsaha](https://github.com/tamalsaha))
- Implement workload initializer in stash operator [\#207](https://github.com/appscode/stash/pull/207) ([diptadas](https://github.com/diptadas))
- Leader election for deployment, replica set and rc [\#206](https://github.com/appscode/stash/pull/206) ([diptadas](https://github.com/diptadas))
- Revise RetentionPolicy in Restic Api [\#205](https://github.com/appscode/stash/pull/205) ([diptadas](https://github.com/diptadas))
- Implement Recovery for Restic Backup [\#202](https://github.com/appscode/stash/pull/202) ([diptadas](https://github.com/diptadas))
