# Migration Guide: Plain YAML to Helm Chart

This guide helps you migrate from the existing plain YAML deployment to the Helm chart.

## Overview

The Helm chart provides the same functionality as the plain YAML files but with:
- Easier configuration management
- Multiple environment support
- Simplified upgrades and rollbacks
- Better documentation

## Pre-Migration Checklist

Before migrating, gather the following information from your current deployment:

1. **Dremio Credentials:**
   - PAT (from `Secrets/secrets.yaml`)
   - URI (from `Secrets/secrets.yaml`)

2. **Current Configuration:**
   - Namespace name
   - Storage sizes (PVC)
   - Resource limits
   - Service types
   - Node labels (if using node affinity)

3. **Persistent Data:**
   - Ollama models location
   - MCPO data location

## Migration Options

### Option 1: Fresh Installation (Recommended)

This is the cleanest approach - deploy the Helm chart alongside the existing deployment, test it, then remove the old one.

#### Step 1: Create values file with your current settings

```bash
cat > migration-values.yaml <<EOF
namespace: openwebui  # Use same namespace or different one

dremio:
  credentials:
    pat: "YOUR_CURRENT_PAT"
    uri: "YOUR_CURRENT_URI"

ollama:
  persistence:
    size: 50Gi  # Match your current PVC size
  modelLoader:
    model: llama3.1  # Match your current model

mcpo:
  persistence:
    size: 10Gi  # Match your current PVC size
  service:
    type: LoadBalancer  # Match your current service type
    annotations:
      service.beta.kubernetes.io/azure-load-balancer-internal: "true"  # If using Azure

openwebui:
  service:
    type: LoadBalancer  # Match your current service type
EOF
```

#### Step 2: Install Helm chart in a test namespace

```bash
# Install in a separate namespace first
helm install openwebui-test ./helm-chart \
  -f migration-values.yaml \
  --set namespace=openwebui-test
```

#### Step 3: Test the new deployment

```bash
# Check pods
kubectl get pods -n openwebui-test

# Get service URL
kubectl get svc -n openwebui-test

# Test the application
```

#### Step 4: Migrate data (if needed)

If you want to preserve Ollama models:

```bash
# Copy data from old PVC to new PVC
kubectl run -it --rm copy-data --image=busybox --restart=Never -- sh

# Inside the pod:
# Mount both PVCs and copy data
```

#### Step 5: Switch to new deployment

```bash
# Delete old deployment
kubectl delete -f Charts/deployments.yaml
kubectl delete -f Charts/services.yaml
kubectl delete -f Charts/jobs.yaml

# Install Helm chart in production namespace
helm install openwebui ./helm-chart \
  -f migration-values.yaml
```

### Option 2: In-Place Migration (Advanced)

This approach converts the existing deployment to Helm management.

⚠️ **Warning:** This is more complex and risky. Backup your data first!

#### Step 1: Backup current deployment

```bash
# Export current resources
kubectl get all -n openwebui -o yaml > backup-deployment.yaml
kubectl get pvc -n openwebui -o yaml > backup-pvc.yaml
kubectl get configmap -n openwebui -o yaml > backup-configmap.yaml
kubectl get secret -n openwebui -o yaml > backup-secret.yaml
```

#### Step 2: Delete non-persistent resources

```bash
# Delete deployments and services (keeps PVCs)
kubectl delete deployment --all -n openwebui
kubectl delete service --all -n openwebui
kubectl delete job --all -n openwebui
kubectl delete configmap --all -n openwebui
kubectl delete secret dremio-credentials -n openwebui
```

#### Step 3: Install Helm chart

```bash
# Install Helm chart (will reuse existing PVCs)
helm install openwebui ./helm-chart \
  -f migration-values.yaml
```

## Mapping: Plain YAML to Helm Values

Here's how the plain YAML files map to Helm values:

### Charts/deployments.yaml → values.yaml

| Plain YAML | Helm Values |
|------------|-------------|
| `ollama` deployment | `ollama.enabled: true` |
| `ollama` image | `ollama.image.repository` + `ollama.image.tag` |
| `ollama` resources | `ollama.resources` |
| `mcpo` deployment | `mcpo.enabled: true` |
| `mcpo` image | `mcpo.image.repository` + `mcpo.image.tag` |
| `mcpo` resources | `mcpo.resources` |
| `openwebui` deployment | `openwebui.enabled: true` |
| `openwebui` image | `openwebui.image.repository` + `openwebui.image.tag` |
| Node affinity | `nodeAffinity.enabled` + `nodeAffinity.key` |

### Charts/services.yaml → values.yaml

| Plain YAML | Helm Values |
|------------|-------------|
| `svc-ollama` type | `ollama.service.type` |
| `svc-ollama` port | `ollama.service.port` |
| `svc-mcpo` type | `mcpo.service.type` |
| `svc-mcpo` annotations | `mcpo.service.annotations` |
| `svc-openwebui` type | `openwebui.service.type` |

### Charts/pvc.yaml → values.yaml

| Plain YAML | Helm Values |
|------------|-------------|
| `pvc-ollama` size | `ollama.persistence.size` |
| `pvc-ollama` storageClass | `ollama.persistence.storageClass` |
| `pvc-mcpo` size | `mcpo.persistence.size` |
| `pvc-mcpo` storageClass | `mcpo.persistence.storageClass` |

### Secrets/secrets.yaml → values.yaml

| Plain YAML | Helm Values |
|------------|-------------|
| `dremio-pat` | `dremio.credentials.pat` |
| `dremio-uri` | `dremio.credentials.uri` |

### Charts/config.yaml → values.yaml

| Plain YAML | Helm Values |
|------------|-------------|
| `mcpo-config.json` | `mcpo.mcpServers` |
| `dremio-config.yaml` | `mcpo.mcpServers.dremio.config` |

## Post-Migration

After successful migration:

1. **Verify all components are running:**
   ```bash
   kubectl get pods -n openwebui
   ```

2. **Test the application:**
   - Access Open WebUI
   - Test Dremio connection
   - Verify models are loaded

3. **Update your deployment process:**
   - Use `helm upgrade` instead of `kubectl apply`
   - Store values files in version control
   - Document your custom values

4. **Clean up old files (optional):**
   ```bash
   # Archive old YAML files
   mkdir -p archive
   mv Charts archive/
   mv Secrets archive/
   ```

## Rollback Plan

If something goes wrong:

### If using Option 1 (Fresh Installation):

```bash
# Delete Helm release
helm uninstall openwebui

# Restore old deployment
kubectl apply -f Charts/
kubectl apply -f Secrets/secrets.yaml
```

### If using Option 2 (In-Place):

```bash
# Uninstall Helm chart
helm uninstall openwebui

# Restore from backup
kubectl apply -f backup-deployment.yaml
kubectl apply -f backup-configmap.yaml
kubectl apply -f backup-secret.yaml
```

## Benefits After Migration

✅ **Easier updates:** `helm upgrade` instead of manual YAML edits  
✅ **Environment management:** Different values files for dev/prod  
✅ **Rollback capability:** `helm rollback` to previous versions  
✅ **Better documentation:** Built-in help and notes  
✅ **Validation:** Schema validation prevents configuration errors  
✅ **Versioning:** Track chart versions separately from app versions  

## Support

If you encounter issues during migration:
- Check the [README.md](README.md) for configuration details
- Review [INSTALL.md](INSTALL.md) for installation steps
- Use `helm template` to preview changes before applying
- Test in a separate namespace first

