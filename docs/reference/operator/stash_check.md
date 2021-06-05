---
title: Check
menu:
  docs_{{ .version }}:
    identifier: stash-check
    name: Check
    parent: reference-operator
menu_name: docs_{{ .version }}
section_menu_id: reference
---
## stash check

Check restic backup

```
stash check [flags]
```

### Options

```
  -h, --help                  help for check
      --host-name string      Host name for workload.
      --kubeconfig string     Path to kubeconfig file with authorization information (the master location is set by the master flag).
      --master string         The address of the Kubernetes API server (overrides any value in kubeconfig)
      --restic-name string    Name of the Restic CRD.
      --smart-prefix string   Smart prefix for workload
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

