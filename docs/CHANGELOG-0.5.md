---
title: Changelog | Stash
description: Changelog
menu:
  docs_{{ .version }}:
    identifier: changelog-stash-0.5
    name: Changelog-0.5
    parent: welcome
    weight: 50
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: welcome
url: /docs/{{ .version }}/welcome/changelog-0.5/
aliases:
  - /docs/{{ .version }}/CHANGELOG-0.5/
---
# Change Log

## [0.5.1](https://github.com/appscode/stash/tree/0.5.1) (2017-10-10)
[Full Changelog](https://github.com/appscode/stash/compare/0.5.0...0.5.1)

**Fixed bugs:**

- invalid header field value for key Authorization - DO s3 bucket [\#189](https://github.com/appscode/stash/issues/189)
- Kops + AWS: cannot unmarshal array into Go value of type types.ContainerJSON [\#147](https://github.com/appscode/stash/issues/147)

**Closed issues:**

- Cut a new release with restic 0.7.1 [\#145](https://github.com/appscode/stash/issues/145)
- Use fixed Hostname for ReplicaSet etc [\#165](https://github.com/appscode/stash/issues/165)
- Update docs for restic tags [\#143](https://github.com/appscode/stash/issues/143)
- Document how to use with kubectl [\#142](https://github.com/appscode/stash/issues/142)

**Merged pull requests:**

- Correctly detect "default" service account [\#200](https://github.com/appscode/stash/pull/200) ([tamalsaha](https://github.com/tamalsaha))
- Clarify that --tag foo,tag bar style tags are not supported. [\#199](https://github.com/appscode/stash/pull/199) ([tamalsaha](https://github.com/tamalsaha))
- Set hostname based on resource type [\#198](https://github.com/appscode/stash/pull/198) ([tamalsaha](https://github.com/tamalsaha))
- Document how to detect operator version [\#196](https://github.com/appscode/stash/pull/196) ([tamalsaha](https://github.com/tamalsaha))
- Manage RoleBinding for rbac enabled cluster [\#197](https://github.com/appscode/stash/pull/197) ([tamalsaha](https://github.com/tamalsaha))

## [0.5.0](https://github.com/appscode/stash/tree/0.5.0) (2017-10-10)
[Full Changelog](https://github.com/appscode/stash/compare/0.5.0-beta.3...0.5.0)

**Closed issues:**

- Apply restic.appscode.com/config annotations on Pod templates [\#141](https://github.com/appscode/stash/issues/141)

**Merged pull requests:**

- Revendor forked robfig/cron [\#139](https://github.com/appscode/stash/pull/139) ([tamalsaha](https://github.com/tamalsaha))

## [0.5.0-beta.3](https://github.com/appscode/stash/tree/0.5.0-beta.3) (2017-10-10)
[Full Changelog](https://github.com/appscode/stash/compare/0.5.0-beta.2...0.5.0-beta.3)

**Merged pull requests:**

- Use workqueue for scheduler [\#194](https://github.com/appscode/stash/pull/194) ([tamalsaha](https://github.com/tamalsaha))

## [0.5.0-beta.2](https://github.com/appscode/stash/tree/0.5.0-beta.2) (2017-10-09)
[Full Changelog](https://github.com/appscode/stash/compare/0.5.0-beta.1...0.5.0-beta.2)

**Merged pull requests:**

- Add tests for DO [\#193](https://github.com/appscode/stash/pull/193) ([tamalsaha](https://github.com/tamalsaha))
- Update tutorial [\#186](https://github.com/appscode/stash/pull/186) ([diptadas](https://github.com/diptadas))

## [0.5.0-beta.1](https://github.com/appscode/stash/tree/0.5.0-beta.1) (2017-10-09)
[Full Changelog](https://github.com/appscode/stash/compare/0.5.0-beta.0...0.5.0-beta.1)

**Fixed bugs:**

- \[Bug\] Success/Fail prometheus metrics inverted condition [\#175](https://github.com/appscode/stash/issues/175)

**Closed issues:**

- `should backup new DaemonSet` failed [\#155](https://github.com/appscode/stash/issues/155)
- Use DaemonSet update \(1.6\) [\#154](https://github.com/appscode/stash/issues/154)

**Merged pull requests:**

- Fix prometheus metrics collection [\#192](https://github.com/appscode/stash/pull/192) ([tamalsaha](https://github.com/tamalsaha))
- Fix StatefulSet tests [\#190](https://github.com/appscode/stash/pull/190) ([tamalsaha](https://github.com/tamalsaha))
- Replace reflect.Equal with github.com/google/go-cmp [\#188](https://github.com/appscode/stash/pull/188) ([tamalsaha](https://github.com/tamalsaha))
- Skip ReplicaSet owned by Deployments [\#187](https://github.com/appscode/stash/pull/187) ([tamalsaha](https://github.com/tamalsaha))

## [0.5.0-beta.0](https://github.com/appscode/stash/tree/0.5.0-beta.0) (2017-10-09)
[Full Changelog](https://github.com/appscode/stash/compare/0.4.1...0.5.0-beta.0)

**Implemented enhancements:**

- Migrate TPR to CRD [\#160](https://github.com/appscode/stash/pull/160) ([sadlil](https://github.com/sadlil))

**Fixed bugs:**

- Error in request: v1.ListOptions is not suitable for converting to "v1" [\#153](https://github.com/appscode/stash/issues/153)
- Fix client-go updates [\#159](https://github.com/appscode/stash/pull/159) ([sadlil](https://github.com/sadlil))

**Closed issues:**

- Switch to CustomResourceDefinitions [\#97](https://github.com/appscode/stash/issues/97)
- Use client-go 4.0.0 [\#56](https://github.com/appscode/stash/issues/56)

**Merged pull requests:**

- Prepare docs for 5.0.0-beta.0 [\#185](https://github.com/appscode/stash/pull/185) ([tamalsaha](https://github.com/tamalsaha))
- Set namespaceIndex as indexer [\#184](https://github.com/appscode/stash/pull/184) ([tamalsaha](https://github.com/tamalsaha))
- Fix e2e tests [\#183](https://github.com/appscode/stash/pull/183) ([tamalsaha](https://github.com/tamalsaha))
- Use workqueue [\#182](https://github.com/appscode/stash/pull/182) ([tamalsaha](https://github.com/tamalsaha))
- Use Deployment from apps/v1beta1 [\#181](https://github.com/appscode/stash/pull/181) ([tamalsaha](https://github.com/tamalsaha))
- Delete \*.generated.go files for ugorji [\#180](https://github.com/appscode/stash/pull/180) ([tamalsaha](https://github.com/tamalsaha))
- Use WaitForCRDReady from kutil [\#179](https://github.com/appscode/stash/pull/179) ([tamalsaha](https://github.com/tamalsaha))
- Only watch apps/v1beta1 Deployment [\#178](https://github.com/appscode/stash/pull/178) ([tamalsaha](https://github.com/tamalsaha))
- Move kutil to client package [\#177](https://github.com/appscode/stash/pull/177) ([tamalsaha](https://github.com/tamalsaha))
- Generate ugorji stuff [\#176](https://github.com/appscode/stash/pull/176) ([tamalsaha](https://github.com/tamalsaha))
- Prepare docs for 0.5.0 [\#174](https://github.com/appscode/stash/pull/174) ([tamalsaha](https://github.com/tamalsaha))
- Install stash as a critical addon [\#173](https://github.com/appscode/stash/pull/173) ([tamalsaha](https://github.com/tamalsaha))
- Set RESTIC\_VER to 0.7.3 [\#172](https://github.com/appscode/stash/pull/172) ([tamalsaha](https://github.com/tamalsaha))
- Refresh charts to match recent convention [\#171](https://github.com/appscode/stash/pull/171) ([tamalsaha](https://github.com/tamalsaha))
- Update kutil [\#170](https://github.com/appscode/stash/pull/170) ([tamalsaha](https://github.com/tamalsaha))
- Fix deployment name in tutorial [\#169](https://github.com/appscode/stash/pull/169) ([the-redback](https://github.com/the-redback))
- Fix command in Developer-guide [\#168](https://github.com/appscode/stash/pull/168) ([the-redback](https://github.com/the-redback))
- Use apis/v1alpha1 instead of internal version [\#167](https://github.com/appscode/stash/pull/167) ([tamalsaha](https://github.com/tamalsaha))
- Remove resource:path [\#166](https://github.com/appscode/stash/pull/166) ([tamalsaha](https://github.com/tamalsaha))
- Move analytics collector to root command [\#164](https://github.com/appscode/stash/pull/164) ([tamalsaha](https://github.com/tamalsaha))
- Use kubernetes/code-generator [\#163](https://github.com/appscode/stash/pull/163) ([tamalsaha](https://github.com/tamalsaha))
- Revendor k8s.io/apiextensions-apiserver [\#162](https://github.com/appscode/stash/pull/162) ([tamalsaha](https://github.com/tamalsaha))
- Update kutil dependency [\#158](https://github.com/appscode/stash/pull/158) ([tamalsaha](https://github.com/tamalsaha))
- Use CheckAPIVersion\(\) [\#157](https://github.com/appscode/stash/pull/157) ([tamalsaha](https://github.com/tamalsaha))
- Use PATCH api instead of UPDATE [\#156](https://github.com/appscode/stash/pull/156) ([tamalsaha](https://github.com/tamalsaha))
- Check version using semver library [\#152](https://github.com/appscode/stash/pull/152) ([tamalsaha](https://github.com/tamalsaha))
- Support adding Sidecar containers for StatefulSet. [\#151](https://github.com/appscode/stash/pull/151) ([tamalsaha](https://github.com/tamalsaha))
- Update client-go to 4.0.0 [\#150](https://github.com/appscode/stash/pull/150) ([tamalsaha](https://github.com/tamalsaha))
- Update build commands for restic. [\#149](https://github.com/appscode/stash/pull/149) ([tamalsaha](https://github.com/tamalsaha))
- Update client-go to 3.0.0 from 3.0.0-beta [\#148](https://github.com/appscode/stash/pull/148) ([tamalsaha](https://github.com/tamalsaha))
- Add uninstall.sh script [\#144](https://github.com/appscode/stash/pull/144) ([tamalsaha](https://github.com/tamalsaha))
- Fix typos of tutorial.md file [\#138](https://github.com/appscode/stash/pull/138) ([sajibcse68](https://github.com/sajibcse68))
