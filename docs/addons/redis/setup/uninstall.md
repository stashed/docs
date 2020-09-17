---
title: Uninstall Redis Addon | Stash
description: A guide on how to uninstall Redis addon for Stash
menu:
  docs_{{ .version }}:
    identifier: stash-redis-uninstall
    name: Uninstall
    parent: stash-redis-setup
    weight: 20
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: stash-addons
---

{{< notice type="warning" message="This is an Enterprise-only feature. Please install [Stash Enterprise Edition](/docs/setup/install/enterprise.md) to try this feature." >}}

# Uninstall Redis addon for Stash

In order to uninstall Redis addon, follow the instruction given below.

<ul class="nav nav-tabs" id="installerTab" role="tablist">
  <li class="nav-item">
    <a class="nav-link active" id="helm3-tab" data-toggle="tab" href="#helm3" role="tab" aria-controls="helm3" aria-selected="true">Helm 3</a>
  </li>
  <li class="nav-item">
    <a class="nav-link" id="helm2-tab" data-toggle="tab" href="#helm2" role="tab" aria-controls="helm2" aria-selected="false">Helm 2</a>
  </li>
  <li class="nav-item">
    <a class="nav-link" id="script-tab" data-toggle="tab" href="#script" role="tab" aria-controls="script" aria-selected="false">YAML</a>
  </li>
</ul>
<div class="tab-content" id="installerTabContent">
  <div class="tab-pane fade show active" id="helm3" role="tabpanel" aria-labelledby="helm3-tab">

## Using Helm 3

Run the following script to uninstall `stash-redis` addon that was installed as a Helm chart using Helm 3.

```bash
curl -fsSL https://github.com/stashed/catalog/raw/{{< param "info.catalog" >}}/deploy/helm3.sh | bash -s -- --uninstall --catalog=stash-redis
```

</div>
<div class="tab-pane fade" id="helm2" role="tabpanel" aria-labelledby="helm2-tab">

## Using Helm 2

Run the following script to uninstall `stash-redis` addon that was installed as a Helm chart using Helm 2.

```bash
curl -fsSL https://github.com/stashed/catalog/raw/{{< param "info.catalog" >}}/deploy/helm2.sh | bash -s -- --uninstall --catalog=stash-redis
```

</div>
<div class="tab-pane fade" id="script" role="tabpanel" aria-labelledby="script-tab">

## Using YAML

Run the following script to uninstall `stash-redis` addon that was installed as Kubernetes YAMLs.

```bash
curl -fsSL https://github.com/stashed/catalog/raw/{{< param "info.catalog" >}}/deploy/script.sh | bash -s -- --uninstall --catalog=stash-redis
```

</div>
</div>

## Customizing Uninstaller

In order to uninstall Redis addon only for a specific database version, use `--version` flag to specify the desired version.

```bash
curl -fsSL https://github.com/stashed/catalog/raw/{{< param "info.catalog" >}}/deploy/helm3.sh | bash -s -- --uninstall --catalog=stash-redis --version=5.0.3
```
