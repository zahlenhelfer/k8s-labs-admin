# Installation eines zwei Node Kubernetes-Cluster

## Login

- `ssh student@k8s-node-XX.dockerlabs.de`

## Installation der Control-Plane

- Login auf den ersten Server
- wechsel in das Verzeichnis: `cd LFD459/SOLUTIONS/s_02/`
- starten des Scripts: `script -q -c "bash k8scp.sh" $HOME/controlplane.out`
- bei der Frage `keep the local version currently installed` auswählen
- Kopieren des Join-Befehls <kubeadm join> aus der Log-Datei per grep: `grep -A1 "kubeadm join" controlplane.out`
- Besipieloutput: `kubeadm join <ip>:6443 --token 1123 --discovery-token-ca-cert-hash sha256:16df252`
- Installation der Auto-Completion:

```bash
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> $HOME/.bashrc
```

- Installation Prüfen:

```bash
kubectl get nodes
NAME         STATUS   ROLES           AGE   VERSION
k8s-node-0   Ready    control-plane   11m   v1.33.1
```

- Taint enfernen:
`kubectl taint nodes --all node-role.kubernetes.io/control-plane-`

## Installation des Worker

- Login auf den zweiten Server
- wechsel in das Verzeichnis: `cd LFD459/SOLUTIONS/s_02/`
- `bash k8sWorker.sh`
- `sudo kubeadm join ...`
- wechsel zur Control-Plane und wiederholtes: `kubectl get nodes`

## Test

```mermaid
graph LR
    hello --> world
    world --> again
    again --> hello
```

``` mermaid
graph LR
  A[Start] --> B{Error?};
  B -->|Yes| C[Hmm...];
  C --> D[Debug];
  D --> B;
  B ---->|No| E[Yay!];
```
