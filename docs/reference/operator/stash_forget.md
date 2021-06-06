---
title: Forget
menu:
  docs_{{ .version }}:
    identifier: stash-forget
    name: Forget
    parent: reference-operator
menu_name: docs_{{ .version }}
section_menu_id: reference
---
## stash forget

Delete snapshots from a restic repository

```
stash forget [snapshotID ...] [flags]
```

### Options

```
  -h, --help                help for forget
      --kubeconfig string   Path to kubeconfig file with authorization information (the master location is set by the master flag).
      --master string       The address of the Kubernetes API server (overrides any value in kubeconfig)
      --repo-name string    Name of the Repository CRD.
```

### Options inherited from parent commands

```
      --bypass-validating-webhook-xray   if true, bypasses validating webhook xray checks
      --enable-analytics                 Send analytical events to Google Analytics (default true)
      --service-name string              Stash service name. (default "stash-operator")
      --use-kubeapiserver-fqdn-for-aks   if true, uses kube-apiserver FQDN for AKS cluster to workaround https://github.com/Azure/AKS/issues/522 (default true)
```

### SEE ALSO

* [stash](/docs/reference/operator/stash.md)	 - Stash by AppsCode - Backup your Kubernetes Volumes

