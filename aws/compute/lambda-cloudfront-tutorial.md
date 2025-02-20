# Tutorial: Accessing AWS Lambda through CloudFront

## Overview
This tutorial demonstrates how to expose AWS Lambda functions to the internet using Amazon CloudFront, providing global content delivery with edge caching capabilities.

## Prerequisites
- AWS Account with appropriate permissions
- AWS CLI installed and configured
- Basic understanding of CDN concepts
- Basic Python programming knowledge

## Step 1: Create a Lambda Function

### 1.1 Create Basic Lambda Function
1. Open AWS Management Console
2. Navigate to Lambda service
3. Click "Create function"
4. Configure settings:
   - Name: `cloudfront-demo-function`
   - Runtime: Python 3.9
   - Architecture: x86_64
   - Permissions: Create new role with basic Lambda permissions

### 1.2 Implement Function Code
```python
import json

def lambda_handler(event, context):
    try:
        # Parse CloudFront event
        cf_event = event['Records'][0]['cf']
        request = cf_event['request']
        
        # Get request details
        method = request['method']
        uri = request['uri']
        headers = request['headers']
        query_string = request.get('querystring', '')
        
        # Create response
        response = {
            'message': 'Hello from Lambda@Edge!',
            'method': method,
            'uri': uri,
            'headers': headers,
            'query_string': query_string
        }
        
        return {
            'status': '200',
            'statusDescription': 'OK',
            'headers': {
                'content-type': [{
                    'key': 'Content-Type',
                    'value': 'application/json'
                }]
            },
            'body': json.dumps(response)
        }
    except Exception as e:
        return {
            'status': '500',
            'statusDescription': 'Internal Server Error',
            'body': json.dumps({'error': str(e)})
        }
```

## Step 2: Configure Lambda@Edge

### 2.1 Prepare Lambda Function for Edge
1. Ensure function:
   - Is in us-east-1 region
   - Has required IAM permissions
   - Has timeout ≤ 5 seconds
   - Memory between 128MB and 10GB

### 2.2 Create Function Version
1. Go to Lambda function
2. Click "Actions" → "Publish new version"
3. Add version description
4. Note the version ARN

## Step 3: Create and Configure CloudFront Distribution

### 3.1 Create Distribution
1. Go to CloudFront Console
2. Click "Create Distribution"
3. Configure settings:
   - Origin domain: Your API endpoint or S3 bucket
   - Origin path: Optional path prefix
   - Name: `lambda-edge-demo`
   - Price class: Choose based on your needs

### 3.2 Configure Behaviors
1. Create or edit default behavior:
   - Path pattern: Default (*)
   - Viewer protocol policy: Redirect HTTP to HTTPS
   - Allowed HTTP methods: GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE
   - Cache policy: CachingDisabled
   - Origin request policy: AllViewerExceptHostHeader

### 3.3 Associate Lambda Function
1. Edit distribution behavior
2. Under Lambda Function Associations:
   - Event Type: Origin Request
   - Lambda Function ARN: [your-function-version-arn]
   - Include body: Yes

## Step 4: Test the Integration

### 4.1 Using curl
```bash
# Test GET request
curl -X GET https://[your-distribution-domain]/test

# Test POST request
curl -X POST https://[your-distribution-domain]/test \
  -H "Content-Type: application/json" \
  -d '{"message":"Hello CloudFront!"}'
```

### 4.2 Using Python
```python
import requests
import json

base_url = 'https://[your-distribution-domain]'
data = {'message': 'Hello CloudFront!'}

# GET request
response = requests.get(f'{base_url}/test')
print(json.dumps(response.json(), indent=2))

# POST request
response = requests.post(f'{base_url}/test', json=data)
print(json.dumps(response.json(), indent=2))
```

## Step 5: Implement Security

### 5.1 Configure HTTPS
1. Use custom SSL certificate:
   - Request certificate in ACM (us-east-1)
   - Associate with distribution
   - Configure SSL/TLS policy

### 5.2 Implement Access Controls
1. Configure security headers
2. Set up WAF rules
3. Implement geo-restrictions if needed
4. Configure signed URLs or cookies

## Best Practices

1. Security:
   - Always use HTTPS
   - Implement proper WAF rules
   - Use appropriate security headers
   - Configure access logging

2. Performance:
   - Optimize cache settings
   - Use appropriate price class
   - Monitor edge performance
   - Configure compression

3. Monitoring:
   - Enable CloudFront logging
   - Set up CloudWatch alarms
   - Monitor Lambda metrics
   - Track error rates

4. Cost Optimization:
   - Choose appropriate price class
   - Optimize cache hit ratio
   - Monitor data transfer costs
   - Use appropriate Lambda memory

5. Edge Function Limitations:
   - Function size ≤ 1MB
   - Execution time ≤ 5s
   - Memory ≤ 10GB
   - No environment variables

## Troubleshooting

1. Common Issues:
   - Cache behavior issues
   - SSL/TLS certificate problems
   - Function timeout errors
   - Invalid response format

2. Debugging Tips:
   - Check CloudFront logs
   - Monitor Lambda@Edge metrics
   - Verify function permissions
   - Test function locally first

## Conclusion
You now have a Lambda function accessible through CloudFront. This setup provides a globally distributed, secure, and scalable way to expose your Lambda functions with edge computing capabilities and content delivery optimization.