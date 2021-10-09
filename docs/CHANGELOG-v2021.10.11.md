---
title: Changelog | Stash
description: Changelog
menu:
  docs_{{.version}}:
    identifier: changelog-stash-v2021.10.11
    name: Changelog-v2021.10.11
    parent: welcome
    weight: 20211011
product_name: stash
menu_name: docs_{{.version}}
section_menu_id: welcome
url: /docs/{{.version}}/welcome/changelog-v2021.10.11/
aliases:
  - /docs/{{.version}}/CHANGELOG-v2021.10.11/
---

# Stash v2021.10.11 (2021-10-09)


## [appscode/stash-enterprise](https://github.com/appscode/stash-enterprise)

### [v0.16.0](https://github.com/appscode/stash-enterprise/releases/tag/v0.16.0)

- [34fe50bb](https://github.com/appscode/stash-enterprise/commit/34fe50bb) Prepare for release v0.16.0 (#126)
- [4807ec4c](https://github.com/appscode/stash-enterprise/commit/4807ec4c) Fix jwt-go security vulnerability (#125)
- [f1160998](https://github.com/appscode/stash-enterprise/commit/f1160998) Merge pull request #123 from appscode/etcd-permission
- [ce3cd3f8](https://github.com/appscode/stash-enterprise/commit/ce3cd3f8) Add support for ETCD restore flow
- [834d8c13](https://github.com/appscode/stash-enterprise/commit/834d8c13) Use nats.go v1.13.0 (#124)
- [446ee320](https://github.com/appscode/stash-enterprise/commit/446ee320) Update dependencies to publish SiteInfo (#122)
- [5d56e0f6](https://github.com/appscode/stash-enterprise/commit/5d56e0f6) Support passing args to restic backup/restore command (#120)
- [f10ab0dc](https://github.com/appscode/stash-enterprise/commit/f10ab0dc) Fix license-reader ClusterRoleBinding not cleaning up properly
- [dffe5c41](https://github.com/appscode/stash-enterprise/commit/dffe5c41) Update go.mod
- [c124a770](https://github.com/appscode/stash-enterprise/commit/c124a770) Undo changes of jobs.go
- [cc7f4827](https://github.com/appscode/stash-enterprise/commit/cc7f4827) fix clusterrolebinding issue
- [13ed9752](https://github.com/appscode/stash-enterprise/commit/13ed9752) Log warning if Community License is used with non-demo namespace (#119)
- [0026511c](https://github.com/appscode/stash-enterprise/commit/0026511c) Merge pull request #117 from appscode/fix-cronjob-name-prefix
- [ba3f466c](https://github.com/appscode/stash-enterprise/commit/ba3f466c) Use `stash-trigger` as backup triggering CronJob name prefix
- [d353c393](https://github.com/appscode/stash-enterprise/commit/d353c393) Use official restic v0.12.1 (#116)
- [e9b1db7e](https://github.com/appscode/stash-enterprise/commit/e9b1db7e) Update repository config (#115)
- [ea1895fc](https://github.com/appscode/stash-enterprise/commit/ea1895fc) Update dependencies (#114)



## [stashed/apimachinery](https://github.com/stashed/apimachinery)

### [v0.16.0](https://github.com/stashed/apimachinery/releases/tag/v0.16.0)

- [e5b1aaea](https://github.com/stashed/apimachinery/commit/e5b1aaea) Fix jwt-go security vulnerability (#129)
- [065a6d36](https://github.com/stashed/apimachinery/commit/065a6d36) Fix jwt-go security vulnerability (#128)
- [561988c6](https://github.com/stashed/apimachinery/commit/561988c6) Add condition for indicating restore completion (#127)
- [3ddabb57](https://github.com/stashed/apimachinery/commit/3ddabb57) Add "Running" phase for the individual restore host (#126)
- [1c75b4ce](https://github.com/stashed/apimachinery/commit/1c75b4ce) Support passing arguments to restic backup/restore command (#121)
- [307b411a](https://github.com/stashed/apimachinery/commit/307b411a) Update dependencies to publish SiteInfo (#125)
- [ce72641a](https://github.com/stashed/apimachinery/commit/ce72641a) Update dependencies to publish SiteInfo (#124)
- [322437d9](https://github.com/stashed/apimachinery/commit/322437d9) Show actual repo size instead of logical size (#123)
- [5e773145](https://github.com/stashed/apimachinery/commit/5e773145) Return new labels by EnsureKubeDBIntegration function  (#122)
- [908628b7](https://github.com/stashed/apimachinery/commit/908628b7) Update repository config (#120)
- [a7994a77](https://github.com/stashed/apimachinery/commit/a7994a77) Add `ADDON_IMAGE` variable for Etcd Addons
- [16529919](https://github.com/stashed/apimachinery/commit/16529919) Run `make fmt`
- [78077d32](https://github.com/stashed/apimachinery/commit/78077d32) Merge branch 'master' into etcd-flag
- [163e7863](https://github.com/stashed/apimachinery/commit/163e7863) Set RESTIC_PROGRESS_FPS in restic wrapper shell
- [b0983010](https://github.com/stashed/apimachinery/commit/b0983010) Merge branch 'master' into progress-fps
- [72b73523](https://github.com/stashed/apimachinery/commit/72b73523) Log warning if Community License is used with non-demo namespace (#119)
- [2eecfe50](https://github.com/stashed/apimachinery/commit/2eecfe50) Merge branch 'master' into etcd-flag
- [b0141c12](https://github.com/stashed/apimachinery/commit/b0141c12) Add addon image flag
- [c1e37507](https://github.com/stashed/apimachinery/commit/c1e37507) Set RESTIC_PROGRESS_FPS in restic wrapper shell
- [fda29913](https://github.com/stashed/apimachinery/commit/fda29913) Merge pull request #116 from stashed/fix-cronjob-name-prefix
- [8cbe8b9d](https://github.com/stashed/apimachinery/commit/8cbe8b9d) Add constants for backup triggering CronJob
- [78e6bc8c](https://github.com/stashed/apimachinery/commit/78e6bc8c) Use official restic v0.12.1 (#115)
- [01704d74](https://github.com/stashed/apimachinery/commit/01704d74) Update repository config (#114)
- [d9f1899d](https://github.com/stashed/apimachinery/commit/d9f1899d) Update dependencies (#113)
- [639e36ec](https://github.com/stashed/apimachinery/commit/639e36ec) Update dependencies (#112)
- [ffb5a0c7](https://github.com/stashed/apimachinery/commit/ffb5a0c7) Update dependencies (#111)
- [cdbb8e52](https://github.com/stashed/apimachinery/commit/cdbb8e52) Update repository config (#110)
- [ae94693a](https://github.com/stashed/apimachinery/commit/ae94693a) Update repository config (#109)



## [stashed/cli](https://github.com/stashed/cli)

### [v0.16.0](https://github.com/stashed/cli/releases/tag/v0.16.0)

- [1fb811f](https://github.com/stashed/cli/commit/1fb811f) Prepare for release v0.16.0 (#142)
- [c1b8251](https://github.com/stashed/cli/commit/c1b8251) Fix jwt-go security vulnerability (#141)
- [23689bb](https://github.com/stashed/cli/commit/23689bb) Fix jwt-go security vulnerability (#140)
- [fc3ffb8](https://github.com/stashed/cli/commit/fc3ffb8) Set restic docker image tag from Makefile during build (#137)
- [56a8b85](https://github.com/stashed/cli/commit/56a8b85) Update dependencies to publish SiteInfo (#139)
- [d8df9e8](https://github.com/stashed/cli/commit/d8df9e8) Update dependencies to publish SiteInfo (#138)
- [c7fb370](https://github.com/stashed/cli/commit/c7fb370) Log warning if Community License is used with non-demo namespace (#136)
- [11397c0](https://github.com/stashed/cli/commit/11397c0) Use official restic v0.12.1 (#135)
- [24ae84f](https://github.com/stashed/cli/commit/24ae84f) Update dependencies (#134)
- [91ab122](https://github.com/stashed/cli/commit/91ab122) Update repository config (#133)
- [47ca564](https://github.com/stashed/cli/commit/47ca564) Update dependencies (#132)
- [3d1a293](https://github.com/stashed/cli/commit/3d1a293) Update dependencies (#131)
- [f9b4bcc](https://github.com/stashed/cli/commit/f9b4bcc) Update dependencies (#130)
- [b2331c7](https://github.com/stashed/cli/commit/b2331c7) Update README.md



## [stashed/elasticsearch](https://github.com/stashed/elasticsearch)

### [5.6.4-v13](https://github.com/stashed/elasticsearch/releases/tag/5.6.4-v13)

- [a5d62837](https://github.com/stashed/elasticsearch/commit/a5d62837) Prepare for release 5.6.4-v13 (#1008)
- [3ef20d48](https://github.com/stashed/elasticsearch/commit/3ef20d48) [cherry-pick] Fix jwt-go security vulnerability (#999) (#1000)
- [39c89194](https://github.com/stashed/elasticsearch/commit/39c89194) [cherry-pick] Fix jwt-go security vulnerability (#990) (#991)
- [574f3b1e](https://github.com/stashed/elasticsearch/commit/574f3b1e) [cherry-pick] Update dependencies to publish SiteInfo (#981) (#982)
- [ec0d1986](https://github.com/stashed/elasticsearch/commit/ec0d1986) [cherry-pick] Update dependencies to publish SiteInfo (#972) (#973)
- [55047964](https://github.com/stashed/elasticsearch/commit/55047964) [cherry-pick] Log warning if Community License is used with non-demo namespace (#963) (#964)
- [18921116](https://github.com/stashed/elasticsearch/commit/18921116) [cherry-pick] Use official restic v0.12.1 (#954) (#955)
- [2b482d40](https://github.com/stashed/elasticsearch/commit/2b482d40) [cherry-pick] Update repository config (#945) (#946)
- [99db22ab](https://github.com/stashed/elasticsearch/commit/99db22ab) [cherry-pick] Update dependencies (#936) (#937)
- [e7432088](https://github.com/stashed/elasticsearch/commit/e7432088) [cherry-pick] Update dependencies (#927) (#928)
- [d996b55c](https://github.com/stashed/elasticsearch/commit/d996b55c) [cherry-pick] Update dependencies (#918) (#919)
- [6f441147](https://github.com/stashed/elasticsearch/commit/6f441147) [cherry-pick] Use user nobody (#910)
- [82d15a3c](https://github.com/stashed/elasticsearch/commit/82d15a3c) [cherry-pick] Update repository config (#901) (#902)
- [6f9313e9](https://github.com/stashed/elasticsearch/commit/6f9313e9) [cherry-pick] Update repository config (#892) (#893)
- [dce43d88](https://github.com/stashed/elasticsearch/commit/dce43d88) [cherry-pick] Update README.md (#884)


### [6.2.4-v13](https://github.com/stashed/elasticsearch/releases/tag/6.2.4-v13)

- [6a8aa63a](https://github.com/stashed/elasticsearch/commit/6a8aa63a) Prepare for release 6.2.4-v13 (#1009)
- [728caef7](https://github.com/stashed/elasticsearch/commit/728caef7) [cherry-pick] Fix jwt-go security vulnerability (#999) (#1001)
- [c3732a8d](https://github.com/stashed/elasticsearch/commit/c3732a8d) [cherry-pick] Fix jwt-go security vulnerability (#990) (#992)
- [4a661d0d](https://github.com/stashed/elasticsearch/commit/4a661d0d) [cherry-pick] Update dependencies to publish SiteInfo (#981) (#983)
- [b25f24a1](https://github.com/stashed/elasticsearch/commit/b25f24a1) [cherry-pick] Update dependencies to publish SiteInfo (#972) (#974)
- [34810993](https://github.com/stashed/elasticsearch/commit/34810993) [cherry-pick] Log warning if Community License is used with non-demo namespace (#963) (#965)
- [e712025c](https://github.com/stashed/elasticsearch/commit/e712025c) [cherry-pick] Use official restic v0.12.1 (#954) (#956)
- [5dc9f263](https://github.com/stashed/elasticsearch/commit/5dc9f263) [cherry-pick] Update repository config (#945) (#947)
- [dfe8fea8](https://github.com/stashed/elasticsearch/commit/dfe8fea8) [cherry-pick] Update dependencies (#936) (#938)
- [80b9a9e1](https://github.com/stashed/elasticsearch/commit/80b9a9e1) [cherry-pick] Update dependencies (#927) (#929)
- [fb0de16f](https://github.com/stashed/elasticsearch/commit/fb0de16f) [cherry-pick] Update dependencies (#918) (#920)
- [5fe518c1](https://github.com/stashed/elasticsearch/commit/5fe518c1) [cherry-pick] Use user nobody (#911)
- [894726f9](https://github.com/stashed/elasticsearch/commit/894726f9) [cherry-pick] Update repository config (#901) (#903)
- [ce7890c4](https://github.com/stashed/elasticsearch/commit/ce7890c4) [cherry-pick] Update repository config (#892) (#894)
- [26be85b5](https://github.com/stashed/elasticsearch/commit/26be85b5) [cherry-pick] Update README.md (#885)


### [6.3.0-v13](https://github.com/stashed/elasticsearch/releases/tag/6.3.0-v13)

- [44e2c7ff](https://github.com/stashed/elasticsearch/commit/44e2c7ff) Prepare for release 6.3.0-v13 (#1010)
- [dcddfb30](https://github.com/stashed/elasticsearch/commit/dcddfb30) [cherry-pick] Fix jwt-go security vulnerability (#999) (#1002)
- [c1dc901b](https://github.com/stashed/elasticsearch/commit/c1dc901b) [cherry-pick] Fix jwt-go security vulnerability (#990) (#993)
- [40735b41](https://github.com/stashed/elasticsearch/commit/40735b41) [cherry-pick] Update dependencies to publish SiteInfo (#981) (#984)
- [06771a08](https://github.com/stashed/elasticsearch/commit/06771a08) [cherry-pick] Update dependencies to publish SiteInfo (#972) (#975)
- [391387dd](https://github.com/stashed/elasticsearch/commit/391387dd) [cherry-pick] Log warning if Community License is used with non-demo namespace (#963) (#966)
- [45848135](https://github.com/stashed/elasticsearch/commit/45848135) [cherry-pick] Use official restic v0.12.1 (#954) (#957)
- [06071809](https://github.com/stashed/elasticsearch/commit/06071809) [cherry-pick] Update repository config (#945) (#948)
- [c0126494](https://github.com/stashed/elasticsearch/commit/c0126494) [cherry-pick] Update dependencies (#936) (#939)
- [43c57d7b](https://github.com/stashed/elasticsearch/commit/43c57d7b) [cherry-pick] Update dependencies (#927) (#930)
- [48f21786](https://github.com/stashed/elasticsearch/commit/48f21786) [cherry-pick] Update dependencies (#918) (#921)
- [72ef8aa8](https://github.com/stashed/elasticsearch/commit/72ef8aa8) [cherry-pick] Use user nobody (#912)
- [dd22de7f](https://github.com/stashed/elasticsearch/commit/dd22de7f) [cherry-pick] Update repository config (#901) (#904)
- [24f633df](https://github.com/stashed/elasticsearch/commit/24f633df) [cherry-pick] Update repository config (#892) (#895)
- [5c79be0b](https://github.com/stashed/elasticsearch/commit/5c79be0b) [cherry-pick] Update README.md (#886)


### [6.4.0-v13](https://github.com/stashed/elasticsearch/releases/tag/6.4.0-v13)

- [707710bb](https://github.com/stashed/elasticsearch/commit/707710bb) Prepare for release 6.4.0-v13 (#1011)
- [77479b55](https://github.com/stashed/elasticsearch/commit/77479b55) [cherry-pick] Fix jwt-go security vulnerability (#999) (#1003)
- [fc6a5ced](https://github.com/stashed/elasticsearch/commit/fc6a5ced) [cherry-pick] Fix jwt-go security vulnerability (#990) (#994)
- [5969c9f8](https://github.com/stashed/elasticsearch/commit/5969c9f8) [cherry-pick] Update dependencies to publish SiteInfo (#981) (#985)
- [262a6e36](https://github.com/stashed/elasticsearch/commit/262a6e36) [cherry-pick] Update dependencies to publish SiteInfo (#972) (#976)
- [d38bd407](https://github.com/stashed/elasticsearch/commit/d38bd407) [cherry-pick] Log warning if Community License is used with non-demo namespace (#963) (#967)
- [3de0c455](https://github.com/stashed/elasticsearch/commit/3de0c455) [cherry-pick] Use official restic v0.12.1 (#954) (#958)
- [27182793](https://github.com/stashed/elasticsearch/commit/27182793) [cherry-pick] Update repository config (#945) (#949)
- [5d10b2a3](https://github.com/stashed/elasticsearch/commit/5d10b2a3) [cherry-pick] Update dependencies (#936) (#940)
- [e95882f5](https://github.com/stashed/elasticsearch/commit/e95882f5) [cherry-pick] Update dependencies (#927) (#931)
- [772ccfa0](https://github.com/stashed/elasticsearch/commit/772ccfa0) [cherry-pick] Update dependencies (#918) (#922)
- [aa751b23](https://github.com/stashed/elasticsearch/commit/aa751b23) [cherry-pick] Use user nobody (#913)
- [73980641](https://github.com/stashed/elasticsearch/commit/73980641) [cherry-pick] Update repository config (#901) (#905)
- [02407a60](https://github.com/stashed/elasticsearch/commit/02407a60) [cherry-pick] Update repository config (#892) (#896)
- [0265d8de](https://github.com/stashed/elasticsearch/commit/0265d8de) [cherry-pick] Update README.md (#887)


### [6.5.3-v13](https://github.com/stashed/elasticsearch/releases/tag/6.5.3-v13)

- [9d2383dd](https://github.com/stashed/elasticsearch/commit/9d2383dd) Prepare for release 6.5.3-v13 (#1012)
- [79fd12e8](https://github.com/stashed/elasticsearch/commit/79fd12e8) [cherry-pick] Fix jwt-go security vulnerability (#999) (#1004)
- [a2e64544](https://github.com/stashed/elasticsearch/commit/a2e64544) [cherry-pick] Fix jwt-go security vulnerability (#990) (#995)
- [abf9549c](https://github.com/stashed/elasticsearch/commit/abf9549c) [cherry-pick] Update dependencies to publish SiteInfo (#981) (#986)
- [b2b0b18e](https://github.com/stashed/elasticsearch/commit/b2b0b18e) [cherry-pick] Update dependencies to publish SiteInfo (#972) (#977)
- [faac9e7d](https://github.com/stashed/elasticsearch/commit/faac9e7d) [cherry-pick] Log warning if Community License is used with non-demo namespace (#963) (#968)
- [09962313](https://github.com/stashed/elasticsearch/commit/09962313) [cherry-pick] Use official restic v0.12.1 (#954) (#959)
- [c71ba67a](https://github.com/stashed/elasticsearch/commit/c71ba67a) [cherry-pick] Update repository config (#945) (#950)
- [58f1375a](https://github.com/stashed/elasticsearch/commit/58f1375a) [cherry-pick] Update dependencies (#936) (#941)
- [b5ed5ab1](https://github.com/stashed/elasticsearch/commit/b5ed5ab1) [cherry-pick] Update dependencies (#927) (#932)
- [ab50d2f2](https://github.com/stashed/elasticsearch/commit/ab50d2f2) [cherry-pick] Update dependencies (#918) (#923)
- [4d19922a](https://github.com/stashed/elasticsearch/commit/4d19922a) [cherry-pick] Use user nobody (#914)
- [dc86a75d](https://github.com/stashed/elasticsearch/commit/dc86a75d) [cherry-pick] Update repository config (#901) (#906)
- [66505f2b](https://github.com/stashed/elasticsearch/commit/66505f2b) [cherry-pick] Update repository config (#892) (#897)
- [637c07d9](https://github.com/stashed/elasticsearch/commit/637c07d9) [cherry-pick] Update README.md (#888)


### [6.8.0-v13](https://github.com/stashed/elasticsearch/releases/tag/6.8.0-v13)

- [db01c3b5](https://github.com/stashed/elasticsearch/commit/db01c3b5) Prepare for release 6.8.0-v13 (#1013)
- [5812e424](https://github.com/stashed/elasticsearch/commit/5812e424) [cherry-pick] Fix jwt-go security vulnerability (#999) (#1005)
- [20c76b9f](https://github.com/stashed/elasticsearch/commit/20c76b9f) [cherry-pick] Fix jwt-go security vulnerability (#990) (#996)
- [439837e1](https://github.com/stashed/elasticsearch/commit/439837e1) [cherry-pick] Update dependencies to publish SiteInfo (#981) (#987)
- [1899770a](https://github.com/stashed/elasticsearch/commit/1899770a) [cherry-pick] Update dependencies to publish SiteInfo (#972) (#978)
- [4ca69b25](https://github.com/stashed/elasticsearch/commit/4ca69b25) [cherry-pick] Log warning if Community License is used with non-demo namespace (#963) (#969)
- [0612ea32](https://github.com/stashed/elasticsearch/commit/0612ea32) [cherry-pick] Use official restic v0.12.1 (#954) (#960)
- [b832eeba](https://github.com/stashed/elasticsearch/commit/b832eeba) [cherry-pick] Update repository config (#945) (#951)
- [0438c6cd](https://github.com/stashed/elasticsearch/commit/0438c6cd) [cherry-pick] Update dependencies (#936) (#942)
- [a2f87a3c](https://github.com/stashed/elasticsearch/commit/a2f87a3c) [cherry-pick] Update dependencies (#927) (#933)
- [08c0717e](https://github.com/stashed/elasticsearch/commit/08c0717e) [cherry-pick] Update dependencies (#918) (#924)
- [04cc7905](https://github.com/stashed/elasticsearch/commit/04cc7905) [cherry-pick] Use user nobody (#915)
- [a9f017dc](https://github.com/stashed/elasticsearch/commit/a9f017dc) [cherry-pick] Update repository config (#901) (#907)
- [4f505600](https://github.com/stashed/elasticsearch/commit/4f505600) [cherry-pick] Update repository config (#892) (#898)
- [1a64b3ee](https://github.com/stashed/elasticsearch/commit/1a64b3ee) [cherry-pick] Update README.md (#889)


### [7.2.0-v13](https://github.com/stashed/elasticsearch/releases/tag/7.2.0-v13)

- [55abc7a5](https://github.com/stashed/elasticsearch/commit/55abc7a5) Prepare for release 7.2.0-v13 (#1014)
- [aa2b8e05](https://github.com/stashed/elasticsearch/commit/aa2b8e05) [cherry-pick] Fix jwt-go security vulnerability (#999) (#1006)
- [8b4d6db6](https://github.com/stashed/elasticsearch/commit/8b4d6db6) [cherry-pick] Fix jwt-go security vulnerability (#990) (#997)
- [d7fb74a3](https://github.com/stashed/elasticsearch/commit/d7fb74a3) [cherry-pick] Update dependencies to publish SiteInfo (#981) (#988)
- [fc8fd676](https://github.com/stashed/elasticsearch/commit/fc8fd676) Update dependencies to publish SiteInfo (#972) (#979)
- [7a3b314c](https://github.com/stashed/elasticsearch/commit/7a3b314c) [cherry-pick] Log warning if Community License is used with non-demo namespace (#963) (#970)
- [d37ac3ba](https://github.com/stashed/elasticsearch/commit/d37ac3ba) [cherry-pick] Use official restic v0.12.1 (#954) (#961)
- [bf69bd17](https://github.com/stashed/elasticsearch/commit/bf69bd17) [cherry-pick] Update repository config (#945) (#952)
- [7b9d74d0](https://github.com/stashed/elasticsearch/commit/7b9d74d0) [cherry-pick] Update dependencies (#936) (#943)
- [50dd7353](https://github.com/stashed/elasticsearch/commit/50dd7353) [cherry-pick] Update dependencies (#927) (#934)
- [bc45e6a3](https://github.com/stashed/elasticsearch/commit/bc45e6a3) [cherry-pick] Update dependencies (#918) (#925)
- [7ef1c360](https://github.com/stashed/elasticsearch/commit/7ef1c360) [cherry-pick] Use user nobody (#916)
- [1ddfc17c](https://github.com/stashed/elasticsearch/commit/1ddfc17c) [cherry-pick] Update repository config (#901) (#908)
- [87806973](https://github.com/stashed/elasticsearch/commit/87806973) [cherry-pick] Update repository config (#892) (#899)
- [c30a871f](https://github.com/stashed/elasticsearch/commit/c30a871f) [cherry-pick] Update README.md (#890)


### [7.3.2-v13](https://github.com/stashed/elasticsearch/releases/tag/7.3.2-v13)

- [f5019d83](https://github.com/stashed/elasticsearch/commit/f5019d83) Prepare for release 7.3.2-v13 (#1015)
- [5e2009c0](https://github.com/stashed/elasticsearch/commit/5e2009c0) [cherry-pick] Fix jwt-go security vulnerability (#999) (#1007)
- [2aa85efa](https://github.com/stashed/elasticsearch/commit/2aa85efa) [cherry-pick] Fix jwt-go security vulnerability (#990) (#998)
- [bd6bd091](https://github.com/stashed/elasticsearch/commit/bd6bd091) [cherry-pick] Update dependencies to publish SiteInfo (#981) (#989)
- [50138325](https://github.com/stashed/elasticsearch/commit/50138325) [cherry-pick] Update dependencies to publish SiteInfo (#972) (#980)
- [a8647cc3](https://github.com/stashed/elasticsearch/commit/a8647cc3) [cherry-pick] Log warning if Community License is used with non-demo namespace (#963) (#971)
- [317f6480](https://github.com/stashed/elasticsearch/commit/317f6480) [cherry-pick] Use official restic v0.12.1 (#954) (#962)
- [c237ef12](https://github.com/stashed/elasticsearch/commit/c237ef12) [cherry-pick] Update repository config (#945) (#953)
- [db0796c2](https://github.com/stashed/elasticsearch/commit/db0796c2) [cherry-pick] Update dependencies (#936) (#944)
- [b35640e9](https://github.com/stashed/elasticsearch/commit/b35640e9) [cherry-pick] Update dependencies (#927) (#935)
- [d7364e5f](https://github.com/stashed/elasticsearch/commit/d7364e5f) [cherry-pick] Update dependencies (#918) (#926)
- [b6bc1e06](https://github.com/stashed/elasticsearch/commit/b6bc1e06) [cherry-pick] Use user nobody (#917)
- [8f275fc7](https://github.com/stashed/elasticsearch/commit/8f275fc7) [cherry-pick] Update repository config (#901) (#909)
- [204da528](https://github.com/stashed/elasticsearch/commit/204da528) [cherry-pick] Update repository config (#892) (#900)
- [17519e39](https://github.com/stashed/elasticsearch/commit/17519e39) [cherry-pick] Update README.md (#891)



## [stashed/etcd](https://github.com/stashed/etcd)

### [3.5.0](https://github.com/stashed/etcd/releases/tag/3.5.0)

- [4c83a10](https://github.com/stashed/etcd/commit/4c83a10) Prepare for release 3.5.0 (#9)
- [1dbf3b1](https://github.com/stashed/etcd/commit/1dbf3b1) Fix jwt-go security vulnerability (#7) (#8)
- [fbcc997](https://github.com/stashed/etcd/commit/fbcc997) Fix jwt-go security vulnerability (#5) (#6)
- [e0919f1](https://github.com/stashed/etcd/commit/e0919f1) Use restic 0.12.1 (#4)
- [f7b65eb](https://github.com/stashed/etcd/commit/f7b65eb) Update repository config (#3)
- [a2e9055](https://github.com/stashed/etcd/commit/a2e9055) Add etcd v3.5.0 addon to Stash (#2)
- [517d580](https://github.com/stashed/etcd/commit/517d580) Merge pull request #1 from stashed/global-replace
- [643ff4b](https://github.com/stashed/etcd/commit/643ff4b) Global replace Redis with Etcd
- [6d880e0](https://github.com/stashed/etcd/commit/6d880e0) Use user nobody
- [5252385](https://github.com/stashed/etcd/commit/5252385) Update repository config (#11)
- [f6dc027](https://github.com/stashed/etcd/commit/f6dc027) Update repository config (#8)
- [d68710e](https://github.com/stashed/etcd/commit/d68710e) Prepare for release v2021.08.02 (#7)
- [1772bc7](https://github.com/stashed/etcd/commit/1772bc7) Update repository config (#2)
- [7d4bfee](https://github.com/stashed/etcd/commit/7d4bfee) Make Redis addon ready to use (#1)



## [stashed/installer](https://github.com/stashed/installer)

### [v2021.10.11](https://github.com/stashed/installer/releases/tag/v2021.10.11)

- [0f063ad](https://github.com/stashed/installer/commit/0f063ad) Prepare for release v2021.10.11 (#210)
- [0092c7e](https://github.com/stashed/installer/commit/0092c7e) Fix jwt-go security vulnerability (#209)
- [17e1fdc](https://github.com/stashed/installer/commit/17e1fdc) Fix jwt-go security vulnerability (#208)
- [01bf7cf](https://github.com/stashed/installer/commit/01bf7cf) Switch from nats 2.4.0 to 2.6.1
- [30c5a63](https://github.com/stashed/installer/commit/30c5a63) Add etcd v3.5.0 addon to Stash (#195)
- [a088dcd](https://github.com/stashed/installer/commit/a088dcd) Add permission for SiteInfo publisher
- [a38d2b3](https://github.com/stashed/installer/commit/a38d2b3) Add mongodb `5.0.3` (#207)
- [bb1d956](https://github.com/stashed/installer/commit/bb1d956) Update dependencies to publish SiteInfo (#206)
- [48c528c](https://github.com/stashed/installer/commit/48c528c) Add Backup and Restore support for Postgres  v14.0 (#205)
- [9ffc6aa](https://github.com/stashed/installer/commit/9ffc6aa) Add stash-metrics chart (#202)
- [0610818](https://github.com/stashed/installer/commit/0610818) Update repository config (#204)
- [7d3116e](https://github.com/stashed/installer/commit/7d3116e) Log warning if Community License is used with non-demo namespace (#203)
- [f0da2b3](https://github.com/stashed/installer/commit/f0da2b3) Use official restic v0.12.1 (#201)
- [01ecae5](https://github.com/stashed/installer/commit/01ecae5) Add mongodb 5.0.2 backup tasks (#200)
- [37eec67](https://github.com/stashed/installer/commit/37eec67) Add catalogs for NATS addon (#196)
- [9d585cd](https://github.com/stashed/installer/commit/9d585cd) Update repository config (#199)
- [ba6c6e8](https://github.com/stashed/installer/commit/ba6c6e8) Update dependencies (#198)
- [cd8b2d0](https://github.com/stashed/installer/commit/cd8b2d0) Update dependencies (#197)
- [9d47a48](https://github.com/stashed/installer/commit/9d47a48) Update repository config (#194)
- [8775569](https://github.com/stashed/installer/commit/8775569) Update repository config (#193)
- [715ac99](https://github.com/stashed/installer/commit/715ac99) Update README.md



## [stashed/mariadb](https://github.com/stashed/mariadb)

### [10.5.8-v6](https://github.com/stashed/mariadb/releases/tag/10.5.8-v6)

- [a4773a3](https://github.com/stashed/mariadb/commit/a4773a3) Prepare for release 10.5.8-v6 (#148)
- [eca6402](https://github.com/stashed/mariadb/commit/eca6402) [cherry-pick] Fix jwt-go security vulnerability (#146) (#147)
- [9570bdf](https://github.com/stashed/mariadb/commit/9570bdf) [cherry-pick] Fix jwt-go security vulnerability (#144) (#145)
- [f18b738](https://github.com/stashed/mariadb/commit/f18b738) Update dependencies to publish SiteInfo (#142) (#143)
- [df6e78c](https://github.com/stashed/mariadb/commit/df6e78c) [cherry-pick] Update dependencies to publish SiteInfo (#140) (#141)
- [3de3692](https://github.com/stashed/mariadb/commit/3de3692) [cherry-pick] Log warning if Community License is used with non-demo namespace (#138) (#139)
- [0f17ca3](https://github.com/stashed/mariadb/commit/0f17ca3) [cherry-pick] Use official restic v0.12.1 (#136) (#137)
- [197f346](https://github.com/stashed/mariadb/commit/197f346) [cherry-pick] Update repository config (#134) (#135)
- [7bc7aed](https://github.com/stashed/mariadb/commit/7bc7aed) [cherry-pick] Update dependencies (#132) (#133)
- [210ba94](https://github.com/stashed/mariadb/commit/210ba94) [cherry-pick] Update dependencies (#130) (#131)
- [1f29ad8](https://github.com/stashed/mariadb/commit/1f29ad8) [cherry-pick] Update dependencies (#128) (#129)
- [7342a45](https://github.com/stashed/mariadb/commit/7342a45) [cherry-pick] Use user nobody (#127)
- [3c1f773](https://github.com/stashed/mariadb/commit/3c1f773) [cherry-pick] Update repository config (#125) (#126)
- [a54da2f](https://github.com/stashed/mariadb/commit/a54da2f) [cherry-pick] Update repository config (#123) (#124)
- [ce30084](https://github.com/stashed/mariadb/commit/ce30084) [cherry-pick] Update README.md (#122)



## [stashed/mongodb](https://github.com/stashed/mongodb)

### [3.4.17-v12](https://github.com/stashed/mongodb/releases/tag/3.4.17-v12)

- [69628a86](https://github.com/stashed/mongodb/commit/69628a86) Prepare for release 3.4.17-v12 (#1264)
- [97ba7285](https://github.com/stashed/mongodb/commit/97ba7285) [cherry-pick] Fix jwt-go security vulnerability (#1250) (#1251)
- [325a3bd0](https://github.com/stashed/mongodb/commit/325a3bd0) [cherry-pick] Fix jwt-go security vulnerability (#1236) (#1237)
- [22a5fab9](https://github.com/stashed/mongodb/commit/22a5fab9) [cherry-pick] Update dependencies to publish SiteInfo (#1222) (#1223)
- [8bf670e1](https://github.com/stashed/mongodb/commit/8bf670e1) [cherry-pick] Update dependencies to publish SiteInfo (#1208) (#1209)
- [a431846c](https://github.com/stashed/mongodb/commit/a431846c) [cherry-pick] Log warning if Community License is used with non-demo namespace (#1194) (#1195)
- [c99cad39](https://github.com/stashed/mongodb/commit/c99cad39) [cherry-pick] Use official restic v0.12.1 (#1180) (#1181)
- [b0aefc96](https://github.com/stashed/mongodb/commit/b0aefc96) [cherry-pick] Update repository config (#1166) (#1167)
- [b930d20d](https://github.com/stashed/mongodb/commit/b930d20d) [cherry-pick] Update dependencies (#1152) (#1153)
- [56417b38](https://github.com/stashed/mongodb/commit/56417b38) [cherry-pick] Update dependencies (#1138) (#1139)
- [7bdd0832](https://github.com/stashed/mongodb/commit/7bdd0832) [cherry-pick] Update dependencies (#1124) (#1125)
- [867a70f1](https://github.com/stashed/mongodb/commit/867a70f1) [cherry-pick] Use user nobody (#1112)
- [528e1529](https://github.com/stashed/mongodb/commit/528e1529) [cherry-pick] Update repository config (#1099) (#1100)
- [0606c894](https://github.com/stashed/mongodb/commit/0606c894) [cherry-pick] Update repository config (#1086) (#1087)
- [2a9d8768](https://github.com/stashed/mongodb/commit/2a9d8768) [cherry-pick] Update README.md (#1074)


### [3.4.22-v12](https://github.com/stashed/mongodb/releases/tag/3.4.22-v12)

- [06b5f60f](https://github.com/stashed/mongodb/commit/06b5f60f) Prepare for release 3.4.22-v12 (#1265)
- [7c5cf34a](https://github.com/stashed/mongodb/commit/7c5cf34a) [cherry-pick] Fix jwt-go security vulnerability (#1250) (#1252)
- [ec1b6cd9](https://github.com/stashed/mongodb/commit/ec1b6cd9) [cherry-pick] Fix jwt-go security vulnerability (#1236) (#1238)
- [bfef68dd](https://github.com/stashed/mongodb/commit/bfef68dd) [cherry-pick] Update dependencies to publish SiteInfo (#1222) (#1224)
- [0360c8a4](https://github.com/stashed/mongodb/commit/0360c8a4) [cherry-pick] Update dependencies to publish SiteInfo (#1208) (#1210)
- [b3978742](https://github.com/stashed/mongodb/commit/b3978742) [cherry-pick] Log warning if Community License is used with non-demo namespace (#1194) (#1196)
- [076e1c7f](https://github.com/stashed/mongodb/commit/076e1c7f) [cherry-pick] Use official restic v0.12.1 (#1180) (#1182)
- [5c2a16bc](https://github.com/stashed/mongodb/commit/5c2a16bc) [cherry-pick] Update repository config (#1166) (#1168)
- [ad1058d8](https://github.com/stashed/mongodb/commit/ad1058d8) [cherry-pick] Update dependencies (#1152) (#1154)
- [1d0b2d12](https://github.com/stashed/mongodb/commit/1d0b2d12) [cherry-pick] Update dependencies (#1138) (#1140)
- [9014381f](https://github.com/stashed/mongodb/commit/9014381f) [cherry-pick] Update dependencies (#1124) (#1126)
- [3ce9f548](https://github.com/stashed/mongodb/commit/3ce9f548) [cherry-pick] Use user nobody (#1113)
- [9d1a648b](https://github.com/stashed/mongodb/commit/9d1a648b) [cherry-pick] Update repository config (#1099) (#1101)
- [ebd987d5](https://github.com/stashed/mongodb/commit/ebd987d5) [cherry-pick] Update repository config (#1086) (#1088)
- [58baff2d](https://github.com/stashed/mongodb/commit/58baff2d) [cherry-pick] Update README.md (#1075)


### [3.6.8-v12](https://github.com/stashed/mongodb/releases/tag/3.6.8-v12)

- [37362a0a](https://github.com/stashed/mongodb/commit/37362a0a) Prepare for release 3.6.8-v12 (#1267)
- [3638de4c](https://github.com/stashed/mongodb/commit/3638de4c) [cherry-pick] Fix jwt-go security vulnerability (#1250) (#1254)
- [ffd0e227](https://github.com/stashed/mongodb/commit/ffd0e227) [cherry-pick] Fix jwt-go security vulnerability (#1236) (#1240)
- [f2b1f979](https://github.com/stashed/mongodb/commit/f2b1f979) [cherry-pick] Update dependencies to publish SiteInfo (#1222) (#1226)
- [d4b8f157](https://github.com/stashed/mongodb/commit/d4b8f157) [cherry-pick] Update dependencies to publish SiteInfo (#1208) (#1212)
- [17c91aa8](https://github.com/stashed/mongodb/commit/17c91aa8) [cherry-pick] Log warning if Community License is used with non-demo namespace (#1194) (#1198)
- [4b2717a0](https://github.com/stashed/mongodb/commit/4b2717a0) [cherry-pick] Use official restic v0.12.1 (#1180) (#1184)
- [3f0f514c](https://github.com/stashed/mongodb/commit/3f0f514c) [cherry-pick] Update repository config (#1166) (#1170)
- [faba0d40](https://github.com/stashed/mongodb/commit/faba0d40) [cherry-pick] Update dependencies (#1152) (#1156)
- [62300371](https://github.com/stashed/mongodb/commit/62300371) [cherry-pick] Update dependencies (#1138) (#1142)
- [f4075d20](https://github.com/stashed/mongodb/commit/f4075d20) [cherry-pick] Update dependencies (#1124) (#1128)
- [24e77c90](https://github.com/stashed/mongodb/commit/24e77c90) [cherry-pick] Use user nobody (#1115)
- [386140af](https://github.com/stashed/mongodb/commit/386140af) [cherry-pick] Update repository config (#1099) (#1103)
- [f69c1d0f](https://github.com/stashed/mongodb/commit/f69c1d0f) [cherry-pick] Update repository config (#1086) (#1090)
- [86c0b41b](https://github.com/stashed/mongodb/commit/86c0b41b) [cherry-pick] Update README.md (#1077)


### [3.6.13-v12](https://github.com/stashed/mongodb/releases/tag/3.6.13-v12)

- [532e72c7](https://github.com/stashed/mongodb/commit/532e72c7) Prepare for release 3.6.13-v12 (#1266)
- [b62cf94b](https://github.com/stashed/mongodb/commit/b62cf94b) [cherry-pick] Fix jwt-go security vulnerability (#1250) (#1253)
- [644f22b3](https://github.com/stashed/mongodb/commit/644f22b3) [cherry-pick] Fix jwt-go security vulnerability (#1236) (#1239)
- [dd2424be](https://github.com/stashed/mongodb/commit/dd2424be) [cherry-pick] Update dependencies to publish SiteInfo (#1222) (#1225)
- [c5b0e852](https://github.com/stashed/mongodb/commit/c5b0e852) [cherry-pick] Update dependencies to publish SiteInfo (#1208) (#1211)
- [b9e008b9](https://github.com/stashed/mongodb/commit/b9e008b9) [cherry-pick] Log warning if Community License is used with non-demo namespace (#1194) (#1197)
- [7e832b9d](https://github.com/stashed/mongodb/commit/7e832b9d) [cherry-pick] Use official restic v0.12.1 (#1180) (#1183)
- [9c515527](https://github.com/stashed/mongodb/commit/9c515527) [cherry-pick] Update repository config (#1166) (#1169)
- [6e59e006](https://github.com/stashed/mongodb/commit/6e59e006) [cherry-pick] Update dependencies (#1152) (#1155)
- [f168d3f3](https://github.com/stashed/mongodb/commit/f168d3f3) [cherry-pick] Update dependencies (#1138) (#1141)
- [e758e8e0](https://github.com/stashed/mongodb/commit/e758e8e0) [cherry-pick] Update dependencies (#1124) (#1127)
- [8a8d96e7](https://github.com/stashed/mongodb/commit/8a8d96e7) [cherry-pick] Use user nobody (#1114)
- [45ab9d0c](https://github.com/stashed/mongodb/commit/45ab9d0c) [cherry-pick] Update repository config (#1099) (#1102)
- [22e5dd21](https://github.com/stashed/mongodb/commit/22e5dd21) [cherry-pick] Update repository config (#1086) (#1089)
- [49dab901](https://github.com/stashed/mongodb/commit/49dab901) [cherry-pick] Update README.md (#1076)


### [4.0.3-v12](https://github.com/stashed/mongodb/releases/tag/4.0.3-v12)

- [4f36edfd](https://github.com/stashed/mongodb/commit/4f36edfd) Prepare for release 4.0.3-v12 (#1269)
- [bea5ee27](https://github.com/stashed/mongodb/commit/bea5ee27) [cherry-pick] Fix jwt-go security vulnerability (#1250) (#1256)
- [bb787054](https://github.com/stashed/mongodb/commit/bb787054) [cherry-pick] Fix jwt-go security vulnerability (#1236) (#1242)
- [ac0bfd12](https://github.com/stashed/mongodb/commit/ac0bfd12) [cherry-pick] Update dependencies to publish SiteInfo (#1222) (#1228)
- [31c7d66c](https://github.com/stashed/mongodb/commit/31c7d66c) [cherry-pick] Update dependencies to publish SiteInfo (#1208) (#1214)
- [ce07bdd7](https://github.com/stashed/mongodb/commit/ce07bdd7) [cherry-pick] Log warning if Community License is used with non-demo namespace (#1194) (#1200)
- [102601ee](https://github.com/stashed/mongodb/commit/102601ee) [cherry-pick] Use official restic v0.12.1 (#1180) (#1186)
- [e75754f8](https://github.com/stashed/mongodb/commit/e75754f8) [cherry-pick] Update repository config (#1166) (#1172)
- [a069745a](https://github.com/stashed/mongodb/commit/a069745a) [cherry-pick] Update dependencies (#1152) (#1158)
- [4e7a1e24](https://github.com/stashed/mongodb/commit/4e7a1e24) [cherry-pick] Update dependencies (#1138) (#1144)
- [80a75e9c](https://github.com/stashed/mongodb/commit/80a75e9c) [cherry-pick] Update dependencies (#1124) (#1130)
- [319a4947](https://github.com/stashed/mongodb/commit/319a4947) [cherry-pick] Use user nobody (#1117)
- [bcb3cfd2](https://github.com/stashed/mongodb/commit/bcb3cfd2) [cherry-pick] Update repository config (#1099) (#1105)
- [a05dce30](https://github.com/stashed/mongodb/commit/a05dce30) [cherry-pick] Update repository config (#1086) (#1092)
- [dd5135b5](https://github.com/stashed/mongodb/commit/dd5135b5) [cherry-pick] Update README.md (#1079)


### [4.0.5-v12](https://github.com/stashed/mongodb/releases/tag/4.0.5-v12)

- [6b691d1f](https://github.com/stashed/mongodb/commit/6b691d1f) Prepare for release 4.0.5-v12 (#1270)
- [39f10575](https://github.com/stashed/mongodb/commit/39f10575) [cherry-pick] Fix jwt-go security vulnerability (#1250) (#1257)
- [53beff3b](https://github.com/stashed/mongodb/commit/53beff3b) [cherry-pick] Fix jwt-go security vulnerability (#1236) (#1243)
- [136c6960](https://github.com/stashed/mongodb/commit/136c6960) [cherry-pick] Update dependencies to publish SiteInfo (#1222) (#1229)
- [6207c465](https://github.com/stashed/mongodb/commit/6207c465) [cherry-pick] Update dependencies to publish SiteInfo (#1208) (#1215)
- [e7e139d3](https://github.com/stashed/mongodb/commit/e7e139d3) [cherry-pick] Log warning if Community License is used with non-demo namespace (#1194) (#1201)
- [30a94da7](https://github.com/stashed/mongodb/commit/30a94da7) [cherry-pick] Use official restic v0.12.1 (#1180) (#1187)
- [f59c5a6c](https://github.com/stashed/mongodb/commit/f59c5a6c) [cherry-pick] Update repository config (#1166) (#1173)
- [dd14094c](https://github.com/stashed/mongodb/commit/dd14094c) [cherry-pick] Update dependencies (#1152) (#1159)
- [636296f1](https://github.com/stashed/mongodb/commit/636296f1) [cherry-pick] Update dependencies (#1138) (#1145)
- [0ccc1b8b](https://github.com/stashed/mongodb/commit/0ccc1b8b) [cherry-pick] Update dependencies (#1124) (#1131)
- [43dc654f](https://github.com/stashed/mongodb/commit/43dc654f) [cherry-pick] Use user nobody (#1118)
- [099f6ea5](https://github.com/stashed/mongodb/commit/099f6ea5) [cherry-pick] Update repository config (#1099) (#1106)
- [6595c0cb](https://github.com/stashed/mongodb/commit/6595c0cb) [cherry-pick] Update repository config (#1086) (#1093)
- [b18af069](https://github.com/stashed/mongodb/commit/b18af069) [cherry-pick] Update README.md (#1080)


### [4.0.11-v12](https://github.com/stashed/mongodb/releases/tag/4.0.11-v12)

- [5ae4abe4](https://github.com/stashed/mongodb/commit/5ae4abe4) Prepare for release 4.0.11-v12 (#1268)
- [86255e75](https://github.com/stashed/mongodb/commit/86255e75) [cherry-pick] Fix jwt-go security vulnerability (#1250) (#1255)
- [bd4ed817](https://github.com/stashed/mongodb/commit/bd4ed817) [cherry-pick] Fix jwt-go security vulnerability (#1236) (#1241)
- [079e1c84](https://github.com/stashed/mongodb/commit/079e1c84) [cherry-pick] Update dependencies to publish SiteInfo (#1222) (#1227)
- [872c68cd](https://github.com/stashed/mongodb/commit/872c68cd) [cherry-pick] Update dependencies to publish SiteInfo (#1208) (#1213)
- [8407c055](https://github.com/stashed/mongodb/commit/8407c055) [cherry-pick] Log warning if Community License is used with non-demo namespace (#1194) (#1199)
- [223707c8](https://github.com/stashed/mongodb/commit/223707c8) [cherry-pick] Use official restic v0.12.1 (#1180) (#1185)
- [cc711ac3](https://github.com/stashed/mongodb/commit/cc711ac3) [cherry-pick] Update repository config (#1166) (#1171)
- [2581d408](https://github.com/stashed/mongodb/commit/2581d408) [cherry-pick] Update dependencies (#1152) (#1157)
- [243f28a0](https://github.com/stashed/mongodb/commit/243f28a0) [cherry-pick] Update dependencies (#1138) (#1143)
- [b706066b](https://github.com/stashed/mongodb/commit/b706066b) [cherry-pick] Update dependencies (#1124) (#1129)
- [20423b74](https://github.com/stashed/mongodb/commit/20423b74) [cherry-pick] Use user nobody (#1116)
- [d4b269d8](https://github.com/stashed/mongodb/commit/d4b269d8) [cherry-pick] Update repository config (#1099) (#1104)
- [f795ab01](https://github.com/stashed/mongodb/commit/f795ab01) [cherry-pick] Update repository config (#1086) (#1091)
- [9360ba4e](https://github.com/stashed/mongodb/commit/9360ba4e) [cherry-pick] Update README.md (#1078)


### [4.1.4-v12](https://github.com/stashed/mongodb/releases/tag/4.1.4-v12)

- [fb2cb972](https://github.com/stashed/mongodb/commit/fb2cb972) Prepare for release 4.1.4-v12 (#1272)
- [828c9c33](https://github.com/stashed/mongodb/commit/828c9c33) [cherry-pick] Fix jwt-go security vulnerability (#1250) (#1259)
- [f7421dfe](https://github.com/stashed/mongodb/commit/f7421dfe) [cherry-pick] Fix jwt-go security vulnerability (#1236) (#1245)
- [e7f3389f](https://github.com/stashed/mongodb/commit/e7f3389f) [cherry-pick] Update dependencies to publish SiteInfo (#1222) (#1231)
- [412cac39](https://github.com/stashed/mongodb/commit/412cac39) [cherry-pick] Update dependencies to publish SiteInfo (#1208) (#1217)
- [2885f7b7](https://github.com/stashed/mongodb/commit/2885f7b7) [cherry-pick] Log warning if Community License is used with non-demo namespace (#1194) (#1203)
- [45880326](https://github.com/stashed/mongodb/commit/45880326) [cherry-pick] Use official restic v0.12.1 (#1180) (#1189)
- [4e0e4040](https://github.com/stashed/mongodb/commit/4e0e4040) [cherry-pick] Update repository config (#1166) (#1175)
- [947563a4](https://github.com/stashed/mongodb/commit/947563a4) [cherry-pick] Update dependencies (#1152) (#1161)
- [a386cc6a](https://github.com/stashed/mongodb/commit/a386cc6a) [cherry-pick] Update dependencies (#1138) (#1147)
- [94a94df5](https://github.com/stashed/mongodb/commit/94a94df5) [cherry-pick] Update dependencies (#1124) (#1133)
- [b60fe1d7](https://github.com/stashed/mongodb/commit/b60fe1d7) [cherry-pick] Use user nobody (#1120)
- [90649dd7](https://github.com/stashed/mongodb/commit/90649dd7) [cherry-pick] Update repository config (#1099) (#1108)
- [147d0b08](https://github.com/stashed/mongodb/commit/147d0b08) [cherry-pick] Update repository config (#1086) (#1095)
- [570b7984](https://github.com/stashed/mongodb/commit/570b7984) [cherry-pick] Update README.md (#1082)


### [4.1.7-v12](https://github.com/stashed/mongodb/releases/tag/4.1.7-v12)

- [55d3315e](https://github.com/stashed/mongodb/commit/55d3315e) Prepare for release 4.1.7-v12 (#1273)
- [363724f2](https://github.com/stashed/mongodb/commit/363724f2) [cherry-pick] Fix jwt-go security vulnerability (#1250) (#1260)
- [a2ddb368](https://github.com/stashed/mongodb/commit/a2ddb368) [cherry-pick] Fix jwt-go security vulnerability (#1236) (#1246)
- [cb493707](https://github.com/stashed/mongodb/commit/cb493707) [cherry-pick] Update dependencies to publish SiteInfo (#1222) (#1232)
- [7eaac8e1](https://github.com/stashed/mongodb/commit/7eaac8e1) [cherry-pick] Update dependencies to publish SiteInfo (#1208) (#1218)
- [c93130b6](https://github.com/stashed/mongodb/commit/c93130b6) [cherry-pick] Log warning if Community License is used with non-demo namespace (#1194) (#1204)
- [604501cf](https://github.com/stashed/mongodb/commit/604501cf) [cherry-pick] Use official restic v0.12.1 (#1180) (#1190)
- [63f1385e](https://github.com/stashed/mongodb/commit/63f1385e) [cherry-pick] Update repository config (#1166) (#1176)
- [a530f03f](https://github.com/stashed/mongodb/commit/a530f03f) [cherry-pick] Update dependencies (#1152) (#1162)
- [910a4581](https://github.com/stashed/mongodb/commit/910a4581) [cherry-pick] Update dependencies (#1138) (#1148)
- [0cb4b9e3](https://github.com/stashed/mongodb/commit/0cb4b9e3) [cherry-pick] Update dependencies (#1124) (#1134)
- [aeab96a9](https://github.com/stashed/mongodb/commit/aeab96a9) [cherry-pick] Use user nobody (#1121)
- [8768b338](https://github.com/stashed/mongodb/commit/8768b338) [cherry-pick] Update repository config (#1099) (#1109)
- [c139f365](https://github.com/stashed/mongodb/commit/c139f365) [cherry-pick] Update repository config (#1086) (#1096)
- [3e57b806](https://github.com/stashed/mongodb/commit/3e57b806) [cherry-pick] Update README.md (#1083)


### [4.1.13-v12](https://github.com/stashed/mongodb/releases/tag/4.1.13-v12)

- [fd8c4934](https://github.com/stashed/mongodb/commit/fd8c4934) Prepare for release 4.1.13-v12 (#1271)
- [3aa25931](https://github.com/stashed/mongodb/commit/3aa25931) [cherry-pick] Fix jwt-go security vulnerability (#1250) (#1258)
- [671bb974](https://github.com/stashed/mongodb/commit/671bb974) [cherry-pick] Fix jwt-go security vulnerability (#1236) (#1244)
- [289a843b](https://github.com/stashed/mongodb/commit/289a843b) [cherry-pick] Update dependencies to publish SiteInfo (#1222) (#1230)
- [9efdc589](https://github.com/stashed/mongodb/commit/9efdc589) [cherry-pick] Update dependencies to publish SiteInfo (#1208) (#1216)
- [ae5780bb](https://github.com/stashed/mongodb/commit/ae5780bb) [cherry-pick] Log warning if Community License is used with non-demo namespace (#1194) (#1202)
- [6928de7f](https://github.com/stashed/mongodb/commit/6928de7f) [cherry-pick] Use official restic v0.12.1 (#1180) (#1188)
- [4d82ec2e](https://github.com/stashed/mongodb/commit/4d82ec2e) [cherry-pick] Update repository config (#1166) (#1174)
- [a76a5f8e](https://github.com/stashed/mongodb/commit/a76a5f8e) [cherry-pick] Update dependencies (#1152) (#1160)
- [1462c0c1](https://github.com/stashed/mongodb/commit/1462c0c1) [cherry-pick] Update dependencies (#1138) (#1146)
- [1ce4bb55](https://github.com/stashed/mongodb/commit/1ce4bb55) [cherry-pick] Update dependencies (#1124) (#1132)
- [0d7ee1bb](https://github.com/stashed/mongodb/commit/0d7ee1bb) [cherry-pick] Use user nobody (#1119)
- [60dd402b](https://github.com/stashed/mongodb/commit/60dd402b) [cherry-pick] Update repository config (#1099) (#1107)
- [76e4d9e0](https://github.com/stashed/mongodb/commit/76e4d9e0) [cherry-pick] Update repository config (#1086) (#1094)
- [887de5c0](https://github.com/stashed/mongodb/commit/887de5c0) [cherry-pick] Update README.md (#1081)


### [4.2.3-v12](https://github.com/stashed/mongodb/releases/tag/4.2.3-v12)

- [c584cacc](https://github.com/stashed/mongodb/commit/c584cacc) Prepare for release 4.2.3-v12 (#1274)
- [4fe9a550](https://github.com/stashed/mongodb/commit/4fe9a550) [cherry-pick] Fix jwt-go security vulnerability (#1250) (#1261)
- [a6a4b593](https://github.com/stashed/mongodb/commit/a6a4b593) [cherry-pick] Fix jwt-go security vulnerability (#1236) (#1247)
- [632f8e76](https://github.com/stashed/mongodb/commit/632f8e76) [cherry-pick] Update dependencies to publish SiteInfo (#1222) (#1233)
- [f4cbabdc](https://github.com/stashed/mongodb/commit/f4cbabdc) [cherry-pick] Update dependencies to publish SiteInfo (#1208) (#1219)
- [d8bbe636](https://github.com/stashed/mongodb/commit/d8bbe636) [cherry-pick] Log warning if Community License is used with non-demo namespace (#1194) (#1205)
- [3dffe923](https://github.com/stashed/mongodb/commit/3dffe923) [cherry-pick] Use official restic v0.12.1 (#1180) (#1191)
- [04f73554](https://github.com/stashed/mongodb/commit/04f73554) [cherry-pick] Update repository config (#1166) (#1177)
- [42cc8004](https://github.com/stashed/mongodb/commit/42cc8004) [cherry-pick] Update dependencies (#1152) (#1163)
- [a75b0f27](https://github.com/stashed/mongodb/commit/a75b0f27) [cherry-pick] Update dependencies (#1138) (#1149)
- [1446f267](https://github.com/stashed/mongodb/commit/1446f267) [cherry-pick] Update dependencies (#1124) (#1135)
- [c51ba8d9](https://github.com/stashed/mongodb/commit/c51ba8d9) [cherry-pick] Use user nobody (#1122)
- [ce4e30fd](https://github.com/stashed/mongodb/commit/ce4e30fd) [cherry-pick] Update repository config (#1099) (#1110)
- [4f0a3d52](https://github.com/stashed/mongodb/commit/4f0a3d52) [cherry-pick] Update repository config (#1086) (#1097)
- [5cfa63c2](https://github.com/stashed/mongodb/commit/5cfa63c2) [cherry-pick] Update README.md (#1084)


### [4.4.6-v3](https://github.com/stashed/mongodb/releases/tag/4.4.6-v3)

- [20eadfb9](https://github.com/stashed/mongodb/commit/20eadfb9) [cherry-pick] Fix jwt-go security vulnerability (#1250) (#1262)
- [c8175506](https://github.com/stashed/mongodb/commit/c8175506) [cherry-pick] Fix jwt-go security vulnerability (#1236) (#1248)
- [63e5fb56](https://github.com/stashed/mongodb/commit/63e5fb56) [cherry-pick] Update dependencies to publish SiteInfo (#1222) (#1234)
- [14c357e4](https://github.com/stashed/mongodb/commit/14c357e4) [cherry-pick] Update dependencies to publish SiteInfo (#1208) (#1220)
- [d8b33503](https://github.com/stashed/mongodb/commit/d8b33503) [cherry-pick] Log warning if Community License is used with non-demo namespace (#1194) (#1206)
- [73382bec](https://github.com/stashed/mongodb/commit/73382bec) [cherry-pick] Use official restic v0.12.1 (#1180) (#1192)
- [a48b9a92](https://github.com/stashed/mongodb/commit/a48b9a92) [cherry-pick] Update repository config (#1166) (#1178)
- [84480622](https://github.com/stashed/mongodb/commit/84480622) [cherry-pick] Update dependencies (#1152) (#1164)
- [4be17009](https://github.com/stashed/mongodb/commit/4be17009) [cherry-pick] Update dependencies (#1138) (#1150)
- [995b5d7a](https://github.com/stashed/mongodb/commit/995b5d7a) [cherry-pick] Update dependencies (#1124) (#1136)
- [915c10a3](https://github.com/stashed/mongodb/commit/915c10a3) [cherry-pick] Use user nobody (#1123)
- [aa7416c2](https://github.com/stashed/mongodb/commit/aa7416c2) [cherry-pick] Update repository config (#1099) (#1111)
- [fe1d8892](https://github.com/stashed/mongodb/commit/fe1d8892) [cherry-pick] Update repository config (#1086) (#1098)
- [3955f424](https://github.com/stashed/mongodb/commit/3955f424) [cherry-pick] Update README.md (#1085)
- [2061ae6f](https://github.com/stashed/mongodb/commit/2061ae6f) Prepare for release 4.4.6-v2 (#1072)


### [5.0.3](https://github.com/stashed/mongodb/releases/tag/5.0.3)




## [stashed/mysql](https://github.com/stashed/mysql)

### [5.7.25-v13](https://github.com/stashed/mysql/releases/tag/5.7.25-v13)

- [aff6e370](https://github.com/stashed/mysql/commit/aff6e370) Prepare for release 5.7.25-v13 (#528)
- [60ddc272](https://github.com/stashed/mysql/commit/60ddc272) [cherry-pick] Fix jwt-go security vulnerability (#523) (#524)
- [799720cb](https://github.com/stashed/mysql/commit/799720cb) [cherry-pick] Fix jwt-go security vulnerability (#518) (#519)
- [008b2bf5](https://github.com/stashed/mysql/commit/008b2bf5) [cherry-pick] Update dependencies to publish SiteInfo (#513) (#514)
- [fdedb6d1](https://github.com/stashed/mysql/commit/fdedb6d1) [cherry-pick] Update dependencies to publish SiteInfo (#508) (#509)
- [28533a79](https://github.com/stashed/mysql/commit/28533a79) [cherry-pick] Log warning if Community License is used with non-demo namespace (#503) (#504)
- [d905ccbb](https://github.com/stashed/mysql/commit/d905ccbb) [cherry-pick] Use official restic v0.12.1 (#498) (#499)
- [9582cc54](https://github.com/stashed/mysql/commit/9582cc54) [cherry-pick] Update repository config (#493) (#494)
- [0ae1d9a0](https://github.com/stashed/mysql/commit/0ae1d9a0) [cherry-pick] Update dependencies (#488) (#489)
- [07722857](https://github.com/stashed/mysql/commit/07722857) [cherry-pick] Update dependencies (#483) (#484)
- [2a177d7d](https://github.com/stashed/mysql/commit/2a177d7d) [cherry-pick] Update dependencies (#478) (#479)
- [fdf16b05](https://github.com/stashed/mysql/commit/fdf16b05) [cherry-pick] Use user nobody (#474)
- [6b54b57d](https://github.com/stashed/mysql/commit/6b54b57d) [cherry-pick] Update repository config (#469) (#470)
- [5c9a5e36](https://github.com/stashed/mysql/commit/5c9a5e36) [cherry-pick] Update repository config (#464) (#465)
- [7c10a203](https://github.com/stashed/mysql/commit/7c10a203) [cherry-pick] Update README.md (#460)


### [8.0.3-v13](https://github.com/stashed/mysql/releases/tag/8.0.3-v13)

- [eec051ee](https://github.com/stashed/mysql/commit/eec051ee) Prepare for release 8.0.3-v13 (#531)
- [1660e950](https://github.com/stashed/mysql/commit/1660e950) [cherry-pick] Fix jwt-go security vulnerability (#523) (#527)
- [71cf139d](https://github.com/stashed/mysql/commit/71cf139d) [cherry-pick] Fix jwt-go security vulnerability (#518) (#522)
- [0140594b](https://github.com/stashed/mysql/commit/0140594b) [cherry-pick] Update dependencies to publish SiteInfo (#513) (#517)
- [891c4a0c](https://github.com/stashed/mysql/commit/891c4a0c) [cherry-pick] Update dependencies to publish SiteInfo (#508) (#512)
- [5eb9b26f](https://github.com/stashed/mysql/commit/5eb9b26f) [cherry-pick] Log warning if Community License is used with non-demo namespace (#503) (#507)
- [e8f12efd](https://github.com/stashed/mysql/commit/e8f12efd) [cherry-pick] Use official restic v0.12.1 (#498) (#502)
- [0a607cb4](https://github.com/stashed/mysql/commit/0a607cb4) [cherry-pick] Update repository config (#493) (#497)
- [8d7a17ae](https://github.com/stashed/mysql/commit/8d7a17ae) [cherry-pick] Update dependencies (#488) (#492)
- [03c31af6](https://github.com/stashed/mysql/commit/03c31af6) [cherry-pick] Update dependencies (#483) (#487)
- [b4aacc9c](https://github.com/stashed/mysql/commit/b4aacc9c) [cherry-pick] Update dependencies (#478) (#482)
- [d1bcfc4b](https://github.com/stashed/mysql/commit/d1bcfc4b) [cherry-pick] Use user nobody (#477)
- [79ede44f](https://github.com/stashed/mysql/commit/79ede44f) [cherry-pick] Update repository config (#469) (#473)
- [7fad3905](https://github.com/stashed/mysql/commit/7fad3905) [cherry-pick] Update repository config (#464) (#468)
- [06308914](https://github.com/stashed/mysql/commit/06308914) [cherry-pick] Update README.md (#463)


### [8.0.14-v13](https://github.com/stashed/mysql/releases/tag/8.0.14-v13)

- [04d8f7d9](https://github.com/stashed/mysql/commit/04d8f7d9) Prepare for release 8.0.14-v13 (#529)
- [235005d4](https://github.com/stashed/mysql/commit/235005d4) [cherry-pick] Fix jwt-go security vulnerability (#523) (#525)
- [ff9e8eb9](https://github.com/stashed/mysql/commit/ff9e8eb9) [cherry-pick] Fix jwt-go security vulnerability (#518) (#520)
- [b0ad3407](https://github.com/stashed/mysql/commit/b0ad3407) [cherry-pick] Update dependencies to publish SiteInfo (#513) (#515)
- [e5ce9e5a](https://github.com/stashed/mysql/commit/e5ce9e5a) [cherry-pick] Update dependencies to publish SiteInfo (#508) (#510)
- [6879bc7f](https://github.com/stashed/mysql/commit/6879bc7f) [cherry-pick] Log warning if Community License is used with non-demo namespace (#503) (#505)
- [9d021bf3](https://github.com/stashed/mysql/commit/9d021bf3) [cherry-pick] Use official restic v0.12.1 (#498) (#500)
- [23a8405c](https://github.com/stashed/mysql/commit/23a8405c) [cherry-pick] Update repository config (#493) (#495)
- [ef9bd7a3](https://github.com/stashed/mysql/commit/ef9bd7a3) [cherry-pick] Update dependencies (#488) (#490)
- [753cf713](https://github.com/stashed/mysql/commit/753cf713) [cherry-pick] Update dependencies (#483) (#485)
- [fe5dbd4e](https://github.com/stashed/mysql/commit/fe5dbd4e) [cherry-pick] Update dependencies (#478) (#480)
- [c8090a9b](https://github.com/stashed/mysql/commit/c8090a9b) [cherry-pick] Use user nobody (#475)
- [36c9a151](https://github.com/stashed/mysql/commit/36c9a151) [cherry-pick] Update repository config (#469) (#471)
- [9bdfdeeb](https://github.com/stashed/mysql/commit/9bdfdeeb) [cherry-pick] Update repository config (#464) (#466)
- [168469fe](https://github.com/stashed/mysql/commit/168469fe) [cherry-pick] Update README.md (#461)


### [8.0.21-v7](https://github.com/stashed/mysql/releases/tag/8.0.21-v7)

- [e5ea20ba](https://github.com/stashed/mysql/commit/e5ea20ba) Prepare for release 8.0.21-v7 (#530)
- [5db2b531](https://github.com/stashed/mysql/commit/5db2b531) [cherry-pick] Fix jwt-go security vulnerability (#523) (#526)
- [52cd9513](https://github.com/stashed/mysql/commit/52cd9513) [cherry-pick] Fix jwt-go security vulnerability (#518) (#521)
- [7d004545](https://github.com/stashed/mysql/commit/7d004545) [cherry-pick] Update dependencies to publish SiteInfo (#513) (#516)
- [6abc719c](https://github.com/stashed/mysql/commit/6abc719c) [cherry-pick] Update dependencies to publish SiteInfo (#508) (#511)
- [cd79c2de](https://github.com/stashed/mysql/commit/cd79c2de) [cherry-pick] Log warning if Community License is used with non-demo namespace (#503) (#506)
- [d974fafb](https://github.com/stashed/mysql/commit/d974fafb) [cherry-pick] Use official restic v0.12.1 (#498) (#501)
- [3828e7d3](https://github.com/stashed/mysql/commit/3828e7d3) [cherry-pick] Update repository config (#493) (#496)
- [2fd7d488](https://github.com/stashed/mysql/commit/2fd7d488) [cherry-pick] Update dependencies (#488) (#491)
- [2c28a324](https://github.com/stashed/mysql/commit/2c28a324) [cherry-pick] Update dependencies (#483) (#486)
- [b54ef541](https://github.com/stashed/mysql/commit/b54ef541) [cherry-pick] Update dependencies (#478) (#481)
- [249af93e](https://github.com/stashed/mysql/commit/249af93e) [cherry-pick] Use user nobody (#476)
- [b3d0bf71](https://github.com/stashed/mysql/commit/b3d0bf71) [cherry-pick] Update repository config (#469) (#472)
- [1f7e61fd](https://github.com/stashed/mysql/commit/1f7e61fd) [cherry-pick] Update repository config (#464) (#467)
- [436e0ab6](https://github.com/stashed/mysql/commit/436e0ab6) [cherry-pick] Update README.md (#462)



## [stashed/nats](https://github.com/stashed/nats)

### [2.6.1](https://github.com/stashed/nats/releases/tag/2.6.1)

- [898dc86](https://github.com/stashed/nats/commit/898dc86) Prepare for release 2.6.1 (#9)
- [b54316d](https://github.com/stashed/nats/commit/b54316d) Fix jwt-go security vulnerability (#7) (#8)
- [dba8685](https://github.com/stashed/nats/commit/dba8685) Use nats.go v1.13.0 (#6)
- [324241b](https://github.com/stashed/nats/commit/324241b) Update TLS constants #5
- [2077d6f](https://github.com/stashed/nats/commit/2077d6f) Append args  (#4)
- [e7dc55d](https://github.com/stashed/nats/commit/e7dc55d) Update repository config (#3)
- [4e97bdc](https://github.com/stashed/nats/commit/4e97bdc) Add nats Stash addon (#2)
- [48941bd](https://github.com/stashed/nats/commit/48941bd) Merge pull request #1 from stashed/global-replace
- [ff46ec0](https://github.com/stashed/nats/commit/ff46ec0) Global replace Redis with NATS
- [6d880e0](https://github.com/stashed/nats/commit/6d880e0) Use user nobody
- [5252385](https://github.com/stashed/nats/commit/5252385) Update repository config (#11)
- [f6dc027](https://github.com/stashed/nats/commit/f6dc027) Update repository config (#8)
- [d68710e](https://github.com/stashed/nats/commit/d68710e) Prepare for release v2021.08.02 (#7)
- [1772bc7](https://github.com/stashed/nats/commit/1772bc7) Update repository config (#2)
- [7d4bfee](https://github.com/stashed/nats/commit/7d4bfee) Make Redis addon ready to use (#1)



## [stashed/percona-xtradb](https://github.com/stashed/percona-xtradb)

### [5.7-v8](https://github.com/stashed/percona-xtradb/releases/tag/5.7-v8)

- [42564482](https://github.com/stashed/percona-xtradb/commit/42564482) Prepare for release 5.7-v8 (#221)
- [22f43849](https://github.com/stashed/percona-xtradb/commit/22f43849) [cherry-pick] Fix jwt-go security vulnerability (#219) (#220)
- [26dbbe0d](https://github.com/stashed/percona-xtradb/commit/26dbbe0d) [cherry-pick] Fix jwt-go security vulnerability (#217) (#218)
- [cfa5e989](https://github.com/stashed/percona-xtradb/commit/cfa5e989) [cherry-pick] Update dependencies to publish SiteInfo (#215) (#216)
- [645df7c7](https://github.com/stashed/percona-xtradb/commit/645df7c7) Update dependencies to publish SiteInfo (#213) (#214)
- [327484ce](https://github.com/stashed/percona-xtradb/commit/327484ce) [cherry-pick] Log warning if Community License is used with non-demo namespace (#211) (#212)
- [70e361f6](https://github.com/stashed/percona-xtradb/commit/70e361f6) [cherry-pick] Use official restic v0.12.1 (#209) (#210)
- [0568a599](https://github.com/stashed/percona-xtradb/commit/0568a599) [cherry-pick] Update repository config (#207) (#208)
- [541cafce](https://github.com/stashed/percona-xtradb/commit/541cafce) [cherry-pick] Update dependencies (#205) (#206)
- [42914499](https://github.com/stashed/percona-xtradb/commit/42914499) [cherry-pick] Update dependencies (#203) (#204)
- [f5d39489](https://github.com/stashed/percona-xtradb/commit/f5d39489) [cherry-pick] Update dependencies (#201) (#202)
- [3b0f52a5](https://github.com/stashed/percona-xtradb/commit/3b0f52a5) [cherry-pick] Use user nobody (#200)
- [91a6d048](https://github.com/stashed/percona-xtradb/commit/91a6d048) [cherry-pick] Update repository config (#198) (#199)
- [27ea0703](https://github.com/stashed/percona-xtradb/commit/27ea0703) [cherry-pick] Update repository config (#196) (#197)
- [3a9ec481](https://github.com/stashed/percona-xtradb/commit/3a9ec481) [cherry-pick] Update README.md (#195)



## [stashed/postgres](https://github.com/stashed/postgres)

### [9.6.19-v11](https://github.com/stashed/postgres/releases/tag/9.6.19-v11)

- [8f1256ef](https://github.com/stashed/postgres/commit/8f1256ef) Prepare for release 9.6.19-v11 (#935)
- [851efd30](https://github.com/stashed/postgres/commit/851efd30) [cherry-pick] Fix jwt-go security vulnerability (#923) (#929)
- [132fad16](https://github.com/stashed/postgres/commit/132fad16) [cherry-pick] Fix jwt-go security vulnerability (#916) (#922)
- [de2c0cd2](https://github.com/stashed/postgres/commit/de2c0cd2) [cherry-pick] Update dependencies to publish SiteInfo (#910) (#915)
- [8529a78e](https://github.com/stashed/postgres/commit/8529a78e) [cherry-pick] Update dependencies to publish SiteInfo (#904) (#909)
- [1ec16ab0](https://github.com/stashed/postgres/commit/1ec16ab0) [cherry-pick] Log warning if Community License is used with non-demo namespace (#897) (#902)
- [a5c59aa8](https://github.com/stashed/postgres/commit/a5c59aa8) [cherry-pick] Use official restic v0.12.1 (#891) (#896)
- [6042476d](https://github.com/stashed/postgres/commit/6042476d) [cherry-pick] Update repository config (#885) (#890)
- [062cf691](https://github.com/stashed/postgres/commit/062cf691) [cherry-pick] Update dependencies (#879) (#884)
- [74f16152](https://github.com/stashed/postgres/commit/74f16152) [cherry-pick] Update dependencies (#873) (#878)
- [665ff907](https://github.com/stashed/postgres/commit/665ff907) [cherry-pick] Update dependencies (#867) (#872)
- [51c55130](https://github.com/stashed/postgres/commit/51c55130) [cherry-pick] Use user nobody (#866)
- [9fd4d8c3](https://github.com/stashed/postgres/commit/9fd4d8c3) [cherry-pick] Update repository config (#856) (#861)


### [10.14-v11](https://github.com/stashed/postgres/releases/tag/10.14-v11)

- [022910d2](https://github.com/stashed/postgres/commit/022910d2) Prepare for release 10.14-v11 (#930)
- [df4ea88e](https://github.com/stashed/postgres/commit/df4ea88e) [cherry-pick] Fix jwt-go security vulnerability (#923) (#924)
- [37492e25](https://github.com/stashed/postgres/commit/37492e25) [cherry-pick] Fix jwt-go security vulnerability (#916) (#917)
- [c36fd40d](https://github.com/stashed/postgres/commit/c36fd40d) [cherry-pick] Update dependencies to publish SiteInfo (#910) (#911)
- [40364b31](https://github.com/stashed/postgres/commit/40364b31) [cherry-pick] Update dependencies to publish SiteInfo (#904) (#905)
- [c7e6e42d](https://github.com/stashed/postgres/commit/c7e6e42d) [cherry-pick] Log warning if Community License is used with non-demo namespace (#897) (#898)
- [a7a14804](https://github.com/stashed/postgres/commit/a7a14804) [cherry-pick] Use official restic v0.12.1 (#891) (#892)
- [62f39a5a](https://github.com/stashed/postgres/commit/62f39a5a) [cherry-pick] Update repository config (#885) (#886)
- [c8c626dd](https://github.com/stashed/postgres/commit/c8c626dd) [cherry-pick] Update dependencies (#879) (#880)
- [8cca6897](https://github.com/stashed/postgres/commit/8cca6897) [cherry-pick] Update dependencies (#873) (#874)
- [2b8823ac](https://github.com/stashed/postgres/commit/2b8823ac) [cherry-pick] Update dependencies (#867) (#868)
- [de1290b4](https://github.com/stashed/postgres/commit/de1290b4) [cherry-pick] Use user nobody (#862)
- [9d8e96ed](https://github.com/stashed/postgres/commit/9d8e96ed) [cherry-pick] Update repository config (#856) (#857)
- [49ab6354](https://github.com/stashed/postgres/commit/49ab6354) [cherry-pick] Update repository config (#850) (#851)
- [ac88e3c1](https://github.com/stashed/postgres/commit/ac88e3c1) [cherry-pick] Update README.md (#845)


### [11.9-v11](https://github.com/stashed/postgres/releases/tag/11.9-v11)

- [809eb881](https://github.com/stashed/postgres/commit/809eb881) Prepare for release 11.9-v11 (#931)
- [f8df2138](https://github.com/stashed/postgres/commit/f8df2138) [cherry-pick] Fix jwt-go security vulnerability (#923) (#925)
- [a5fcf2d7](https://github.com/stashed/postgres/commit/a5fcf2d7) [cherry-pick] Fix jwt-go security vulnerability (#916) (#918)
- [bc9b05a7](https://github.com/stashed/postgres/commit/bc9b05a7) [cherry-pick] Update dependencies to publish SiteInfo (#910) (#912)
- [eba577ca](https://github.com/stashed/postgres/commit/eba577ca) [cherry-pick] Update dependencies to publish SiteInfo (#904) (#906)
- [dd7e5b3c](https://github.com/stashed/postgres/commit/dd7e5b3c) [cherry-pick] Log warning if Community License is used with non-demo namespace (#897) (#899)
- [dbc851fe](https://github.com/stashed/postgres/commit/dbc851fe) [cherry-pick] Use official restic v0.12.1 (#891) (#893)
- [96c10215](https://github.com/stashed/postgres/commit/96c10215) [cherry-pick] Update repository config (#885) (#887)
- [89f96f4f](https://github.com/stashed/postgres/commit/89f96f4f) [cherry-pick] Update dependencies (#879) (#881)
- [540665e0](https://github.com/stashed/postgres/commit/540665e0) [cherry-pick] Update dependencies (#873) (#875)
- [89108c2e](https://github.com/stashed/postgres/commit/89108c2e) [cherry-pick] Update dependencies (#867) (#869)
- [6fb330f8](https://github.com/stashed/postgres/commit/6fb330f8) [cherry-pick] Use user nobody (#863)
- [057473bc](https://github.com/stashed/postgres/commit/057473bc) [cherry-pick] Update repository config (#856) (#858)
- [9387dbe7](https://github.com/stashed/postgres/commit/9387dbe7) [cherry-pick] Update repository config (#850) (#852)
- [0dda31f0](https://github.com/stashed/postgres/commit/0dda31f0) [cherry-pick] Update README.md (#846)


### [12.4-v11](https://github.com/stashed/postgres/releases/tag/12.4-v11)

- [b6980ee7](https://github.com/stashed/postgres/commit/b6980ee7) Prepare for release 12.4-v11 (#932)
- [409eb663](https://github.com/stashed/postgres/commit/409eb663) [cherry-pick] Fix jwt-go security vulnerability (#923) (#926)
- [1f593322](https://github.com/stashed/postgres/commit/1f593322) [cherry-pick] Fix jwt-go security vulnerability (#916) (#919)
- [b19d3f18](https://github.com/stashed/postgres/commit/b19d3f18) [cherry-pick] Update dependencies to publish SiteInfo (#910) (#913)
- [8135d138](https://github.com/stashed/postgres/commit/8135d138) [cherry-pick] Update dependencies to publish SiteInfo (#904) (#907)
- [105bcabe](https://github.com/stashed/postgres/commit/105bcabe) [cherry-pick] Log warning if Community License is used with non-demo namespace (#897) (#900)
- [51a7933d](https://github.com/stashed/postgres/commit/51a7933d) [cherry-pick] Use official restic v0.12.1 (#891) (#894)
- [b49bb69f](https://github.com/stashed/postgres/commit/b49bb69f) [cherry-pick] Update repository config (#885) (#888)
- [bc4490f3](https://github.com/stashed/postgres/commit/bc4490f3) [cherry-pick] Update dependencies (#879) (#882)
- [7ec191fd](https://github.com/stashed/postgres/commit/7ec191fd) [cherry-pick] Update dependencies (#873) (#876)
- [16a2ee1d](https://github.com/stashed/postgres/commit/16a2ee1d) [cherry-pick] Update dependencies (#867) (#870)
- [9d392dfa](https://github.com/stashed/postgres/commit/9d392dfa) [cherry-pick] Use user nobody (#864)
- [c142e39c](https://github.com/stashed/postgres/commit/c142e39c) [cherry-pick] Update repository config (#856) (#859)
- [d8e8c2d2](https://github.com/stashed/postgres/commit/d8e8c2d2) [cherry-pick] Update repository config (#850) (#853)
- [3221e02a](https://github.com/stashed/postgres/commit/3221e02a) [cherry-pick] Update README.md (#847)


### [13.1-v8](https://github.com/stashed/postgres/releases/tag/13.1-v8)

- [9211494b](https://github.com/stashed/postgres/commit/9211494b) Prepare for release 13.1-v8 (#933)
- [199edbf1](https://github.com/stashed/postgres/commit/199edbf1) [cherry-pick] Fix jwt-go security vulnerability (#923) (#927)
- [3efe6ad3](https://github.com/stashed/postgres/commit/3efe6ad3) [cherry-pick] Fix jwt-go security vulnerability (#916) (#920)
- [f6d36c5f](https://github.com/stashed/postgres/commit/f6d36c5f) [cherry-pick] Update dependencies to publish SiteInfo (#910) (#914)
- [9721595a](https://github.com/stashed/postgres/commit/9721595a) [cherry-pick] Update dependencies to publish SiteInfo (#904) (#908)
- [7b9c8fa6](https://github.com/stashed/postgres/commit/7b9c8fa6) [cherry-pick] Log warning if Community License is used with non-demo namespace (#897) (#901)
- [a71c6929](https://github.com/stashed/postgres/commit/a71c6929) [cherry-pick] Use official restic v0.12.1 (#891) (#895)
- [f795f28e](https://github.com/stashed/postgres/commit/f795f28e) [cherry-pick] Update repository config (#885) (#889)
- [927c795c](https://github.com/stashed/postgres/commit/927c795c) [cherry-pick] Update dependencies (#879) (#883)
- [72f86c43](https://github.com/stashed/postgres/commit/72f86c43) [cherry-pick] Update dependencies (#873) (#877)
- [3bc7ed49](https://github.com/stashed/postgres/commit/3bc7ed49) [cherry-pick] Update dependencies (#867) (#871)
- [6cf5b484](https://github.com/stashed/postgres/commit/6cf5b484) [cherry-pick] Use user nobody (#865)
- [f0c36245](https://github.com/stashed/postgres/commit/f0c36245) [cherry-pick] Update repository config (#856) (#860)
- [f73a8fdc](https://github.com/stashed/postgres/commit/f73a8fdc) [cherry-pick] Update repository config (#850) (#854)
- [08392aad](https://github.com/stashed/postgres/commit/08392aad) [cherry-pick] Update README.md (#848)


### [14.0](https://github.com/stashed/postgres/releases/tag/14.0)




## [stashed/redis](https://github.com/stashed/redis)

### [5.0.13-v1](https://github.com/stashed/redis/releases/tag/5.0.13-v1)

- [2cf8e50](https://github.com/stashed/redis/commit/2cf8e50) Prepare for release 5.0.13-v1 (#52)
- [19a146f](https://github.com/stashed/redis/commit/19a146f) [cherry-pick] Fix jwt-go security vulnerability (#49) (#50)
- [89a01a1](https://github.com/stashed/redis/commit/89a01a1) [cherry-pick] Fix jwt-go security vulnerability (#46) (#47)
- [71ef640](https://github.com/stashed/redis/commit/71ef640) Update dependencies to publish SiteInfo (#43) (#44)
- [07f2b0a](https://github.com/stashed/redis/commit/07f2b0a) [cherry-pick] Update dependencies to publish SiteInfo (#40) (#41)
- [c4fe884](https://github.com/stashed/redis/commit/c4fe884) [cherry-pick] Use forked redis-dump-go (#37) (#38)
- [c857baf](https://github.com/stashed/redis/commit/c857baf) [cherry-pick] Add Support for TLS enabled Redis client (#31) (#35)
- [fa42bbe](https://github.com/stashed/redis/commit/fa42bbe) [cherry-pick] Log warning if Community License is used with non-demo namespace (#32) (#33)
- [a14feb6](https://github.com/stashed/redis/commit/a14feb6) [cherry-pick] Use official restic v0.12.1 (#28) (#29)
- [800ad77](https://github.com/stashed/redis/commit/800ad77) [cherry-pick] Update repository config (#25) (#26)
- [8472251](https://github.com/stashed/redis/commit/8472251) [cherry-pick] Update dependencies (#22) (#23)
- [e6bbc7e](https://github.com/stashed/redis/commit/e6bbc7e) [cherry-pick] Update dependencies (#19) (#20)
- [d81c5d4](https://github.com/stashed/redis/commit/d81c5d4) [cherry-pick] Update dependencies (#16) (#17)
- [ac57ea0](https://github.com/stashed/redis/commit/ac57ea0) [cherry-pick] Use user nobody (#14)
- [c65e4af](https://github.com/stashed/redis/commit/c65e4af) [cherry-pick] Update repository config (#11) (#12)
- [62782d5](https://github.com/stashed/redis/commit/62782d5) [cherry-pick] Update repository config (#8) (#9)


### [6.2.5-v1](https://github.com/stashed/redis/releases/tag/6.2.5-v1)

- [5882057](https://github.com/stashed/redis/commit/5882057) Prepare for release 6.2.5-v1 (#53)
- [59eeffa](https://github.com/stashed/redis/commit/59eeffa) [cherry-pick] Fix jwt-go security vulnerability (#49) (#51)
- [55237a0](https://github.com/stashed/redis/commit/55237a0) [cherry-pick] Fix jwt-go security vulnerability (#46) (#48)
- [149186e](https://github.com/stashed/redis/commit/149186e) Update dependencies to publish SiteInfo (#43) (#45)
- [1935030](https://github.com/stashed/redis/commit/1935030) [cherry-pick] Update dependencies to publish SiteInfo (#40) (#42)
- [07a1317](https://github.com/stashed/redis/commit/07a1317) [cherry-pick] Use forked redis-dump-go (#37) (#39)
- [ed58c90](https://github.com/stashed/redis/commit/ed58c90) [cherry-pick] Add Support for TLS enabled Redis client (#31) (#36)
- [11a3fb3](https://github.com/stashed/redis/commit/11a3fb3) [cherry-pick] Log warning if Community License is used with non-demo namespace (#32) (#34)
- [71bd761](https://github.com/stashed/redis/commit/71bd761) [cherry-pick] Use official restic v0.12.1 (#28) (#30)
- [281c595](https://github.com/stashed/redis/commit/281c595) [cherry-pick] Update repository config (#25) (#27)
- [7beb611](https://github.com/stashed/redis/commit/7beb611) [cherry-pick] Update dependencies (#22) (#24)
- [39852fa](https://github.com/stashed/redis/commit/39852fa) [cherry-pick] Update dependencies (#19) (#21)
- [ee90e7c](https://github.com/stashed/redis/commit/ee90e7c) [cherry-pick] Update dependencies (#16) (#18)
- [085749d](https://github.com/stashed/redis/commit/085749d) [cherry-pick] Use user nobody (#15)
- [6bd0302](https://github.com/stashed/redis/commit/6bd0302) [cherry-pick] Update repository config (#11) (#13)
- [81550b0](https://github.com/stashed/redis/commit/81550b0) [cherry-pick] Update repository config (#8) (#10)



## [stashed/stash](https://github.com/stashed/stash)

### [v0.16.0](https://github.com/stashed/stash/releases/tag/v0.16.0)

- [535efca8](https://github.com/stashed/stash/commit/535efca8) Prepare for release v0.16.0 (#1394)
- [7c26c7dd](https://github.com/stashed/stash/commit/7c26c7dd) Fix jwt-go security vulnerability (#1393)
- [8f63f724](https://github.com/stashed/stash/commit/8f63f724) Add support for ETCD restore flow (#1392)
- [5665ea7e](https://github.com/stashed/stash/commit/5665ea7e) Use nats.go v1.13.0 (#1391)
- [cc2da916](https://github.com/stashed/stash/commit/cc2da916) Setup SiteInfo publisher (#1390)
- [8daab52d](https://github.com/stashed/stash/commit/8daab52d) Update dependencies to publish SiteInfo (#1389)
- [50b58218](https://github.com/stashed/stash/commit/50b58218) Support passing args to restic backup/restore command (#1385)
- [28d878ed](https://github.com/stashed/stash/commit/28d878ed) Update dependencies to publish SiteInfo (#1387)
- [c063a536](https://github.com/stashed/stash/commit/c063a536) Fix license-reader ClusterRoleBinding not cleaning up properly  (#1386)
- [e09bd7b0](https://github.com/stashed/stash/commit/e09bd7b0) Log warning if Community License is used with non-demo namespace (#1383)
- [3da6094c](https://github.com/stashed/stash/commit/3da6094c) Use stash-trigger as backup triggering CronJob name prefix #1382
- [e08ce8f6](https://github.com/stashed/stash/commit/e08ce8f6) Use `stash-trigger` as backup triggering CronJob name prefix
- [5027ae41](https://github.com/stashed/stash/commit/5027ae41) Use official restic v0.12.1 (#1381)
- [f23d8f42](https://github.com/stashed/stash/commit/f23d8f42) Update repository config (#1379)
- [0348419d](https://github.com/stashed/stash/commit/0348419d) Update dependencies (#1378)
- [0f6400d5](https://github.com/stashed/stash/commit/0f6400d5) Update repository config (#1375)




