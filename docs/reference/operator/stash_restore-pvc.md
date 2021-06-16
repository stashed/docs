---
title: Restore-Pvc
menu:
  docs_{{ .version }}:
    identifier: stash-restore-pvc
    name: Restore-Pvc
    parent: reference-operator
menu_name: docs_{{ .version }}
section_menu_id: reference
---
## stash restore-pvc

Takes a restore of Persistent Volume Claim

```
stash restore-pvc [flags]
```

### Options

```
      --bucket string           Name of the cloud bucket/container (keep empty for local backend)
      --enable-cache            Specify whether to enable caching for restic
      --endpoint string         Endpoint for s3/s3 compatible backend or REST server URL
      --exclude strings         List of pattern for directory/file to ignore during restore. Stash will not restore those files that matches these patterns.
  -h, --help                    help for restore-pvc
      --hostname string         Name of the host machine (default "host-0")
      --include strings         List of pattern for directory/file to restore. Stash will restore only those files that matches these patterns.
      --invoker-kind string     Kind of the backup invoker
      --invoker-name string     Name of the respective backup invoker
      --kubeconfig string       Path to kubeconfig file with authorization information (the master location is set by the master flag).
      --master string           The address of the Kubernetes API server (overrides any value in kubeconfig)
      --max-connections int     Specify maximum concurrent connections for GCS, Azure and B2 backend
      --output-dir string       Directory where output.json file will be written (keep empty if you don't need to write output in file)
      --path string             Directory inside the bucket where backed up data will be stored
      --provider string         Backend provider (i.e. gcs, s3, azure etc)
      --region string           Region for s3/s3 compatible backend
      --restore-paths strings   List of paths to restore
      --scratch-dir string      Temporary directory (default "/tmp")
      --secret-dir string       Directory where storage secret has been mounted
      --snapshots strings       List of snapshots to be restored
      --target-kind string      Kind of the Target
      --target-name string      Name of the Target
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

