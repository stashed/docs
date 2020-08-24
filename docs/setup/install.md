---
title: Install
description: Stash Install
menu:
  docs_{{ .version }}:
    identifier: install-stash
    name: Install
    parent: setup
    weight: 10
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: setup
---

# Installation Guide

Stash operator can be installed via a script or as a Helm chart.

<ul class="nav nav-tabs" id="installerTab" role="tablist">
  <li class="nav-item">
    <a class="nav-link active" id="helm3-tab" data-toggle="tab" href="#helm3" role="tab" aria-controls="helm3" aria-selected="true">Helm 3 (Recommended)</a>
  </li>
  <li class="nav-item">
    <a class="nav-link" id="helm2-tab" data-toggle="tab" href="#helm2" role="tab" aria-controls="helm2" aria-selected="false">Helm 2</a>
  </li>
  <li class="nav-item">
    <a class="nav-link" id="script-tab" data-toggle="tab" href="#script" role="tab" aria-controls="script" aria-selected="false">YAML</a>
  </li>
</ul>
<div class="tab-content" id="installerTabContent">
  <div class="tab-pane fade show active" id="helm3" role="tabpanel" aria-labelledby="helm3-tab">

## Using Helm 3

Stash can be installed via [Helm](https://helm.sh/) using the [chart](https://github.com/stashed/installer/tree/{{< param "info.version" >}}/charts/stash) from [AppsCode Charts Repository](https://github.com/appscode/charts). To install the chart with the release name `stash-operator`:

```console
$ helm repo add appscode https://charts.appscode.com/stable/
$ helm repo update
$ helm search repo appscode/stash --version {{< param "info.version" >}}
NAME            CHART VERSION APP VERSION DESCRIPTION
appscode/stash  {{< param "info.version" >}}    {{< param "info.version" >}}  Stash by AppsCode - Backup your Kubernetes Volumes

$ helm install stash-operator appscode/stash \
  --version {{< param "info.version" >}} \
  --namespace kube-system
```

To see the detailed configuration options, visit [here](https://github.com/stashed/installer/tree/{{< param "info.version" >}}/charts/stash).

</div>
<div class="tab-pane fade" id="helm2" role="tabpanel" aria-labelledby="helm2-tab">

## Using Helm 2

Stash can be installed via [Helm](https://helm.sh/) using the [chart](https://github.com/stashed/installer/tree/{{< param "info.version" >}}/charts/stash) from [AppsCode Charts Repository](https://github.com/appscode/charts). To install the chart with the release name `stash-operator`:

```console
$ helm repo add appscode https://charts.appscode.com/stable/
$ helm repo update
$ helm search appscode/stash --version {{< param "info.version" >}}
NAME            CHART VERSION APP VERSION DESCRIPTION
appscode/stash  {{< param "info.version" >}}    {{< param "info.version" >}}  Stash by AppsCode - Backup your Kubernetes Volumes

$ helm install appscode/stash --name stash-operator \
  --version {{< param "info.version" >}} \
  --namespace kube-system
```

To see the detailed configuration options, visit [here](https://github.com/stashed/installer/tree/{{< param "info.version" >}}/charts/stash).

</div>
<div class="tab-pane fade" id="script" role="tabpanel" aria-labelledby="script-tab">

## Using YAML

If you prefer to not use Helm, you can generate YAMLs from Stash chart and deploy using `kubectl`. Here we are going to show the prodecure using Helm 3.

```console
$ helm repo add appscode https://charts.appscode.com/stable/
$ helm repo update
$ helm search repo appscode/stash --version {{< param "info.version" >}}
NAME            CHART VERSION APP VERSION DESCRIPTION
appscode/stash  {{< param "info.version" >}}    {{< param "info.version" >}}  Stash by AppsCode - Backup your Kubernetes Volumes

$ helm template stash-operator appscode/stash \
  --version {{< param "info.version" >}} \
  --namespace kube-system \
  --no-hooks | kubectl apply -f -
```

To see the detailed configuration options, visit [here](https://github.com/stashed/installer/tree/{{< param "info.version" >}}/charts/stash).

</div>
</div>

### Installing in GKE Cluster

If you are installing Stash on a GKE cluster, you will need cluster admin permissions to install Stash operator. Run the following command to grant admin permision to the cluster.

```console
$ kubectl create clusterrolebinding "cluster-admin-$(whoami)" \
  --clusterrole=cluster-admin \
  --user="$(gcloud config get-value core/account)"
```

In addition, if your GKE cluster is a [private cluster](https://cloud.google.com/kubernetes-engine/docs/how-to/private-clusters), you will need to either add an additional firewall rule that allows master nodes access port `8443/tcp` on worker nodes, or change the existing rule that allows access to ports `443/tcp` and `10250/tcp` to also allow access to port `8443/tcp`. The procedure to add or modify firewall rules is described in the official GKE documentation for private clusters mentioned before.

### Configuring Network Volume Accessor

To use network volumes (i.e. NFS) as a backend, Stash needs an additional deployment in the respective `Repository` namespace to provide Snapshot listing functionality. Stash automatically creates a network volume accessor deployment in a namespace that has at least one Repository with a network volume as backend. It automatically removes the deployment from the namespace if there are no more repositories with network volumes as backend in that namespace.

You can configure the network volume accessor deployment's cpu, memory, user id, and privileged mode by providing the `netVolAccessor` parameters as below:

```console
helm install stash-operator appscode/stash \
  --version {{< param "info.version" >}}   \
  --namespace kube-system                  \
  --set netVolAccessor.cpu=100m            \
  --set netVolAccessor.memory=128Mi        \
  --set netVolAccessor.runAsUser=0         \
  --set netVolAccessor.privileged=true     \
```

## Verify installation

To check if Stash operator pods have started, run the following command:

```console
$ kubectl get pods --all-namespaces -l app=stash --watch

NAMESPACE     NAME                              READY     STATUS    RESTARTS   AGE
kube-system   stash-operator-859d6bdb56-m9br5   2/2       Running   2          5s
```

Once the operator pods are running, you can cancel the above command by typing `Ctrl+C`.

Now, to confirm CRD groups have been registered by the operator, run the following command:

```console
$ kubectl get crd -l app=stash

NAME                                 AGE
recoveries.stash.appscode.com        5s
repositories.stash.appscode.com      5s
restics.stash.appscode.com           5s
```

Now, you are ready to [take your first backup](/docs/guides/latest/README.md) using Stash.

## Configuring RBAC

Stash introduces resources, such as, `Restic`, `Repository`, `Recovery` and `Snapshot`. Stash installer will create 2 user facing cluster roles:

| ClusterRole         | Aggregates To | Desription                                                                                            |
| ------------------- | ------------- | ----------------------------------------------------------------------------------------------------- |
| appscode:stash:edit | admin, edit   | Allows edit access to Stash CRDs, intended to be granted within a namespace using a RoleBinding.      |
| appscode:stash:view | view          | Allows read-only access to Stash CRDs, intended to be granted within a namespace using a RoleBinding. |

These user facing roles supports [ClusterRole Aggregation](https://kubernetes.io/docs/admin/authorization/rbac/#aggregated-clusterroles) feature in Kubernetes 1.9 or later clusters.

## Install Stash kubectl plugin

Stash provides a CLI using kubectl plugin to work with the stash Objects quickly. Download pre-build binaries from [stashed/cli Githhub release]() and put the binary to some directory in your `PATH`. To install linux 64-bit you can run the following commands:

```console
# Linux amd 64-bit
wget -O kubectl-stash https://github.com/stashed/cli/releases/download/{{< param "info.cli" >}}/kubectl-stash-linux-amd64 \
  && chmod +x kubectl-stash \
  && sudo mv kubectl-stash /usr/local/bin/
```

If you prefer to install kubectl Stash cli from source code, you will need to set up a GO development environment following [these instructions](https://golang.org/doc/code.html). Then, install the CLI using `go get` from source code.

```console
go get github.com/stashed/cli/...
```

> Please note that this will install kubectl stash cli from master branch which might include breaking and/or undocumented changes.

## Detect Stash version

To detect Stash version, exec into the operator pod and run `stash version` command.

```console
$ POD_NAMESPACE=kube-system
$ POD_NAME=$(kubectl get pods -n $POD_NAMESPACE -l app=stash -o jsonpath={.items[0].metadata.name})
$ kubectl exec -it $POD_NAME -c operator -n $POD_NAMESPACE /stash version

Version = {{< param "info.version" >}}
VersionStrategy = tag
Os = alpine
Arch = amd64
CommitHash = 85b0f16ab1b915633e968aac0ee23f877808ef49
GitBranch = release-0.5
GitTag = {{< param "info.version" >}}
CommitTimestamp = 2017-10-10T05:24:23

$ kubectl exec -it $POD_NAME -c operator -n $POD_NAMESPACE restic version
restic 0.9.1
compiled with go1.9 on linux/amd64
```
