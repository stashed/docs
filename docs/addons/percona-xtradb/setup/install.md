---
title: Install Percona XtraDB Addon | Stash
description: An guide on how to install Percona XtraDB addon for Stash
menu:
  product_stash_{{ .version }}:
    identifier: stash-percona-xtradb-install
    name: Install
    parent: stash-percona-xtradb-setup
    weight: 10
product_name: stash
menu_name: product_stash_{{ .version }}
section_menu_id: stash-addons
---

# Install Percona XtraDB Addon for Stash

Stash uses `Function-Task` model to backup databases. This `Function-Task` model enables Stash to extend its capability via addons. In order to backup Percona XtraDB databases, you have to install Percona XtraDB addon (`stash-percona-xtradb`) for Stash. This addon creates necessary `Function` and `Task` definitions to backup/restore Percona XtraDB database.

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

Run the following script to install `stash-percona-xtradb` addon as a Helm chart using Helm 3.

```bash
curl -fsSL https://github.com/stashed/catalog/raw/{{< param "info.catalog" >}}/deploy/helm3.sh | bash -s -- --catalog=stash-percona-xtradb
```

</div>
<div class="tab-pane fade" id="helm2" role="tabpanel" aria-labelledby="helm2-tab">

## Using Helm 2

Run the following script to install `stash-percona-xtradb` addon as a Helm chart using Helm 2.

```bash
curl -fsSL https://github.com/stashed/catalog/raw/{{< param "info.catalog" >}}/deploy/helm2.sh | bash -s -- --catalog=stash-percona-xtradb
```

</div>
<div class="tab-pane fade" id="script" role="tabpanel" aria-labelledby="script-tab">

## Using YAML

Run the following script to install `stash-percona-xtradb` addon as Kubernetes YAMLs.

```bash
curl -fsSL https://github.com/stashed/catalog/raw/{{< param "info.catalog" >}}/deploy/script.sh | bash -s -- --catalog=stash-percona-xtradb
```

>The above script uses Helm 3 for rendering the charts to generate the YAMLs.

</div>
</div>

## Verify Installation

After installation is completed, this addon will create `percona-xtradb-backup-*` and `percona-xtradb-restore-*` Functions and Tasks for all supported Percona XtraDB versions. To verify, run the following command:

```bash
$ kubectl get functions.stash.appscode.com
NAME                        AGE
percona-xtradb-backup-5.7   20s
percona-xtradb-restore-5.7  20s
pvc-backup                  7h6m
pvc-restore                 7h6m
update-status               7h6m
```

Also, verify that the `Task` have been created.

```bash
$ kubectl get tasks.stash.appscode.com
NAME                        AGE
percona-xtradb-backup-5.7   2m7s
percona-xtradb-restore-5.7  2m7s
pvc-backup                  7h7m
pvc-restore                 7h7m
```

Now, Stash is ready to backup Percona XtraDB databases.

## Customizing Installation

In order to install `Function` and `Task` only for a specific Percona XtraDB version, use `--version` flag to specify the desired database version.

```bash
curl -fsSL https://github.com/stashed/catalog/raw/{{< param "info.catalog" >}}/deploy/helm3.sh | bash -s -- --catalog=stash-percona-xtradb --version=5.7
```

The flowing flags are available for customizing Percona XtraDB addon installation:

| Flag                    | Usage                                                                                                                                                                                                                                                                                                              |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `--version`             | Specify a specific version of a specific addon to install. Use it along with `--catalog` flag.                                                                                                                                                                                                                     |
| `--docker-registry`     | Specify the docker registry to use to pull respective addon images. Default Value: `stashed`.                                                                                                                                                                                                                      |
| `--image`               | Specify the name of the docker image to use for respective addons.                                                                                                                                                                                                                                                 |
| `--image-tag`           | Specify the tag of the docker image to use for respective addon.                                                                                                                                                                                                                                                   |
| `--xtradb-backup-args`  | Specify optional arguments to pass to `xtrabackup` command during backup. These arguments apply to all Percona XtraDB instances in this cluster. To set arguments for a specific Percona XtraDB database instance, set `xtradbArgs` parameter in `spec.task.params` field of the respective `BackupConfiguration`. |
| `--xtradb-restore-args` | Specify optional arguments to pass to `xtrabackup` command during restore. These arguments apply to all Percona XtraDB instances in this cluster. To set arguments for a specific Percona XtraDB database instance, set `xtradbArgs` parameter in `spec.task.params` field of the respective `RestoreSession`.     |
