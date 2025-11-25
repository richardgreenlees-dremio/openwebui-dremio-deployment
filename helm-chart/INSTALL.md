# Installation Guide

This guide provides step-by-step instructions for installing the Open WebUI with Dremio MCP integration using Helm.

## Prerequisites

Before you begin, ensure you have:

1. **Kubernetes cluster** (1.19+) with kubectl configured
2. **Helm 3.0+** installed
3. **Dremio instance** accessible from your Kubernetes cluster
4. **Dremio Personal Access Token (PAT)** - [How to create a PAT](https://docs.dremio.com/cloud/security/authentication/personal-access-token/)
5. **Storage provisioner** configured in your cluster (for PersistentVolumes)

## Step 1: Prepare Your Credentials

Create a values file with your Dremio credentials:

```bash
cat > my-credentials.yaml <<EOF
dremio:
  credentials:
    pat: "your-dremio-personal-access-token"
    uri: "http://your-dremio-host:9047"
EOF
```

**Security Note:** Keep this file secure and do not commit it to version control!

## Step 2: (Optional) Label Your Nodes

If you want to use node affinity to schedule pods on specific nodes:

```bash
# Label the nodes where you want to run the workload
kubectl label nodes <node-name> webui=true
```

To disable node affinity, create a custom values file:

```yaml
nodeAffinity:
  enabled: false
```

## Step 3: Install the Chart

### Option A: Basic Installation

```bash
helm install openwebui-dremio ./helm-chart -f my-credentials.yaml
```

### Option B: Development Installation

```bash
helm install openwebui-dremio ./helm-chart \
  -f helm-chart/values-development.yaml \
  -f my-credentials.yaml
```

### Option C: Production Installation

```bash
helm install openwebui-dremio ./helm-chart \
  -f helm-chart/values-production.yaml \
  -f my-credentials.yaml
```

### Option D: Custom Installation

Create your own values file:

```bash
cat > my-values.yaml <<EOF
namespace: my-namespace

ollama:
  persistence:
    size: 100Gi
    storageClass: "fast-ssd"
  modelLoader:
    model: llama3.1

mcpo:
  service:
    type: ClusterIP
  resources:
    requests:
      cpu: 4
      memory: 16Gi

openwebui:
  service:
    type: LoadBalancer
EOF

helm install openwebui-dremio ./helm-chart \
  -f my-values.yaml \
  -f my-credentials.yaml
```

## Step 4: Verify Installation

Check that all pods are running:

```bash
kubectl get pods -n openwebui
```

Expected output:
```
NAME                         READY   STATUS    RESTARTS   AGE
mcpo-xxxxxxxxxx-xxxxx        1/1     Running   0          2m
ollama-xxxxxxxxxx-xxxxx      1/1     Running   0          2m
ollama-model-loader-xxxxx    0/1     Completed 0          2m
openwebui-xxxxxxxxxx-xxxxx   1/1     Running   0          2m
```

Check the model loader job:

```bash
kubectl get jobs -n openwebui
```

## Step 5: Access Open WebUI

### For LoadBalancer Service Type:

```bash
# Get the external IP
kubectl get svc svc-openwebui -n openwebui

# Wait for EXTERNAL-IP to be assigned (may take a few minutes)
# Then access: http://<EXTERNAL-IP>:3000
```

### For NodePort Service Type:

```bash
export NODE_PORT=$(kubectl get svc svc-openwebui -n openwebui -o jsonpath='{.spec.ports[0].nodePort}')
export NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[0].address}')
echo "Access Open WebUI at: http://$NODE_IP:$NODE_PORT"
```

### For ClusterIP Service Type:

```bash
kubectl port-forward -n openwebui svc/svc-openwebui 8080:3000
# Access: http://localhost:8080
```

## Step 6: Verify Dremio Connection

1. Open the Open WebUI in your browser
2. Create an account or log in
3. The Dremio MCP integration should be automatically available
4. Test querying your Dremio data sources

## Troubleshooting

### Pods Not Starting

Check pod events:
```bash
kubectl describe pod <pod-name> -n openwebui
```

### Init Container Failures

Check init container logs:
```bash
kubectl logs -n openwebui deployment/mcpo -c dremio-mcp-loader
kubectl logs -n openwebui deployment/mcpo -c config-substitution
```

### PVC Not Binding

Check PVC status:
```bash
kubectl get pvc -n openwebui
```

If pending, check if you have a storage provisioner:
```bash
kubectl get storageclass
```

### Model Not Loading

Check the job logs:
```bash
kubectl logs -n openwebui job/ollama-model-loader
```

### Dremio Connection Issues

1. Verify credentials are correct
2. Check that Dremio is accessible from the cluster:
```bash
kubectl run -it --rm debug --image=curlimages/curl --restart=Never -- \
  curl -v http://your-dremio-host:9047
```

## Upgrading

To upgrade the deployment:

```bash
helm upgrade openwebui-dremio ./helm-chart -f my-credentials.yaml
```

## Uninstalling

To remove the deployment:

```bash
helm uninstall openwebui-dremio
```

To also delete the namespace and persistent data:

```bash
kubectl delete namespace openwebui
```

## Next Steps

- Configure additional MCP servers
- Set up ingress for HTTPS access
- Configure resource limits based on your workload
- Set up monitoring and logging
- Configure backup for persistent volumes

## Support

For issues and questions:
- Check the [README.md](README.md) for configuration options
- Review Kubernetes events and logs
- Consult the Open WebUI and Dremio documentation

