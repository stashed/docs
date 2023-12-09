---
title: PostgreSQL Addon Overview | Stash
description: PostgreSQL Addon Overview | Stash
menu:
  docs_{{ .version }}:
    identifier: stash-postgres-readme
    name: Readme
    parent: stash-postgres
    weight: -1
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: stash-addons
url: /docs/{{ .version }}/addons/postgres/
aliases:
  - /docs/{{ .version }}/addons/postgres/README/
---

# Stash PostgreSQL Addon

Stash 0.9.0+ supports extending its functionality through addons. Stash PostgreSQL addon enables Stash to backup and restore PostgreSQL databases.

This guide will give you an overview of which PostgreSQL versions are supported and how the docs are organized.

## Supported PostgreSQL Versions

Stash has the following addon versions for PostgreSQL:

{{< versionlist "postgres">}}

Here, the addon follows `M.M.P` versioning scheme where `M.M.P` (Major.Minor.Patch) represents the respective database version.

## Addon Version Compatibility

Any addon with matching major version with the database version should be able to take backup of that database. For example, PostgreSQL addon with version `12.x.x` should be able take backup of any PostgreSQL of `12.x.x` series. However, this might not be true for some versions. In that case, we will have separate addon for that version.

## Documentation Overview

Stash PostgreSQL documentations are organized as below:

- [How does it work?](/docs/addons/postgres/overview/index.md) gives an overview of how backup and restore process for PostgreSQL database works in Stash.
- [Standalone PostgreSQL](/docs/addons/postgres/standalone/index.md) shows how to backup and restore a standalone PostgreSQL database using Stash.
- [Auto-Backup](/docs/addons/postgres/auto-backup/index.md) shows how to configure a generic backup template for all the PostgreSQL databases of a cluster.
