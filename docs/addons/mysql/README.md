---
title: MySQL Addon Overview | Stash
description: MySQL Addon Overview | Stash
menu:
  docs_{{ .version }}:
    identifier: stash-mysql-readme
    name: Readme
    parent: stash-mysql
    weight: -1
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: stash-addons
url: /docs/{{ .version }}/addons/mysql/
aliases:
  - /docs/{{ .version }}/addons/mysql/README/
---

{{< notice type="warning" message="This is an Enterprise-only feature. Please install [Stash Enterprise Edition](/docs/setup/install/enterprise.md) to try this feature." >}}

# Stash MySQL Addon

Stash 0.9.0+ supports extending its functionality through addons. Stash MySQL addon enables Stash to backup and restore MySQL databases.

This guide will give you an overview of which MySQL versions are supported and how the docs are organized.

## Supported MySQL Versions

Stash has the following addon versions for MySQL:

{{< versionlist "mysql">}}

Here, the addon follows `M.M.P` versioning scheme where `M.M.P` (Major.Minor.Patch) represents the respective database version.

## Addon Version Compatibility

Any addon with matching major version with the database version should be able to take backup of that database. For example, MySQL addon with version `8.x.x` should be able take backup of any MySQL of `8.x.x` series. However, this might not be true for some versions. In that case, we will have separate addon for that version.

## Documentation Overview

Stash MySQL documentations are organized as below:

- [How does it works?](/docs/addons/mysql/overview/index.md) gives an overview of how backup and restore process for MySQL database works in Stash.
- [Standalone MySQL Database](/docs/addons/mysql/standalone/index.md) shows how to backup and restore a standalone MySQL database.
