---
title: Kubectl-Stash Clone Pvc
menu:
  docs_{{ .version }}:
    identifier: kubectl-stash-clone-pvc
    name: Kubectl-Stash Clone Pvc
    parent: reference-cli
menu_name: docs_{{ .version }}
section_menu_id: reference
---
## kubectl-stash clone pvc

Clone PVC

### Synopsis

Use Backup and Restore process for cloning PVC

```
kubectl-stash clone pvc [flags]
```

### Examples

```
  # Clone PVC
  stash clone pvc source-pvc -n demo --to-namespace=demo1 --secret=<secret> --bucket=<bucket> --prefix=<prefix> --provider=<provider>
```

### Options

```
      --bucket string         Name of the cloud bucket/container
      --endpoint string       Endpoint for s3/s3 compatible backend
  -h, --help                  help for pvc
      --max-connections int   Specify maximum concurrent connections for GCS, Azure and B2 backend
      --prefix string         Prefix denotes the directory inside the backend
      --provider string       Backend provider (i.e. gcs, s3, azure etc)
      --secret string         Name of the Storage Secret
```

### Options inherited from parent commands

```
      --alsologtostderr                  log to standard error as well as files
      --as string                        Username to impersonate for the operation
      --as-group stringArray             Group to impersonate for the operation, this flag can be repeated to specify multiple groups.
      --cache-dir string                 Default HTTP cache directory (default "/home/runner/.kube/http-cache")
      --certificate-authority string     Path to a cert file for the certificate authority
      --client-certificate string        Path to a client certificate file for TLS
      --client-key string                Path to a client key file for TLS
      --cluster string                   The name of the kubeconfig cluster to use
      --context string                   The name of the kubeconfig context to use
      --enable-analytics                 Send analytical events to Google Analytics (default true)
      --insecure-skip-tls-verify         If true, the server's certificate will not be checked for validity. This will make your HTTPS connections insecure
      --kubeconfig string                Path to the kubeconfig file to use for CLI requests.
      --log-backtrace-at traceLocation   when logging hits line file:N, emit a stack trace (default :0)
      --log-dir string                   If non-empty, write log files in this directory
      --log-flush-frequency duration     Maximum number of seconds between log flushes (default 5s)
      --logtostderr                      log to standard error instead of files
      --match-server-version             Require server version to match client version
  -n, --namespace string                 If present, the namespace scope for this CLI request
      --request-timeout string           The length of time to wait before giving up on a single server request. Non-zero values should contain a corresponding time unit (e.g. 1s, 2m, 3h). A value of zero means don't timeout requests. (default "0")
  -s, --server string                    The address and port of the Kubernetes API server
      --stderrthreshold severity         logs at or above this threshold go to stderr
      --tls-server-name string           Server name to use for server certificate validation. If it is not provided, the hostname used to contact the server is used
      --to-namespace string              Destination namespace.
      --token string                     Bearer token for authentication to the API server
      --use-kubeapiserver-fqdn-for-aks   if true, uses kube-apiserver FQDN for AKS cluster to workaround https://github.com/Azure/AKS/issues/522 (default true)
      --user string                      The name of the kubeconfig user to use
  -v, --v Level                          log level for V logs
      --vmodule moduleSpec               comma-separated list of pattern=N settings for file-filtered logging
```

### SEE ALSO

* [kubectl-stash clone](/docs/reference/cli/kubectl-stash_clone.md)	 - Clone Kubernetes resources

