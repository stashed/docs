---
title: Etcd Addon Overview | Stash
description: Etcd Addon Overview | Stash
menu:
  docs_{{ .version }}:
    identifier: stash-etcd-readme
    name: Readme
    parent: stash-etcd
    weight: -1
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: stash-addons
url: /docs/{{ .version }}/addons/etcd/
aliases:
  - /docs/{{ .version }}/addons/etcd/README/
---

{{< notice type="warning" message="This is an Enterprise-only feature. Please install [Stash Enterprise Edition](/docs/setup/install/enterprise.md) to try this feature." >}}

# Stash Etcd Addon

Stash `{{< param "info.version" >}}` supports extending its functionality through its addons. Stash Etcd addon enables Stash to backup and restore Etcd databases.

This guide will give you an overview of which Etcd versions are supported and how the docs are organized.

## Supported Etcd Versions

Stash has the following addon versions for Etcd:

{{< versionlist "etcd">}}

Here, the addon follows `M.M.P` versioning scheme where `M.M.P` (Major.Minor.Patch) represents the respective database version.

## Addon Version Compatibility

Any addon with matching major version with the database version should be able to take backup of that database. For example, Etcd addon with version `3.x.x` should be able take backup of any Etcd of `3.x.x` series. However, this might not be true for some versions. In that case, we will have a separate addon for that version.

## Documentation Overview

Stash Etcd documentations are organized as below:

- [How does it works?](/docs/addons/etcd/overview/index.md) gives an overview of how backup and restore process for etcd database works in Stash.
- [Backup & Restore Etcd Cluster](/docs/addons/etcd/etcd/index.md) shows how to backup and restore a Etcd database.
- [Backup & Restore TLS Secured Etcd Cluster](/docs/addons/etcd/tls-secured-etcd/index.md) shows how to configure a backup and restore process for TLS secured Etcd database.
- [Customizing Etcd Backup & Restore Process](/docs/addons/etcd/customization/index.md) shows how to customize the backup & restore process.
