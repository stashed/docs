---
title: Restore-Pvc
menu:
  docs_{{ .version }}:
    identifier: stash-restore-pvc
    name: Restore-Pvc
    parent: operator
product_name: stash
section_menu_id: reference
menu_name: docs_{{ .version }}
---
## stash restore-pvc

Takes a restore of Persistent Volume Claim

### Synopsis

Takes a restore of Persistent Volume Claim

```
stash restore-pvc [flags]
```

### Options

```
      --bucket string           Name of the cloud bucket/container (keep empty for local backend)
      --enable-cache            Specify whether to enable caching for restic
      --endpoint string         Endpoint for s3/s3 compatible backend or REST server URL
  -h, --help                    help for restore-pvc
      --hostname string         Name of the host machine (default "host-0")
      --max-connections int     Specify maximum concurrent connections for GCS, Azure and B2 backend
      --output-dir string       Directory where output.json file will be written (keep empty if you don't need to write output in file)
      --path string             Directory inside the bucket where backed up data has been stored
      --provider string         Backend provider (i.e. gcs, s3, azure etc)
      --region string           Region for s3/s3 compatible backend
      --restore-paths strings   List of paths to restore
      --scratch-dir string      Temporary directory (default "/tmp")
      --secret-dir string       Directory where storage secret has been mounted
      --snapshots strings       List of snapshots to be restored
```

### Options inherited from parent commands

```
      --alsologtostderr                  log to standard error as well as files
      --bypass-validating-webhook-xray   if true, bypasses validating webhook xray checks
      --enable-analytics                 Send analytical events to Google Analytics (default true)
      --log-flush-frequency duration     Maximum number of seconds between log flushes (default 5s)
      --log_backtrace_at traceLocation   when logging hits line file:N, emit a stack trace (default :0)
      --log_dir string                   If non-empty, write log files in this directory
      --logtostderr                      log to standard error instead of files (default true)
      --service-name string              Stash service name. (default "stash-operator")
      --stderrthreshold severity         logs at or above this threshold go to stderr
      --use-kubeapiserver-fqdn-for-aks   if true, uses kube-apiserver FQDN for AKS cluster to workaround https://github.com/Azure/AKS/issues/522 (default true)
  -v, --v Level                          log level for V logs
      --vmodule moduleSpec               comma-separated list of pattern=N settings for file-filtered logging
```

### SEE ALSO

* [stash](/docs/reference/operator/stash.md)	 - Stash by AppsCode - Backup your Kubernetes Volumes

