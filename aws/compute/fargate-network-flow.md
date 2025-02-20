# AWS Fargate Network Flow Diagram

```mermaid
graph TB
    Internet((Internet))
    IGW[Internet Gateway]
    ALB[Application Load Balancer]
    NATGW[NAT Gateway]
    FargateTask1[Fargate Task 1]
    FargateTask2[Fargate Task 2]
    
    subgraph VPC[VPC - 10.0.0.0/16]
        subgraph PublicSubnets[Public Subnets]
            subgraph AZ1-Public[AZ-1 - 10.0.1.0/24]
                ALB
                NATGW
            end
            subgraph AZ2-Public[AZ-2 - 10.0.2.0/24]
                ALB-2[ALB]
            end
        end
        
        subgraph PrivateSubnets[Private Subnets]
            subgraph AZ1-Private[AZ-1 - 10.0.3.0/24]
                FargateTask1
            end
            subgraph AZ2-Private[AZ-2 - 10.0.4.0/24]
                FargateTask2
            end
        end
    end
    
    Internet -->|HTTPS/HTTP| IGW
    IGW -->|Port 80/443| ALB
    IGW -->|Port 80/443| ALB-2
    
    ALB -->|Target Group| FargateTask1
    ALB -->|Target Group| FargateTask2
    ALB-2 -->|Target Group| FargateTask1
    ALB-2 -->|Target Group| FargateTask2
    
    FargateTask1 -->|Outbound Traffic| NATGW
    FargateTask2 -->|Outbound Traffic| NATGW
    NATGW -->|Internet Access| IGW
    
    classDef public fill:#ff9,stroke:#333,stroke-width:2px;
    classDef private fill:#9cf,stroke:#333,stroke-width:2px;
    classDef gateway fill:#f96,stroke:#333,stroke-width:2px;
    
    class ALB,ALB-2 public;
    class FargateTask1,FargateTask2 private;
    class IGW,NATGW gateway;

```

## Network Flow Description

1. **Internet to VPC**:
   - External traffic from the internet reaches the VPC through the Internet Gateway (IGW)
   - The IGW serves as the entry/exit point for all internet traffic

2. **Load Balancer Layer**:
   - Traffic is directed to Application Load Balancers in public subnets
   - ALBs are distributed across two Availability Zones for high availability
   - Security groups control inbound traffic to ALBs (typically ports 80/443)

3. **Container Layer**:
   - ALBs forward requests to Fargate tasks in private subnets
   - Tasks are distributed across AZs for high availability
   - Security groups control traffic between ALB and Fargate tasks

4. **Outbound Traffic Flow**:
   - Fargate tasks in private subnets use NAT Gateway for outbound internet access
   - NAT Gateway is placed in public subnet
   - Outbound traffic flows: Fargate Task → NAT Gateway → Internet Gateway → Internet

## Security Considerations

1. **Public Access Control**:
   - Only ALBs are in public subnets
   - Internet traffic can only reach the application through ALB

2. **Private Resources Protection**:
   - Fargate tasks run in private subnets
   - No direct internet access to containers
   - Outbound internet access controlled via NAT Gateway

3. **Network Isolation**:
   - Clear separation between public and private resources
   - Security groups provide additional layer of access control
   - Network ACLs can be used for subnet-level security