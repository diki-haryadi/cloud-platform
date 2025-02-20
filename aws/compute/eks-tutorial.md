# Amazon EKS Setup Tutorial

## Prerequisites
- AWS CLI installed and configured
- kubectl installed
- eksctl installed
- AWS account with appropriate permissions
- Basic understanding of Kubernetes concepts

## Step 1: Install Required Tools

### Install AWS CLI
```bash
# macOS (using Homebrew)
brew install awscli

# Verify installation
aws --version
```

### Install kubectl
```bash
# macOS (using Homebrew)
brew install kubectl

# Verify installation
kubectl version
```

### Install eksctl
```bash
# macOS (using Homebrew)
brew tap weaveworks/tap
brew install weaveworks/tap/eksctl

# Verify installation
eksctl version
```

## Step 2: Configure AWS CLI
```bash
aws configure
```
Enter your AWS Access Key ID, Secret Access Key, default region, and output format.

## Step 3: Create an EKS Cluster

### Option 1: Using eksctl (Recommended)
```bash
eksctl create cluster \
    --name my-eks-cluster \
    --region us-west-2 \
    --node-type t3.medium \
    --nodes 2 \
    --nodes-min 1 \
    --nodes-max 3 \
    --managed
```

### Option 2: Using AWS Management Console
1. Open the Amazon EKS console
2. Click "Create cluster"
3. Configure cluster settings:
   - Name your cluster
   - Select Kubernetes version
   - Choose VPC and subnets
   - Configure security groups
   - Set up IAM roles

## Step 4: Configure kubectl
```bash
aws eks update-kubeconfig --name my-eks-cluster --region us-west-2
```

## Step 5: Verify Cluster Setup
```bash
# Check cluster info
kubectl cluster-info

# View nodes
kubectl get nodes

# Check pods in kube-system namespace
kubectl get pods -n kube-system
```

## Step 6: Deploy Sample Application

### Create a sample deployment
```yaml
# sample-app.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: nginx
```

```bash
# Apply the deployment
kubectl apply -f sample-app.yaml

# Verify deployment
kubectl get deployments
kubectl get services
```

## Step 7: Cluster Management

### Scale node group
```bash
eksctl scale nodegroup --cluster=my-eks-cluster --name=ng-1 --nodes=3
```

### Update cluster version
```bash
eksctl upgrade cluster --name=my-eks-cluster --approve
```

## Best Practices
1. **Security**
   - Use AWS IAM roles for service accounts
   - Implement network policies
   - Regular security updates

2. **Monitoring**
   - Set up CloudWatch Container Insights
   - Deploy Prometheus and Grafana
   - Configure logging

3. **Cost Optimization**
   - Use spot instances for non-critical workloads
   - Implement cluster autoscaling
   - Regular resource optimization

4. **High Availability**
   - Deploy across multiple AZs
   - Use managed node groups
   - Implement proper backup strategies

## Cleanup
```bash
# Delete sample application
kubectl delete -f sample-app.yaml

# Delete cluster
eksctl delete cluster --name my-eks-cluster
```

## Troubleshooting
1. **Unable to connect to cluster**
   - Verify AWS credentials
   - Check VPC and security group settings
   - Ensure kubectl is properly configured

2. **Node group issues**
   - Check IAM roles and policies
   - Verify Auto Scaling group settings
   - Review CloudWatch logs

## Additional Resources
- [EKS Documentation](https://docs.aws.amazon.com/eks/)
- [eksctl Documentation](https://eksctl.io/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- AWS EKS Workshop