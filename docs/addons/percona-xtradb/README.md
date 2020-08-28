---
title: Percona XtraDB Addon Overview | Stash
description: Percona XtraDB Addon Overview | Stash
menu:
  product_stash_{{ .version }}:
    identifier: stash-percona-xtradb-readme
    name: Readme
    parent: stash-percona-xtradb
    weight: -1
product_name: stash
menu_name: product_stash_{{ .version }}
section_menu_id: stash-addons
url: /products/stash/{{ .version }}/addons/percona-xtradb/
aliases:
  - /products/stash/{{ .version }}/addons/percona-xtradb/README/
---

# Stash Percona XtraDB Addon

Stash 0.9.0+ supports extending its functionality through addons. Stash Percona XtraDB addon enables Stash to backup and restore Percona XtraDB databases.

This guide will give you an overview of which Percona XtraDB versions are supported and how the docs are organized.

## Supported Percona XtraDB Versions

Stash supports backup and restore of the following Percona XtraDB versions:

{{< versionlist "percona-xtradb" "/docs/addons/percona-xtradb/guides/%s/clustered.md" >}}

## Documentation Overview

Stash Percona XtraDB documentations are organized as below:

- [How does it works?](/docs/addons/percona-xtradb/overview.md) gives an overview of how backup and restore process for Percona XtraDB database works in Stash.
- [Setup](/docs/addons/percona-xtradb/setup/install.md) shows how to install and uninstall Percona XtraDB addon for Stash.
- [Guides](/docs/addons/percona-xtradb/guides/5.7/clustered.md) contains step by step guides to backup and restore different versions of Percona XtraDB databases.
