---
title: Prometheus Operator | Stash
description: Monitor Stash using Prometheus operator
menu:
  docs_{{ .version }}:
    identifier: monitoring-prometheus-operator
    name: Prometheus Operator
    parent: monitoring
    weight: 20
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: guides
---

# Monitoring Using Prometheus Operator

[Prometheus Operator](https://github.com/prometheus-operator/prometheus-operator) provides a simple and Kubernetes native way to deploy and configure a Prometheus server. This tutorial will show you how to use the Prometheus operator for monitoring Stash.

To keep Prometheus resources isolated, we are going to use a separate namespace `monitoring` to deploy the Prometheus operator and respective resources. Create the namespace if you haven't created it yet,

```bash
$ kubectl create ns monitoring
namespace/monitoring created
```

## Install Prometheus Stack

At first, you have to install Prometheus operator in your cluster. In this section, we are going to install Prometheus operator from [prometheus-community/kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack). You can skip this section if you already have Prometheus operator running.

Install `prometheus-community/kube-prometheus-stack` chart as below,

- Add necessary helm repositories.

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add stable https://charts.helm.sh/stable
helm repo update
```

- Install `kube-prometheus-stack` chart.

```bash
helm install prometheus-stack prometheus-community/kube-prometheus-stack -n monitoring
```

This chart will install [prometheus-operator/prometheus-operator](https://github.com/prometheus-operator/prometheus-operator), [kubernetes/kube-state-metrics](https://github.com/kubernetes/kube-state-metrics), [prometheus/node_exporter](https://github.com/prometheus/node_exporter), and [grafana/grafana](https://github.com/grafana/grafana) etc.

The above chart will also deploy a Prometheus server. Verify that the Prometheus server has been deployed by the following command:

```bash
$ kubectl get prometheus -n monitoring
NAME                                    VERSION   REPLICAS   AGE
prometheus-stack-kube-prom-prometheus   v2.22.1   1          3m
```

Let's check the YAML of the above Prometheus object,

```bash
$ kubectl get prometheus -n monitoring prometheus-stack-kube-prom-prometheus -o yaml
```

```yaml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  annotations:
    meta.helm.sh/release-name: prometheus-stack
    meta.helm.sh/release-namespace: monitoring
  labels:
    app: kube-prometheus-stack-prometheus
    app.kubernetes.io/managed-by: Helm
    chart: kube-prometheus-stack-12.8.0
    heritage: Helm
    release: prometheus-stack
  name: prometheus-stack-kube-prom-prometheus
  namespace: monitoring
spec:
  alerting:
    alertmanagers:
    - apiVersion: v2
      name: prometheus-stack-kube-prom-alertmanager
      namespace: monitoring
      pathPrefix: /
      port: web
  enableAdminAPI: false
  externalUrl: http://prometheus-stack-kube-prom-prometheus.monitoring:9090
  image: quay.io/prometheus/prometheus:v2.22.1
  listenLocal: false
  logFormat: logfmt
  logLevel: info
  paused: false
  podMonitorNamespaceSelector: {}
  podMonitorSelector:
    matchLabels:
      release: prometheus-stack
  portName: web
  probeNamespaceSelector: {}
  probeSelector:
    matchLabels:
      release: prometheus-stack
  replicas: 1
  retention: 10d
  routePrefix: /
  ruleNamespaceSelector: {}
  ruleSelector:
    matchLabels:
      app: kube-prometheus-stack
      release: prometheus-stack
  securityContext:
    fsGroup: 2000
    runAsGroup: 2000
    runAsNonRoot: true
    runAsUser: 1000
  serviceAccountName: prometheus-stack-kube-prom-prometheus
  serviceMonitorNamespaceSelector: {}
  serviceMonitorSelector:
    matchLabels:
      release: prometheus-stack
  version: v2.22.1
```

Notice the following ServiceMonitor related sections,

```yaml
serviceMonitorNamespaceSelector: {} # select from all namespaces
serviceMonitorSelector:
  matchLabels:
    release: prometheus-stack
```

Here, you can see the Prometheus server is selecting the ServiceMonitors from all namespaces that have `release: prometheus-stack` label.

The above chart will also create a Service for the Prometheus server so that we can access the Prometheus Web UI. Let's verify the Service has been created,

```bash
$ kubectl get service -n monitoring
NAME                                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
alertmanager-operated                       ClusterIP   None             <none>        9093/TCP,9094/TCP,9094/UDP   10m
prometheus-operated                         ClusterIP   None             <none>        9090/TCP                     10m
prometheus-stack-grafana                    ClusterIP   10.105.244.221   <none>        80/TCP                       11m
prometheus-stack-kube-prom-alertmanager     ClusterIP   10.97.172.208    <none>        9093/TCP                     11m
prometheus-stack-kube-prom-operator         ClusterIP   10.97.94.139     <none>        443/TCP                      11m
prometheus-stack-kube-prom-prometheus       ClusterIP   10.105.123.218   <none>        9090/TCP                     11m
prometheus-stack-kube-state-metrics         ClusterIP   10.96.52.8       <none>        8080/TCP                     11m
prometheus-stack-prometheus-node-exporter   ClusterIP   10.107.204.248   <none>        9100/TCP                     11m
```

Here, we can use the `prometheus-stack-kube-prom-prometheus` Service to access the Web UI of our Prometheus Server.

## Enable Monitoring in Stash

In this section, we are going to enable Prometheus monitoring in Stash. We have to enable Prometheus monitoring during installing Stash. You have to use `prometheus.io/operator` as the agent for monitoring via Prometheus operator.

Here, we are going to enable monitoring for both backup metrics and operator metrics using Helm 3. We are going to tell Stash to create ServiceMonitor with `release: prometheus-stack` label so that the Prometheus server we have deployed in the previous section can collect Stash metrics without any further configuration.

<ul class="nav nav-tabs" id="installerTab" role="tablist">
  <li class="nav-item">
    <a class="nav-link active" id="new-installer-tab" data-toggle="tab" href="#new-installation-tab" role="tab" aria-controls="new-installation-tab" aria-selected="true">New Installation</a>
  </li>
  <li class="nav-item">
    <a class="nav-link" id="existing-installation" data-toggle="tab" href="#existing-installation-tab" role="tab" aria-controls="existing-installation-tab" aria-selected="false">Existing Installation</a>
  </li>
</ul>
<div class="tab-content" id="installerTabContent">
  <div class="tab-pane fade show active" id="new-installation-tab" role="tabpanel" aria-labelledby="new-installation-tab">

### New Installation

If you haven't installed Stash yet, run the following command to enable Prometheus monitoring during installation

```bash
$ helm install stash appscode/stash-community -n kube-system \
--version {{< param "info.version" >}} \
--set monitoring.agent=prometheus.io/operator \
--set monitoring.backup=true \
--set monitoring.operator=true \
--set monitoring.serviceMonitor.labels.release=prometheus-stack \
--set-file license=/path/to/license-file.txt
```

</div>
<div class="tab-pane fade" id="existing-installation-tab" role="tabpanel" aria-labelledby="existing-installation-tab">

### Existing Installation

If you have installed Stash already in your cluster but didn't enable monitoring during installation, you can use `helm upgrade` command to enable monitoring in the existing installation.

```bash
$ helm upgrade stash appscode/stash-community -n kube-system \
--reuse-values \
--set monitoring.agent=prometheus.io/operator \
--set monitoring.backup=true \
--set monitoring.operator=true \
--set monitoring.serviceMonitor.labels.release=prometheus-stack
```

</div>
</div>

This will create a `ServiceMonitor` object with the same name and namespace as the Stash operator. The `ServiceMonitor` will have the label `release: prometheus-stack` as we have provided it through the `--set monitoring.serviceMonitor.labels` parameter.

Let's verify that the ServiceMonitor has been created in the Stash operator namespace.

```bash
$ kubectl get servicemonitor -n kube-system
NAME    AGE
stash   65s
```

Let's check the YAML of the `ServiceMonitor` object,

```bash
$ kubectl get servicemonitor stash -n kube-system -o yaml
```

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  annotations:
    meta.helm.sh/release-name: stash
    meta.helm.sh/release-namespace: kube-system
  labels:
    app.kubernetes.io/managed-by: Helm
    release: prometheus-stack
  name: stash
  namespace: kube-system
spec:
  endpoints:
  - honorLabels: true
    port: pushgateway
  - bearerTokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token
    port: api
    scheme: https
    tlsConfig:
      ca:
        secret:
          key: tls.crt
          name: stash-apiserver-cert
      serverName: stash.kube-system.svc
  namespaceSelector:
    matchNames:
    - kube-system
  selector:
    matchLabels:
      app.kubernetes.io/instance: stash
      app.kubernetes.io/name: stash
```

Here, we have two endpoints in `spec.endpoints` section. The `pushgateway` endpoint exports backup and recovery metrics and the `api` endpoint exports the operator metrics.

## Verify Monitoring

As soon as the Stash operator pod goes into the `Running` state, the Prometheus server we have deployed in the first section should discover the endpoints exposed by Stash for metrics.

Now, we are going to [forward port](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/) of `prometheus-stack-kube-prom-prometheus` Service to access the Prometheus web UI. Run the following command on a separate terminal,

```bash
$ kubectl port-forward -n monitoring service/prometheus-stack-kube-prom-prometheus 9090
Forwarding from 127.0.0.1:9090 -> 9090
Forwarding from [::1]:9090 -> 9090
```

Now, you can access the Web UI at `localhost:9090`. Open [http://localhost:9090/targets](http://localhost:9090/targets) in your browser. You should see `pushgateway` and `api` endpoints of the Stash operator are among the targets. This verifies that the Prometheus server is scrapping Stash metrics.

<figure align="center">
  <img alt="Stash Monitoring Flow" src="/docs/guides/latest/monitoring/images/prom_operator_web_ui.png">
<figcaption align="center">Fig: Prometheus Web UI</figcaption>
</figure>

## Cleanup

To cleanup the Kubernetes resources created by this tutorial, run:

```bash
# cleanup Prometheus resources
helm delete prometheus-stack -n monitoring

# delete namespace
kubectl delete ns monitoring
```

To uninstall Stash follow this [guide](/docs/setup/README.md).
