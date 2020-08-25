---
title: Upgrade | Stash
description: Stash Upgrade
menu:
  docs_{{ .version }}:
    identifier: upgrade-stash
    name: Upgrade
    parent: setup
    weight: 30
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: setup
---

# Upgrading Stash

This guide will show you how to upgrade Stash operator. Here, we are going to show how to update the license and how to upgrade between two Stash versions.

## Updating License

Stash support updating license without requiring any re-installation or restart. Stash creates a Secret named `<helm release name>-license` with the license file. You just need to update the Secret. The changes will propagate automatically to the operator and it will use the updated license going forward.

Follow the below instructions to update the license:

- Get a new license and save it into a file.
- Encode the license file content in `base64` format. Make sure that the encoded content is not wrapped. Here is a Linux instruction to encode the license file into `base64` format.

```bash
$ cat /path/to/the/license.txt | base64 -w 0
```

- Now, find out the license secret.

```bash
$ kubectl get Secret -n kube-system | grep license
stash-license        Opaque           1      2m43s
```

- Finally, update the value of `key.txt` in the `data` section of the Secret with your new `base64` encoded license.

```yaml
$ kubectl edit secret/stash-license -n kube-system
apiVersion: v1
data:
  key.txt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURTRENDQWpDZ0F3SUJBZ0lJUFRHNDh2SlhZSVV3RFFZSktvWklodmNOQVFFTEJRQXdKVEVXTUJRR0ExVUUKQ2hNTlFYQndjME52WkdVZ1NXNWpMakVMTUFrR0ExVUVBeE1DWTJFd0hoY05NakF3T0RBNU1URXhPRE15V2hjTgpNakV3T0RJeE1EazBOakl6V2pCSk1SZ3dGZ1lEVlFRS0V3OXpkR0Z6YUMxamIyMXRkVzVwZEhreExUQXJCZ05WCkJBTVRKREE1WWpsaE5qY3hMVFExTXpFdE5EZzNOeTFpTlRreExUUm1NMll5T1RZeFpXVm1aRENDQVNJd0RRWUoKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTTJFSEtXM2hwYUVkZ3ZJZ0prcDJJVVV4b2dWK3pyYgp6dCsvTGZWc3lkdC9KNGd1R0RNTExtT016WHFXWjViK21sOUZ4ZGdaai9GVnp6emxwK1ZCR2V6VEZpWWlUMnlQCkdRSk8xRnRDdmVkeGJUWXV4Y2p1T2dtRyt3WEwraCt5RFpHckNycUpEN3IxblFwbzZMVml5alZhcXRFU1hIYjkKT1VMTXNDazBaSTJlc2h1Q3FkWXBKSEt0c1dlUmlFL0ZHamJ5anUyMnd0VE80eDNtb2JReE9zSUsrSlh2Y25YcApaRWpIaU1OVFJEN3BlNWl0cnRYR1p5c0EyN3lNVVhmRWdrQUUybU1BVTlJN0R4d1AyUzc0NHdxTEtUbUNRcWsxCnV3bThxNWZHZEdFbXdvem5zM0hWb05raGw0Ukp3TnpyU3R0SFBvT0ZSUDNNT3JEN21uSEozenNDQXdFQUFhTlkKTUZZd0RnWURWUjBQQVFIL0JBUURBZ1dnTUJNR0ExVWRKUVFNTUFvR0NDc0dBUVVGQndNQ01DOEdBMVVkRVFRbwpNQ2FDSkRBNVlqbGhOamN4TFRRMU16RXRORGczTnkxaU5Ua3hMVFJtTTJZeU9UWXhaV1ZtWkRBTkJna3Foa2lHCjl3MEJBUXNGQUFPQ0FRRUFIb1RBRk03OExGbmlpbDNJZlpxb1l4N0l4M0YyWVZjU0dMQ2hYaG13T0Uyd1hIeFUKSndlMm9FcjBOeEtBOStjVFhkWmsvR1RZU3lwdnJOSDdLbVh3RXlEZWJGNVdmZG0yQTQyT0ZSeEFOU1Q0bGxjWApaakdqbjRUYzNqWHUvMDhhaUJheDhaTlcvT3BFRUxYNDI1cEZsdGc2dG1FTDI3cVhHQ1RFK2xZQXllRlpPRW5nCldsdHlvM21URzV0RHVTZXBkYXFyMUhJUDZjM0dKNnptS1lBRnB4YXJYdVhOa284L1Q3QkF0UU5KMWEvM1hRcGMKelRZbGpLS2dBOTRGMGZYZjdkeFhYNTFtWWZNM1FDclJTM05yQllEbCtOYUZpWWlsdXA1cmNxNTlwNzk2S080QgorM0hpSGZtcDFxUEJIS2hqL2ZYMk1xcU5od2RkWDJ6enlHbWRNZz09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
kind: Secret
metadata:
  creationTimestamp: "2020-08-25T05:23:20Z"
  labels:
    app.kubernetes.io/instance: stash
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: stash
    app.kubernetes.io/version: v0.10.0-beta.1
    helm.sh/chart: stash-v0.10.0-beta.1
  name: stash-license
  namespace: kube-system
  resourceVersion: "111192"
  selfLink: /api/v1/namespaces/kube-system/secrets/stash-license
  uid: b5acd352-e9e6-4358-8fb9-90f58c162ce9
type: Opaque
```

## Upgrading Between Community Edition and Enterprise Edition

Stash uses two different binaries for Community edition and Enterprise edition. So, it is not possible to upgrade between the Community edition and Enterprise edition without re-installation. However, it is possible to re-install Stash without losing the existing backup resources.

Follow the below instructions to re-install Stash:

- Uninstall the old version by following the respective uninstallation guide. Don't delete the CRDs.
- Install the new version by following the respective installation guide.

## Upgrading between patch versions

If you are upgrading Stash to a patch release, please reapply the [installation instructions](/docs/setup/README.md). That will upgrade the operator pod to the new version and fix any RBAC issues.

## Upgrading from 0.9.x to v2020.x.x

If you are upgrading from `0.9.x` which did not use license verification to new `v2020.x.x`, you have to first uninstall the old version. Then, you have to re-install the new version.

If you are upgrading from `0.9.x` to `v2020.x.x` Community edition, please note that following features are only available in Enterprise edition:

- **Auto-Backup:** Auto-backup is now an enterprise feature. You won't be able to setup any new backup using auto-backup. However, your existing auto-backup resources should keep functioning.
- **Batch Backup:** Batch backup and restore is also now an enterprise feature. You won't be able to create any new backup using batch-backup. However, your existing backup should continue to work and you would be able to restore the data that were backed up using BatchBackup.
- **Local Backend:** Local backend now is an enterprise feature. If you are using any Kubernetes volume (i.e. NFS, PVC, HostPath, etc.) as backend, you won't be able to create any new backup using those backends. However, your existing backup that uses sidecar model should keep functioning. You have to use the Enterprise edition to restore from the backed up data. If you are interested in purchasing Enterprise license, please contact us via sales@appscode.com for further discussion. You can also set up a meeting via our [calendly link](https://calendly.com/appscode/30min).

If you are using any Stash addons, you might need to update the `Task` name in your `BackupConfiguration` to comply with the new naming scheme of the `Function` and `Task`.
