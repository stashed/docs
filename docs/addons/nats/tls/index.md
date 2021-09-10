---
title: TLS secured NATS
description: Backup TLS secured NATS using Stash
menu:
  docs_{{ .version }}:
    identifier: tls-auth
    name: TLS secured NATS
    parent: stash-nats
    weight: 40
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: stash-addons
---

# Backup TLS secured NATS using Stash

Stash `{{< param "info.version" >}}` supports backup and restoration of NATS streams. This guide will show you how you can backup & restore a TLS secured NATS server using Stash.

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster.
- Install Stash Enterprise in your cluster following the steps [here](/docs/setup/install/enterprise.md).
- Install cert-manager in your cluster following the instruction [here](https://cert-manager.io/docs/installation/)
- If you are not familiar with how Stash backup and restore NATS streams, please check the following guide [here](/docs/addons/nats/overview/index.md).

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

> Note: YAML files used in this tutorial are stored [here](https://github.com/stashed/docs/tree/{{< param "info.version" >}}/docs/addons/nats/tls/examples).

## Prepare NATS

In this section, we are going to deploy a TLS secured NATS cluster. Then, we are going to create a stream and publish some messages into it.


### Create Certificate
To issue a CA certificate we have to create a `ClusterIssuer`. Below is the YAML of `ClusterIssuer` object we are going to create.
```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: selfsigning
  namespace: demo
spec:
  selfSigned: {}
```
Let's create the `ClusterIssuer` we have shown above,
```bash
$ kubectl apply -f https://github.com/stashed/docs/tree/{{< param "info.version" >}}/docs/addons/nats/tls/examples/clusterissuer.yaml
clusterissuer.cert-manager.io/selfsigning created
```

Now we are going to create a CA certificate issued by the the `ClusterIssuer`. Below is the YAML of `Certificate` object we are going to create.

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: nats-ca
  namespace: demo
spec:
  secretName: nats-ca
  duration: 8736h # 1 year
  renewBefore: 240h # 10 days
  issuerRef:
    name: selfsigning
    kind: ClusterIssuer
  commonName: nats-ca
  isCA: true
```
Let's create the `Certificate` we have shown above,
```bash
$ kubectl apply -f https://github.com/stashed/docs/tree/{{< param "info.version" >}}/docs/addons/nats/tls/examples/ca.yaml
certificate.cert-manager.io/nats-ca
```
To issue a TLS certificate we have to create an `Issuer`. Below is the YAML of `Issuer` object we are going to create.

```yaml
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: nats-ca
spec:
  ca:
    secretName: nats-ca
```
Let's create the `Certificate` we have shown above,
```bash
$ kubectl apply -f https://github.com/stashed/docs/tree/{{< param "info.version" >}}/docs/addons/nats/tls/examples/issuer.yaml
issuer.cert-manager.io/nats-ca created

Now we are going to create a TLS certificate issued by the the `Issuer`. Below is the YAML of `Certificate` object we are going to create.
```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: nats-tls
  namespace: demo
spec:
  secretName: nats-client-tls
  duration: 2160h # 90 days
  renewBefore: 240h # 10 days
  issuerRef:
    name: nats-ca
    kind: Issuer
  commonName: sample-nats-client
  dnsNames:
  - sample-nats
```
Let's create the `Certificate` we have shown above,
```bash
$ kubectl apply -f https://github.com/stashed/docs/tree/{{< param "info.version" >}}/docs/addons/nats/tls/examples/cert.yaml
certificate.cert-manager.io/nats-tls created
```

### Deploy NATS Cluster

At first, let's deploy a NATS cluster. Here, we are going to use [NATS]( https://nats-io.github.io/k8s/helm/charts/)  chart from [nats.io](https://nats.io/).

Let's deploy a NATS cluster named `sample-nats` using Helm as below,

```bash
# Add nats chart registry
$ helm repo add nats https://nats-io.github.io/k8s/helm/charts/ 
# Update helm registries
$ helm repo update
# Install nats/nats chart into demo namespace
$ helm install sample-nats nats/nats -n demo \
--set nats.jetstream.enabled=true \
--set nats.jetstream.fileStorage.enabled=true \
--set nats.tls.secret.name=nats-tls \
--set nats.tls.ca="ca.crt" \
--set nats.tls.cert="tls.crt" \
--set nats.tls.key="tls.key" \
--set nats.tls.verify=true \
--set cluster.enabled=true \
--set cluster.recplicas=3 

```

This chart will create the necessary StatefulSet, Service, PVCs etc. for the NATS cluster. You can easily view all the resources created by chart using [ketall](https://github.com/corneliusweig/ketall) `kubectl` plugin as below,

```bash
❯ kubectl get-all -n demo -l app.kubernetes.io/instance=sample-nats
NAME                                                    NAMESPACE  AGE
configmap/sample-nats-config                            demo       11m  
endpoints/sample-nats                                   demo       11m  
persistentvolumeclaim/sample-nats-js-pvc-sample-nats-0  demo       11m  
persistentvolumeclaim/sample-nats-js-pvc-sample-nats-1  demo       10m  
persistentvolumeclaim/sample-nats-js-pvc-sample-nats-2  demo       10m  
pod/sample-nats-0                                       demo       11m  
pod/sample-nats-1                                       demo       10m  
pod/sample-nats-2                                       demo       10m  
service/sample-nats                                     demo       11m  
controllerrevision.apps/sample-nats-775468b94f          demo       11m  
statefulset.apps/sample-nats                            demo       11m  
endpointslice.discovery.k8s.io/sample-nats-7n7v6        demo       11m  
```

Now, wait for the NATS server pods `sample-nats-0`, `sample-nats-1`, `sample-nats-2` to go into `Running` state,

```bash
❯ kubectl get pod -n demo -l app.kubernetes.io/instance=sample-nats
NAME            READY   STATUS    RESTARTS   AGE
sample-nats-0   3/3     Running   0          9m58s
sample-nats-1   3/3     Running   0          9m35s
sample-nats-2   3/3     Running   0          9m12s
```

Once the pods are in `Running` state, verify that the NATS server is ready to accept the connections.

```bash
❯ kubectl logs -n demo sample-nats-0 -c nats
[7] 2021/09/06 08:33:53.111508 [INF] Starting nats-server
[7] 2021/09/06 08:33:53.111560 [INF] Version:  2.4.0
...
[7] 2021/09/06 08:33:53.116004 [INF] Server is ready
```

From the above log, we can see the NATS server is ready to accept connections.

### Insert Sample Data
The above Helm chart also deploy a pod with nats-box image which can be used to interact with the NATS server. Let's verify the nats-box pod has been created.

```bash
❯ kubectl get pod -n demo -l app=sample-nats-box
NAME                               READY   STATUS    RESTARTS   AGE
sample-nats-box-785f8458d7-wtnfx   1/1     Running   0          7m20s
```

Let's exec into the nats-box pod,

```
❯ kubectl exec -n demo -it sample-nats-box-785f8458d7-wtnfx -- sh -l
...

# Let's export the ca.crt, tls.crt and tls.key file paths as environment variables to make further commands re-usable.
sample-nats-box-785f8458d7-wtnfx:~# export NATS_CA=/etc/nats-certs/clients/nats-tls/ca.crt
sample-nats-box-785f8458d7-wtnfx:~# export NATS_CERT=/etc/nats-certs/clients/nats-tls/tls.crt
sample-nats-box-785f8458d7-wtnfx:~# export NATS_KEY=/etc/nats-certs/clients/nats-tls/tls.key

# Let's create a stream named "ORDERS"
sample-nats-box-785f8458d7-wtnfx:~# nats stream add ORDERS --subjects "ORDERS.*" --ack --max-msgs=-1 --max-bytes=-1 --max-age=1y --storage file --retention limits --max-msg-size=-1 --max-msgs-per-subject=-1 --discard old --dupe-window="0s" --replicas 1
Stream ORDERS was created

Information for Stream ORDERS created 2021-09-03T07:12:07Z

Configuration:

             Subjects: ORDERS.*
     Acknowledgements: true
            Retention: File - Limits
             Replicas: 1
       Discard Policy: Old
     Duplicate Window: 2m0s
     Maximum Messages: unlimited
        Maximum Bytes: unlimited
          Maximum Age: 1y0d0h0m0s
 Maximum Message Size: unlimited
    Maximum Consumers: unlimited


Cluster Information:

                 Name: nats
               Leader: nats-0

State:

             Messages: 0
                Bytes: 0 B
             FirstSeq: 0
              LastSeq: 0
     Active Consumers: 0


# Verify that the stream has been created successfully
sample-nats-box-785f8458d7-wtnfx:~#  nats stream ls
Streams:

	ORDERS
	
# Lets add some messages to the stream "ORDERS"
sample-nats-box-785f8458d7-wtnfx:~# nats pub ORDERS.scratch hello
08:55:39 Published 5 bytes to "ORDERS.scratch"

# Add another message
sample-nats-box-785f8458d7-wtnfx:~# nats pub ORDERS.scratch world
08:56:11 Published 5 bytes to "ORDERS.scratch"

# Verify that the messages have been published to the stream successfully
sample-nats-box-785f8458d7-wtnfx:~# nats stream info ORDERS
Information for Stream ORDERS created 2021-09-03T07:12:07Z

Configuration:

             Subjects: ORDERS.*
     Acknowledgements: true
            Retention: File - Limits
             Replicas: 1
       Discard Policy: Old
     Duplicate Window: 2m0s
     Maximum Messages: unlimited
        Maximum Bytes: unlimited
          Maximum Age: 1y0d0h0m0s
 Maximum Message Size: unlimited
    Maximum Consumers: unlimited


Cluster Information:

                 Name: nats
               Leader: nats-0

State:

             Messages: 2
                Bytes: 98 B
             FirstSeq: 1 @ 2021-09-03T08:55:39 UTC
              LastSeq: 2 @ 2021-09-03T08:56:11 UTC
     Active Consumers: 0

sample-nats-box-785f8458d7-wtnfx:~# exit
```

We have successfully deployed a NATS cluster, created a stream and publish some messages into the stream. In the subsequent sections, we are going to backup this sample data using Stash.

## Prepare for Backup

In this section, we are going to prepare the necessary resources (i.e. connection information, backend information, etc.) before backup.

### Ensure NATS Addon

When you install Stash Enterprise version, it will automatically install all the official addons. Make sure that NATS addon was installed properly using the following command.

```bash
❯ kubectl get tasks.stash.appscode.com | grep nats
nats-backup-2.4.0            24m
nats-restore-2.4.0           24m
```

This addon should be able to take backup of the NATS streams with matching major versions as discussed in [Addon Version Compatibility](/docs/addons/nats/README.md#addon-version-compatibility).

### Create Secret

 Lets create a secret with access credentials.  Below is the YAML of `Secret` object we are going to create.

```bash
apiVersion: v1
kind: Secret
metadata:
  labels:
    app.kubernetes.io/instance: sample-nats
  name: sample-nats-auth
  namespace: demo

data:
  crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURDRENDQWZDZ0F3SUJBZ0lRRVk3WGozMDJoVmkvSElRWnlzWVE0akFOQmdrcWhraUc5dzBCQVFzRkFEQVMKTVJBd0RnWURWUVFERXdkdVlYUnpMV05oTUI0WERUSXhNRGt4TURFd016Y3pNMW9YRFRJeE1USXdPVEV3TXpjegpNMW93RXpFUk1BOEdBMVVFQXhNSWJtRjBjeTEwYkhNd2dnRWlNQTBHQ1NxR1NJYjNEUUVCQVFVQUE0SUJEd0F3CmdnRUtBb0lCQVFEUytXRjNPaGVXSG5aR3EvdHNGWDRRU3ZMb2FzbjlSZE9lYjhnWHlnUTdacWFTR3BFbERURGsKbGxRc0NtNlh6UjloQythMnByaDI4ZEcyRnZLQXNObDI3WGFVbUpyaWpRMk5XTWxKS1dIMHpSOUtDZk9DZnFxcAp5Vzh4RW95ZkZ5dmdMS3FFckh1RDRWZmdrNEE5ZDBoS2huMEJFSzZKaHhKL082anUweXN0eEpwaXVkSWoyUWZZClB0Rmhyak5VUVRnd2lyTlpIV09Kc043eDUrM2h1aTdFekwrUXNMVnlwNlFVb1h1K3JQZThnVk5oYW1DY2tWNFoKc1pSUjEwYzB0VDhpd09YOERSM0dwOEgwYVVheHFHVWFYRGs5ejZ0d0s0a0k0ZGFXTmRGZVJNa0NhNXR0aEtvMgp2MDVFNEh2bHBSenR3ZkJBM3ZRWDg2M0xBdjQ5ZzR0ZEFnTUJBQUdqV1RCWE1BNEdBMVVkRHdFQi93UUVBd0lGCm9EQU1CZ05WSFJNQkFmOEVBakFBTUI4R0ExVWRJd1FZTUJhQUZCNWRuQWlUdDZMV1AxYVhGUHJZaVpFaldZOWwKTUJZR0ExVWRFUVFQTUEyQ0MzTmhiWEJzWlMxdVlYUnpNQTBHQ1NxR1NJYjNEUUVCQ3dVQUE0SUJBUUJvRWJ4MgoxNzNvTUpJNjNyWkdDazFVVWhLYm1QVngzbnE2Um50d1kzODYvNGtZcG4rejlkNExOelF1d0xnZ0NsU3NWZzU0Cjh1YndYM3JVckpXeEVXY2hLdG1YWGtHRDlhQkRFdmdrUlZwZE8wcDVyN05DKys4c25xSWVrdXk3M21tTjNidW0KR2dpdzIyVGREazg3K0ppUTdmeGs1LzY1UlFpb1BGOGs3VE5pajlKbG81OFJIaWFzaWJIM0xDLzgrZnNGc2l4bgpVK2gzOVBkeWtMTWQ4eHRBa1p6VG94ZGlZT3VCQlQxNjBUdTNaZTgwdCthUlU5VSsveEpTaGtBR2dubVRhS0VtCkRtUHFZTXpJUWlONi9tRzY3bXBVaU5GMlJMbTkxNzEyTHVuQ1FWRi8wak5MKzQzV0RCMzlqUHJLWlFSSm9sMWQKNWQ5VzkrZUZHYmY2RnJLTgotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
  key: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcEFJQkFBS0NBUUVBMHZsaGR6b1hsaDUyUnF2N2JCVitFRXJ5NkdySi9VWFRubS9JRjhvRU8yYW1raHFSCkpRMHc1SlpVTEFwdWw4MGZZUXZtdHFhNGR2SFJ0aGJ5Z0xEWmR1MTJsSmlhNG8wTmpWakpTU2xoOU0wZlNnbnoKZ242cXFjbHZNUktNbnhjcjRDeXFoS3g3ZytGWDRKT0FQWGRJU29aOUFSQ3VpWWNTZnp1bzd0TXJMY1NhWXJuUwpJOWtIMkQ3UllhNHpWRUU0TUlxeldSMWppYkRlOGVmdDRib3V4TXkva0xDMWNxZWtGS0Y3dnF6M3ZJRlRZV3BnCm5KRmVHYkdVVWRkSE5MVS9Jc0RsL0EwZHhxZkI5R2xHc2FobEdsdzVQYytyY0N1SkNPSFdsalhSWGtUSkFtdWIKYllTcU5yOU9ST0I3NWFVYzdjSHdRTjcwRi9PdHl3TCtQWU9MWFFJREFRQUJBb0lCQVFDTm92cTZMbmZ6UXBRcApNZ0RqT1RLZkFaSStVcU0wdEJJMFJkdFJqVUIyR0o0dmJFS1JXMSs5dHViZmdrbHBCRDdTeWtpanl4NEFUS3g3CjBNQVJnYWtkWEtMRmRBWlhubGJBMUNDZVMzZ0YvMGZJVHhINlE4dVRuVFpXL1pzR20ybkxZSEZDRFFJSE1kdnIKVzFzMjZNUFJ0S1JyZUtXVWt1ZnFIMFYwd0xRMlRwVXU4SU9vM0ppQ2FxRjZEZGxtNWEvbFgwUEpGS0FWbnIyMApwbytIV0tobmlPMk1lazlDbmZLQ1h2RXYyaWRLQTArQlVLNzFEaGw4Y25QdzFaZWRQMXF6WlNTU1pUWEhNQm9sCkJOVjh6U2xVWEliekQ0NVAyZmpNSDhVOWF6eFU2NTRDRnFNeHp3U254YmEwMUNOL0dQK3ErVVV3a0pxampzTGcKUkR5VWFSREpBb0dCQU8zQXBab3ZGd2lrS3dMcW5idElZdjBLcVRCTXpWSFJpUUp4dU02b3VKY1JXSzg3T2NmSApVQmxmUXRmaFN0dDN6dzl6Y1lWWm85cnNTc3gvYVluZDY2bVZhdXRtWGkxVDNhZXhwUkdLcDhGZnlzdEJaYTBQClZ1SytGc3lUaDA3RS9DVHlNWFZBdE9aMlJPckg4OHo2cFdDNmRMVE9DZHJ5RTdsZjJTc2F2ZHRIQW9HQkFPTXEKbDljSk9jRjhzbE1yRVZWT1R2UDRkNDBmcHFFWW1QUGNDRkxTT21USjJMMXByYjFiOHJPY1BMaWIxaUtmQWJ5SwoxaGh6SWg2aHBJd0RNNXhYOGd2RVZTMFVjZjd2RzJuV3lESTEyTzhJclAxanl4S0FlenBaQ0lzMVBjQUhiVitHCkYrM2ZoNXo4R25hQzJyeS9QY3BOOGRqRXg5WTVKaVZlZVFKRjZ1NDdBb0dBSTlsNm53Y2V1QVRaSDNWMUZ6cFIKQXNyS3ZDZTRoZS9NY3Z2bTIvS0E4dmFBb3R1UldOaHE4WWgxc2N1YzEvNzJ6K09lYUhjZHgvTDlURnloODFIdApLUU1JdmpvUFZWSmlCOWszaEsrZG9BRHJ1VDVCTUpreGhyc1hBUDMxMXlESXpHRmdwOGQ3LzR3eDFCMFdYQUFuClU3Q0p6SUdNNXVDOXJLUVJRUGlsVEIwQ2dZRUEyRUV5L1VYT0VyRVh2ZTd3K0VtdEJicFNiU2xsWWxUZFBzRUgKdDNoa21KQkM0Y1paM0R0Tko4a2pVUWNoYWlIKzhETW5MMjFqWE0rNnFvTmR2WWRIYUFaR283eWo3UEpKSVkrVApVNkZKVy96aFdmT0hYWnlzTXRhUk9KeTlwVElzMzlQeXNjT3JBVHBLSXVuZE8vTys2ZmticzZWWkxFbUpVK2ZFCndQSTRmUU1DZ1lCTHZydWx3MGl6eDFkeStiN1JVaFp0d3R4bGtvVkxvdGdyVm5UTHFRY0VjNjJ0Q3ZOZW5hUVYKdHlKd0dOaTNaYkRUMW5WcEQ0UmhTcCswdjBJM0VTYURhUmcvUDlmNy9teUNoN3RyazlEcHduQ3J4N1lGWFhGTAp3M0F4RzJtV2NSb1dQbnhGN0dtRUpDOWlPQ2FzeW4wWHUyYnVIcXZNQkxIVUJ0VTZmbFFjakE9PQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=
```

Let's create the `Secret` we have shown above,
```bash
$ kubectl apply -f https://github.com/stashed/docs/tree/{{< param "info.version" >}}/docs/addons/nats/tls/examples/secret.yaml
secret/sample-nats-auth created
```


### Create AppBinding

Stash needs to know how to connect with the NATS server. An `AppBinding` exactly provides this information. It holds the Service and Secret information of the NATS server. You have to point to the respective `AppBinding` as a target of backup instead of the NATS server itself.

Here, is the YAML of the `AppBinding` that we are going to create for the NATS server we have deployed earlier.

```yaml
apiVersion: appcatalog.appscode.com/v1alpha1
kind: AppBinding
metadata:
  labels:
    app.kubernetes.io/instance: sample-nats
  name: sample-nats
  namespace: demo
spec:
  clientConfig:
    caBundle: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM4VENDQWRtZ0F3SUJBZ0lSQUptTUdvMzUyNHZHdWo4SkRVR05WUVl3RFFZSktvWklodmNOQVFFTEJRQXcKRWpFUU1BNEdBMVVFQXhNSGJtRjBjeTFqWVRBZUZ3MHlNVEE1TVRBeE1ETTNNamxhRncweU1qQTVNRGt4TURNMwpNamxhTUJJeEVEQU9CZ05WQkFNVEIyNWhkSE10WTJFd2dnRWlNQTBHQ1NxR1NJYjNEUUVCQVFVQUE0SUJEd0F3CmdnRUtBb0lCQVFEbjNDanFBaE94MG5SbGYrYmJaZXRQenJwSGFkOUdBUS9aa3ZHd2drMWFuY1hwclplVkZjaUYKTVJ5d0IvL3pvdXZGNm9WVTV5NXNlbzRoOU5CQ3NHRzFrRFU3Q1M3Q3dvaGhCNXR5MHBjb2xEaVhCVUVBQXVVMwpIL3ZEV0pGY04wdmpGbnFndWk3ZjFuOVluQzY4aGU1cTczYVdzNnArbEVuMVZOR2dZeHl6c1JUbTdCK2htV045CmJrSzFOd0JaNWN5b3dpVkl0UE5ES3ZRQ2lrT1hrNUZPMGxBL0FSRlA3K01mNFNleXBHM3Z6c2ZnR3k4eEZ2R08KRnJpUDI1VXUvTmNBREdnRW92SDBJWUpSd1VpZkJRVHUvbTAzVytmaVA1eWxQcEUxMVJvOXlFZk45cEd5T2RsYgp6R0xXRGZsNGh3OEhzdXQ2UW1zUTdNczg1c0pZLzBRVkFnTUJBQUdqUWpCQU1BNEdBMVVkRHdFQi93UUVBd0lDCnBEQVBCZ05WSFJNQkFmOEVCVEFEQVFIL01CMEdBMVVkRGdRV0JCUWVYWndJazdlaTFqOVdseFQ2MkltUkkxbVAKWlRBTkJna3Foa2lHOXcwQkFRc0ZBQU9DQVFFQWw1WmFjWjhKVXNQWGNpU1hEU0U2OG4zR0VPSDl6UExYZUk3MQpnSk4rZjNWTXZzNVUzR3FjdGkvbjR6bXNIU3dDOUU4SDB4RmM4VnR0d2xRTEgrOWtwRnpMcHBaeEVYYXNtOEhhCi93SDdISzF4NEIzeEw3Tk9QemM3YVFKaVZ0S2pMSmNkQW95VGZwWGFnY2dya05OVWVUaUNucXpWUXcyLzFudzgKMWxMYWRpUXJPK0RHck5qcUxUYXB0RVR2anJwczhIMjlFWGJaZmhFWUhDRGtLd2pYMWdNSGJXNXEwWWFyWDFRNQpCTGplb3JVZEZkWTBzNytKeVlUWTRkcGhYTmdJeHMzdWFBdTdMWjMxUVRsd0RRZk5KWVV2cGtaa2RDemVwWDF6Cmo3NVI4Z3lUWno0WEdoTS8wdHFrU3JyWWMwWjU1QTdFR3h1dUg0Si9FcGRrMHJJcnV3PT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    service:
      name: sample-nats
      port: 4222
      scheme: nats
  secret:
    name: sample-nats-auth
  type: nats
  version: 2.4.0
```

Here,

- `.spec.clientConfig.caBundle` specifies a PEM encoded CA bundle which will be used to validate the serving certificate of the NATS server.
- `.spec.clientConfig.service` specifies the Service information to use to connects with the NATS server.
- `.spec.secret` specifies the name of the Secret that holds necessary credentials to access the server.
- `.spec.type` specifies the type of the target. This is particularly helpful in auto-backup where you want to use different path prefixes for different types of target.

Let's create the `AppBinding` we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/tree/{{< param "info.version" >}}/docs/addons/nats/tls/examples/appbinding.yaml
appbinding.appcatalog.appscode.com/sample-nats created
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
      prefix: /demo/nats/sample-nats
    storageSecretName: gcs-secret
```

Let's create the `Repository` we have shown above,

```bash
$ kubectl create -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/addons/nats/tls/examples/repository.yaml
repository.stash.appscode.com/gcs-repo created
```

Now, we are ready to backup our streams into our desired backend.

### Backup

To schedule a backup, we have to create a `BackupConfiguration` object targeting the respective `AppBinding` of our NATS server. Then, Stash will create a CronJob to periodically backup the streams.

#### Create BackupConfiguration

Below is the YAML for `BackupConfiguration` object that we are going to use to backup the streams of the NATS server we have created earlier,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: sample-nats-backup
  namespace: demo
spec:
  task:
    name: nats-backup-2.4.0
  schedule: "*/2 * * * *"
  repository:
    name: gcs-repo
  target:
    ref:
      apiVersion: appcatalog.appscode.com/v1alpha1
      kind: AppBinding
      name: sample-nats
  interimVolumeTemplate:
    metadata:
      name: sample-nats-backup-tmp-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "standard"
      resources:
        requests:
          storage: 1Gi  
  runtimeSettings:
    pod:
      securityContext:
        runAsUser: 0
        runAsGroup: 0
  retentionPolicy:
    name: keep-last-5
    keepLast: 5
    prune: true
```

Here,

- `.spec.schedule` specifies that we want to backup the streams at 5 minutes intervals.
- `.spec.task.name` specifies the name of the Task object that specifies the necessary Functions and their execution order to backup NATS streams.
- `.spec.repository.name` specifies the Repository CR name we have created earlier with backend information.
- `.spec.target.ref` refers to the AppBinding object that holds the connection information of our targeted NATS server.
- `spec.interimVolumeTemplate` specifies a PVC template that will be used by Stash to hold the dumped data temporarily before uploading it into the cloud bucket.
- `spec.runtimeSettings.pod.securityContext` specifies security options that backup job’s pod should run with.
- `.spec.retentionPolicy` specifies a policy indicating how we want to cleanup the old backups.

Let's create the `BackupConfiguration` object we have shown above,

```bash
$ kubectl create -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/addons/nats/tls/examples/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/sample-nats-backup created
```

#### Verify CronJob

If everything goes well, Stash will create a CronJob with the schedule specified in `spec.schedule` field of `BackupConfiguration` object.

Verify that the CronJob has been created using the following command,

```bash
❯ kubectl get cronjob -n demo
NAME                               SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
stash-backup-sample-nats-backup   */5 * * * *   False     0        <none>          14s
```

#### Wait for BackupSession

The `sample-nats-backup` CronJob will trigger a backup on each scheduled slot by creating a `BackupSession` object.

Now, wait for a schedule to appear. Run the following command to watch for `BackupSession` object,

```bash
❯ kubectl get backupsession -n demo -w
NAME                       INVOKER-TYPE          INVOKER-NAME         PHASE       DURATION   AGE
sample-nats-backup-8x8fp   BackupConfiguration   sample-nats-backup   Succeeded   42s        8m28s
```

Here, the phase `Succeeded` means that the backup process has been completed successfully.

#### Verify Backup

Now, we are going to verify whether the backed up data is present in the backend or not. Once a backup is completed, Stash will update the respective `Repository` object to reflect the backup completion. Check that the repository `gcs-repo` has been updated by the following command,

```bash
❯ kubectl get repository -n demo
NAME       INTEGRITY   SIZE        SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
gcs-repo   true        1.382 KiB   1                9m4s                     24m
```

Now, if we navigate to the GCS bucket, we will see the backed up data has been stored in `demo/nats/sample-nats` directory as specified by `.spec.backend.gcs.prefix` field of the `Repository` object.

<figure align="center">
  <img alt="Backup data in GCS Bucket" src="/docs/addons/nats/authentications/token/images/sample-nats-backup.png">
  <figcaption align="center">Fig: Backup data in GCS Bucket</figcaption>
</figure>



> Note: Stash keeps all the backed up data encrypted. So, data in the backend will not make any sense until they are decrypted.

## Restore

If you have followed the previous sections properly, you should have a successful backup of your nats streams. Now, we are going to show how you can restore the streams from the backed up data.

### Restore Into the Same NATS Cluster

You can restore your data into the same NATS cluster you have backed up from or into a different NATS cluster in the same cluster or a different cluster. In this section, we are going to show you how to restore in the same NATS cluster which may be necessary when you have accidentally lost any data.

#### Temporarily Pause Backup

At first, let's stop taking any further backup of the NATS streams so that no backup runs after we delete the sample data. We are going to pause the `BackupConfiguration` object. Stash will stop taking any further backup when the `BackupConfiguration` is paused.

Let's pause the `sample-nats-backup` BackupConfiguration,

```bash
$ kubectl patch backupconfiguration -n demo sample-nats-backup --type="merge" --patch='{"spec": {"paused": true}}'
backupconfiguration.stash.appscode.com/sample-nats-backup patched
```

Verify that the `BackupConfiguration` has been paused,

```bash
❯ kubectl get backupconfiguration -n demo sample-nats-backup
NAME                 TASK                SCHEDULE      PAUSED   AGE
sample-nats-backup   nats-backup-2.4.0   */2 * * * *   true     2d18h
```

Notice the `PAUSED` column. Value `true` for this field means that the `BackupConfiguration` has been paused.

Stash will also suspend the respective CronJob.

```bash
❯ kubectl get cronjob -n demo
NAME                              SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
stash-backup-sample-nats-backup   */2 * * * *   True      0        56s             2d18h
```

#### Simulate Disaster

Now, let's simulate a disaster scenario. Here, we are going to exec into the nats-box pod and delete the sample data we have inserted earlier.

```
❯ kubectl exec -n demo -it sample-nats-box-785f8458d7-wtnfx -- sh -l
...
# Let's export the ca.crt, tls.crt and tls.key file paths as environment variables to make further commands re-usable.
sample-nats-box-785f8458d7-wtnfx:~# export NATS_CA=/etc/nats-certs/clients/nats-tls/ca.crt
sample-nats-box-785f8458d7-wtnfx:~# export NATS_CERT=/etc/nats-certs/clients/nats-tls/tls.crt
sample-nats-box-785f8458d7-wtnfx:~# export NATS_KEY=/etc/nats-certs/clients/nats-tls/tls.key

# delete the stream "ORDERS"
sample-nats-box-785f8458d7-wtnfx:~# nats stream rm ORDERS -f

# verify that the stream has been deleted
sample-nats-box-785f8458d7-wtnfx:~# nats stream ls
No Streams defined
sample-nats-box-785f8458d7-wtnfx:~# exit
```

#### Create RestoreSession

To restore the streams, you have to create a `RestoreSession` object pointing to the `AppBinding` of the targeted NATS server.

Here, is the YAML of the `RestoreSession` object that we are going to use for restoring the streams of the NATS server.

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: sample-nats-restore
  namespace: demo
spec:
  task:
    name: nats-restore-2.4.0
  repository:
    name: gcs-repo
  target:
    ref:
      apiVersion: appcatalog.appscode.com/v1alpha1
      kind: AppBinding
      name: sample-nats
  interimVolumeTemplate:
    metadata:
      name: nats-restore-tmp-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "standard"
      resources:
        requests:
          storage: 1Gi
  rules:
  - snapshots: [latest]
```

Here,

- `.spec.task.name` specifies the name of the Task object that specifies the necessary Functions and their execution order to restore NATS streams.
- `.spec.repository.name` specifies the Repository object that holds the backend information where our backed up data has been stored.
- `.spec.target.ref` refers to the AppBinding object that holds the connection information of our targeted NATS server.
- `.spec.interimVolumeTemplate` specifies a PVC template that will be used by Stash to hold the restored data temporarily before restoring the streams.
- `.spec.rules` specifies that we are restoring data from the latest backup snapshot of the streams.

Let's create the `RestoreSession` object object we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/addons/nats/tls/examples/restoresession.yaml
restoresession.stash.appscode.com/sample-nats-restore created
```

Once, you have created the `RestoreSession` object, Stash will create a restore Job. Run the following command to watch the phase of the `RestoreSession` object,

```bash
❯ kubectl get restoresession -n demo -w
NAME                  REPOSITORY   PHASE       DURATION   AGE
sample-nats-restore   gcs-repo     Succeeded   15s        55s
```

The `Succeeded` phase means that the restore process has been completed successfully.

#### Verify Restored Data

Now, let's exec into the nats-box pod and verify whether data actual data has been restored or not,

```
❯ kubectl exec -n demo -it sample-nats-box-785f8458d7-wtnfx -- sh -l
...
# Let's export the ca.crt, tls.crt and tls.key file paths as environment variables to make further commands re-usable.
sample-nats-box-785f8458d7-wtnfx:~# export NATS_CA=/etc/nats-certs/clients/nats-tls/ca.crt
sample-nats-box-785f8458d7-wtnfx:~# export NATS_CERT=/etc/nats-certs/clients/nats-tls/tls.crt
sample-nats-box-785f8458d7-wtnfx:~# export NATS_KEY=/etc/nats-certs/clients/nats-tls/tls.key

# Verify that the stream has been restored successfully
sample-nats-box-785f8458d7-wtnfx:~#  nats stream ls
Streams:

	ORDERS

# Verify that the messages have been restored successfully
sample-nats-box-785f8458d7-wtnfx:~# nats stream info ORDERS
Information for Stream ORDERS created 2021-09-03T07:12:07Z

Configuration:

             Subjects: ORDERS.*
     Acknowledgements: true
            Retention: File - Limits
             Replicas: 1
       Discard Policy: Old
     Duplicate Window: 2m0s
     Maximum Messages: unlimited
        Maximum Bytes: unlimited
          Maximum Age: 1y0d0h0m0s
 Maximum Message Size: unlimited
    Maximum Consumers: unlimited


Cluster Information:

                 Name: nats
               Leader: nats-0

State:

             Messages: 2
                Bytes: 98 B
             FirstSeq: 1 @ 2021-09-03T08:55:39 UTC
              LastSeq: 2 @ 2021-09-03T08:56:11 UTC
     Active Consumers: 0

sample-nats-box-785f8458d7-wtnfx:~# exit
```

Hence, we can see from the above output that the deleted data has been restored successfully from the backup.

#### Resume Backup

Since our data has been restored successfully we can now resume our usual backup process. Resume the `BackupConfiguration` using following command,

```bash
❯ kubectl patch backupconfiguration -n demo sample-nats-backup --type="merge" --patch='{"spec": {"paused": false}}'
backupconfiguration.stash.appscode.com/sample-nats-backup patched
```

Verify that the `BackupConfiguration` has been resumed,
```bash
❯ kubectl get backupconfiguration -n demo sample-nats-backup
NAME                 TASK                SCHEDULE      PAUSED   AGE
sample-nats-backup   nats-backup-2.4.0   */2 * * * *   true     2d19h
```

Here,  `false` in the `PAUSED` column means the backup has been resumed successfully. The CronJob also should be resumed now.

```bash
❯ kubectl get cronjob -n demo
NAME                               SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
stash-backup-sample-nats-backup    */2 * * * *   False     0        3m24s           4h54m
```

Here, `False` in the `SUSPEND` column means the CronJob is no longer suspended and will trigger in the next schedule.

## Cleanup

To cleanup the Kubernetes resources created by this tutorial, run:

```bash
kubectl delete -n demo backupconfiguration sample-nats-backup
kubectl delete -n demo restoresession sample-nats-restore
kubectl delete -n demo repository gcs-repo
# delete the nats chart
helm delete sample-nats -n demo
```
