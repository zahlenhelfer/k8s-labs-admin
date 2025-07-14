# Übung: NGINX-Ingress-Controller installieren

## Schritt 1: Anlegen einer values.yaml Datei

```yaml
controller:
  hostNetwork: true
  kind: DaemonSet
  ingressClassResource:
    enabled: true
    default: true
  service:
    externalIPs: [<IP>]

```

## Schritt 2: Auslesen der externen HOST-IP Adresse

- Die IP Adresse kann einfach Shell ausgelesen werden:
`ip -4 -o addr show scope global | awk '{print $4}' | cut -d/ -f1 | head -n1`

- Ersetzen von `<IP>` in der `values.yaml`

## Schritt 3: Helm Installation des NGINX-Ingress Controller inkl. `values.yaml`

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx && helm repo update
helm install nginx-ingress ingress-nginx/ingress-nginx\
  --create-namespace --namespace ingress-controller\
  --version 4.12.2\
  -f values.yaml
```
