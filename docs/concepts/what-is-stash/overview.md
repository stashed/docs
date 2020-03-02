---
title: Stash Overview
description: Stash Overview
menu:
  docs_{{ .version }}:
    identifier: overview-concepts
    name: Overview
    parent: what-is-stash
    weight: 10
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: concepts
---

# Stash

[Stash](https://stash.run) by AppsCode is a cloud native data backup and recovery solution for Kubernetes workloads. If you are running production workloads in Kubernetes, you might want to take backup of your disks, databases etc. Traditional tools are too complex to setup and maintain in a dynamic compute environment like Kubernetes. Stash is a Kubernetes operator that uses [restic](https://github.com/restic/restic) or Kubernetes CSI Driver VolumeSnapshotter functionality to address these issues. Using Stash, you can backup Kubernetes volumes mounted in workloads, stand-alone volumes and databases. User may even extend Stash via [addons](https://stash.run/docs/latest/guides/latest/addons/overview/) for any custom workload.

## Features

| Features                                                                        | Availability | Scope                                                                                                                                                 |
| ------------------------------------------------------------------------------- | :----------: | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| Backup & Restore Workload Data                                                  |   &#10003;   | Deployment, DaemonSet, StatefulSet, ReplicaSet, ReplicationController, OpenShift DeploymentConfig                                                     |
| Backup & Restore Stand-alone Volume (PVC)                                       |   &#10003;   | PersistentVolumeClaim, PersistentVolume                                                                                                               |
| Backup & Restore databases                                                      |   &#10003;   | PostgreSQL, MySQL, MongoDB, Elasticsearch                                                                                                             |
| [VolumeSnapshot](https://kubernetes.io/docs/concepts/storage/volume-snapshots/) |   &#10003;   | CSI Driver must support VolumeSnapshot and Kubernetes Alpha features must be enabled                                                                  |
| Schedule Backup                                                                 |   &#10003;   | Schedule through [cron expression](https://en.wikipedia.org/wiki/Cron)                                                                                |
| Instant Backup                                                                  |   &#10003;   | Use CLI or create BackupSession manually                                                                                                              |
| Auto Backup                                                                     |   &#10003;   | Using a Template and annotations                                                                                                                      |
| Batch Backup                                                                    |   &#10003;   | Backup multiple co-related targets under a single configuration                                                                                       |
| Pause Scheduled Backup                                                          |   &#10003;   |                                                                                                                                                       |
| Support Multiple Storage Provider                                               |   &#10003;   | AWS S3, Minio, Rook, GCS, Azure, OpenStack Swift,  Backblaze B2, Rest Server, any PV/PVC                                                              |
| Encryption                                                                      |   &#10003;   | AES-256 in counter mode (CTR) (for Restic driver)                                                                                                     |
| Deduplication (send only diff)                                                  |   &#10003;   | Uses [Content Defined Chunking (CDC)](https://restic.net/blog/2015-09-12/restic-foundation1-cdc) (for Restic driver)                                  |
| Cleanup old snapshots automatically                                             |   &#10003;   | Cleanup according to different [retention policies](https://restic.readthedocs.io/en/stable/060_forget.html#removing-snapshots-according-to-a-policy) |
| Prometheus Metrics for Backup & Restore Process                                 |   &#10003;   | Official Prometheus Server, CoreOS Prometheus Operator                                                                                                |
| Prometheus Metrics for Stash Operator                                           |   &#10003;   | Official Prometheus Server, CoreOS Prometheus Operator                                                                                                |
| Support RBAC enabled cluster                                                    |   &#10003;   |                                                                                                                                                       |
| Support PSP enabled cluster                                                     |   &#10003;   |                                                                                                                                                       |
| CLI                                                                             |   &#10003;   | `kubectl` plugin (for Kubernetes 1.12+)                                                                                                               |
| Extensibility                                                                   |   &#10003;   | Extend using `Function` and `Task`                                                                                                                    |
| Customizability                                                                 |   &#10003;   | Customize backup / restore process using `Function` and `Task`                                                                                        |
| Hooks                                                                           |   &#10003;   | Execute `httpGet`, `httpPost`, `tcpSocket` and `exec` hooks before and after  of backup or restore process.                                           |
| Send Notification to Webhook                                                    |   &#10003;   | Use hooks to send notification to webhooks(i.e. Slack channel)                                                                                        |
