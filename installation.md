# Installation Guide

# Kubernetes Observability Stack Installation

## Overview

This document describes the installation procedure for a complete open-source Kubernetes Observability Stack.

## Components

| Component               | Purpose                     |
| ----------------------- | --------------------------- |
| Prometheus              | Metrics collection          |
| Thanos                  | Long-term metrics storage   |
| Grafana                 | Dashboards                  |
| Fluent Bit              | Log collection              |
| Loki                    | Log storage                 |
| Tempo                   | Distributed tracing         |
| OpenTelemetry Collector | Telemetry pipeline          |
| OpenTelemetry SDK       | Application instrumentation |
| Alertmanager            | Alert routing               |
| Node Exporter           | Node metrics                |
| kube-state-metrics      | Kubernetes metrics          |
| cAdvisor                | Container metrics           |
| Kiali                   | Istio visualization         |

---

# Prerequisites

## Kubernetes

* Kubernetes v1.28+
* Helm v3.x
* kubectl configured
* StorageClass available
* Ingress Controller installed
* LoadBalancer support or MetalLB
* DNS configured (optional)

---

# Verify Cluster

```bash
kubectl get nodes

kubectl get ns

kubectl cluster-info
```

---

# Create Namespace

```bash
kubectl create namespace observability
```

---

# Add Helm Repositories

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm repo add grafana https://grafana.github.io/helm-charts

helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts

helm repo add bitnami https://charts.bitnami.com/bitnami

helm repo add kedacore https://kedacore.github.io/charts

helm repo update
```

---

# Installation Order

Install the components in the following order:

1. Prometheus
2. Alertmanager
3. Node Exporter
4. kube-state-metrics
5. cAdvisor
6. Thanos
7. Loki
8. Tempo
9. Fluent Bit
10. OpenTelemetry Collector
11. Grafana
12. Kiali

---

# Install Prometheus

```bash
helm install prometheus prometheus-community/prometheus \
-n observability
```

Verify:

```bash
kubectl get pods -n observability
```

---

# Install Alertmanager

Alertmanager is included with the Prometheus chart.

Verify:

```bash
kubectl get svc -n observability
```

---

# Install Node Exporter

```bash
helm install node-exporter prometheus-community/prometheus-node-exporter \
-n observability
```

Verify:

```bash
kubectl get daemonset -n observability
```

---

# Install kube-state-metrics

```bash
helm install kube-state-metrics prometheus-community/kube-state-metrics \
-n observability
```

---

# Install cAdvisor

```bash
kubectl apply -f cadvisor.yaml
```

Verify:

```bash
kubectl get pods -n observability
```

---

# Install Thanos

```bash
helm install thanos bitnami/thanos \
-n observability
```

Configure object storage (S3, MinIO, GCS, or Azure Blob) before production use.

---

# Install Loki

```bash
helm install loki grafana/loki \
-n observability
```

Verify:

```bash
kubectl get pods -n observability
```

---

# Install Tempo

```bash
helm install tempo grafana/tempo \
-n observability
```

---

# Install Fluent Bit

```bash
helm install fluent-bit grafana/fluent-bit \
-n observability
```

Configure outputs to send logs to Loki.

---

# Install OpenTelemetry Collector

```bash
helm install otel-collector open-telemetry/opentelemetry-collector \
-n observability
```

Configure exporters for:

* Prometheus
* Loki
* Tempo

---

# Install Grafana

```bash
helm install grafana grafana/grafana \
-n observability
```

Retrieve admin password:

```bash
kubectl get secret grafana \
-n observability \
-o jsonpath="{.data.admin-password}" | base64 -d
```

Port forward:

```bash
kubectl port-forward svc/grafana 3000:80 -n observability
```

Open:

```
http://localhost:3000
```

---

# Add Data Sources

Configure the following data sources:

## Prometheus

```
http://prometheus-server
```

---

## Loki

```
http://loki
```

---

## Tempo

```
http://tempo
```

---

# Install Kiali

```bash
helm install kiali-server kiali/kiali-server \
-n observability
```

If Istio is installed:

```bash
kubectl get svc -n istio-system
```

Verify Kiali dashboard.

---

# Instrument Applications

Add OpenTelemetry SDK to applications.

Example telemetry flow:

```
Application

↓

OpenTelemetry SDK

↓

OpenTelemetry Collector

↓

Prometheus
Loki
Tempo

↓

Grafana
```

---

# Configure Prometheus Scraping

Add scrape targets:

* Node Exporter
* kube-state-metrics
* cAdvisor
* OpenTelemetry Collector
* Application endpoints

Example:

```yaml
scrape_configs:

- job_name: application

  static_configs:

  - targets:

    - app-service:8080
```

---

# Configure Fluent Bit

Example output:

```ini
[OUTPUT]

Name loki

Host loki

Port 3100
```

---

# Configure OpenTelemetry Collector

Exporters:

```yaml
exporters:

  prometheus:

  otlp:

  loki:
```

Receivers:

```yaml
receivers:

  otlp:

  prometheus:
```

---

# Verify Installation

## Pods

```bash
kubectl get pods -n observability
```

---

## Services

```bash
kubectl get svc -n observability
```

---

## Deployments

```bash
kubectl get deploy -n observability
```

---

## StatefulSets

```bash
kubectl get sts -n observability
```

---

## DaemonSets

```bash
kubectl get ds -n observability
```

---

# Health Checks

## Prometheus

```
/targets
```

All targets should be UP.

---

## Grafana

Verify dashboards load successfully.

---

## Loki

Run a LogQL query.

---

## Tempo

Verify traces are visible.

---

## Alertmanager

Verify alert routing.

---

## Kiali

Verify service topology and traffic graph.

---

# Production Recommendations

* Enable persistent volumes for all stateful components.
* Use Thanos with object storage for long-term metrics.
* Configure TLS for all endpoints.
* Secure Grafana with SSO/OAuth.
* Configure retention policies for logs and traces.
* Deploy multiple replicas for high availability.
* Back up Grafana dashboards and Prometheus configurations.
* Integrate Alertmanager with Slack, Teams, Email, or PagerDuty.
* Monitor the observability stack itself using dedicated dashboards.

---

# Deployment Flow

```
Kubernetes Cluster

│

├── Applications

│     │

│     ▼

│ OpenTelemetry SDK

│     │

│     ▼

│ OpenTelemetry Collector

│

├── Metrics → Prometheus → Thanos

│

├── Logs → Fluent Bit → Loki

│

├── Traces → Tempo

│

└── Grafana

        │

        ├── Dashboards

        ├── Explore

        └── Alerting

                  │

                  ▼

            Alertmanager

                  │

     Slack • Teams • Email • PagerDuty
```

---

# Next Steps

After installation:

* Import Grafana dashboards.
* Configure alert rules.
* Instrument applications.
* Enable distributed tracing.
* Configure long-term storage.
* Secure ingress with TLS.
* Set up backup and disaster recovery.
* Enable authentication and RBAC.
* Scale components based on workload.
