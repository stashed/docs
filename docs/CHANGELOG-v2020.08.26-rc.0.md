---
title: Changelog | Stash
description: Changelog
menu:
  docs_{{.version}}:
    identifier: changelog-stash-v2020.08.26-rc.0
    name: Changelog-v2020.08.26-rc.0
    parent: welcome
    weight: 20200826
product_name: stash
menu_name: docs_{{.version}}
section_menu_id: welcome
url: /docs/{{.version}}/welcome/changelog-v2020.08.26-rc.0/
aliases:
  - /docs/{{.version}}/CHANGELOG-v2020.08.26-rc.0/
---

# Stash v2020.08.26-rc.0 (2020-08-26)


## [stashed/apimachinery](https://github.com/stashed/apimachinery)

### [v0.10.0-rc.0](https://github.com/stashed/apimachinery/releases/tag/v0.10.0-rc.0)

- [254ee64e](https://github.com/stashed/apimachinery/commit/254ee64e) Rename param annotation (#42)
- [2210ed42](https://github.com/stashed/apimachinery/commit/2210ed42) Update Kubernetes v1.18.3 dependencies (#41)
- [835c0358](https://github.com/stashed/apimachinery/commit/835c0358) Update Kubernetes v1.18.3 dependencies (#40)
- [692909fe](https://github.com/stashed/apimachinery/commit/692909fe) Add params and schedule annotation (#38)
- [b9ff4f14](https://github.com/stashed/apimachinery/commit/b9ff4f14) Update Kubernetes v1.18.3 dependencies (#37)
- [c44c52c8](https://github.com/stashed/apimachinery/commit/c44c52c8) Update Kubernetes v1.18.3 dependencies (#36)
- [073a93c8](https://github.com/stashed/apimachinery/commit/073a93c8) Rename StashLocalBackendAccessor to StashNetVolAccessor (#35)
- [e29c2724](https://github.com/stashed/apimachinery/commit/e29c2724) Update Kubernetes v1.18.3 dependencies (#34)
- [a4807149](https://github.com/stashed/apimachinery/commit/a4807149) Update Kubernetes v1.18.3 dependencies (#33)
- [d6fcb2c6](https://github.com/stashed/apimachinery/commit/d6fcb2c6) Fix protobuf tags (#32)
- [d255d800](https://github.com/stashed/apimachinery/commit/d255d800) Pass target reference for restore job condition (#31)
- [a3ce1cee](https://github.com/stashed/apimachinery/commit/a3ce1cee) Add helper method for NFS backend (#30)
- [a4be15a1](https://github.com/stashed/apimachinery/commit/a4be15a1) Introduce RestoreBatch CRD + Additional Improvements (#22)
- [710228f4](https://github.com/stashed/apimachinery/commit/710228f4) Update to Kubernetes v1.18.3 (#29)
- [0f69a3ab](https://github.com/stashed/apimachinery/commit/0f69a3ab) Update to Kubernetes v1.18.3 (#28)
- [504a57e7](https://github.com/stashed/apimachinery/commit/504a57e7) Update to Kubernetes v1.18.3 (#27)
- [843421e4](https://github.com/stashed/apimachinery/commit/843421e4) Show AppsCode in copyright file header (#26)
- [ad462cc4](https://github.com/stashed/apimachinery/commit/ad462cc4) Update to Kubernetes v1.18.3 (#25)
- [28d53699](https://github.com/stashed/apimachinery/commit/28d53699) Update update-release-tracker.sh
- [34624044](https://github.com/stashed/apimachinery/commit/34624044) Update update-release-tracker.sh
- [5f5de63c](https://github.com/stashed/apimachinery/commit/5f5de63c) Fix openapi path (#24)
- [6e0ad5f8](https://github.com/stashed/apimachinery/commit/6e0ad5f8) Add script to update release tracker on pr merge (#23)
- [cbf9b376](https://github.com/stashed/apimachinery/commit/cbf9b376) Update .kodiak.toml
- [d12b3d4b](https://github.com/stashed/apimachinery/commit/d12b3d4b) Update to Kubernetes v1.18.3 (#21)
- [1956a312](https://github.com/stashed/apimachinery/commit/1956a312) Update to Kubernetes v1.18.3
- [c3966002](https://github.com/stashed/apimachinery/commit/c3966002) Unwrap top level api folder (#20)
- [5ba03fb5](https://github.com/stashed/apimachinery/commit/5ba03fb5) Update to Kubernetes v1.18.3 (#19)
- [abeb620e](https://github.com/stashed/apimachinery/commit/abeb620e) Update to Kubernetes v1.18.3
- [6fdf8a60](https://github.com/stashed/apimachinery/commit/6fdf8a60) Enable https://kodiakhq.com (#13)
- [479258ed](https://github.com/stashed/apimachinery/commit/479258ed) Update dev scripts (#12)
- [a85ced99](https://github.com/stashed/apimachinery/commit/a85ced99) Merge pull request #11 from stashed/k8s-gomod-refresher-1591208266
- [82df6f26](https://github.com/stashed/apimachinery/commit/82df6f26) Update to Kubernetes v1.18.3
- [788f6921](https://github.com/stashed/apimachinery/commit/788f6921) Add default annotation for Snapshotter (#9)
- [d2f3f5d4](https://github.com/stashed/apimachinery/commit/d2f3f5d4) Remove defaults from crd v1beta1 YAML (#8)
- [1a09ffde](https://github.com/stashed/apimachinery/commit/1a09ffde) Update dependencies
- [58525b4b](https://github.com/stashed/apimachinery/commit/58525b4b) Update dependencies
- [c34c2ec1](https://github.com/stashed/apimachinery/commit/c34c2ec1) Generate both v1beta1 and v1 CRD YAML (#7)
- [e81205a3](https://github.com/stashed/apimachinery/commit/e81205a3) Bring back mistakenly removed SetRecoveryStats
- [5f8cf3a6](https://github.com/stashed/apimachinery/commit/5f8cf3a6) Merge pull request #6 from stashed/k-1.18.3
- [723f4de9](https://github.com/stashed/apimachinery/commit/723f4de9) Add context to crd utils
- [59478af4](https://github.com/stashed/apimachinery/commit/59478af4) Update to Kubernetes 1.18.3
- [e83b90a7](https://github.com/stashed/apimachinery/commit/e83b90a7) Merge pull request #3 from stashed/wait-for-target
- [a5b9a011](https://github.com/stashed/apimachinery/commit/a5b9a011) Simplify targetMatched() function
- [58948bd9](https://github.com/stashed/apimachinery/commit/58948bd9) Refactor
- [5568cb90](https://github.com/stashed/apimachinery/commit/5568cb90) Add RestoreSession conditions
- [906c5910](https://github.com/stashed/apimachinery/commit/906c5910) Add TypeMeta to invoker
- [22843fdb](https://github.com/stashed/apimachinery/commit/22843fdb) Use Go 1.14.3
- [238d1bd0](https://github.com/stashed/apimachinery/commit/238d1bd0) Add backup invoker condition transion reasons
- [275965f9](https://github.com/stashed/apimachinery/commit/275965f9) Introduce conditions for BackupConfiguration and BackupBatch
- [35159c81](https://github.com/stashed/apimachinery/commit/35159c81) Merge pull request #5 from stashed/fix-updatestatus
- [f1d78326](https://github.com/stashed/apimachinery/commit/f1d78326) Fix helper methods
- [dbb02873](https://github.com/stashed/apimachinery/commit/dbb02873) Fix UpdateStatus() function
- [a7bd75ad](https://github.com/stashed/apimachinery/commit/a7bd75ad) Update crazy-max/ghaction-docker-buildx flag
- [1d65a7d4](https://github.com/stashed/apimachinery/commit/1d65a7d4) Use recommended kubernetes app labels (#4)
- [5b322e9f](https://github.com/stashed/apimachinery/commit/5b322e9f) Add Enum markers to api types
- [e6017151](https://github.com/stashed/apimachinery/commit/e6017151) Trigger the workflow on push or pull request
- [54097441](https://github.com/stashed/apimachinery/commit/54097441) Use kubectl v1.17 (#1)



## [stashed/catalog](https://github.com/stashed/catalog)

### [v2020.08.26-rc.0](https://github.com/stashed/catalog/releases/tag/v2020.08.26-rc.0)




## [stashed/cli](https://github.com/stashed/cli)

### [v0.10.0-rc.0](https://github.com/stashed/cli/releases/tag/v0.10.0-rc.0)

- [fdfe6b8](https://github.com/stashed/cli/commit/fdfe6b8) Prepare for release v0.10.0-rc.0 (#38)
- [132ae15](https://github.com/stashed/cli/commit/132ae15) Fix build (#37)
- [77a76a7](https://github.com/stashed/cli/commit/77a76a7) Update Kubernetes v1.18.3 dependencies (#36)
- [efe0fc5](https://github.com/stashed/cli/commit/efe0fc5) Update Kubernetes v1.18.3 dependencies (#35)
- [d39bd97](https://github.com/stashed/cli/commit/d39bd97) Update Kubernetes v1.18.3 dependencies (#34)
- [3e074e7](https://github.com/stashed/cli/commit/3e074e7) Update Kubernetes v1.18.3 dependencies (#33)
- [f020137](https://github.com/stashed/cli/commit/f020137) Update Kubernetes v1.18.3 dependencies (#32)
- [99d27f0](https://github.com/stashed/cli/commit/99d27f0) Update Kubernetes v1.18.3 dependencies (#31)
- [3c78ec5](https://github.com/stashed/cli/commit/3c78ec5) Use actions/upload-artifact@v2
- [e416569](https://github.com/stashed/cli/commit/e416569) Update to Kubernetes v1.18.3 (#30)
- [2baaae1](https://github.com/stashed/cli/commit/2baaae1) Update to Kubernetes v1.18.3 (#29)
- [44a1514](https://github.com/stashed/cli/commit/44a1514) Update to Kubernetes v1.18.3 (#28)
- [250372c](https://github.com/stashed/cli/commit/250372c) Prepare for release v0.10.0-beta.1 (#27)
- [3728110](https://github.com/stashed/cli/commit/3728110) Prepare for release v0.10.0-beta.0 (#26)
- [e7111a5](https://github.com/stashed/cli/commit/e7111a5) Update License
- [7fe1e07](https://github.com/stashed/cli/commit/7fe1e07) Update to Kubernetes v1.18.3 (#25)
- [66b3b46](https://github.com/stashed/cli/commit/66b3b46) Shorten command name for cli (#24)
- [b913cfc](https://github.com/stashed/cli/commit/b913cfc) Add workflow to update docs (#23)
- [1881c64](https://github.com/stashed/cli/commit/1881c64) Update update-release-tracker.sh
- [0548bdc](https://github.com/stashed/cli/commit/0548bdc) Update update-release-tracker.sh
- [b1b28ff](https://github.com/stashed/cli/commit/b1b28ff) Use GITHUB_BASE_REF to detect target branch
- [1e16b99](https://github.com/stashed/cli/commit/1e16b99) Add script to update release tracker on pr merge (#21)
- [f91bf33](https://github.com/stashed/cli/commit/f91bf33) Make release non-draft
- [d29bdb6](https://github.com/stashed/cli/commit/d29bdb6) Update .kodiak.toml
- [b727108](https://github.com/stashed/cli/commit/b727108) Update to Kubernetes v1.18.3 (#20)
- [f3f03aa](https://github.com/stashed/cli/commit/f3f03aa) Update to Kubernetes v1.18.3
- [bcd7c5e](https://github.com/stashed/cli/commit/bcd7c5e) Create .kodiak.toml
- [9882aa2](https://github.com/stashed/cli/commit/9882aa2) Add blank line after license header (#19)
- [7774218](https://github.com/stashed/cli/commit/7774218) Update dev scripts (#18)
- [38eb35c](https://github.com/stashed/cli/commit/38eb35c) Run unit tests against SRC_PKGS
- [526949c](https://github.com/stashed/cli/commit/526949c) Update to Kubernetes v1.18.3 (#17)
- [fc3e6c5](https://github.com/stashed/cli/commit/fc3e6c5) Update crazy-max/ghaction-docker-buildx flag
- [3943575](https://github.com/stashed/cli/commit/3943575) Trigger the workflow on push or pull request



## [stashed/elasticsearch](https://github.com/stashed/elasticsearch)

### [5.6.4-beta.20200826](https://github.com/stashed/elasticsearch/releases/tag/5.6.4-beta.20200826)

- [df82b88](https://github.com/stashed/elasticsearch/commit/df82b88) Prepare for release 5.6.4-beta.20200826 (#158)
- [465a9f0](https://github.com/stashed/elasticsearch/commit/465a9f0) [cherry-pick] Update Stash installation link (#149) (#150)
- [4fd2af7](https://github.com/stashed/elasticsearch/commit/4fd2af7) [cherry-pick] Make image.tag in values.yaml file same as the $APP_VERSION (#132) (#141)
- [f9f6ab1](https://github.com/stashed/elasticsearch/commit/f9f6ab1) [cherry-pick] Fix output format + Add PreBackupActions logic (#131) (#133)
- [cf70698](https://github.com/stashed/elasticsearch/commit/cf70698) [cherry-pick] Update chart icon (#123)
- [f1c9257](https://github.com/stashed/elasticsearch/commit/f1c9257) [cherry-pick] Make chart registry configurable (#114) (#115)


### [6.2.4-beta.20200826](https://github.com/stashed/elasticsearch/releases/tag/6.2.4-beta.20200826)

- [6c3dfa5](https://github.com/stashed/elasticsearch/commit/6c3dfa5) Prepare for release 6.2.4-beta.20200826 (#159)
- [4aa0746](https://github.com/stashed/elasticsearch/commit/4aa0746) [cherry-pick] Update Stash installation link (#149) (#151)
- [9847b08](https://github.com/stashed/elasticsearch/commit/9847b08) [cherry-pick] Make image.tag in values.yaml file same as the $APP_VERSION (#132) (#142)
- [e13632d](https://github.com/stashed/elasticsearch/commit/e13632d) [cherry-pick] Fix output format + Add PreBackupActions logic (#131) (#134)
- [94dc85b](https://github.com/stashed/elasticsearch/commit/94dc85b) [cherry-pick] Update chart icon (#124)
- [efc6ad0](https://github.com/stashed/elasticsearch/commit/efc6ad0) [cherry-pick] Make chart registry configurable (#114) (#116)


### [6.3.0-beta.20200826](https://github.com/stashed/elasticsearch/releases/tag/6.3.0-beta.20200826)

- [4a32c49](https://github.com/stashed/elasticsearch/commit/4a32c49) Prepare for release 6.3.0-beta.20200826 (#160)
- [4bc4449](https://github.com/stashed/elasticsearch/commit/4bc4449) [cherry-pick] Update Stash installation link (#149) (#152)
- [31d3860](https://github.com/stashed/elasticsearch/commit/31d3860) [cherry-pick] Make image.tag in values.yaml file same as the $APP_VERSION (#132) (#143)
- [c19466a](https://github.com/stashed/elasticsearch/commit/c19466a) [cherry-pick] Fix output format + Add PreBackupActions logic (#131) (#135)
- [1aaa172](https://github.com/stashed/elasticsearch/commit/1aaa172) [cherry-pick] Update chart icon (#125)
- [30412a8](https://github.com/stashed/elasticsearch/commit/30412a8) [cherry-pick] Make chart registry configurable (#114) (#117)


### [6.4.0-beta.20200826](https://github.com/stashed/elasticsearch/releases/tag/6.4.0-beta.20200826)

- [0770775](https://github.com/stashed/elasticsearch/commit/0770775) Prepare for release 6.4.0-beta.20200826 (#161)
- [38be1db](https://github.com/stashed/elasticsearch/commit/38be1db) [cherry-pick] Update Stash installation link (#149) (#153)
- [87ca905](https://github.com/stashed/elasticsearch/commit/87ca905) [cherry-pick] Make image.tag in values.yaml file same as the $APP_VERSION (#132) (#144)
- [75dd0a5](https://github.com/stashed/elasticsearch/commit/75dd0a5) [cherry-pick] Fix output format + Add PreBackupActions logic (#131) (#136)
- [ad75b67](https://github.com/stashed/elasticsearch/commit/ad75b67) [cherry-pick] Update chart icon (#126)
- [e5e9c7b](https://github.com/stashed/elasticsearch/commit/e5e9c7b) [cherry-pick] Make chart registry configurable (#114) (#118)


### [6.5.3-beta.20200826](https://github.com/stashed/elasticsearch/releases/tag/6.5.3-beta.20200826)

- [58af00f](https://github.com/stashed/elasticsearch/commit/58af00f) Prepare for release 6.5.3-beta.20200826 (#162)
- [1365067](https://github.com/stashed/elasticsearch/commit/1365067) [cherry-pick] Update Stash installation link (#149) (#154)
- [16f9593](https://github.com/stashed/elasticsearch/commit/16f9593) [cherry-pick] Make image.tag in values.yaml file same as the $APP_VERSION (#132) (#145)
- [8413eba](https://github.com/stashed/elasticsearch/commit/8413eba) [cherry-pick] Fix output format + Add PreBackupActions logic (#131) (#137)
- [b0237a3](https://github.com/stashed/elasticsearch/commit/b0237a3) [cherry-pick] Update chart icon (#127)
- [daec12d](https://github.com/stashed/elasticsearch/commit/daec12d) [cherry-pick] Make chart registry configurable (#114) (#119)


### [6.8.0-beta.20200826](https://github.com/stashed/elasticsearch/releases/tag/6.8.0-beta.20200826)

- [8811915](https://github.com/stashed/elasticsearch/commit/8811915) Prepare for release 6.8.0-beta.20200826 (#163)
- [46e0b2a](https://github.com/stashed/elasticsearch/commit/46e0b2a) [cherry-pick] Update Stash installation link (#149) (#155)
- [9a00bba](https://github.com/stashed/elasticsearch/commit/9a00bba) [cherry-pick] Make image.tag in values.yaml file same as the $APP_VERSION (#132) (#146)
- [f0ae71f](https://github.com/stashed/elasticsearch/commit/f0ae71f) [cherry-pick] Fix output format + Add PreBackupActions logic (#131) (#138)
- [9561dd8](https://github.com/stashed/elasticsearch/commit/9561dd8) [cherry-pick] Update chart icon (#128)
- [bc40c89](https://github.com/stashed/elasticsearch/commit/bc40c89) [cherry-pick] Make chart registry configurable (#114) (#120)


### [7.2.0-beta.20200826](https://github.com/stashed/elasticsearch/releases/tag/7.2.0-beta.20200826)

- [bd2c6bc](https://github.com/stashed/elasticsearch/commit/bd2c6bc) Prepare for release 7.2.0-beta.20200826 (#164)
- [2c3a9e1](https://github.com/stashed/elasticsearch/commit/2c3a9e1) [cherry-pick] Update Stash installation link (#149) (#156)
- [c862e31](https://github.com/stashed/elasticsearch/commit/c862e31) [cherry-pick] Make image.tag in values.yaml file same as the $APP_VERSION (#132) (#147)
- [1010749](https://github.com/stashed/elasticsearch/commit/1010749) [cherry-pick] Fix output format + Add PreBackupActions logic (#131) (#139)
- [b0cac90](https://github.com/stashed/elasticsearch/commit/b0cac90) [cherry-pick] Update chart icon (#129)
- [abd52d3](https://github.com/stashed/elasticsearch/commit/abd52d3) [cherry-pick] Make chart registry configurable (#114) (#121)


### [7.3.2-beta.20200826](https://github.com/stashed/elasticsearch/releases/tag/7.3.2-beta.20200826)

- [a163678](https://github.com/stashed/elasticsearch/commit/a163678) Prepare for release 7.3.2-beta.20200826 (#165)
- [152792e](https://github.com/stashed/elasticsearch/commit/152792e) [cherry-pick] Update Stash installation link (#149) (#157)
- [2d0f427](https://github.com/stashed/elasticsearch/commit/2d0f427) [cherry-pick] Make image.tag in values.yaml file same as the $APP_VERSION (#132) (#148)
- [281a0ce](https://github.com/stashed/elasticsearch/commit/281a0ce) [cherry-pick] Fix output format + Add PreBackupActions logic (#131) (#140)
- [e89cbde](https://github.com/stashed/elasticsearch/commit/e89cbde) [cherry-pick] Update chart icon (#130)
- [a4c3327](https://github.com/stashed/elasticsearch/commit/a4c3327) [cherry-pick] Make chart registry configurable (#114) (#122)



## [stashed/installer](https://github.com/stashed/installer)

### [v0.10.0-rc.0](https://github.com/stashed/installer/releases/tag/v0.10.0-rc.0)

- [6018270](https://github.com/stashed/installer/commit/6018270) Prepare for release v0.10.0-rc.0 (#87)
- [9da1502](https://github.com/stashed/installer/commit/9da1502) Port changes from enterprise version (#85)
- [0838112](https://github.com/stashed/installer/commit/0838112) Add offline license verification (#84)
- [d869f33](https://github.com/stashed/installer/commit/d869f33) Always give use permission for baseline psp to operator (#83)
- [d1e0142](https://github.com/stashed/installer/commit/d1e0142) Support Snapshot listing for NFS backend without workload running (#80)
- [53bab27](https://github.com/stashed/installer/commit/53bab27) Pass imagePullSecrets as operator flag (#71)
- [4e3a984](https://github.com/stashed/installer/commit/4e3a984) Update to Kubernetes v1.18.3 (#79)
- [6e57dd4](https://github.com/stashed/installer/commit/6e57dd4) Update to Kubernetes v1.18.3 (#78)
- [5c681fe](https://github.com/stashed/installer/commit/5c681fe) Update to Kubernetes v1.18.3 (#77)
- [c268a57](https://github.com/stashed/installer/commit/c268a57) Make chart registry configurable
- [bed8319](https://github.com/stashed/installer/commit/bed8319) Prepare for release v0.10.0-beta.1 (#76)
- [cd44ba9](https://github.com/stashed/installer/commit/cd44ba9) Prepare for release v0.10.0-beta.0 (#75)
- [a04e173](https://github.com/stashed/installer/commit/a04e173) Publish to testing dir for alpha/beta releases
- [4b23e1c](https://github.com/stashed/installer/commit/4b23e1c) Update License (#74)
- [9b7a4e0](https://github.com/stashed/installer/commit/9b7a4e0) Update to Kubernetes v1.18.3 (#72)
- [4318306](https://github.com/stashed/installer/commit/4318306) Update ci.yml
- [15d1594](https://github.com/stashed/installer/commit/15d1594) Fix Stash Enterprise installer (#70)
- [31c9dcc](https://github.com/stashed/installer/commit/31c9dcc) Tag chart and app version as string for yq (#69)
- [1782049](https://github.com/stashed/installer/commit/1782049) Update links (#68)
- [634da4d](https://github.com/stashed/installer/commit/634da4d) Update update-release-tracker.sh
- [1155610](https://github.com/stashed/installer/commit/1155610) Update update-release-tracker.sh
- [1b10b5e](https://github.com/stashed/installer/commit/1b10b5e) Add script to update release tracker on pr merge (#67)
- [ce0b28e](https://github.com/stashed/installer/commit/ce0b28e) Update release workflow
- [c3ac668](https://github.com/stashed/installer/commit/c3ac668) Update ci.yml
- [98bad7e](https://github.com/stashed/installer/commit/98bad7e) Add Stash Enterprise chart (#63)
- [73f52a6](https://github.com/stashed/installer/commit/73f52a6) Add commands to update chart (#65)
- [0dc7f91](https://github.com/stashed/installer/commit/0dc7f91) Fix chart release process (#64)
- [0d5c4e1](https://github.com/stashed/installer/commit/0d5c4e1) Update .kodiak.toml
- [3b53e64](https://github.com/stashed/installer/commit/3b53e64) Update to Kubernetes v1.18.3 (#58)
- [43c5dbe](https://github.com/stashed/installer/commit/43c5dbe) Update to Kubernetes v1.18.3
- [b9e784c](https://github.com/stashed/installer/commit/b9e784c) Create .kodiak.toml
- [b30b3b0](https://github.com/stashed/installer/commit/b30b3b0) Merge pull request #57 from stashed/psp
- [1b89401](https://github.com/stashed/installer/commit/1b89401) Disable apparmor and seccomp by default
- [6bed1aa](https://github.com/stashed/installer/commit/6bed1aa) Pass psp names for the jobs through flag
- [bd35d81](https://github.com/stashed/installer/commit/bd35d81) Always use baseline psp for stash
- [4e3474a](https://github.com/stashed/installer/commit/4e3474a) Add RBAC permission for generic-garbage-collector (#56)
- [be006f6](https://github.com/stashed/installer/commit/be006f6) Permit configmap list/watch -ing for delegated authentication checking (#55)
- [5685c15](https://github.com/stashed/installer/commit/5685c15) Update dependencies
- [8b7b805](https://github.com/stashed/installer/commit/8b7b805) Update dependencies
- [d2b2b09](https://github.com/stashed/installer/commit/d2b2b09) Generate both v1beta1 and v1 CRD YAML (#54)
- [7fbcb29](https://github.com/stashed/installer/commit/7fbcb29) Update to Kubernetes v1.18.3 (#53)
- [88e5e8c](https://github.com/stashed/installer/commit/88e5e8c) Use Go 1.14.3
- [8e56cb1](https://github.com/stashed/installer/commit/8e56cb1) Trigger build on push to only master branch
- [562caf8](https://github.com/stashed/installer/commit/562caf8) Use recommended kubernetes app labels (#52)
- [cc55e5a](https://github.com/stashed/installer/commit/cc55e5a) Trigger the workflow on push or pull request
- [fd8acf5](https://github.com/stashed/installer/commit/fd8acf5) Update chart readme
- [672f37e](https://github.com/stashed/installer/commit/672f37e) Show examples in chart readme
- [39f4ca1](https://github.com/stashed/installer/commit/39f4ca1) Auto generate chart readme file (#50)
- [47f4250](https://github.com/stashed/installer/commit/47f4250) Update release.yml
- [b68d9cb](https://github.com/stashed/installer/commit/b68d9cb) Cleanup newlines
- [20d51b0](https://github.com/stashed/installer/commit/20d51b0) Reformat stash chart template (#49)
- [65f8bee](https://github.com/stashed/installer/commit/65f8bee) Use kubectl v1.16 as cleaner (#48)
- [85a7cfd](https://github.com/stashed/installer/commit/85a7cfd) Rename prometheus.io/coreos-operator to prometheus.io/operator (#47)
- [b042def](https://github.com/stashed/installer/commit/b042def) Move apireg annotation to operator pod (#46)
- [a543953](https://github.com/stashed/installer/commit/a543953) Various cleanup (#44)
- [b6e2bec](https://github.com/stashed/installer/commit/b6e2bec) Fix helm install --wait flag (#42)
- [806aada](https://github.com/stashed/installer/commit/806aada) Do not harcode namespace (#40)



## [stashed/mongodb](https://github.com/stashed/mongodb)

### [3.4.1-beta.20200826](https://github.com/stashed/mongodb/releases/tag/3.4.1-beta.20200826)

- [1c08dfd](https://github.com/stashed/mongodb/commit/1c08dfd) Prepare for release 3.4.1-beta.20200826 (#185)
- [a4d2b49](https://github.com/stashed/mongodb/commit/a4d2b49) [cherry-pick] Update Stash installation link (#173) (#174)
- [3ae949e](https://github.com/stashed/mongodb/commit/3ae949e) [cherry-pick] Fix output format + Add PreBackupActions logic (#149) (#162)
- [75ebc36](https://github.com/stashed/mongodb/commit/75ebc36) [cherry-pick] Make image.tag in values.yaml same as $APP_VERSION (#150) (#151)
- [c261899](https://github.com/stashed/mongodb/commit/c261899) [cherry-pick] Update chart icon (#138)
- [465f36c](https://github.com/stashed/mongodb/commit/465f36c) [cherry-pick] Make chart registry configurable (#126) (#127)


### [3.4.2-beta.20200826](https://github.com/stashed/mongodb/releases/tag/3.4.2-beta.20200826)

- [b9d6d4d](https://github.com/stashed/mongodb/commit/b9d6d4d) Prepare for release 3.4.2-beta.20200826 (#186)
- [adf04e8](https://github.com/stashed/mongodb/commit/adf04e8) [cherry-pick] Update Stash installation link (#173) (#175)
- [bdd4240](https://github.com/stashed/mongodb/commit/bdd4240) [cherry-pick] Fix output format + Add PreBackupActions logic (#149) (#163)
- [2078ffb](https://github.com/stashed/mongodb/commit/2078ffb) [cherry-pick] Make image.tag in values.yaml same as $APP_VERSION (#150) (#152)
- [848c360](https://github.com/stashed/mongodb/commit/848c360) [cherry-pick] Update chart icon (#139)
- [20e0d4f](https://github.com/stashed/mongodb/commit/20e0d4f) [cherry-pick] Make chart registry configurable (#126) (#128)


### [3.6.1-beta.20200826](https://github.com/stashed/mongodb/releases/tag/3.6.1-beta.20200826)

- [fbf6dfb](https://github.com/stashed/mongodb/commit/fbf6dfb) Prepare for release 3.6.1-beta.20200826 (#187)
- [50d9b51](https://github.com/stashed/mongodb/commit/50d9b51) [cherry-pick] Update Stash installation link (#173) (#176)
- [138c910](https://github.com/stashed/mongodb/commit/138c910) [cherry-pick] Fix output format + Add PreBackupActions logic (#149) (#164)
- [95d8800](https://github.com/stashed/mongodb/commit/95d8800) [cherry-pick] Make image.tag in values.yaml same as $APP_VERSION (#150) (#153)
- [f6dcc94](https://github.com/stashed/mongodb/commit/f6dcc94) [cherry-pick] Update chart icon (#140)
- [0449fbf](https://github.com/stashed/mongodb/commit/0449fbf) [cherry-pick] Make chart registry configurable (#126) (#129)


### [3.6.8-beta.20200826](https://github.com/stashed/mongodb/releases/tag/3.6.8-beta.20200826)

- [a2098a8](https://github.com/stashed/mongodb/commit/a2098a8) Prepare for release 3.6.8-beta.20200826 (#188)
- [81cad93](https://github.com/stashed/mongodb/commit/81cad93) [cherry-pick] Update Stash installation link (#173) (#177)
- [0711cec](https://github.com/stashed/mongodb/commit/0711cec) [cherry-pick] Fix output format + Add PreBackupActions logic (#149) (#165)
- [cd921b7](https://github.com/stashed/mongodb/commit/cd921b7) [cherry-pick] Make image.tag in values.yaml same as $APP_VERSION (#150) (#154)
- [5d29173](https://github.com/stashed/mongodb/commit/5d29173) [cherry-pick] Update chart icon (#141)
- [a8265a7](https://github.com/stashed/mongodb/commit/a8265a7) [cherry-pick] Make chart registry configurable (#126) (#130)


### [4.0.1-beta.20200826](https://github.com/stashed/mongodb/releases/tag/4.0.1-beta.20200826)

- [b794eee](https://github.com/stashed/mongodb/commit/b794eee) Prepare for release 4.0.1-beta.20200826 (#189)
- [8e835ce](https://github.com/stashed/mongodb/commit/8e835ce) [cherry-pick] Update Stash installation link (#173) (#178)
- [0025cc4](https://github.com/stashed/mongodb/commit/0025cc4) [cherry-pick] Fix output format + Add PreBackupActions logic (#149) (#166)
- [bc8f7a5](https://github.com/stashed/mongodb/commit/bc8f7a5) [cherry-pick] Make image.tag in values.yaml same as $APP_VERSION (#150) (#155)
- [7b0bb50](https://github.com/stashed/mongodb/commit/7b0bb50) [cherry-pick] Update chart icon (#142)
- [a9621b6](https://github.com/stashed/mongodb/commit/a9621b6) [cherry-pick] Make chart registry configurable (#126) (#131)


### [4.0.3-beta.20200826](https://github.com/stashed/mongodb/releases/tag/4.0.3-beta.20200826)

- [fd4d40e](https://github.com/stashed/mongodb/commit/fd4d40e) Prepare for release 4.0.3-beta.20200826 (#190)
- [3b2f40b](https://github.com/stashed/mongodb/commit/3b2f40b) [cherry-pick] Update Stash installation link (#173) (#179)
- [5ba31a4](https://github.com/stashed/mongodb/commit/5ba31a4) [cherry-pick] Fix output format + Add PreBackupActions logic (#149) (#167)
- [349f6c0](https://github.com/stashed/mongodb/commit/349f6c0) [cherry-pick] Make image.tag in values.yaml same as $APP_VERSION (#150) (#156)
- [eb32485](https://github.com/stashed/mongodb/commit/eb32485) [cherry-pick] Update chart icon (#143)
- [e2b05d8](https://github.com/stashed/mongodb/commit/e2b05d8) [cherry-pick] Make chart registry configurable (#126) (#132)


### [4.0.5-beta.20200826](https://github.com/stashed/mongodb/releases/tag/4.0.5-beta.20200826)

- [1bbff77](https://github.com/stashed/mongodb/commit/1bbff77) Prepare for release 4.0.5-beta.20200826 (#191)
- [51d0388](https://github.com/stashed/mongodb/commit/51d0388) [cherry-pick] Update Stash installation link (#173) (#180)
- [444ed15](https://github.com/stashed/mongodb/commit/444ed15) [cherry-pick] Fix output format + Add PreBackupActions logic (#149) (#168)
- [00cfb11](https://github.com/stashed/mongodb/commit/00cfb11) [cherry-pick] Make image.tag in values.yaml same as $APP_VERSION (#150) (#157)
- [8a0c6ce](https://github.com/stashed/mongodb/commit/8a0c6ce) [cherry-pick] Update chart icon (#144)
- [e514864](https://github.com/stashed/mongodb/commit/e514864) [cherry-pick] Make chart registry configurable (#126) (#133)


### [4.1.1-beta.20200826](https://github.com/stashed/mongodb/releases/tag/4.1.1-beta.20200826)

- [ad80c46](https://github.com/stashed/mongodb/commit/ad80c46) Prepare for release 4.1.1-beta.20200826 (#192)
- [a6017d0](https://github.com/stashed/mongodb/commit/a6017d0) [cherry-pick] Update Stash installation link (#173) (#181)
- [bbb9a42](https://github.com/stashed/mongodb/commit/bbb9a42) [cherry-pick] Fix output format + Add PreBackupActions logic (#149) (#169)
- [a207fd0](https://github.com/stashed/mongodb/commit/a207fd0) [cherry-pick] Make image.tag in values.yaml same as $APP_VERSION (#150) (#158)
- [f877621](https://github.com/stashed/mongodb/commit/f877621) [cherry-pick] Update chart icon (#145)
- [628df7e](https://github.com/stashed/mongodb/commit/628df7e) [cherry-pick] Make chart registry configurable (#126) (#134)


### [4.1.4-beta.20200826](https://github.com/stashed/mongodb/releases/tag/4.1.4-beta.20200826)

- [c092f8a](https://github.com/stashed/mongodb/commit/c092f8a) Prepare for release 4.1.4-beta.20200826 (#193)
- [cbb683a](https://github.com/stashed/mongodb/commit/cbb683a) [cherry-pick] Update Stash installation link (#173) (#182)
- [f8b1f2c](https://github.com/stashed/mongodb/commit/f8b1f2c) [cherry-pick] Fix output format + Add PreBackupActions logic (#149) (#170)
- [b99744e](https://github.com/stashed/mongodb/commit/b99744e) [cherry-pick] Make image.tag in values.yaml same as $APP_VERSION (#150) (#159)
- [9ee1654](https://github.com/stashed/mongodb/commit/9ee1654) [cherry-pick] Update chart icon (#146)
- [8604d7b](https://github.com/stashed/mongodb/commit/8604d7b) [cherry-pick] Make chart registry configurable (#126) (#135)


### [4.1.7-beta.20200826](https://github.com/stashed/mongodb/releases/tag/4.1.7-beta.20200826)

- [8334c89](https://github.com/stashed/mongodb/commit/8334c89) Prepare for release 4.1.7-beta.20200826 (#194)
- [905bdde](https://github.com/stashed/mongodb/commit/905bdde) [cherry-pick] Update Stash installation link (#173) (#183)
- [1eea836](https://github.com/stashed/mongodb/commit/1eea836) [cherry-pick] Fix output format + Add PreBackupActions logic (#149) (#171)
- [52c06d6](https://github.com/stashed/mongodb/commit/52c06d6) [cherry-pick] Make image.tag in values.yaml same as $APP_VERSION (#150) (#160)
- [7765f65](https://github.com/stashed/mongodb/commit/7765f65) [cherry-pick] Update chart icon (#147)
- [7426cba](https://github.com/stashed/mongodb/commit/7426cba) [cherry-pick] Make chart registry configurable (#126) (#136)


### [4.2.3-beta.20200826](https://github.com/stashed/mongodb/releases/tag/4.2.3-beta.20200826)

- [244da2c](https://github.com/stashed/mongodb/commit/244da2c) Prepare for release 4.2.3-beta.20200826 (#195)
- [953e51f](https://github.com/stashed/mongodb/commit/953e51f) Update Stash installation link (#173) (#184)
- [d9c6d0b](https://github.com/stashed/mongodb/commit/d9c6d0b) [cherry-pick] Fix output format + Add PreBackupActions logic (#149) (#172)
- [0ef5cc0](https://github.com/stashed/mongodb/commit/0ef5cc0) [cherry-pick] Make image.tag in values.yaml same as $APP_VERSION (#150) (#161)
- [a51bdac](https://github.com/stashed/mongodb/commit/a51bdac) [cherry-pick] Update chart icon (#148)
- [2ad3b80](https://github.com/stashed/mongodb/commit/2ad3b80) [cherry-pick] Make chart registry configurable (#126) (#137)



## [stashed/mysql](https://github.com/stashed/mysql)

### [5.7.25-beta.20200826](https://github.com/stashed/mysql/releases/tag/5.7.25-beta.20200826)

- [813ee01](https://github.com/stashed/mysql/commit/813ee01) Prepare for release 5.7.25-beta.20200826 (#85)
- [e646be7](https://github.com/stashed/mysql/commit/e646be7) [cherry-pick] Update Stash installation link (#81) (#82)
- [ecc7d0c](https://github.com/stashed/mysql/commit/ecc7d0c) [cherry-pick] Fix output format (#46) (#78)
- [7fcbd24](https://github.com/stashed/mysql/commit/7fcbd24) [cherry-pick] Pass image tag in values.yaml file (#74) (#75)
- [3188584](https://github.com/stashed/mysql/commit/3188584) [cherry-pick] Update chart icon (#71)
- [9060c7a](https://github.com/stashed/mysql/commit/9060c7a) [cherry-pick] Make chart registry configurable (#67) (#68)


### [8.0.3-beta.20200826](https://github.com/stashed/mysql/releases/tag/8.0.3-beta.20200826)

- [7326b37](https://github.com/stashed/mysql/commit/7326b37) Prepare for release 8.0.3-beta.20200826 (#87)
- [72ba0ee](https://github.com/stashed/mysql/commit/72ba0ee) [cherry-pick] Update Stash installation link (#81) (#84)
- [3e856a9](https://github.com/stashed/mysql/commit/3e856a9) [cherry-pick] Fix output format (#46) (#80)
- [8621d51](https://github.com/stashed/mysql/commit/8621d51) [cherry-pick] Pass image tag in values.yaml file (#74) (#77)
- [41702eb](https://github.com/stashed/mysql/commit/41702eb) [cherry-pick] Update chart icon (#73)
- [31740f9](https://github.com/stashed/mysql/commit/31740f9) [cherry-pick] Make chart registry configurable (#67) (#70)


### [8.0.14-beta.20200826](https://github.com/stashed/mysql/releases/tag/8.0.14-beta.20200826)

- [5816915](https://github.com/stashed/mysql/commit/5816915) Prepare for release 8.0.14-beta.20200826 (#86)
- [96b78ca](https://github.com/stashed/mysql/commit/96b78ca) [cherry-pick] Update Stash installation link (#81) (#83)
- [4f6c15b](https://github.com/stashed/mysql/commit/4f6c15b) [cherry-pick] Fix output format (#46) (#79)
- [9da1616](https://github.com/stashed/mysql/commit/9da1616) [cherry-pick] Pass image tag in values.yaml file (#74) (#76)
- [72edf00](https://github.com/stashed/mysql/commit/72edf00) [cherry-pick] Update chart icon (#72)
- [9f253db](https://github.com/stashed/mysql/commit/9f253db) [cherry-pick] Make chart registry configurable (#67) (#69)



## [stashed/percona-xtradb](https://github.com/stashed/percona-xtradb)

### [5.7-beta.20200826](https://github.com/stashed/percona-xtradb/releases/tag/5.7-beta.20200826)

- [10b9a66](https://github.com/stashed/percona-xtradb/commit/10b9a66) Prepare for release 5.7-beta.20200826 (#49)
- [53f26da](https://github.com/stashed/percona-xtradb/commit/53f26da) [cherry-pick] Update Stash installation link (#47) (#48)
- [3eae3d3](https://github.com/stashed/percona-xtradb/commit/3eae3d3) [cherry-pick] Fix output format + Add PreBackupActions logic (#43) (#46)
- [0fa2765](https://github.com/stashed/percona-xtradb/commit/0fa2765) [cherry-pick] Make image.tag in values.yaml file as same as $APP_VERSION (#44) (#45)
- [7ad46c0](https://github.com/stashed/percona-xtradb/commit/7ad46c0) [cherry-pick] Update chart icon (#42)
- [97f7e6d](https://github.com/stashed/percona-xtradb/commit/97f7e6d) [cherry-pick] Make chart registry configurable (#40) (#41)



## [stashed/postgres](https://github.com/stashed/postgres)

### [9.6-beta.20200826](https://github.com/stashed/postgres/releases/tag/9.6-beta.20200826)

- [61c4425](https://github.com/stashed/postgres/commit/61c4425) Prepare for release 9.6-beta.20200826 (#141)
- [b313c2c](https://github.com/stashed/postgres/commit/b313c2c) Update Stash installation link (#131) (#136)
- [d73595e](https://github.com/stashed/postgres/commit/d73595e) [cherry-pick] Update twitter handle (#125) (#130)
- [875edd5](https://github.com/stashed/postgres/commit/875edd5) [cherry-pick] Use image tag same as the $APP_VERSION (#114) (#124)
- [5a9a784](https://github.com/stashed/postgres/commit/5a9a784) [cherry-pick] Fix output format + Add PreBackupActions logic (#113) (#119)
- [ce47f3a](https://github.com/stashed/postgres/commit/ce47f3a) [cherry-pick] Update chart icon (#112)
- [decfbb0](https://github.com/stashed/postgres/commit/decfbb0) [cherry-pick] Make chart registry configurable (#102) (#107)


### [10.2-beta.20200826](https://github.com/stashed/postgres/releases/tag/10.2-beta.20200826)

- [ac65186](https://github.com/stashed/postgres/commit/ac65186) Prepare for release 10.2-beta.20200826 (#137)
- [1757b5f](https://github.com/stashed/postgres/commit/1757b5f) [cherry-pick] Update Stash installation link (#131) (#132)
- [56f90c7](https://github.com/stashed/postgres/commit/56f90c7) [cherry-pick] Update twitter handle (#125) (#126)
- [9734711](https://github.com/stashed/postgres/commit/9734711) [cherry-pick] Use image tag same as the $APP_VERSION (#114) (#120)
- [9556ade](https://github.com/stashed/postgres/commit/9556ade) [cherry-pick] Fix output format + Add PreBackupActions logic (#113) (#115)
- [ce7257e](https://github.com/stashed/postgres/commit/ce7257e) [cherry-pick] Update chart icon (#108)
- [754640f](https://github.com/stashed/postgres/commit/754640f) [cherry-pick] Make chart registry configurable (#102) (#103)


### [10.6-beta.20200826](https://github.com/stashed/postgres/releases/tag/10.6-beta.20200826)

- [1446996](https://github.com/stashed/postgres/commit/1446996) Prepare for release 10.6-beta.20200826 (#138)
- [720cc7b](https://github.com/stashed/postgres/commit/720cc7b) [cherry-pick] Update Stash installation link (#131) (#133)
- [6b17dea](https://github.com/stashed/postgres/commit/6b17dea) [cherry-pick] Update twitter handle (#125) (#127)
- [d1af1c6](https://github.com/stashed/postgres/commit/d1af1c6) [cherry-pick] Use image tag same as the $APP_VERSION (#114) (#121)
- [cd2f8ec](https://github.com/stashed/postgres/commit/cd2f8ec) [cherry-pick] Fix output format + Add PreBackupActions logic (#113) (#116)
- [d89559c](https://github.com/stashed/postgres/commit/d89559c) [cherry-pick] Update chart icon (#109)
- [192c564](https://github.com/stashed/postgres/commit/192c564) [cherry-pick] Make chart registry configurable (#102) (#104)


### [11.1-beta.20200826](https://github.com/stashed/postgres/releases/tag/11.1-beta.20200826)

- [0cc9647](https://github.com/stashed/postgres/commit/0cc9647) Prepare for release 11.1-beta.20200826 (#139)
- [dff51be](https://github.com/stashed/postgres/commit/dff51be) [cherry-pick] Update Stash installation link (#131) (#134)
- [b8480c8](https://github.com/stashed/postgres/commit/b8480c8) [cherry-pick] Update twitter handle (#125) (#128)
- [33fe94b](https://github.com/stashed/postgres/commit/33fe94b) [cherry-pick] Use image tag same as the $APP_VERSION (#114) (#122)
- [6a631b0](https://github.com/stashed/postgres/commit/6a631b0) [cherry-pick] Fix output format + Add PreBackupActions logic (#113) (#117)
- [8bccb9c](https://github.com/stashed/postgres/commit/8bccb9c) [cherry-pick] Update chart icon (#110)
- [aca1577](https://github.com/stashed/postgres/commit/aca1577) [cherry-pick] Make chart registry configurable (#102) (#105)


### [11.2-beta.20200826](https://github.com/stashed/postgres/releases/tag/11.2-beta.20200826)

- [cf876d3](https://github.com/stashed/postgres/commit/cf876d3) Prepare for release 11.2-beta.20200826 (#140)
- [22d4f2c](https://github.com/stashed/postgres/commit/22d4f2c) [cherry-pick] Update Stash installation link (#131) (#135)
- [462bf9f](https://github.com/stashed/postgres/commit/462bf9f) [cherry-pick] Update twitter handle (#125) (#129)
- [baa8b56](https://github.com/stashed/postgres/commit/baa8b56) [cherry-pick] Use image tag same as the $APP_VERSION (#114) (#123)
- [b09b460](https://github.com/stashed/postgres/commit/b09b460) [cherry-pick] Fix output format + Add PreBackupActions logic (#113) (#118)
- [08ea901](https://github.com/stashed/postgres/commit/08ea901) [cherry-pick] Update chart icon (#111)
- [a6b8c9e](https://github.com/stashed/postgres/commit/a6b8c9e) [cherry-pick] Make chart registry configurable (#102) (#106)



## [stashed/stash](https://github.com/stashed/stash)

### [v0.10.0-rc.0](https://github.com/stashed/stash/releases/tag/v0.10.0-rc.0)

- [3ebb7788](https://github.com/stashed/stash/commit/3ebb7788) Prepare for release v0.10.0-rc.0 (#1179)
- [48e5c87e](https://github.com/stashed/stash/commit/48e5c87e) Update Kubernetes v1.18.3 dependencies (#1178)
- [c269c8ed](https://github.com/stashed/stash/commit/c269c8ed) Port changes from enterprise version (#1176)
- [0fea720b](https://github.com/stashed/stash/commit/0fea720b) Update Kubernetes v1.18.3 dependencies (#1174)
- [c4b6013f](https://github.com/stashed/stash/commit/c4b6013f) Update Kubernetes v1.18.3 dependencies (#1173)
- [77b3eab8](https://github.com/stashed/stash/commit/77b3eab8) Update Kubernetes v1.18.3 dependencies (#1169)
- [64f7c292](https://github.com/stashed/stash/commit/64f7c292) Update Kubernetes v1.18.3 dependencies (#1168)
- [2d6fd7da](https://github.com/stashed/stash/commit/2d6fd7da) Update Kubernetes v1.18.3 dependencies (#1167)
- [378de04d](https://github.com/stashed/stash/commit/378de04d) Update Kubernetes v1.18.3 dependencies (#1159)
- [20decffa](https://github.com/stashed/stash/commit/20decffa) Build images in e2e workflow
- [4919cb03](https://github.com/stashed/stash/commit/4919cb03) Allow configuring k8s in e2e tests (#1155)
- [6e71662a](https://github.com/stashed/stash/commit/6e71662a) Update to Kubernetes v1.18.3 (#1154)
- [a83021fd](https://github.com/stashed/stash/commit/a83021fd) Trigger e2e tests on /ok-to-test command (#1150)
- [6bf44ceb](https://github.com/stashed/stash/commit/6bf44ceb) Update to Kubernetes v1.18.3 (#1149)
- [bcbb3a98](https://github.com/stashed/stash/commit/bcbb3a98) Update to Kubernetes v1.18.3 (#1148)
- [3b717aac](https://github.com/stashed/stash/commit/3b717aac) Prepare for release v0.10.0-beta.1 (#1146)
- [c8b81cf7](https://github.com/stashed/stash/commit/c8b81cf7) Prepare for release v0.10.0-beta.0 (#1145)
- [2d145f47](https://github.com/stashed/stash/commit/2d145f47) Clarify Docker images are dually licensed
- [693ab7df](https://github.com/stashed/stash/commit/693ab7df) Update License (#1144)
- [e13d67eb](https://github.com/stashed/stash/commit/e13d67eb) Update to Kubernetes v1.18.3 (#1142)
- [26ee605a](https://github.com/stashed/stash/commit/26ee605a) Update ci.yml
- [9fa95666](https://github.com/stashed/stash/commit/9fa95666) Add workflow to update docs (#1136)
- [95a62a95](https://github.com/stashed/stash/commit/95a62a95) Update update-release-tracker.sh
- [379c90d5](https://github.com/stashed/stash/commit/379c90d5) Update update-release-tracker.sh
- [cd0a70ee](https://github.com/stashed/stash/commit/cd0a70ee) Use GITHUB_BASE_REF to detect target branch
- [e27c5f66](https://github.com/stashed/stash/commit/e27c5f66) Add script to update release tracker on pr merge (#1132)
- [b0dd5051](https://github.com/stashed/stash/commit/b0dd5051) Update .kodiak.toml
- [e87bad80](https://github.com/stashed/stash/commit/e87bad80) Parameterize installer namespace
- [da8d8956](https://github.com/stashed/stash/commit/da8d8956) Format CI workflows
- [bbde40a3](https://github.com/stashed/stash/commit/bbde40a3) Update to Kubernetes v1.18.3 (#1129)
- [38eb3781](https://github.com/stashed/stash/commit/38eb3781) Update to Kubernetes v1.18.3
- [197aa7bd](https://github.com/stashed/stash/commit/197aa7bd) Create .kodiak.toml
- [181ca49e](https://github.com/stashed/stash/commit/181ca49e) Update coverage script
- [26602c96](https://github.com/stashed/stash/commit/26602c96) Merge pull request #1125 from stashed/fix-ci-tests
- [54f87b78](https://github.com/stashed/stash/commit/54f87b78) Increase wait timeout
- [43428085](https://github.com/stashed/stash/commit/43428085) Remove unnecessary test codes + run test in parallel
- [8a780e0c](https://github.com/stashed/stash/commit/8a780e0c) Fix clone-pvc tests
- [7027c0f6](https://github.com/stashed/stash/commit/7027c0f6) Fix E2E test
- [31de588a](https://github.com/stashed/stash/commit/31de588a) Change GCS test bucket name to stash-ci (#1122)
- [30a490a6](https://github.com/stashed/stash/commit/30a490a6) Merge pull request #1121 from stashed/baseline-psp
- [419a18e3](https://github.com/stashed/stash/commit/419a18e3) Use StringSlice type flag
- [9dd3804d](https://github.com/stashed/stash/commit/9dd3804d) Make PSP names configurable through flag
- [e4edef44](https://github.com/stashed/stash/commit/e4edef44) Always use baseline PSP
- [cf1538a0](https://github.com/stashed/stash/commit/cf1538a0) Use filepath.Join to generate Repository prefix for BackupBatch (#1120)
- [be189169](https://github.com/stashed/stash/commit/be189169) Go back to using engineerd/setup-kind
- [ae2d74fa](https://github.com/stashed/stash/commit/ae2d74fa) Update dependencies (#1117)
- [a93a5b4c](https://github.com/stashed/stash/commit/a93a5b4c) Remove defaults from CRD v1beta1 (#1116)
- [40e65761](https://github.com/stashed/stash/commit/40e65761) Use CRD v1 for Kubernetes >= 1.16 (#1115)
- [7d851e53](https://github.com/stashed/stash/commit/7d851e53) Merge pull request #1114 from stashed/x7
- [352ddeed](https://github.com/stashed/stash/commit/352ddeed) Use preinstalled kind
- [11c9e422](https://github.com/stashed/stash/commit/11c9e422) Pass context
- [21053603](https://github.com/stashed/stash/commit/21053603) Update to Kubernetes 1.18.3
- [f450e9cc](https://github.com/stashed/stash/commit/f450e9cc) Add wait for target logic + add conditions for BackupConfiguration + BackupBatch + RestoreSession (#1108)
- [8f8ff87e](https://github.com/stashed/stash/commit/8f8ff87e) Fix volume snapshot job cleanup (#1090)
- [a4a868b5](https://github.com/stashed/stash/commit/a4a868b5) Merge pull request #1111 from stashed/fix-interimVolume
- [108d0252](https://github.com/stashed/stash/commit/108d0252) Set BackupSession as owner of the pvc created from interimVolumeTemplate
- [fd136c53](https://github.com/stashed/stash/commit/fd136c53) Use Go 1.14.3
- [74c71d22](https://github.com/stashed/stash/commit/74c71d22) Update crazy-max/ghaction-docker-buildx flag
- [f783899b](https://github.com/stashed/stash/commit/f783899b) Trigger the workflow on push to master
- [e7eceb30](https://github.com/stashed/stash/commit/e7eceb30) Trigger the workflow on push or pull request
- [fe479e8c](https://github.com/stashed/stash/commit/fe479e8c) Use kind v0.8.0
- [9fc4665a](https://github.com/stashed/stash/commit/9fc4665a) Merge pull request #1093 from robotinfra/master
- [ef2d57e3](https://github.com/stashed/stash/commit/ef2d57e3) fix typo succesSfully
- [d8d35c49](https://github.com/stashed/stash/commit/d8d35c49) fix event types mismatch
- [53dfe8b0](https://github.com/stashed/stash/commit/53dfe8b0) Update stash labels in Makefile
- [c8081c1d](https://github.com/stashed/stash/commit/c8081c1d) Pass image pull secrets to helm chart
- [37b9b312](https://github.com/stashed/stash/commit/37b9b312) Use Go 1.14.2 (#1074)
- [09621974](https://github.com/stashed/stash/commit/09621974) Update K8s version 1.14.6 to 1.14.10 (#1084)
- [8a1ab32c](https://github.com/stashed/stash/commit/8a1ab32c) Give backup triggering CronJob all permissions for Stash crds (#1083)
- [53b932b1](https://github.com/stashed/stash/commit/53b932b1) Use kubectl 1.17 (#1082)
- [5cdeebee](https://github.com/stashed/stash/commit/5cdeebee) Fix nil pointer exception during VolumeSnapshot (#1073)
- [30630d60](https://github.com/stashed/stash/commit/30630d60) Assign returned error properly crateRestoreSessoin() (#1069)
- [3fcbe1b7](https://github.com/stashed/stash/commit/3fcbe1b7) Update README.md to reflect Stash's capability properly (#1060)
- [53513cfe](https://github.com/stashed/stash/commit/53513cfe) Update README.md
- [d615e2c0](https://github.com/stashed/stash/commit/d615e2c0) Add license scan report and status (#1031)




