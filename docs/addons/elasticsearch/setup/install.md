---
title: Install Elasticsearch Addon | Stash
description: An guide on how to install Elasticsearch addon for Stash
menu:
  docs_{{ .version }}:
    identifier: stash-elasticsearch-install
    name: Install
    parent: stash-elasticsearch-setup
    weight: 10
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: stash-addons
---

# Install Elasticsearch Addon for Stash

Stash uses `Function-Task` model to backup databases. This `Function-Task` model enables Stash to extend its capability via addons. In order to backup Elasticsearch databases, you have to install Elasticsearch addon (`stash-elasticsearch`) for Stash. This addon creates necessary `Function` and `Task` definitions to backup/restore Elasticsearch database.

You can install the addon either as a helm chart or you can create only the YAMLs of the respective resources.

<ul class="nav nav-tabs" id="installerTab" role="tablist">
  <li class="nav-item">
    <a class="nav-link active" id="helm3-tab" data-toggle="tab" href="#helm3" role="tab" aria-controls="helm3" aria-selected="true">Helm 3 (Recommended)</a>
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

Run the following script to install `stash-elasticsearch` addon as a Helm chart using Helm 3.

```bash
curl -fsSL https://github.com/stashed/catalog/raw/{{< param "info.catalog" >}}/deploy/helm3.sh | bash -s -- --catalog=stash-elasticsearch
```

</div>
<div class="tab-pane fade" id="helm2" role="tabpanel" aria-labelledby="helm2-tab">

## Using Helm 2

Run the following script to install `stash-elasticsearch` addon as a Helm chart using Helm 2.

```bash
curl -fsSL https://github.com/stashed/catalog/raw/{{< param "info.catalog" >}}/deploy/helm2.sh | bash -s -- --catalog=stash-elasticsearch
```

</div>
<div class="tab-pane fade" id="script" role="tabpanel" aria-labelledby="script-tab">

## Using YAML

Run the following script to install `stash-elasticsearch` addon as Kubernetes YAMLs.

```bash
curl -fsSL https://github.com/stashed/catalog/raw/{{< param "info.catalog" >}}/deploy/script.sh | bash -s -- --catalog=stash-elasticsearch
```

>The above script uses Helm 3 for rendering the charts to generate the YAMLs.

</div>
</div>

## Verify Installation

After installation is completed, this addon will create `elasticsearch-backup-*` and `elasticsearch-restore-*` Functions and Tasks for all supported Elasticsearch versions. To verify, run the following command:

```bash
$ kubectl get functions.stash.appscode.com
NAME                        AGE
elasticsearch-backup-7.2    20s
elasticsearch-backup-6.8    20s
elasticsearch-backup-6.5    19s
elasticsearch-backup-6.4    20s
elasticsearch-backup-6.3    20s
elasticsearch-backup-6.2    20s
elasticsearch-backup-5.6    20s
elasticsearch-restore-7.2   20s
elasticsearch-restore-6.8   20s
elasticsearch-restore-6.5   19s
elasticsearch-restore-6.4   20s
elasticsearch-restore-6.3   20s
elasticsearch-restore-6.2   20s
elasticsearch-restore-5.6   20s
pvc-backup                  7h6m
pvc-restore                 7h6m
update-status               7h6m
```

Also, verify that the `Task` have been created.

```bash
$ kubectl get tasks.stash.appscode.com
NAME                        AGE
elasticsearch-backup-7.2    2m7s
elasticsearch-backup-6.8    2m7s
elasticsearch-backup-6.5    2m6s
elasticsearch-backup-6.4    2m7s
elasticsearch-backup-6.3    2m7s
elasticsearch-backup-6.2    2m7s
elasticsearch-backup-5.6    2m7s
elasticsearch-restore-7.2   2m7s
elasticsearch-restore-6.8   2m7s
elasticsearch-restore-6.5   2m6s
elasticsearch-restore-6.4   2m7s
elasticsearch-restore-6.3   2m7s
elasticsearch-restore-6.2   2m7s
elasticsearch-restore-5.6   2m7s
pvc-backup                  7h7m
pvc-restore                 7h7m
```

Now, Stash is ready to backup Elasticsearch databases.

## Customizing Installation

In order to install `Function` and `Task` only for a specific Elasticsearch version, use `--version` flag to specify the desired database version.

```bash
curl -fsSL https://github.com/stashed/catalog/raw/{{< param "info.catalog" >}}/deploy/helm3.sh | bash -s -- --catalog=stash-elasticsearch --version=6.5
```

The flowing flags are available for customizing Elasticsearch addon installation:

| Flag                | Usage                                                                                                                                                                                                                                                                                                             |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `--version`         | Specify a specific version of a specific addon to install. Use it along with `--catalog` flag.                                                                                                                                                                                                                    |
| `--docker-registry` | Specify the docker registry to use to pull respective addon images. Default Value: `stashed`.                                                                                                                                                                                                                     |
| `--image`           | Specify the name of the docker image to use for respective addons.                                                                                                                                                                                                                                                |
| `--image-tag`       | Specify the tag of the docker image to use for respective addon.                                                                                                                                                                                                                                                  |
| `--es-backup-args`  | Specify optional arguments to pass to `multielaticdump` command during backup. These arguments apply to all Elasticsearch instances in this cluster. To set arguments for a specific Elasticsearch database instance, set `esArgs` parameter in `spec.task.params` field of the respective `BackupConfiguration`. |
| `--es-restore-args` | Specify optional arguments to pass to `multielastic` command during restore. These arguments apply to all Elasticsearch instances in this cluster. To set arguments for a specific Elasticsearch database instance, set `esArgs` parameter in `spec.task.params` field of the respective `RestoreSession`.        |
