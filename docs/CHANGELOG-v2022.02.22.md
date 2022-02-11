---
title: Changelog | Stash
description: Changelog
menu:
  docs_{{.version}}:
    identifier: changelog-stash-v2022.02.22
    name: Changelog-v2022.02.22
    parent: welcome
    weight: 20220222
product_name: stash
menu_name: docs_{{.version}}
section_menu_id: welcome
url: /docs/{{.version}}/welcome/changelog-v2022.02.22/
aliases:
  - /docs/{{.version}}/CHANGELOG-v2022.02.22/
---

# Stash v2022.02.22 (2022-02-11)


## [stashed/apimachinery](https://github.com/stashed/apimachinery)

### [v0.18.0](https://github.com/stashed/apimachinery/releases/tag/v0.18.0)

- [79d844fb](https://github.com/stashed/apimachinery/commit/79d844fb) Refactor restore invoker status updater logic (#152)
- [49e0fc85](https://github.com/stashed/apimachinery/commit/49e0fc85) Update SiteInfo (#151)
- [7c80de5c](https://github.com/stashed/apimachinery/commit/7c80de5c) Publish GenericResource (#150)
- [0906fab7](https://github.com/stashed/apimachinery/commit/0906fab7) Add stash-crd-installer image to install CRDs on helm pre-install hook (#145)
- [9c528743](https://github.com/stashed/apimachinery/commit/9c528743) Add phase field to backupconfiguration status. (#144)
- [3e17dd8d](https://github.com/stashed/apimachinery/commit/3e17dd8d) Refactor `metrics.go` file + Add `pushgatewayURL` singletone (#149)
- [553a63a8](https://github.com/stashed/apimachinery/commit/553a63a8) Add `GetCaPath()` method (#148)
- [3f6beb6d](https://github.com/stashed/apimachinery/commit/3f6beb6d) Update repository config (#147)
- [d7bf57bf](https://github.com/stashed/apimachinery/commit/d7bf57bf) Refactor invoker package (#146)
- [7da4b19c](https://github.com/stashed/apimachinery/commit/7da4b19c) Add method to create ResticWrapper from existing shell (#143)
- [68c89614](https://github.com/stashed/apimachinery/commit/68c89614) Show Snapshot ID in kubectl list command (#141)
- [8017cd43](https://github.com/stashed/apimachinery/commit/8017cd43) Review ui apis (#142)
- [d19cc472](https://github.com/stashed/apimachinery/commit/d19cc472) Modify BackupOverview API (#140)
- [c60601b5](https://github.com/stashed/apimachinery/commit/c60601b5) Add helper method for UsagePolicy (#139)
- [bc75d16a](https://github.com/stashed/apimachinery/commit/bc75d16a) Add support for cross-namespace repository (#136)
- [e845509f](https://github.com/stashed/apimachinery/commit/e845509f) Add ui types (#137)
- [6402d026](https://github.com/stashed/apimachinery/commit/6402d026) Allow cross namespace referencing for Repository + Cleanup old APIs (#135)
- [e914179d](https://github.com/stashed/apimachinery/commit/e914179d) Update repository config (#134)



## [stashed/cli](https://github.com/stashed/cli)

### [v0.18.0](https://github.com/stashed/cli/releases/tag/v0.18.0)

- [fdf799c](https://github.com/stashed/cli/commit/fdf799c) Prepare for release v0.18.0 (#155)
- [71d2142](https://github.com/stashed/cli/commit/71d2142) Update SiteInfo (#153)
- [2363171](https://github.com/stashed/cli/commit/2363171) Add `pause` and  `resume` command (#152)
- [e6d907b](https://github.com/stashed/cli/commit/e6d907b) Publish GenericResource (#151)
- [c90bfcc](https://github.com/stashed/cli/commit/c90bfcc) Use stashed/restic image for darwin/arm64 support (#150)
- [b42d851](https://github.com/stashed/cli/commit/b42d851) Release cli for darwin/arm64 (#149)
- [34e30bd](https://github.com/stashed/cli/commit/34e30bd) Fix stash broken cli (#148)



## [stashed/elasticsearch](https://github.com/stashed/elasticsearch)

### [5.6.4-v15](https://github.com/stashed/elasticsearch/releases/tag/5.6.4-v15)

- [5eafa076](https://github.com/stashed/elasticsearch/commit/5eafa076) Prepare for release 5.6.4-v15 (#1097)
- [717701f3](https://github.com/stashed/elasticsearch/commit/717701f3) [cherry-pick] Update SiteInfo (#1089) (#1090)
- [2dacaf65](https://github.com/stashed/elasticsearch/commit/2dacaf65) [cherry-pick] Publish GenericResource (#1079) (#1080)
- [3097dc28](https://github.com/stashed/elasticsearch/commit/3097dc28) Add support for backup and restore from different namespace (#1069) (#1070)


### [6.2.4-v15](https://github.com/stashed/elasticsearch/releases/tag/6.2.4-v15)

- [7a7c194d](https://github.com/stashed/elasticsearch/commit/7a7c194d) Prepare for release 6.2.4-v15 (#1098)
- [6c41704d](https://github.com/stashed/elasticsearch/commit/6c41704d) [cherry-pick] Update SiteInfo (#1089) (#1091)
- [398d08d4](https://github.com/stashed/elasticsearch/commit/398d08d4) [cherry-pick] Publish GenericResource (#1079) (#1081)
- [5b9a3530](https://github.com/stashed/elasticsearch/commit/5b9a3530) Add support for backup and restore from different namespace (#1069) (#1071)


### [6.3.0-v15](https://github.com/stashed/elasticsearch/releases/tag/6.3.0-v15)

- [811094fc](https://github.com/stashed/elasticsearch/commit/811094fc) Prepare for release 6.3.0-v15 (#1099)
- [7bc3cbe5](https://github.com/stashed/elasticsearch/commit/7bc3cbe5) [cherry-pick] Update SiteInfo (#1089) (#1092)
- [6027c9da](https://github.com/stashed/elasticsearch/commit/6027c9da) [cherry-pick] Publish GenericResource (#1079) (#1082)
- [8b486870](https://github.com/stashed/elasticsearch/commit/8b486870) Add support for backup and restore from different namespace (#1069) (#1072)


### [6.4.0-v15](https://github.com/stashed/elasticsearch/releases/tag/6.4.0-v15)

- [c351d557](https://github.com/stashed/elasticsearch/commit/c351d557) Prepare for release 6.4.0-v15 (#1100)
- [12cad4fb](https://github.com/stashed/elasticsearch/commit/12cad4fb) [cherry-pick] Publish GenericResource (#1079) (#1083)
- [ce67978c](https://github.com/stashed/elasticsearch/commit/ce67978c) Add support for backup and restore from different namespace (#1069) (#1073)


### [6.5.3-v15](https://github.com/stashed/elasticsearch/releases/tag/6.5.3-v15)

- [885a5c59](https://github.com/stashed/elasticsearch/commit/885a5c59) Prepare for release 6.5.3-v15 (#1101)
- [0012c4c5](https://github.com/stashed/elasticsearch/commit/0012c4c5) [cherry-pick] Publish GenericResource (#1079) (#1084)
- [268a0310](https://github.com/stashed/elasticsearch/commit/268a0310) Add support for backup and restore from different namespace (#1069) (#1074)


### [6.8.0-v15](https://github.com/stashed/elasticsearch/releases/tag/6.8.0-v15)

- [702ac2f8](https://github.com/stashed/elasticsearch/commit/702ac2f8) Prepare for release 6.8.0-v15 (#1102)
- [8c2a442f](https://github.com/stashed/elasticsearch/commit/8c2a442f) [cherry-pick] Update SiteInfo (#1089) (#1093)
- [9d595583](https://github.com/stashed/elasticsearch/commit/9d595583) [cherry-pick] Publish GenericResource (#1079) (#1085)
- [43fb310d](https://github.com/stashed/elasticsearch/commit/43fb310d) Add support for backup and restore from different namespace (#1069) (#1075)


### [7.2.0-v15](https://github.com/stashed/elasticsearch/releases/tag/7.2.0-v15)

- [b78c499d](https://github.com/stashed/elasticsearch/commit/b78c499d) Prepare for release 7.2.0-v15 (#1104)
- [666a162a](https://github.com/stashed/elasticsearch/commit/666a162a) [cherry-pick] Update SiteInfo (#1089) (#1095)
- [7c39ac5c](https://github.com/stashed/elasticsearch/commit/7c39ac5c) [cherry-pick] Publish GenericResource (#1079) (#1087)
- [e813c925](https://github.com/stashed/elasticsearch/commit/e813c925) Add support for backup and restore from different namespace (#1069) (#1077)


### [7.3.2-v15](https://github.com/stashed/elasticsearch/releases/tag/7.3.2-v15)

- [ab9c11e5](https://github.com/stashed/elasticsearch/commit/ab9c11e5) Prepare for release 7.3.2-v15 (#1105)
- [d49adbf5](https://github.com/stashed/elasticsearch/commit/d49adbf5) [cherry-pick] Update SiteInfo (#1089) (#1096)
- [41b2b371](https://github.com/stashed/elasticsearch/commit/41b2b371) [cherry-pick] Publish GenericResource (#1079) (#1088)
- [90c57e9b](https://github.com/stashed/elasticsearch/commit/90c57e9b) Add support for backup and restore from different namespace (#1069) (#1078)
- [6ecd2ef3](https://github.com/stashed/elasticsearch/commit/6ecd2ef3) Prepare for release 7.3.2-v14 (#1067)


### [7.14.0-v1](https://github.com/stashed/elasticsearch/releases/tag/7.14.0-v1)

- [c2388a12](https://github.com/stashed/elasticsearch/commit/c2388a12) Prepare for release 7.14.0-v1 (#1103)
- [2b05ab4a](https://github.com/stashed/elasticsearch/commit/2b05ab4a) [cherry-pick] Update SiteInfo (#1089) (#1094)
- [b62eb6a1](https://github.com/stashed/elasticsearch/commit/b62eb6a1) [cherry-pick] Publish GenericResource (#1079) (#1086)
- [e6f4b52d](https://github.com/stashed/elasticsearch/commit/e6f4b52d) [cherry-pick] Add support for backup and restore from different namespace (#1069) (#1076)



## [stashed/enterprise](https://github.com/stashed/enterprise)

### [v0.18.0](https://github.com/stashed/enterprise/releases/tag/v0.18.0)

- [fcf0f4f5](https://github.com/stashed/enterprise/commit/fcf0f4f5) Prepare for release v0.18.0 (#147)
- [55a9a92d](https://github.com/stashed/enterprise/commit/55a9a92d) Update UID generation for GenericResource (#146)
- [cb30a32f](https://github.com/stashed/enterprise/commit/cb30a32f) Refactor restore process status handling + add cross namespace tests (#145)
- [3f50cb35](https://github.com/stashed/enterprise/commit/3f50cb35) Update SiteInfo (#144)
- [ac18da55](https://github.com/stashed/enterprise/commit/ac18da55) Publish GenericResource (#143)
- [efcb6727](https://github.com/stashed/enterprise/commit/efcb6727) Expose original UID for audit events
- [301f2a56](https://github.com/stashed/enterprise/commit/301f2a56) Always set type meta for GenericResource
- [ccfc9ccf](https://github.com/stashed/enterprise/commit/ccfc9ccf) Publish GenericResource (#142)
- [477401a6](https://github.com/stashed/enterprise/commit/477401a6) Don't register CRDs from operator + allow passing crd-installer tag (#138)
- [fc98dd68](https://github.com/stashed/enterprise/commit/fc98dd68) Add validation webhooks for double opt-in check (#137)
- [1fb0d151](https://github.com/stashed/enterprise/commit/1fb0d151) Always keep the last completed backup (when `backupHistoryLimit > 0`) even if outside of history limit (#141)
- [db7fdbf3](https://github.com/stashed/enterprise/commit/db7fdbf3) Add support for custom Pushgateway (#140)
- [83940a05](https://github.com/stashed/enterprise/commit/83940a05) Use refactored invoker package (#139)
- [7a2d8610](https://github.com/stashed/enterprise/commit/7a2d8610) Cleanup legacy code + Support cross namespace Repository reference (#136)



## [stashed/etcd](https://github.com/stashed/etcd)

### [3.5.0-v2](https://github.com/stashed/etcd/releases/tag/3.5.0-v2)

- [5127c53](https://github.com/stashed/etcd/commit/5127c53) Prepare for release 3.5.0-v2 (#26)
- [f1dd201](https://github.com/stashed/etcd/commit/f1dd201) [cherry-pick] Update SiteInfo (#24) (#25)
- [b6aa1c1](https://github.com/stashed/etcd/commit/b6aa1c1) [cherry-pick] Publish GenericResource (#22) (#23)
- [b14a960](https://github.com/stashed/etcd/commit/b14a960) Add support for backup and restore from different namespace (#20) (#21)



## [stashed/installer](https://github.com/stashed/installer)

### [v2022.02.22](https://github.com/stashed/installer/releases/tag/v2022.02.22)

- [29686b4](https://github.com/stashed/installer/commit/29686b4) Prepare for release v2022.02.22 (#234)
- [31da5db](https://github.com/stashed/installer/commit/31da5db) Update chart version is stash-opscenter chart
- [a341dd1](https://github.com/stashed/installer/commit/a341dd1) Update repository config (#233)
- [c417a24](https://github.com/stashed/installer/commit/c417a24) Update stash-ui-server image tag using makefile
- [0b5b92a](https://github.com/stashed/installer/commit/0b5b92a) Make operator ready for arm64 nodes (#232)
- [fc96a55](https://github.com/stashed/installer/commit/fc96a55) Publish GenericResource (#231)
- [ea1d95f](https://github.com/stashed/installer/commit/ea1d95f) Add crd-installer job (#227)
- [e9b9c1f](https://github.com/stashed/installer/commit/e9b9c1f) Add validating webhook for BackupConfiguration in community chart (#230)
- [c0e15c6](https://github.com/stashed/installer/commit/c0e15c6) Add support for custom Pushgateway (#229)
- [4ad1c1f](https://github.com/stashed/installer/commit/4ad1c1f) Update repository config (#228)
- [5334b19](https://github.com/stashed/installer/commit/5334b19) Update catalogs chart to support backup and restore from different namespace (#226)
- [419b7c3](https://github.com/stashed/installer/commit/419b7c3) Add permission for nonResourceURLs
- [3a5f578](https://github.com/stashed/installer/commit/3a5f578) Updated charts after adding some validator webhook. (#222)
- [774a76b](https://github.com/stashed/installer/commit/774a76b) Correctly generate grafana dashboard name (#225)
- [73fde3c](https://github.com/stashed/installer/commit/73fde3c) Always pull ui-server (#224)
- [f3d6a22](https://github.com/stashed/installer/commit/f3d6a22) Add permission for AppBinding & KubeDB resources in ui server chart (#223)
- [cdeac58](https://github.com/stashed/installer/commit/cdeac58) Add stash-ui-server chart (#220)
- [bfddb33](https://github.com/stashed/installer/commit/bfddb33) Update repository config (#219)
- [dbce5b5](https://github.com/stashed/installer/commit/dbce5b5) Add k8s.io/group label to grafana dashboards
- [3f07f11](https://github.com/stashed/installer/commit/3f07f11) Add Grafana Dashboards (#218)



## [stashed/mariadb](https://github.com/stashed/mariadb)

### [10.5.8-v8](https://github.com/stashed/mariadb/releases/tag/10.5.8-v8)

- [3241977](https://github.com/stashed/mariadb/commit/3241977) Prepare for release 10.5.8-v8 (#167)
- [2c34c07](https://github.com/stashed/mariadb/commit/2c34c07) [cherry-pick] Update SiteInfo (#165) (#166)
- [39d3273](https://github.com/stashed/mariadb/commit/39d3273) [cherry-pick] Publish GenericResource (#163) (#164)
- [4ad02b8](https://github.com/stashed/mariadb/commit/4ad02b8) Add support for backup and restore from different namespace (#161) (#162)



## [stashed/mongodb](https://github.com/stashed/mongodb)

### [3.4.17-v14](https://github.com/stashed/mongodb/releases/tag/3.4.17-v14)

- [7fdea5ac](https://github.com/stashed/mongodb/commit/7fdea5ac) Prepare for release 3.4.17-v14 (#1410)
- [5770d137](https://github.com/stashed/mongodb/commit/5770d137) [cherry-pick] Update SiteInfo (#1397) (#1398)
- [c97fb271](https://github.com/stashed/mongodb/commit/c97fb271) [cherry-pick] Publish GenericResource (#1383) (#1384)
- [0869cd91](https://github.com/stashed/mongodb/commit/0869cd91) Replace rs.secondaryOk() with rs.slaveOk() (#1370)
- [50073b07](https://github.com/stashed/mongodb/commit/50073b07) Add support for backup and restore from different namespace (#1354) (#1355)


### [3.4.22-v14](https://github.com/stashed/mongodb/releases/tag/3.4.22-v14)

- [256947cb](https://github.com/stashed/mongodb/commit/256947cb) Prepare for release 3.4.22-v14 (#1411)
- [f37eb065](https://github.com/stashed/mongodb/commit/f37eb065) [cherry-pick] Update SiteInfo (#1397) (#1399)
- [2878238c](https://github.com/stashed/mongodb/commit/2878238c) [cherry-pick] Publish GenericResource (#1383) (#1385)
- [234668b6](https://github.com/stashed/mongodb/commit/234668b6) Replace rs.secondaryOk() with rs.slaveOk() (#1371)
- [7548c1ce](https://github.com/stashed/mongodb/commit/7548c1ce) Add support for backup and restore from different namespace (#1354) (#1356)


### [3.6.8-v14](https://github.com/stashed/mongodb/releases/tag/3.6.8-v14)

- [8014a998](https://github.com/stashed/mongodb/commit/8014a998) Prepare for release 3.6.8-v14 (#1413)
- [889e8919](https://github.com/stashed/mongodb/commit/889e8919) [cherry-pick] Update SiteInfo (#1397) (#1401)
- [bc49f316](https://github.com/stashed/mongodb/commit/bc49f316) [cherry-pick] Publish GenericResource (#1383) (#1387)
- [0e83032b](https://github.com/stashed/mongodb/commit/0e83032b) Replace rs.secondaryOk() with rs.slaveOk() (#1372)
- [2a315847](https://github.com/stashed/mongodb/commit/2a315847) Add support for backup and restore from different namespace (#1354) (#1358)


### [3.6.13-v14](https://github.com/stashed/mongodb/releases/tag/3.6.13-v14)

- [9bb40904](https://github.com/stashed/mongodb/commit/9bb40904) Prepare for release 3.6.13-v14 (#1412)
- [f684aea1](https://github.com/stashed/mongodb/commit/f684aea1) [cherry-pick] Update SiteInfo (#1397) (#1400)
- [1fe57cba](https://github.com/stashed/mongodb/commit/1fe57cba) [cherry-pick] Publish GenericResource (#1383) (#1386)
- [2ba645e5](https://github.com/stashed/mongodb/commit/2ba645e5) Replace rs.secondaryOk() with rs.slaveOk() (#1373)
- [9b3eab86](https://github.com/stashed/mongodb/commit/9b3eab86) Add support for backup and restore from different namespace (#1354) (#1357)


### [4.0.3-v14](https://github.com/stashed/mongodb/releases/tag/4.0.3-v14)

- [257c6783](https://github.com/stashed/mongodb/commit/257c6783) Prepare for release 4.0.3-v14 (#1415)
- [7c40ea09](https://github.com/stashed/mongodb/commit/7c40ea09) [cherry-pick] Update SiteInfo (#1397) (#1403)
- [17fae849](https://github.com/stashed/mongodb/commit/17fae849) [cherry-pick] Publish GenericResource (#1383) (#1389)
- [a0f8b46f](https://github.com/stashed/mongodb/commit/a0f8b46f) Replace rs.secondaryOk() with rs.slaveOk() (#1374)
- [12958a7d](https://github.com/stashed/mongodb/commit/12958a7d) Add support for backup and restore from different namespace (#1354) (#1360)


### [4.0.5-v14](https://github.com/stashed/mongodb/releases/tag/4.0.5-v14)

- [d081d8c2](https://github.com/stashed/mongodb/commit/d081d8c2) Prepare for release 4.0.5-v14 (#1416)
- [e8cf5007](https://github.com/stashed/mongodb/commit/e8cf5007) [cherry-pick] Update SiteInfo (#1397) (#1404)
- [8e3796dc](https://github.com/stashed/mongodb/commit/8e3796dc) [cherry-pick] Publish GenericResource (#1383) (#1390)
- [64379f10](https://github.com/stashed/mongodb/commit/64379f10) Replace rs.secondaryOk() with rs.slaveOk() (#1375)
- [5f0670c7](https://github.com/stashed/mongodb/commit/5f0670c7) Add support for backup and restore from different namespace (#1354) (#1361)


### [4.0.11-v14](https://github.com/stashed/mongodb/releases/tag/4.0.11-v14)

- [9a366c96](https://github.com/stashed/mongodb/commit/9a366c96) Prepare for release 4.0.11-v14 (#1414)
- [7666db40](https://github.com/stashed/mongodb/commit/7666db40) [cherry-pick] Update SiteInfo (#1397) (#1402)
- [17e7281a](https://github.com/stashed/mongodb/commit/17e7281a) [cherry-pick] Publish GenericResource (#1383) (#1388)
- [f4535fa3](https://github.com/stashed/mongodb/commit/f4535fa3) Replace rs.secondaryOk() with rs.slaveOk() (#1376)
- [06bd1077](https://github.com/stashed/mongodb/commit/06bd1077) Add support for backup and restore from different namespace (#1354) (#1359)


### [4.1.4-v14](https://github.com/stashed/mongodb/releases/tag/4.1.4-v14)

- [e2fc99c1](https://github.com/stashed/mongodb/commit/e2fc99c1) Prepare for release 4.1.4-v14 (#1418)
- [b81412b3](https://github.com/stashed/mongodb/commit/b81412b3) [cherry-pick] Update SiteInfo (#1397) (#1405)
- [d0e8bc14](https://github.com/stashed/mongodb/commit/d0e8bc14) [cherry-pick] Publish GenericResource (#1383) (#1392)
- [dc2da4b8](https://github.com/stashed/mongodb/commit/dc2da4b8) Replace rs.secondaryOk() with rs.slaveOk() (#1377)
- [9a4d21a2](https://github.com/stashed/mongodb/commit/9a4d21a2) Add support for backup and restore from different namespace (#1354) (#1363)


### [4.1.7-v14](https://github.com/stashed/mongodb/releases/tag/4.1.7-v14)

- [ce6044eb](https://github.com/stashed/mongodb/commit/ce6044eb) Prepare for release 4.1.7-v14 (#1419)
- [21fff000](https://github.com/stashed/mongodb/commit/21fff000) [cherry-pick] Update SiteInfo (#1397) (#1406)
- [e9036951](https://github.com/stashed/mongodb/commit/e9036951) [cherry-pick] Publish GenericResource (#1383) (#1393)
- [f1a5a8db](https://github.com/stashed/mongodb/commit/f1a5a8db) Replace rs.secondaryOk() with rs.slaveOk() (#1378)
- [8a495673](https://github.com/stashed/mongodb/commit/8a495673) Add support for backup and restore from different namespace (#1354) (#1364)


### [4.1.13-v14](https://github.com/stashed/mongodb/releases/tag/4.1.13-v14)

- [cb3eb5ce](https://github.com/stashed/mongodb/commit/cb3eb5ce) Prepare for release 4.1.13-v14 (#1417)
- [55690daa](https://github.com/stashed/mongodb/commit/55690daa) [cherry-pick] Publish GenericResource (#1383) (#1391)
- [2743c644](https://github.com/stashed/mongodb/commit/2743c644) Replace rs.secondaryOk() with rs.slaveOk() (#1379)
- [c5646467](https://github.com/stashed/mongodb/commit/c5646467) Add support for backup and restore from different namespace (#1354) (#1362)


### [4.2.3-v14](https://github.com/stashed/mongodb/releases/tag/4.2.3-v14)

- [fe2ff6dc](https://github.com/stashed/mongodb/commit/fe2ff6dc) Prepare for release 4.2.3-v14 (#1420)
- [680e2e09](https://github.com/stashed/mongodb/commit/680e2e09) [cherry-pick] Update SiteInfo (#1397) (#1407)
- [561836a6](https://github.com/stashed/mongodb/commit/561836a6) [cherry-pick] Publish GenericResource (#1383) (#1394)
- [9bbe8802](https://github.com/stashed/mongodb/commit/9bbe8802) Replace rs.secondaryOk() with rs.slaveOk() (#1380)
- [37d09b42](https://github.com/stashed/mongodb/commit/37d09b42) Add support for backup and restore from different namespace (#1354) (#1365)
- [51c4a8dc](https://github.com/stashed/mongodb/commit/51c4a8dc) Prepare for release 4.2.3-v13 (#1350)


### [4.4.6-v5](https://github.com/stashed/mongodb/releases/tag/4.4.6-v5)

- [b7533ecb](https://github.com/stashed/mongodb/commit/b7533ecb) Prepare for release 4.4.6-v5 (#1421)
- [1de5ff46](https://github.com/stashed/mongodb/commit/1de5ff46) [cherry-pick] Update SiteInfo (#1397) (#1408)
- [ed94bdd8](https://github.com/stashed/mongodb/commit/ed94bdd8) [cherry-pick] Publish GenericResource (#1383) (#1395)
- [1b2e4091](https://github.com/stashed/mongodb/commit/1b2e4091) Replace `adminCreds` with `mongoCreds` (#1381)
- [be037ead](https://github.com/stashed/mongodb/commit/be037ead) Add support for backup and restore from different namespace (#1354) (#1366)
- [e79762be](https://github.com/stashed/mongodb/commit/e79762be) Prepare for release 4.4.6-v4 (#1351)


### [5.0.3-v2](https://github.com/stashed/mongodb/releases/tag/5.0.3-v2)

- [dd8649d1](https://github.com/stashed/mongodb/commit/dd8649d1) Prepare for release 5.0.3-v2 (#1422)
- [db232f1c](https://github.com/stashed/mongodb/commit/db232f1c) [cherry-pick] Update SiteInfo (#1397) (#1409)
- [b157e44d](https://github.com/stashed/mongodb/commit/b157e44d) [cherry-pick] Publish GenericResource (#1383) (#1396)
- [b45c3ad7](https://github.com/stashed/mongodb/commit/b45c3ad7) Replace `adminCreds` with `mongoCreds` (#1382)
- [caf4c255](https://github.com/stashed/mongodb/commit/caf4c255) Add support for backup and restore from different namespace (#1354) (#1367)
- [52d5104b](https://github.com/stashed/mongodb/commit/52d5104b) Prepare for release 5.0.3-v1 (#1352)



## [stashed/mysql](https://github.com/stashed/mysql)

### [5.7.25-v15](https://github.com/stashed/mysql/releases/tag/5.7.25-v15)

- [5d1d85ce](https://github.com/stashed/mysql/commit/5d1d85ce) Prepare for release 5.7.25-v15 (#574)
- [3e19e8b5](https://github.com/stashed/mysql/commit/3e19e8b5) [cherry-pick] Update SiteInfo (#569) (#570)
- [ea59f1e9](https://github.com/stashed/mysql/commit/ea59f1e9) Publish GenericResource (#564) (#568)
- [0975295b](https://github.com/stashed/mysql/commit/0975295b) Add support for backup and restore from different namespace (#559) (#560)


### [8.0.3-v15](https://github.com/stashed/mysql/releases/tag/8.0.3-v15)

- [ea377e4e](https://github.com/stashed/mysql/commit/ea377e4e) Prepare for release 8.0.3-v15 (#577)
- [78289a40](https://github.com/stashed/mysql/commit/78289a40) [cherry-pick] Update SiteInfo (#569) (#573)
- [c41d4c88](https://github.com/stashed/mysql/commit/c41d4c88) Publish GenericResource (#564) (#565)
- [810af637](https://github.com/stashed/mysql/commit/810af637) Add support for backup and restore from different namespace (#559) (#563)


### [8.0.14-v15](https://github.com/stashed/mysql/releases/tag/8.0.14-v15)

- [1fefc3d4](https://github.com/stashed/mysql/commit/1fefc3d4) Prepare for release 8.0.14-v15 (#575)
- [1bc03e61](https://github.com/stashed/mysql/commit/1bc03e61) [cherry-pick] Update SiteInfo (#569) (#571)
- [d2400622](https://github.com/stashed/mysql/commit/d2400622) Publish GenericResource (#564) (#567)
- [e29d03ef](https://github.com/stashed/mysql/commit/e29d03ef) Add support for backup and restore from different namespace (#559) (#561)


### [8.0.21-v9](https://github.com/stashed/mysql/releases/tag/8.0.21-v9)

- [52988134](https://github.com/stashed/mysql/commit/52988134) Prepare for release 8.0.21-v9 (#576)
- [1b8be622](https://github.com/stashed/mysql/commit/1b8be622) [cherry-pick] Update SiteInfo (#569) (#572)
- [b6ad6e0a](https://github.com/stashed/mysql/commit/b6ad6e0a) Publish GenericResource (#564) (#566)
- [96d2927f](https://github.com/stashed/mysql/commit/96d2927f) Add support for backup and restore from different namespace (#559) (#562)



## [stashed/nats](https://github.com/stashed/nats)

### [2.6.1-v2](https://github.com/stashed/nats/releases/tag/2.6.1-v2)

- [0a8c599](https://github.com/stashed/nats/commit/0a8c599) Prepare for release 2.6.1-v2 (#28)
- [101a0b2](https://github.com/stashed/nats/commit/101a0b2) [cherry-pick] Update SiteInfo (#26) (#27)
- [5eecf9c](https://github.com/stashed/nats/commit/5eecf9c) [cherry-pick] Publish GenericResource (#24) (#25)
- [032b9da](https://github.com/stashed/nats/commit/032b9da) Refactor code by adding new methods (#23)
- [3a59f1f](https://github.com/stashed/nats/commit/3a59f1f) Support backup & restore from different namespace and external NATS Server (#21) (#22)



## [stashed/percona-xtradb](https://github.com/stashed/percona-xtradb)

### [5.7-v10](https://github.com/stashed/percona-xtradb/releases/tag/5.7-v10)

- [25cd0958](https://github.com/stashed/percona-xtradb/commit/25cd0958) Prepare for release 5.7-v10 (#238)
- [685feb7d](https://github.com/stashed/percona-xtradb/commit/685feb7d) [cherry-pick] Update SiteInfo (#236) (#237)
- [d265928b](https://github.com/stashed/percona-xtradb/commit/d265928b) [cherry-pick] Publish GenericResource (#234) (#235)
- [45cc6e62](https://github.com/stashed/percona-xtradb/commit/45cc6e62) Add support for backup and restore from different namespace (#232) (#233)



## [stashed/postgres](https://github.com/stashed/postgres)

### [9.6.19-v13](https://github.com/stashed/postgres/releases/tag/9.6.19-v13)

- [2e928b1b](https://github.com/stashed/postgres/commit/2e928b1b) Prepare for release 9.6.19-v13 (#991)
- [7bd10a34](https://github.com/stashed/postgres/commit/7bd10a34) [cherry-pick] Update SiteInfo (#980) (#985)
- [23a9982b](https://github.com/stashed/postgres/commit/23a9982b) [cherry-pick] Publish GenericResource (#973) (#979)
- [0db5519e](https://github.com/stashed/postgres/commit/0db5519e) Add support for backup and restore from different namespace (#966) (#972)


### [10.14-v13](https://github.com/stashed/postgres/releases/tag/10.14-v13)

- [2813a98a](https://github.com/stashed/postgres/commit/2813a98a) Prepare for release 10.14-v13 (#986)
- [50dba7d7](https://github.com/stashed/postgres/commit/50dba7d7) [cherry-pick] Update SiteInfo (#980) (#981)
- [616fd1e5](https://github.com/stashed/postgres/commit/616fd1e5) [cherry-pick] Publish GenericResource (#973) (#974)
- [2e46777c](https://github.com/stashed/postgres/commit/2e46777c) Add support for backup and restore from different namespace (#966) (#967)


### [11.9-v13](https://github.com/stashed/postgres/releases/tag/11.9-v13)

- [91c7414d](https://github.com/stashed/postgres/commit/91c7414d) Prepare for release 11.9-v13 (#987)
- [32e60021](https://github.com/stashed/postgres/commit/32e60021) [cherry-pick] Update SiteInfo (#980) (#982)
- [4e3b5204](https://github.com/stashed/postgres/commit/4e3b5204) [cherry-pick] Publish GenericResource (#973) (#975)
- [c87ee334](https://github.com/stashed/postgres/commit/c87ee334) Add support for backup and restore from different namespace (#966) (#968)


### [12.4-v13](https://github.com/stashed/postgres/releases/tag/12.4-v13)

- [938c3f3a](https://github.com/stashed/postgres/commit/938c3f3a) Prepare for release 12.4-v13 (#988)
- [57b64ffa](https://github.com/stashed/postgres/commit/57b64ffa) [cherry-pick] Update SiteInfo (#980) (#983)
- [4a123cdf](https://github.com/stashed/postgres/commit/4a123cdf) [cherry-pick] Publish GenericResource (#973) (#976)
- [d375d3e6](https://github.com/stashed/postgres/commit/d375d3e6) Add support for backup and restore from different namespace (#966) (#969)


### [13.1-v10](https://github.com/stashed/postgres/releases/tag/13.1-v10)

- [0e9f274d](https://github.com/stashed/postgres/commit/0e9f274d) Prepare for release 13.1-v10 (#989)
- [933223ea](https://github.com/stashed/postgres/commit/933223ea) [cherry-pick] Publish GenericResource (#973) (#977)
- [6491c2c7](https://github.com/stashed/postgres/commit/6491c2c7) Add support for backup and restore from different namespace (#966) (#970)


### [14.0-v2](https://github.com/stashed/postgres/releases/tag/14.0-v2)

- [6335e10e](https://github.com/stashed/postgres/commit/6335e10e) Prepare for release 14.0-v2 (#990)
- [fe4c5c2d](https://github.com/stashed/postgres/commit/fe4c5c2d) [cherry-pick] Update SiteInfo (#980) (#984)
- [a69989d5](https://github.com/stashed/postgres/commit/a69989d5) [cherry-pick] Publish GenericResource (#973) (#978)
- [bd847a6e](https://github.com/stashed/postgres/commit/bd847a6e) Add support for backup and restore from different namespace (#966) (#971)



## [stashed/redis](https://github.com/stashed/redis)

### [5.0.13-v3](https://github.com/stashed/redis/releases/tag/5.0.13-v3)

- [191fc96](https://github.com/stashed/redis/commit/191fc96) Prepare for release 5.0.13-v3 (#78)
- [38599a8](https://github.com/stashed/redis/commit/38599a8) [cherry-pick] Publish GenericResource (#73) (#74)
- [732fae8](https://github.com/stashed/redis/commit/732fae8) Add support for backup and restore from different namespace (#70) (#71)


### [6.2.5-v3](https://github.com/stashed/redis/releases/tag/6.2.5-v3)

- [ba2bd19](https://github.com/stashed/redis/commit/ba2bd19) Prepare for release 6.2.5-v3 (#79)
- [6e5f3d2](https://github.com/stashed/redis/commit/6e5f3d2) [cherry-pick] Publish GenericResource (#73) (#75)
- [d837a32](https://github.com/stashed/redis/commit/d837a32) Add support for backup and restore from different namespace (#70) (#72)



## [stashed/stash](https://github.com/stashed/stash)

### [v0.18.0](https://github.com/stashed/stash/releases/tag/v0.18.0)

- [9536bc1b](https://github.com/stashed/stash/commit/9536bc1b) Prepare for release v0.18.0 (#1421)
- [0c84840e](https://github.com/stashed/stash/commit/0c84840e) Update UID generation for GenericResource (#1420)
- [2162c125](https://github.com/stashed/stash/commit/2162c125) Refactor restore process status handling + add cross namespace tests (#1419)
- [080ebfcf](https://github.com/stashed/stash/commit/080ebfcf) Update SiteInfo (#1418)
- [2727c071](https://github.com/stashed/stash/commit/2727c071) Publish GenericResource (#1416)
- [dd6b10f0](https://github.com/stashed/stash/commit/dd6b10f0) Expose original UID for generic resource
- [a121b8c3](https://github.com/stashed/stash/commit/a121b8c3) Always set type meta for GenericResource
- [9300292a](https://github.com/stashed/stash/commit/9300292a) Publish GenericResource (#1415)
- [ffe0d847](https://github.com/stashed/stash/commit/ffe0d847) Don't register CRDs from operator + allow passing crd-installer tag (#1409)
- [eb69ad4e](https://github.com/stashed/stash/commit/eb69ad4e) Port UsagePolicy validation related changes from Enterprise operator (#1414)
- [06988c00](https://github.com/stashed/stash/commit/06988c00) Always keep the last completed BackupSession (when `backupHistoryLimit > 0`) even if outside of history limit (#1413)
- [c81cf41b](https://github.com/stashed/stash/commit/c81cf41b) Add support for custom Pusgateway (#1412)
- [127017c9](https://github.com/stashed/stash/commit/127017c9) Update repository config (#1411)
- [1e41192e](https://github.com/stashed/stash/commit/1e41192e) Cleanup legacy code + Port changes from enterprise operator (#1410)
- [1ca2129b](https://github.com/stashed/stash/commit/1ca2129b) Update repository config (#1407)
- [849c782a](https://github.com/stashed/stash/commit/849c782a) Fix invalid task name in restore task resolver (#1406)



## [stashed/ui-server](https://github.com/stashed/ui-server)

### [v0.1.0](https://github.com/stashed/ui-server/releases/tag/v0.1.0)

- [e2ce681](https://github.com/stashed/ui-server/commit/e2ce681) Prepare for release v0.1.0 (#8)
- [264756c](https://github.com/stashed/ui-server/commit/264756c) Update dependencies (#7)




