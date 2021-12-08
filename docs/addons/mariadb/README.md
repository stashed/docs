---
title: MariaDB Addon Overview | Stash
description: MariaDB Addon Overview | Stash
menu:
  docs_{{ .version }}:
    identifier: stash-mariadb-readme
    name: Readme
    parent: stash-mariadb
    weight: -1
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: stash-addons
url: /docs/{{ .version }}/addons/mariadb/
aliases:
  - /docs/{{ .version }}/addons/mariadb/README/
---

{{< notice type="warning" message="This is an Enterprise-only feature. Please install [Stash Enterprise Edition](/docs/setup/install/enterprise.md) to try this feature." >}}

# Stash MariaDB Addon

Stash 0.9.0+ supports extending its functionality through addons. Stash MariaDB addon enables Stash to backup and restore MariaDB databases.

This guide will give you an overview of which MariaDB versions are supported and how the docs are organized.

## Supported MariaDB Versions

Stash has the following addon versions for MariaDB:

{{< versionlist "mariadb">}}

Here, the addon follows `M.M.P` versioning scheme where `M.M.P` (Major.Minor.Patch) represents the respective database version.

## Addon Version Compatibility

Any addon with matching major version with the database version should be able to take backup of that database. For example, MariaDB addon with version `10.x.x` should be able take backup of any MariaDB of `10.x.x` series. However, this might not be true for some versions. In that case, we will have separate addon for that version.

## Documentation Overview

Stash MariaDB documentations are organized as below:

- [How does it work?](/docs/addons/mariadb/overview/index.md) gives an overview of how backup and restore process for MariaDB database works in Stash.
- [Helm managed MariaDB](/docs/addons/mariadb/helm/index.md) shows how to backup and restore a Helm managed MariaDB database.
- [Auto-Backup](/docs/addons/mariadb/auto-backup/index.md) shows how to configure a generic backup template for all the MariaDB databases of a cluster.
- [Customizing Backup & Restore Process](/docs/addons/mariadb/customization/index.md) shows how to customize the backup & restore process.
