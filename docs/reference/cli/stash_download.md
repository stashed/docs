---
title: Download
menu:
  product_stash_0.1.0:
    identifier: stash-download
    name: Download
    parent: stash-cli
product_name: stash
section_menu_id: reference
menu_name: product_stash_0.1.0
---
## stash download

Download snapshots

### Synopsis

Download contents of snapshots from Repository

```
stash download [flags]
```

### Options

```
      --destination string       Destination path where snapshot will be restored.
      --directories strings      List of directories to be restored
      --docker-registry string   Docker image registry (default "appscode")
  -h, --help                     help for download
      --host string              Name of the source host machine (default "host-0")
      --image-tag string         Stash image tag (default "latest")
      --kubeconfig string        Path of the Kube config file.
      --namespace string         Namespace of the Repository. (default "default")
      --repository string        Name of the Repository.
      --snapshots strings        List of snapshots to be restored
```

### Options inherited from parent commands

```
      --alsologtostderr                  log to standard error as well as files
      --enable-analytics                 Send analytical events to Google Analytics (default true)
      --enable-status-subresource        If true, uses sub resource for crds.
      --log-flush-frequency duration     Maximum number of seconds between log flushes (default 5s)
      --log_backtrace_at traceLocation   when logging hits line file:N, emit a stack trace (default :0)
      --log_dir string                   If non-empty, write log files in this directory
      --logtostderr                      log to standard error instead of files
      --service-name string              Stash service name. (default "stash-operator")
      --stderrthreshold severity         logs at or above this threshold go to stderr
      --use-kubeapiserver-fqdn-for-aks   if true, uses kube-apiserver FQDN for AKS cluster to workaround https://github.com/Azure/AKS/issues/522 (default true)
  -v, --v Level                          log level for V logs
      --vmodule moduleSpec               comma-separated list of pattern=N settings for file-filtered logging
```

### SEE ALSO

* [stash](/docs/reference/stash/stash.md)	 - kubectl plugin for Stash by AppsCode

