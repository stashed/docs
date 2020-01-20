---
title: Hooks in Batch Backup | Stash
menu:
  docs_{{ .version }}:
    identifier: backup-batch-hooks
    name: BackupBatch Hooks
    parent: hooks
    weight: 30
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: guides
---

# Hooks in Batch Backup

## Prepare Application

### Deploy Database

**Create MySQL:**

```yaml
```

```console
kubectl apply -f ./docs/examples/guides/latest/hooks/batch-backup/wordpress-mysql.yaml
mysql.kubedb.com/wordpress-mysql created
```

Wait for the database to be ready,

```console
$ kubectl get mysql -n demo
NAME              VERSION   STATUS    AGE
wordpress-mysql   8.0.14    Running   3m10s
```

**Verify Database Secret:**

```console
kubectl get secret -n demo -l=kubedb.com/name=wordpress-mysql
NAME                   TYPE     DATA   AGE
wordpress-mysql-auth   Opaque   2      4m1s
```

**Verify AppBinding:**

```console
kubectl get appbindings -n demo -l=kubedb.com/name=wordpress-mysql
NAME              TYPE               VERSION   AGE
wordpress-mysql   kubedb.com/mysql   8.0.14    2m10s
```

### Deploy Wordpress

```yaml
```

```console
$ kubectl apply -f ./docs/examples/guides/latest/hooks/batch-backup/wordpress-deployment.yaml

persistentvolumeclaim/wordpress-pvc created
deployment.apps/wordpress-deployment created
```

Verify wordpress pod ready

```console
kubectl get pod -n demo -l=app=wordpress,tier=frontend
NAME                                    READY   STATUS    RESTARTS   AGE
wordpress-deployment-586f94487c-nm8p5   1/1     Running   0          2m26s
```

## Prepare for Backup

### Configure a Slack Incoming Webhook

URL: https://api.slack.com/


### Create Storage Secret

```console
$ echo -n 'changeit' > RESTIC_PASSWORD
$ echo -n '<your-project-id>' > GOOGLE_PROJECT_ID
$ cat downloaded-sa-json.key > GOOGLE_SERVICE_ACCOUNT_JSON_KEY
$ kubectl create secret generic -n demo gcs-secret \
    --from-file=./RESTIC_PASSWORD \
    --from-file=./GOOGLE_PROJECT_ID \
    --from-file=./GOOGLE_SERVICE_ACCOUNT_JSON_KEY
secret/gcs-secret created
```

### Create Repository

```yaml
```

```console
kubectl apply -f ./docs/examples/guides/latest/hooks/batch-backup/repository.yaml
repository.stash.appscode.com/gcs-repo created
```

## Backup

### Create BackupBatch

```yaml
```

```console
kubectl apply -f ./docs/examples/guides/latest/hooks/batch-backup/wordpress-backup.yaml
backupbatch.stash.appscode.com/wordpress-backup created
```

### Verify CronJob

```console
kubectl get cronjob -n demo
NAME                           SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
stash-backup-wordprss-backup   */5 * * * *   False     0        <none>          27s
```

### Wait for BackupSession

```console
$ kubectl get backupsession -n demo -w
NAME                          INVOKER-TYPE   INVOKER-NAME       PHASE       AGE
wordpress-backup-1579526291   BackupBatch    wordpress-backup   Succeeded   2m30s
wordpress-backup-1579526461   BackupBatch    wordpress-backup               0s
wordpress-backup-1579526461   BackupBatch    wordpress-backup               0s
wordpress-backup-1579526461   BackupBatch    wordpress-backup               0s
wordpress-backup-1579526461   BackupBatch    wordpress-backup   Running     0s
wordpress-backup-1579526461   BackupBatch    wordpress-backup   Running     24s
wordpress-backup-1579526461   BackupBatch    wordpress-backup   Running     37s
wordpress-backup-1579526461   BackupBatch    wordpress-backup   Succeeded   38s

```

### Verify Backup

```console
kubectl get repository -n demo gcs-repo
NAME       INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
gcs-repo   true               5                72s                      18m

```

### Verify Backup Hooks Executed

## Cleanup

```console
kubectl -n demo delete backupbatch wordpress-backup
kubectl -n demo delete repository gcs-repo
kubectl -n demo delete secret gcs-secret

kubectl -n demo delete deployment wordpress-deployment
kubectl -n demo delete mysql wordpress-mysql
```
