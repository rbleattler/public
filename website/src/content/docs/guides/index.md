---
title: Quick-Start
sidebar:
  order: 1
---

:::note[Clustertool]

Clusters created using Clustertool come pre-packed with most of these charts pre-installed.

:::

## Prerequisites

- Running Kubernetes Cluster
- Container Storage Interface (CSI)
- LoadBalancer like Metallb

## Required Charts for most Truecharts Charts

Install the following charts if not already installed:

- [Cert-Manager](#cert-manager)
- [Cloudnative-PG](#cloudnative-pg)
- [Prometheus](#prometheus)

---

## Recommended Charts

- [Blocky](https://truecharts.org/charts/premium/blocky/): Local DNS Resolving with k8s-gateway
- [Clusterissuer](https://truecharts.org/charts/premium/clusterissuer/): Configuring Cert-Manager
- [Kubernetes-Reflector](https://truecharts.org/charts/system/kubernetes-reflector/): Reflect Resources across Namespaces
- [Metallb](https://metallb.io/) with [Metallb-Config](https://truecharts.org/charts/premium/metallb-config/) as LoadBalancer
- [Node Feature Discovery](https://github.com/kubernetes-sigs/node-feature-discovery): For Node Discovery
- [Nginx-Ingress](https://kubernetes.github.io/ingress-nginx/): For Ingress and Reverse Proxying
- [Snapshot-Controller](https://truecharts.org/charts/system/snapshot-controller/): Required for Volsync
- [Volsync](https://truecharts.org/charts/system/volsync/): For Backup and Restore of PVCs

## Upstream Operators

Truecharts relies on multiple Charts for functionality like Postgres Databases and Metrics.
Therefore we require certain Charts to be installed. Below you will find example configurations for most of them:

### Cert-Manager

Cert-Manager is used  together with our clusterissuer to create SSL certificates for ingress.
The chart installation can be found [here](https://cert-manager.io/docs/installation/helm/).

Example configuration:

```yaml

crds:
  enabled: true
dns01RecursiveNameservers: "1.1.1.1:53,1.0.0.1:53"
dns01RecursiveNameserversOnly: false
enableCertificateOwnerRef: true

```

### Cloudnative-PG

Cloudnative-PG is used for Postgres databases in many of our charts.
The chart can be found [here](https://github.com/cloudnative-pg/charts).

Example configuration:

```yaml

crds:
  create: true

```

### Prometheus

Kube-promotheus-stack is used for metrics.
The chart can be found [here](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack).

As we provide our own grafana with included dashboard. We recommend to disable grafana and add a few tweaks in the kube-prometheus-stack:

```yaml
grafana:
  enabled: false
  forceDeployDashboards: true
  defaultDashboardsEnabled: true
  forceDeployDatasources: true
crds:
  enabled: true
  upgradeJob:
    enabled: true
    forceConflicts: true
cleanPrometheusOperatorObjectNames: true
alertmanager:
  enabled: false
kubeProxy:
  enabled: false
kubeEtcd:
  service:
    selector:
      component: kube-apiserver # etcd runs on control plane nodes
prometheus:
  prometheusSpec:
    podMonitorSelectorNilUsesHelmValues: false
    probeSelectorNilUsesHelmValues: false
    ruleSelectorNilUsesHelmValues: false
    scrapeConfigSelectorNilUsesHelmValues: false
    serviceMonitorSelectorNilUsesHelmValues: false
    enableAdminAPI: true
    walCompression: true
    enableFeatures:
      - memory-snapshot-on-shutdown
    retention: 14d
    retentionSize: 50GB
    resources:
      requests:
        cpu: 100m
        memory: 500Mi
      limits:
        memory: 2000Mi
```

We generally advice to run the full kube-prometheus-stack but as it is quite resource intensive you can run the minimum requirement which only requires to add the CRDs. This can be done like this:

```yaml
crds:
  enabled: true
  upgradeJob:
    enabled: true
    forceConflicts: true
prometheusOperator:
  enabled: false
## Everything down here, explicitly disables everything except CRDs and grafana dashboards
global:
  rbac:
    create: false
defaultRules:
  create: false
windowsMonitoring:
  enabled: false
prometheus-windows-exporter:
  prometheus:
    monitor:
      enabled: false
alertmanager:
  enabled: false
grafana:
  enabled: false
  forceDeployDashboards: true
  defaultDashboardsEnabled: true
  forceDeployDatasources: true
kubernetesServiceMonitors:
  enabled: true
kubeApiServer:
  enabled: false
kubelet:
  enabled: false
kubeControllerManager:
  enabled: false
coreDns:
  enabled: false
kubeDns:
  enabled: false
kubeEtcd:
  enabled: false
kubeScheduler:
  enabled: false
kubeProxy:
  enabled: false
kubeStateMetrics:
  enabled: false
nodeExporter:
  enabled: false
prometheus:
  enabled: false
thanosRuler:
  enabled: false
```
