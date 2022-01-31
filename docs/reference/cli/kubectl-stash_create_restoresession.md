---
title: Create Restoresession
menu:
  docs_{{ .version }}:
    identifier: kubectl-stash-create-restoresession
    name: Create Restoresession
    parent: reference-cli
menu_name: docs_{{ .version }}
section_menu_id: reference
---
## kubectl-stash create restoresession

Create a new RestoreSession

### Synopsis

Create a new RestoreSession to restore backed up data

```
kubectl-stash create restoresession [flags]
```

### Examples

```
  # Create a RestoreSession
  # stash create restore --namespace=demo <restore session name> [Flag]
  # For Restic driver
  stash create restoresession ss-restore --namespace=demo --repo-name=gcs-repo --target-apiversion=apps/v1 --target-kind=StatefulSet --target-name=stash-recovered --paths=/source/data --volume-mounts=source-data:/source/data
  # For VolumeSnapshotter driver
  stash create restoresession restore-pvc --namespace=demo --driver=VolumeSnapshotter --replica=3 --claim.name=restore-data-restore-demo-${POD_ORDINAL} --claim.access-modes=ReadWriteOnce --claim.storageclass=standard --claim.size=1Gi --claim.datasource=source-data-stash-demo-0-1567146010
```

### Options

```
      --alias string                 Host identifier of the backed up data. It must be same as the alias used during backup
      --claim.access-modes strings   Access mode of the VolumeClaimTemplates
      --claim.datasource string      DataSource of the VolumeClaimTemplate
      --claim.name string            Name of the VolumeClaimTemplate
      --claim.size string            Total requested size of the VolumeClaimTemplate
      --claim.storageclass string    Name of the Storage secret for VolumeClaimTemplate
      --driver string                Driver indicates the mechanism used to backup (i.e. VolumeSnapshotter, Restic)
  -h, --help                         help for restoresession
      --host string                  Name of the Source host
      --paths strings                List of paths to backup
      --replica int32                Replica specifies the number of replicas whose data should be backed up
      --repo-name string             Name of the Repository
      --repo-namespace string        Namespace of the Repository
      --snapshots strings            Name of the Snapshot(single)
      --target-apiversion string     API-Version of the target resource
      --target-kind string           Kind of the target resource
      --target-name string           Name of the target resource
      --task string                  Name of the Task
      --volume-mounts strings        List of volumes and their mountPaths
```

### Options inherited from parent commands

```
      --as string                        Username to impersonate for the operation
      --as-group stringArray             Group to impersonate for the operation, this flag can be repeated to specify multiple groups.
      --cache-dir string                 Default cache directory (default "/home/runner/.kube/cache")
      --certificate-authority string     Path to a cert file for the certificate authority
      --client-certificate string        Path to a client certificate file for TLS
      --client-key string                Path to a client key file for TLS
      --cluster string                   The name of the kubeconfig cluster to use
      --context string                   The name of the kubeconfig context to use
      --insecure-skip-tls-verify         If true, the server's certificate will not be checked for validity. This will make your HTTPS connections insecure
      --kubeconfig string                Path to the kubeconfig file to use for CLI requests.
      --match-server-version             Require server version to match client version
  -n, --namespace string                 If present, the namespace scope for this CLI request
      --request-timeout string           The length of time to wait before giving up on a single server request. Non-zero values should contain a corresponding time unit (e.g. 1s, 2m, 3h). A value of zero means don't timeout requests. (default "0")
  -s, --server string                    The address and port of the Kubernetes API server
      --tls-server-name string           Server name to use for server certificate validation. If it is not provided, the hostname used to contact the server is used
      --token string                     Bearer token for authentication to the API server
      --use-kubeapiserver-fqdn-for-aks   if true, uses kube-apiserver FQDN for AKS cluster to workaround https://github.com/Azure/AKS/issues/522 (default true)
      --user string                      The name of the kubeconfig user to use
```

### SEE ALSO

* [kubectl-stash create](/docs/reference/cli/kubectl-stash_create.md)	 - create stash resources

