# Lab: Create and Use a Custom Resource Definition (CRD)

## Goal

Create a `Fruit` CRD that allows you to define fruits like `apple`, `banana`, etc., and deploy a controller that prints them.

---

## Prerequisites

* Kubernetes cluster (e.g. Minikube)
* `kubectl` installed
* Optional: `kustomize` and `controller-gen` if building a full controller

---

## Lab Structure

```bash
crd-lab/
├── crds/
│   └── fruit-crd.yaml
├── examples/
│   └── apple.yaml
└── controller/
    └── fruit-controller.sh
```

---

## Define the CRD

📄 `crds/fruit-crd.yaml`

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: fruits.example.com
spec:
  group: example.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                color:
                  type: string
                taste:
                  type: string
      subresources:
        status: {}
  scope: Namespaced
  names:
    plural: fruits
    singular: fruit
    kind: Fruit
    shortNames:
      - frt
```

Apply the CRD:

```bash
kubectl apply -f crds/fruit-crd.yaml
```

---

## Create a Custom Resource (CR)

📄 `examples/apple.yaml`

```yaml
apiVersion: example.com/v1
kind: Fruit
metadata:
  name: apple
spec:
  color: red
  taste: sweet
```

Apply the CR:

```bash
kubectl apply -f examples/apple.yaml
```

Verify:

```bash
kubectl get fruits
kubectl get fruit apple -o yaml
```

---

## Create a Simple Controller (optional)

This is a *demo controller* (bash script) that watches the fruit CRs and prints them. In real apps you'd use a proper controller with `client-go` or Kubebuilder.

📄 `controller/fruit-controller.sh`

```bash
#!/bin/bash

echo "Watching for Fruit resources..."
kubectl get fruit --watch -o json | jq -c '.'
```

Run:

```bash
bash controller/fruit-controller.sh
```

Then try creating more fruits:

```yaml
# examples/banana.yaml
apiVersion: example.com/v1
kind: Fruit
metadata:
  name: banana
spec:
  color: yellow
  taste: sweet
```

```bash
kubectl apply -f examples/banana.yaml
```

---

## Clean Up

```bash
kubectl delete -f examples/
kubectl delete -f crds/
```

---

## Summary

| Component        | Description                      |
| ---------------- | -------------------------------- |
| `Fruit` CRD      | Defines the structure of a fruit |
| Custom Resources | Concrete fruit instances         |
| Controller       | Watches and prints changes       |
