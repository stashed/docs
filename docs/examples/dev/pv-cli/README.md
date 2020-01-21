# Backup Persistent Volume using Stash CLI

```
$ kubectl create ns demo
namespace/demo created
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/pv-cli/functions.yaml
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/pv-cli/tasks.yaml
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/pv-cli/repo_secret.yaml
secret/local-secret created
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/pv-cli/backup_template.yaml
backupblueprints.stash.appscode.com/pvc-backup created
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/pv-cli/rbac.yaml
clusterrole.rbac.authorization.k8s.io/update-status-roles created
serviceaccount/pvc-backup-restore created
rolebinding.rbac.authorization.k8s.io/update-status-binding created
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/pv-cli/pv.yam
persistentvolume/demo-pv created
$ stash cli backup-pv --namespace demo --template pvc-backup --volume demo-pv --directories /source/data --mountpath /source/data
```
