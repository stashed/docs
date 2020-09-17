---
title: MongoDB Addon Overview | Stash
description: MongoDB Addon Overview | Stash
menu:
  docs_{{ .version }}:
    identifier: stash-mongodb-readme
    name: Readme
    parent: stash-mongodb
    weight: -1
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: stash-addons
url: /docs/{{ .version }}/addons/mongodb/
aliases:
  - /docs/{{ .version }}/addons/mongodb/README/
---

{{< notice type="warning" message="This is an Enterprise-only feature. Please install [Stash Enterprise Edition](/docs/setup/install/enterprise.md) to try this feature." >}}

# Stash MongoDB Addon

Stash 0.9.0+ supports extending its functionality through addons. Stash MongoDB addon enables Stash to backup and restore MongoDB databases.

This guide will give you an overview of which MongoDB versions are supported and how the docs are organized.

## Supported MongoDB Versions

Stash supports backup and restore of the following MongoDB versions:

{{< versionlist "mongodb" "/docs/addons/mongodb/guides/%s/mongodb.md" >}}

>Version **M.M** actually represents the latest patch of **M.M.P** series. For example, version **4.1** actually represents **4.1.13** as it is the latest supported patch of **4.1** series. Now, if **4.1.14** is released then **4.1** will represents **4.1.14**.

## Documentation Overview

Stash MongoDB documentations are organized as below:

- [How does it works?](/docs/addons/mongodb/overview.md) gives an overview of how backup and restore process for MongoDB database works in Stash.
- [Setup](/docs/addons/mongodb/setup/install.md) shows how to install and uninstall MongoDB addon for Stash.
- [Guides](/docs/addons/mongodb/guides/3.6/mongodb.md) contains step by step guides to backup and restore different versions of MongoDB databases.
