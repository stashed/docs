---
title: Install MySQL Addon | Stash
description: An guide on how to install MySQL addon for Stash
menu:
  product_stash_{{ .version }}:
    identifier: stash-mysql-install
    name: Install
    parent: stash-mysql-setup
    weight: 10
product_name: stash
menu_name: product_stash_{{ .version }}
section_menu_id: stash-addons
---

# Install MySQL Addon for Stash

Stash uses `Function-Task` model to backup databases. This `Function-Task` model enables Stash to extend its capability via addons. In order to backup MySQL databases, you have to install MySQL addon (`stash-mysql`) for Stash. This addon creates necessary `Function` and `Task` definitions to backup/restore MySQL database.

You can install the addon either as a helm chart or you can create only the YAMLs of the respective resources.

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

## Install only YAMLs

Run the following script to install `stash-mysql` addon as Kubernetes YAMLs.

```console
curl -fsSL https://github.com/stashed/catalog/raw/master/deploy/script.sh | bash -s -- --catalog=stash-mysql
```

</div>
<div class="tab-pane fade" id="helm" role="tabpanel" aria-labelledby="helm-tab">

## Install as chart release

Run the following script to install `stash-mysql` addon as a Helm chart.

```console
curl -fsSL https://github.com/stashed/catalog/raw/master/deploy/chart.sh | bash -s -- --catalog=stash-mysql
```

</div>
</div>

## Verify Installation

After installation is completed, this addon will create `mysql-backup-*` and `mysql-restore-*` Functions and Tasks for all supported MySQL versions. To verify, run the following command:

```console
$ kubectl get functions.stash.appscode.com
NAME                    AGE
mysql-backup-8.0.14     20s
mysql-backup-5.7        20s
pvc-backup              7h6m
pvc-restore             7h6m
update-status           7h6m
```

Also, verify that the `Task` have been created.

```console
$ kubectl get tasks.stash.appscode.com
NAME                    AGE
mysql-backup-8.0.14     2m7s
mysql-backup-5.7        2m7s
pvc-backup              7h7m
pvc-restore             7h7m
```

Now, Stash is ready to backup MySQL databases.

## Customizing Installation

In order to install `Function` and `Task` only for a specific MySQL version, use `--version` flag to specify the desired database version.

```console
curl -fsSL https://github.com/stashed/catalog/raw/master/deploy/chart.sh | bash -s -- --catalog=stash-mysql --version=8.0.14
```

The flowing flags are available for customizing MySQL addon installation:

| Flag                | Usage                                                                                                                                                                                                                                                                                   |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `--version`         | Specify a specific version of a specific addon to install. Use it along with `--catalog` flag.                                                                                                                                                                                          |
| `--docker-registry` | Specify the docker registry to use to pull respective addon images. Default Value: `stashed`.                                                                                                                                                                                           |
| `--image`           | Specify the name of the docker image to use for respective addons.                                                                                                                                                                                                                      |
| `--image-tag`       | Specify the tag of the docker image to use for respective addon.                                                                                                                                                                                                                        |
| `--my-backup-args`  | Specify optional arguments to pass to `mysqldump` command during backup. This args applies to all MySQL instances in this cluster. To set arguments for a specific MySQL database instance, set `myArgs` parameter in `spec.task.params` field of the respective `BackupConfiguration`. |
| `--my-restore-args` | Specify optional arguments to pass to `mysql` command during restore. This args applies to all MySQL instances in this cluster. To set arguments for a specific MySQL database instance, set `myArgs` parameter in `spec.task.params` field of the respective `RestoreSession`.         |
