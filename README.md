# Azure Kubernetes Service (AKS) - Best Practices Implementation

This Terraform configuration implements Azure Kubernetes Service (AKS) following Microsoft's best practices for production workloads.

## 🏗️ Architecture Overview

This configuration creates an enterprise-ready AKS cluster with:

- **High Availability**: Multi-zone deployment across availability zones
- **Security**: Azure AD integration, RBAC, network policies, and workload identity
- **Scalability**: Auto-scaling node pools with optimized configurations
- **Monitoring**: Comprehensive logging and metrics collection
- **Cost Optimization**: Spot instances support and right-sized node pools

## 🔧 Key Features Implemented

### Security Best Practices
- ✅ **Azure AD Integration**: Managed Azure AD with RBAC
- ✅ **Network Policies**: Azure CNI with network policies enabled
- ✅ **Private Cluster**: Optional private cluster deployment
- ✅ **Workload Identity**: Secure pod-to-Azure resource access
- ✅ **Key Vault Integration**: Secrets provider with rotation
- ✅ **Least Privilege**: Proper RBAC and role assignments

### Reliability & High Availability
- ✅ **Availability Zones**: Multi-zone node pool distribution
- ✅ **Auto-scaling**: Horizontal pod and cluster autoscaling
- ✅ **System Node Pool**: Dedicated system workload isolation
- ✅ **Upgrade Strategy**: Controlled rolling updates with surge settings
- ✅ **SLA Tiers**: Support for Standard and Premium SLA tiers

### Performance & Scaling
- ✅ **Multiple Node Pools**: Specialized pools for different workloads
- ✅ **GPU Support**: GPU-enabled nodes for AI/ML workloads
- ✅ **Spot Instances**: Cost-optimized spot instance support
- ✅ **Enhanced Storage**: Multiple CSI drivers enabled
- ✅ **Optimized VM Sizes**: Right-sized instances for each workload

### Monitoring & Observability
- ✅ **Log Analytics**: Integrated monitoring workspace
- ✅ **Diagnostic Settings**: Comprehensive log collection
- ✅ **Container Insights**: Built-in monitoring solution
- ✅ **Metrics Collection**: Performance and resource metrics

## 📋 Node Pool Strategy

### System Node Pool (`system`)
- **Purpose**: Critical system components (kube-system pods)
- **VM Size**: `Standard_D4s_v3` (configurable)
- **Scaling**: 1-5 nodes with auto-scaling
- **Isolation**: Optional taints to prevent user workloads

### CPU Node Pool (`cpunode`)
- **Purpose**: General compute workloads
- **VM Size**: `Standard_D8s_v3` (configurable)
- **Scaling**: 0-10 nodes with auto-scaling
- **Labels**: `workload-type=cpu`, `node-type=compute`

### AI Node Pool (`ainode`)
- **Purpose**: AI/ML and GPU workloads
- **VM Size**: Dynamic (GPU: `Standard_NC24ads_A100_v4`, CPU: `Standard_D32as_v5`)
- **Scaling**: 0-5 nodes with auto-scaling
- **Features**: GPU taints, spot instance support

## 🚀 Quick Start

### Prerequisites
- Azure CLI installed and authenticated
- Terraform >= 1.0
- Appropriate Azure permissions

### Basic Deployment

```hcl
module "aks" {
  source = "./path-to-this-module"

  # Core configuration
  cluster_name = "my-aks-cluster"
  location     = "East US"
  dns_prefix   = "myaks"

  # Network configuration
  subnet_id   = "/subscriptions/.../subnets/aks-subnet"
  aks_vnet    = "my-vnet"
  aks_vnet_rg = "networking-rg"

  # Security
  admin_group_object_ids = ["group-object-id-1", "group-object-id-2"]
  
  # High availability
  availability_zones = ["1", "2", "3"]
  sku_tier          = "Standard"

  tags = {
    Environment = "production"
    Project     = "my-project"
  }
}
```

### Advanced Configuration

```hcl
module "aks" {
  source = "./path-to-this-module"

  # Core configuration
  cluster_name       = "enterprise-aks"
  location          = "East US"
  dns_prefix        = "entaks"
  kubernetes_version = "1.28.5"
  sku_tier          = "Premium"

  # Network configuration
  subnet_id                    = var.subnet_id
  aks_vnet                    = var.vnet_name
  aks_vnet_rg                 = var.network_rg
  private_cluster_enabled     = true
  dns_service_ip              = "10.1.0.10"
  service_cidr                = "10.1.0.0/16"

  # Security & Identity
  admin_group_object_ids      = ["aad-group-1", "aad-group-2"]
  azure_rbac_enabled         = true
  workload_identity_enabled  = true
  acr_id                     = var.acr_id

  # High Availability
  availability_zones = ["1", "2", "3"]

  # Node pool scaling
  default_min_count = 2
  default_max_count = 10
  cpu_min_count     = 1
  cpu_max_count     = 20
  ai_min_count      = 0
  ai_max_count      = 5

  # GPU/AI workloads
  cluster_type         = "GPU"
  use_spot_instances   = true
  spot_max_price       = 0.5

  # System isolation
  create_separate_system_pool = true
  enable_system_node_taints   = true

  # Monitoring
  log_retention_days = 90

  # Maintenance
  maintenance_window = {
    day   = "Sunday"
    hours = [2, 3, 4, 5]
  }

  tags = {
    Environment = "production"
    CostCenter  = "engineering"
    Project     = "ml-platform"
  }
}
```

## 🔧 Configuration Options

### Security Configuration

| Variable | Description | Default | Best Practice |
|----------|-------------|---------|---------------|
| `admin_group_object_ids` | Azure AD admin groups | `[]` | Use AD groups, not individual users |
| `azure_rbac_enabled` | Enable Azure RBAC | `true` | Always enable for production |
| `workload_identity_enabled` | Enable workload identity | `true` | Required for pod-to-Azure access |
| `private_cluster_enabled` | Private cluster | `true` | Enable for enhanced security |

### High Availability Configuration

| Variable | Description | Default | Best Practice |
|----------|-------------|---------|---------------|
| `availability_zones` | AZ distribution | `["1","2","3"]` | Use all available zones |
| `sku_tier` | Cluster SLA tier | `"Standard"` | Use Premium for critical workloads |
| `default_min_count` | Min system nodes | `1` | Minimum 2 for HA |
| `default_max_count` | Max system nodes | `5` | Size based on workload |

### Cost Optimization

| Variable | Description | Default | Best Practice |
|----------|-------------|---------|---------------|
| `use_spot_instances` | Use spot for AI pool | `false` | Enable for non-critical AI workloads |
| `spot_max_price` | Max spot price | `-1` | Set appropriate budget limit |
| `cpu_min_count` | Min CPU nodes | `0` | Start at 0 for cost savings |

## 📊 Monitoring & Observability

The configuration automatically sets up:

- **Log Analytics Workspace**: Centralized logging
- **Container Insights**: AKS-specific monitoring
- **Diagnostic Settings**: API server, controller, scheduler logs
- **Metrics Collection**: Resource and performance metrics

### Key Logs Collected:
- `kube-apiserver`: API server operations
- `kube-controller-manager`: Controller activities
- `kube-scheduler`: Scheduling decisions
- `kube-audit`: Audit trail
- `cluster-autoscaler`: Scaling events

## 🛡️ Security Hardening

### Network Security
- Azure CNI with network policies
- Private cluster option
- Controlled outbound access
- Standard load balancer

### Identity & Access
- Azure AD integration
- RBAC enabled by default
- Workload identity for pods
- Minimal required permissions

### Secrets Management
- Key Vault integration
- Secret rotation enabled
- No hardcoded credentials

## 🔄 Upgrade Strategy

The configuration implements safe upgrade practices:

- **Max Surge**: 33% for faster, safer upgrades
- **Maintenance Windows**: Scheduled maintenance
- **Version Management**: Explicit Kubernetes versions
- **Rolling Updates**: Minimized downtime

## 💰 Cost Optimization Features

- **Auto-scaling**: Right-size based on demand
- **Spot Instances**: Up to 90% cost savings for AI workloads
- **Ephemeral OS Disks**: Better performance, lower cost
- **Resource Tagging**: Cost allocation and tracking

## 🚨 Production Checklist

Before deploying to production:

- [ ] Review and set appropriate `admin_group_object_ids`
- [ ] Configure network CIDR ranges appropriately
- [ ] Set up Azure Container Registry integration
- [ ] Configure appropriate node pool sizes
- [ ] Set up backup and disaster recovery
- [ ] Configure monitoring alerts
- [ ] Review security policies
- [ ] Test upgrade procedures

## 🤝 Contributing

When making changes:

1. Follow Azure AKS best practices
2. Update documentation
3. Test in non-production environment
4. Consider backward compatibility

## 📚 References

- [Azure AKS Best Practices](https://docs.microsoft.com/en-us/azure/aks/best-practices)
- [AKS Cluster Reliability](https://docs.microsoft.com/en-us/azure/aks/best-practices-app-cluster-reliability)
- [AKS Security Best Practices](https://docs.microsoft.com/en-us/azure/aks/security-baseline)
- [AKS Performance Best Practices](https://docs.microsoft.com/en-us/azure/aks/best-practices-performance-scale)
