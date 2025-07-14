
# Lab: Deploy Kyverno and Enforce `costcenter` Label on Pods

---

## Objective

1. Deploy Kyverno in your cluster
2. Create a Kyverno policy that **validates all pods** have a `costcenter` label
3. Test pod creation with and without the label

---

## Prerequisites

* A running Kubernetes cluster (Minikube, Kind, etc.)
* `kubectl` installed
* Cluster-admin access

---

## 📁 Lab Structure

```
kyverno-lab/
├── policies/
│   └── require-costcenter-label.yaml
└── test/
    ├── good-pod.yaml
    └── bad-pod.yaml
```

---

## Install Kyverno

```bash
kubectl create -f https://github.com/kyverno/kyverno/releases/latest/download/install.yaml
```

Verify it’s running:

```bash
kubectl get pods -n kyverno
```

---

## Create the Required Label Policy

📄 `policies/require-costcenter-label.yaml`

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-costcenter-label
spec:
  validationFailureAction: Enforce
  rules:
    - name: check-costcenter-label
      match:
        resources:
          kinds:
            - Pod
      validate:
        message: "All pods must have a 'costcenter' label."
        pattern:
          metadata:
            labels:
              costcenter: "?*"
```

Apply the policy:

```bash
kubectl apply -f policies/require-costcenter-label.yaml
```

---

## Test the Policy

---

### Good Pod

📄 `test/good-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: good-nginx
  labels:
    costcenter: devops
spec:
  containers:
  - name: nginx
    image: nginx
```

Apply:

```bash
kubectl apply -f test/good-pod.yaml
```

🟢 Should succeed.

---

### ❌ Bad Pod (Missing `costcenter`)

📄 `test/bad-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: bad-nginx
spec:
  containers:
  - name: nginx
    image: nginx
```

Apply:

```bash
kubectl apply -f test/bad-pod.yaml
```

🔴 Should be rejected with:

```
Error from server: admission webhook ... denied the request: All pods must have a 'costcenter' label.
```

---

## Summary

| Component          | Description                             |
| ------------------ | --------------------------------------- |
| Kyverno            | Validating admission controller         |
| ClusterPolicy      | Enforces label presence on pod creation |
| `costcenter` label | Required label to pass policy check     |

---

## Cleanup

```bash
kubectl delete -f test/
kubectl delete -f policies/require-costcenter-label.yaml
kubectl delete -f https://github.com/kyverno/kyverno/releases/latest/download/install.yaml
```
