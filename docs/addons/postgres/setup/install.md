---
title: Install stash-postgres Catalog | Stash
description: An guide on how to install stash-postgres catalog
menu:
  product_stash_{{ .version }}:
    identifier: stash-postgres-install
    name: Install
    parent: stash-postgres-setup
    weight: 10
product_name: stash
menu_name: product_stash_{{ .version }}
section_menu_id: stash-addons
---

# Install PostgreSQL Addon for Stash

Stash uses `Function-Task` model to backup databases. This `Function-Task` model enables Stash to extends its capability through addon mechanism. In order to backup PostgreSQL database, you have to install PostgreSQL addon (`stash-postgres`) for Stash. This addon creates necessary `Function` and `Task` definitions to backup/restore PostgreSQL database.

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

Run the following script to install `stash-postgres` addon as Kubernetes YAMLs.

```console
curl -fsSL https://github.com/stashed/catalog/raw/master/deploy/script.sh | bash -s -- --catalog=stash-postgres
```

</div>
<div class="tab-pane fade" id="helm" role="tabpanel" aria-labelledby="helm-tab">

## Install as chart release

Run the following script to install `stash-postgres` addon as a Helm chart.

```console
curl -fsSL https://github.com/stashed/catalog/raw/master/deploy/chart.sh | bash -s -- --catalog=stash-postgres
```

</div>
</div>

## Verify Installation

After installation is completed, this addon will create `postgres-backup-*` and `postgres-restore-*` Functions and Tasks for all supported PostgreSQL versions. To verify, run the following command:

```console
$ kubectl get functions.stash.appscode.com
NAME                    AGE
postgres-backup-10.2    20s
postgres-backup-10.6    20s
postgres-backup-11.1    19s
postgres-backup-9.6     20s
postgres-restore-10.2   20s
postgres-restore-10.6   20s
postgres-restore-11.1   19s
postgres-restore-9.6    20s
postgres-backup-11.2    19s
postgres-restore-11.2   19s
pvc-backup              7h6m
pvc-restore             7h6m
update-status           7h6m
```

Also, verify that the `Task` have been created.

```console
$ kubectl get tasks.stash.appscode.com
NAME                    AGE
postgres-backup-10.2    2m7s
postgres-backup-10.6    2m7s
postgres-backup-11.1    2m6s
postgres-backup-9.6     2m7s
postgres-restore-10.2   2m7s
postgres-restore-10.6   2m7s
postgres-restore-11.1   2m6s
postgres-restore-9.6    m7s
postgres-backup-11.2    2m6s
postgres-restore-11.2   2m6s
pvc-backup              7h7m
pvc-restore             7h7m
```

Now, Stash is ready to backup PostgreSQL database.

## Customizing Installation

In order to install `Function` and `Task` only for a specific PostgreSQL version, use `--version` flag to specify the desired database version.

```console
curl -fsSL https://github.com/stashed/catalog/raw/master/deploy/chart.sh | bash -s -- --catalog=stash-postgres --version=11.2
```

The flowing flags are available for customizing PostgreSQL addon installation:

| Flag                | Usage                                                                                          |
| ------------------- | ---------------------------------------------------------------------------------------------- |
| `--version`         | Specify a specific version of a specific addon to install. Use it along with `--catalog` flag. |
| `--docker-registry` | Specify the docker registry to use to pull respective addon images. Default Value: `stashed`.  |
| `--image`           | Specify the name of the docker image to use for respective addons.                             |
| `--image-tag`       | Specify the tag of the docker image to use for respective addon.                               |
| `--pg-backup-args`  | Specify optional arguments to pass to `pgdump` command during backup.                          |
| `--pg-restore-args` | Specify optional arguments to pass to `psql` command during restore.                           |
