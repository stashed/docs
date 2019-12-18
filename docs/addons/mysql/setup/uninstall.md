---
title: Uninstall MySQL Addon | Stash
description: An guide on how to uninstall MySQL addon for Stash
menu:
  docs_{{ .version }}:
    identifier: stash-mysql-uninstall
    name: Uninstall
    parent: stash-mysql-setup
    weight: 20
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: stash-addons
---

# Uninstall MySQL addon for Stash

In order to uninstall MySQL addon, follow the instruction given below.

<ul class="nav nav-tabs" id="installerTab" role="tablist">
  <li class="nav-item">
    <a class="nav-link active" id="script-tab" data-toggle="tab" href="#script" role="tab" aria-controls="script" aria-selected="true">Script</a>
  </li>
  <li class="nav-item">
    <a class="nav-link" id="helm-tab" data-toggle="tab" href="#helm" role="tab" aria-controls="helm" aria-selected="false">Helm</a>
  </li>
</ul>
<div class="tab-content" id="installerTabContent">
  <div class="tab-pane fade show active" id="script" role="tabpanel" aria-labelledby="script-tab">

## Uninstall `stash-mysql` addon YAMLs

Run the following script to uninstall `stash-mysql` addon that was installed as Kubernetes YAMLs.

```console
curl -fsSL https://github.com/stashed/catalog/raw/{{< param "info.catalog" >}}/deploy/script.sh | bash -s -- --uninstall --catalog=stash-mysql
```

</div>
<div class="tab-pane fade" id="helm" role="tabpanel" aria-labelledby="helm-tab">

## Uninstall `stash-mysql` chart

Run the following script to uninstall `stash-mysql` addon that was installed as a Helm chart.

```console
curl -fsSL https://github.com/stashed/catalog/raw/{{< param "info.catalog" >}}/deploy/chart.sh | bash -s -- --uninstall --catalog=stash-mysql
```

</div>
</div>

## Customizing Uninstaller

In order to uninstall MySQL addon only for a specific database version, use `--version` flag to specify the desired version.

```console
curl -fsSL https://github.com/stashed/catalog/raw/{{< param "info.catalog" >}}/deploy/chart.sh | bash -s -- --uninstall --catalog=stash-mysql --version=8.0.14
```
