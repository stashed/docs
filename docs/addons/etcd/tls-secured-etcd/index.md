---
title: TLS-secured-Etcd Backup | Stash
description: Backup TLS-secured-Etcd using Stash
menu:
  docs_{{ .version }}:
    identifier: tls-secured-etcd-backup
    name: TLS-secured-etcd
    parent: stash-etcd
    weight: 30
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: stash-addons
---


# Backup TLS secured Etcd using Stash

Stash `{{< param "info.version" >}}` supports backup and restoration of Etcd databases. This guide will show you how you can backup & restore a TLS secured Etcd database using Stash.

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster.
- Install Stash Enterprise in your cluster following the steps [here](/docs/setup/install/enterprise.md).
- Install cert-manager in your cluster following the instruction [here](https://cert-manager.io/docs/installation/).
- If you are not familiar with how Stash backup and restore Etcd databases, please check the following guide [here](/docs/addons/etcd/overview/index.md).

You have to be familiar with following custom resources:

- [AppBinding](/docs/concepts/crds/appbinding.md)
- [Function](/docs/concepts/crds/function.md)
- [Task](/docs/concepts/crds/task.md)
- [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md)
- [BackupSession](/docs/concepts/crds/backupsession.md)
- [RestoreSession](/docs/concepts/crds/restoresession.md)

To keep things isolated, we are going to use a separate namespace called `demo` throughout this tutorial. Create `demo` namespace if you haven't created already.

```bash
$ kubectl create ns demo
namespace/demo created
```

> Note: YAML files used in this tutorial are stored [here](https://github.com/stashed/docs/tree/{{< param "info.version" >}}/docs/addons/etcd/tls-secured-etcd/examples).

## Prepare Etcd

In this section, we are going to deploy a TLS secured NATS cluster. Then, we are going to create a stream and publish some messages into it.


### Create Certificate
At first, let's create certificates that our Etcd database cluster will use for handling client requests and maintaining peer communications. We will be using Etcd recommended [Cfssl](https://github.com/cloudflare/cfssl) tool for creating our necessary certificates.

If you have `go` installed on your machine, you can run the following commands to install `Cfssl`,
```bash
$ go get github.com/cloudflare/cfssl/cmd/cfssl
$ go get github.com/cloudflare/cfssl/cmd/cfssljson
````

Now, let's create the certificates using `Cfssl`. You can follow the commands from below. We have stored our SANs in the `ADDRESS` variable here.  The following commands will create 5 certificates named `ca.pem`, `server.pem`, `server-key.pem`, `client.pem`, and `client-key.pem`. The `ca.pem` is a self-signed CA certificate. Rest of the certificates are signed by this CA.
```bash
$ echo '{"CN":"CA","key":{"algo":"rsa","size":2048}}' | cfssl gencert  -initca - | cfssljson -bare ca -

$ echo '{"signing":{"default":{"expiry":"43800h","usages":["signing","key encipherment","server auth","client auth", "any"]}}}' > ca-config.json

$ export ADDRESS='etcd-0.etcd,etcd-1.etcd,etcd-2.etcd,*.etcd,etcd-service'

$ export NAME=server

$ echo '{"CN":"'$NAME'","hosts":[""],"key":{"algo":"rsa","size":2048}}' | cfssl gencert -config=ca-config.json -ca=ca.pem -ca-key=ca-key.pem -hostname="$ADDRESS" - |  cfssljson -bare $NAME

$ export NAME=client

$ echo '{"CN":"'$NAME'","hosts":[""],"key":{"algo":"rsa","size":2048}}' | cfssl gencert -config=ca-config.json -ca=ca.pem -ca-key=ca-key.pem -hostname="$ADDRESS" - | cfssljson -bare $NAME
```

### Create Secret
Let's create a generic secret with the certificates created above. To create a secret named `etcd-tls-auth` run the following command,
```bash
$ kubectl create secret generic -n demo etcd-tls-auth\
                     --from-file=./ca.pem\
                     --from-file=./server.pem\
                     --from-file=./server-key.pem\
                     --from-file=./client.pem\
                     --from-file=./client-key.pem
````

### Deploy Etcd


At first, let's deploy an Etcd database cluster. Here, we will use a statefulset and a service for deploying an Etcd database cluster consisting of three members. The service is used for handling peer communications and client requests.

Let's deploy an Etcd database cluster named `etcd` using a statefulset and a service  from the YAML manifest as below,

```yaml
apiVersion: v1
kind: Service
metadata:
  name: etcd
  namespace: demo
spec:
  clusterIP: None
  ports:
    - port: 2379
      name: client
    - port: 2380
      name: peer
  selector:
    app: etcd-tls
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: etcd-tls
  namespace: demo
  labels:
    app: etcd-tls
spec:
  serviceName: etcd
  replicas: 3
  selector:
    matchLabels:
      app: etcd-tls
  template:
    metadata:
      name: etcd-tls
      namespace: demo
      labels:
        app: etcd-tls
    spec:
      containers:
        - name: etcd
          image: gcr.io/etcd-development/etcd:v3.5.0
          ports:
            - containerPort: 2379
              name: client
            - containerPort: 2380
              name: peer
          volumeMounts:
            - name: data
              mountPath: /var/run/etcd
            - name: etcd-secret
              mountPath: /etc/etcd-secret
          command:
            - /bin/sh
            - -c
            - |
              PEERS="etcd-tls-0=https://etcd-tls-0.etcd:2380,etcd-tls-1=https://etcd-tls-1.etcd:2380,etcd-tls-2=https://etcd-tls-2.etcd:2380" ;\
              exec etcd --name ${HOSTNAME} \
                --listen-peer-urls https://0.0.0.0:2380 \
                --listen-client-urls https://0.0.0.0:2379 \
                --advertise-client-urls https://${HOSTNAME}.etcd:2379 \
                --initial-advertise-peer-urls https://${HOSTNAME}.etcd:2380 \
                --initial-cluster etcd-tls-0=https://etcd-tls-0.etcd:2380,etcd-tls-1=https://etcd-tls-1.etcd:2380,etcd-tls-2=https://etcd-tls-2.etcd:2380 \
                --initial-cluster-token etcd-cluster-1 \
                --data-dir /var/run/etcd \
                --client-cert-auth \
                --cert-file "/etc/etcd-secret/server.pem" \
                --key-file "/etc/etcd-secret/server-key.pem" \
                --trusted-ca-file "/etc/etcd-secret/ca.pem" \
                --peer-auto-tls  
      volumes:
        - name: etcd-secret
          secret:
            secretName: etcd-tls-auth
  volumeClaimTemplates:
    - metadata:
        name: data
        namespace: demo
      spec:
        storageClassName: standard
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 1Gi
```

This YAML will create the necessary Statefulset, Service, and PVCs for the Etcd database Cluster. 

Now, wait for the database pods `etcd-tls-0`, `etcd-tls-1`, and `etcd-tls-2` to go into `Running` state,

```bash
❯ kubectl get pods -n demo --selector=app=etcd-tls

NAME                READY   STATUS    RESTARTS   AGE
etcd-tls-0          1/1     Running   0          6m
etcd-tls-1          1/1     Running   0          6m
etcd-tls-2          1/1     Running   0          6m
```

Once the database pod is in `Running` state, verify that the database is ready to accept the connections. For that, we have to exec into any one of the Etcd database cluster pods and run the `endpoint health` command. If the command returns a healthy state, then we can conclude that the database is ready to accept connections. To exec into a database pod and check endpoint health, run the following commands,
```bash
❯ kubectl exec -it -n demo etcd-tls-0 -- /bin/sh

127.0.0.1:2379> etcdctl --cacert  /etc/etcd-secret/ca.pem --cert /etc/etcd-secret/client.pem --key /etc/etcd-secret/client-key.pem endpoint health 
127.0.0.1:2379 is healthy: successfully committed proposal: took = 8.497131ms
```
From the above log, we can see the Etcd database is ready to accept connections.

### Insert Sample Data
Now, we are going to exec into the database pod and create some sample data. To access TLS-secured Etcd database from Stash, we need to create a secret containing necessary `client-key` and `client certificate` in the `base64` encoded format. Let's create the secret containing the necessary TLS certificates required to access our Etcd database cluster from the following YAML,

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: etcd-stash-auth
  namespace: demo
type: Opaque
data:
 client-key.pem: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb3dJQkFBS0NBUUVBc1ZVeUQ3L25QanBnOWlrYlpIdk5pN2ZUZFUyM3NXLzZjT2p6cWdXYm9HRDJLbTdFCk9GeVFXb2FacWxWNHM5V3F3cy9vaGxjb0JHUzMrVUVNbkJjNi9QU0tTUmxVUUUxS3ZqeXhFUkwzU0Yxc3h4UnMKMExVZVFnWno2T1dubkhORTBjNXZObjlSUVo5VENOeHl0WjBqVi94MG9wd0c3KzBzNSs0R05GbmZQMjB5ZXFLNQpDM1JEbEJSWVFKRWZMYTR5dHh6TjI1TENUQ2dsaTlkdWZoaC9ieXVLdHUxVzVaTmowVVh0bXRNRkd5N2JhZ1J2CmlLQ0NqVDQvc0RqWXpTWlZ2VitESlFFemhLZmdqc2NrNXBhaTJsWFhpaXJQVzE4N3JUSkxoc1pacWt3NUlCWEEKSUpqWjFBSSttYytSUE1mVitxV0h0ZG5XOHM1TDZxd2VvVUdwUlFJREFRQUJBb0lCQUFMMGJIVWV1WGVyK1ZtZwpyYmdxNSszZ0RrSHlIWkZ6VURUNWJMWDBpZmRPSmt2bXRKWkwxSXZ0bWpuZ1dyYUVaT2dDRnRuR01nQ0F2U0FHCkdYT3dYMmMvbTk1RDhjZHdna0pST0pJVVF0S04yL1lsUFBydFNhZkgrNzV4dFMxQ0xtOWdoVEhmUlRkV3RFZDkKaE52SjFvRHN6L1Mxck5mcWw4ajFpbHpzOG05WUYxWEtoRlozQmpSWHBFRFdCWUIzeTE2dGV6RlU0NlJaMSs1UwpaR3pTakxhOTNuK0JEeGc2ejdMVjhON0xyYXVQY3lCbkFnUC9KMlR0MTQ3bHM2cHhUWlh2UFd0ZXdvRHV1aHF1CndWQ1FkUWZ3bmlPTmJmbUxDNUp6OWNrNzJvWUg2ditjbEtFc05GT2xZWjdjRGkzaTZ6K2RuK1JOYTUvc295Mm0KV0RsQ1ZURUNnWUVBNHlUSGlXYk5FMEk1N2NnNDNlWkJ6eTBSMGtvTktJRGlJMWZkQmRBWUcraU1vZzZUSEdBSApvYkRFSnI0YkZ3aUM1QmlYeHVxZ2RQSVhqcXNNTGtYM2Zra3ErNkh0VS9XRS9pSmJyRnN6clUreFdoYmxEc0ZPCkRCUWw3SjlkaitRaGcyeXBOcEtjUWVYS0E5WVdvV1VqTkp0Z1dVMG81MmFVcUF3NnRKWW8wMzhDZ1lFQXg5eDAKZVg1YkUyWU02ZU9tZWVXWWdBZC93cVV4NmlYaWl4S2M3Z1Z6WG9qVUVaaUQwVEdrSi9RTW9TNHNHV01OekRlYQorbTdXNzVSTUZoOGlzQjdlTUxlSE0zRTRBV08yVlc4SUVuM1h6U0JLNngvcWIwS2VoWVNIS0R4YnQ0UDBGSVdTCjdYc0pYc09mRlRMclZLbHYyZnE2ZzBwZ1R1Ri9QZzdaRUIxQWxUc0NnWUJ4VDJhdTEzYVVGZVI2QnZpL1VWOGcKNzdYRk5xV3J2K2VQaEFSQkl4Ynp6U1Zpcm15YXFoa0VndjdHNk96d3A1Rk1JaXlNMFh5citoemdVZG1vdDhTSAozZzR3S3c0T1pSc3IvNDNGeEZWYUxyZ2xYZWgwWE9BSFRJSENzWmxsNzRMOFlkZGozdTFPUGtoeGMzb2tseVJoCjJPVE9oNXhSR3k0clNyWjZZYklLRndLQmdDUEg1eDVkTGNjQ1RTdU9jeDU5cVZpNmZ2Z0ZCVE9yUnF5cFQya1oKbHJjRS9ocU1XSVVhUXc1WUZlN0JTbW5kSHZwQnRrQkJtYjlZcUdxSmRuZGJmMkh2YVlnZksreXJ3bGYzUWRXMQpxKzN3YXhrL0pJUjR3OUtaa0d6MnFXRG9nY2t1eE1nNWI4c0VjTFdsNFJYT0k5VTlteWlvSnlmWUhTU3FHZGhWCnRGdERBb0dCQUlEZ1RVOHZNL2VkTml3TVZSMmRnZjRXNEVhcE9zWXhmdFZnOU5YUFNOOEdVNHFtWURCNGtIWnMKQjg5dzFEajB0WU0xQnNseXRYN3llSTUxNi8rUDVuYnQ4eHRtS3RUWVlrWmlvOGFEM2FaZXh0ZTIySXRjTE1xTwpMMERWZ0s3bzRxMVdUVWVCZWkxbDB4b3JXWUR0a0tIYkUvWGJDeHFtOGgzWDdpMlVWOWZFCi0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0t
 client.pem: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURnekNDQW11Z0F3SUJBZ0lVT1VLSTFzY2pVaEROTEJPazdma3J4dFFlNU9Fd0RRWUpLb1pJaHZjTkFRRUwKQlFBd0RURUxNQWtHQTFVRUF4TUNRMEV3SGhjTk1qRXhNREU1TVRFMU56QXdXaGNOTWpZeE1ERTRNVEUxTnpBdwpXakFSTVE4d0RRWURWUVFERXdaamJHbGxiblF3Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQkR3QXdnZ0VLCkFvSUJBUUN4VlRJUHYrYytPbUQyS1J0a2U4Mkx0OU4xVGJleGIvcHc2UE9xQlp1Z1lQWXFic1E0WEpCYWhwbXEKVlhpejFhckN6K2lHVnlnRVpMZjVRUXljRnpyODlJcEpHVlJBVFVxK1BMRVJFdmRJWFd6SEZHelF0UjVDQm5Qbwo1YWVjYzBUUnptODJmMUZCbjFNSTNISzFuU05YL0hTaW5BYnY3U3puN2dZMFdkOC9iVEo2b3JrTGRFT1VGRmhBCmtSOHRyakszSE0zYmtzSk1LQ1dMMTI1K0dIOXZLNHEyN1ZibGsyUFJSZTJhMHdVYkx0dHFCRytJb0lLTlBqK3cKT05qTkpsVzlYNE1sQVRPRXArQ094eVRtbHFMYVZkZUtLczliWHp1dE1rdUd4bG1xVERrZ0ZjQWdtTm5VQWo2Wgp6NUU4eDlYNnBZZTEyZGJ5emt2cXJCNmhRYWxGQWdNQkFBR2pnZFl3Z2RNd0RnWURWUjBQQVFIL0JBUURBZ1dnCk1DTUdBMVVkSlFRY01Cb0dDQ3NHQVFVRkJ3TUJCZ2dyQmdFRkJRY0RBZ1lFVlIwbEFEQU1CZ05WSFJNQkFmOEUKQWpBQU1CMEdBMVVkRGdRV0JCU3dUcWpsZWlxVCsyU1pHM1lndzBTY3RDeEkxekFmQmdOVkhTTUVHREFXZ0JRawpMSDVQNmE2bDVkZnZCL1BlV0FINE9MeTdkVEJPQmdOVkhSRUVSekJGZ2c5bGRHTmtMWFJzY3kwd0xtVjBZMlNDCkQyVjBZMlF0ZEd4ekxURXVaWFJqWklJUFpYUmpaQzEwYkhNdE1pNWxkR05rZ2dSbGRHTmtod1IvQUFBQmh3UUEKQUFBQU1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRQXJlNHd2b2hGK094dHpONGgvM0RjUHd1dk5hU3BoMnpEWQp4SUwzVDY1U2kxcmNYL3NPdmVVZ0hBZ0RIeGRqeWoyRXB0ckpDWFBsaElQSGt4NWYydHVzV1hxRWVUNHhVdmo3CnlGaXZTSFR3OEtBSWlWL0FjWi96Z2JXYXFqaXo4dTFzd2JZVXFGWERkbDR2ZktVOFc2dVh2elU5U2k2RDRMTWwKbEtHMzY0VStCVzcxS2lBNnR3MmZ1ZitOd3E0TkFNUHZhRlhZaU5JOHVWUDV5eFNFZnBFcldicjBFZ2JwU3JWcwo4SVJPTEJ4cFo5b09qWGloNHBMWTFXV29xVXR4bVR6SWZhMjgxRi85ZzAyMUVrMmErb1lKbnZhMjQvMGEwNHlKCkYyYTM1VDZ0eVhsU0oxMGhkTFpDbi9QMnVNTkNJajBVZk5LZ1M3RVVMQ0x0UnpzbzhMWGkKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQ==
```
Let's create the `etcd-stash-auth` secret we have shown above,
```bash
$ kubectl apply -f https://github.com/stashed/docs/tree/{{< param "info.version" >}}/docs/addons/etcd/tls-secured-etcd/examples/etcd-secret.yaml
secret/etcd-stash-auth created
```


Now, let's exec into any one of the database pods and insert some sample data,
```bash
❯ kubectl exec -it -n demo etcd-tls-0 -- /bin/sh
127.0.0.1:2379> etcdctl --cacert  /etc/etcd-secret/ca.pem --cert /etc/etcd-secret/client.pem --key /etc/etcd-secret/client-key.pem put foo bar
OK
127.0.0.1:2379> etcdctl --cacert  /etc/etcd-secret/ca.pem --cert /etc/etcd-secret/client.pem --key /etc/etcd-secret/client-key.pem put foo2 bar2
OK
127.0.0.1:2379> etcdctl --cacert  /etc/etcd-secret/ca.pem --cert /etc/etcd-secret/client.pem --key /etc/etcd-secret/client-key.pem put foo3 bar3
OK
127.0.0.1:2379> etcdctl --cacert  /etc/etcd-secret/ca.pem --cert /etc/etcd-secret/client.pem --key /etc/etcd-secret/client-key.pem put foo4 bar4
OK

# Verify that the data has been inserted successfully
127.0.0.1:2379> etcdctl --cacert  /etc/etcd-secret/ca.pem --cert /etc/etcd-secret/client.pem --key /etc/etcd-secret/client-key.pem get --prefix foo
foo
bar
foo2
bar2
foo3
bar3
foo4
bar4
127.0.0.1:2379> exit
```

We have successfully deployed an Etcd database cluster and inserted some sample data into it. In the subsequent sections, we are going to backup these data using Stash.


## Prepare for Backup

In this section, we are going to prepare the necessary resources (i.e. connection information, backend information, etc.) before backup.

### Ensure Etcd Addon

When you install Stash Enterprise version, it will automatically install all the official database addons. Make sure that Etcd addon was installed properly using the following command.

```bash
❯ kubectl get tasks.stash.appscode.com | grep etcd
etcd-backup-3.5.0             18m
etcd-restore-3.5.0            18m
```

This addon should be able to take backup of the databases with matching major versions as discussed in [Addon Version Compatibility](/docs/addons/etcd/README.md#addon-version-compatibility).


### Create AppBinding

Stash needs to know how to connect with the Etcd databse cluster. An `AppBinding` exactly provides this information. It holds the Service and Secret information of the Etcd database cluster. You have to point to the respective `AppBinding` as a target of backup instead of the Etcd database cluster itself.

Here, is the YAML of the `AppBinding` that we are going to create for the Etcd database cluster we have deployed earlier.

```yaml
apiVersion: appcatalog.appscode.com/v1alpha1
kind: AppBinding
metadata:
  name: etcd-appbinding 
  namespace: demo
spec:
  clientConfig:
    caBundle: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM2akNDQWRLZ0F3SUJBZ0lVZEJYNnU0RHRyNDA3WkZDTWN6Nzg4Y1YyWHVJd0RRWUpLb1pJaHZjTkFRRUwKQlFBd0RURUxNQWtHQTFVRUF4TUNRMEV3SGhjTk1qRXhNREU1TVRBeE56QXdXaGNOTWpZeE1ERTRNVEF4TnpBdwpXakFOTVFzd0NRWURWUVFERXdKRFFUQ0NBU0l3RFFZSktvWklodmNOQVFFQkJRQURnZ0VQQURDQ0FRb0NnZ0VCCkFMZmp6OEJhbDhpWTdjSTd2OHNFdndpSmJnQXBqTDIvNnVZbXBaQVZvcW50cjJCcmFMbTlCb1czdkMydm5tbWYKWDhRUFBsWnFIeUxzY1gxZUlpazJyWHJEYUdQaU45VHhLQXVrbWJjZXFsUXZScGNZZkVaVTBQMzhNc0xsQUlHaQpZamZxRjR5Z1UyMjA0L3FucVVPbFFjLzh2MmJpRFNSQXNUM1NUT2FVemdMd05KT20wOUlqT1dUQW15Q2xXWmxnClJmV2tETzVTQ0xZd1pmQ1Z1MTdBalJRMTNsZTdocW9GcW9SUW96dUZQMFp0dlVFdWVPSXlSa3ZlYTFRYVNyTjgKcm1aN1BaL0lIb3dyNjFtMFd0bVE3ckw1eTMyOTgxb3hRNGg2UHdLMGNGNVNyOG9WRUR0TGZVeE1OWHpoMXpRUAorY2FhZWlUWklyc1dqRGNxSm9VWk1wVUNBd0VBQWFOQ01FQXdEZ1lEVlIwUEFRSC9CQVFEQWdFR01BOEdBMVVkCkV3RUIvd1FGTUFNQkFmOHdIUVlEVlIwT0JCWUVGQ1FzZmsvcHJxWGwxKzhIODk1WUFmZzR2THQxTUEwR0NTcUcKU0liM0RRRUJDd1VBQTRJQkFRQmpOOHlxalhEaUdqaDFuUGdKVFpZa1FCa1E4SFZHeThIdlFWWG0zdFhHM3RLcApTaVdDQTlPbittR0xZZTlCMi9lOEFJNkIyMFB1Z0NZWHljMHVOb1RiSFdiZ01pNURiNEVZYXdDdVpkN3M4MXlUClRXY1N0VWZFeUdKNW1ycWZ6OFFPaUt0Sk1UeWFYbHllWllURnhnWUsyOThTcDhubFBRQlgzNVBDakZTbXZwa1UKY0N5TllYMDF1RnFhWE9FZTdkTnpBUmhIaU1xVFM0dkNUUnFrbFpMWHB6a3ViZER3WTVKK3gzYmptcDk1RUtmVwpJbkhERTJ2ckxsU0crZDVWNnBuUG54bXc3aGg2L0NMUTNVMXM3V1d0clVlSU9NUDdCcFJxRG05TjJhajJXUHNLCmpEYXdRcFpGdGhmbEdJR1ZqUUNCNFMwTkQzWWliSkVqdHZHMnBuTlkKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQ==
    service:
      name: etcd
      port: 2379
      scheme: https
  secret:
    name: etcd-stash-auth
  type: etcd
  version: 3.5.0 
```

Here,

- `.spec.clientConfig.caBundle` specifies a PEM encoded CA bundle which will be used to validate the serving certificate of the Etcd database cluster.
- `.spec.clientConfig.service` specifies the Service information to use to connects with the Etcd database cluster.
- `.spec.secret` specifies the name of the Secret that holds necessary credentials to access the database.
- `spec.type` specifies the type of the database.

Let's create the `AppBinding` we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/tree/{{< param "info.version" >}}/docs/addons/etcd/tls-secured-etcd/examples/appbinding.yaml
appbinding.appcatalog.appscode.com/etcd-appbinding created
```

### Prepare Backend

We are going to store our backed up data into a GCS bucket. So, we need to create a Secret with GCS credentials and a `Repository` object with the bucket information. If you want to use a different backend, please read the respective backend configuration doc from [here](/docs/guides/latest/backends/overview.md).

**Create Storage Secret:**

At first, let's create a secret called `gcs-secret` with access credentials to our desired GCS bucket,

```bash
$ echo -n 'changeit' > RESTIC_PASSWORD
$ echo -n '<your-project-id>' > GOOGLE_PROJECT_ID
$ cat downloaded-sa-json.key > GOOGLE_SERVICE_ACCOUNT_JSON_KEY
$ kubectl create secret generic -n demo gcs-secret \
    --from-file=./RESTIC_PASSWORD \
    --from-file=./GOOGLE_PROJECT_ID \
    --from-file=./GOOGLE_SERVICE_ACCOUNT_JSON_KEY
secret/gcs-secret created
```

**Create Repository:**

Now, create a `Repository` object with the information of your desired bucket. Below is the YAML of `Repository` object we are going to create,

```yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: gcs-repo
  namespace: demo
spec:
  backend:
    gcs:
      bucket: stash-testing
      prefix: /demo/etcd/etcd-tls
    storageSecretName: gcs-secret
```

Let's create the `Repository` we have shown above,

```bash
$ kubectl create -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/addons/etcd/tls-secured-etcd/examples/repository.yaml
repository.stash.appscode.com/gcs-repo created
```

Now, we are ready to backup our streams into our desired backend.

### Backup

To schedule a backup, we have to create a `BackupConfiguration` object targeting the respective `AppBinding` of our `Etcd` database cluster. Then, Stash will create a CronJob to periodically backup the streams.

#### Create BackupConfiguration

Below is the YAML for `BackupConfiguration` object that we are going to use to backup the data of the Etcd database cluster we have created earlier,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: etcd-tls-backup
  namespace: demo
spec:
  schedule: "*/5 * * * *"
  task:
    name: etcd-backup-3.5.0
  repository:
    name: gcs-repo
  target:
    ref:
      apiVersion: appcatalog.appscode.com/v1alpha1
      kind: AppBinding
      name: etcd-appbinding 
  retentionPolicy:
    name: keep-last-5
    keepLast: 5
    prune: true
```

Here,

- `.spec.schedule` specifies that we want to backup the streams at 5 minutes intervals.
- `.spec.task.name` specifies the name of the Task object that specifies the necessary Functions and their execution order to backup Etcd database.
- `.spec.repository.name` specifies the Repository CR name we have created earlier with backend information.
- `.spec.target.ref` refers to the AppBinding object that holds the connection information of our targeted Etcd database cluster.
- `.spec.retentionPolicy` specifies a policy indicating how we want to cleanup the old backups.

Let's create the `BackupConfiguration` object we have shown above,

```bash
$ kubectl create -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/addons/etcd/tls-secured-etcd/examples/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/etcd-tls-backup created
```

#### Verify CronJob

If everything goes well, Stash will create a CronJob with the schedule specified in `spec.schedule` field of `BackupConfiguration` object.

Verify that the CronJob has been created using the following command,

```bash
❯ kubectl get cronjob -n demo
NAME                                  SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
stash-trigger-etcd-tls-backup         */5 * * * *   False     0        <none>          65s
```

#### Wait for BackupSession

The `stash-trigger-etcd-tls-backup` CronJob will trigger a backup on each scheduled slot by creating a `BackupSession` object.

Now, wait for a schedule to appear. Run the following command to watch for `BackupSession` object,

```bash
❯ kubectl get backupsession -n demo -w
NAME                           INVOKER-TYPE          INVOKER-NAME             PHASE       DURATION   AGE
etcd-tls-backup-q2dk6          BackupConfiguration   etcd-tls-backup          Succeeded   41s        60s
```

Here, the phase `Succeeded` means that the backup process has been completed successfully.

#### Verify Backup

Now, we are going to verify whether the backed up data is present in the backend or not. Once a backup is completed, Stash will update the respective `Repository` object to reflect the backup completion. Check that the repository `gcs-repo` has been updated by the following command,

```bash
❯ kubectl get repository -n demo
NAME         INTEGRITY   SIZE        SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
gcs-repo     true        42.105 KiB  2                2m10s                    47m
```

Now, if we navigate to the GCS bucket, we will see the backed up data has been stored in `demo/nats/sample-nats-tls` directory as specified by `.spec.backend.gcs.prefix` field of the `Repository` object.

<figure align="center">
  <img alt="Backup data in GCS Bucket" src="/docs/addons/etcd/tls-secured-etcd/images/etcd-tls-backup.jpg">
  <figcaption align="center">Fig: Backup data in GCS Bucket</figcaption>
</figure>



> Note: Stash keeps all the backed up data encrypted. So, data in the backend will not make any sense until they are decrypted.

## Restore

If you have followed the previous sections properly, you should have a successful backup of your Etcd database cluster. Now, we are going to show how you can restore the database from the backed up data.

### Restore Into the Same Database

You can restore your data into the same database you have backed up from or into a different database in the same cluster or a different cluster. In this section, we are going to show you how to restore in the same database which may be necessary when you have accidentally deleted any data from the running database.


#### Temporarily Pause Backup

At first, let's stop taking any further backup of the database so that no backup runs after we delete the sample data. We are going to pause the `BackupConfiguration` object. Stash will stop taking any further backup when the `BackupConfiguration` is paused.

Let's pause the `etcd-tls-backup` BackupConfiguration,

```bash
$ kubectl patch backupconfiguration -n demo etcd-tls-backup --type="merge" --patch='{"spec": {"paused": true}}'
backupconfiguration.stash.appscode.com/etcd-tls-backup patched
```

Verify that the `BackupConfiguration` has been paused,

```bash
❯ kubectl get backupconfiguration -n demo etcd-tls-backup
NAME              TASK                SCHEDULE      PAUSED   AGE
etcd-tls-backup   etcd-backup-3.5.0   */5 * * * *   true     25m
```

Notice the `PAUSED` column. Value `true` for this field means that the `BackupConfiguration` has been paused.

Stash will also suspend the respective CronJob.

```bash
❯ kubectl get cronjob -n demo
NAME                            SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
stash-trigger-etcd-tls-backup   */5 * * * *   True      0        6m25s           95m
```

#### Simulate Disaster

Now, let's simulate an accidental deletion scenario. Here, we are going to exec into a database pod and delete the sample data we have inserted earlier.

```bash
❯ kubectl exec -it -n demo etcd-tls-0 -- /bin/sh
127.0.0.1:2379> etcdctl --cacert  /etc/etcd-secret/ca.pem --cert /etc/etcd-secret/client.pem --key /etc/etcd-secret/client-key.pem del foo
1
127.0.0.1:2379> etcdctl --cacert  /etc/etcd-secret/ca.pem --cert /etc/etcd-secret/client.pem --key /etc/etcd-secret/client-key.pem del foo2 
1
127.0.0.1:2379> etcdctl --cacert  /etc/etcd-secret/ca.pem --cert /etc/etcd-secret/client.pem --key /etc/etcd-secret/client-key.pem del foo3
1
127.0.0.1:2379> etcdctl --cacert  /etc/etcd-secret/ca.pem --cert /etc/etcd-secret/client.pem --key /etc/etcd-secret/client-key.pem del foo4
1

# Verify that the data has been deleted successfully
127.0.0.1:2379> etcdctl --cacert  /etc/etcd-secret/ca.pem --cert /etc/etcd-secret/client.pem --key /etc/etcd-secret/client-key.pem get --prefix foo
(nil)
127.0.0.1:2379> exit
```

#### Create RestoreSession

To restore the database, you have to create a `RestoreSession` object pointing to the `AppBinding` of the targeted database.

Here, is the YAML of the `RestoreSession` object that we are going to use for restoring our `Etcd` database cluster.

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: etcd-tls-restore
  namespace: demo
spec:
  task:
    name: etcd-restore-3.5.0 
    params:
      - name: initialCluster
        value:  "etcd-tls-0=https://etcd-tls-0.etcd:2380,etcd-tls-1=https://etcd-tls-1.etcd:2380,etcd-tls-2=https://etcd-tls-2.etcd:2380"
      - name: initialClusterToken
        value: "etcd-cluster-1"
      - name: dataDir
        value: "/var/run/etcd"
      - name: workloadKind
        value: "StatefulSet"
      - name: workloadName
        value: "etcd"
  repository:
    name: gcs-repo
  target:
    ref:
      apiVersion: appcatalog.appscode.com/v1alpha1
      kind: AppBinding
      name: etcd-appbinding
  runtimeSettings:
    container:
      securityContext:
        runAsUser: 0
        runAsGroup: 0
  rules:
  - snapshots: [latest]
```

Here,

- `.spec.task.name` specifies the name of the Task object that specifies the necessary Functions and their execution order to restore a Etcd database.
- `.spec.task.params` refers to the names and values of the Params objects specifying necessary parameters and their values for restoring backupdata into an Etcd database cluster.
- `.spec.repository.name` specifies the Repository object that holds the backend information where our backed up data has been stored.
- `.spec.target.ref` refers to the respective AppBinding of the `etcd` database.
- `.spec.rules` specifies that we are restoring data from the latest backup snapshot of the database.

Let's create the `RestoreSession` object object we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/addons/etcd/tls-secured-etcd/examples/restoresession.yaml
restoresession.stash.appscode.com/sample-nats-restore-tls created
```

Once, you have created the `RestoreSession` object, Stash will create a restore Job. Run the following command to watch the phase of the `RestoreSession` object,

```bash
❯ kubectl get restoresession -n demo -w
NAME               REPOSITORY   PHASE     DURATION   AGE
etcd-tls-restore   gcs-repo     Running              5s
etcd-tls-restore   gcs-repo     Running              60s
etcd-tls-restore   gcs-repo     Running              2m3s
etcd-tls-restore   gcs-repo     Succeeded              2m3s
etcd-tls-restore   gcs-repo     Succeeded   2m3s       2m3s
```

The `Succeeded` phase means that the restore process has been completed successfully.

#### Verify Restored Data

Now, let's exec into the database pod and verify whether data actual data has been restored or not,

```bash
❯ kubectl exec -it -n demo etcd-tls-0 -- /bin/sh

127.0.0.1:2379> etcdctl --cacert  /etc/etcd-secret/ca.pem --cert /etc/etcd-secret/client.pem --key /etc/etcd-secret/client-key.pem get --prefix foo
foo
bar
foo2
bar2
foo3
bar3
foo4
bar4
127.0.0.1:2379> exit
```

Hence, we can see from the above output that the deleted data has been restored successfully from the backup.

#### Resume Backup

Since our data has been restored successfully we can now resume our usual backup process. Resume the `BackupConfiguration` using following command,

```bash
❯ kubectl patch backupconfiguration -n demo etcd-tls-backup --type="merge" --patch='{"spec": {"paused": false}}'
backupconfiguration.stash.appscode.com/etcd-tls-backup patched
```

Verify that the `BackupConfiguration` has been resumed,
```bash
❯ kubectl get backupconfiguration -n demo etcd-tls-backup
NAME              TASK                SCHEDULE      PAUSED   AGE
etcd-tls-backup   etcd-backup-3.5.0   */5 * * * *   false    39m
```

Here,  `false` in the `PAUSED` column means the backup has been resumed successfully. The CronJob also should be resumed now.

```bash
❯ kubectl get cronjob -n demo
NAME                            SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
stash-trigger-etcd-tls-backup   */5 * * * *   False     0        2m39s           42m
```

Here, `False` in the `SUSPEND` column means the CronJob is no longer suspended and will trigger in the next schedule.

## Cleanup

To cleanup the Kubernetes resources created by this tutorial, run:

```bash
kubectl delete -n demo backupconfiguration etcd-tls-backup
kubectl delete -n demo restoresession etcd-tls-restore
kubectl delete -n demo repository gcs-repo
kubectl delete -n demo appbinding etcd-appbinding
# delete the database, service, and PVCs
kubectl delete -f https://github.com/stashed/docs/tree/{{< param "info.version" >}}/docs/addons/etcd/tls-secured-etcd/examples/etcd-tls.yaml
kubectl delete pvc -n demo data-etcd-tls-0 data-etcd-tls-1 data-etcd-tls-2
```
