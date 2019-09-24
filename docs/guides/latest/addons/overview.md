---
title: Addons | Stash
menu:
  product_stash_{{ .version }}:
    identifier: addons-overview
    name: Overview
    parent: addons
    weight: 10
product_name: stash
menu_name: product_stash_{{ .version }}
section_menu_id: guides
---

# Stash Addons

Stash 0.9.0+ supports extending its functionality through addon mechanism. This guide will give you an overview of what is an addon, how addon works and available Stash addons.

## What is an Addon

Stash 0.9.0+ uses two different models for backup/restore based on target types. One of them is **sidecar model** where Stash injects a `sidecar`/`init-container` into the targeted workload for backup/restore. Another is **job model** where backup or restore is done via an external job.

The job model is further divided into two categories. In the first category, the targeted resource is well known to Stash. Hence, the Stash operator itself can create the required job to backup/restore the target. In the second category, Stash follows `Function-Task` model where the targeted resource is not known to Stash. In this case, the user creates some [Function](/docs/concepts/crds/function.md) which resembles a step of backup/restore process and a [Task](/docs/concepts/crds/task.md) which specifies the order of execution of these steps. Stash uses these `Function` and `Task` to generate the required job definition to backup/restore the target.

The `Function-Task` model enables Stash to backup/restore the resources that the operator itself is not aware of. The users themselves can extend Stash by creating respective `Function` and `Task` to backup their desired resources.

A Stash addon is just some [Function](/docs/concepts/crds/function.md)  and [Task](/docs/concepts/crds/task.md) definition to backup & restore a specific resource.

## How Addon Works

When an user install a Stash addon, it creates some `Function` and `Task` definitions. Then, when he creates a `BackupConfiguration` or `RestoreSession` object to backup/restore his desired resource, Stash operator resolves the `Function` and `Task` then creates a Job to backup/restore the target. The following diagram shows how addons works in Stash:

<figure align="center">
  <img alt="Stash Addon Overview" src="/docs/images/guides/latest/addons/addon_overview.svg">
  <figcaption align="center">Fig: Stash Addon Overview</figcaption>
</figure>

A `Function` is fundamentally a container specification and `Task` specifies the execution order of the containers. Stash operator injects all the containers except last one resolved from the `Function` specified in  `Task` as `init-container` into the job in the same order they appear in the `Task`. Then, it injects the last container as the main container of the Job.

## Available Addons

The following addons are available for Stash:

| Addons                                                | Usage                                    | Available Versions |
| ----------------------------------------------------- | ---------------------------------------- | ------------------ |
| [PostgreSQL](/docs/addons/postgres/README.md)         | Backup & Restore PostgreSQL databases    | 11.2               |
| [Elasticsearch](/docs/addons/elasticsearch/README.md) | Backup & Restore Elasticsearch databases | 6.5                |
| [MySQL](/docs/addons/mysql/README.md)                 | Backup & Restore MySQL databases         | 8.0.14             |
| [MongoDB](/docs/addons/mongodb/README.md)             | Backup & Restore MongoDB databases       | 3.6                |
