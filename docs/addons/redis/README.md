---
title: Redis Addon Overview | Stash
description: Redis Addon Overview | Stash
menu:
  docs_{{ .version }}:
    identifier: stash-redis-readme
    name: Readme
    parent: stash-redis
    weight: -1
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: stash-addons
url: /docs/{{ .version }}/addons/redis/
aliases:
  - /docs/{{ .version }}/addons/redis/README/
---

# Stash Redis Addon

Stash `{{< param "info.version" >}}` supports extending its functionality through addons. Stash Redis addon enables Stash to backup and restore Redis databases.

This guide will give you an overview of which Redis versions are supported and how the docs are organized.

## Supported Redis Versions

Stash has the following addon versions for Redis:

{{< versionlist "redis">}}

Here, the addon follows `M.M.P` versioning scheme where `M.M.P` (Major.Minor.Patch) represents the respective database version.

## Addon Version Compatibility

Any addon with matching major version with the database version should be able to take backup of that database. For example, Redis addon with version `6.x.x` should be able take backup of any Redis of `6.x.x` series. However, this might not be true for some versions. In that case, we will have separate addon for that version.

## Documentation Overview

Stash Redis documentations are organized as below:

- [How does it work?](/docs/addons/redis/overview/index.md) gives an overview of how backup and restore process for Redis database works in Stash.
- [Helm managed Redis](/docs/addons/redis/helm/index.md) shows how to backup and restore a Helm managed Redis database.
- [Auto-Backup](/docs/addons/redis/auto-backup/index.md) shows how to configure a generic backup template for all the Redis databases of a cluster.
- [Customizing Backup & Restore Process](/docs/addons/redis/customization/index.md) shows how to customize the backup & restore process.
