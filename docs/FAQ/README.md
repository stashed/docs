---
title: FAQ | Stash  
menu:
  docs_{{ .version }}:
    identifier: faq-readme
    name: README
    parent: faq
    weight: -1
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: faq
url: /docs/{{ .version }}/faq/
aliases:
  - /docs/{{ .version }}/faq/README/
---

# Frequently Asked Questions

## How to temporarily disable backup?

Run the following command to disable a backup temporarily,

```bash
❯ kubectl patch backupconfiguration -n <namespace> <backupconfiguration name> --type="merge" --patch='{"spec": {"paused": true}}'
```

Or you can use the Stash `kubectl` plugin to pause a backup, 

```bash
❯ kubectl stash pause backup -n demo --backupconfig=<backupconfiguration name>
```

Similarly you can also resume a disabled/paused backup. Run the following command to resume a backup, 

```bash
kubectl patch backupconfiguration -n <namespace> <backupconfiguration name> --type="merge" --patch='{"spec": {"paused": false}}'
```

Or you can use the Stash `kubectl` plugin to resume a backup, 

```bash
❯ kubectl stash resume backup -n demo --backupconfig=<backupconfiguration name>
```

## When `retentionPolicy` is applied?

`retentionPolicy` specifies the policy to follow for cleaning old snapshots. The `rententionPolicy` is applied when the backend conatains any snapshot that falls outside the scope of the policy. If you use the policy `keep-last-5`, Stash will remove any snapshot that is older than the most recent 5 snapshots.

## Do I need to delete the init containers after recovery?

You don't need to delete the init containers after recovery.  If your workload restarts with the `stash-init` init-container for any reason, the init-container will skip running restore process if there is no pending RestoreSession for this workload. If you delete the RestoreSession, Stash will remove the `init-container` automatically. Beware that it will cause your workload to restart.

## Need More Help?

To speak with us, please leave a message on [our website](https://appscode.com/contact/).

To join public discussions with the Stash community, join us in the [AppsCode Slack team](https://appscode.slack.com/messages/C8NCX6N23/details/) channel `#stash`. To sign up, use our [Slack inviter](https://slack.appscode.com/).
