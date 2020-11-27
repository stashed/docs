---
title: Install MariaDB Addon | Stash
description: An guide on how to install MariaDB addon for Stash
menu:
  docs_{{ .version }}:
    identifier: stash-mariadb-install
    name: Install
    parent: stash-mariadb-setup
    weight: 10
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: stash-addons
---

{{< notice type="warning" message="This is an Enterprise-only feature. Please install [Stash Enterprise Edition](/docs/setup/install/enterprise.md) to try this feature." >}}

# Install MariaDB Addon for Stash

Stash uses `Function-Task` model to backup databases. This `Function-Task` model enables Stash to extend its capability via addons. In order to backup MariaDB databases, you have to install MariaDB addon (`stash-mariadb`) for Stash. This addon creates necessary `Function` and `Task` definitions to backup/restore MariaDB database.

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

Run the following script to install `stash-mariadb` addon as a Helm chart using Helm 3.

```bash
curl -fsSL https://github.com/stashed/catalog/raw/{{< param "info.catalog" >}}/deploy/helm3.sh | bash -s -- --catalog=stash-mariadb
```

</div>
<div class="tab-pane fade" id="helm2" role="tabpanel" aria-labelledby="helm2-tab">

## Using Helm 2

Run the following script to install `stash-mariadb` addon as a Helm chart using Helm 2.

```bash
curl -fsSL https://github.com/stashed/catalog/raw/{{< param "info.catalog" >}}/deploy/helm2.sh | bash -s -- --catalog=stash-mariadb
```

</div>
<div class="tab-pane fade" id="script" role="tabpanel" aria-labelledby="script-tab">

## Using YAML

Run the following script to install `stash-mariadb` addon as Kubernetes YAMLs.

```bash
curl -fsSL https://github.com/stashed/catalog/raw/{{< param "info.catalog" >}}/deploy/script.sh | bash -s -- --catalog=stash-mariadb
```

>The above script uses Helm 3 for rendering the charts to generate the YAMLs.

</div>
</div>

## Verify Installation

After installation is completed, this addon will create `mariadb-backup-*` and `mariadb-restore-*` Functions and Tasks for all supported MariaDB versions. To verify, run the following command:

```bash
$ kubectl get functions.stash.appscode.com
NAME                     AGE
mariadb-backup-10.5.8    25s
mariadb-restore-10.5.8   25s
pvc-backup               3m14s
pvc-restore              3m14s
update-status            3m14s
```

Also, verify that the `Task` have been created.

```bash
$ kubectl get tasks.stash.appscode.com
NAME                     AGE
mariadb-backup-10.5.8    101s
mariadb-restore-10.5.8   101s
pvc-backup               4m30s
pvc-restore              4m30s
```

Now, Stash is ready to backup MariaDB databases.

## Customizing Installation

In order to install `Function` and `Task` only for a specific MariaDB version, use `--version` flag to specify the desired database version.

```bash
curl -fsSL https://github.com/stashed/catalog/raw/{{< param "info.catalog" >}}/deploy/helm3.sh | bash -s -- --catalog=stash-mariadb --version=10.5.8
```

The flowing flags are available for customizing MariaDB addon installation:

| Flag                | Usage                                                                                                                                                                                                                                                                                         |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `--version`         | Specify a specific version of a specific addon to install. Use it along with `--catalog` flag.                                                                                                                                                                                                |
| `--docker-registry` | Specify the docker registry to use to pull respective addon images. Default Value: `stashed`.                                                                                                                                                                                                 |
| `--image`           | Specify the name of the docker image to use for respective addons.                                                                                                                                                                                                                            |
| `--image-tag`       | Specify the tag of the docker image to use for respective addon.                                                                                                                                                                                                                              |
| `--backup-args`     | Specify optional arguments to pass to `mysqldump` command during backup. These arguments apply to all MariaDB instances in this cluster. To set arguments for a specific MariaDB database instance, set `args` parameter in `spec.task.params` field of the respective `BackupConfiguration`. |
| `--restore-args`    | Specify optional arguments to pass to `mysql` command during restore. These arguments apply to all MariaDB instances in this cluster. To set arguments for a specific MariaDB database instance, set `args` parameter in `spec.task.params` field of the respective `RestoreSession`.         |
