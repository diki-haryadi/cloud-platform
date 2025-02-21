# Jenkins Integration with Amazon EKS: Step-by-Step Guide

## Overview

This guide provides two approaches to integrate Jenkins with Amazon EKS:
1. AWS CLI Approach - Using AWS credentials and CLI tools
2. Direct Kubernetes URL Approach - Using Kubernetes service account and direct cluster access

## Prerequisites

### Common Requirements
1. Running Jenkins server
2. Amazon EKS cluster
3. kubectl installed

### AWS CLI Approach Requirements
1. AWS CLI installed
2. AWS IAM credentials with appropriate permissions

### Direct Kubernetes Approach Requirements
1. Kubernetes service account with appropriate RBAC permissions
2. Cluster API server URL

## Approach 1: AWS CLI Integration

### Step 1: Install Required Jenkins Plugins
1. Install the following Jenkins plugins:
   - AWS Credentials Plugin
   - Pipeline AWS Steps Plugin
   - Kubernetes CLI Plugin
   Navigate to: Jenkins Dashboard > Manage Jenkins > Manage Plugins > Available

### Step 2: Configure AWS Credentials in Jenkins
1. Go to Jenkins Dashboard > Manage Jenkins > Manage Credentials
2. Click on 'Credentials' under System
3. Click on 'Global credentials (unrestricted)'
4. Click 'Add Credentials'
5. Configure AWS credentials:
   - Kind: AWS Credentials
   - ID: aws-eks-credentials
   - Description: AWS Credentials for EKS
   - Access Key ID: Your AWS Access Key
   - Secret Access Key: Your AWS Secret Key

### Step 3: Configure IAM Roles and Permissions
1. Create an IAM role with the following permissions:
   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": [
                   "eks:DescribeCluster",
                   "eks:ListClusters",
                   "eks:AccessKubernetesApi"
               ],
               "Resource": "*"
           }
       ]
   }
   ```

### Step 4: Configure kubectl and AWS CLI
1. Install kubectl on Jenkins server:
   ```bash
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
   chmod +x kubectl
   sudo mv kubectl /usr/local/bin/
   ```

2. Create Jenkins Pipeline (AWS CLI Approach):
   ```groovy
   pipeline {
       agent any
       
       environment {
           CLUSTER_NAME = 'my-eks-cluster'
           REGION = 'ap-southeast-2'
       }
       
       stages {
           stage('Install AWS CLI') {
               steps {
                   sh '''
                       curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
                       unzip awscliv2.zip
                       sudo ./aws/install
                       aws --version
                   '''
               }
           }
           
           stage('Configure kubectl') {
               steps {
                   withAWS(credentials: 'aws-eks-credentials', region: REGION) {
                       sh '''
                           aws eks update-kubeconfig --name ${CLUSTER_NAME} --region ${REGION}
                           kubectl get nodes
                       '''
                   }
               }
           }
           
           stage('Deploy to EKS') {
               steps {
                   withAWS(credentials: 'aws-eks-credentials', region: REGION) {
                       sh '''
                           kubectl apply -f k8s/deployment.yaml
                           kubectl apply -f k8s/service.yaml
                       '''
                   }
               }
           }
       }
   }
   ```
## Build User Vars Plugin Configuration

### Step 1: Install Build User Vars Plugin
1. Navigate to Jenkins Dashboard > Manage Jenkins > Manage Plugins
2. Go to the 'Available' tab
3. Search for "Build User Vars Plugin"
4. Select the plugin and click "Install without restart"

### Step 2: Configure Pipeline with Build User Vars
1. Modify your Jenkins pipeline to include the build user variables:
   ```groovy
   pipeline {
       agent any
       
       environment {
           CLUSTER_NAME = 'my-eks-cluster'
           REGION = 'ap-southeast-2'
       }
       
       stages {
           stage('Get User Info') {
               steps {
                   wrap([$class: 'BuildUser']) {
                       script {
                           // Access build user variables
                           echo "Build User ID: ${env.BUILD_USER_ID}"
                           echo "Build User: ${env.BUILD_USER}"
                           echo "Build User Email: ${env.BUILD_USER_EMAIL}"
                       }
                   }
               }
           }
           
           // Your existing stages...
       }
   }
   ```

### Step 3: Available Variables
The plugin provides the following variables:
- BUILD_USER: Full name of the user who started the build
- BUILD_USER_ID: User ID of the user who started the build
- BUILD_USER_EMAIL: Email address of the user who started the build
- BUILD_USER_FIRST_NAME: First name of the user
- BUILD_USER_LAST_NAME: Last name of the user

### Step 4: Usage Examples
1. Logging user actions:
   ```groovy
   wrap([$class: 'BuildUser']) {
       sh "echo 'Deployment initiated by: ${env.BUILD_USER}' >> deployment.log"
   }
   ```

2. Conditional execution based on user:
   ```groovy
   wrap([$class: 'BuildUser']) {
       script {
           if (env.BUILD_USER_ID == 'admin') {
               // Execute admin-only tasks
           }
       }
   }
   ```

## Approach 2: Direct Kubernetes URL Integration

### Step 1: Install Required Jenkins Plugins
1. Install the following Jenkins plugins:
   - Kubernetes Plugin
   - Kubernetes Credentials Plugin
   Navigate to: Jenkins Dashboard > Manage Jenkins > Manage Plugins > Available

### Step 2: Create Kubernetes Service Account
1. Connect to your EKS cluster using kubectl:
   ```bash
   aws eks update-kubeconfig --name your-cluster-name --region your-region
   ```

2. Create the jenkins namespace:
   ```bash
   kubectl create namespace jenkins
   ```

3. Create a service account in your EKS cluster by saving the following YAML to `jenkins-sa.yaml` and applying it:
   ```bash
   kubectl apply -f jenkins-sa.yaml
   ```

   Content of `jenkins-sa.yaml`:
   ```yaml
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: jenkins
     namespace: jenkins
   ```

4. Create RBAC roles by saving the following YAML to `jenkins-rbac.yaml` and applying it:
   ```bash
   kubectl apply -f jenkins-rbac.yaml
   ```
   Content of `jenkins-rbac.yaml`:
   ```yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: ClusterRole
   metadata:
     name: jenkins-role
   rules:
   - apiGroups: ["", "apps", "extensions"]
     resources: ["*"]
     verbs: ["*"]
   - apiGroups: [""]
     resources: ["pods", "pods/exec", "pods/log"]
     verbs: ["*"]
   - apiGroups: [""]
     resources: ["secrets", "configmaps"]
     verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
   - apiGroups: [""]
     resources: ["namespaces"]
     verbs: ["get", "list"]
   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: ClusterRoleBinding
   metadata:
     name: jenkins-role-binding
   subjects:
   - kind: ServiceAccount
     name: jenkins
     namespace: jenkins
   roleRef:
     kind: ClusterRole
     name: jenkins-role
     apiGroup: rbac.authorization.k8s.io
   ```
5. Verify the RBAC configuration:
   ```bash
   # Verify ClusterRole creation
   kubectl get clusterrole jenkins-role
   kubectl describe clusterrole jenkins-role

   # Verify ClusterRoleBinding creation
   kubectl get clusterrolebinding jenkins-role-binding
   kubectl describe clusterrolebinding jenkins-role-binding

   # Verify permissions for the Jenkins service account
   kubectl auth can-i --list --as=system:serviceaccount:jenkins:jenkins
   ```

6. Create a service account token by saving the following YAML to `jenkins-token.yaml` and applying it:
   ```bash
   kubectl apply -f jenkins-token.yaml
   ```

   Content of `jenkins-token.yaml`:
   ```yaml
   apiVersion: v1
   kind: Secret
   metadata:
     name: jenkins-token
     namespace: jenkins
     annotations:
       kubernetes.io/service-account.name: jenkins
   type: kubernetes.io/service-account-token
   ```

7. Retrieve and verify the token:
   ```bash
   kubectl get secret jenkins-token -n jenkins -o jsonpath='{.data.token}' | base64 --decode
   ```

### Step 3: Configure Jenkins Kubernetes Cloud
1. Go to Jenkins Dashboard > Manage Jenkins > Manage Nodes and Clouds
2. Click on 'Configure Clouds'
3. Click 'Add a new cloud' and select 'Kubernetes'
4. Configure the Kubernetes cloud:
   - Name: kubernetes
   - Kubernetes URL: Your cluster API server URL
   - Kubernetes Namespace: jenkins
   - Credentials: Click 'Add' to add Kubernetes credentials
     - Kind: Kubernetes Service Account
     - ID: kubernetes-sa
     - Description: Kubernetes Service Account
   - Jenkins URL: http://jenkins-service.jenkins.svc.cluster.local:8080
   - Jenkins tunnel: jenkins-agent.jenkins.svc.cluster.local:50000

   Note: Make sure to configure the following:
   1. Network Security:
      - Allow inbound traffic on port 50000 for JNLP connections
      - Configure security groups to allow traffic between Jenkins and EKS
   2. Jenkins Controller Settings:
      - Go to Manage Jenkins > Configure Global Security
      - Under 'Agents' section, enable 'TCP port for inbound agents'
      - Set fixed port to 50000
   3. Verify Connectivity:
      ```bash
      # Test JNLP port connectivity
      nc -zv jenkins-agent.jenkins.svc.cluster.local 50000
      
      # Check service endpoints
      kubectl get endpoints -n jenkins
      ```

5. Create Jenkins Pipeline (Direct Kubernetes Approach):
   ```groovy
   pipeline {
       agent {
           kubernetes {
               yaml '''
               apiVersion: v1
               kind: Pod
               metadata:
                 labels:
                   app: jenkins-agent
               spec:
                 serviceAccountName: jenkins
                 containers:
                 - name: kubectl
                   image: bitnami/kubectl:latest
                   command:
                   - cat
                   tty: true
               '''
           }
       }
       
       stages {
           stage('Deploy to EKS') {
               steps {
                   container('kubectl') {
                       sh '''
                           kubectl apply -f k8s/deployment.yaml
                           kubectl apply -f k8s/service.yaml
                       '''
                   }
               }
           }
       }
   }
   ```

## Troubleshooting

### AWS CLI Approach Issues
1. AWS Authentication:
   ```bash
   aws sts get-caller-identity  # Verify AWS credentials
   aws eks list-clusters --region ${REGION}  # Check cluster access
   ```

2. Kubeconfig:
   ```bash
   kubectl config view --minify  # View current context
   kubectl config get-contexts   # List available contexts
   ```

### Direct Kubernetes Approach Issues
1. Service Account:
   ```bash
   kubectl get serviceaccount jenkins -n jenkins
   kubectl describe serviceaccount jenkins -n jenkins
   ```

2. RBAC Permissions:
   ```bash
   kubectl auth can-i --list --as=system:serviceaccount:jenkins:jenkins
   ```

6. Create a service account token by saving the following YAML to `jenkins-token.yaml` and applying it:
   ```bash
   kubectl apply -f jenkins-token.yaml
   ```

   Content of `jenkins-token.yaml`:
   ```yaml
   apiVersion: v1
   kind: Secret
   metadata:
     name: jenkins-token
     namespace: jenkins
     annotations:
       kubernetes.io/service-account.name: jenkins
   type: kubernetes.io/service-account-token
   ```

7. Retrieve and verify the token:
   ```bash
   kubectl get secret jenkins-token -n jenkins -o jsonpath='{.data.token}' | base64 --decode
   ```

## Best Practices

### Security
1. AWS CLI Approach:
   - Use IAM roles with minimum required permissions
   - Rotate AWS credentials regularly
   - Store AWS credentials in Jenkins credentials store

2. Direct Kubernetes Approach:
   - Use namespaced RBAC permissions when possible
   - Regularly rotate service account tokens
   - Implement network policies to restrict pod communication

### Pipeline
1. Use declarative pipeline syntax
2. Implement proper error handling
3. Add timeout and retry mechanisms
4. Version control your pipeline code
5. Use shared libraries for common functions"}},"next_plan_guideline":"This is the final plan. The file has been updated with both integration approaches clearly documented.