---
title: etcd Encryption
---

# 🧪 Kubernetes Lab: Encrypting `etcd` for Secrets

## 🧰 Requirements

* A Kubernetes cluster (minikube, kind, kubeadm, or cloud-based)
* Root/sudo access to the control plane node
* `kubectl` configured to access the cluster

---

## 🔐 Step 1: Generate Encryption Config

On the **control plane node**, create a file called `encryption-config.yaml`:

```bash
mkdir -p /etc/kubernetes/pki/encryption/
```

create the File - `/etc/kubernetes/pki/encryption/encryption-config.yaml`

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: <BASE64_ENCODED_32BYTE_KEY>
      - identity: {}
```

Generate a 32-byte base64 key:

```bash
head -c 32 /dev/urandom | base64
```

Paste this in place of `<BASE64_ENCODED_32BYTE_KEY>`.

---

## 🧠 Step 2: Configure kube-apiserver to Use Encryption

Edit the kube-apiserver manifest `/etc/kubernetes/manifests/kube-apiserver.yaml`

Add this flag under `spec.containers.args`:

```yaml
--encryption-provider-config=/etc/kubernetes/pki/encryption/encryption-config.yaml
```

Save and exit. `kubelet` will automatically restart the API server due to the manifest change.

---

## ✅ Step 3: Verify Encryption

### 1. Create a secret

```bash
kubectl create secret generic mysecret --from-literal=password='s3cr3t'
```

### 2. Get the secret (as Kubernetes sees it)

```bash
kubectl get secret mysecret -o yaml
```

You will still see a base64-encoded string, which is **not** the encrypted value — just encoding.

### 3. View etcd data directly

Install etcdctl if needed. Use it to check raw storage:

```bash
ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  get /registry/secrets/default/mysecret | hexdump -C
```

You should **not** see `s3cr3t` in the output — indicating that the secret is encrypted at rest.

---

## ♻️ Optional: Rotate Encryption Keys

To rotate keys:

1. Add a new key as `key2` and set it as the first in the list.
2. Run `kubectl get secrets --all-namespaces -o json | kubectl replace -f -` to rewrite and re-encrypt.
3. Remove the old key (`key1`) from the config once all secrets have been rewritten.
