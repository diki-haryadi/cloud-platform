# Tutorial: Accessing AWS Lambda through Application Load Balancer

## Overview
This tutorial demonstrates how to expose AWS Lambda functions to the internet using an Application Load Balancer (ALB), which provides HTTP/HTTPS access with advanced routing capabilities.

## Prerequisites
- AWS Account with appropriate permissions
- AWS CLI installed and configured
- Basic understanding of networking concepts
- Basic Python programming knowledge

## Step 1: Create a Lambda Function

### 1.1 Create Basic Lambda Function
1. Open AWS Management Console
2. Navigate to Lambda service
3. Click "Create function"
4. Configure settings:
   - Name: `alb-demo-function`
   - Runtime: Python 3.9
   - Architecture: x86_64
   - Permissions: Create new role with basic Lambda permissions

### 1.2 Implement Function Code
```python
import json

def lambda_handler(event, context):
    try:
        # Parse ALB event
        request_context = event.get('requestContext', {})
        http_method = request_context.get('http', {}).get('method')
        path = request_context.get('http', {}).get('path')
        
        # Get headers
        headers = event.get('headers', {})
        
        # Get query string parameters
        query_string = event.get('queryStringParameters', {})
        
        # Get body if any
        body = event.get('body', '')
        if body:
            body = json.loads(body)
        
        # Create response
        response = {
            'message': 'Hello from Lambda!',
            'method': http_method,
            'path': path,
            'headers': headers,
            'query_string': query_string,
            'body': body
        }
        
        return {
            'statusCode': 200,
            'headers': {
                'Content-Type': 'application/json'
            },
            'body': json.dumps(response)
        }
    except Exception as e:
        return {
            'statusCode': 500,
            'body': json.dumps({'error': str(e)})
        }
```

## Step 2: Set Up VPC and Networking

### 2.1 Create or Configure VPC
1. Go to VPC Console
2. Ensure you have:
   - At least two public subnets in different AZs
   - Internet Gateway attached
   - Proper route tables configured

### 2.2 Configure Security Groups
1. Create ALB Security Group:
   - Name: `alb-lambda-sg`
   - Inbound rules: Allow HTTP (80) and HTTPS (443) from anywhere
   - Outbound rules: Allow all

## Step 3: Create and Configure Application Load Balancer

### 3.1 Create ALB
1. Go to EC2 Console → Load Balancers
2. Click "Create Load Balancer"
3. Choose "Application Load Balancer"
4. Configure settings:
   - Name: `lambda-alb`
   - Scheme: internet-facing
   - IP address type: ipv4
   - VPC: Select your VPC
   - Subnets: Select at least two public subnets
   - Security Group: Select `alb-lambda-sg`

### 3.2 Create Target Group
1. Click "Create target group":
   - Target type: Lambda
   - Name: `lambda-target-group`
   - Register Lambda function: Select `alb-demo-function`

### 3.3 Configure Listener
1. Add listener:
   - Protocol: HTTP
   - Port: 80
   - Default action: Forward to lambda-target-group

## Step 4: Test the Integration

### 4.1 Using curl
```bash
# Test GET request
curl -X GET http://[your-alb-dns-name]/test

# Test POST request
curl -X POST http://[your-alb-dns-name]/test \
  -H "Content-Type: application/json" \
  -d '{"message":"Hello ALB!"}'
```

### 4.2 Using Python
```python
import requests
import json

base_url = 'http://[your-alb-dns-name]'
data = {'message': 'Hello ALB!'}

# GET request
response = requests.get(f'{base_url}/test')
print(json.dumps(response.json(), indent=2))

# POST request
response = requests.post(f'{base_url}/test', json=data)
print(json.dumps(response.json(), indent=2))
```

## Step 5: Implement Security

### 5.1 Add HTTPS Listener
1. Request or import SSL certificate in ACM
2. Add HTTPS listener to ALB:
   - Protocol: HTTPS
   - Port: 443
   - Default certificate: Select your certificate
   - Security policy: ELBSecurityPolicy-2016-08

### 5.2 Configure Security Rules
1. Update security group rules:
   - Remove HTTP access if HTTPS-only is required
   - Restrict source IPs if needed

2. Implement IP-based restrictions:
   - Use ALB IP address ranges
   - Configure WAF rules if needed

## Best Practices

1. Security:
   - Use HTTPS with valid certificates
   - Implement proper security groups
   - Enable access logs
   - Consider using AWS WAF

2. High Availability:
   - Use multiple Availability Zones
   - Monitor health checks
   - Set up proper alarms

3. Performance:
   - Configure appropriate timeout values
   - Monitor ALB metrics
   - Use proper target group settings

4. Monitoring:
   - Enable ALB access logs
   - Set up CloudWatch alarms
   - Monitor Lambda metrics
   - Track error rates

5. Cost Optimization:
   - Monitor ALB usage
   - Optimize Lambda concurrency
   - Use appropriate instance types

## Troubleshooting

1. Common Issues:
   - Connection timeouts
   - SSL/TLS certificate issues
   - Health check failures
   - Lambda permission issues

2. Debugging Tips:
   - Check ALB access logs
   - Verify security group rules
   - Monitor CloudWatch logs
   - Test Lambda function directly

## Conclusion
You now have a Lambda function accessible through an Application Load Balancer. This setup provides a robust and secure way to expose your Lambda functions to the internet with advanced routing capabilities and built-in high availability.