
# Lab: Using Kustomize with Kubernetes

## Prerequisites

* Kubernetes cluster (Minikube, Kind, or any cluster)
* `kubectl` installed
* `kustomize` installed (or use `kubectl kustomize`)

---

## 📁 Directory Structure

```bash
kustomize-lab/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── overlays/
    ├── dev/
    │   ├── kustomization.yaml
    │   └── patch.yaml
    └── prod/
        ├── kustomization.yaml
        └── patch.yaml
```

---

## Create the Base Configuration

📄 `base/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.21
          ports:
            - containerPort: 80
```

📄 `base/service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

📄 `base/kustomization.yaml`

```yaml
resources:
  - deployment.yaml
  - service.yaml
```

---

## Create Overlays

### Dev Overlay

📄 `overlays/dev/kustomization.yaml`

```yaml
resources:
  - ../../base

patches:
  - path: patch.yaml
```

📄 `overlays/dev/patch.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 1
```

---

### Prod Overlay

📄 `overlays/prod/kustomization.yaml`

```yaml
resources:
  - ../../base

patches:
  - path: patch.yaml
```

📄 `overlays/prod/patch.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 4
```

---

## Apply the Configurations

### Dev

```bash
kubectl apply -k overlays/dev
```

### Prod

```bash
kubectl apply -k overlays/prod
```

---

## Verify

```bash
kubectl get deployments
kubectl get services
```

You should see an `nginx` deployment with the correct number of replicas depending on the environment.

---

## Clean Up

```bash
kubectl delete -k overlays/dev
# or
kubectl delete -k overlays/prod
```
