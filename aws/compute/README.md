# AWS Compute Services

## EC2 and Auto Scaling

### Amazon EC2 (Elastic Compute Cloud)
- Virtual servers in the cloud
- Instance types and sizing
- AMIs (Amazon Machine Images)
- Spot Instances vs On-Demand vs Reserved
- Instance storage options

### Auto Scaling
- Auto Scaling Groups (ASG)
- Launch Templates/Configurations
- Scaling policies
  - Target tracking
  - Simple/Step scaling
  - Scheduled scaling
- Integration with ELB

## ECS (Elastic Container Service)
- Container management
- Task definitions
- Services and tasks

### AWS Fargate
- Serverless container execution
- Task networking
- Container image requirements
- Service auto scaling
- Load balancer integration
- Security and IAM roles
- Cost optimization strategies
- Monitoring and logging
- CI/CD integration

### ECS Launch Types
- Fargate vs EC2 launch type
- Load balancing
- Service discovery

## EKS (Elastic Kubernetes Service)
- Kubernetes architecture
- Cluster management
- Node groups
- Pod deployment
- Service mesh
- Monitoring and logging

## Hands-on Projects
1. Deploy a scalable web application using EC2 and Auto Scaling
2. Set up container orchestration with ECS
3. Build a Kubernetes cluster with EKS
4. Deploy containerized applications using Fargate
   - Set up ECS cluster with Fargate
   - Create task definitions and services
   - Implement auto scaling
   - Configure load balancing
   - Monitor container performance
5. Implement serverless applications with Lambda
   - Create Lambda functions
   - Set up API Gateway integration
   - Configure event triggers
   - Implement error handling
   - Monitor function performance

## Best Practices
- Right-sizing instances
- Cost optimization
- High availability design
- Security hardening
- Performance monitoring
- Resource tagging
- Container optimization
- Serverless architecture patterns

## Learning Resources
- [EC2 Documentation](https://docs.aws.amazon.com/ec2/)
- [ECS Documentation](https://docs.aws.amazon.com/ecs/)
- [EKS Documentation](https://docs.aws.amazon.com/eks/)
- [Fargate Documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html)
- [Lambda Documentation](https://docs.aws.amazon.com/lambda/)
- AWS Workshops
- Hands-on Labs