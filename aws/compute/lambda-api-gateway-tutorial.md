# Tutorial: Accessing AWS Lambda through API Gateway

## Overview
This tutorial demonstrates how to expose AWS Lambda functions to the internet using Amazon API Gateway, creating secure and scalable HTTP/REST APIs.

## Prerequisites
- AWS Account with appropriate permissions
- AWS CLI installed and configured
- Basic understanding of REST APIs
- Basic Python programming knowledge

## Step 1: Create a Lambda Function

### 1.1 Create Basic Lambda Function
1. Open AWS Management Console
2. Navigate to Lambda service
3. Click "Create function"
4. Configure settings:
   - Name: `api-demo-function`
   - Runtime: Python 3.9
   - Architecture: x86_64
   - Permissions: Create new role with basic Lambda permissions

### 1.2 Implement Function Code
```python
import json

def lambda_handler(event, context):
    try:
        # Get query parameters if any
        query_params = event.get('queryStringParameters', {})
        
        # Get path parameters if any
        path_params = event.get('pathParameters', {})
        
        # Get body if any
        body = json.loads(event.get('body', '{}'))
        
        # Create response
        response = {
            'message': 'Hello from Lambda!',
            'query_params': query_params,
            'path_params': path_params,
            'body': body
        }
        
        return {
            'statusCode': 200,
            'headers': {
                'Content-Type': 'application/json',
                'Access-Control-Allow-Origin': '*'  # Enable CORS
            },
            'body': json.dumps(response)
        }
    except Exception as e:
        return {
            'statusCode': 500,
            'body': json.dumps({'error': str(e)})
        }
```

## Step 2: Create and Configure API Gateway

### 2.1 Create REST API
1. Go to API Gateway Console
2. Click "Create API"
3. Choose "REST API" (not private)
4. Configure settings:
   - API name: `lambda-api-demo`
   - Description: API Gateway for Lambda function
   - Endpoint Type: Regional

### 2.2 Create Resource and Method
1. In the Resources panel:
   - Click "Actions" → "Create Resource"
   - Resource Name: `demo`
   - Resource Path: `/demo`
   - Enable CORS: Yes

2. Create Method:
   - Select the resource
   - Click "Actions" → "Create Method"
   - Choose HTTP method: POST
   - Integration type: Lambda Function
   - Lambda Function: `api-demo-function`

### 2.3 Configure Method Settings
1. Method Request:
   - Authorization: NONE (for demo purposes)
   - API Key Required: No
   - Request Validator: Validate body

2. Integration Request:
   - Keep default settings
   - Optionally add mapping templates if needed

### 2.4 Deploy API
1. Click "Actions" → "Deploy API"
2. Create new stage:
   - Stage name: `dev`
   - Stage description: Development stage
3. Note the API endpoint URL

## Step 3: Test the Integration

### 3.1 Using curl
```bash
# Test GET request
curl -X POST https://[your-api-id].execute-api.[region].amazonaws.com/dev/demo \
  -H "Content-Type: application/json" \
  -d '{"message":"Hello API Gateway!"}'
```

### 3.2 Using Python
```python
import requests
import json

api_url = 'https://[your-api-id].execute-api.[region].amazonaws.com/dev/demo'
data = {'message': 'Hello API Gateway!'}

response = requests.post(api_url, json=data)
print(json.dumps(response.json(), indent=2))
```

## Step 4: Implement Security

### 4.1 Add API Key
1. In API Gateway Console:
   - Go to API Keys
   - Create API Key
   - Name it `demo-api-key`

2. Create Usage Plan:
   - Name: `demo-usage-plan`
   - Throttling: 1000 requests per second
   - Quota: 1000000 requests per month

3. Associate API Key with Usage Plan

4. Update Method Settings:
   - Edit method request
   - Set "API Key Required" to true

### 4.2 Secure API Calls
```bash
# Test with API Key
curl -X POST https://[your-api-id].execute-api.[region].amazonaws.com/dev/demo \
  -H "Content-Type: application/json" \
  -H "x-api-key: [your-api-key]" \
  -d '{"message":"Hello Secure API Gateway!"}'
```

## Best Practices
1. Security:
   - Always use HTTPS endpoints
   - Implement proper authentication
   - Use API keys for API management
   - Enable AWS WAF for additional security

2. Performance:
   - Enable caching when appropriate
   - Use appropriate timeout values
   - Monitor API metrics

3. Monitoring:
   - Set up CloudWatch alarms
   - Monitor API usage
   - Track error rates

4. Cost Optimization:
   - Use appropriate caching strategies
   - Implement request throttling
   - Monitor API usage costs

## Troubleshooting
1. Common Issues:
   - CORS errors
   - Authentication failures
   - Timeout issues
   - Integration errors

2. Debugging Tips:
   - Check CloudWatch Logs
   - Verify IAM permissions
   - Test API in API Gateway console
   - Validate request/response mapping

## Conclusion
You now have a working Lambda function exposed via API Gateway. This setup provides a secure and scalable way to access your Lambda function through HTTP endpoints.