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

Stash supports backup and restore of the following MariaDB versions:

{{< versionlist "mariadb" "/docs/addons/mariadb/guides/%s/mariadb.md" >}}

Here, the addon follows `M.M.P-vX` versioning scheme where `M.M.P` (Major.Minor.Patch) represents the respective database version and an optional `-vX` (here, `X` is a monotonically increasing integer) is added if there is any breaking change in the addon image compared to the previous release.

{{< notice type="warning" message="If you update Stash operator to a newer release and the supported addon versions in the newer release has different `-vX` suffix, you have to update the old addons too. Otherwise, backup may not work. In this case, just uninstall the old addons and install the new addons." >}}

## Addon Version Compatibility

Any addon with matching major version with the database version should be able to take backup of that database. For example, MariaDB addon with version `10.x.x-vX` should be able take backup of any MariaDB of `10.x.x` series. However, this might not be true for some versions. In that case, we will have separate addon for that version.

## Documentation Overview

Stash MariaDB documentations are organized as below:

- [How does it works?](/docs/addons/mariadb/overview.md) gives an overview of how backup and restore process for MariaDB database works in Stash.
- [Setup](/docs/addons/mariadb/setup/install.md) shows how to install and uninstall MariaDB addon for Stash.
- **Guides** contains step by step guides to backup and restore different versions of MariaDB databases.
