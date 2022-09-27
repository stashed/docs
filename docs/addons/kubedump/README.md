---
title: KubeDump Backup Addon Overview | Stash
description: KubeDump Backup Addon Overview | Stash
menu:
  docs_{{ .version }}:
    identifier: stash-kubedump-readme
    name: Readme
    parent: stash-kubedump
    weight: -1
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: stash-addons
url: /docs/{{ .version }}/addons/kubedump/
aliases:
  - /docs/{{ .version }}/addons/kubedump/README/
---

{{< notice type="warning" message="This is an Enterprise-only feature. Please install [Stash Enterprise Edition](/docs/setup/install/enterprise/index.md) to try this feature." >}}

# Stash KubeDump Backup Addon

Stash `{{< param "info.version" >}}` supports extending its functionality through addons. Stash KubeDump backup addon enables Stash to backup and restore Kubernetes manifests. You can backup the manifest of your entire cluster, a particular namespace, or a particular application.

This guide will give you an overview of how the docs are organized.

## Documentation Overview

Stash KubeDump backup documentations are organized as below:

- [How does it work?](/docs/addons/kubedump/overview/index.md) gives an overview of how manifest backup works in Stash.
- [Cluster Backup](/docs/addons/kubedump/cluster/index.md) shows how to backup all the cluster manifests.
- [Namespace Backup](/docs/addons/kubedump/namespace/index.md) shows how to backup the manifests only for a particular namespace.
- [Application Backup](/docs/addons/kubedump/application/index.md) shows how to manifests of a particular application.
- [Customizing Backup](/docs/addons/kubedump/customization/index.md) shows how to customize the backup process.
