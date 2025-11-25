# Open WebUI with Dremio MCP Integration - Helm Chart

This Helm chart deploys Open WebUI with Dremio MCP (Model Context Protocol) integration on Kubernetes.

## Components

- **Ollama**: LLM inference server
- **Open WebUI**: Web interface for interacting with LLMs
- **MCPO**: MCP Orchestrator for connecting to Dremio
- **Dremio MCP Server**: Provides data access through MCP protocol

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+
- PersistentVolume provisioner support (for persistent storage)
- Access to a Dremio instance
- Dremio Personal Access Token (PAT)

## Installation

### Quick Start

1. **Add your Dremio credentials:**

```bash
# Create a values file with your credentials
cat > my-values.yaml <<EOF
dremio:
  credentials:
    pat: "your-dremio-pat-here"
    uri: "http://your-dremio-host:9047"
EOF
```

2. **Install the chart:**

```bash
helm install openwebui-dremio ./helm-chart -f my-values.yaml
```

### Using Command Line Parameters

```bash
helm install openwebui-dremio ./helm-chart \
  --set dremio.credentials.pat="your-pat" \
  --set dremio.credentials.uri="http://your-dremio-host:9047"
```

### Development Installation

```bash
helm install openwebui-dremio ./helm-chart \
  -f helm-chart/values-development.yaml \
  --set dremio.credentials.pat="your-pat" \
  --set dremio.credentials.uri="http://your-dremio-host:9047"
```

### Production Installation

```bash
helm install openwebui-dremio ./helm-chart \
  -f helm-chart/values-production.yaml \
  --set dremio.credentials.pat="your-pat" \
  --set dremio.credentials.uri="http://your-dremio-host:9047"
```

## Configuration

The following table lists the configurable parameters and their default values.

### Global Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `namespace` | Kubernetes namespace | `openwebui` |
| `nodeAffinity.enabled` | Enable node affinity | `true` |
| `nodeAffinity.key` | Node label key | `webui` |
| `nodeAffinity.operator` | Node selector operator | `Exists` |

### Ollama Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `ollama.enabled` | Enable Ollama deployment | `true` |
| `ollama.image.repository` | Ollama image repository | `ollama/ollama` |
| `ollama.image.tag` | Ollama image tag | `latest` |
| `ollama.service.type` | Kubernetes service type | `ClusterIP` |
| `ollama.service.port` | Service port | `11434` |
| `ollama.resources.requests.cpu` | CPU request | `1` |
| `ollama.resources.requests.memory` | Memory request | `2Gi` |
| `ollama.persistence.size` | PVC size | `50Gi` |
| `ollama.modelLoader.model` | Model to load | `llama3.1` |

### MCPO Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `mcpo.enabled` | Enable MCPO deployment | `true` |
| `mcpo.image.repository` | MCPO image repository | `ghcr.io/open-webui/mcpo` |
| `mcpo.image.tag` | MCPO image tag | `main` |
| `mcpo.service.type` | Kubernetes service type | `LoadBalancer` |
| `mcpo.service.port` | Service port | `8000` |
| `mcpo.resources.requests.cpu` | CPU request | `2` |
| `mcpo.resources.requests.memory` | Memory request | `8Gi` |
| `mcpo.persistence.size` | PVC size | `10Gi` |

### Open WebUI Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `openwebui.enabled` | Enable Open WebUI deployment | `true` |
| `openwebui.image.repository` | Open WebUI image repository | `ghcr.io/open-webui/open-webui` |
| `openwebui.image.tag` | Open WebUI image tag | `main` |
| `openwebui.service.type` | Kubernetes service type | `LoadBalancer` |
| `openwebui.service.port` | Service port | `3000` |
| `openwebui.resources.requests.cpu` | CPU request | `1` |
| `openwebui.resources.requests.memory` | Memory request | `2Gi` |

### Dremio Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `dremio.credentials.pat` | Dremio Personal Access Token | `YOUR_DREMIO_PAT_HERE` |
| `dremio.credentials.uri` | Dremio URI | `http://YOUR_DREMIO_HOST:PORT` |

## Accessing the Application

After installation, get the external IP:

```bash
# Get Open WebUI URL
kubectl get svc svc-openwebui -n openwebui

# Access the application
# Navigate to http://<EXTERNAL-IP>:3000
```

## Upgrading

```bash
helm upgrade openwebui-dremio ./helm-chart -f my-values.yaml
```

## Uninstalling

```bash
helm uninstall openwebui-dremio
```

To also delete the namespace and PVCs:

```bash
kubectl delete namespace openwebui
```

## Advanced Configuration

### Using Internal Load Balancer (AWS)

```yaml
mcpo:
  service:
    annotations:
      service.beta.kubernetes.io/aws-load-balancer-internal: "true"
```

### Custom Storage Class

```yaml
ollama:
  persistence:
    storageClass: "fast-ssd"

mcpo:
  persistence:
    storageClass: "fast-ssd"
```

### Disable Components

```yaml
ollama:
  enabled: false  # Disable Ollama if using external instance
```

## Troubleshooting

### Check Pod Status

```bash
kubectl get pods -n openwebui
```

### View Logs

```bash
# Ollama logs
kubectl logs -n openwebui deployment/ollama

# MCPO logs
kubectl logs -n openwebui deployment/mcpo

# Open WebUI logs
kubectl logs -n openwebui deployment/openwebui
```

### Check Init Container Logs

```bash
kubectl logs -n openwebui deployment/mcpo -c dremio-mcp-loader
kubectl logs -n openwebui deployment/mcpo -c config-substitution
```

## Support

For issues and questions, please refer to the project repository.

