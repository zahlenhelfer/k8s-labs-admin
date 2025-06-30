# 🔬 Lab: Kubernetes Metrics Pipeline

**Goal:** Deploy `metrics-server` → Scrape with `Prometheus` → Visualize in `Grafana`

---

## 🧰 Prerequisites

* A running Kubernetes cluster (Minikube, Kind, or similar)
* `kubectl` and optionally `helm` installed

---

## 📦 Overview

```text
+-------------------+
| metrics-server    |  <-- collects resource metrics (CPU/mem) from nodes and pods
+-------------------+
         ↓
+-------------------+
| Prometheus        |  <-- scrapes metrics-server via HTTP
+-------------------+
         ↓
+-------------------+
| Grafana           |  <-- visualizes Prometheus metrics
+-------------------+
```

---

## 📁 Step-by-Step Instructions

---

### 1️⃣ Deploy `metrics-server`

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

> ⚠️ If you're using Minikube, patch the deployment:

```bash
kubectl patch deployment metrics-server -n kube-system \
  --type=json -p='[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'
```

Confirm it's working:

```bash
kubectl get --raw "/apis/metrics.k8s.io/v1beta1/nodes"
```

---

### 2️⃣ Install Prometheus using Helm

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install prometheus prometheus-community/prometheus \
  --namespace monitoring --create-namespace
```

Verify installation:

```bash
kubectl get pods -n monitoring
kubectl port-forward svc/prometheus-server -n monitoring 9090:80
```

Visit: [http://localhost:9090](http://localhost:9090)

---

### 3️⃣ Configure Prometheus to Scrape Metrics Server

#### 📄 Add a new scrape config

Create a configmap to add custom scrape configs:

```yaml
# metrics-scrape-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: additional-scrape-configs
  namespace: monitoring
data:
  scrape-configs.yaml: |
    - job_name: 'metrics-server'
      metrics_path: /metrics
      scheme: https
      static_configs:
        - targets: ['metrics-server.kube-system.svc:443']
      tls_config:
        insecure_skip_verify: true
```

Apply it:

```bash
kubectl apply -f metrics-scrape-config.yaml
```

Now update the Prometheus release to use it:

```bash
helm upgrade prometheus prometheus-community/prometheus \
  --namespace monitoring \
  --set server.extraScrapeConfigsSecret.enabled=true \
  --set server.extraScrapeConfigsSecret.name=additional-scrape-configs \
  --set-file server.extraScrapeConfigsSecret.data.scrape-configs.yaml=metrics-scrape-config.yaml
```

---

### 4️⃣ Install Grafana with Helm

```bash
helm install grafana grafana/grafana \
  --namespace monitoring \
  --set adminPassword='admin' \
  --set service.type=NodePort \
  --create-namespace
```

Forward port:

```bash
kubectl port-forward svc/grafana -n monitoring 3000:80
```

Visit: [http://localhost:3000](http://localhost:3000)
Login with: `admin / admin`

---

### 5️⃣ Configure Grafana

* Add Prometheus as a data source:

  * URL: `http://prometheus-server.monitoring.svc.cluster.local`
* Import dashboard (ID: **6417** for Kubernetes resource metrics)

---

## ✅ Done

Now you're visualizing real-time Kubernetes metrics via metrics-server → Prometheus → Grafana.

---

## 🧹 Cleanup

```bash
helm uninstall prometheus -n monitoring
helm uninstall grafana -n monitoring
kubectl delete ns monitoring
kubectl delete -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

---

## 📦 Want a GitHub-ready repo version?

I can generate a repo with YAML manifests, Helm values, and a README—just say the word.
