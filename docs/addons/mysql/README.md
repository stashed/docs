---
title: MySQL Addon Overview | Stash
description: MySQL Addon Overview | Stash
menu:
  product_stash_{{ .version }}:
    identifier: stash-mysql-readme
    name: Readme
    parent: stash-mysql
    weight: -1
product_name: stash
menu_name: product_stash_{{ .version }}
section_menu_id: stash-addons
url: /products/stash/{{ .version }}/addons/mysql/
aliases:
  - /products/stash/{{ .version }}/addons/mysql/README/
---

# Stash MySQL Addon

Stash 0.9.0+ supports extending its functionality through addons. Stash MySQL addon enables Stash to backup and restore MySQL databases.

This guide will give you an overview of which MySQL versions are supported and how the docs are organized.

## Supported MySQL Versions

Stash supports backup and restore of the following MySQL versions:

- [5.7.25](/docs/addons/mysql/guides/5.7.25/mysql.md)
- [8.0.3](/docs/addons/mysql/guides/8.0.3/mysql.md)
- [8.0.14](/docs/addons/mysql/guides/8.0.14/mysql.md)

## Documentation Overview

Stash MySQL documentations are organized as below:

- [How does it works?](/docs/addons/mysql/overview.md) gives an overview of how backup and restore process for MySQL database works in Stash.
- [Setup](/docs/addons/mysql/setup/install.md) shows how to install and uninstall MySQL addon for Stash.
- [Guides](/docs/addons/mysql/guides/8.0.14/mysql.md) contains step by step guides to backup and restore different versions of MySQL databases.
