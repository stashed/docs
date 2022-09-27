---
title: TLS secured NATS
description: Backup TLS secured NATS using Stash
menu:
  docs_{{ .version }}:
    identifier: stash-nats-tls-auth
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
- Install Stash Enterprise in your cluster following the steps [here](/docs/setup/install/enterprise/index.md).
- Install cert-manager in your cluster following the instruction [here](https://cert-manager.io/docs/installation/).
- If you are not familiar with how Stash backup and restore NATS streams, please check the following guide [here](/docs/addons/nats/overview/index.md).

You have to be familiar with following custom resources:

- [AppBinding](/docs/concepts/crds/appbinding/index.md)
- [Function](/docs/concepts/crds/function/index.md)
- [Task](/docs/concepts/crds/task/index.md)
- [BackupConfiguration](/docs/concepts/crds/backupconfiguration/index.md)
- [BackupSession](/docs/concepts/crds/backupsession/index.md)
- [RestoreSession](/docs/concepts/crds/restoresession/index.md)

To keep things isolated, we are going to use a separate namespace called `demo` throughout this tutorial. Create `demo` namespace if you haven't created already.

```bash
$ kubectl create ns demo
namespace/demo created
```

> Note: YAML files used in this tutorial are stored [here](https://github.com/stashed/docs/tree/{{< param "info.version" >}}/docs/addons/nats/tls/examples).

## Prepare NATS

In this section, we are going to deploy a TLS secured NATS cluster. Then, we are going to create a stream and publish some messages into it.


### Create Certificate
At first, let's create a ` ClusterIssuer` that we will be using to issue our CA certificates. Below is the YAML of `ClusterIssuer` object we are going to create.
```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: selfsigning
spec:
  selfSigned: {}
```
Let's create the `ClusterIssuer` we have shown above,
```bash
$ kubectl apply -f https://github.com/stashed/docs/tree/{{< param "info.version" >}}/docs/addons/nats/tls/examples/clusterissuer.yaml
clusterissuer.cert-manager.io/selfsigning created
```

Now, let's issue the CA certificate using the `ClusterIssuer` we have created above. Below is the YAML of `Certificate` object we are going to create.
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
certificate.cert-manager.io/nats-ca created
```

Cert-manager will automatically create a Secret named specified by `spec.secretName` field with the desired CA certificate. Let's verify that the Secret has been created successfully,

```bash
❯ kubectl get secret -n demo nats-ca
NAME      TYPE                DATA   AGE
nats-ca   kubernetes.io/tls   3      24h
```

Now, we are going create a `Issuer` with the above CA Secret. We are going to use this `Issuer` to issue server and client certificates for our NATS server. Below is the YAML of `Issuer` object we are going to create.

```yaml
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: nats-ca
  namespace: demo
spec:
  ca:
    secretName: nats-ca
```
Let's create the `Issuer` we have shown above,
```bash
$ kubectl apply -f https://github.com/stashed/docs/tree/{{< param "info.version" >}}/docs/addons/nats/tls/examples/issuer.yaml
issuer.cert-manager.io/nats-ca created
```

Now, lets create the server and client certificates. Below is the YAML of `Certificate` objects we are going to create.

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: nats-server-tls
  namespace: demo
spec:
  secretName: nats-server-tls
  duration: 2160h # 90 days
  renewBefore: 240h # 10 days
  issuerRef:
    name: nats-ca
    kind: Issuer
  commonName: sample-nats-server
  dnsNames:
  - sample-nats-tls
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: nats-client-tls
  namespace: demo
spec:
  secretName: nats-client-tls
  duration: 2160h # 90 days
  renewBefore: 240h # 10 days
  issuerRef:
    name: nats-ca
    kind: Issuer
  commonName: sample-nats-client
```
Let's create the `Certificates` we have shown above,
```bash
$ kubectl apply -f https://github.com/stashed/docs/tree/{{< param "info.version" >}}/docs/addons/nats/tls/examples/cert.yaml
certificate.cert-manager.io/nats-server-tls created
certificate.cert-manager.io/nats-client-tls created
```

Cert-manager will automatically create `nats-server-tls` and `nats-client-tls` Secrets with the desired certificates. Let's verify the Secrets have been created successfully,

```bash
❯ kubectl get secrets -n demo nats-client-tls nats-server-tls
NAME              TYPE                DATA   AGE
nats-client-tls   kubernetes.io/tls   3      24h
nats-server-tls   kubernetes.io/tls   3      24h

```

### Deploy NATS Cluster

Now, let's deploy a NATS cluster. Here, we are going to use [NATS]( https://nats-io.github.io/k8s/helm/charts/)  chart from [nats.io](https://nats.io/).

Let's deploy a NATS cluster named `sample-nats` using Helm as below,

```bash
# Add nats chart registry
$ helm repo add nats https://nats-io.github.io/k8s/helm/charts/ 
# Update helm registries
$ helm repo update
# Install nats/nats chart into demo namespace
$ helm install sample-nats-tls nats/nats -n demo \
--set nats.jetstream.enabled=true \
--set nats.jetstream.fileStorage.enabled=true \
--set nats.tls.secret.name=nats-server-tls \
--set nats.tls.ca="ca.crt" \
--set nats.tls.cert="tls.crt" \
--set nats.tls.key="tls.key" \
--set nats.tls.verify=true \
--set cluster.enabled=true \
--set cluster.replicas=3 

```

This chart will create the necessary StatefulSet, Service, PVCs etc. for the NATS cluster. You can easily view all the resources created by chart using [ketall](https://github.com/corneliusweig/ketall) `kubectl` plugin as below,

```bash
❯ kubectl get-all -n demo -l app.kubernetes.io/instance=sample-nats-tls
NAME                                                            NAMESPACE  AGE
configmap/sample-nats-tls-config                                demo       9m40s  
endpoints/sample-nats-tls                                       demo       9m40s  
persistentvolumeclaim/sample-nats-tls-js-pvc-sample-nats-tls-0  demo       9m40s  
persistentvolumeclaim/sample-nats-tls-js-pvc-sample-nats-tls-1  demo       9m17s  
persistentvolumeclaim/sample-nats-tls-js-pvc-sample-nats-tls-2  demo       8m54s  
pod/sample-nats-tls-0                                           demo       9m40s  
pod/sample-nats-tls-1                                           demo       9m17s  
pod/sample-nats-tls-2                                           demo       8m54s  
service/sample-nats-tls                                         demo       9m40s  
controllerrevision.apps/sample-nats-tls-76dfb9c75               demo       9m40s  
statefulset.apps/sample-nats-tls                                demo       9m40s  
endpointslice.discovery.k8s.io/sample-nats-tls-6lxps            demo       9m40s 

```

Now, wait for the NATS server pods `sample-nats-tls-0`, `sample-nats-tls-1`, `sample-nats-tls-2` to go into `Running` state,

```bash
❯ kubectl get pod -n demo -l app.kubernetes.io/instance=sample-nats
NAME                READY   STATUS    RESTARTS   AGE
sample-nats-tls-0   3/3     Running   0          11m
sample-nats-tls-1   3/3     Running   0          11m
sample-nats-tls-2   3/3     Running   0          11m
```

Once the pods are in `Running` state, verify that the NATS server is ready to accept the connections.

```bash
❯ kubectl logs -n demo sample-nats-tls-0 -c nats
[7] 2021/09/06 08:33:53.111508 [INF] Starting nats-server
[7] 2021/09/06 08:33:53.111560 [INF] Version:  2.6.1
...
[7] 2021/09/06 08:33:53.116004 [INF] Server is ready
```

From the above log, we can see the NATS server is ready to accept connections.

### Insert Sample Data
The above Helm chart also deploy a pod with nats-box image which can be used to interact with the NATS server. Let's verify the nats-box pod has been created.

```bash
❯ kubectl get pod -n demo -l app=sample-nats-tls-box
NAME                                   READY   STATUS    RESTARTS   AGE
sample-nats-tls-box-67fb4fb4f9-gtt9z   1/1     Running   0          13m
```

Now, we are going to exec into the nats-box pod and create some sample data, We are going to use the client certificates created in `nats-client-tls` Secret to connect with the NATS server. So, let's create the certificates files inside the nats-box pod.

At first, let's get the certificates from the `nats-client-tls` Secret,

```bash
❯ kubectl get secret -n demo  nats-client-tls -o yaml
apiVersion: v1
data:
  ca.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM4RENDQWRpZ0F3SUJBZ0lRUDZ1UXIxQVlFWnJzREF6ZHBRR09HekFOQmdrcWhraUc5dzBCQVFzRkFEQVMKTVJBd0RnWURWUVFERXdkdVlYUnpMV05oTUI0WERUSXhNRGt5TnpBMU5EWTBOVm9YRFRJeU1Ea3lOakExTkRZMApOVm93RWpFUU1BNEdBMVVFQXhNSGJtRjBjeTFqWVRDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDCkFRb0NnZ0VCQUovelJtSE1CUVdGMTNXZlJubzFsV0ZJajZmNmhyRGRVR0RTMXBrY0hmMDlqNS90bEpYSHpCbVMKZSs2YS9Qb1MrdkMyWEtyeVp3UVB0NW5BaUxXR1NxM3VBRnJ3TUJncVBBQktOa1hHL1hjamNvbU5lVTFaQlNYYgo1WmlJa2F6TUZPOGFqRWxYb3RmYnQ2cVc5MGNCTVduRW9pcnUyWkFyam50WjJpMmpPeGRodUJpRTkxamRsZWMyCkdZWGFKVlJ5RkF1eVdXanVEV3o0NjFKdXBMdXcxVWJyVHpmMExUenQxdk9ONnZNU1RQT0Z0S0tnd0RGenB5ZkgKVjFFQlZ5aG1KSk42QW13SkErZGEvMmsxMUJCeHFEWldsOEZMWE1TWUcvU0hKak5sQ3VsQTFvVVJWbFI3MVF6KwpTanB2bkxKVm9nL01sYVcvTzB5N0lRcTVQNUZGeDBNQ0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trCk1BOEdBMVVkRXdFQi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZNcm5ZN0Izek5VY1AvN3hHTzhkTFIwZVcxUnIKTUEwR0NTcUdTSWIzRFFFQkN3VUFBNElCQVFDQVZPQUZhTjRVOUQ0U1cxcWJUTDhkcWhFbklXTFd3YVBJdXJGSAo1MVVRMEUxenFTOWcvQ1gwUElOUmJ1bFpseHVKRGFBZEwweVYwYmZYZExLQnJacDNwS001eGRyaEoxQ3luNjV5CkRML0RTd3hTOHlxT3NwTXF2SkoyUTBhQ0JQTXhDRFZoOGVFZ2krOG9ISmdobkZzaTkvanNoZ0dUS09QbVVWdHcKTyszS1B0MFBiNVRDSVpJdlA1cXBybkU0U2hDWnRRZ0UyY0dJTEJPZEt5VEl6QlpuM3ZNZjc2Zjd4NU4rWEtINgpQN3Q4Yks0SUFSbzR1WUN0cDQ0K0dkY2FlcjlDL2RVNlpaMSs1Nm4xcUo3a3FTV3cwNFZqbi9CVWt5WnhIdFZPCkFLcUNCRWtnK3NBQytYUmNiOFdxTHkreEEzdmU0TmxqalE3T2MrVXVzanNrSndOVQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMrekNDQWVPZ0F3SUJBZ0lSQUlnc1laWkI2RU1MQjR6NGRnZTI2ZUV3RFFZSktvWklodmNOQVFFTEJRQXcKRWpFUU1BNEdBMVVFQXhNSGJtRjBjeTFqWVRBZUZ3MHlNVEE1TWpjd05UUTJORGxhRncweU1URXlNall3TlRRMgpORGxhTUIweEd6QVpCZ05WQkFNVEVuTmhiWEJzWlMxdVlYUnpMV05zYVdWdWREQ0NBU0l3RFFZSktvWklodmNOCkFRRUJCUUFEZ2dFUEFEQ0NBUW9DZ2dFQkFOZEdselNWRTBGS1cyY1M5V3VqZmk1Wk5oY3I3dXJsUjZZSGR1dkwKQ2ZDRGdEZTNBUVBpWG92YzI4elgxY3d2cVVQU3l2WXpGUFN4dUtVRXpoLzBGZE5kMGk2SkVRQUp4dVgwU2JSdwpjYXZXMVR5MkZFWExtYTNiMnBWVWJ6dUE1VVdGQzFwd3hZVCsvcERHNmI4YnFuQVJiaFNvdUowQUoxTGNGT08zCjg1V1RtWUFDRHY4dmFyRFQvM0xmYWtndXJqYWc4SWdMMURyd2hxNFNjRllveElYbXJJZjhMTVVERkN4Y251aE0KNnIraUl4OXFhWkJjMys2eU4yNXNvc2J6ZDlXbXRlT3J3Z2pKUzRLdU9ZaWl2VzBsSDNzQTlOSG9HU3cwSDVtWgpjcndsMHZxV0JvaWFoMXdWSjM5S1NrWlRvLzhUaGpMQUV5QUZKdzRuNzk1eHFwOENBd0VBQWFOQk1EOHdEZ1lEClZSMFBBUUgvQkFRREFnV2dNQXdHQTFVZEV3RUIvd1FDTUFBd0h3WURWUjBqQkJnd0ZvQVV5dWRqc0hmTTFSdy8KL3ZFWTd4MHRIUjViVkdzd0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFGRTE0OTJkdS9HSVpKS3BuMEdRNE1STApHbDI5bXc1Wi9nUWxwTTN3NGdYU1hxbmNGczREcHJFd2g5R25PTkEzcXpta1NIakFwWmwwQzdWZi84Y2RnNS8zCk90UVZSSGkxQVJFcGFHMlVUMnFJSXp1SUVLN0tRZE5maXpVYVVaMFgzb041Kyt4YWU4WSsxa3dZOXZxaXdWRlcKbS96T1JzSmRtcnRqNGZRSTVaVGRzVG1jRkxqUXBOcktOSWFVU2pHOFM1Q3pOMlJBekZHTTBCZWYzSWFzWTF2WQpDTWpOZlBxaUNtZFNlOFFxRE1UMURwRExjaFltQlQ3UjdyR0JBaEFXWEpEZlVMRXlXUk9XdmRrWlhnTk5ZMnlJCkhmbktUTy9TL1FpUUs4N0Y2SWtEM2tKalZPVDhUNVBmYjBwYTJnaDlnSkZmZUJPVW9FUkYrdG4zYWI3Z3BkQT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
  tls.key: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb2dJQkFBS0NBUUVBMTBhWE5KVVRRVXBiWnhMMWE2TitMbGsyRnl2dTZ1VkhwZ2QyNjhzSjhJT0FON2NCCkErSmVpOXpiek5mVnpDK3BROUxLOWpNVTlMRzRwUVRPSC9RVjAxM1NMb2tSQUFuRzVmUkp0SEJ4cTliVlBMWVUKUmN1WnJkdmFsVlJ2TzREbFJZVUxXbkRGaFA3K2tNYnB2eHVxY0JGdUZLaTRuUUFuVXR3VTQ3ZnpsWk9aZ0FJTwoveTlxc05QL2N0OXFTQzZ1TnFEd2lBdlVPdkNHcmhKd1ZpakVoZWFzaC93c3hRTVVMRnllNkV6cXY2SWpIMnBwCmtGemY3ckkzYm15aXh2TjMxYWExNDZ2Q0NNbExncTQ1aUtLOWJTVWZld0QwMGVnWkxEUWZtWmx5dkNYUytwWUcKaUpxSFhCVW5mMHBLUmxPai94T0dNc0FUSUFVbkRpZnYzbkdxbndJREFRQUJBb0lCQUZWZ0NvRnhDY1RmLzJYZQpiL1J6VDR5RUZ0NlRydG43ZWpIUFRndHZaNDY2S0RSd1lIZXc0L3dsNkFuU0kxa3FJYi9qTGxqN296ano3cDJMClRWQUExbE1RSjFZTFIvR3k3dTJ0dHpsWFNzMXlrdmpUNFRCWThhYXd4WHhwa3YrUE85NFpTSXBpcFFMOHVlcWkKNkhyQk54UGc1YjVOdDRHVVdRUVVnamhaY01JRm9DUUwrRmNkZGs4RkQ1UFkxNnprWDNReUpITlVkZXRoYmJKdQoveVAvMTk3b2l6aVdzbWJSQ3Z6Z3Q2bEtOUk51VjZJRitMdnM4RWxGUlAyeStLcmI2SXRjT3lwRjY5TGE1N1pZCjY0enJWSVJnc2FYZVBTTWpGVkN1ZHFDY2QrL0FEeFU3YVc2TzhFaXJBQ0pqcXFYcnBYamd3MVNUY21WR2ZYK1gKQithRjIza0NnWUVBNXFsYnJGaHhWQlZBVTlMR3JnVGNLZXRGUEJveTQyOWxVTnJXMU5xSUxEaExZVjgxRGZIQQpXQVdYK1l0NGJUTXZqTUd2TDhDM0pXL0NiNDNHSDhmU2lwUTZDUlR2dDBYNXlyNytWL0tVdWdpKzV4ZmVoMlIzCldiRUNBVWNKM0UxVmtvRkN2ZEFtbHZBMlNROGlRVHNkV3JuclJxWFhEVkFGQzlCNEtZQ0JiWk1DZ1lFQTd1eU4KNVVGMkg0dmZmSWtZQUZzS0k0YlM4blQ5UGw4dmNPeUNoTUFLNUlnSHQyQUk0RTRVa20zeHFiblV0cDdjdHoySgplbUxJaTJ3M2pVMXdjWittc0pIYnRyMmxDdGVMNjJjMENLYXVsaDA1YWhLZ0VZUjhlVzcyL1F6Skg1WDhPTkZsCkx4eW9vRUo0Vmo2T1VwcTZKZmJTUW03YUFKMWE1dVBHUzhZWmxrVUNnWUJhcm5oSThGaFZpeWxJQ3hScTg2UXUKb3IwTVhPeG10N09vTHZESXE4VmZSUjUxZ0gyV0p0WE1oUjV6VDk2Zlo4RW80RGhrV0twb0FHRDdoRXhBMEVrNAppLytvOUY4dHVVZno2bFNKOU9kOW45U1ZlNi9Ub0s2L1J6U1hsZnNOYmlYWFBCUW1GWUFtVlBleWowMlRRWTlQCnpNbnZjMkZ4YldVZWVPM1V1eDJuR3dLQmdCQkFreDVuSjR2WnplZ0F3MXN5MWl1NGZoejBERTN6MTV4TTJrd0IKYkR4RGJKTHl1MmZXcDl1V0V2eENvYytTV3QwMEdHZjAxRU4zcHdlN25zeDcyYkRsR3hjQkszcmpVcWMrcS9GeQp0U21NNzF6aHkzV2xsM29ETEZYbVNzQVZTY1RycVlCYzZMT09FZlY3NTk2Q20rcjlNU3hIc2hpY201UmRKaDM5CmFid3BBb0dBQmZKeC91cXk3RGpubnZPLzUxMW1CSEh5bS9pdG9TNzNVL2FOd1pqWmhyWlpzZGVxUUZNcm5xSTQKQU83S05ldE9oa3NmQnVndTg5Q3dXTjRYMkpYWjd1aFFlVWlKQWNhNFc4ZmJLdTNjYytsZUVMTisrVEZ5Um91MgphRlpRYnZSazdYU01nM3d0ekk5SUtoNzIyZXRPVXJPb0FaNWthRlcrRWt2WjBoNmlTTzg9Ci0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==
kind: Secret
metadata:
  annotations:
    cert-manager.io/alt-names: ""
    cert-manager.io/certificate-name: nats-client-tls
    cert-manager.io/common-name: sample-nats-client
    cert-manager.io/ip-sans: ""
    cert-manager.io/issuer-group: ""
    cert-manager.io/issuer-kind: Issuer
    cert-manager.io/issuer-name: nats-ca
    cert-manager.io/uri-sans: ""
  creationTimestamp: "2021-09-27T05:46:49Z"
  name: nats-client-tls
  namespace: demo
  resourceVersion: "386072"
  uid: 89931f0b-aa0d-499c-b7f9-d3b6ada4ab08
type: kubernetes.io/tls
```

Now, let's create `tls.crt` and `tls.key` files in the local machine,

```bash
❯ echo LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMrekNDQWVPZ0F3SUJBZ0lSQUlnc1laWkI2RU1MQjR6NGRnZTI2ZUV3RFFZSktvWklodmNOQVFFTEJRQXcKRWpFUU1BNEdBMVVFQXhNSGJtRjBjeTFqWVRBZUZ3MHlNVEE1TWpjd05UUTJORGxhRncweU1URXlNall3TlRRMgpORGxhTUIweEd6QVpCZ05WQkFNVEVuTmhiWEJzWlMxdVlYUnpMV05zYVdWdWREQ0NBU0l3RFFZSktvWklodmNOCkFRRUJCUUFEZ2dFUEFEQ0NBUW9DZ2dFQkFOZEdselNWRTBGS1cyY1M5V3VqZmk1Wk5oY3I3dXJsUjZZSGR1dkwKQ2ZDRGdEZTNBUVBpWG92YzI4elgxY3d2cVVQU3l2WXpGUFN4dUtVRXpoLzBGZE5kMGk2SkVRQUp4dVgwU2JSdwpjYXZXMVR5MkZFWExtYTNiMnBWVWJ6dUE1VVdGQzFwd3hZVCsvcERHNmI4YnFuQVJiaFNvdUowQUoxTGNGT08zCjg1V1RtWUFDRHY4dmFyRFQvM0xmYWtndXJqYWc4SWdMMURyd2hxNFNjRllveElYbXJJZjhMTVVERkN4Y251aE0KNnIraUl4OXFhWkJjMys2eU4yNXNvc2J6ZDlXbXRlT3J3Z2pKUzRLdU9ZaWl2VzBsSDNzQTlOSG9HU3cwSDVtWgpjcndsMHZxV0JvaWFoMXdWSjM5S1NrWlRvLzhUaGpMQUV5QUZKdzRuNzk1eHFwOENBd0VBQWFOQk1EOHdEZ1lEClZSMFBBUUgvQkFRREFnV2dNQXdHQTFVZEV3RUIvd1FDTUFBd0h3WURWUjBqQkJnd0ZvQVV5dWRqc0hmTTFSdy8KL3ZFWTd4MHRIUjViVkdzd0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFGRTE0OTJkdS9HSVpKS3BuMEdRNE1STApHbDI5bXc1Wi9nUWxwTTN3NGdYU1hxbmNGczREcHJFd2g5R25PTkEzcXpta1NIakFwWmwwQzdWZi84Y2RnNS8zCk90UVZSSGkxQVJFcGFHMlVUMnFJSXp1SUVLN0tRZE5maXpVYVVaMFgzb041Kyt4YWU4WSsxa3dZOXZxaXdWRlcKbS96T1JzSmRtcnRqNGZRSTVaVGRzVG1jRkxqUXBOcktOSWFVU2pHOFM1Q3pOMlJBekZHTTBCZWYzSWFzWTF2WQpDTWpOZlBxaUNtZFNlOFFxRE1UMURwRExjaFltQlQ3UjdyR0JBaEFXWEpEZlVMRXlXUk9XdmRrWlhnTk5ZMnlJCkhmbktUTy9TL1FpUUs4N0Y2SWtEM2tKalZPVDhUNVBmYjBwYTJnaDlnSkZmZUJPVW9FUkYrdG4zYWI3Z3BkQT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo= | base64 -d > /tmp/tls.crt
❯ echo LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb2dJQkFBS0NBUUVBMTBhWE5KVVRRVXBiWnhMMWE2TitMbGsyRnl2dTZ1VkhwZ2QyNjhzSjhJT0FON2NCCkErSmVpOXpiek5mVnpDK3BROUxLOWpNVTlMRzRwUVRPSC9RVjAxM1NMb2tSQUFuRzVmUkp0SEJ4cTliVlBMWVUKUmN1WnJkdmFsVlJ2TzREbFJZVUxXbkRGaFA3K2tNYnB2eHVxY0JGdUZLaTRuUUFuVXR3VTQ3ZnpsWk9aZ0FJTwoveTlxc05QL2N0OXFTQzZ1TnFEd2lBdlVPdkNHcmhKd1ZpakVoZWFzaC93c3hRTVVMRnllNkV6cXY2SWpIMnBwCmtGemY3ckkzYm15aXh2TjMxYWExNDZ2Q0NNbExncTQ1aUtLOWJTVWZld0QwMGVnWkxEUWZtWmx5dkNYUytwWUcKaUpxSFhCVW5mMHBLUmxPai94T0dNc0FUSUFVbkRpZnYzbkdxbndJREFRQUJBb0lCQUZWZ0NvRnhDY1RmLzJYZQpiL1J6VDR5RUZ0NlRydG43ZWpIUFRndHZaNDY2S0RSd1lIZXc0L3dsNkFuU0kxa3FJYi9qTGxqN296ano3cDJMClRWQUExbE1RSjFZTFIvR3k3dTJ0dHpsWFNzMXlrdmpUNFRCWThhYXd4WHhwa3YrUE85NFpTSXBpcFFMOHVlcWkKNkhyQk54UGc1YjVOdDRHVVdRUVVnamhaY01JRm9DUUwrRmNkZGs4RkQ1UFkxNnprWDNReUpITlVkZXRoYmJKdQoveVAvMTk3b2l6aVdzbWJSQ3Z6Z3Q2bEtOUk51VjZJRitMdnM4RWxGUlAyeStLcmI2SXRjT3lwRjY5TGE1N1pZCjY0enJWSVJnc2FYZVBTTWpGVkN1ZHFDY2QrL0FEeFU3YVc2TzhFaXJBQ0pqcXFYcnBYamd3MVNUY21WR2ZYK1gKQithRjIza0NnWUVBNXFsYnJGaHhWQlZBVTlMR3JnVGNLZXRGUEJveTQyOWxVTnJXMU5xSUxEaExZVjgxRGZIQQpXQVdYK1l0NGJUTXZqTUd2TDhDM0pXL0NiNDNHSDhmU2lwUTZDUlR2dDBYNXlyNytWL0tVdWdpKzV4ZmVoMlIzCldiRUNBVWNKM0UxVmtvRkN2ZEFtbHZBMlNROGlRVHNkV3JuclJxWFhEVkFGQzlCNEtZQ0JiWk1DZ1lFQTd1eU4KNVVGMkg0dmZmSWtZQUZzS0k0YlM4blQ5UGw4dmNPeUNoTUFLNUlnSHQyQUk0RTRVa20zeHFiblV0cDdjdHoySgplbUxJaTJ3M2pVMXdjWittc0pIYnRyMmxDdGVMNjJjMENLYXVsaDA1YWhLZ0VZUjhlVzcyL1F6Skg1WDhPTkZsCkx4eW9vRUo0Vmo2T1VwcTZKZmJTUW03YUFKMWE1dVBHUzhZWmxrVUNnWUJhcm5oSThGaFZpeWxJQ3hScTg2UXUKb3IwTVhPeG10N09vTHZESXE4VmZSUjUxZ0gyV0p0WE1oUjV6VDk2Zlo4RW80RGhrV0twb0FHRDdoRXhBMEVrNAppLytvOUY4dHVVZno2bFNKOU9kOW45U1ZlNi9Ub0s2L1J6U1hsZnNOYmlYWFBCUW1GWUFtVlBleWowMlRRWTlQCnpNbnZjMkZ4YldVZWVPM1V1eDJuR3dLQmdCQkFreDVuSjR2WnplZ0F3MXN5MWl1NGZoejBERTN6MTV4TTJrd0IKYkR4RGJKTHl1MmZXcDl1V0V2eENvYytTV3QwMEdHZjAxRU4zcHdlN25zeDcyYkRsR3hjQkszcmpVcWMrcS9GeQp0U21NNzF6aHkzV2xsM29ETEZYbVNzQVZTY1RycVlCYzZMT09FZlY3NTk2Q20rcjlNU3hIc2hpY201UmRKaDM5CmFid3BBb0dBQmZKeC91cXk3RGpubnZPLzUxMW1CSEh5bS9pdG9TNzNVL2FOd1pqWmhyWlpzZGVxUUZNcm5xSTQKQU83S05ldE9oa3NmQnVndTg5Q3dXTjRYMkpYWjd1aFFlVWlKQWNhNFc4ZmJLdTNjYytsZUVMTisrVEZ5Um91MgphRlpRYnZSazdYU01nM3d0ekk5SUtoNzIyZXRPVXJPb0FaNWthRlcrRWt2WjBoNmlTTzg9Ci0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg== | base64 -d > /tmp/tls.key
```

Then, let's copy these files from local machine to nats-box pod,

```bash
❯ kubectl cp -n demo /tmp/tls.crt sample-nats-box-785f8458d7-wtnfx:/tmp/tls.crt
❯ kubectl cp -n demo /tmp/tls.key sample-nats-box-785f8458d7-wtnfx:/tmp/tls.key
```

Finally, Let's exec into the nats-box pod,

```bash
❯ kubectl exec -n demo sample-nats-tls-box-67fb4fb4f9-gtt9z  -it -- sh -l
...
# Let's export the tls.crt and tls.key file paths as environment variables to make further commands re-usable.
sample-nats-tls-box-67fb4fb4f9-gtt9z:~# export NATS_CERT=/tmp/tls.crt
sample-nats-tls-box-67fb4fb4f9-gtt9z:~# export NATS_KEY=/tmp/tls.key

# Let's create a stream named "ORDERS"
sample-nats-tls-box-67fb4fb4f9-gtt9z:~# nats stream add ORDERS --subjects "ORDERS.*" --ack --max-msgs=-1 --max-bytes=-1 --max-age=1y --storage file --retention limits --max-msg-size=-1 --max-msgs-per-subject=-1 --discard old --dupe-window="0s" --replicas 1
Stream ORDERS was created

Information for Stream ORDERS created 2021-09-27T06:27:30Z

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
               Leader: sample-nats-tls-2

State:

             Messages: 0
                Bytes: 0 B
             FirstSeq: 0
              LastSeq: 0
     Active Consumers: 0
     
# Verify that the stream has been created successfully
sample-nats-tls-box-67fb4fb4f9-gtt9z:~# nats stream ls
Streams:

        ORDERS
        
# Lets add some messages to the stream "ORDERS"
sample-nats-tls-box-67fb4fb4f9-gtt9z:~# nats pub ORDERS.scratch hello
06:29:18 Published 5 bytes to "ORDERS.scratch"

# Add another message
ample-nats-tls-box-67fb4fb4f9-gtt9z:~# nats pub ORDERS.scratch world
06:29:41 Published 5 bytes to "ORDERS.scratch"

# Verify that the messages have been published to the stream successfully
sample-nats-tls-box-67fb4fb4f9-gtt9z:~# nats stream info ORDERS
Information for Stream ORDERS created 2021-09-27T06:27:30Z

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
               Leader: sample-nats-tls-2

State:

             Messages: 2
                Bytes: 98 B
             FirstSeq: 1 @ 2021-09-27T06:29:18 UTC
              LastSeq: 2 @ 2021-09-27T06:29:41 UTC
     Active Consumers: 0

sample-nats-tls-box-67fb4fb4f9-gtt9z:~# exit
```

We have successfully deployed a NATS cluster, created a stream and publish some messages into the stream. In the subsequent sections, we are going to backup this sample data using Stash.

## Prepare for Backup

In this section, we are going to prepare the necessary resources (i.e. connection information, backend information, etc.) before backup.

### Ensure NATS Addon

When you install Stash Enterprise version, it will automatically install all the official addons. Make sure that NATS addon was installed properly using the following command.

```bash
❯ kubectl get tasks.stash.appscode.com | grep nats
nats-backup-2.6.1            24m
nats-restore-2.6.1           24m
```

This addon should be able to take backup of the NATS streams with matching major versions as discussed in [Addon Version Compatibility](/docs/addons/nats/README.md#addon-version-compatibility).


### Create AppBinding

Stash needs to know how to connect with the NATS server. An `AppBinding` exactly provides this information. It holds the Service and Secret information of the NATS server. You have to point to the respective `AppBinding` as a target of backup instead of the NATS server itself.

Here, is the YAML of the `AppBinding` that we are going to create for the NATS server we have deployed earlier.

```yaml
apiVersion: appcatalog.appscode.com/v1alpha1
kind: AppBinding
metadata:
  labels:
    app.kubernetes.io/instance: sample-nats-tls
  name: sample-nats-tls
  namespace: demo
spec:
  clientConfig:
    caBundle: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM4RENDQWRpZ0F3SUJBZ0lRUDZ1UXIxQVlFWnJzREF6ZHBRR09HekFOQmdrcWhraUc5dzBCQVFzRkFEQVMKTVJBd0RnWURWUVFERXdkdVlYUnpMV05oTUI0WERUSXhNRGt5TnpBMU5EWTBOVm9YRFRJeU1Ea3lOakExTkRZMApOVm93RWpFUU1BNEdBMVVFQXhNSGJtRjBjeTFqWVRDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDCkFRb0NnZ0VCQUovelJtSE1CUVdGMTNXZlJubzFsV0ZJajZmNmhyRGRVR0RTMXBrY0hmMDlqNS90bEpYSHpCbVMKZSs2YS9Qb1MrdkMyWEtyeVp3UVB0NW5BaUxXR1NxM3VBRnJ3TUJncVBBQktOa1hHL1hjamNvbU5lVTFaQlNYYgo1WmlJa2F6TUZPOGFqRWxYb3RmYnQ2cVc5MGNCTVduRW9pcnUyWkFyam50WjJpMmpPeGRodUJpRTkxamRsZWMyCkdZWGFKVlJ5RkF1eVdXanVEV3o0NjFKdXBMdXcxVWJyVHpmMExUenQxdk9ONnZNU1RQT0Z0S0tnd0RGenB5ZkgKVjFFQlZ5aG1KSk42QW13SkErZGEvMmsxMUJCeHFEWldsOEZMWE1TWUcvU0hKak5sQ3VsQTFvVVJWbFI3MVF6KwpTanB2bkxKVm9nL01sYVcvTzB5N0lRcTVQNUZGeDBNQ0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trCk1BOEdBMVVkRXdFQi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZNcm5ZN0Izek5VY1AvN3hHTzhkTFIwZVcxUnIKTUEwR0NTcUdTSWIzRFFFQkN3VUFBNElCQVFDQVZPQUZhTjRVOUQ0U1cxcWJUTDhkcWhFbklXTFd3YVBJdXJGSAo1MVVRMEUxenFTOWcvQ1gwUElOUmJ1bFpseHVKRGFBZEwweVYwYmZYZExLQnJacDNwS001eGRyaEoxQ3luNjV5CkRML0RTd3hTOHlxT3NwTXF2SkoyUTBhQ0JQTXhDRFZoOGVFZ2krOG9ISmdobkZzaTkvanNoZ0dUS09QbVVWdHcKTyszS1B0MFBiNVRDSVpJdlA1cXBybkU0U2hDWnRRZ0UyY0dJTEJPZEt5VEl6QlpuM3ZNZjc2Zjd4NU4rWEtINgpQN3Q4Yks0SUFSbzR1WUN0cDQ0K0dkY2FlcjlDL2RVNlpaMSs1Nm4xcUo3a3FTV3cwNFZqbi9CVWt5WnhIdFZPCkFLcUNCRWtnK3NBQytYUmNiOFdxTHkreEEzdmU0TmxqalE3T2MrVXVzanNrSndOVQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    service:
      name: sample-nats-tls
      port: 4222
      scheme: nats
  secret:
    name: nats-client-tls
  type: nats.io/nats
  version: 2.6.1
```

Here,

- `.spec.clientConfig.caBundle` specifies a PEM encoded CA bundle which will be used to validate the serving certificate of the NATS server.
- `.spec.clientConfig.service` specifies the Service information to use to connects with the NATS server.
- `.spec.secret` specifies the name of the Secret that holds necessary credentials to access the server.
- `.spec.type` specifies the type of the target. This is particularly helpful in auto-backup where you want to use different path prefixes for different types of target.

Let's create the `AppBinding` we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/tree/{{< param "info.version" >}}/docs/addons/nats/tls/examples/appbinding.yaml
appbinding.appcatalog.appscode.com/sample-nats-tls created
```

### Prepare Backend

We are going to store our backed up data into a GCS bucket. So, we need to create a Secret with GCS credentials and a `Repository` object with the bucket information. If you want to use a different backend, please read the respective backend configuration doc from [here](/docs/guides/backends/overview/index.md).

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
      prefix: /demo/nats/sample-nats-tls
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
  name: sample-nats-backup-tls
  namespace: demo
spec:
  task:
    name: nats-backup-2.6.1
  schedule: "*/5 * * * *"
  repository:
    name: gcs-repo
  target:
    ref:
      apiVersion: appcatalog.appscode.com/v1alpha1
      kind: AppBinding
      name: sample-nats-tls
  interimVolumeTemplate:
    metadata:
      name: nats-backup-tmp-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "standard"
      resources:
        requests:
          storage: 1Gi
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
- `.spec.retentionPolicy` specifies a policy indicating how we want to cleanup the old backups.

Let's create the `BackupConfiguration` object we have shown above,

```bash
$ kubectl create -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/addons/nats/tls/examples/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/sample-nats-backup-tls created
```

#### Verify Backup Setup Successful

If everything goes well, the phase of the `BackupConfiguration` should be `Ready`. The `Ready` phase indicates that the backup setup is successful. Let's verify the `Phase` of the BackupConfiguration,

```bash
$ kubectl get backupconfiguration -n demo
NAME                     TASK                    SCHEDULE      PAUSED   PHASE      AGE
sample-nats-backup-tls   nats-backup-2.6.1       */5 * * * *            Ready      11s
```

#### Verify CronJob

Stash will create a CronJob with the schedule specified in `spec.schedule` field of `BackupConfiguration` object.

Verify that the CronJob has been created using the following command,

```bash
❯ kubectl get cronjob -n demo
NAME                                  SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
stash-backup-sample-nats-backup-tls   */5 * * * *   False     0        <none>          14s
```

#### Wait for BackupSession

The `stash-sample-nats-backup` CronJob will trigger a backup on each scheduled slot by creating a `BackupSession` object.

Now, wait for a schedule to appear. Run the following command to watch for `BackupSession` object,

```bash
❯ kubectl get backupsession -n demo -w
NAME                           INVOKER-TYPE          INVOKER-NAME             PHASE       DURATION   AGE
sample-nats-backup-tls-prszs   BackupConfiguration   sample-nats-backup-tls   Succeeded   35s        84s
```

Here, the phase `Succeeded` means that the backup process has been completed successfully.

#### Verify Backup

Now, we are going to verify whether the backed up data is present in the backend or not. Once a backup is completed, Stash will update the respective `Repository` object to reflect the backup completion. Check that the repository `gcs-repo` has been updated by the following command,

```bash
❯ kubectl get repository -n demo
NAME         INTEGRITY   SIZE        SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
gcs-repo     true        4.156 KiB   3                2m2s                     97m
```

Now, if we navigate to the GCS bucket, we will see the backed up data has been stored in `demo/nats/sample-nats-tls` directory as specified by `.spec.backend.gcs.prefix` field of the `Repository` object.

<figure align="center">
  <img alt="Backup data in GCS Bucket" src="/docs/addons/nats/tls/images/sample-nats-backup.png">
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
$ kubectl patch backupconfiguration -n demo sample-nats-backup-tls --type="merge" --patch='{"spec": {"paused": true}}'
```

Verify that the `BackupConfiguration` has been paused,

```bash
❯ kubectl get backupconfiguration -n demo sample-nats-backup-tls
NAME                     TASK                SCHEDULE      PAUSED   PHASE   AGE
sample-nats-backup-tls   nats-backup-2.6.1   */5 * * * *   true     Ready   4m26s
```

Notice the `PAUSED` column. Value `true` for this field means that the `BackupConfiguration` has been paused.

Stash will also suspend the respective CronJob.

```bash
❯ kubectl get cronjob -n demo
NAME                                  SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
stash-backup-sample-nats-backup-tls   */5 * * * *   True      0        2m12s           5m4s
```

#### Simulate Disaster

Now, let's simulate a disaster scenario. Here, we are going to exec into the nats-box pod and delete the sample data we have inserted earlier.

```bash
❯ kubectl exec -n demo sample-nats-tls-box-67fb4fb4f9-gtt9z  -it -- sh -l
...
# Let's export the tls.crt and tls.key file paths as environment variables to make further commands re-usable.
sample-nats-tls-box-67fb4fb4f9-gtt9z:~# export NATS_CERT=/tmp/tls.crt
sample-nats-tls-box-67fb4fb4f9-gtt9z:~# export NATS_KEY=/tmp/tls.key

# delete the stream "ORDERS"
sample-nats-tls-box-67fb4fb4f9-gtt9z:~# nats stream rm ORDERS -f

# verify that the stream has been deleted
sample-nats-tls-box-67fb4fb4f9-gtt9z:~# nats stream ls
No Streams defined

sample-nats-tls-box-67fb4fb4f9-gtt9z:~# exit
```

#### Create RestoreSession

To restore the streams, you have to create a `RestoreSession` object pointing to the `AppBinding` of the targeted NATS server.

Here, is the YAML of the `RestoreSession` object that we are going to use for restoring the streams of the NATS server.

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: sample-nats-restore-tls
  namespace: demo
spec:
  task:
    name: nats-restore-2.6.1
  repository:
    name: gcs-repo
  target:
    ref:
      apiVersion: appcatalog.appscode.com/v1alpha1
      kind: AppBinding
      name: sample-nats-tls
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
- `.spec.interimVolumeTemplate` specifies a PVC template that will be used by Stash to hold the restored data temporarily before injecting into the NATS server.
- `.spec.rules` specifies that we are restoring data from the latest backup snapshot of the streams.

Let's create the `RestoreSession` object object we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/addons/nats/tls/examples/restoresession.yaml
restoresession.stash.appscode.com/sample-nats-restore-tls created
```

Once, you have created the `RestoreSession` object, Stash will create a restore Job. Run the following command to watch the phase of the `RestoreSession` object,

```bash
❯ kubectl get restoresession -n demo -w
NAME                      REPOSITORY   PHASE       DURATION   AGE
sample-nats-restore-tls   gcs-repo     Succeeded   15s        55s
```

The `Succeeded` phase means that the restore process has been completed successfully.

#### Verify Restored Data

Now, let's exec into the nats-box pod and verify whether data actual data has been restored or not,

```bash
❯ kubectl exec -n demo sample-nats-tls-box-67fb4fb4f9-gtt9z  -it -- sh -l
...
# Let's export the tls.crt and tls.key file paths as environment variables to make further commands re-usable.
sample-nats-tls-box-67fb4fb4f9-gtt9z:~# export NATS_CERT=/tmp/tls.crt
sample-nats-tls-box-67fb4fb4f9-gtt9z:~# export NATS_KEY=/tmp/tls.key

# Verify that the stream has been restored successfully
sample-nats-tls-box-67fb4fb4f9-gtt9z:~# nats str ls
Streams:

        ORDERS

# Verify that the messages have been restored successfully
sample-nats-tls-box-67fb4fb4f9-gtt9z:~# nats stream info ORDERS
Information for Stream ORDERS created 2021-09-27T08:23:58Z

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
               Leader: sample-nats-tls-2

State:

             Messages: 2
                Bytes: 98 B
             FirstSeq: 1 @ 2021-09-27T06:29:18 UTC
              LastSeq: 2 @ 2021-09-27T06:29:41 UTC
     Active Consumers: 0
     
sample-nats-tls-box-67fb4fb4f9-gtt9z:~# exit
```

Hence, we can see from the above output that the deleted data has been restored successfully from the backup.

#### Resume Backup

Since our data has been restored successfully we can now resume our usual backup process. Resume the `BackupConfiguration` using following command,

```bash
❯ kubectl patch backupconfiguration -n demo sample-nats-backup-tls --type="merge" --patch='{"spec": {"paused": false}}'
backupconfiguration.stash.appscode.com/sample-nats-backup-tls patched
```

Verify that the `BackupConfiguration` has been resumed,
```bash
❯ kubectl get backupconfiguration -n demo sample-nats-backup-tls
NAME                     TASK                SCHEDULE      PAUSED   PHASE   AGE
sample-nats-backup-tls   nats-backup-2.6.1   */5 * * * *   false    Ready   16m
```

Here,  `false` in the `PAUSED` column means the backup has been resumed successfully. The CronJob also should be resumed now.

```bash
❯ kubectl get cronjob -n demo
NAME                                  SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
stash-backup-sample-nats-backup-tls   */5 * * * *   False     0        23s             17m
```

Here, `False` in the `SUSPEND` column means the CronJob is no longer suspended and will trigger in the next schedule.

## Cleanup

To cleanup the Kubernetes resources created by this tutorial, run:

```bash
kubectl delete -n demo backupconfiguration sample-nats-backup-tls
kubectl delete -n demo restoresession sample-nats-restore-tls
kubectl delete -n demo repository gcs-repo
# delete the nats chart
helm delete sample-nats-tls -n demo
```
