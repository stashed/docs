---
title: Changelog | Stash
description: Changelog
menu:
  docs_{{.version}}:
    identifier: changelog-stash-v2021.6.18
    name: Changelog-v2021.6.18
    parent: welcome
    weight: 20210618
product_name: stash
menu_name: docs_{{.version}}
section_menu_id: welcome
url: /docs/{{.version}}/welcome/changelog-v2021.6.18/
aliases:
  - /docs/{{.version}}/CHANGELOG-v2021.6.18/
---

# Stash v2021.6.18 (2021-06-16)


## [appscode/stash-enterprise](https://github.com/appscode/stash-enterprise)

### [v0.14.0](https://github.com/appscode/stash-enterprise/releases/tag/v0.14.0)

- [6d2613e7](https://github.com/appscode/stash-enterprise/commit/6d2613e7) Prepare for release v0.14.0 (#100)
- [f8afa76d](https://github.com/appscode/stash-enterprise/commit/f8afa76d) Send audit events when analytics are enabled (#99)
- [cdb3b3fe](https://github.com/appscode/stash-enterprise/commit/cdb3b3fe) Fix operator crash + E2E tests (#98)
- [4288f0c2](https://github.com/appscode/stash-enterprise/commit/4288f0c2) Use namespace when looking up running BackupSessions for Invoker (#97)
- [64c7d7c9](https://github.com/appscode/stash-enterprise/commit/64c7d7c9) Create auditor if license file is provided (#96)
- [5e193ea9](https://github.com/appscode/stash-enterprise/commit/5e193ea9) Disable api priortiy and fairness feature for webhook server (#95)
- [b4fcc0b6](https://github.com/appscode/stash-enterprise/commit/b4fcc0b6) Publish audit events (#94)
- [d785fec3](https://github.com/appscode/stash-enterprise/commit/d785fec3) Use klog/v2 (#93)
- [324e9aa4](https://github.com/appscode/stash-enterprise/commit/324e9aa4) Use kglog helper
- [8b4c6fd6](https://github.com/appscode/stash-enterprise/commit/8b4c6fd6) Update Kubernetes toolchain to v1.21.0 (#92)



## [stashed/apimachinery](https://github.com/stashed/apimachinery)

### [v0.14.0](https://github.com/stashed/apimachinery/releases/tag/v0.14.0)

- [a382bbe2](https://github.com/stashed/apimachinery/commit/a382bbe2) Fix build
- [0d9f6eec](https://github.com/stashed/apimachinery/commit/0d9f6eec) Update dependencies (#101)
- [6abdb1e2](https://github.com/stashed/apimachinery/commit/6abdb1e2) Replace go-bindata with //go:embed (#100)
- [974fc12c](https://github.com/stashed/apimachinery/commit/974fc12c) Update dependencies
- [ae165464](https://github.com/stashed/apimachinery/commit/ae165464) Fix build
- [15d8c532](https://github.com/stashed/apimachinery/commit/15d8c532) Update dependencies
- [06ac37bd](https://github.com/stashed/apimachinery/commit/06ac37bd) Only depend on klog/v2
- [79f9a3fe](https://github.com/stashed/apimachinery/commit/79f9a3fe) Use Kubernetes v1.21.0 toolchain (#98)
- [1157edb1](https://github.com/stashed/apimachinery/commit/1157edb1) Cleanup dependencies



## [stashed/cli](https://github.com/stashed/cli)

### [v0.14.0](https://github.com/stashed/cli/releases/tag/v0.14.0)

- [ceef781](https://github.com/stashed/cli/commit/ceef781) Prepare for release v0.14.0 (#124)
- [7e1fb45](https://github.com/stashed/cli/commit/7e1fb45) Use klog/v2 (#123)
- [4eee2a0](https://github.com/stashed/cli/commit/4eee2a0) Use kglog helper
- [5de2bb5](https://github.com/stashed/cli/commit/5de2bb5) Use klog/v2 (#122)
- [6a110fc](https://github.com/stashed/cli/commit/6a110fc) Use Kubernetes v1.21.0 toolchain (#121)
- [bea398c](https://github.com/stashed/cli/commit/bea398c) Use Kubernetes v1.21.0 toolchain (#120)
- [8f1a18f](https://github.com/stashed/cli/commit/8f1a18f) Use Kubernetes v1.21.0 toolchain (#119)
- [45b1b13](https://github.com/stashed/cli/commit/45b1b13) Update Kubernetes toolchain to v1.21.0 (#118)



## [stashed/elasticsearch](https://github.com/stashed/elasticsearch)

### [5.6.4-v10](https://github.com/stashed/elasticsearch/releases/tag/5.6.4-v10)

- [00e2fb3b](https://github.com/stashed/elasticsearch/commit/00e2fb3b) Prepare for release 5.6.4-v10 (#819)
- [d3954cad](https://github.com/stashed/elasticsearch/commit/d3954cad) [cherry-pick] Use klog/v2 (#810) (#811)
- [51f5e943](https://github.com/stashed/elasticsearch/commit/51f5e943) [cherry-pick] Use kglog helper (#802)
- [bb125283](https://github.com/stashed/elasticsearch/commit/bb125283) Use k8s 1.21.0 toolchain


### [6.2.4-v10](https://github.com/stashed/elasticsearch/releases/tag/6.2.4-v10)

- [6cd15ccb](https://github.com/stashed/elasticsearch/commit/6cd15ccb) Prepare for release 6.2.4-v10 (#820)
- [e6a3e186](https://github.com/stashed/elasticsearch/commit/e6a3e186) [cherry-pick] Use klog/v2 (#810) (#812)
- [d969c43d](https://github.com/stashed/elasticsearch/commit/d969c43d) [cherry-pick] Use kglog helper (#803)
- [b7cc8f39](https://github.com/stashed/elasticsearch/commit/b7cc8f39) Use k8s 1.21.0 toolchain


### [6.3.0-v10](https://github.com/stashed/elasticsearch/releases/tag/6.3.0-v10)

- [3368493b](https://github.com/stashed/elasticsearch/commit/3368493b) Prepare for release 6.3.0-v10 (#821)
- [134b13c9](https://github.com/stashed/elasticsearch/commit/134b13c9) [cherry-pick] Use klog/v2 (#810) (#813)
- [0b48f103](https://github.com/stashed/elasticsearch/commit/0b48f103) [cherry-pick] Use kglog helper (#804)
- [ac207bc9](https://github.com/stashed/elasticsearch/commit/ac207bc9) Use k8s 1.21.0 toolchain


### [6.4.0-v10](https://github.com/stashed/elasticsearch/releases/tag/6.4.0-v10)

- [fe813d50](https://github.com/stashed/elasticsearch/commit/fe813d50) Prepare for release 6.4.0-v10 (#822)
- [d69452ab](https://github.com/stashed/elasticsearch/commit/d69452ab) [cherry-pick] Use klog/v2 (#810) (#814)
- [74afb891](https://github.com/stashed/elasticsearch/commit/74afb891) [cherry-pick] Use kglog helper (#805)
- [9b94b942](https://github.com/stashed/elasticsearch/commit/9b94b942) Use k8s 1.21.0 toolchain


### [6.5.3-v10](https://github.com/stashed/elasticsearch/releases/tag/6.5.3-v10)

- [a101ccaa](https://github.com/stashed/elasticsearch/commit/a101ccaa) Prepare for release 6.5.3-v10 (#823)
- [73230f53](https://github.com/stashed/elasticsearch/commit/73230f53) [cherry-pick] Use klog/v2 (#810) (#815)
- [023a0131](https://github.com/stashed/elasticsearch/commit/023a0131) [cherry-pick] Use kglog helper (#806)
- [a4d28503](https://github.com/stashed/elasticsearch/commit/a4d28503) Use k8s 1.21.0 toolchain


### [6.8.0-v10](https://github.com/stashed/elasticsearch/releases/tag/6.8.0-v10)

- [8a683935](https://github.com/stashed/elasticsearch/commit/8a683935) Prepare for release 6.8.0-v10 (#824)
- [db27f0d1](https://github.com/stashed/elasticsearch/commit/db27f0d1) [cherry-pick] Use klog/v2 (#810) (#816)
- [970ef1a1](https://github.com/stashed/elasticsearch/commit/970ef1a1) [cherry-pick] Use kglog helper (#807)
- [e1e24ec7](https://github.com/stashed/elasticsearch/commit/e1e24ec7) Use k8s 1.21.0 toolchain


### [7.2.0-v10](https://github.com/stashed/elasticsearch/releases/tag/7.2.0-v10)

- [4eac1ea9](https://github.com/stashed/elasticsearch/commit/4eac1ea9) Prepare for release 7.2.0-v10 (#825)
- [2aa02a10](https://github.com/stashed/elasticsearch/commit/2aa02a10) [cherry-pick] Use klog/v2 (#810) (#817)
- [9dd713c7](https://github.com/stashed/elasticsearch/commit/9dd713c7) [cherry-pick] Use kglog helper (#808)
- [1495f883](https://github.com/stashed/elasticsearch/commit/1495f883) Use k8s 1.21.0 toolchain


### [7.3.2-v10](https://github.com/stashed/elasticsearch/releases/tag/7.3.2-v10)

- [2808b0ba](https://github.com/stashed/elasticsearch/commit/2808b0ba) Prepare for release 7.3.2-v10 (#826)
- [478a1f1d](https://github.com/stashed/elasticsearch/commit/478a1f1d) [cherry-pick] Use klog/v2 (#810) (#818)
- [68236dc9](https://github.com/stashed/elasticsearch/commit/68236dc9) [cherry-pick] Use kglog helper (#809)
- [bfab7862](https://github.com/stashed/elasticsearch/commit/bfab7862) Use k8s 1.21.0 toolchain



## [stashed/installer](https://github.com/stashed/installer)

### [v2021.6.18](https://github.com/stashed/installer/releases/tag/v2021.6.18)

- [2082d2d](https://github.com/stashed/installer/commit/2082d2d) Prepare for release v2021.6.18 (#177)
- [1a0fccb](https://github.com/stashed/installer/commit/1a0fccb) Update user-roles for v1beta1 APIs (#176)
- [3334ea0](https://github.com/stashed/installer/commit/3334ea0) Use k8s 1.21.0 toolchain (#175)



## [stashed/mongodb](https://github.com/stashed/mongodb)

### [3.4.17-v9](https://github.com/stashed/mongodb/releases/tag/3.4.17-v9)

- [3c439686](https://github.com/stashed/mongodb/commit/3c439686) Prepare for release 3.4.17-v9 (#981)
- [767e3e91](https://github.com/stashed/mongodb/commit/767e3e91) [cherry-pick] Use klog/v2 (#969) (#970)
- [18054cc7](https://github.com/stashed/mongodb/commit/18054cc7) [cherry-pick] Use kglog helper (#958)
- [27c8e858](https://github.com/stashed/mongodb/commit/27c8e858) Use k8s 1.21.0 toolchain


### [3.4.22-v9](https://github.com/stashed/mongodb/releases/tag/3.4.22-v9)

- [d79dd9fa](https://github.com/stashed/mongodb/commit/d79dd9fa) Prepare for release 3.4.22-v9 (#982)
- [5af76acd](https://github.com/stashed/mongodb/commit/5af76acd) [cherry-pick] Use klog/v2 (#969) (#971)
- [07acc184](https://github.com/stashed/mongodb/commit/07acc184) [cherry-pick] Use kglog helper (#959)
- [b1607397](https://github.com/stashed/mongodb/commit/b1607397) Use k8s 1.21.0 toolchain


### [3.6.8-v9](https://github.com/stashed/mongodb/releases/tag/3.6.8-v9)

- [5b8ba177](https://github.com/stashed/mongodb/commit/5b8ba177) Prepare for release 3.6.8-v9 (#984)
- [1c02bbf8](https://github.com/stashed/mongodb/commit/1c02bbf8) [cherry-pick] Use klog/v2 (#969) (#973)
- [c580d161](https://github.com/stashed/mongodb/commit/c580d161) [cherry-pick] Use kglog helper (#961)
- [b13bf55e](https://github.com/stashed/mongodb/commit/b13bf55e) Use k8s 1.21.0 toolchain


### [3.6.13-v9](https://github.com/stashed/mongodb/releases/tag/3.6.13-v9)

- [d947b446](https://github.com/stashed/mongodb/commit/d947b446) Prepare for release 3.6.13-v9 (#983)
- [b2f4a881](https://github.com/stashed/mongodb/commit/b2f4a881) [cherry-pick] Use klog/v2 (#969) (#972)
- [0551d36c](https://github.com/stashed/mongodb/commit/0551d36c) [cherry-pick] Use kglog helper (#960)
- [07d09f0d](https://github.com/stashed/mongodb/commit/07d09f0d) Use k8s 1.21.0 toolchain


### [4.0.3-v9](https://github.com/stashed/mongodb/releases/tag/4.0.3-v9)

- [401bc9aa](https://github.com/stashed/mongodb/commit/401bc9aa) Prepare for release 4.0.3-v9 (#986)
- [fcdd878e](https://github.com/stashed/mongodb/commit/fcdd878e) [cherry-pick] Use klog/v2 (#969) (#975)
- [ab65b810](https://github.com/stashed/mongodb/commit/ab65b810) [cherry-pick] Use kglog helper (#963)
- [184f230e](https://github.com/stashed/mongodb/commit/184f230e) Use k8s 1.21.0 toolchain


### [4.0.5-v9](https://github.com/stashed/mongodb/releases/tag/4.0.5-v9)

- [ae5c3955](https://github.com/stashed/mongodb/commit/ae5c3955) Prepare for release 4.0.5-v9 (#987)
- [09acaf2c](https://github.com/stashed/mongodb/commit/09acaf2c) [cherry-pick] Use klog/v2 (#969) (#976)
- [a9519e31](https://github.com/stashed/mongodb/commit/a9519e31) [cherry-pick] Use kglog helper (#964)
- [e1bf6ac9](https://github.com/stashed/mongodb/commit/e1bf6ac9) Use k8s 1.21.0 toolchain


### [4.0.11-v9](https://github.com/stashed/mongodb/releases/tag/4.0.11-v9)

- [7b685c24](https://github.com/stashed/mongodb/commit/7b685c24) Prepare for release 4.0.11-v9 (#985)
- [7fdf52db](https://github.com/stashed/mongodb/commit/7fdf52db) [cherry-pick] Use klog/v2 (#969) (#974)
- [880ba142](https://github.com/stashed/mongodb/commit/880ba142) [cherry-pick] Use kglog helper (#962)
- [d555bcd5](https://github.com/stashed/mongodb/commit/d555bcd5) Use k8s 1.21.0 toolchain


### [4.1.4-v9](https://github.com/stashed/mongodb/releases/tag/4.1.4-v9)

- [6e674543](https://github.com/stashed/mongodb/commit/6e674543) Prepare for release 4.1.4-v9 (#989)
- [84241d32](https://github.com/stashed/mongodb/commit/84241d32) [cherry-pick] Use klog/v2 (#969) (#978)
- [795b28db](https://github.com/stashed/mongodb/commit/795b28db) [cherry-pick] Use kglog helper (#966)
- [784b8616](https://github.com/stashed/mongodb/commit/784b8616) Use k8s 1.21.0 toolchain


### [4.1.7-v9](https://github.com/stashed/mongodb/releases/tag/4.1.7-v9)

- [42699cc7](https://github.com/stashed/mongodb/commit/42699cc7) Prepare for release 4.1.7-v9 (#990)
- [8e77beec](https://github.com/stashed/mongodb/commit/8e77beec) [cherry-pick] Use klog/v2 (#969) (#979)
- [6e078f0b](https://github.com/stashed/mongodb/commit/6e078f0b) [cherry-pick] Use kglog helper (#967)
- [ff4a14a3](https://github.com/stashed/mongodb/commit/ff4a14a3) Use k8s 1.21.0 toolchain


### [4.1.13-v9](https://github.com/stashed/mongodb/releases/tag/4.1.13-v9)

- [178750b5](https://github.com/stashed/mongodb/commit/178750b5) Prepare for release 4.1.13-v9 (#988)
- [378f5fb7](https://github.com/stashed/mongodb/commit/378f5fb7) [cherry-pick] Use klog/v2 (#969) (#977)
- [0571b8fd](https://github.com/stashed/mongodb/commit/0571b8fd) [cherry-pick] Use kglog helper (#965)
- [af3b544f](https://github.com/stashed/mongodb/commit/af3b544f) Use k8s 1.21.0 toolchain


### [4.2.3-v9](https://github.com/stashed/mongodb/releases/tag/4.2.3-v9)

- [54bf57ea](https://github.com/stashed/mongodb/commit/54bf57ea) Prepare for release 4.2.3-v9 (#991)
- [f6276ddd](https://github.com/stashed/mongodb/commit/f6276ddd) [cherry-pick] Use klog/v2 (#969) (#980)
- [8ead06e7](https://github.com/stashed/mongodb/commit/8ead06e7) [cherry-pick] Use kglog helper (#968)
- [1be76961](https://github.com/stashed/mongodb/commit/1be76961) Use k8s 1.21.0 toolchain


### [4.4.6](https://github.com/stashed/mongodb/releases/tag/4.4.6)

- [cf1ad82d](https://github.com/stashed/mongodb/commit/cf1ad82d) Stash addon for 4.4.6



## [stashed/mysql](https://github.com/stashed/mysql)

### [5.7.25-v10](https://github.com/stashed/mysql/releases/tag/5.7.25-v10)

- [fbbdea8e](https://github.com/stashed/mysql/commit/fbbdea8e) Prepare for release 5.7.25-v10 (#423)
- [9dbc56ef](https://github.com/stashed/mysql/commit/9dbc56ef) [cherry-pick] Bring back the wait loop (#418) (#419)
- [27d3b793](https://github.com/stashed/mysql/commit/27d3b793) Merge pull request #414 from stashed/master-b0db4d5e-5.7.25
- [363734fc](https://github.com/stashed/mysql/commit/363734fc) Use klog/v2 (#413)
- [84f0a160](https://github.com/stashed/mysql/commit/84f0a160) [cherry-pick] Use kglog helper (#409)
- [c430ec91](https://github.com/stashed/mysql/commit/c430ec91) Use k8s 1.21.0 toolchain


### [8.0.3-v10](https://github.com/stashed/mysql/releases/tag/8.0.3-v10)

- [30d727cb](https://github.com/stashed/mysql/commit/30d727cb) Prepare for release 8.0.3-v10 (#426)
- [641cdaab](https://github.com/stashed/mysql/commit/641cdaab) [cherry-pick] Bring back the wait loop (#418) (#422)
- [abe44d1a](https://github.com/stashed/mysql/commit/abe44d1a) Merge pull request #417 from stashed/master-b0db4d5e-8.0.3
- [2208a806](https://github.com/stashed/mysql/commit/2208a806) Use klog/v2 (#413)
- [5d71f395](https://github.com/stashed/mysql/commit/5d71f395) [cherry-pick] Use kglog helper (#412)
- [37726d79](https://github.com/stashed/mysql/commit/37726d79) Use k8s 1.21.0 toolchain


### [8.0.14-v10](https://github.com/stashed/mysql/releases/tag/8.0.14-v10)

- [deeb3b99](https://github.com/stashed/mysql/commit/deeb3b99) Prepare for release 8.0.14-v10 (#424)
- [dca508ca](https://github.com/stashed/mysql/commit/dca508ca) [cherry-pick] Bring back the wait loop (#418) (#420)
- [1c547c34](https://github.com/stashed/mysql/commit/1c547c34) Merge pull request #415 from stashed/master-b0db4d5e-8.0.14
- [62b23b52](https://github.com/stashed/mysql/commit/62b23b52) Use klog/v2 (#413)
- [ce373065](https://github.com/stashed/mysql/commit/ce373065) [cherry-pick] Use kglog helper (#410)
- [274264cc](https://github.com/stashed/mysql/commit/274264cc) Use k8s 1.21.0 toolchain
- [cbfe8255](https://github.com/stashed/mysql/commit/cbfe8255) [cherry-pick] Update Kubernetes toolchain to v1.21.0 (#377) (#379)


### [8.0.21-v4](https://github.com/stashed/mysql/releases/tag/8.0.21-v4)

- [784b9d43](https://github.com/stashed/mysql/commit/784b9d43) Prepare for release 8.0.21-v4 (#425)
- [00cebf5b](https://github.com/stashed/mysql/commit/00cebf5b) [cherry-pick] Bring back the wait loop (#418) (#421)
- [061cb623](https://github.com/stashed/mysql/commit/061cb623) Merge pull request #416 from stashed/master-b0db4d5e-8.0.21
- [e2b260b6](https://github.com/stashed/mysql/commit/e2b260b6) Use klog/v2 (#413)
- [c56b36ef](https://github.com/stashed/mysql/commit/c56b36ef) [cherry-pick] Use kglog helper (#411)
- [c6999cf0](https://github.com/stashed/mysql/commit/c6999cf0) Use k8s 1.21.0 toolchain



## [stashed/percona-xtradb](https://github.com/stashed/percona-xtradb)

### [5.7-v5](https://github.com/stashed/percona-xtradb/releases/tag/5.7-v5)

- [1e66b08](https://github.com/stashed/percona-xtradb/commit/1e66b08) Prepare for release 5.7-v5 (#181)
- [dedc120](https://github.com/stashed/percona-xtradb/commit/dedc120) [cherry-pick] Use klog/v2 (#179) (#180)
- [95cc309](https://github.com/stashed/percona-xtradb/commit/95cc309) [cherry-pick] Use kglog helper (#178)
- [a8338c8](https://github.com/stashed/percona-xtradb/commit/a8338c8) Use k8s 1.21.0 toolchain
- [a0a9785](https://github.com/stashed/percona-xtradb/commit/a0a9785) [cherry-pick] Use license-verifier v0.8.1 (#168)
- [1f9029a](https://github.com/stashed/percona-xtradb/commit/1f9029a) [cherry-pick] Update license verifier to v0.8.0 (#167)
- [0dd44c3](https://github.com/stashed/percona-xtradb/commit/0dd44c3) [cherry-pick] Update license verifier (#166)
- [20481d6](https://github.com/stashed/percona-xtradb/commit/20481d6) [cherry-pick] Fix spelling (#165)
- [0fd7b26](https://github.com/stashed/percona-xtradb/commit/0fd7b26) Move docs into stashed/docs repo + Cleanup (#164)
- [bc52132](https://github.com/stashed/percona-xtradb/commit/bc52132) Prepare for release 5.7.0-v2 (#161)
- [68eb8b6](https://github.com/stashed/percona-xtradb/commit/68eb8b6) [cherry-pick] Update repository config (#159) (#160)
- [aaecf26](https://github.com/stashed/percona-xtradb/commit/aaecf26) Support multiple commands in backup/restore pipeline (#157) (#158)
- [4c6a086](https://github.com/stashed/percona-xtradb/commit/4c6a086) Update repository config (#153) (#154)
- [22f48d0](https://github.com/stashed/percona-xtradb/commit/22f48d0) Use restic 0.12.0 (#150) (#151)
- [a3645ac](https://github.com/stashed/percona-xtradb/commit/a3645ac) Update Kubernetes v1.18.9 dependencies (#148) (#149)
- [affd376](https://github.com/stashed/percona-xtradb/commit/affd376) [cherry-pick] Check codespan schema (#146) (#147)
- [0c3d3ad](https://github.com/stashed/percona-xtradb/commit/0c3d3ad) [cherry-pick] Update repository config (#141) (#142)
- [937cfc8](https://github.com/stashed/percona-xtradb/commit/937cfc8) [cherry-pick] Update repository config (#139) (#140)
- [8d03640](https://github.com/stashed/percona-xtradb/commit/8d03640) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#137) (#138)
- [d6442e5](https://github.com/stashed/percona-xtradb/commit/d6442e5) [cherry-pick] Update repository config (#134) (#135)
- [301bd0a](https://github.com/stashed/percona-xtradb/commit/301bd0a) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#132) (#133)
- [b9865e8](https://github.com/stashed/percona-xtradb/commit/b9865e8) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#130) (#131)
- [5ab3eaa](https://github.com/stashed/percona-xtradb/commit/5ab3eaa) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#128) (#129)
- [3e48d47](https://github.com/stashed/percona-xtradb/commit/3e48d47) Prepare for release 5.7.0-v1 (#126)
- [42458b3](https://github.com/stashed/percona-xtradb/commit/42458b3) Use port from AppBinding (#120) (#125)
- [70d872d](https://github.com/stashed/percona-xtradb/commit/70d872d) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#123) (#124)
- [2927d77](https://github.com/stashed/percona-xtradb/commit/2927d77) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#121) (#122)
- [fadf2d3](https://github.com/stashed/percona-xtradb/commit/fadf2d3) Prepare for release 5.7.0 (#118)
- [4ea69c8](https://github.com/stashed/percona-xtradb/commit/4ea69c8) [cherry-pick] Update repository config (#116) (#117)
- [a133516](https://github.com/stashed/percona-xtradb/commit/a133516) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#114) (#115)
- [1b7be58](https://github.com/stashed/percona-xtradb/commit/1b7be58) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#112) (#113)



## [stashed/postgres](https://github.com/stashed/postgres)

### [9.6.19-v8](https://github.com/stashed/postgres/releases/tag/9.6.19-v8)

- [788fde1b](https://github.com/stashed/postgres/commit/788fde1b) Prepare for release 9.6.19-v8 (#799)
- [173bb49a](https://github.com/stashed/postgres/commit/173bb49a) [cherry-pick] Use gomodules.xyz/go-sh (#789) (#794)
- [e1192aa4](https://github.com/stashed/postgres/commit/e1192aa4) [cherry-pick] Use klog/v2 (#783) (#788)
- [d936ce1b](https://github.com/stashed/postgres/commit/d936ce1b) [cherry-pick] Use kglog helper (#782)
- [aef3ed32](https://github.com/stashed/postgres/commit/aef3ed32) Use k8s 1.21.0 toolchain


### [10.14-v8](https://github.com/stashed/postgres/releases/tag/10.14-v8)

- [2032c1f6](https://github.com/stashed/postgres/commit/2032c1f6) Prepare for release 10.14-v8 (#795)
- [ba31a9af](https://github.com/stashed/postgres/commit/ba31a9af) [cherry-pick] Use gomodules.xyz/go-sh (#789) (#790)
- [986207ab](https://github.com/stashed/postgres/commit/986207ab) [cherry-pick] Use klog/v2 (#783) (#784)
- [497c60cb](https://github.com/stashed/postgres/commit/497c60cb) [cherry-pick] Use kglog helper (#778)
- [45c6ea73](https://github.com/stashed/postgres/commit/45c6ea73) Use k8s 1.21.0 toolchain


### [11.9-v8](https://github.com/stashed/postgres/releases/tag/11.9-v8)

- [531246ba](https://github.com/stashed/postgres/commit/531246ba) Prepare for release 11.9-v8 (#796)
- [abebf170](https://github.com/stashed/postgres/commit/abebf170) [cherry-pick] Use gomodules.xyz/go-sh (#789) (#791)
- [7d51c48b](https://github.com/stashed/postgres/commit/7d51c48b) [cherry-pick] Use klog/v2 (#783) (#785)
- [65c3506e](https://github.com/stashed/postgres/commit/65c3506e) [cherry-pick] Use kglog helper (#779)
- [984994b5](https://github.com/stashed/postgres/commit/984994b5) Use k8s 1.21.0 toolchain


### [12.4-v8](https://github.com/stashed/postgres/releases/tag/12.4-v8)

- [f8a784c1](https://github.com/stashed/postgres/commit/f8a784c1) Prepare for release 12.4-v8 (#797)
- [0de72265](https://github.com/stashed/postgres/commit/0de72265) [cherry-pick] Use gomodules.xyz/go-sh (#789) (#792)
- [5d325eb4](https://github.com/stashed/postgres/commit/5d325eb4) [cherry-pick] Use klog/v2 (#783) (#786)
- [74d0b192](https://github.com/stashed/postgres/commit/74d0b192) [cherry-pick] Use kglog helper (#780)
- [4d4a1fb8](https://github.com/stashed/postgres/commit/4d4a1fb8) Use k8s 1.21.0 toolchain


### [13.1-v5](https://github.com/stashed/postgres/releases/tag/13.1-v5)

- [20973db6](https://github.com/stashed/postgres/commit/20973db6) Prepare for release 13.1-v5 (#798)
- [85fac622](https://github.com/stashed/postgres/commit/85fac622) [cherry-pick] Use gomodules.xyz/go-sh (#789) (#793)
- [a903b4df](https://github.com/stashed/postgres/commit/a903b4df) [cherry-pick] Use klog/v2 (#783) (#787)
- [9c91b43d](https://github.com/stashed/postgres/commit/9c91b43d) [cherry-pick] Use kglog helper (#781)
- [dc1f023f](https://github.com/stashed/postgres/commit/dc1f023f) Use k8s 1.21.0 toolchain



## [stashed/stash](https://github.com/stashed/stash)

### [v0.14.0](https://github.com/stashed/stash/releases/tag/v0.14.0)

- [028736ed](https://github.com/stashed/stash/commit/028736ed) Prepare for release v0.14.0 (#1354)
- [dc602c0d](https://github.com/stashed/stash/commit/dc602c0d) Prepare for release v0.13.1 (#1353)
- [5d8b403f](https://github.com/stashed/stash/commit/5d8b403f) Send audit events when analytics are enabled (#1352)
- [0d934ce8](https://github.com/stashed/stash/commit/0d934ce8) Use ImagePullPolicy "IfNotPresent" instead of "Always" for addons (#1351)
- [e9f48ec4](https://github.com/stashed/stash/commit/e9f48ec4) Use namespace when looking up running BackupSessions for Invoker (#1343)
- [f79218f1](https://github.com/stashed/stash/commit/f79218f1) Create auditor if license file is provided (#1350)
- [a53710bf](https://github.com/stashed/stash/commit/a53710bf) Disable api priortiy and fairness feature for webhook server (#1349)
- [2780fbc6](https://github.com/stashed/stash/commit/2780fbc6) Publish audit events (#1348)
- [ca26878d](https://github.com/stashed/stash/commit/ca26878d) Use klog/v2 (#1347)
- [f8b0c245](https://github.com/stashed/stash/commit/f8b0c245) Use kglog helpers
- [e9fec332](https://github.com/stashed/stash/commit/e9fec332) Update Kubernetes toolchain to v1.21.0 (#1344)




