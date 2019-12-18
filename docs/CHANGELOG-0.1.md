---
title: Changelog | Stash
description: Changelog
menu:
  docs_{{ .version }}:
    identifier: changelog-stash-0.1
    name: Changelog-0.1
    parent: welcome
    weight: 10
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: welcome
url: /docs/{{ .version }}/welcome/changelog-0.1/
aliases:
  - /docs/{{ .version }}/CHANGELOG-0.1/
---
# Change Log

## [0.1.0](https://github.com/appscode/stash/tree/0.1.0) (2017-06-27)
**Implemented enhancements:**

- Allow modifying the cron expression [\#21](https://github.com/appscode/stash/issues/21)
- Use RBAC objects for operator. [\#64](https://github.com/appscode/stash/issues/64)
- Support Azure as backup destination [\#35](https://github.com/appscode/stash/issues/35)
- Support GCS as backup destination [\#34](https://github.com/appscode/stash/issues/34)
- Change Destination definition to point to S3 [\#33](https://github.com/appscode/stash/issues/33)
- TODOs [\#22](https://github.com/appscode/stash/issues/22)
- Send performance stats to Prometheus [\#9](https://github.com/appscode/stash/issues/9)

**Fixed bugs:**

- Bubble up errors to the caller. [\#24](https://github.com/appscode/stash/issues/24)
- Fix registration of wrong group [\#39](https://github.com/appscode/stash/pull/39) ([sadlil](https://github.com/sadlil))

**Closed issues:**

- Add /snapshots endpoint in operator [\#81](https://github.com/appscode/stash/issues/81)
- CLI: restic-ctl [\#8](https://github.com/appscode/stash/issues/8)
- Sanitize metric labels [\#68](https://github.com/appscode/stash/issues/68)
- Mount an empty directory to write local files. [\#61](https://github.com/appscode/stash/issues/61)
- Support BackBlaze as backup destination [\#60](https://github.com/appscode/stash/issues/60)
- Support Swift as backup destination [\#59](https://github.com/appscode/stash/issues/59)
- Add e2e tests using Ginkgo [\#57](https://github.com/appscode/stash/issues/57)
- Review analytics [\#55](https://github.com/appscode/stash/issues/55)
- Support updated Kube object versions [\#42](https://github.com/appscode/stash/issues/42)
- Update restic to 0.6.x [\#32](https://github.com/appscode/stash/issues/32)
- Add analytics [\#31](https://github.com/appscode/stash/issues/31)
- HTTP api to exposing restic repository data [\#7](https://github.com/appscode/stash/issues/7)
- Provision new restic repositories [\#6](https://github.com/appscode/stash/issues/6)
- Proposal: Imeplement Restic TPR Resource for Kubernetes [\#1](https://github.com/appscode/stash/issues/1)

**Merged pull requests:**

- Add e2e tests for major cloud providers [\#84](https://github.com/appscode/stash/pull/84) ([tamalsaha](https://github.com/tamalsaha))
- Add /snapshots endpoint in operator [\#82](https://github.com/appscode/stash/pull/82) ([tamalsaha](https://github.com/tamalsaha))
- Handle update conflicts [\#78](https://github.com/appscode/stash/pull/78) ([tamalsaha](https://github.com/tamalsaha))
- Test e2e tests [\#76](https://github.com/appscode/stash/pull/76) ([tamalsaha](https://github.com/tamalsaha))
- Delete old testify tests [\#75](https://github.com/appscode/stash/pull/75) ([tamalsaha](https://github.com/tamalsaha))
- Create a cli wrapper for restic [\#74](https://github.com/appscode/stash/pull/74) ([tamalsaha](https://github.com/tamalsaha))
- Revise EnsureXXXSidecar methods [\#73](https://github.com/appscode/stash/pull/73) ([tamalsaha](https://github.com/tamalsaha))
- Add ginkgo based e2e tests [\#70](https://github.com/appscode/stash/pull/70) ([tamalsaha](https://github.com/tamalsaha))
- Push metrics to Prometheus push gateway [\#67](https://github.com/appscode/stash/pull/67) ([tamalsaha](https://github.com/tamalsaha))
- Use go-sh to execute restic commands [\#63](https://github.com/appscode/stash/pull/63) ([tamalsaha](https://github.com/tamalsaha))
- Add scratchDir & prefixHostname flags [\#62](https://github.com/appscode/stash/pull/62) ([tamalsaha](https://github.com/tamalsaha))
- Support remote backends [\#58](https://github.com/appscode/stash/pull/58) ([tamalsaha](https://github.com/tamalsaha))
- Organize backup code. [\#54](https://github.com/appscode/stash/pull/54) ([tamalsaha](https://github.com/tamalsaha))
- Synchronize scheduler reconfiguration [\#53](https://github.com/appscode/stash/pull/53) ([tamalsaha](https://github.com/tamalsaha))
- Fix unit tests [\#51](https://github.com/appscode/stash/pull/51) ([tamalsaha](https://github.com/tamalsaha))
- Check docker image tag before starting operator [\#45](https://github.com/appscode/stash/pull/45) ([tamalsaha](https://github.com/tamalsaha))
- Expose metrics from operator [\#44](https://github.com/appscode/stash/pull/44) ([tamalsaha](https://github.com/tamalsaha))
- Add analytics [\#41](https://github.com/appscode/stash/pull/41) ([aerokite](https://github.com/aerokite))
- Use V1alpha1SchemeGroupVersion for Restik [\#40](https://github.com/appscode/stash/pull/40) ([aerokite](https://github.com/aerokite))
- Fix status update [\#38](https://github.com/appscode/stash/pull/38) ([saumanbiswas](https://github.com/saumanbiswas))
- Upgrade restic version to 0.6.1 [\#37](https://github.com/appscode/stash/pull/37) ([tamalsaha](https://github.com/tamalsaha))
- Change api version to v1alpha1 [\#30](https://github.com/appscode/stash/pull/30) ([tamalsaha](https://github.com/tamalsaha))
- Rename function and structure [\#29](https://github.com/appscode/stash/pull/29) ([saumanbiswas](https://github.com/saumanbiswas))
- Rename Backup into Restik [\#28](https://github.com/appscode/stash/pull/28) ([saumanbiswas](https://github.com/saumanbiswas))
- Move api from k8s-addons [\#27](https://github.com/appscode/stash/pull/27) ([saumanbiswas](https://github.com/saumanbiswas))
- Bubble up errors to caller [\#26](https://github.com/appscode/stash/pull/26) ([saumanbiswas](https://github.com/saumanbiswas))
- Allow modifying the cron expression [\#25](https://github.com/appscode/stash/pull/25) ([saumanbiswas](https://github.com/saumanbiswas))
- Use unversioned time [\#23](https://github.com/appscode/stash/pull/23) ([tamalsaha](https://github.com/tamalsaha))
- Restik chart [\#20](https://github.com/appscode/stash/pull/20) ([saumanbiswas](https://github.com/saumanbiswas))
- example added [\#19](https://github.com/appscode/stash/pull/19) ([saumanbiswas](https://github.com/saumanbiswas))
- Move restik api and client to k8s-addons [\#18](https://github.com/appscode/stash/pull/18) ([saumanbiswas](https://github.com/saumanbiswas))
- Error print fix [\#17](https://github.com/appscode/stash/pull/17) ([saumanbiswas](https://github.com/saumanbiswas))
- Check group registration [\#16](https://github.com/appscode/stash/pull/16) ([saumanbiswas](https://github.com/saumanbiswas))
- Restik docs [\#15](https://github.com/appscode/stash/pull/15) ([saumanbiswas](https://github.com/saumanbiswas))
- Restik unit test, e2e test [\#14](https://github.com/appscode/stash/pull/14) ([saumanbiswas](https://github.com/saumanbiswas))
- Restik create delete initial implementation [\#12](https://github.com/appscode/stash/pull/12) ([saumanbiswas](https://github.com/saumanbiswas))
- Build docker image [\#11](https://github.com/appscode/stash/pull/11) ([tamalsaha](https://github.com/tamalsaha))
- Clone skeleton from appscode/k3pc [\#10](https://github.com/appscode/stash/pull/10) ([tamalsaha](https://github.com/tamalsaha))
- Fix e2e tests [\#83](https://github.com/appscode/stash/pull/83) ([tamalsaha](https://github.com/tamalsaha))
- Mount scratchDir with operator [\#80](https://github.com/appscode/stash/pull/80) ([tamalsaha](https://github.com/tamalsaha))
- Fix scheduler  [\#79](https://github.com/appscode/stash/pull/79) ([tamalsaha](https://github.com/tamalsaha))
- Create RBAC objects for operator [\#69](https://github.com/appscode/stash/pull/69) ([tamalsaha](https://github.com/tamalsaha))
- Mount labels using Downward api [\#66](https://github.com/appscode/stash/pull/66) ([tamalsaha](https://github.com/tamalsaha))
- Vendor go-sh dependency [\#65](https://github.com/appscode/stash/pull/65) ([tamalsaha](https://github.com/tamalsaha))
- Update e2e tests [\#52](https://github.com/appscode/stash/pull/52) ([tamalsaha](https://github.com/tamalsaha))
- Run watchers for preferred api group version kind [\#50](https://github.com/appscode/stash/pull/50) ([tamalsaha](https://github.com/tamalsaha))
- Build restic from source by default [\#49](https://github.com/appscode/stash/pull/49) ([tamalsaha](https://github.com/tamalsaha))
- Watch individual object types. [\#48](https://github.com/appscode/stash/pull/48) ([tamalsaha](https://github.com/tamalsaha))
- Various code cleanup [\#47](https://github.com/appscode/stash/pull/47) ([tamalsaha](https://github.com/tamalsaha))
- Reorganize cron controller [\#46](https://github.com/appscode/stash/pull/46) ([tamalsaha](https://github.com/tamalsaha))
- Run push gateway as a side-car for restik operator. [\#43](https://github.com/appscode/stash/pull/43) ([tamalsaha](https://github.com/tamalsaha))
- Use client-go [\#36](https://github.com/appscode/stash/pull/36) ([tamalsaha](https://github.com/tamalsaha))
