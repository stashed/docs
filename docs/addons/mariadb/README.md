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

## Documentation Overview

Stash MariaDB documentations are organized as below:

- [How does it works?](/docs/addons/mariadb/overview.md) gives an overview of how backup and restore process for MariaDB database works in Stash.
- [Setup](/docs/addons/mariadb/setup/install.md) shows how to install and uninstall MariaDB addon for Stash.
- **Guides** contains step by step guides to backup and restore different versions of MariaDB databases.
