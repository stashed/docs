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

Stash supports backup and restore of the following PostgreSQL versions:

{{< versionlist "postgres" "/docs/addons/postgres/guides/%s/standalone.md" >}}

## Documentation Overview

Stash PostgreSQL documentations are organized as below:

- [How does it works?](/docs/addons/postgres/overview.md) gives an overview of how backup and restore process for PostgreSQL database works in Stash.
- [Setup](/docs/addons/postgres/setup/install.md) shows how to install and uninstall PostgreSQL addon for Stash.
- [Guides](/docs/addons/postgres/guides/11.2/standalone.md) contains step by step guides to backup and restore different versions of PostgreSQL databases.
