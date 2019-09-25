---
title: MongoDB Addon Overview | Stash
description: MongoDB Addon Overview | Stash
menu:
  product_stash_{{ .version }}:
    identifier: stash-mongodb-readme
    name: Readme
    parent: stash-mongodb
    weight: -1
product_name: stash
menu_name: product_stash_{{ .version }}
section_menu_id: stash-addons
url: /products/stash/{{ .version }}/addons/mongodb/
aliases:
  - /products/stash/{{ .version }}/addons/mongodb/README/
---

# Stash MongoDB Addon

Stash 0.9.0+ supports extending its functionality through addons. Stash MongoDB addon enables Stash to backup and restore MongoDB databases.

This guide will give you an overview of which MongoDB versions are supported and how the docs are organized.

## Supported MongoDB Versions

Stash supports backup and restore of the following MongoDB versions:

- [3.6](/docs/addons/mongodb/guides/3.6/mongodb.md)

## Documentation Overview

Stash MongoDB documentations are organized as below:

- [How does it works?](/docs/addons/mongodb/overview.md) gives an overview of how backup and restore process for MongoDB database works in Stash.
- [Setup](/docs/addons/mongodb/setup/install.md) shows how to install and uninstall MongoDB addon for Stash.
- [Guides](/docs/addons/mongodb/guides/3.6/mongodb.md) contains step by step guides to backup and restore different versions of MongoDB databases.
