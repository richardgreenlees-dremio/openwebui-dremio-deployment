# Helm Chart Summary

## Overview

A complete Helm chart has been created for deploying Open WebUI with Dremio MCP integration on Kubernetes.

## Chart Structure

```
helm-chart/
├── Chart.yaml                      # Chart metadata
├── values.yaml                     # Default configuration values
├── values.schema.json              # Values validation schema
├── values-development.yaml         # Development environment preset
├── values-production.yaml          # Production environment preset
├── .helmignore                     # Files to ignore when packaging
├── README.md                       # Full documentation
├── INSTALL.md                      # Detailed installation guide
├── QUICK_START.md                  # Quick start guide
└── templates/
    ├── _helpers.tpl                # Template helper functions
    ├── NOTES.txt                   # Post-installation notes
    ├── namespace.yaml              # Namespace creation
    ├── secrets.yaml                # Dremio credentials secret
    ├── configmap.yaml              # MCPO and Dremio configuration
    ├── pvc.yaml                    # Persistent volume claims
    ├── ollama-deployment.yaml      # Ollama deployment
    ├── mcpo-deployment.yaml        # MCPO deployment
    ├── openwebui-deployment.yaml   # Open WebUI deployment
    ├── services.yaml               # All services
    └── jobs.yaml                   # Ollama model loader job
```

## Key Features

### 1. **Templated Configuration**
- All values are parameterized via `values.yaml`
- Support for multiple environments (dev/prod)
- Easy customization without modifying templates

### 2. **Flexible Deployment Options**
- Enable/disable individual components
- Configurable resource requests and limits
- Support for different service types (ClusterIP, NodePort, LoadBalancer)
- Optional node affinity

### 3. **Storage Management**
- Configurable PVC sizes
- Support for custom storage classes
- Separate volumes for Ollama and MCPO

### 4. **Security**
- Credentials stored in Kubernetes secrets
- Environment variable substitution for sensitive data
- Values schema validation

### 5. **Production Ready**
- Health checks and resource limits
- Init containers for proper startup ordering
- Configurable load balancer annotations
- Support for internal load balancers

## Installation Methods

### Quick Install
```bash
helm install openwebui ./helm-chart \
  --set dremio.credentials.pat="YOUR_PAT" \
  --set dremio.credentials.uri="http://YOUR_HOST:9047"
```

### Using Values File
```bash
cat > my-values.yaml <<EOF
dremio:
  credentials:
    pat: "YOUR_PAT"
    uri: "http://YOUR_HOST:9047"
EOF

helm install openwebui ./helm-chart -f my-values.yaml
```

### Development Environment
```bash
helm install openwebui ./helm-chart \
  -f helm-chart/values-development.yaml \
  -f my-values.yaml
```

### Production Environment
```bash
helm install openwebui ./helm-chart \
  -f helm-chart/values-production.yaml \
  -f my-values.yaml
```

## Configuration Highlights

### Resource Presets

**Development (values-development.yaml):**
- Minimal resources (500m CPU, 1Gi RAM per component)
- Smaller storage (10Gi for Ollama, 5Gi for MCPO)
- NodePort/ClusterIP services
- Smaller model (llama3.2)
- Node affinity disabled

**Production (values-production.yaml):**
- Higher resources (2-8 CPU, 8-32Gi RAM)
- Larger storage (100Gi for Ollama, 50Gi for MCPO)
- LoadBalancer services
- Production model (llama3.1)
- Node affinity enabled
- Custom storage classes

### Customizable Components

**Ollama:**
- Image repository and tag
- Service type and port
- Resource limits
- Persistence settings
- Model to load
- Model loader job configuration

**MCPO:**
- Image repository and tag
- Service type and port
- Resource limits
- Persistence settings
- MCP servers configuration
- Load balancer annotations

**Open WebUI:**
- Image repository and tag
- Service type and port
- Resource limits

**Dremio:**
- Credentials (PAT and URI)
- MCP server configuration

## Advanced Features

### 1. **Dynamic MCP Server Configuration**
The ConfigMap template dynamically builds the `mcpo-config.json` from values:

```yaml
mcpo:
  mcpServers:
    dremio:
      enabled: true
      command: uv
      args: [...]
      config:
        enable_search: false
        server_mode: FOR_DATA_PATTERNS
```

### 2. **Init Container Pattern**
- `dremio-mcp-loader`: Clones the Dremio MCP repository
- `config-substitution`: Injects secrets into configuration files

### 3. **Helper Templates**
- Common labels
- Selector labels
- Node affinity configuration
- Reusable across all resources

### 4. **Post-Install Notes**
Displays helpful information after installation:
- How to access the application
- Service URLs
- Useful kubectl commands
- Warnings for default credentials

## Testing the Chart

### Lint the Chart
```bash
helm lint ./helm-chart
```

### Dry Run
```bash
helm install openwebui ./helm-chart \
  -f my-values.yaml \
  --dry-run --debug
```

### Template Output
```bash
helm template openwebui ./helm-chart -f my-values.yaml
```

### Validate Values
```bash
helm install openwebui ./helm-chart \
  -f my-values.yaml \
  --dry-run --debug --validate
```

## Upgrading

```bash
# Upgrade with new values
helm upgrade openwebui ./helm-chart -f my-values.yaml

# Upgrade with specific values
helm upgrade openwebui ./helm-chart \
  --set ollama.modelLoader.model=llama3.2

# Rollback if needed
helm rollback openwebui
```

## Documentation

- **README.md**: Complete documentation with all parameters
- **INSTALL.md**: Step-by-step installation guide
- **QUICK_START.md**: Get started in 5 minutes
- **values.yaml**: Inline comments for all options
- **NOTES.txt**: Post-installation instructions

## Next Steps

1. **Test the chart:**
   ```bash
   helm lint ./helm-chart
   helm install test ./helm-chart -f my-values.yaml --dry-run
   ```

2. **Package the chart:**
   ```bash
   helm package ./helm-chart
   ```

3. **Publish to a chart repository:**
   ```bash
   # Upload to your chart repository
   ```

4. **Add CI/CD:**
   - Automated testing
   - Chart versioning
   - Automated releases

## Benefits Over Plain YAML

✅ **Reusability**: One chart, multiple environments  
✅ **Maintainability**: Single source of truth  
✅ **Flexibility**: Easy customization via values  
✅ **Validation**: Schema validation for values  
✅ **Documentation**: Built-in help and notes  
✅ **Versioning**: Track chart versions  
✅ **Rollback**: Easy rollback to previous versions  
✅ **Templating**: DRY principle, no duplication  
✅ **Best Practices**: Follows Helm conventions  

## Comparison: Plain YAML vs Helm Chart

| Feature | Plain YAML | Helm Chart |
|---------|------------|------------|
| Multiple environments | Multiple file sets | Single chart + values files |
| Customization | Edit YAML directly | Override values |
| Secrets management | Separate files | Templated secrets |
| Upgrades | Manual kubectl apply | `helm upgrade` |
| Rollback | Manual | `helm rollback` |
| Documentation | Separate README | Built-in NOTES.txt |
| Validation | Manual | Schema validation |
| Versioning | Git only | Chart versions + Git |

