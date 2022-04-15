---
title: Sending Backup Notification to Slack | Stash
menu:
  docs_{{ .version }}:
    identifier: hoooks-slack-notification
    name: Slack Notification
    parent: hooks
    weight: 25
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: guides
---

# Sending Backup Notification to Slack Channel

In this guide, we are going to show you how to send backup notification to a Slack channel. Here, we are going to use [Slack Incoming Webook](https://api.slack.com/messaging/webhooks) to send the notification.

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [kind](https://kind.sigs.k8s.io/docs/user/quick-start/).
- Install Stash Enterprise in your cluster following the steps [here](/docs/setup/install/enterprise.md).
- If you haven't read about how hooks work in Stash, please check it from [here](/docs/guides/hooks/overview.md).

You should be familiar with the following `Stash` concepts:

- [BackupBatch](/docs/concepts/crds/backupbatch.md)
- [BackupSession](/docs/concepts/crds/backupsession.md)
- [Repository](/docs/concepts/crds/repository.md)
- [Function](/docs/concepts/crds/function.md)
- [Task](/docs/concepts/crds/task.md)

To keep everything isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```bash
$ kubectl create ns demo
namespace/demo created
```

## Configure Slack Incoming Webhook

## Prepare Application

## Backup

## Restore

## Cleanup
