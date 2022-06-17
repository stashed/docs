---
title: Changelog | Stash
description: Changelog
menu:
  docs_{{.version}}:
    identifier: changelog-stash-v2022.06.21
    name: Changelog-v2022.06.21
    parent: welcome
    weight: 20220621
product_name: stash
menu_name: docs_{{.version}}
section_menu_id: welcome
url: /docs/{{.version}}/welcome/changelog-v2022.06.21/
aliases:
  - /docs/{{.version}}/CHANGELOG-v2022.06.21/
---

# Stash v2022.06.21 (2022-06-17)


## [stashed/apimachinery](https://github.com/stashed/apimachinery)

### [v0.21.0](https://github.com/stashed/apimachinery/releases/tag/v0.21.0)

- [e19298fd](https://github.com/stashed/apimachinery/commit/e19298fd) Add Hook and TimeOut support in auto-backup (#174)
- [b90c85f4](https://github.com/stashed/apimachinery/commit/b90c85f4) Update to k8s 1.24 toolchain (#172)
- [5d2e9a06](https://github.com/stashed/apimachinery/commit/5d2e9a06) Add timeout for backup and restore (#169)
- [1c812e70](https://github.com/stashed/apimachinery/commit/1c812e70) Update metrics labels to sync with Panopticon. (#170)
- [6891e99a](https://github.com/stashed/apimachinery/commit/6891e99a) Test against Kubernetes 1.24.0 (#171)



## [stashed/cli](https://github.com/stashed/cli)

### [v0.21.0](https://github.com/stashed/cli/releases/tag/v0.21.0)

- [4942f3ec](https://github.com/stashed/cli/commit/4942f3ec) Prepare for release v0.21.0 (#164)
- [fdd47d95](https://github.com/stashed/cli/commit/fdd47d95) Update to k8s 1.24 toolchain
- [fabefbdd](https://github.com/stashed/cli/commit/fabefbdd) Cancel concurrent in pogress CI runs



## [stashed/elasticsearch](https://github.com/stashed/elasticsearch)

### [5.6.4-v18](https://github.com/stashed/elasticsearch/releases/tag/5.6.4-v18)

- [f71cc84e](https://github.com/stashed/elasticsearch/commit/f71cc84e) Prepare for release 5.6.4-v18 (#1213)
- [d13474d2](https://github.com/stashed/elasticsearch/commit/d13474d2) Use Go 1.18 (#1203)
- [f942b902](https://github.com/stashed/elasticsearch/commit/f942b902) [cherry-pick] Make sure to fail the backup/restore sessions if license check fails. (#1192) (#1193)


### [6.2.4-v18](https://github.com/stashed/elasticsearch/releases/tag/6.2.4-v18)

- [60daf578](https://github.com/stashed/elasticsearch/commit/60daf578) Prepare for release 6.2.4-v18 (#1214)
- [7e107d0d](https://github.com/stashed/elasticsearch/commit/7e107d0d) Use Go 1.18 (#1204)
- [e097a0f7](https://github.com/stashed/elasticsearch/commit/e097a0f7) [cherry-pick] Make sure to fail the backup/restore sessions if license check fails. (#1192) (#1194)


### [6.3.0-v18](https://github.com/stashed/elasticsearch/releases/tag/6.3.0-v18)

- [2ab1c09a](https://github.com/stashed/elasticsearch/commit/2ab1c09a) Prepare for release 6.3.0-v18 (#1215)
- [902833b5](https://github.com/stashed/elasticsearch/commit/902833b5) Use Go 1.18 (#1205)
- [9b81f591](https://github.com/stashed/elasticsearch/commit/9b81f591) [cherry-pick] Make sure to fail the backup/restore sessions if license check fails. (#1192) (#1195)


### [6.4.0-v18](https://github.com/stashed/elasticsearch/releases/tag/6.4.0-v18)

- [a7c65048](https://github.com/stashed/elasticsearch/commit/a7c65048) Prepare for release 6.4.0-v18 (#1216)
- [95e1e146](https://github.com/stashed/elasticsearch/commit/95e1e146) Use Go 1.18 (#1206)
- [c2127005](https://github.com/stashed/elasticsearch/commit/c2127005) [cherry-pick] Make sure to fail the backup/restore sessions if license check fails. (#1192) (#1196)


### [6.5.3-v18](https://github.com/stashed/elasticsearch/releases/tag/6.5.3-v18)

- [08aa5cd0](https://github.com/stashed/elasticsearch/commit/08aa5cd0) Prepare for release 6.5.3-v18 (#1217)
- [39022b9c](https://github.com/stashed/elasticsearch/commit/39022b9c) Use Go 1.18 (#1207)
- [ea839cea](https://github.com/stashed/elasticsearch/commit/ea839cea) [cherry-pick] Make sure to fail the backup/restore sessions if license check fails. (#1192) (#1197)


### [6.8.0-v18](https://github.com/stashed/elasticsearch/releases/tag/6.8.0-v18)

- [917089a9](https://github.com/stashed/elasticsearch/commit/917089a9) Prepare for release 6.8.0-v18 (#1218)
- [6cddcce7](https://github.com/stashed/elasticsearch/commit/6cddcce7) Use Go 1.18 (#1208)
- [0b23666b](https://github.com/stashed/elasticsearch/commit/0b23666b) [cherry-pick] Make sure to fail the backup/restore sessions if license check fails. (#1192) (#1198)


### [7.2.0-v18](https://github.com/stashed/elasticsearch/releases/tag/7.2.0-v18)

- [9b84e4c5](https://github.com/stashed/elasticsearch/commit/9b84e4c5) Use Go 1.18 (#1210)
- [4bae3459](https://github.com/stashed/elasticsearch/commit/4bae3459) [cherry-pick] Make sure to fail the backup/restore sessions if license check fails. (#1192) (#1200)


### [7.3.2-v18](https://github.com/stashed/elasticsearch/releases/tag/7.3.2-v18)

- [6f2b4ece](https://github.com/stashed/elasticsearch/commit/6f2b4ece) Use Go 1.18 (#1212)
- [fb895a65](https://github.com/stashed/elasticsearch/commit/fb895a65) [cherry-pick] Make sure to fail the backup/restore sessions if license check fails. (#1192) (#1201)


### [7.14.0-v4](https://github.com/stashed/elasticsearch/releases/tag/7.14.0-v4)

- [ccd4b478](https://github.com/stashed/elasticsearch/commit/ccd4b478) Stop sending Google analytics events
- [9aaec379](https://github.com/stashed/elasticsearch/commit/9aaec379) Use Go 1.18 (#1209)
- [24eb7559](https://github.com/stashed/elasticsearch/commit/24eb7559) Make sure to fail the backup/restore sessions if license check fails. (#1192) (#1199)


### [8.2.0-v1](https://github.com/stashed/elasticsearch/releases/tag/8.2.0-v1)

- [055cf095](https://github.com/stashed/elasticsearch/commit/055cf095) Use Go 1.18 (#1211)
- [0da3aec9](https://github.com/stashed/elasticsearch/commit/0da3aec9) Make sure to fail the backup/restore sessions if license check fails. (#1192) (#1202)



## [stashed/enterprise](https://github.com/stashed/enterprise)

### [v0.21.0](https://github.com/stashed/enterprise/releases/tag/v0.21.0)

- [a6f2dc48](https://github.com/stashed/enterprise/commit/a6f2dc48) Prepare for release v0.21.0 (#179)
- [1da47ae2](https://github.com/stashed/enterprise/commit/1da47ae2) Add support for Hook and TimeOut in auto-backup (#178)
- [92d46877](https://github.com/stashed/enterprise/commit/92d46877) Cleanup dependencies (#177)
- [7124c7f3](https://github.com/stashed/enterprise/commit/7124c7f3) Update to k8s 1.24 toolchain (#176)
- [ba59e128](https://github.com/stashed/enterprise/commit/ba59e128) Add timeout for backup and restore (#173)
- [a51a3725](https://github.com/stashed/enterprise/commit/a51a3725) Check restore output before updating restore status (#175)



## [stashed/etcd](https://github.com/stashed/etcd)

### [3.5.0-v5](https://github.com/stashed/etcd/releases/tag/3.5.0-v5)

- [7d96b9b](https://github.com/stashed/etcd/commit/7d96b9b) Prepare for release 3.5.0-v5 (#49)
- [a59967d](https://github.com/stashed/etcd/commit/a59967d) Use Go 1.18 (#48)
- [3d20c8f](https://github.com/stashed/etcd/commit/3d20c8f) Use Go 1.18 (#32) (#47)
- [8705d44](https://github.com/stashed/etcd/commit/8705d44) Make sure to fail the backup/restore sessions if license check fails. (#45) (#46)



## [stashed/installer](https://github.com/stashed/installer)

### [v2022.06.21](https://github.com/stashed/installer/releases/tag/v2022.06.21)

- [38ca633b](https://github.com/stashed/installer/commit/38ca633b) Prepare for release v2022.06.21 (#263)
- [6a6c4726](https://github.com/stashed/installer/commit/6a6c4726) Remove stash-metrics from stash chart (#262)
- [32e21c6f](https://github.com/stashed/installer/commit/32e21c6f) Update crds (#261)
- [dd3340dd](https://github.com/stashed/installer/commit/dd3340dd) Don't set tag in values files
- [803d36db](https://github.com/stashed/installer/commit/803d36db) Add RBAC permissions for `coordination.k8s.io/leases` (#260)
- [c0bf346e](https://github.com/stashed/installer/commit/c0bf346e) Rename percona-xtradb to  perconaxtradb in catalog.json (#259)
- [5b581a8b](https://github.com/stashed/installer/commit/5b581a8b) Update to k8s 1.24 toolchain
- [0dd58f75](https://github.com/stashed/installer/commit/0dd58f75) Update Grafana dashboard's labels. (#257)
- [b92b8989](https://github.com/stashed/installer/commit/b92b8989) Get operator tag from .Chart.AppVersion (#258)
- [72537db0](https://github.com/stashed/installer/commit/72537db0) Test against Kubernetes 1.24.0 (#256)



## [stashed/kubedump](https://github.com/stashed/kubedump)

### [0.1.0-v1](https://github.com/stashed/kubedump/releases/tag/0.1.0-v1)

- [8b71a18](https://github.com/stashed/kubedump/commit/8b71a18) Prepare for release 0.1.0-v1 (#9)
- [d976331](https://github.com/stashed/kubedump/commit/d976331) Go 1.18 (#8)
- [3b2d4f8](https://github.com/stashed/kubedump/commit/3b2d4f8) Make sure to fail the backup/restore sessions if license check fails. (#7)



## [stashed/mariadb](https://github.com/stashed/mariadb)

### [10.5.8-v11](https://github.com/stashed/mariadb/releases/tag/10.5.8-v11)

- [75a46ec](https://github.com/stashed/mariadb/commit/75a46ec) Prepare for release 10.5.8-v11 (#190)
- [f105e19](https://github.com/stashed/mariadb/commit/f105e19) Go 1.18 (#189)
- [b825468](https://github.com/stashed/mariadb/commit/b825468) Make sure to fail the backup/restore sessions if license check fails. (#187) (#188)



## [stashed/mongodb](https://github.com/stashed/mongodb)

### [3.4.17-v17](https://github.com/stashed/mongodb/releases/tag/3.4.17-v17)

- [da8cc237](https://github.com/stashed/mongodb/commit/da8cc237) Prepare for release 3.4.17-v17 (#1579)
- [7180c291](https://github.com/stashed/mongodb/commit/7180c291) Go 1.18 (#1568)
- [1ca50bd5](https://github.com/stashed/mongodb/commit/1ca50bd5) Make sure to fail the backup/restore sessions if license check fails. (#1554) (#1555)


### [3.4.22-v17](https://github.com/stashed/mongodb/releases/tag/3.4.22-v17)

- [3d2739cf](https://github.com/stashed/mongodb/commit/3d2739cf) Prepare for release 3.4.22-v17 (#1580)
- [2be6391c](https://github.com/stashed/mongodb/commit/2be6391c) Go 1.18 (#1578)
- [fc17eb80](https://github.com/stashed/mongodb/commit/fc17eb80) Make sure to fail the backup/restore sessions if license check fails. (#1554) (#1556)


### [3.6.8-v17](https://github.com/stashed/mongodb/releases/tag/3.6.8-v17)

- [b41ddb46](https://github.com/stashed/mongodb/commit/b41ddb46) Prepare for release 3.6.8-v17 (#1582)
- [0b147350](https://github.com/stashed/mongodb/commit/0b147350) Go 1.18 (#1576)
- [f27ba96e](https://github.com/stashed/mongodb/commit/f27ba96e) Make sure to fail the backup/restore sessions if license check fails. (#1554) (#1558)


### [3.6.13-v17](https://github.com/stashed/mongodb/releases/tag/3.6.13-v17)

- [687f7d8f](https://github.com/stashed/mongodb/commit/687f7d8f) Prepare for release 3.6.13-v17 (#1581)
- [c672503e](https://github.com/stashed/mongodb/commit/c672503e) Go 1.18 (#1577)
- [896889f1](https://github.com/stashed/mongodb/commit/896889f1) Make sure to fail the backup/restore sessions if license check fails. (#1554) (#1557)


### [4.0.3-v17](https://github.com/stashed/mongodb/releases/tag/4.0.3-v17)

- [c0cb607a](https://github.com/stashed/mongodb/commit/c0cb607a) Prepare for release 4.0.3-v17 (#1584)
- [fdabcd75](https://github.com/stashed/mongodb/commit/fdabcd75) Go 1.18 (#1574)
- [c309e641](https://github.com/stashed/mongodb/commit/c309e641) Make sure to fail the backup/restore sessions if license check fails. (#1554) (#1560)


### [4.0.5-v17](https://github.com/stashed/mongodb/releases/tag/4.0.5-v17)

- [f30c6a13](https://github.com/stashed/mongodb/commit/f30c6a13) Prepare for release 4.0.5-v17 (#1585)
- [2ab11e66](https://github.com/stashed/mongodb/commit/2ab11e66) Go 1.18 (#1573)
- [41529a1b](https://github.com/stashed/mongodb/commit/41529a1b) Make sure to fail the backup/restore sessions if license check fails. (#1554) (#1561)


### [4.0.11-v17](https://github.com/stashed/mongodb/releases/tag/4.0.11-v17)

- [b9ea05bd](https://github.com/stashed/mongodb/commit/b9ea05bd) Prepare for release 4.0.11-v17 (#1583)
- [f4984613](https://github.com/stashed/mongodb/commit/f4984613) Go 1.18 (#1575)
- [66fc2e57](https://github.com/stashed/mongodb/commit/66fc2e57) Make sure to fail the backup/restore sessions if license check fails. (#1554) (#1559)


### [4.1.4-v17](https://github.com/stashed/mongodb/releases/tag/4.1.4-v17)

- [4c92d8a3](https://github.com/stashed/mongodb/commit/4c92d8a3) Prepare for release 4.1.4-v17 (#1587)
- [236d7475](https://github.com/stashed/mongodb/commit/236d7475) Go 1.18 (#1571)
- [7ab9f03d](https://github.com/stashed/mongodb/commit/7ab9f03d) Make sure to fail the backup/restore sessions if license check fails. (#1554) (#1563)


### [4.1.7-v17](https://github.com/stashed/mongodb/releases/tag/4.1.7-v17)

- [104d28bd](https://github.com/stashed/mongodb/commit/104d28bd) Prepare for release 4.1.7-v17 (#1588)
- [0a8dbae0](https://github.com/stashed/mongodb/commit/0a8dbae0) Go 1.18 (#1570)
- [6f3d4da7](https://github.com/stashed/mongodb/commit/6f3d4da7) Make sure to fail the backup/restore sessions if license check fails. (#1554) (#1564)


### [4.1.13-v17](https://github.com/stashed/mongodb/releases/tag/4.1.13-v17)

- [252b5708](https://github.com/stashed/mongodb/commit/252b5708) Prepare for release 4.1.13-v17 (#1586)
- [6ffef02c](https://github.com/stashed/mongodb/commit/6ffef02c) Go 1.18 (#1572)
- [a1de60e6](https://github.com/stashed/mongodb/commit/a1de60e6) Make sure to fail the backup/restore sessions if license check fails. (#1554) (#1562)


### [4.2.3-v17](https://github.com/stashed/mongodb/releases/tag/4.2.3-v17)

- [fb5eb455](https://github.com/stashed/mongodb/commit/fb5eb455) Prepare for release 4.2.3-v17 (#1589)
- [730ff66a](https://github.com/stashed/mongodb/commit/730ff66a) Go 1.18 (#1569)
- [820fffd0](https://github.com/stashed/mongodb/commit/820fffd0) Make sure to fail the backup/restore sessions if license check fails. (#1554) (#1565)


### [4.4.6-v8](https://github.com/stashed/mongodb/releases/tag/4.4.6-v8)

- [ba6f8429](https://github.com/stashed/mongodb/commit/ba6f8429) Prepare for release 4.4.6-v8 (#1590)
- [768a40c1](https://github.com/stashed/mongodb/commit/768a40c1) Make sure to fail the backup/restore sessions if license check fails. (#1554) (#1566)


### [5.0.3-v5](https://github.com/stashed/mongodb/releases/tag/5.0.3-v5)

- [40168c44](https://github.com/stashed/mongodb/commit/40168c44) Prepare for release 5.0.3-v5 (#1591)
- [78999c9e](https://github.com/stashed/mongodb/commit/78999c9e) Make sure to fail the backup/restore sessions if license check fails. (#1554) (#1567)



## [stashed/mysql](https://github.com/stashed/mysql)

### [5.7.25-v18](https://github.com/stashed/mysql/releases/tag/5.7.25-v18)

- [6788bc23](https://github.com/stashed/mysql/commit/6788bc23) Prepare for release 5.7.25-v18 (#633)
- [bf7bb0bc](https://github.com/stashed/mysql/commit/bf7bb0bc) [cherry-pick] Use Go 1.18 (#629)
- [6fca045a](https://github.com/stashed/mysql/commit/6fca045a) Make sure to fail the backup/restore sessions if license check fails. (#622) (#623)


### [8.0.3-v18](https://github.com/stashed/mysql/releases/tag/8.0.3-v18)

- [00a92f31](https://github.com/stashed/mysql/commit/00a92f31) Prepare for release 8.0.3-v18 (#636)
- [071a7d3f](https://github.com/stashed/mysql/commit/071a7d3f) [cherry-pick] Use Go 1.18 (#632)
- [d5b5a0b3](https://github.com/stashed/mysql/commit/d5b5a0b3) Use restic 0.13.1 (#604) (#627)
- [881b2a0e](https://github.com/stashed/mysql/commit/881b2a0e) Make sure to fail the backup/restore sessions if license check fails. (#622) (#626)


### [8.0.14-v18](https://github.com/stashed/mysql/releases/tag/8.0.14-v18)

- [0b2d6ec7](https://github.com/stashed/mysql/commit/0b2d6ec7) Prepare for release 8.0.14-v18 (#634)
- [e0034276](https://github.com/stashed/mysql/commit/e0034276) [cherry-pick] Use Go 1.18 (#630)
- [afae11d3](https://github.com/stashed/mysql/commit/afae11d3) Make sure to fail the backup/restore sessions if license check fails. (#622) (#624)


### [8.0.21-v12](https://github.com/stashed/mysql/releases/tag/8.0.21-v12)

- [c2d3d592](https://github.com/stashed/mysql/commit/c2d3d592) Prepare for release 8.0.21-v12 (#635)
- [4f532409](https://github.com/stashed/mysql/commit/4f532409) [cherry-pick] Use Go 1.18 (#631)
- [b8a3b870](https://github.com/stashed/mysql/commit/b8a3b870) Use restic 0.13.1 (#604) (#628)
- [b045519d](https://github.com/stashed/mysql/commit/b045519d) Make sure to fail the backup/restore sessions if license check fails. (#622) (#625)



## [stashed/nats](https://github.com/stashed/nats)

### [2.6.1-v6](https://github.com/stashed/nats/releases/tag/2.6.1-v6)

- [45e3eb5](https://github.com/stashed/nats/commit/45e3eb5) Prepare for release 2.6.1-v5 (#57)
- [918fba8](https://github.com/stashed/nats/commit/918fba8) [cherry-pick] Use Go 1.18 (#55)
- [d895e5b](https://github.com/stashed/nats/commit/d895e5b) Make sure to fail the backup/restore sessions if license check fails. (#52) (#53)


### [2.8.2-v1](https://github.com/stashed/nats/releases/tag/2.8.2-v1)

- [b2540f1](https://github.com/stashed/nats/commit/b2540f1) Prepare for release 2.8.2-v1 (#58)
- [f462469](https://github.com/stashed/nats/commit/f462469) [cherry-pick] Use Go 1.18 (#56)
- [bb05983](https://github.com/stashed/nats/commit/bb05983) Make sure to fail the backup/restore sessions if license check fails. (#52) (#54)



## [stashed/percona-xtradb](https://github.com/stashed/percona-xtradb)

### [5.7-v13](https://github.com/stashed/percona-xtradb/releases/tag/5.7-v13)

- [e02373e5](https://github.com/stashed/percona-xtradb/commit/e02373e5) Prepare for release 5.7-v13 (#261)
- [fc1a4395](https://github.com/stashed/percona-xtradb/commit/fc1a4395) [cherry-pick] Use Go 1.18 (#260)
- [e33797e5](https://github.com/stashed/percona-xtradb/commit/e33797e5) Make sure to fail the backup/restore sessions if license check fails. (#258) (#259)



## [stashed/postgres](https://github.com/stashed/postgres)

### [9.6.19-v16](https://github.com/stashed/postgres/releases/tag/9.6.19-v16)

- [1db055d0](https://github.com/stashed/postgres/commit/1db055d0) Prepare for release 9.6.19-v16 (#1074)
- [f245c188](https://github.com/stashed/postgres/commit/f245c188) [cherry-pick] Use Go 1.18 (#1068)
- [df3de5f4](https://github.com/stashed/postgres/commit/df3de5f4) Make sure to fail the backup/restore sessions if license check fails. (#1053) (#1059)


### [10.14-v16](https://github.com/stashed/postgres/releases/tag/10.14-v16)

- [7a6cca43](https://github.com/stashed/postgres/commit/7a6cca43) Prepare for release 10.14-v16 (#1069)
- [5e1d6ba6](https://github.com/stashed/postgres/commit/5e1d6ba6) [cherry-pick] Use Go 1.18 (#1063)
- [a8548d33](https://github.com/stashed/postgres/commit/a8548d33) Make sure to fail the backup/restore sessions if license check fails. (#1053) (#1054)


### [11.9-v16](https://github.com/stashed/postgres/releases/tag/11.9-v16)

- [ade2cb88](https://github.com/stashed/postgres/commit/ade2cb88) Prepare for release 11.9-v16 (#1070)
- [f02ceab3](https://github.com/stashed/postgres/commit/f02ceab3) [cherry-pick] Use Go 1.18 (#1064)
- [47293bab](https://github.com/stashed/postgres/commit/47293bab) Make sure to fail the backup/restore sessions if license check fails. (#1053) (#1055)


### [12.4-v16](https://github.com/stashed/postgres/releases/tag/12.4-v16)

- [a632d377](https://github.com/stashed/postgres/commit/a632d377) Prepare for release 12.4-v16 (#1071)
- [bcca56e6](https://github.com/stashed/postgres/commit/bcca56e6) [cherry-pick] Use Go 1.18 (#1065)
- [ba595303](https://github.com/stashed/postgres/commit/ba595303) Make sure to fail the backup/restore sessions if license check fails. (#1053) (#1056)


### [13.1-v13](https://github.com/stashed/postgres/releases/tag/13.1-v13)

- [61861efc](https://github.com/stashed/postgres/commit/61861efc) Prepare for release 13.1-v13 (#1072)
- [b3bba6a9](https://github.com/stashed/postgres/commit/b3bba6a9) [cherry-pick] Use Go 1.18 (#1066)
- [051ce90c](https://github.com/stashed/postgres/commit/051ce90c) Use restic 0.13.1 (#1027) (#1061)
- [b7fc16da](https://github.com/stashed/postgres/commit/b7fc16da) Make sure to fail the backup/restore sessions if license check fails. (#1053) (#1057)


### [14.0-v5](https://github.com/stashed/postgres/releases/tag/14.0-v5)

- [1fba7100](https://github.com/stashed/postgres/commit/1fba7100) Prepare for release 14.0-v5 (#1073)
- [a4d857ef](https://github.com/stashed/postgres/commit/a4d857ef) [cherry-pick] Use Go 1.18 (#1067)
- [b90b5890](https://github.com/stashed/postgres/commit/b90b5890) Use restic 0.13.1 (#1027) (#1060)
- [4638c6e1](https://github.com/stashed/postgres/commit/4638c6e1) Make sure to fail the backup/restore sessions if license check fails. (#1053) (#1058)



## [stashed/redis](https://github.com/stashed/redis)

### [5.0.13-v6](https://github.com/stashed/redis/releases/tag/5.0.13-v6)

- [2d26534](https://github.com/stashed/redis/commit/2d26534) Prepare for release 5.0.13-v6 (#113)
- [1ca8f25](https://github.com/stashed/redis/commit/1ca8f25) [cherry-pick] Use Go 1.18 (#111)
- [51ff930](https://github.com/stashed/redis/commit/51ff930) Make sure to fail the backup/restore sessions if license check fails. (#106) (#107)


### [6.2.5-v6](https://github.com/stashed/redis/releases/tag/6.2.5-v6)

- [e9528e1](https://github.com/stashed/redis/commit/e9528e1) Prepare for release 6.2.5-v6 (#114)
- [d565078](https://github.com/stashed/redis/commit/d565078) [cherry-pick] Use Go 1.18 (#112)
- [e7bb8c9](https://github.com/stashed/redis/commit/e7bb8c9) Make sure to fail the backup/restore sessions if license check fails. (#106) (#108)



## [stashed/stash](https://github.com/stashed/stash)

### [v0.21.0](https://github.com/stashed/stash/releases/tag/v0.21.0)

- [76df41cc](https://github.com/stashed/stash/commit/76df41cc) Prepare for release v0.21.0 (#1453)
- [e1bae66f](https://github.com/stashed/stash/commit/e1bae66f) Add RBAC permissions for `coordination.k8s.io/leases`
- [df9553d0](https://github.com/stashed/stash/commit/df9553d0) Add RBAC permissions for `coordination.k8s.io/leases`
- [846d952f](https://github.com/stashed/stash/commit/846d952f) Fix linter warning
- [6e5fd706](https://github.com/stashed/stash/commit/6e5fd706) Clean up dependencies (#1451)
- [4b6dd4eb](https://github.com/stashed/stash/commit/4b6dd4eb) Use github.com/imdario/mergo@v0.3.5
- [f2109b78](https://github.com/stashed/stash/commit/f2109b78) Update to k8s 1.24 toolchain (#1450)
- [33073c9b](https://github.com/stashed/stash/commit/33073c9b) Add timeout for backup and restore (#1449)
- [49926ef1](https://github.com/stashed/stash/commit/49926ef1) Check if restore output is nil before updating restore status (#1448)
- [a9ce58ed](https://github.com/stashed/stash/commit/a9ce58ed) Test against Kubernetes 1.24.0 (#1447)



## [stashed/ui-server](https://github.com/stashed/ui-server)

### [v0.3.0](https://github.com/stashed/ui-server/releases/tag/v0.3.0)

- [dbe1690](https://github.com/stashed/ui-server/commit/dbe1690) Prepare for release v0.3.0 (#14)
- [ccb5f80](https://github.com/stashed/ui-server/commit/ccb5f80) Use k8s 1.24 tool chain




