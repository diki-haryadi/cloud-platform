# AWS CLI and eksctl Commands Reference for EKS

## AWS CLI Commands

### Authentication and Configuration
```bash
# Configure AWS CLI with credentials
aws configure

# Get current caller identity
aws sts get-caller-identity

# List available EKS clusters
aws eks list-clusters
```

### EKS Cluster Management
```bash
# Describe EKS cluster
aws eks describe-cluster --name cluster-name

# Update kubeconfig for EKS cluster
aws eks update-kubeconfig --name cluster-name --region region-name

# List nodegroups in a cluster
aws eks list-nodegroups --cluster-name cluster-name

# Describe nodegroup
aws eks describe-nodegroup --cluster-name cluster-name --nodegroup-name nodegroup-name
```

## eksctl Commands

### Cluster Creation and Deletion
```bash
# Create a basic EKS cluster
eksctl create cluster \
    --name cluster-name \
    --region region-name \
    --node-type instance-type \
    --nodes 2 \
    --nodes-min 1 \
    --nodes-max 3 \
    --managed

# Delete cluster
eksctl delete cluster --name cluster-name
```

### Node Group Management
```bash
# Create a nodegroup
eksctl create nodegroup \
    --cluster cluster-name \
    --name nodegroup-name \
    --node-type instance-type \
    --nodes 2

# Scale nodegroup
eksctl scale nodegroup \
    --cluster cluster-name \
    --name nodegroup-name \
    --nodes 3

# Delete nodegroup
eksctl delete nodegroup \
    --cluster cluster-name \
    --name nodegroup-name
```

### Cluster Updates and Maintenance
```bash
# Upgrade cluster version
eksctl upgrade cluster --name cluster-name --approve

# Update nodegroup
eksctl upgrade nodegroup \
    --cluster cluster-name \
    --name nodegroup-name

# Enable cluster logging
eksctl utils update-cluster-logging \
    --cluster cluster-name \
    --enable-types all
```

### IAM and Authentication
```bash
# Create IAM OIDC provider
eksctl utils associate-iam-oidc-provider \
    --cluster cluster-name \
    --approve

# Create IAM service account
eksctl create iamserviceaccount \
    --cluster cluster-name \
    --namespace namespace \
    --name service-account-name \
    --attach-policy-arn policy-arn \
    --approve
```

### Cluster Information and Diagnostics
```bash
# Get cluster information
eksctl get cluster --name cluster-name

# List nodegroups
eksctl get nodegroup --cluster cluster-name

# Get cluster configuration
eksctl get cluster --name cluster-name -o yaml
```

## Common Options and Flags

- `--region`: Specify AWS region
- `--profile`: Use specific AWS profile
- `--version`: Specify Kubernetes version
- `--managed`: Use managed node groups
- `--dry-run`: Preview changes without applying
- `--approve`: Auto-approve changes
- `-o, --output`: Specify output format (yaml/json)

## Best Practices

1. Always specify region explicitly
2. Use managed node groups when possible
3. Implement proper tagging for resources
4. Review changes with --dry-run before applying
5. Keep cluster and nodegroups updated
6. Use IAM roles for service accounts
7. Enable and monitor cluster logging