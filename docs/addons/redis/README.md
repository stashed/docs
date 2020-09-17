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

{{< notice type="warning" message="This is an Enterprise-only feature. Please install [Stash Enterprise Edition](/docs/setup/install/enterprise.md) to try this feature." >}}

# Stash Redis Addon

Stash 0.9.0+ supports extending its functionality through addons. Stash Redis addon enables Stash to backup and restore Redis databases.

This guide will give you an overview of which Redis versions are supported and how the docs are organized.

## Supported Redis Versions

Stash supports backup and restore of the following Redis versions:

{{< versionlist "redis" "/docs/addons/redis/guides/%s/redis.md" >}}

## Documentation Overview

Stash Redis documentations are organized as below:

- [How does it works?](/docs/addons/redis/overview.md) gives an overview of how backup and restore process for Redis database works in Stash.
- [Setup](/docs/addons/redis/setup/install.md) shows how to install and uninstall Redis addon for Stash.
- [Guides](/docs/addons/redis/guides/5.0.3/redis.md) contains step by step guides to backup and restore different versions of Redis databases.
