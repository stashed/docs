---
title: NATS Addon Overview | Stash
description: NATS Addon Overview | Stash
menu:
  docs_{{ .version }}:
    identifier: stash-nats-readme
    name: Readme
    parent: stash-nats
    weight: -1
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: stash-addons
url: /docs/{{ .version }}/addons/nats/
aliases:
  - /docs/{{ .version }}/addons/nats/README/
---

{{< notice type="warning" message="This is an Enterprise-only feature. Please install [Stash Enterprise Edition](/docs/setup/install/enterprise.md) to try this feature." >}}

# Stash NATS Addon

Stash `{{< param "info.version" >}}` supports extending its functionality through addons. Stash NATS addon enables Stash to backup and restore NATS streams.

This guide will give you an overview of which NATS versions are supported and how the docs are organized.

## Supported NATS Versions

Stash has the following addon versions for NATS:

{{< versionlist "nats">}}

Here, the addon follows `M.M.P` versioning scheme where `M.M.P` (Major.Minor.Patch) represents the respective NATS version.

## Addon Version Compatibility

This addon with matching major version with the NATS version should be able to take backup of that NATS streams. For example, NATS addon with version `2.x.x` should be able take backup of any NATS of `2.x.x` series. However, this might not be true for some versions. In that case, we will have separate addon for that version.

## Documentation Overview

Stash NATS documentations are organized as below:

- [How does it works?](/docs/addons/nats/overview/index.md) gives an overview of how backup and restore process for NATS works in Stash.
- [Helm managed Redis](/docs/addons/nats/helm/index.md) shows how to backup and restore a Helm managed NATS.
- [Auto-Backup](/docs/addons/nats/auto-backup/index.md) shows how to configure a generic backup template for all the NATS streams of a cluster.
- [Customizing Backup & Restore Process](/docs/addons/nats/customization/index.md) shows how to customize the backup & restore process.