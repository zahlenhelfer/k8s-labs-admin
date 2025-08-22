# Kubernetes Exercise: Deploy Loki Stack with Log Collector and Sample App

## Objectives

By the end of this exercise, you will:

1. Deploy the **Loki Stack** (Loki, Promtail, and Grafana) using Helm.
2. Deploy a **sample application** that emits logs.
3. View the application logs in **Grafana** using **LogQL**.

---

## Prerequisites

* A Kubernetes cluster (minikube, kind, cloud, etc.)
* `kubectl` installed and configured
* `helm` installed ([https://helm.sh/docs/intro/install/](https://helm.sh/docs/intro/install/))
* Optional: port 3000 open on localhost for Grafana

---

## Step 1: Add and Install Loki Stack with Helm

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

# Create a monitoring namespace
kubectl create namespace monitoring

# Install the Loki stack (Loki + Promtail + Grafana)
helm install loki-stack grafana/loki-stack --namespace monitoring \
  --set grafana.enabled=true \
  --set promtail.enabled=true \
  --set loki.persistence.enabled=false \
  --set grafana.service.type=NodePort
```

Wait until everything is ready:

```bash
kubectl get pods -n monitoring
```

---

## Step 2: Access Grafana UI

Port forward Grafana:

```bash
kubectl port-forward -n monitoring service/loki-stack-grafana 3000:80
```

Then open: [http://localhost:3000](http://localhost:3000)

**Login credentials:**

* **User:** `admin`
* **Password:** `prom-operator` (default for Loki Helm chart)

---

## Step 3: Deploy a Sample App with Logs

Here’s a simple "log generator" app that prints messages to stdout.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: log-emitter
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: log-emitter
  template:
    metadata:
      labels:
        app: log-emitter
    spec:
      containers:
      - name: logger
        image: busybox
        command: ["/bin/sh"]
        args: ["-c", "while true; do echo Hello from $(hostname); sleep 5; done"]
```

Apply it:

```bash
kubectl apply -f log-emitter.yaml
```

---

## Step 4: View Logs in Grafana

### Add the Loki data source (if not already added)

1. Go to Grafana UI → ⚙️ **Configuration** → **Data Sources**
2. Click **Add data source** → choose **Loki**
3. URL: `http://loki-stack:3100`
4. Click **Save & Test**

### Explore logs

1. Go to **Explore**
2. Select **Loki** as the datasource
3. Use a query like:

```logql
{app="log-emitter"}
```

You should see logs like:

```console
Hello from log-emitter-xxxx
```

---

## Bonus Tasks

* Scale the deployment to `replicas: 3` and observe the logs.
* Modify the log message (e.g., include timestamps or random IDs).
* Add a `label` to logs and filter by it in LogQL.

---

## Cleanup

```bash
kubectl delete deployment log-emitter
helm uninstall loki-stack -n monitoring
kubectl delete namespace monitoring
```
