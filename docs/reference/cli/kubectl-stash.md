---
title: Kubectl-Stash
menu:
  docs_{{ .version }}:
    identifier: kubectl-stash
    name: Kubectl-Stash
    parent: reference-cli
    weight: 0

menu_name: docs_{{ .version }}
section_menu_id: reference
url: /docs/{{ .version }}/reference/cli/
aliases:
- /docs/{{ .version }}/reference/cli/kubectl-stash/
---
## kubectl-stash

kubectl plugin for Stash by AppsCode

### Synopsis

kubectl plugin for Stash by AppsCode. For more information, visit here: https://appscode.com/products/stash

### Options

```
      --as string                      Username to impersonate for the operation. User could be a regular user or a service account in a namespace.
      --as-group stringArray           Group to impersonate for the operation, this flag can be repeated to specify multiple groups.
      --as-uid string                  UID to impersonate for the operation.
      --cache-dir string               Default cache directory (default "/home/runner/.kube/cache")
      --certificate-authority string   Path to a cert file for the certificate authority
      --client-certificate string      Path to a client certificate file for TLS
      --client-key string              Path to a client key file for TLS
      --cluster string                 The name of the kubeconfig cluster to use
      --context string                 The name of the kubeconfig context to use
      --disable-compression            If true, opt-out of response compression for all requests to the server
  -h, --help                           help for kubectl-stash
      --insecure-skip-tls-verify       If true, the server's certificate will not be checked for validity. This will make your HTTPS connections insecure
      --kubeconfig string              Path to the kubeconfig file to use for CLI requests.
      --match-server-version           Require server version to match client version
  -n, --namespace string               If present, the namespace scope for this CLI request
      --request-timeout string         The length of time to wait before giving up on a single server request. Non-zero values should contain a corresponding time unit (e.g. 1s, 2m, 3h). A value of zero means don't timeout requests. (default "0")
  -s, --server string                  The address and port of the Kubernetes API server
      --tls-server-name string         Server name to use for server certificate validation. If it is not provided, the hostname used to contact the server is used
      --token string                   Bearer token for authentication to the API server
      --user string                    The name of the kubeconfig user to use
```

### SEE ALSO

* [kubectl-stash check](/docs/reference/cli/kubectl-stash_check.md)	 - Check the repository for errors
* [kubectl-stash clone](/docs/reference/cli/kubectl-stash_clone.md)	 - Clone Kubernetes resources
* [kubectl-stash completion](/docs/reference/cli/kubectl-stash_completion.md)	 - Generate completion script
* [kubectl-stash cp](/docs/reference/cli/kubectl-stash_cp.md)	 - Copy stash resources from one namespace to another namespace
* [kubectl-stash create](/docs/reference/cli/kubectl-stash_create.md)	 - create stash resources
* [kubectl-stash debug](/docs/reference/cli/kubectl-stash_debug.md)	 - Debug common Stash issues
* [kubectl-stash delete](/docs/reference/cli/kubectl-stash_delete.md)	 - Delete stash resources
* [kubectl-stash download](/docs/reference/cli/kubectl-stash_download.md)	 - Download snapshots
* [kubectl-stash gen](/docs/reference/cli/kubectl-stash_gen.md)	 - generate stash resources
* [kubectl-stash key](/docs/reference/cli/kubectl-stash_key.md)	 - manages restic keys (passwords) for accessing the repository
* [kubectl-stash pause](/docs/reference/cli/kubectl-stash_pause.md)	 - Pause Stash backup temporarily
* [kubectl-stash rebuild-index](/docs/reference/cli/kubectl-stash_rebuild-index.md)	 - Build a new index
* [kubectl-stash resume](/docs/reference/cli/kubectl-stash_resume.md)	 - Resume Stash backup
* [kubectl-stash trigger](/docs/reference/cli/kubectl-stash_trigger.md)	 - Trigger a backup
* [kubectl-stash unlock](/docs/reference/cli/kubectl-stash_unlock.md)	 - Unlock restic repository
* [kubectl-stash version](/docs/reference/cli/kubectl-stash_version.md)	 - Prints binary version number.

