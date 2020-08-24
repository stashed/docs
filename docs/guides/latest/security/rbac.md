---
title: RBAC | Stash
description: Using Stash in RBAC enabled cluster
menu:
  docs_{{ .version }}:
    identifier: security-rbac
    name: RBAC
    parent: security
    weight: 10
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: guides
---

# Stash with RBAC Enabled Cluster

## Configuring RBAC

Stash introduces resources, such as, `Restic`, `Repository`, `Recovery` and `Snapshot`. Stash installer will create 2 user facing cluster roles:

| ClusterRole         | Aggregates To | Desription                                                                                            |
| ------------------- | ------------- | ----------------------------------------------------------------------------------------------------- |
| appscode:stash:edit | admin, edit   | Allows edit access to Stash CRDs, intended to be granted within a namespace using a RoleBinding.      |
| appscode:stash:view | view          | Allows read-only access to Stash CRDs, intended to be granted within a namespace using a RoleBinding. |

These user facing roles supports [ClusterRole Aggregation](https://kubernetes.io/docs/admin/authorization/rbac/#aggregated-clusterroles) feature in Kubernetes 1.9 or later clusters.
