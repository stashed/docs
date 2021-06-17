---
title: Backup
menu:
  docs_{{ .version }}:
    identifier: stash-backup
    name: Backup
    parent: reference-operator
menu_name: docs_{{ .version }}
section_menu_id: reference
---
## stash backup

Run Stash Backup

```
stash backup [flags]
```

### Options

```
      --burst int                The maximum burst for throttle (default 100)
      --docker-registry string   Check job image registry. (default "appscode")
  -h, --help                     help for backup
      --image-tag string         Check job image tag.
      --kubeconfig string        Path to kubeconfig file with authorization information (the master location is set by the master flag).
      --master string            The address of the Kubernetes API server (overrides any value in kubeconfig)
      --pushgateway-url string   URL of Prometheus pushgateway used to cache backup metrics
      --qps float                The maximum QPS to the master from this client (default 100)
      --restic-name string       Name of the Restic used as configuration.
      --resync-period duration   If non-zero, will re-list this often. Otherwise, re-list will be delayed aslong as possible (until the upstream source closes the watch or times out. (default 5m0s)
      --run-via-cron             Run backup periodically via cron.
      --scratch-dir emptyDir     Directory used to store temporary files. Use an emptyDir in Kubernetes. (default "/tmp")
      --workload-kind string     Kind of workload where sidecar pod is added.
      --workload-name string     Name of workload where sidecar pod is added.
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

