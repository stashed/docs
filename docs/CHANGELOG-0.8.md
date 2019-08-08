---
title: Changelog | Stash
description: Changelog
menu:
  product_stash_0.8.3:
    identifier: changelog-stash-0.8
    name: Changelog
    parent: welcome
    weight: 10
product_name: stash
menu_name: product_stash_0.8.3
section_menu_id: welcome
url: /products/stash/0.8.3/welcome/changelog/
aliases:
  - /products/stash/0.8.3/CHANGELOG/
---

# Change Log

## [0.8.3](https://github.com/appscode/stash/tree/0.8.3) (2019-02-19)
[Full Changelog](https://github.com/appscode/stash/compare/0.8.2...0.8.3)

**Closed issues:**

- Instance label in pushgateway is deprecated - Not use HostnameGroupingKey\(\) [\#670](https://github.com/appscode/stash/issues/670)
- Design review [\#616](https://github.com/appscode/stash/issues/616)
- Rename Restic to Backup [\#320](https://github.com/appscode/stash/issues/320)

**Merged pull requests:**

- Update dependencies [\#681](https://github.com/appscode/stash/pull/681) ([tamalsaha](https://github.com/tamalsaha))
- Don't add hostname label to Prometheus metrics. [\#680](https://github.com/appscode/stash/pull/680) ([tamalsaha](https://github.com/tamalsaha))
- Pass pod annotation to deployment [\#679](https://github.com/appscode/stash/pull/679) ([tamalsaha](https://github.com/tamalsaha))
- Fix the case for deploying using MINGW64 for windows [\#678](https://github.com/appscode/stash/pull/678) ([tamalsaha](https://github.com/tamalsaha))
- Use onessl 0.10.0 [\#677](https://github.com/appscode/stash/pull/677) ([tamalsaha](https://github.com/tamalsaha))
- s/rook/azure/ in possible copy/paste error. [\#669](https://github.com/appscode/stash/pull/669) ([lastcoolnameleft](https://github.com/lastcoolnameleft))
- Fix builtin monitoring doc [\#668](https://github.com/appscode/stash/pull/668) ([hossainemruz](https://github.com/hossainemruz))
- Don't use priority class when operator namespace is not kube-system [\#666](https://github.com/appscode/stash/pull/666) ([hossainemruz](https://github.com/hossainemruz))
- Separate type definitions into individual files [\#646](https://github.com/appscode/stash/pull/646) ([hossainemruz](https://github.com/hossainemruz))

## [0.8.2](https://github.com/appscode/stash/tree/0.8.2) (2019-01-02)
[Full Changelog](https://github.com/appscode/stash/compare/0.8.1...0.8.2)

**Fixed bugs:**

- Fix typo in installer [\#638](https://github.com/appscode/stash/pull/638) ([tamalsaha](https://github.com/tamalsaha))

**Closed issues:**

- Backend configuration doc link broken [\#640](https://github.com/appscode/stash/issues/640)
- Architecture questions [\#635](https://github.com/appscode/stash/issues/635)
- Restart operator pod on update [\#611](https://github.com/appscode/stash/issues/611)

**Merged pull requests:**

- Prepare docs for 0.8.2 release [\#644](https://github.com/appscode/stash/pull/644) ([tamalsaha](https://github.com/tamalsaha))
- Update copyright notice for 2019 [\#643](https://github.com/appscode/stash/pull/643) ([tamalsaha](https://github.com/tamalsaha))
- Use stash.labels template in chart [\#642](https://github.com/appscode/stash/pull/642) ([tamalsaha](https://github.com/tamalsaha))
- Fixed broken link for bakend [\#641](https://github.com/appscode/stash/pull/641) ([hossainemruz](https://github.com/hossainemruz))
-  Only mount stash apiserver `tls.crt` into Prometheus [\#639](https://github.com/appscode/stash/pull/639) ([hossainemruz](https://github.com/hossainemruz))
- Fix monitoring in helm + update doc to match with third-party-tools tutorial [\#637](https://github.com/appscode/stash/pull/637) ([hossainemruz](https://github.com/hossainemruz))
- Add certificate health checker [\#636](https://github.com/appscode/stash/pull/636) ([tamalsaha](https://github.com/tamalsaha))
- Update chart readme [\#632](https://github.com/appscode/stash/pull/632) ([tamalsaha](https://github.com/tamalsaha))
- Update webhook error message format for Kubernetes 1.13+ [\#631](https://github.com/appscode/stash/pull/631) ([tamalsaha](https://github.com/tamalsaha))
- Fix typos [\#630](https://github.com/appscode/stash/pull/630) ([tamalsaha](https://github.com/tamalsaha))

## [0.8.1](https://github.com/appscode/stash/tree/0.8.1) (2018-12-09)
[Full Changelog](https://github.com/appscode/stash/compare/0.8.0...0.8.1)

**Fixed bugs:**

- Stash chart is throwing error [\#627](https://github.com/appscode/stash/issues/627)

**Merged pull requests:**

- Prepare docs for 0.8.1 release [\#629](https://github.com/appscode/stash/pull/629) ([tamalsaha](https://github.com/tamalsaha))
- Add missing validator for respository resource in chart [\#628](https://github.com/appscode/stash/pull/628) ([tamalsaha](https://github.com/tamalsaha))

## [0.8.0](https://github.com/appscode/stash/tree/0.8.0) (2018-12-08)
[Full Changelog](https://github.com/appscode/stash/compare/0.7.0...0.8.0)

**Fixed bugs:**

- Delete snapshot command does not check for snapshot's existence [\#549](https://github.com/appscode/stash/issues/549)
- Backup not triggered  [\#461](https://github.com/appscode/stash/issues/461)
- Service name hardcoded in func PushgatewayURL, no metrics available [\#596](https://github.com/appscode/stash/issues/596)
- Fix extended apiserver issues with Kubernetes 1.11 [\#536](https://github.com/appscode/stash/pull/536) ([tamalsaha](https://github.com/tamalsaha))
- Correctly handle ignored openapi prefixes [\#533](https://github.com/appscode/stash/pull/533) ([tamalsaha](https://github.com/tamalsaha))
- Add rbac permissions for snapshots [\#531](https://github.com/appscode/stash/pull/531) ([tamalsaha](https://github.com/tamalsaha))

**Closed issues:**

- Problem creating backups [\#588](https://github.com/appscode/stash/issues/588)
- Issue while installing stash kubernetes 1.11.2 [\#587](https://github.com/appscode/stash/issues/587)
- Hardcoded cleaner kubectl image in Helm chart [\#583](https://github.com/appscode/stash/issues/583)
- Deployed latest helm chart and getting error during sidecar creation [\#556](https://github.com/appscode/stash/issues/556)
- Minio backup fails: 'net/http: invalid header field value "..." for key Authorization' [\#547](https://github.com/appscode/stash/issues/547)
- Repository overwrite for different workload with same name in different namespace [\#539](https://github.com/appscode/stash/issues/539)
- Unexpected behavior in offline backup [\#535](https://github.com/appscode/stash/issues/535)
- Offline backup not working \(permissions\) [\#534](https://github.com/appscode/stash/issues/534)
- Support node selector for recovery job [\#515](https://github.com/appscode/stash/issues/515)
- Clarify that hostpaths are just example [\#514](https://github.com/appscode/stash/issues/514)
- Internal error occurred: failed calling admission webhook "deployment.admission.stash.appscode.com": the server could not find the requested resource [\#510](https://github.com/appscode/stash/issues/510)
- GKE page missing front matter [\#505](https://github.com/appscode/stash/issues/505)
- Could not list snapshots on kubernetes 1.8.4 [\#503](https://github.com/appscode/stash/issues/503)
- Admission webhook denied rquest: Rolebindings not found [\#501](https://github.com/appscode/stash/issues/501)
- Incorrect image name for sidecar container [\#485](https://github.com/appscode/stash/issues/485)
- Using Stash with TLS secured Minio Server Can't succeed [\#478](https://github.com/appscode/stash/issues/478)
- Add cluster name in repo path [\#374](https://github.com/appscode/stash/issues/374)
- Stash don't pass `nodeSelector` from Recovery crd to recovery Job. [\#617](https://github.com/appscode/stash/issues/617)
- Recovery task is not working [\#613](https://github.com/appscode/stash/issues/613)
- Permissions problem with the Helm chart in master branch [\#592](https://github.com/appscode/stash/issues/592)
- Add Prometheus config sample for pushgateway [\#582](https://github.com/appscode/stash/issues/582)
- Handle security context [\#566](https://github.com/appscode/stash/issues/566)
- \[Request\] Add backup details to "kubectl get" for stash objects on K8s 1.11 [\#525](https://github.com/appscode/stash/issues/525)
- matchLabels on Restic CRD not working when using hyphens in keys [\#521](https://github.com/appscode/stash/issues/521)

**Merged pull requests:**

- Prepare docs for 0.8.0 release [\#626](https://github.com/appscode/stash/pull/626) ([tamalsaha](https://github.com/tamalsaha))
- Update docs \(Minio, Rook, NFS\) [\#625](https://github.com/appscode/stash/pull/625) ([hossainemruz](https://github.com/hossainemruz))
- Use flags.DumpAll to dump flags [\#624](https://github.com/appscode/stash/pull/624) ([tamalsaha](https://github.com/tamalsaha))
- Set periodic analytics [\#623](https://github.com/appscode/stash/pull/623) ([tamalsaha](https://github.com/tamalsaha))
- Fix e2e test [\#622](https://github.com/appscode/stash/pull/622) ([hossainemruz](https://github.com/hossainemruz))
- Recovery Job: Use nodeName for DaemonSet and nodeSelector for other workloads [\#620](https://github.com/appscode/stash/pull/620) ([hossainemruz](https://github.com/hossainemruz))
- Pass --enable-\*\*\*-webhook flags to operator [\#619](https://github.com/appscode/stash/pull/619) ([tamalsaha](https://github.com/tamalsaha))
- Add validation webhook xray [\#618](https://github.com/appscode/stash/pull/618) ([tamalsaha](https://github.com/tamalsaha))
- Use dynamic pushgateway url [\#614](https://github.com/appscode/stash/pull/614) ([hossainemruz](https://github.com/hossainemruz))
- Add docs for AKS and EKS [\#609](https://github.com/appscode/stash/pull/609) ([hossainemruz](https://github.com/hossainemruz))
- Improve monitoring facility [\#606](https://github.com/appscode/stash/pull/606) ([hossainemruz](https://github.com/hossainemruz))
- Pass image pull secrets for cleaner job in chart [\#598](https://github.com/appscode/stash/pull/598) ([tamalsaha](https://github.com/tamalsaha))
- Update kubernetes client libraries to 1.12.0 [\#597](https://github.com/appscode/stash/pull/597) ([tamalsaha](https://github.com/tamalsaha))
- Support LogLevel in chart [\#594](https://github.com/appscode/stash/pull/594) ([tamalsaha](https://github.com/tamalsaha))
- Check if Kubernetes version is supported before running operator [\#593](https://github.com/appscode/stash/pull/593) ([tamalsaha](https://github.com/tamalsaha))
- Enable webhooks by default in chart [\#591](https://github.com/appscode/stash/pull/591) ([tamalsaha](https://github.com/tamalsaha))
- Update chart readme for cleaner values [\#590](https://github.com/appscode/stash/pull/590) ([tamalsaha](https://github.com/tamalsaha))
- Fix \#583 and pushgateway version [\#584](https://github.com/appscode/stash/pull/584) ([sebastien-prudhomme](https://github.com/sebastien-prudhomme))
- Use --pull flag with docker build [\#581](https://github.com/appscode/stash/pull/581) ([tamalsaha](https://github.com/tamalsaha))
- Use kubernetes-1.11.3 [\#578](https://github.com/appscode/stash/pull/578) ([tamalsaha](https://github.com/tamalsaha))
- Update CertStore [\#576](https://github.com/appscode/stash/pull/576) ([tamalsaha](https://github.com/tamalsaha))
- Use apps/v1 apigroup in installer scripts [\#574](https://github.com/appscode/stash/pull/574) ([tamalsaha](https://github.com/tamalsaha))
- Support pod annotations in chart [\#573](https://github.com/appscode/stash/pull/573) ([tamalsaha](https://github.com/tamalsaha))
- Set serviceAccount for clearner job [\#572](https://github.com/appscode/stash/pull/572) ([tamalsaha](https://github.com/tamalsaha))
- Set SecurityContext for stash sidecar [\#570](https://github.com/appscode/stash/pull/570) ([tamalsaha](https://github.com/tamalsaha))
- Cleanup webhooks when chart is deleted [\#569](https://github.com/appscode/stash/pull/569) ([tamalsaha](https://github.com/tamalsaha))
- Use IntHash as status.observedGeneration [\#568](https://github.com/appscode/stash/pull/568) ([tamalsaha](https://github.com/tamalsaha))
- fix success list in grafana dashboard [\#567](https://github.com/appscode/stash/pull/567) ([unteem](https://github.com/unteem))
- Update pipeline [\#565](https://github.com/appscode/stash/pull/565) ([tahsinrahman](https://github.com/tahsinrahman))
- Add observedGenerationHash field [\#564](https://github.com/appscode/stash/pull/564) ([tamalsaha](https://github.com/tamalsaha))
- fix uninstall for concourse [\#563](https://github.com/appscode/stash/pull/563) ([tahsinrahman](https://github.com/tahsinrahman))
- Fix chart values file [\#562](https://github.com/appscode/stash/pull/562) ([tamalsaha](https://github.com/tamalsaha))
- Improve Helm chart options [\#561](https://github.com/appscode/stash/pull/561) ([tamalsaha](https://github.com/tamalsaha))
- Refactor concourse scripts [\#554](https://github.com/appscode/stash/pull/554) ([tahsinrahman](https://github.com/tahsinrahman))
- Add AlreadyObserved methods [\#553](https://github.com/appscode/stash/pull/553) ([tamalsaha](https://github.com/tamalsaha))
- Add categories support to crds [\#552](https://github.com/appscode/stash/pull/552) ([tamalsaha](https://github.com/tamalsaha))
- Improve logging [\#551](https://github.com/appscode/stash/pull/551) ([hossainemruz](https://github.com/hossainemruz))
- Improve doc [\#550](https://github.com/appscode/stash/pull/550) ([hossainemruz](https://github.com/hossainemruz))
- Check for snapshot existence before delete [\#548](https://github.com/appscode/stash/pull/548) ([hossainemruz](https://github.com/hossainemruz))
- Enable status sub resource for crd yamls [\#546](https://github.com/appscode/stash/pull/546) ([tamalsaha](https://github.com/tamalsaha))
- Retry UpdateStatus calls [\#544](https://github.com/appscode/stash/pull/544) ([tamalsaha](https://github.com/tamalsaha))
- Move crds to api folder [\#543](https://github.com/appscode/stash/pull/543) ([tamalsaha](https://github.com/tamalsaha))
- Revendor objectstore api [\#542](https://github.com/appscode/stash/pull/542) ([tamalsaha](https://github.com/tamalsaha))
- Use kmodules.xyz/objectstore-api [\#541](https://github.com/appscode/stash/pull/541) ([tamalsaha](https://github.com/tamalsaha))
- Fix offline backup [\#537](https://github.com/appscode/stash/pull/537) ([hossainemruz](https://github.com/hossainemruz))
- Rename dev script [\#532](https://github.com/appscode/stash/pull/532) ([tamalsaha](https://github.com/tamalsaha))
- Use version and additional columns for crds [\#530](https://github.com/appscode/stash/pull/530) ([tamalsaha](https://github.com/tamalsaha))
- Don't add admission/v1beta1 group as a prioritized version [\#529](https://github.com/appscode/stash/pull/529) ([tamalsaha](https://github.com/tamalsaha))
- Update client-go to v8.0.0 [\#528](https://github.com/appscode/stash/pull/528) ([tamalsaha](https://github.com/tamalsaha))
- Format shell scripts [\#526](https://github.com/appscode/stash/pull/526) ([tamalsaha](https://github.com/tamalsaha))
- Enable status subresource for crds [\#524](https://github.com/appscode/stash/pull/524) ([tamalsaha](https://github.com/tamalsaha))
- Upgrade to restic 0.9.1 [\#522](https://github.com/appscode/stash/pull/522) ([tamalsaha](https://github.com/tamalsaha))
- Move openapi-spec to api folder [\#513](https://github.com/appscode/stash/pull/513) ([tamalsaha](https://github.com/tamalsaha))
- Deploy operator in kube-system namespace via Helm [\#511](https://github.com/appscode/stash/pull/511) ([tamalsaha](https://github.com/tamalsaha))
- Add togglable tabs for Installation: Script & Helm [\#509](https://github.com/appscode/stash/pull/509) ([sajibcse68](https://github.com/sajibcse68))
- Revendor dependencies [\#508](https://github.com/appscode/stash/pull/508) ([tamalsaha](https://github.com/tamalsaha))
- Added front matter [\#507](https://github.com/appscode/stash/pull/507) ([hossainemruz](https://github.com/hossainemruz))
- Improve installer [\#504](https://github.com/appscode/stash/pull/504) ([tamalsaha](https://github.com/tamalsaha))
- Use apps/v1 apigroup [\#555](https://github.com/appscode/stash/pull/555) ([tamalsaha](https://github.com/tamalsaha))
- Update chart installation instruction for Kubernetes 1.11 [\#527](https://github.com/appscode/stash/pull/527) ([tamalsaha](https://github.com/tamalsaha))
- Remove status from crd.yaml [\#523](https://github.com/appscode/stash/pull/523) ([tamalsaha](https://github.com/tamalsaha))
- Upgrade to prom/pushgateway:v0.5.2 [\#519](https://github.com/appscode/stash/pull/519) ([tamalsaha](https://github.com/tamalsaha))
- Remove ops-address port [\#518](https://github.com/appscode/stash/pull/518) ([tamalsaha](https://github.com/tamalsaha))
- Set cpu limits to 100m [\#517](https://github.com/appscode/stash/pull/517) ([tamalsaha](https://github.com/tamalsaha))
- Support node selector for recovery job [\#516](https://github.com/appscode/stash/pull/516) ([tamalsaha](https://github.com/tamalsaha))
- Fix concourse test [\#496](https://github.com/appscode/stash/pull/496) ([hossainemruz](https://github.com/hossainemruz))
