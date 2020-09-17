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

Stash supports backup and restore of the following MySQL versions:

{{< versionlist "mysql" "/docs/addons/mysql/guides/%s/mysql.md" >}}

## Documentation Overview

Stash MySQL documentations are organized as below:

- [How does it works?](/docs/addons/mysql/overview.md) gives an overview of how backup and restore process for MySQL database works in Stash.
- [Setup](/docs/addons/mysql/setup/install.md) shows how to install and uninstall MySQL addon for Stash.
- [Guides](/docs/addons/mysql/guides/8.0.14/mysql.md) contains step by step guides to backup and restore different versions of MySQL databases.
