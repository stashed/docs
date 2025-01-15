---
title: Upgrading Repository Format Version | Stash
menu:
  docs_{{ .version }}:
    identifier: how-to-upgrade
    name: How to upgrade?
    parent: repo-upgrade
    weight: 10
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: guides
---

# Upgrading Repository Format Version

Repositories created with older versions of `Stash` use an older repository format version, which needs to be upgraded to unlock new features. This upgrade process is optional but required if you want to take advantage of the latest enhancements. Keep in mind that upgrading the repository format will increase the up-to-date Stash version needed to access it. For example, repositories upgraded to format version 2 can only be read by `Stash v2025.1.9` or later.

> Upgrading repository format version will take some time depending on the repository size. Repository issues must be corrected before upgrading. It is recommended to contact with Stash team before upgrading repository.

## How to upgrade

Upgrading to repository version 2 involves the following steps:

1. **Pause the Corresponding Backup:** Before upgrading, pause the backup associated with the repository. You can do this using the [pause backup](/docs/guides/cli/kubectl-plugin/index.md#pause-backup) command provided by the Stash kubectl plugin.
2. **Run the Migration Command:** Next, execute the [migrate](/docs/guides/cli/kubectl-plugin/index.md#migrate-repository) command provided by the Stash kubectl plugin. This command will first check the repository’s integrity and then upgrade its format to version 2. Note that if any issues are found during the integrity check, they must be resolved before the migration can proceed.
3. **Run the Prune Command:** After a successful migration, use the [prune](/docs/guides/cli/kubectl-plugin/index.md#prune) command provided by Stash kubectl plugin to compress the repository metadata. If you want to limit the amount of data rewritten in a single operation, use the `--max-repack-size` flag with the `prune` command.
4. **Resume the Corresponding Backup:** Now resume the backup associated with the repository. You can do this using the [resume backup](/docs/guides/cli/kubectl-plugin/index.md#resume-backup) command provided by the Stash kubectl plugin.

Keep in mind that the contents of files already stored in the repository will not be rewritten during the upgrade. Only data from new backups will be compressed. Over time, more and more of the repository will be automatically compressed as new backups are added.
