---
title: Install Stash Community Version
description: Installation guide for Stash community version
menu:
  docs_{{ .version }}:
    identifier: install-stash-community
    name: Community Version
    parent: installation-guide
    weight: 10
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: setup
---

# Install Stash Community Version

## Get License

## Install

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

```bash
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

```bash
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

```bash
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

### Configuring Network Volume Accessor

To use network volumes (i.e. NFS) as a backend, Stash needs an additional deployment in the respective `Repository` namespace to provide Snapshot listing functionality. Stash automatically creates a network volume accessor deployment in a namespace that has at least one Repository with a network volume as backend. It automatically removes the deployment from the namespace if there are no more repositories with network volumes as backend in that namespace.

You can configure the network volume accessor deployment's cpu, memory, user id, and privileged mode by providing the `netVolAccessor` parameters as below:

```bash
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

```bash
$ kubectl get pods --all-namespaces -l app=stash --watch

NAMESPACE     NAME                              READY     STATUS    RESTARTS   AGE
kube-system   stash-operator-859d6bdb56-m9br5   2/2       Running   2          5s
```

Once the operator pods are running, you can cancel the above command by typing `Ctrl+C`.

Now, to confirm CRD groups have been registered by the operator, run the following command:

```bash
$ kubectl get crd -l app=stash

NAME                                 AGE
recoveries.stash.appscode.com        5s
repositories.stash.appscode.com      5s
restics.stash.appscode.com           5s
```

Now, you are ready to [take your first backup](/docs/guides/latest/README.md) using Stash.
