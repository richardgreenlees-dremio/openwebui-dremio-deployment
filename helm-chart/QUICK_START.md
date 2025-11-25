# Quick Start Guide

Get up and running with Open WebUI + Dremio in 5 minutes!

## Prerequisites

- Kubernetes cluster running
- Helm 3 installed
- Dremio PAT and URI

## Installation (3 Steps)

### 1. Create credentials file

```bash
cat > my-creds.yaml <<EOF
dremio:
  credentials:
    pat: "YOUR_DREMIO_PAT"
    uri: "http://YOUR_DREMIO_HOST:9047"
EOF
```

### 2. Install the chart

```bash
helm install openwebui ./helm-chart -f my-creds.yaml
```

### 3. Get the URL

```bash
kubectl get svc svc-openwebui -n openwebui
# Access at http://<EXTERNAL-IP>:3000
```

## Common Commands

```bash
# Check status
kubectl get pods -n openwebui

# View logs
kubectl logs -n openwebui deployment/openwebui
kubectl logs -n openwebui deployment/mcpo
kubectl logs -n openwebui deployment/ollama

# Port forward (if using ClusterIP)
kubectl port-forward -n openwebui svc/svc-openwebui 8080:3000

# Upgrade
helm upgrade openwebui ./helm-chart -f my-creds.yaml

# Uninstall
helm uninstall openwebui
kubectl delete namespace openwebui
```

## Customization Examples

### Use smaller resources (development)

```bash
helm install openwebui ./helm-chart \
  -f helm-chart/values-development.yaml \
  -f my-creds.yaml
```

### Use production settings

```bash
helm install openwebui ./helm-chart \
  -f helm-chart/values-production.yaml \
  -f my-creds.yaml
```

### Custom model

```bash
helm install openwebui ./helm-chart \
  -f my-creds.yaml \
  --set ollama.modelLoader.model=llama3.2
```

### Internal load balancer (AWS)

```bash
helm install openwebui ./helm-chart \
  -f my-creds.yaml \
  --set mcpo.service.annotations."service\.beta\.kubernetes\.io/aws-load-balancer-internal"="true"
```

### Disable node affinity

```bash
helm install openwebui ./helm-chart \
  -f my-creds.yaml \
  --set nodeAffinity.enabled=false
```

## Troubleshooting

### Pods stuck in Pending

```bash
# Check PVC status
kubectl get pvc -n openwebui

# Check events
kubectl get events -n openwebui --sort-by='.lastTimestamp'
```

### Can't access the UI

```bash
# Check service
kubectl get svc -n openwebui

# Port forward as workaround
kubectl port-forward -n openwebui svc/svc-openwebui 8080:3000
# Then access http://localhost:8080
```

### Init containers failing

```bash
# Check init container logs
kubectl logs -n openwebui deployment/mcpo -c dremio-mcp-loader
kubectl logs -n openwebui deployment/mcpo -c config-substitution
```

## Next Steps

- Read [INSTALL.md](INSTALL.md) for detailed installation guide
- Read [README.md](README.md) for full configuration options
- Configure ingress for HTTPS access
- Set up monitoring and backups

## Support

For detailed documentation, see:
- [README.md](README.md) - Full documentation
- [INSTALL.md](INSTALL.md) - Detailed installation guide
- [values.yaml](values.yaml) - All configuration options

