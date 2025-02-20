# Amazon EC2 (Elastic Compute Cloud) Guide

## Overview
Amazon Elastic Compute Cloud (EC2) is a web service that provides secure, resizable compute capacity in the cloud. It is designed to make web-scale cloud computing easier for developers.

## Core Concepts

### Instance Types
- General Purpose (t3, m5, m6)
- Compute Optimized (c5, c6)
- Memory Optimized (r5, r6, x1)
- Storage Optimized (i3, d2)
- Accelerated Computing (p3, g4)

### Amazon Machine Images (AMI)
- Pre-configured templates for instances
- Types of AMIs:
  - Amazon-provided AMIs
  - AWS Marketplace AMIs
  - Community AMIs
  - Custom AMIs

### Purchase Options
1. On-Demand Instances
   - Pay by the hour
   - No long-term commitments
   - Ideal for short-term workloads

2. Reserved Instances
   - 1 or 3-year term commitment
   - Up to 72% discount compared to On-Demand
   - Payment options:
     - All Upfront
     - Partial Upfront
     - No Upfront

3. Spot Instances
   - Up to 90% discount
   - Can be interrupted with 2-minute notice
   - Ideal for fault-tolerant workloads

## Storage Options

### Instance Store
- Temporary block-level storage
- Physically attached to the host computer
- Data lost when instance stops/terminates

### Amazon EBS (Elastic Block Store)
- Persistent block storage volumes
- Types:
  - General Purpose SSD (gp2/gp3)
  - Provisioned IOPS SSD (io1/io2)
  - Throughput Optimized HDD (st1)
  - Cold HDD (sc1)

## Networking

### VPC Integration
- Launch instances in a Virtual Private Cloud
- Configure security groups and NACLs
- Public and private subnets

### IP Addressing
- Public IPv4 addresses
- Private IPv4 addresses
- Elastic IP addresses
- IPv6 addresses

## Security

### Security Groups
- Virtual firewalls for instances
- Control inbound and outbound traffic
- Stateful packet filtering

### Key Pairs
- Secure SSH access to instances
- Public-private key cryptography
- Key management best practices

## Auto Scaling

### Auto Scaling Groups (ASG)
- Automatically scale EC2 capacity
- Maintain instance count
- Replace unhealthy instances

### Scaling Policies
1. Target Tracking
   - Maintain specific metric value
   - Example: CPU utilization at 70%

2. Simple/Step Scaling
   - Add/remove instances based on alarms
   - Configurable step adjustments

3. Scheduled Scaling
   - Scale based on predictable patterns
   - Time-based scaling rules

## Monitoring and Management

### CloudWatch Integration
- Basic monitoring (5-minute intervals)
- Detailed monitoring (1-minute intervals)
- Custom metrics

### Instance Management
- Start/Stop instances
- Hibernate instances
- Instance metadata and user data
- Tags for resource organization

## Best Practices

### Cost Optimization
1. Right-sizing instances
2. Using appropriate purchase options
3. Implementing auto scaling
4. Monitoring and analyzing usage

### High Availability
1. Using multiple Availability Zones
2. Implementing load balancing
3. Creating redundant instances
4. Using auto scaling groups

### Security
1. Regular security group audits
2. Implementing least privilege access
3. Encrypting data at rest and in transit
4. Regular patching and updates

### Performance
1. Choosing appropriate instance types
2. Optimizing storage configuration
3. Network optimization
4. Regular performance monitoring

## Common Operations

### Launch an EC2 Instance
1. Choose an AMI
2. Select instance type
3. Configure instance details
4. Add storage
5. Configure security group
6. Review and launch

### Connect to an Instance
1. For Linux:
   ```bash
   ssh -i /path/to/key.pem ec2-user@instance-public-ip
   ```
2. For Windows:
   - Use Remote Desktop Protocol (RDP)
   - Decrypt administrator password using key pair

### Create a Custom AMI
1. Prepare the instance
2. Create image from instance
3. Tag and manage AMI

## Troubleshooting

### Common Issues
1. Instance launch failures
2. Connection issues
3. Performance problems
4. Capacity errors

### Resolution Steps
1. Check instance status
2. Review security groups
3. Verify network configuration
4. Analyze system logs

## Learning Resources
- [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/)
- [EC2 User Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/)
- [EC2 API Reference](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/)
- AWS Workshops and Labs