---
title: Changelog | Stash
description: Changelog
menu:
  docs_{{.version}}:
    identifier: changelog-stash-v2021.03.08
    name: Changelog-v2021.03.08
    parent: welcome
    weight: 20210308
product_name: stash
menu_name: docs_{{.version}}
section_menu_id: welcome
url: /docs/{{.version}}/welcome/changelog-v2021.03.08/
aliases:
  - /docs/{{.version}}/CHANGELOG-v2021.03.08/
---

# Stash v2021.03.08 (2021-03-09)


## [appscode/stash-enterprise](https://github.com/appscode/stash-enterprise)

### [v0.11.10](https://github.com/appscode/stash-enterprise/releases/tag/v0.11.10)

- [ef190112](https://github.com/appscode/stash-enterprise/commit/ef190112) Prepare for release v0.11.10 (#81)
- [ee861f80](https://github.com/appscode/stash-enterprise/commit/ee861f80) Delete ClusterRoleBinding when the backup/restore invoker is removed (#80)
- [15e9596d](https://github.com/appscode/stash-enterprise/commit/15e9596d) Use addon info from AppBinding for KubeDB managed databases (#79)
- [315d42ae](https://github.com/appscode/stash-enterprise/commit/315d42ae) Update repository config (#77)
- [ce9dd2e4](https://github.com/appscode/stash-enterprise/commit/ce9dd2e4) Update repository config (#76)
- [ec68a28d](https://github.com/appscode/stash-enterprise/commit/ec68a28d) Enable running as a kubedb extension
- [68c9eef3](https://github.com/appscode/stash-enterprise/commit/68c9eef3) Use restic 0.12.0 (#74)
- [50f3d395](https://github.com/appscode/stash-enterprise/commit/50f3d395) Update Kubernetes v1.18.9 dependencies (#73)
- [47cb4438](https://github.com/appscode/stash-enterprise/commit/47cb4438) Update Kubernetes v1.18.9 dependencies (#72)
- [e1293f53](https://github.com/appscode/stash-enterprise/commit/e1293f53) Update Kubernetes v1.18.9 dependencies (#70)
- [6d253b3d](https://github.com/appscode/stash-enterprise/commit/6d253b3d) Update repository config (#67)



## [stashed/apimachinery](https://github.com/stashed/apimachinery)

### [v0.11.10](https://github.com/stashed/apimachinery/releases/tag/v0.11.10)

- [224ba637](https://github.com/stashed/apimachinery/commit/224ba637) Use import-crds script (#95)
- [96a42481](https://github.com/stashed/apimachinery/commit/96a42481) Add helper function to extract addon info from AppBinding (#94)
- [08e07356](https://github.com/stashed/apimachinery/commit/08e07356) Update repository config (#93)
- [6a253c41](https://github.com/stashed/apimachinery/commit/6a253c41) Generate crd yaml for Snapshot (#92)
- [c403645d](https://github.com/stashed/apimachinery/commit/c403645d) verify-gen before verify-modules (#90)
- [b3a033ea](https://github.com/stashed/apimachinery/commit/b3a033ea) Make pipe commands a array + fix output parsing of cleanup command (#87)
- [896c3ef5](https://github.com/stashed/apimachinery/commit/896c3ef5) Revendor kmodules.xyz/custom-resources
- [f29b2219](https://github.com/stashed/apimachinery/commit/f29b2219) Update crd-importer command
- [b39a182f](https://github.com/stashed/apimachinery/commit/b39a182f) Update repository config (#86)
- [680c4df3](https://github.com/stashed/apimachinery/commit/680c4df3) Use restic 0.12.0 (#84)
- [78134f02](https://github.com/stashed/apimachinery/commit/78134f02) Update crds via GitHub actions (#83)
- [a828a28e](https://github.com/stashed/apimachinery/commit/a828a28e) Update Kubernetes v1.18.9 dependencies (#82)
- [456e1a20](https://github.com/stashed/apimachinery/commit/456e1a20) Update Kubernetes v1.18.9 dependencies (#81)



## [stashed/catalog](https://github.com/stashed/catalog)

### [v2021.03.08](https://github.com/stashed/catalog/releases/tag/v2021.03.08)

- [d4246cf](https://github.com/stashed/catalog/commit/d4246cf) Prepare for release v2021.03.08 (#62)
- [1306ba5](https://github.com/stashed/catalog/commit/1306ba5) Generate catalog library using go:embed (#61)
- [5307970](https://github.com/stashed/catalog/commit/5307970) Update repository config (#57)
- [fe7d130](https://github.com/stashed/catalog/commit/fe7d130) Fix passing image args (#56)



## [stashed/cli](https://github.com/stashed/cli)

### [v0.11.10](https://github.com/stashed/cli/releases/tag/v0.11.10)

- [3c37cc4](https://github.com/stashed/cli/commit/3c37cc4) Prepare for release v0.11.10 (#112)
- [fab9603](https://github.com/stashed/cli/commit/fab9603) Update repository config (#110)
- [59b5bea](https://github.com/stashed/cli/commit/59b5bea) Update repository config (#108)
- [5bba6cf](https://github.com/stashed/cli/commit/5bba6cf) Update Kubernetes v1.18.9 dependencies (#107)
- [4c34c15](https://github.com/stashed/cli/commit/4c34c15) Update Kubernetes v1.18.9 dependencies (#106)
- [743a3b9](https://github.com/stashed/cli/commit/743a3b9) Update Kubernetes v1.18.9 dependencies (#105)
- [c305c1a](https://github.com/stashed/cli/commit/c305c1a) Update Kubernetes v1.18.9 dependencies (#104)



## [stashed/elasticsearch](https://github.com/stashed/elasticsearch)

### [5.6.4-v7](https://github.com/stashed/elasticsearch/releases/tag/5.6.4-v7)

- [dcba74b2](https://github.com/stashed/elasticsearch/commit/dcba74b2) Prepare for release 5.6.4-v7 (#708)
- [5a5e1542](https://github.com/stashed/elasticsearch/commit/5a5e1542) Fix Makefile (#699) (#700)
- [537d1393](https://github.com/stashed/elasticsearch/commit/537d1393) Ignore 404 status code when listing legacy templates (#690) (#691)
- [dc8f26f2](https://github.com/stashed/elasticsearch/commit/dc8f26f2) Quote password in authFile (#689)
- [755ffac5](https://github.com/stashed/elasticsearch/commit/755ffac5) [cherry-pick] Update repository config (#674) (#675)
- [27b1ee52](https://github.com/stashed/elasticsearch/commit/27b1ee52) [cherry-pick] Update documentation for KubeDB v1alpha2 (#616) (#666)
- [f49353f2](https://github.com/stashed/elasticsearch/commit/f49353f2) Use auth-file for authentication (#615) (#658)
- [eb8e7ac0](https://github.com/stashed/elasticsearch/commit/eb8e7ac0) Update repository config (#648) (#649)
- [38c6c8f9](https://github.com/stashed/elasticsearch/commit/38c6c8f9) Use restic 0.12.0 (#642) (#643)
- [1c41ea18](https://github.com/stashed/elasticsearch/commit/1c41ea18) Update Kubernetes v1.18.9 dependencies (#633) (#634)
- [9bed4e5c](https://github.com/stashed/elasticsearch/commit/9bed4e5c) [cherry-pick] Check codespan schema (#625) (#626)
- [157cb63a](https://github.com/stashed/elasticsearch/commit/157cb63a) [cherry-pick] Update repository config (#606) (#607)
- [8454a167](https://github.com/stashed/elasticsearch/commit/8454a167) [cherry-pick] Update repository config (#597) (#598)
- [84293f0b](https://github.com/stashed/elasticsearch/commit/84293f0b) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#588) (#589)


### [6.2.4-v7](https://github.com/stashed/elasticsearch/releases/tag/6.2.4-v7)

- [dcdff6ca](https://github.com/stashed/elasticsearch/commit/dcdff6ca) Prepare for release 6.2.4-v7 (#709)
- [8a62603d](https://github.com/stashed/elasticsearch/commit/8a62603d) Fix Makefile (#699) (#701)
- [122f0d5e](https://github.com/stashed/elasticsearch/commit/122f0d5e) Ignore 404 status code when listing legacy templates (#690) (#692)
- [3a9a98f3](https://github.com/stashed/elasticsearch/commit/3a9a98f3) Quote password in authFile (#689)
- [4659e942](https://github.com/stashed/elasticsearch/commit/4659e942) Check codespan schema (#625) (#686)
- [7c8aeb93](https://github.com/stashed/elasticsearch/commit/7c8aeb93) [cherry-pick] Update repository config (#674) (#676)
- [59f32d92](https://github.com/stashed/elasticsearch/commit/59f32d92) [cherry-pick] Update documentation for KubeDB v1alpha2 (#616) (#667)
- [65bbb66f](https://github.com/stashed/elasticsearch/commit/65bbb66f) Use auth-file for authentication (#615) (#659)
- [5ea72da5](https://github.com/stashed/elasticsearch/commit/5ea72da5) Update repository config (#648) (#650)
- [59c7bd9e](https://github.com/stashed/elasticsearch/commit/59c7bd9e) Use restic 0.12.0 (#642) (#644)
- [4d375599](https://github.com/stashed/elasticsearch/commit/4d375599) Update Kubernetes v1.18.9 dependencies (#633) (#635)
- [9f281533](https://github.com/stashed/elasticsearch/commit/9f281533) [cherry-pick] Update repository config (#606) (#608)
- [87352ada](https://github.com/stashed/elasticsearch/commit/87352ada) [cherry-pick] Update repository config (#597) (#599)
- [d8604274](https://github.com/stashed/elasticsearch/commit/d8604274) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#588) (#590)


### [6.3.0-v7](https://github.com/stashed/elasticsearch/releases/tag/6.3.0-v7)

- [e085d630](https://github.com/stashed/elasticsearch/commit/e085d630) Prepare for release 6.3.0-v7 (#710)
- [f045125b](https://github.com/stashed/elasticsearch/commit/f045125b) Fix Makefile (#699) (#702)
- [e8cab7b9](https://github.com/stashed/elasticsearch/commit/e8cab7b9) Ignore 404 status code when listing legacy templates (#690) (#693)
- [e41e7e43](https://github.com/stashed/elasticsearch/commit/e41e7e43) Quote password in authFile (#689)
- [56ab642d](https://github.com/stashed/elasticsearch/commit/56ab642d) [cherry-pick] Update repository config (#674) (#677)
- [69131a02](https://github.com/stashed/elasticsearch/commit/69131a02) [cherry-pick] Update documentation for KubeDB v1alpha2 (#616) (#668)
- [f46a84ae](https://github.com/stashed/elasticsearch/commit/f46a84ae) Use auth-file for authentication (#615) (#660)
- [648d0ff1](https://github.com/stashed/elasticsearch/commit/648d0ff1) Update repository config (#648) (#651)
- [e7201912](https://github.com/stashed/elasticsearch/commit/e7201912) Use restic 0.12.0 (#642) (#645)
- [640e1889](https://github.com/stashed/elasticsearch/commit/640e1889) Update Kubernetes v1.18.9 dependencies (#633) (#636)
- [c82402f4](https://github.com/stashed/elasticsearch/commit/c82402f4) [cherry-pick] Check codespan schema (#625) (#627)
- [af41c505](https://github.com/stashed/elasticsearch/commit/af41c505) [cherry-pick] Update repository config (#606) (#609)
- [898df067](https://github.com/stashed/elasticsearch/commit/898df067) [cherry-pick] Update repository config (#597) (#600)
- [eacda9a1](https://github.com/stashed/elasticsearch/commit/eacda9a1) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#588) (#591)


### [6.4.0-v7](https://github.com/stashed/elasticsearch/releases/tag/6.4.0-v7)

- [5ed94a48](https://github.com/stashed/elasticsearch/commit/5ed94a48) Prepare for release 6.4.0-v7 (#711)
- [35fd1f71](https://github.com/stashed/elasticsearch/commit/35fd1f71) Fix Makefile (#699) (#703)
- [46e0fabb](https://github.com/stashed/elasticsearch/commit/46e0fabb) Ignore 404 status code when listing legacy templates (#690) (#694)
- [9da39aeb](https://github.com/stashed/elasticsearch/commit/9da39aeb) Quote password in authFile (#689)
- [3fb2bd21](https://github.com/stashed/elasticsearch/commit/3fb2bd21) [cherry-pick] Update repository config (#674) (#678)
- [e70e0656](https://github.com/stashed/elasticsearch/commit/e70e0656) [cherry-pick] Update documentation for KubeDB v1alpha2 (#616) (#669)
- [cd226d80](https://github.com/stashed/elasticsearch/commit/cd226d80) Use auth-file for authentication (#615) (#661)
- [edf5ba72](https://github.com/stashed/elasticsearch/commit/edf5ba72) Update repository config (#648) (#652)
- [3d3e99a1](https://github.com/stashed/elasticsearch/commit/3d3e99a1) Use restic 0.12.0 (#642) (#646)
- [0787f7f2](https://github.com/stashed/elasticsearch/commit/0787f7f2) Update Kubernetes v1.18.9 dependencies (#633) (#637)
- [0dafdc7a](https://github.com/stashed/elasticsearch/commit/0dafdc7a) [cherry-pick] Check codespan schema (#625) (#628)
- [54d3204e](https://github.com/stashed/elasticsearch/commit/54d3204e) [cherry-pick] Update repository config (#606) (#610)
- [66df46bd](https://github.com/stashed/elasticsearch/commit/66df46bd) [cherry-pick] Update repository config (#597) (#601)
- [c962f800](https://github.com/stashed/elasticsearch/commit/c962f800) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#588) (#592)


### [6.5.3-v7](https://github.com/stashed/elasticsearch/releases/tag/6.5.3-v7)

- [e44bce97](https://github.com/stashed/elasticsearch/commit/e44bce97) Prepare for release 6.5.3-v7 (#712)
- [e6cee0a2](https://github.com/stashed/elasticsearch/commit/e6cee0a2) Fix Makefile (#699) (#704)
- [d6452e0a](https://github.com/stashed/elasticsearch/commit/d6452e0a) Ignore 404 status code when listing legacy templates (#690) (#695)
- [5b15ac67](https://github.com/stashed/elasticsearch/commit/5b15ac67) Quote password in authFile (#689)
- [ec5efb61](https://github.com/stashed/elasticsearch/commit/ec5efb61) Use restic 0.12.0 (#642) (#682)
- [8101a27e](https://github.com/stashed/elasticsearch/commit/8101a27e) [cherry-pick] Update repository config (#674) (#679)
- [e7b742e0](https://github.com/stashed/elasticsearch/commit/e7b742e0) [cherry-pick] Update documentation for KubeDB v1alpha2 (#616) (#670)
- [bf880257](https://github.com/stashed/elasticsearch/commit/bf880257) Use auth-file for authentication (#615) (#662)
- [d7632175](https://github.com/stashed/elasticsearch/commit/d7632175) Update repository config (#648) (#653)
- [f5c3ad75](https://github.com/stashed/elasticsearch/commit/f5c3ad75) Update Kubernetes v1.18.9 dependencies (#633) (#638)
- [f0da9c7b](https://github.com/stashed/elasticsearch/commit/f0da9c7b) [cherry-pick] Check codespan schema (#625) (#629)
- [fdaaf918](https://github.com/stashed/elasticsearch/commit/fdaaf918) [cherry-pick] Update repository config (#606) (#611)
- [db2b3682](https://github.com/stashed/elasticsearch/commit/db2b3682) [cherry-pick] Update repository config (#597) (#602)
- [bcf79171](https://github.com/stashed/elasticsearch/commit/bcf79171) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#588) (#593)


### [6.8.0-v7](https://github.com/stashed/elasticsearch/releases/tag/6.8.0-v7)

- [0ec393d2](https://github.com/stashed/elasticsearch/commit/0ec393d2) Prepare for release 6.8.0-v7 (#713)
- [4073987f](https://github.com/stashed/elasticsearch/commit/4073987f) Fix Makefile (#699) (#705)
- [04d80d85](https://github.com/stashed/elasticsearch/commit/04d80d85) Ignore 404 status code when listing legacy templates (#690) (#696)
- [dcf26649](https://github.com/stashed/elasticsearch/commit/dcf26649) Quote password in authFile (#689)
- [b0181bc6](https://github.com/stashed/elasticsearch/commit/b0181bc6) Use restic 0.12.0 (#642) (#683)
- [5a2bbc2a](https://github.com/stashed/elasticsearch/commit/5a2bbc2a) [cherry-pick] Update repository config (#674) (#680)
- [587de767](https://github.com/stashed/elasticsearch/commit/587de767) [cherry-pick] Update documentation for KubeDB v1alpha2 (#616) (#671)
- [f69ced9f](https://github.com/stashed/elasticsearch/commit/f69ced9f) Use auth-file for authentication (#615) (#663)
- [f9fea1e4](https://github.com/stashed/elasticsearch/commit/f9fea1e4) Update repository config (#648) (#654)
- [dfe9f1ee](https://github.com/stashed/elasticsearch/commit/dfe9f1ee) Update Kubernetes v1.18.9 dependencies (#633) (#639)
- [efd4b10b](https://github.com/stashed/elasticsearch/commit/efd4b10b) [cherry-pick] Check codespan schema (#625) (#630)
- [39ba5829](https://github.com/stashed/elasticsearch/commit/39ba5829) [cherry-pick] Update repository config (#606) (#612)
- [ab71cf1d](https://github.com/stashed/elasticsearch/commit/ab71cf1d) [cherry-pick] Update repository config (#597) (#603)
- [de556d0f](https://github.com/stashed/elasticsearch/commit/de556d0f) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#588) (#594)


### [7.2.0-v7](https://github.com/stashed/elasticsearch/releases/tag/7.2.0-v7)

- [ea85c44e](https://github.com/stashed/elasticsearch/commit/ea85c44e) Prepare for release 7.2.0-v7 (#714)
- [db945208](https://github.com/stashed/elasticsearch/commit/db945208) Fix Makefile (#699) (#706)
- [a7916ad9](https://github.com/stashed/elasticsearch/commit/a7916ad9) Ignore 404 status code when listing legacy templates (#690) (#697)
- [66cb2cc7](https://github.com/stashed/elasticsearch/commit/66cb2cc7) Quote password in authFile (#689)
- [7118f02d](https://github.com/stashed/elasticsearch/commit/7118f02d) Use restic 0.12.0 (#642) (#684)
- [5cc37a43](https://github.com/stashed/elasticsearch/commit/5cc37a43) [cherry-pick] Update repository config (#674) (#681)
- [3162e056](https://github.com/stashed/elasticsearch/commit/3162e056) [cherry-pick] Update documentation for KubeDB v1alpha2 (#616) (#672)
- [033f476c](https://github.com/stashed/elasticsearch/commit/033f476c) Use auth-file for authentication (#615) (#664)
- [7aa78714](https://github.com/stashed/elasticsearch/commit/7aa78714) Update repository config (#648) (#655)
- [387d719d](https://github.com/stashed/elasticsearch/commit/387d719d) Update Kubernetes v1.18.9 dependencies (#633) (#640)
- [91ee06fd](https://github.com/stashed/elasticsearch/commit/91ee06fd) [cherry-pick] Check codespan schema (#625) (#631)
- [1893fa75](https://github.com/stashed/elasticsearch/commit/1893fa75) [cherry-pick] Update repository config (#606) (#613)
- [bc69b962](https://github.com/stashed/elasticsearch/commit/bc69b962) [cherry-pick] Update repository config (#597) (#604)
- [aa2ec507](https://github.com/stashed/elasticsearch/commit/aa2ec507) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#588) (#595)


### [7.3.2-v7](https://github.com/stashed/elasticsearch/releases/tag/7.3.2-v7)

- [34b60133](https://github.com/stashed/elasticsearch/commit/34b60133) Prepare for release 7.3.2-v7 (#715)
- [a64b4e09](https://github.com/stashed/elasticsearch/commit/a64b4e09) Fix Makefile (#699) (#707)
- [5516debf](https://github.com/stashed/elasticsearch/commit/5516debf) Ignore 404 status code when listing legacy templates (#690) (#698)
- [1af9b4f1](https://github.com/stashed/elasticsearch/commit/1af9b4f1) Quote password in authFile (#689)
- [73f1363d](https://github.com/stashed/elasticsearch/commit/73f1363d) Use Go 1.16 (#625) (#632) (#688)
- [33d55c00](https://github.com/stashed/elasticsearch/commit/33d55c00) Update repository config (#674) (#687)
- [d5acfdfe](https://github.com/stashed/elasticsearch/commit/d5acfdfe) Use restic 0.12.0 (#642) (#685)
- [b1a5ddef](https://github.com/stashed/elasticsearch/commit/b1a5ddef) [cherry-pick] Update documentation for KubeDB v1alpha2 (#616) (#673)
- [3ac327b4](https://github.com/stashed/elasticsearch/commit/3ac327b4) Use auth-file for authentication (#615) (#665)
- [b8d6fd88](https://github.com/stashed/elasticsearch/commit/b8d6fd88) Update Kubernetes v1.18.9 dependencies (#633) (#641)
- [68a99e32](https://github.com/stashed/elasticsearch/commit/68a99e32) Check codespan schema (#625) (#632)
- [2837c894](https://github.com/stashed/elasticsearch/commit/2837c894) [cherry-pick] Update repository config (#606) (#614)
- [e330a642](https://github.com/stashed/elasticsearch/commit/e330a642) [cherry-pick] Update repository config (#597) (#605)
- [a018c4f1](https://github.com/stashed/elasticsearch/commit/a018c4f1) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#588) (#596)



## [stashed/installer](https://github.com/stashed/installer)

### [v0.11.10](https://github.com/stashed/installer/releases/tag/v0.11.10)

- [bc7afef](https://github.com/stashed/installer/commit/bc7afef) Prepare for release v0.11.10 (#156)
- [34d9f1c](https://github.com/stashed/installer/commit/34d9f1c) Avoid duplicating PSPs (#139)
- [bb53887](https://github.com/stashed/installer/commit/bb53887) Fix script permission
- [0982937](https://github.com/stashed/installer/commit/0982937) Add RBAC resources list permission (#153)
- [225c09b](https://github.com/stashed/installer/commit/225c09b) Add import-crds.sh script (#152)
- [ffbd911](https://github.com/stashed/installer/commit/ffbd911) Update crds for stashed/apimachinery@96a42481 (#151)
- [2a66966](https://github.com/stashed/installer/commit/2a66966) make ct (#147)
- [fe6391b](https://github.com/stashed/installer/commit/fe6391b) Remove unused chart templates
- [eb82779](https://github.com/stashed/installer/commit/eb82779) Update repository config (#144)
- [22123dc](https://github.com/stashed/installer/commit/22123dc) Remove v1alpha1 crds from chart
- [e4752a0](https://github.com/stashed/installer/commit/e4752a0) Update stash-crds chart version
- [7fc1b31](https://github.com/stashed/installer/commit/7fc1b31) Update stash community chart name
- [1ccd0d7](https://github.com/stashed/installer/commit/1ccd0d7) Add open-pr.sh script (#143)
- [e9f1dd3](https://github.com/stashed/installer/commit/e9f1dd3) Add stash-crds chart (#142)
- [7fe38fe](https://github.com/stashed/installer/commit/7fe38fe) Rename stash chart to stash-community (#141)



## [stashed/mariadb](https://github.com/stashed/mariadb)

### [10.5.8-v1](https://github.com/stashed/mariadb/releases/tag/10.5.8-v1)

- [b620e7d](https://github.com/stashed/mariadb/commit/b620e7d) Prepare for release 10.5.8-v1 (#83)
- [4c06280](https://github.com/stashed/mariadb/commit/4c06280) [cherry-pick] Adding TLS support on MariaDB plugin (#62) (#82)
- [d6d9a6e](https://github.com/stashed/mariadb/commit/d6d9a6e) [cherry-pick] Update Docs (#79) (#81)
- [04c81ed](https://github.com/stashed/mariadb/commit/04c81ed) [cherry-pick] Support multiple commands in backup/restore pipeline (#77) (#80)
- [95f51d1](https://github.com/stashed/mariadb/commit/95f51d1) [cherry-pick] Update repository config (#75) (#78)
- [25933ed](https://github.com/stashed/mariadb/commit/25933ed) Update repository config (#73) (#74)
- [c0e03fe](https://github.com/stashed/mariadb/commit/c0e03fe) Use restic 0.12.0 (#70) (#71)
- [41082a8](https://github.com/stashed/mariadb/commit/41082a8) Update Kubernetes v1.18.9 dependencies (#68) (#69)
- [0e8a6d6](https://github.com/stashed/mariadb/commit/0e8a6d6) Fix links to concept docs (#56) (#67)
- [68fe2fe](https://github.com/stashed/mariadb/commit/68fe2fe) [cherry-pick] Check codespan schema (#65) (#66)
- [26f0958](https://github.com/stashed/mariadb/commit/26f0958) [cherry-pick] Update repository config (#59) (#60)
- [5d08f71](https://github.com/stashed/mariadb/commit/5d08f71) [cherry-pick] Update repository config (#57) (#58)
- [e076ca9](https://github.com/stashed/mariadb/commit/e076ca9) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#54) (#55)
- [eee3847](https://github.com/stashed/mariadb/commit/eee3847) [cherry-pick] Update repository config (#52) (#53)
- [6e1230c](https://github.com/stashed/mariadb/commit/6e1230c) [cherry-pick] Update repository config (#50) (#51)
- [cd1252b](https://github.com/stashed/mariadb/commit/cd1252b) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#48) (#49)
- [bee27ba](https://github.com/stashed/mariadb/commit/bee27ba) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#46) (#47)
- [eba6dad](https://github.com/stashed/mariadb/commit/eba6dad) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#44) (#45)
- [59d93e6](https://github.com/stashed/mariadb/commit/59d93e6) [cherry-pick] Generate README.md from templates (#42) (#43)
- [5763554](https://github.com/stashed/mariadb/commit/5763554) [cherry-pick] Speed up schema generation process (#40) (#41)



## [stashed/mongodb](https://github.com/stashed/mongodb)

### [3.4.17-v6](https://github.com/stashed/mongodb/releases/tag/3.4.17-v6)

- [1d030cb5](https://github.com/stashed/mongodb/commit/1d030cb5) Prepare for release 3.4.17-v6 (#843)
- [6aead6a1](https://github.com/stashed/mongodb/commit/6aead6a1) [cherry-pick] Update repository config (#834) (#835)
- [605d610e](https://github.com/stashed/mongodb/commit/605d610e) [cherry-pick] Support multiple commands in backup/restore pipeline (#823) (#824)
- [af2471a0](https://github.com/stashed/mongodb/commit/af2471a0) Update repository config (#809) (#810)
- [834dd170](https://github.com/stashed/mongodb/commit/834dd170) Use restic 0.12.0 (#799) (#800)
- [aa183f6d](https://github.com/stashed/mongodb/commit/aa183f6d) Update Kubernetes v1.18.9 dependencies (#788) (#789)
- [69115627](https://github.com/stashed/mongodb/commit/69115627) [cherry-pick] Check codespan schema (#776) (#777)
- [ea1b54f2](https://github.com/stashed/mongodb/commit/ea1b54f2) [cherry-pick] Update repository config (#751) (#752)
- [a4a9dffe](https://github.com/stashed/mongodb/commit/a4a9dffe) [cherry-pick] Update repository config (#742) (#743)
- [d7815dec](https://github.com/stashed/mongodb/commit/d7815dec) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#730) (#731)
- [f2c6e6f2](https://github.com/stashed/mongodb/commit/f2c6e6f2) [cherry-pick] Update repository config (#723) (#724)
- [739c59f8](https://github.com/stashed/mongodb/commit/739c59f8) [cherry-pick] Update repository config (#711) (#712)
- [a8f735f3](https://github.com/stashed/mongodb/commit/a8f735f3) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#699) (#700)
- [43bfddc1](https://github.com/stashed/mongodb/commit/43bfddc1) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#687) (#688)
- [91b544ba](https://github.com/stashed/mongodb/commit/91b544ba) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#675) (#676)
- [a457ccd6](https://github.com/stashed/mongodb/commit/a457ccd6) [cherry-pick] Speed up schema generation process (#663) (#664)


### [3.4.22-v6](https://github.com/stashed/mongodb/releases/tag/3.4.22-v6)

- [cb42eb68](https://github.com/stashed/mongodb/commit/cb42eb68) Prepare for release 3.4.22-v6 (#844)
- [bf775397](https://github.com/stashed/mongodb/commit/bf775397) [cherry-pick] Update repository config (#834) (#836)
- [bf74b479](https://github.com/stashed/mongodb/commit/bf74b479) [cherry-pick] Support multiple commands in backup/restore pipeline (#823) (#825)
- [f00814d0](https://github.com/stashed/mongodb/commit/f00814d0) Update repository config (#809) (#811)
- [892cdd6e](https://github.com/stashed/mongodb/commit/892cdd6e) Use restic 0.12.0 (#799) (#801)
- [226e63ed](https://github.com/stashed/mongodb/commit/226e63ed) Update Kubernetes v1.18.9 dependencies (#788) (#790)
- [a2278585](https://github.com/stashed/mongodb/commit/a2278585) [cherry-pick] Check codespan schema (#776) (#778)
- [9ad4d52e](https://github.com/stashed/mongodb/commit/9ad4d52e) [cherry-pick] Update repository config (#751) (#753)
- [84f82ec0](https://github.com/stashed/mongodb/commit/84f82ec0) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#730) (#732)
- [df61f073](https://github.com/stashed/mongodb/commit/df61f073) [cherry-pick] Update repository config (#723) (#725)
- [5fa8af10](https://github.com/stashed/mongodb/commit/5fa8af10) [cherry-pick] Update repository config (#711) (#713)
- [3daa60e9](https://github.com/stashed/mongodb/commit/3daa60e9) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#699) (#701)
- [5179d087](https://github.com/stashed/mongodb/commit/5179d087) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#687) (#689)
- [183c0a71](https://github.com/stashed/mongodb/commit/183c0a71) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#675) (#677)
- [ab565586](https://github.com/stashed/mongodb/commit/ab565586) [cherry-pick] Speed up schema generation process (#663) (#665)


### [3.6.8-v6](https://github.com/stashed/mongodb/releases/tag/3.6.8-v6)

- [20ebf67e](https://github.com/stashed/mongodb/commit/20ebf67e) Prepare for release 3.6.8-v6 (#846)
- [0d22261d](https://github.com/stashed/mongodb/commit/0d22261d) [cherry-pick] Support multiple commands in backup/restore pipeline (#823) (#827)
- [ade100a1](https://github.com/stashed/mongodb/commit/ade100a1) Update repository config (#809) (#813)
- [0dccae77](https://github.com/stashed/mongodb/commit/0dccae77) Use restic 0.12.0 (#799) (#803)
- [be4a808c](https://github.com/stashed/mongodb/commit/be4a808c) Update Kubernetes v1.18.9 dependencies (#788) (#792)
- [98181167](https://github.com/stashed/mongodb/commit/98181167) [cherry-pick] Check codespan schema (#776) (#780)
- [1cbdbf2a](https://github.com/stashed/mongodb/commit/1cbdbf2a) [cherry-pick] Update repository config (#751) (#755)
- [f09ed6dc](https://github.com/stashed/mongodb/commit/f09ed6dc) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#730) (#734)
- [5df0856f](https://github.com/stashed/mongodb/commit/5df0856f) [cherry-pick] Update repository config (#723) (#727)
- [3037de41](https://github.com/stashed/mongodb/commit/3037de41) [cherry-pick] Update repository config (#711) (#715)
- [214a2741](https://github.com/stashed/mongodb/commit/214a2741) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#699) (#703)
- [75ae3f11](https://github.com/stashed/mongodb/commit/75ae3f11) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#687) (#691)
- [effbb2a3](https://github.com/stashed/mongodb/commit/effbb2a3) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#675) (#679)
- [5cfd4984](https://github.com/stashed/mongodb/commit/5cfd4984) [cherry-pick] Speed up schema generation process (#663) (#667)


### [3.6.13-v6](https://github.com/stashed/mongodb/releases/tag/3.6.13-v6)

- [c857aca3](https://github.com/stashed/mongodb/commit/c857aca3) Prepare for release 3.6.13-v6 (#845)
- [4e30586c](https://github.com/stashed/mongodb/commit/4e30586c) [cherry-pick] Update repository config (#834) (#837)
- [98503034](https://github.com/stashed/mongodb/commit/98503034) [cherry-pick] Support multiple commands in backup/restore pipeline (#823) (#826)
- [0d0f2c56](https://github.com/stashed/mongodb/commit/0d0f2c56) Update repository config (#809) (#812)
- [d44a02ee](https://github.com/stashed/mongodb/commit/d44a02ee) Use restic 0.12.0 (#799) (#802)
- [15d8451c](https://github.com/stashed/mongodb/commit/15d8451c) Update Kubernetes v1.18.9 dependencies (#788) (#791)
- [a5dd78c9](https://github.com/stashed/mongodb/commit/a5dd78c9) [cherry-pick] Check codespan schema (#776) (#779)
- [af7e1c21](https://github.com/stashed/mongodb/commit/af7e1c21) [cherry-pick] Update repository config (#751) (#754)
- [8f1c8e34](https://github.com/stashed/mongodb/commit/8f1c8e34) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#730) (#733)
- [504df2a0](https://github.com/stashed/mongodb/commit/504df2a0) [cherry-pick] Update repository config (#723) (#726)
- [b2d56067](https://github.com/stashed/mongodb/commit/b2d56067) [cherry-pick] Update repository config (#711) (#714)
- [34db4b8d](https://github.com/stashed/mongodb/commit/34db4b8d) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#699) (#702)
- [393ee1ed](https://github.com/stashed/mongodb/commit/393ee1ed) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#687) (#690)
- [46630738](https://github.com/stashed/mongodb/commit/46630738) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#675) (#678)
- [2628b661](https://github.com/stashed/mongodb/commit/2628b661) [cherry-pick] Speed up schema generation process (#663) (#666)


### [4.0.3-v6](https://github.com/stashed/mongodb/releases/tag/4.0.3-v6)

- [c4d3534d](https://github.com/stashed/mongodb/commit/c4d3534d) Prepare for release 4.0.3-v6 (#848)
- [ba7946f5](https://github.com/stashed/mongodb/commit/ba7946f5) [cherry-pick] Support multiple commands in backup/restore pipeline (#823) (#829)
- [cdf8c4ec](https://github.com/stashed/mongodb/commit/cdf8c4ec) Update repository config (#809) (#815)
- [d33cb16d](https://github.com/stashed/mongodb/commit/d33cb16d) Use restic 0.12.0 (#799) (#805)
- [6695c171](https://github.com/stashed/mongodb/commit/6695c171) Update Kubernetes v1.18.9 dependencies (#788) (#794)
- [7a7ef475](https://github.com/stashed/mongodb/commit/7a7ef475) [cherry-pick] Check codespan schema (#776) (#782)
- [3ed7924b](https://github.com/stashed/mongodb/commit/3ed7924b) [cherry-pick] Update repository config (#751) (#757)
- [dc8f11da](https://github.com/stashed/mongodb/commit/dc8f11da) [cherry-pick] Update repository config (#742) (#745)
- [b5cee38e](https://github.com/stashed/mongodb/commit/b5cee38e) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#730) (#736)
- [92fc2ef0](https://github.com/stashed/mongodb/commit/92fc2ef0) [cherry-pick] Update repository config (#723) (#729)
- [0a722b53](https://github.com/stashed/mongodb/commit/0a722b53) [cherry-pick] Update repository config (#711) (#717)
- [6487d000](https://github.com/stashed/mongodb/commit/6487d000) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#699) (#705)
- [02b8fa07](https://github.com/stashed/mongodb/commit/02b8fa07) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#687) (#693)
- [90d43ee5](https://github.com/stashed/mongodb/commit/90d43ee5) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#675) (#681)
- [296e3e78](https://github.com/stashed/mongodb/commit/296e3e78) [cherry-pick] Speed up schema generation process (#663) (#669)


### [4.0.5-v6](https://github.com/stashed/mongodb/releases/tag/4.0.5-v6)

- [61478c2a](https://github.com/stashed/mongodb/commit/61478c2a) Prepare for release 4.0.5-v6 (#849)
- [68a14362](https://github.com/stashed/mongodb/commit/68a14362) [cherry-pick] Support multiple commands in backup/restore pipeline (#823) (#830)
- [44c65016](https://github.com/stashed/mongodb/commit/44c65016) Update repository config (#809) (#816)
- [a86475a8](https://github.com/stashed/mongodb/commit/a86475a8) Update Kubernetes v1.18.9 dependencies (#788) (#795)
- [c2bf2223](https://github.com/stashed/mongodb/commit/c2bf2223) [cherry-pick] Check codespan schema (#776) (#783)
- [3f1e60bf](https://github.com/stashed/mongodb/commit/3f1e60bf) [cherry-pick] Update repository config (#751) (#758)
- [917188b5](https://github.com/stashed/mongodb/commit/917188b5) [cherry-pick] Update repository config (#742) (#746)
- [8ee922e7](https://github.com/stashed/mongodb/commit/8ee922e7) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#730) (#737)
- [d5712991](https://github.com/stashed/mongodb/commit/d5712991) [cherry-pick] Update repository config (#711) (#718)
- [d2c6e8a1](https://github.com/stashed/mongodb/commit/d2c6e8a1) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#699) (#706)
- [9f99d7ed](https://github.com/stashed/mongodb/commit/9f99d7ed) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#687) (#694)
- [ba5273b3](https://github.com/stashed/mongodb/commit/ba5273b3) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#675) (#682)
- [be53a7ba](https://github.com/stashed/mongodb/commit/be53a7ba) [cherry-pick] Speed up schema generation process (#663) (#670)


### [4.0.11-v6](https://github.com/stashed/mongodb/releases/tag/4.0.11-v6)

- [42b8d5b5](https://github.com/stashed/mongodb/commit/42b8d5b5) Prepare for release 4.0.11-v6 (#847)
- [3ef255bf](https://github.com/stashed/mongodb/commit/3ef255bf) [cherry-pick] Support multiple commands in backup/restore pipeline (#823) (#828)
- [fb647f1a](https://github.com/stashed/mongodb/commit/fb647f1a) Update repository config (#809) (#814)
- [7408bd14](https://github.com/stashed/mongodb/commit/7408bd14) Use restic 0.12.0 (#799) (#804)
- [647a9b97](https://github.com/stashed/mongodb/commit/647a9b97) Update Kubernetes v1.18.9 dependencies (#788) (#793)
- [e00d3aa8](https://github.com/stashed/mongodb/commit/e00d3aa8) [cherry-pick] Check codespan schema (#776) (#781)
- [b3049dd7](https://github.com/stashed/mongodb/commit/b3049dd7) [cherry-pick] Update repository config (#751) (#756)
- [459c1f22](https://github.com/stashed/mongodb/commit/459c1f22) [cherry-pick] Update repository config (#742) (#744)
- [95525f30](https://github.com/stashed/mongodb/commit/95525f30) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#730) (#735)
- [b4eeea5a](https://github.com/stashed/mongodb/commit/b4eeea5a) [cherry-pick] Update repository config (#723) (#728)
- [e2b3dac7](https://github.com/stashed/mongodb/commit/e2b3dac7) [cherry-pick] Update repository config (#711) (#716)
- [30038e11](https://github.com/stashed/mongodb/commit/30038e11) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#699) (#704)
- [a731b4f8](https://github.com/stashed/mongodb/commit/a731b4f8) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#687) (#692)
- [d45bce7d](https://github.com/stashed/mongodb/commit/d45bce7d) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#675) (#680)
- [305e2f09](https://github.com/stashed/mongodb/commit/305e2f09) [cherry-pick] Speed up schema generation process (#663) (#668)


### [4.1.4-v6](https://github.com/stashed/mongodb/releases/tag/4.1.4-v6)

- [d897c8af](https://github.com/stashed/mongodb/commit/d897c8af) Prepare for release 4.1.4-v6 (#851)
- [ea3fe513](https://github.com/stashed/mongodb/commit/ea3fe513) [cherry-pick] Update repository config (#834) (#839)
- [131e8186](https://github.com/stashed/mongodb/commit/131e8186) [cherry-pick] Support multiple commands in backup/restore pipeline (#823) (#832)
- [fc8aadb3](https://github.com/stashed/mongodb/commit/fc8aadb3) Update repository config (#809) (#818)
- [540bb495](https://github.com/stashed/mongodb/commit/540bb495) [cherry-pick] Check codespan schema (#776) (#785)
- [f0223f83](https://github.com/stashed/mongodb/commit/f0223f83) [cherry-pick] Update repository config (#751) (#760)
- [181d91ff](https://github.com/stashed/mongodb/commit/181d91ff) [cherry-pick] Update repository config (#742) (#748)
- [bae3a785](https://github.com/stashed/mongodb/commit/bae3a785) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#730) (#739)
- [6794e116](https://github.com/stashed/mongodb/commit/6794e116) [cherry-pick] Update repository config (#711) (#720)
- [6679db70](https://github.com/stashed/mongodb/commit/6679db70) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#699) (#708)
- [a1e1d7ad](https://github.com/stashed/mongodb/commit/a1e1d7ad) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#687) (#696)
- [38234e03](https://github.com/stashed/mongodb/commit/38234e03) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#675) (#684)
- [bc3a9d3f](https://github.com/stashed/mongodb/commit/bc3a9d3f) [cherry-pick] Speed up schema generation process (#663) (#672)


### [4.1.7-v6](https://github.com/stashed/mongodb/releases/tag/4.1.7-v6)

- [ea23f76b](https://github.com/stashed/mongodb/commit/ea23f76b) Prepare for release 4.1.7-v6 (#852)
- [0fc5aa51](https://github.com/stashed/mongodb/commit/0fc5aa51) [cherry-pick] Update repository config (#834) (#840)
- [7f3fd5ba](https://github.com/stashed/mongodb/commit/7f3fd5ba) [cherry-pick] Support multiple commands in backup/restore pipeline (#823) (#833)
- [f35f4d0a](https://github.com/stashed/mongodb/commit/f35f4d0a) Update repository config (#809) (#819)
- [dea74649](https://github.com/stashed/mongodb/commit/dea74649) Use restic 0.12.0 (#799) (#806)
- [eb99931f](https://github.com/stashed/mongodb/commit/eb99931f) Update Kubernetes v1.18.9 dependencies (#788) (#797)
- [0f047a25](https://github.com/stashed/mongodb/commit/0f047a25) [cherry-pick] Check codespan schema (#776) (#786)
- [27a57e7b](https://github.com/stashed/mongodb/commit/27a57e7b) [cherry-pick] Update repository config (#751) (#761)
- [1e887f7c](https://github.com/stashed/mongodb/commit/1e887f7c) [cherry-pick] Update repository config (#742) (#749)
- [d4cd6f9b](https://github.com/stashed/mongodb/commit/d4cd6f9b) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#730) (#740)
- [bd003974](https://github.com/stashed/mongodb/commit/bd003974) [cherry-pick] Update repository config (#711) (#721)
- [3fc526dc](https://github.com/stashed/mongodb/commit/3fc526dc) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#699) (#709)
- [b62a22cc](https://github.com/stashed/mongodb/commit/b62a22cc) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#687) (#697)
- [78981022](https://github.com/stashed/mongodb/commit/78981022) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#675) (#685)
- [e660275a](https://github.com/stashed/mongodb/commit/e660275a) [cherry-pick] Speed up schema generation process (#663) (#673)


### [4.1.13-v6](https://github.com/stashed/mongodb/releases/tag/4.1.13-v6)

- [7dd79a12](https://github.com/stashed/mongodb/commit/7dd79a12) Prepare for release 4.1.13-v6 (#850)
- [2e0bf537](https://github.com/stashed/mongodb/commit/2e0bf537) [cherry-pick] Update repository config (#834) (#838)
- [5f423a11](https://github.com/stashed/mongodb/commit/5f423a11) [cherry-pick] Support multiple commands in backup/restore pipeline (#823) (#831)
- [16e63401](https://github.com/stashed/mongodb/commit/16e63401) Update repository config (#809) (#817)
- [6e5807e9](https://github.com/stashed/mongodb/commit/6e5807e9) Update Kubernetes v1.18.9 dependencies (#788) (#796)
- [00e7f5fa](https://github.com/stashed/mongodb/commit/00e7f5fa) [cherry-pick] Check codespan schema (#776) (#784)
- [e70421c3](https://github.com/stashed/mongodb/commit/e70421c3) [cherry-pick] Update repository config (#751) (#759)
- [349469bf](https://github.com/stashed/mongodb/commit/349469bf) [cherry-pick] Update repository config (#742) (#747)
- [20ab2b31](https://github.com/stashed/mongodb/commit/20ab2b31) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#730) (#738)
- [db398d0f](https://github.com/stashed/mongodb/commit/db398d0f) [cherry-pick] Update repository config (#711) (#719)
- [d7df2584](https://github.com/stashed/mongodb/commit/d7df2584) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#699) (#707)
- [c8ed983d](https://github.com/stashed/mongodb/commit/c8ed983d) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#687) (#695)
- [17fd7bb0](https://github.com/stashed/mongodb/commit/17fd7bb0) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#675) (#683)
- [ed17070e](https://github.com/stashed/mongodb/commit/ed17070e) [cherry-pick] Speed up schema generation process (#663) (#671)


### [4.2.3-v6](https://github.com/stashed/mongodb/releases/tag/4.2.3-v6)

- [c9fe7d9a](https://github.com/stashed/mongodb/commit/c9fe7d9a) Prepare for release 4.2.3-v6 (#853)
- [76b6c898](https://github.com/stashed/mongodb/commit/76b6c898) [cherry-pick] Update repository config (#834) (#841)
- [f1f64838](https://github.com/stashed/mongodb/commit/f1f64838) Support multiple commands in backup/restore pipeline (#823) (#842)
- [a12d4baf](https://github.com/stashed/mongodb/commit/a12d4baf) Update repository config (#809) (#820)
- [4fb8b170](https://github.com/stashed/mongodb/commit/4fb8b170) Use restic 0.12.0 (#799) (#807)
- [46f2effa](https://github.com/stashed/mongodb/commit/46f2effa) Update Kubernetes v1.18.9 dependencies (#788) (#798)
- [e10f42bf](https://github.com/stashed/mongodb/commit/e10f42bf) [cherry-pick] Check codespan schema (#776) (#787)
- [8c09902f](https://github.com/stashed/mongodb/commit/8c09902f) [cherry-pick] Update repository config (#751) (#762)
- [0cc4c0bf](https://github.com/stashed/mongodb/commit/0cc4c0bf) [cherry-pick] Update repository config (#742) (#750)
- [08f5ff3b](https://github.com/stashed/mongodb/commit/08f5ff3b) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#730) (#741)
- [78a92f84](https://github.com/stashed/mongodb/commit/78a92f84) [cherry-pick] Update repository config (#711) (#722)
- [705dc0f9](https://github.com/stashed/mongodb/commit/705dc0f9) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#699) (#710)
- [08aecd88](https://github.com/stashed/mongodb/commit/08aecd88) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#687) (#698)
- [c7f8715b](https://github.com/stashed/mongodb/commit/c7f8715b) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#675) (#686)
- [8d3f3e59](https://github.com/stashed/mongodb/commit/8d3f3e59) [cherry-pick] Speed up schema generation process (#663) (#674)



## [stashed/mysql](https://github.com/stashed/mysql)

### [5.7.25-v7](https://github.com/stashed/mysql/releases/tag/5.7.25-v7)

- [8a1ee71](https://github.com/stashed/mysql/commit/8a1ee71) Prepare for release 5.7.25-v7 (#349)
- [366167a](https://github.com/stashed/mysql/commit/366167a) [cherry-pick] Update repository config (#344) (#345)
- [96f4404](https://github.com/stashed/mysql/commit/96f4404) [cherry-pick] Support multiple commands in backup/restore pipeline (#335) (#340)
- [02a3750](https://github.com/stashed/mysql/commit/02a3750) [cherry-pick] Add TLS support for backup and Restore (#328) (#336)
- [e8c8781](https://github.com/stashed/mysql/commit/e8c8781) Update repository config (#327) (#329)
- [864ba45](https://github.com/stashed/mysql/commit/864ba45) Use restic 0.12.0 (#321) (#322)
- [5b26633](https://github.com/stashed/mysql/commit/5b26633) Update Kubernetes v1.18.9 dependencies (#316) (#317)
- [b17c4b4](https://github.com/stashed/mysql/commit/b17c4b4) [cherry-pick] Check codespan schema (#312) (#313)
- [bd1460c](https://github.com/stashed/mysql/commit/bd1460c) [cherry-pick] Update repository config (#301) (#302)
- [4a19d88](https://github.com/stashed/mysql/commit/4a19d88) [cherry-pick] Update repository config (#296) (#297)
- [2e1a630](https://github.com/stashed/mysql/commit/2e1a630) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#291) (#292)


### [8.0.3-v7](https://github.com/stashed/mysql/releases/tag/8.0.3-v7)

- [8a5311d](https://github.com/stashed/mysql/commit/8a5311d) Prepare for release 8.0.3-v7 (#352)
- [60a0157](https://github.com/stashed/mysql/commit/60a0157) [cherry-pick] Update repository config (#344) (#348)
- [a0e9506](https://github.com/stashed/mysql/commit/a0e9506) [cherry-pick] Support multiple commands in backup/restore pipeline (#335) (#343)
- [f4a09d8](https://github.com/stashed/mysql/commit/f4a09d8) [cherry-pick] Add TLS support for backup and Restore (#328) (#339)
- [271db44](https://github.com/stashed/mysql/commit/271db44) Update repository config (#327) (#332)
- [1af5d1b](https://github.com/stashed/mysql/commit/1af5d1b) Use restic 0.12.0 (#321) (#325)
- [3c6d8f7](https://github.com/stashed/mysql/commit/3c6d8f7) Update Kubernetes v1.18.9 dependencies (#316) (#320)
- [dbd0d49](https://github.com/stashed/mysql/commit/dbd0d49) [cherry-pick] Update repository config (#301) (#305)
- [889711e](https://github.com/stashed/mysql/commit/889711e) [cherry-pick] Update repository config (#296) (#300)
- [1418e6d](https://github.com/stashed/mysql/commit/1418e6d) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#291) (#295)


### [8.0.14-v7](https://github.com/stashed/mysql/releases/tag/8.0.14-v7)

- [a38a02b](https://github.com/stashed/mysql/commit/a38a02b) Prepare for release 8.0.14-v7 (#350)
- [b4091b6](https://github.com/stashed/mysql/commit/b4091b6) [cherry-pick] Update repository config (#344) (#346)
- [2218549](https://github.com/stashed/mysql/commit/2218549) [cherry-pick] Support multiple commands in backup/restore pipeline (#335) (#341)
- [5e38d81](https://github.com/stashed/mysql/commit/5e38d81) [cherry-pick] Add TLS support for backup and Restore (#328) (#337)
- [431967e](https://github.com/stashed/mysql/commit/431967e) Update repository config (#327) (#330)
- [d6adbb2](https://github.com/stashed/mysql/commit/d6adbb2) Use restic 0.12.0 (#321) (#323)
- [d3cf790](https://github.com/stashed/mysql/commit/d3cf790) Update Kubernetes v1.18.9 dependencies (#316) (#318)
- [5e2356e](https://github.com/stashed/mysql/commit/5e2356e) [cherry-pick] Check codespan schema (#312) (#314)
- [bbbbc8e](https://github.com/stashed/mysql/commit/bbbbc8e) [cherry-pick] Update repository config (#301) (#303)
- [00ddfb4](https://github.com/stashed/mysql/commit/00ddfb4) [cherry-pick] Update repository config (#296) (#298)
- [08189bf](https://github.com/stashed/mysql/commit/08189bf) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#291) (#293)


### [8.0.21-v1](https://github.com/stashed/mysql/releases/tag/8.0.21-v1)

- [df6aa7c](https://github.com/stashed/mysql/commit/df6aa7c) Prepare for release 8.0.21-v1 (#351)
- [bb35f6b](https://github.com/stashed/mysql/commit/bb35f6b) [cherry-pick] Update repository config (#344) (#347)
- [8571d10](https://github.com/stashed/mysql/commit/8571d10) [cherry-pick] Support multiple commands in backup/restore pipeline (#335) (#342)
- [e3bb016](https://github.com/stashed/mysql/commit/e3bb016) [cherry-pick] Add TLS support for backup and Restore (#328) (#338)
- [c9a5ddf](https://github.com/stashed/mysql/commit/c9a5ddf) Update repository config (#327) (#331)
- [4e0a065](https://github.com/stashed/mysql/commit/4e0a065) Use restic 0.12.0 (#321) (#324)
- [367e1ce](https://github.com/stashed/mysql/commit/367e1ce) Update Kubernetes v1.18.9 dependencies (#316) (#319)
- [a09b250](https://github.com/stashed/mysql/commit/a09b250) [cherry-pick] Check codespan schema (#312) (#315)
- [344d3ac](https://github.com/stashed/mysql/commit/344d3ac) [cherry-pick] Update repository config (#301) (#304)
- [fcc0e9b](https://github.com/stashed/mysql/commit/fcc0e9b) [cherry-pick] Update repository config (#296) (#299)
- [c4ab29f](https://github.com/stashed/mysql/commit/c4ab29f) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#291) (#294)



## [stashed/percona-xtradb](https://github.com/stashed/percona-xtradb)

### [5.7.0-v2](https://github.com/stashed/percona-xtradb/releases/tag/5.7.0-v2)




## [stashed/postgres](https://github.com/stashed/postgres)

### [9.6.19-v5](https://github.com/stashed/postgres/releases/tag/9.6.19-v5)

- [66a76378](https://github.com/stashed/postgres/commit/66a76378) Prepare for release 9.6.19-v5 (#720)
- [362b2897](https://github.com/stashed/postgres/commit/362b2897) Fix Makefile (#705) (#715)
- [9c47a1fd](https://github.com/stashed/postgres/commit/9c47a1fd) TLS support for postgres (#694) (#704)
- [e2ad17a8](https://github.com/stashed/postgres/commit/e2ad17a8) [cherry-pick] Update repository config (#683) (#693)
- [e739d000](https://github.com/stashed/postgres/commit/e739d000) Add auto-backup doc + Restructure docs (#618) (#682)
- [3b4f6709](https://github.com/stashed/postgres/commit/3b4f6709) [cherry-pick] Don't overwrite superuser password of restored database (#660) (#672)
- [b5b9acb1](https://github.com/stashed/postgres/commit/b5b9acb1) Update repository config (#649) (#659)
- [d7fc51c9](https://github.com/stashed/postgres/commit/d7fc51c9) Use restic 0.12.0 (#637) (#647)
- [3a8c97eb](https://github.com/stashed/postgres/commit/3a8c97eb) Update Kubernetes v1.18.9 dependencies (#626) (#636)
- [2adf3188](https://github.com/stashed/postgres/commit/2adf3188) [cherry-pick] Update repository config (#600) (#610)
- [4143a5e1](https://github.com/stashed/postgres/commit/4143a5e1) [cherry-pick] Update repository config (#589) (#599)
- [31a2d0c1](https://github.com/stashed/postgres/commit/31a2d0c1) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#578) (#588)
- [a40a7321](https://github.com/stashed/postgres/commit/a40a7321) [cherry-pick] Update repository config (#566) (#576)
- [05b91feb](https://github.com/stashed/postgres/commit/05b91feb) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#555) (#565)
- [4f5958c7](https://github.com/stashed/postgres/commit/4f5958c7) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#544) (#554)
- [b70c6cfa](https://github.com/stashed/postgres/commit/b70c6cfa) [cherry-pick] Update README.md template (#523) (#533)
- [850e18b3](https://github.com/stashed/postgres/commit/850e18b3) [cherry-pick] Generate README.md using templates (#512) (#522)
- [4ccb4b4a](https://github.com/stashed/postgres/commit/4ccb4b4a) [cherry-pick] Speed up schema generation process (#501) (#511)


### [10.14.0-v5](https://github.com/stashed/postgres/releases/tag/10.14.0-v5)

- [57eecdc3](https://github.com/stashed/postgres/commit/57eecdc3) Prepare for release 10.14.0-v5 (#716)
- [9a56adea](https://github.com/stashed/postgres/commit/9a56adea) Fix Makefile (#705) (#706)
- [925bd8e6](https://github.com/stashed/postgres/commit/925bd8e6) TLS support for postgres (#694) (#695)
- [6509c44f](https://github.com/stashed/postgres/commit/6509c44f) [cherry-pick] Update repository config (#683) (#684)
- [31626977](https://github.com/stashed/postgres/commit/31626977) Add auto-backup doc + Restructure docs (#618) (#673)
- [2602a2e5](https://github.com/stashed/postgres/commit/2602a2e5) [cherry-pick] Don't overwrite superuser password of restored database (#660) (#663)
- [3a5c3f93](https://github.com/stashed/postgres/commit/3a5c3f93) Update repository config (#649) (#650)
- [4638354a](https://github.com/stashed/postgres/commit/4638354a) Use restic 0.12.0 (#637) (#638)
- [b81480ec](https://github.com/stashed/postgres/commit/b81480ec) Update Kubernetes v1.18.9 dependencies (#626) (#627)
- [7f7c4eb8](https://github.com/stashed/postgres/commit/7f7c4eb8) [cherry-pick] Check codespan schema (#617) (#619)
- [cab4cb38](https://github.com/stashed/postgres/commit/cab4cb38) [cherry-pick] Update repository config (#600) (#601)
- [fd9fa30f](https://github.com/stashed/postgres/commit/fd9fa30f) [cherry-pick] Update repository config (#589) (#590)
- [29f17df8](https://github.com/stashed/postgres/commit/29f17df8) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#578) (#579)
- [cb6be80f](https://github.com/stashed/postgres/commit/cb6be80f) [cherry-pick] Update repository config (#566) (#567)
- [3033a772](https://github.com/stashed/postgres/commit/3033a772) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#555) (#556)
- [bf03d98d](https://github.com/stashed/postgres/commit/bf03d98d) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#544) (#545)
- [b2fdc143](https://github.com/stashed/postgres/commit/b2fdc143) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#534) (#535)
- [c898ee57](https://github.com/stashed/postgres/commit/c898ee57) [cherry-pick] Update README.md template (#523) (#524)
- [fadedccd](https://github.com/stashed/postgres/commit/fadedccd) [cherry-pick] Generate README.md using templates (#512) (#513)
- [46e2b6dd](https://github.com/stashed/postgres/commit/46e2b6dd) [cherry-pick] Speed up schema generation process (#501) (#502)


### [11.9.0-v5](https://github.com/stashed/postgres/releases/tag/11.9.0-v5)

- [2a7a4f96](https://github.com/stashed/postgres/commit/2a7a4f96) Prepare for release 11.9.0-v5 (#717)
- [e673d3c0](https://github.com/stashed/postgres/commit/e673d3c0) Fix Makefile (#705) (#711)
- [c8e9b998](https://github.com/stashed/postgres/commit/c8e9b998) TLS support for postgres (#694) (#700)
- [ebc0a1f9](https://github.com/stashed/postgres/commit/ebc0a1f9) [cherry-pick] Update repository config (#683) (#689)
- [baa94cf6](https://github.com/stashed/postgres/commit/baa94cf6) Add auto-backup doc + Restructure docs (#618) (#678)
- [06d62688](https://github.com/stashed/postgres/commit/06d62688) [cherry-pick] Don't overwrite superuser password of restored database (#660) (#668)
- [7fa729aa](https://github.com/stashed/postgres/commit/7fa729aa) Update repository config (#649) (#655)
- [637daf85](https://github.com/stashed/postgres/commit/637daf85) Use restic 0.12.0 (#637) (#643)
- [2ec2a638](https://github.com/stashed/postgres/commit/2ec2a638) Update Kubernetes v1.18.9 dependencies (#626) (#632)
- [db0433ed](https://github.com/stashed/postgres/commit/db0433ed) [cherry-pick] Check codespan schema (#617) (#624)
- [bb3d5515](https://github.com/stashed/postgres/commit/bb3d5515) [cherry-pick] Update repository config (#600) (#606)
- [c5b40a97](https://github.com/stashed/postgres/commit/c5b40a97) [cherry-pick] Update repository config (#589) (#595)
- [8ad35d50](https://github.com/stashed/postgres/commit/8ad35d50) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#578) (#584)
- [b5230e5f](https://github.com/stashed/postgres/commit/b5230e5f) [cherry-pick] Update repository config (#566) (#572)
- [593a9b7e](https://github.com/stashed/postgres/commit/593a9b7e) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#555) (#561)
- [ce7aa9d1](https://github.com/stashed/postgres/commit/ce7aa9d1) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#544) (#550)
- [ce8bdb41](https://github.com/stashed/postgres/commit/ce8bdb41) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#534) (#540)
- [da4c7f5b](https://github.com/stashed/postgres/commit/da4c7f5b) [cherry-pick] Update README.md template (#523) (#529)
- [d928f7ac](https://github.com/stashed/postgres/commit/d928f7ac) [cherry-pick] Generate README.md using templates (#512) (#518)
- [8bdef7b5](https://github.com/stashed/postgres/commit/8bdef7b5) [cherry-pick] Speed up schema generation process (#501) (#507)


### [12.4.0-v5](https://github.com/stashed/postgres/releases/tag/12.4.0-v5)

- [b1140414](https://github.com/stashed/postgres/commit/b1140414) Prepare for release 12.4.0-v5 (#718)
- [403dfb51](https://github.com/stashed/postgres/commit/403dfb51) Fix Makefile (#705) (#712)
- [0e98e14e](https://github.com/stashed/postgres/commit/0e98e14e) TLS support for postgres (#694) (#701)
- [dfc98b6e](https://github.com/stashed/postgres/commit/dfc98b6e) [cherry-pick] Update repository config (#683) (#690)
- [3a8c1036](https://github.com/stashed/postgres/commit/3a8c1036) Add auto-backup doc + Restructure docs (#618) (#679)
- [a14c26d8](https://github.com/stashed/postgres/commit/a14c26d8) [cherry-pick] Don't overwrite superuser password of restored database (#660) (#669)
- [fc9f2302](https://github.com/stashed/postgres/commit/fc9f2302) Update repository config (#649) (#656)
- [ee83ac04](https://github.com/stashed/postgres/commit/ee83ac04) Use restic 0.12.0 (#637) (#644)
- [9b27b063](https://github.com/stashed/postgres/commit/9b27b063) Update Kubernetes v1.18.9 dependencies (#626) (#633)
- [32d5df70](https://github.com/stashed/postgres/commit/32d5df70) [cherry-pick] Check codespan schema (#617) (#625)
- [54bba3b1](https://github.com/stashed/postgres/commit/54bba3b1) [cherry-pick] Update repository config (#600) (#607)
- [2ac504d2](https://github.com/stashed/postgres/commit/2ac504d2) [cherry-pick] Update repository config (#589) (#596)
- [0b4f7bea](https://github.com/stashed/postgres/commit/0b4f7bea) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#578) (#585)
- [e3e822f3](https://github.com/stashed/postgres/commit/e3e822f3) [cherry-pick] Update repository config (#566) (#573)
- [956653e3](https://github.com/stashed/postgres/commit/956653e3) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#555) (#562)
- [ef5cc730](https://github.com/stashed/postgres/commit/ef5cc730) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#544) (#551)
- [02a8fc1b](https://github.com/stashed/postgres/commit/02a8fc1b) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#534) (#541)
- [8c4e765f](https://github.com/stashed/postgres/commit/8c4e765f) [cherry-pick] Update README.md template (#523) (#530)
- [66da0577](https://github.com/stashed/postgres/commit/66da0577) [cherry-pick] Generate README.md using templates (#512) (#519)
- [a5be3888](https://github.com/stashed/postgres/commit/a5be3888) [cherry-pick] Speed up schema generation process (#501) (#508)


### [13.1.0-v2](https://github.com/stashed/postgres/releases/tag/13.1.0-v2)

- [6c59e1eb](https://github.com/stashed/postgres/commit/6c59e1eb) Prepare for release 13.1.0-v2 (#719)
- [be6c043a](https://github.com/stashed/postgres/commit/be6c043a) Fix Makefile (#705) (#713)
- [8805ee06](https://github.com/stashed/postgres/commit/8805ee06) TLS support for postgres (#694) (#702)
- [517a4e20](https://github.com/stashed/postgres/commit/517a4e20) [cherry-pick] Update repository config (#683) (#691)
- [fc935ae7](https://github.com/stashed/postgres/commit/fc935ae7) Add auto-backup doc + Restructure docs (#618) (#680)
- [ac84e6d3](https://github.com/stashed/postgres/commit/ac84e6d3) [cherry-pick] Don't overwrite superuser password of restored database (#660) (#670)
- [6c195dfb](https://github.com/stashed/postgres/commit/6c195dfb) Update repository config (#649) (#657)
- [ecd22af1](https://github.com/stashed/postgres/commit/ecd22af1) Use restic 0.12.0 (#637) (#645)
- [3a1fe354](https://github.com/stashed/postgres/commit/3a1fe354) Update Kubernetes v1.18.9 dependencies (#626) (#634)
- [c82407f8](https://github.com/stashed/postgres/commit/c82407f8) [cherry-pick] Update repository config (#600) (#608)
- [6432d062](https://github.com/stashed/postgres/commit/6432d062) [cherry-pick] Update repository config (#589) (#597)
- [2fe1bdf3](https://github.com/stashed/postgres/commit/2fe1bdf3) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#578) (#586)
- [719fb462](https://github.com/stashed/postgres/commit/719fb462) [cherry-pick] Update repository config (#566) (#574)
- [45871d26](https://github.com/stashed/postgres/commit/45871d26) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#555) (#563)
- [ecd689c8](https://github.com/stashed/postgres/commit/ecd689c8) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#544) (#552)
- [ad7687a2](https://github.com/stashed/postgres/commit/ad7687a2) [cherry-pick] Update Kubernetes v1.18.9 dependencies (#534) (#542)
- [53fdf211](https://github.com/stashed/postgres/commit/53fdf211) [cherry-pick] Update README.md template (#523) (#531)
- [5c5e670c](https://github.com/stashed/postgres/commit/5c5e670c) [cherry-pick] Generate README.md using templates (#512) (#520)
- [a60735d0](https://github.com/stashed/postgres/commit/a60735d0) [cherry-pick] Speed up schema generation process (#501) (#509)



## [stashed/stash](https://github.com/stashed/stash)

### [v0.11.10](https://github.com/stashed/stash/releases/tag/v0.11.10)

- [cd515d79](https://github.com/stashed/stash/commit/cd515d79) Prepare for release v0.11.10 (#1327)
- [34f6576e](https://github.com/stashed/stash/commit/34f6576e) Delete ClusterRoleBinding when backup/restore invoker is removed (#1326)
- [23c25326](https://github.com/stashed/stash/commit/23c25326) Use addon info from AppBinding for KubeDB managed databases (#1325)
- [99d6e168](https://github.com/stashed/stash/commit/99d6e168) Update repository config (#1323)
- [14a940fb](https://github.com/stashed/stash/commit/14a940fb) Update repository config (#1321)
- [0338a91d](https://github.com/stashed/stash/commit/0338a91d) Update Kubernetes v1.18.9 dependencies (#1319)
- [b9063896](https://github.com/stashed/stash/commit/b9063896) Enable running as a kubedb extension
- [79240c53](https://github.com/stashed/stash/commit/79240c53) Use restic 0.12.0 (#1318)
- [7e0e2105](https://github.com/stashed/stash/commit/7e0e2105) Update Kubernetes v1.18.9 dependencies (#1317)
- [757af516](https://github.com/stashed/stash/commit/757af516) Update Kubernetes v1.18.9 dependencies (#1314)
- [f955eff8](https://github.com/stashed/stash/commit/f955eff8) Update repository config (#1305)
- [e583607f](https://github.com/stashed/stash/commit/e583607f) Update repository config (#1304)
- [ba70668b](https://github.com/stashed/stash/commit/ba70668b) Update Kubernetes v1.18.9 dependencies (#1301)




