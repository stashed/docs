---
title: Concepts | Stash
menu:
  docs_{{ .version }}:
    identifier: concepts-readme
    name: README
    parent: concepts
    weight: -1
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: concepts
url: /docs/{{ .version }}/concepts/
aliases:
  - /docs/{{ .version }}/concepts/README/
---

# Concepts

Concepts help you to learn about the different parts of the Stash and the abstractions it uses.

This concept section is divided into the following modules:

- What is Stash?
  - [Overview](/docs/concepts/what-is-stash/overview.md) provides an introduction to Stash. It also give an overview of the features it provides.
  - [Architecture](/docs/concepts/what-is-stash/architecture.md) provides a visual representation of Stash architecture. It also provides a brief overview of the components it uses.

- Declarative API
  - [Repository](/docs/concepts/crds/repository.md) introduces the concept of `Repository` crd that holds backend information in a Kubernetes native way.
  - [BackupConfiguration](/docs/concepts/crds/backupconfiguration/index.md) introduces the concept of `BackupConfiguration` crd that is used to configure backup for a target resource in a Kubernetes native way.
  - [BackupBatch](/docs/concepts/crds/backupbatch/index.md) introduces the concept of `BackupBatch` crd that is used to setup backup of multiple co-related targets under single configuration.
  - [BackupSession](/docs/concepts/crds/backupsession/index.md) introduces the concept of `BackupSession` crd that represents a backup run of a target resource for the respective `BackupConfiguration` or `BackupBatch` object.
  - [RestoreSession](/docs/concepts/crds/restoresession/index.md) introduces the concept of `RestoreSession` crd that represents a restore run of a target resource.
  - [RestoreBatch](/docs/concepts/crds/restorebatch/index.md) introduces the concept of `RestoreBatch` crd that allows restore of multiple targets that were backed up using `BackupBatch` under single configuration.
  - [Function](/docs/concepts/crds/function/index.md) introduces the concept of `Function` crd that represents a step of a backup or restore process.
  - [Task](/docs/concepts/crds/task/index.md) introduces the concept of `Task` crd which specifies an ordered collection of multiple `Function`s and their parameters that make up a complete backup or restore process.
  - [BackupBlueprint](/docs/concepts/crds/backupblueprint/index.md) introduces the concept of `BackupBlueprint` crd that specifies a blueprint for `Repository` and `BackupConfiguration` object which provides an option to share backup configuration across similar targets.
  - [AppBinding](/docs/concepts/crds/appbinding/index.md) introduces the concept of `AppBinding` crd which holds the information that are necessary to connect with an application like database.
  - [Snapshot](/docs/concepts/crds/snapshot/index.md) introduces the concept of `Snapshot` object that represents backed up snapshots in a Kubernetes native way.
