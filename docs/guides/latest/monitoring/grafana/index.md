---
title: Stash Grafana Dashboard | Stash
description: Using Stash Grafana Dashboard
menu:
  docs_{{ .version }}:
    identifier: monitoring-grafana-dashboard
    name: Grafana Dashboard
    parent: monitoring
    weight: 30
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: guides
---

{{< notice type="warning" message="This is an Enterprise-only feature. You must be **Stash Enterprise** customer to use pre-built Stash Grafana dashboard." >}}

# Stash Grafana Dashboard

Grafana provides an elegant graphical user interface to visualize data. You can create beautiful dashboard easily with a meaningful representation of your Prometheus metrics.

We provide a pre-built Grafana dashboard to our **Stash Enterprise** users only. In this guide, we are going to show you how you can import this dashboard from your Grafana UI. We will also give you an overview of the different components of the dashboard.

>Some basic metrics are also available for Stash Community Edition. You can create your own dashboard using those metrics.

## Before You Begin

- At first, you need to setup a Prometheus monitoring stack in your cluster. Please, follow the [Prometheus Operator](/docs/guides/latest/monitoring/prom-operator/index.md) guide to setup your monitoring stack if you haven't done already.
- Then, install Stash Enterprise edition with monitoring enabled. Please, follow [this guide](/docs/guides/latest/monitoring/prom-operator/index.md#enable-monitoring-in-stash) if you haven't done already.
- You must have `stash_dashboard.json` file. Please, contact us to get the dashboard json file.

## Install Panopticon

Stash Grafana dashboard depends on our another product called [Panopticon](https://blog.byte.builders/post/introducing-panopticon/). It is a Kubernetes state metric exporter similar to [kube-state-metrics](https://github.com/kubernetes/kube-state-metrics) but generic for all Kubernetes resources including CRDs. In this section, we are going to show the installation procedure for Panopticon.

**Issue License:**

Like other AppsCode products, [Panopticon](https://blog.byte.builders/post/introducing-panopticon/) also need a license to to run. You can grab a 30 days trial license for Panopticon from [here](https://license-issuer.appscode.com/?p=panopticon-enterprise).

>**If you already have an enterprise license for KubeDB or Stash, you do not need to issue a new license for Panopticon. Your existing KubeDB or Stash license will work with Panopticon.**

**Install Panopticon:**

Now, install Panopticon using the following commands:

```bash
$ helm repo add appscode https://charts.appscode.com/stable/
$ helm repo update

$ helm install panopticon appscode/panopticon -n kubeops \
    --create-namespace \
    --set monitoring.enabled=true \
    --set monitoring.agent=prometheus.io/operator \
    --set monitoring.serviceMonitor.labels.release=prometheus-stack \
    --set-file license=/path/to/license-file.txt
```

## Import Stash Garafana Dashboard

At first, let's port-forward the respective service for Grafana dashboard so that we can access through our browser.

```bash
❯ kubectl port-forward -n monitoring service/prometheus-stack-grafana 3000:80
Forwarding from 127.0.0.1:3000 -> 3000
Forwarding from [::1]:3000 -> 3000
```

Now, go to http://localhost:3000/ and login to your Grafana UI. Then, on the Grafana UI, click the `+` icon from the left sidebar and then click on `Import` button as below,

<figure align="center">
  <img alt="Import Stash Grafana Dashboard: Step 1" src="/docs/guides/latest/monitoring/grafana/images/import_dashboard_1.png">
<figcaption align="center">Fig: Import Stash Grafana Dashboard (Step 1)</figcaption>
</figure>

Then, on the import UI, you can either upload the `stash_dashboard.json` file by clicking `Upload JSON file` button or you can paste the content of the JSON file in the text area labeled as `Import via panel json`.

<figure align="center">
  <img alt="Import Stash Grafana Dashboard: Step 2" src="/docs/guides/latest/monitoring/grafana/images/import_dashboard_2.png">
<figcaption align="center">Fig: Import Stash Grafana Dashboard (Step 2)</figcaption>
</figure>

Then, on the next step click `Import` button as below.

<figure align="center">
  <img alt="Import Stash Grafana Dashboard: Step 3" src="/docs/guides/latest/monitoring/grafana/images/import_dashboard_3.png">
<figcaption align="center">Fig: Import Stash Grafana Dashboard (Step 3)</figcaption>
</figure>

Once, you have successfully imported the dashboard, you should see the Stash dashboard similar to this.

<figure align="center">
  <img alt="Stash Grafana Dashboard" src="/docs/guides/latest/monitoring/grafana/images/stash_grafana_dashboard.png">
<figcaption align="center">Fig: Stash Grafana Dashboard</figcaption>
</figure>

If your cluster does not have any backup configured, you may see the dashboard panels are empty. Nothing to worry here. Just, run some backups and your dashboard should be populated automatically.

## Dashboard Tour

The following video gives a tour of different components of the Stash Grafana dashboard.

{{< youtube VYx1OI1tgkE >}}
