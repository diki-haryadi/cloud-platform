# Step-by-Step Tutorial: Deploying Applications with AWS Fargate

## Prerequisites
- AWS Account with appropriate permissions
- AWS CLI installed and configured
- Basic understanding of Docker and containerization

## Step 1: Set Up Networking Infrastructure

### Create a VPC
1. Navigate to VPC Dashboard in AWS Console
2. Click "Create VPC"
3. Configure the following:
   - Name tag: `fargate-vpc`
   - IPv4 CIDR block: `10.0.0.0/16`
   - Enable DNS hostnames

### Create Subnets
1. Create two public subnets:
   - Subnet 1: `10.0.1.0/24` (AZ-1)
   - Subnet 2: `10.0.2.0/24` (AZ-2)
2. Create two private subnets:
   - Subnet 3: `10.0.3.0/24` (AZ-1)
   - Subnet 4: `10.0.4.0/24` (AZ-2)

### Configure Internet Access
1. Create Internet Gateway
   - Attach to VPC
2. Create NAT Gateway in public subnet
3. Configure route tables:
   - Public route table with Internet Gateway
   - Private route table with NAT Gateway

## Step 2: Create ECS Cluster

1. Open ECS Console
2. Click "Create Cluster"
3. Configure cluster:
   - Cluster name: `fargate-cluster`
   - VPC: Select created VPC
   - Choose Fargate launch type

## Step 3: Create Task Definition

1. Navigate to Task Definitions
2. Click "Create new Task Definition"
3. Configure:
   - Launch type: FARGATE
   - Task definition name: `fargate-app`
   - Task role: Create new role
   - Task execution role: Create new role
   - Task memory: 2GB
   - Task CPU: 1 vCPU
4. Add container:
   - Container name: `app-container`
   - Image: `nginx:latest` (example)
   - Port mappings: 80

## Step 4: Create Application Load Balancer

1. Go to EC2 Console > Load Balancers
2. Create Application Load Balancer:
   - Name: `fargate-alb`
   - Scheme: internet-facing
   - VPC: Select created VPC
   - Subnets: Select public subnets
3. Configure Security Group for ALB:
   - Create new security group: `fargate-alb-sg`
   - Inbound rules:
     - Allow HTTP (80) from 0.0.0.0/0
     - Allow HTTPS (443) from 0.0.0.0/0
   - Outbound rules:
     - Allow all traffic to 0.0.0.0/0
4. Create target group:
   - Target type: IP
   - Protocol: HTTP
   - Port: 80
5. In the EC2 Console under Load Balancing, select Target Groups and create new
- Configure basic settings:
    - Choose target type as "IP" for Fargate
    - Specify protocol (HTTP/HTTPS) and port
    - Select your VPC
    - Configure health checks:
    - Set health check path
    - Adjust health check intervals
    - Define healthy/unhealthy thresholds
    - Register targets:
        - For Fargate, this will be handled automatically when you create the ECS service
        - The service will register container IPs as targets
    - Advanced settings (optional):
        - Configure stickiness if needed
        - Set deregistration delay
        - Add tags for better organization

## Step 5: Create ECS Service

1. In ECS Cluster, click "Create Service"
2. Configure service:
   - Launch type: FARGATE
   - Service name: `fargate-service`
   - Task Definition: Select created task
   - Number of tasks: 2
3. Configure networking:
   - VPC: Select created VPC
   - Subnets: Select private subnets
   - Create security group: `fargate-task-sg`
   - Inbound rules:
     - Allow TCP 80 from ALB security group (`fargate-alb-sg`)
   - Outbound rules:
     - Allow all traffic to 0.0.0.0/0
4. Configure load balancing:
   - Select created ALB
   - Select target group

## Step 6: Configure Auto Scaling

1. In ECS Service, select "Update Service"
2. Configure Service Auto Scaling:
   - Minimum tasks: 2
   - Maximum tasks: 6
   - Target tracking scaling policies:
     - ECSServiceAverageCPUUtilization: 70%
     - ECSServiceAverageMemoryUtilization: 70%

## Step 7: Set Up Monitoring

1. Configure CloudWatch:
   - Enable Container Insights
   - Set up log groups
2. Create CloudWatch Dashboard:
   - Add CPU/Memory metrics
   - Add ALB metrics
   - Add service metrics

## Step 8: Testing and Validation

1. Access application through ALB DNS name
2. Monitor service metrics
3. Test auto scaling:
   - Generate load
   - Observe scaling behavior

## Best Practices

1. Security:
   - Use private subnets for tasks
   - Implement least privilege IAM roles
   - Regular security group audits

2. Cost Optimization:
   - Right-size task resources
   - Use auto scaling effectively
   - Monitor and optimize based on metrics

3. Operational:
   - Implement proper logging
   - Set up alerts for critical metrics
   - Regular backup and disaster recovery testing

## Troubleshooting

1. Common Issues:
   - Task failing to start
   - Network connectivity issues
   - Load balancer health check failures

2. Debug Steps:
   - Check CloudWatch Logs
   - Verify security group rules
   - Validate task definition
   - Check service events

## Next Steps

1. Implement CI/CD pipeline
2. Set up service discovery
3. Configure custom metrics and alerts
4. Implement blue/green deployments