# StreamingApp — Kubernetes Orchestration and Scaling

## 1. Project Overview

StreamingApp is a multi-service MERN-based video streaming application deployed using Docker, Amazon ECR, Amazon EKS, Kubernetes, Helm, Amazon S3, AWS Load Balancer Controller, and Amazon CloudWatch.

The application consists of:

* Frontend
* Authentication service
* Streaming service
* Admin service
* Chat service
* MongoDB
* Amazon S3 for video and thumbnail storage
* AWS Application Load Balancer for external access
* Amazon ECR for container images
* Amazon EKS for Kubernetes orchestration
* Helm for application deployment and configuration
* Horizontal Pod Autoscaler for scaling
* CloudWatch Container Insights for monitoring and logging
* Jenkins for CI/CD
* Slack for ChatOps notifications

---

# 2. Overall System Architecture

```text
                              Internet
                                  |
                                  v
                         +----------------+
                         |   Cloudflare   |
                         |  imreading.xyz |
                         +-------+--------+
                                 |
                                 v
                    +-------------------------+
                    | AWS Application Load    |
                    | Balancer (ALB)          |
                    +-----------+-------------+
                                |                                
                                v
                    +--------------------------+
                    | Kubernetes Ingress       |
                    | streamingapp-ingress     |
                    | Ingress Class: ALB       |   
                    +---------------+----------+
                                |
                                v
                    +-------------------------+
                    |        AWS EKS          |
                    |  streaming-app-cluster  |
                    |                         |
                    |  Namespace: streamingapp|
                    +-----------+-------------+
                                |
          +---------------------+----------------------+
          |            |             |        |        |
          v            v             v        v        v
      Frontend       Auth        Streaming   Admin    Chat
      Deployment   Deployment   Deployment Deployment Deployment
          |            |             |        |        |
          v            v             v        v        v
       Service      Service       Service   Service   Service
                                       |
                                       |
                              +--------+--------+
                              |                 |
                              v                 v
                         MongoDB            Amazon S3
                         StatefulSet       Media Storage
                         mongo-0           Videos/Thumbnails
                              |
                              v
                           PVC
                          5 GiB gp2

                    +-----------------------+
                    |   CloudWatch          |
                    | Container Insights    |
                    | Metrics + Logs         |
                    +-----------------------+

                    +-----------------------+
                    | Jenkins               |
                    | Build → ECR Push      |
                    +----------+------------+
                               |
                               v
                    +-----------------------+
                    | Slack ChatOps         |
                    | #streamingapp-devops  |
                    +-----------------------+

```

---

# 3. Application Components

| Component | Kubernetes Resource  |   Port | Purpose                                    |
| --------- | -------------------- | -----: | ------------------------------------------ |
| Frontend  | Deployment + Service |     80 | React user interface                       |
| Auth      | Deployment + Service |   3001 | Registration, login and JWT authentication |
| Streaming | Deployment + Service |   3002 | Video catalogue and streaming              |
| Admin     | Deployment + Service |   3003 | Video and thumbnail upload                 |
| Chat      | Deployment + Service |   3004 | Live chat/WebSocket functionality          |
| MongoDB   | StatefulSet + PVC    |  27017 | Application database                       |
| Ingress   | AWS ALB Ingress      | 80/443 | External application access                |

---

# 4. Containerization

Each application component has its own Dockerfile.

```text
backend/
├── authService/
│   └── Dockerfile
├── streamingService/
│   └── Dockerfile
├── adminService/
│   └── Dockerfile
└── chatService/
    └── Dockerfile

frontend/
└── Dockerfile
```

The five Docker images are:

```text
streaming-auth
streaming-stream
streaming-admin
streaming-chat
streaming-frontend
```

---

# 5. Amazon ECR

Five ECR repositories were created in the `ap-south-1` region:

```text
streaming-auth
streaming-stream
streaming-admin
streaming-chat
streaming-frontend
```

ECR registry:

```text
218014315198.dkr.ecr.ap-south-1.amazonaws.com
```

Example image:

```text
218014315198.dkr.ecr.ap-south-1.amazonaws.com/streaming-auth:1.0.1
```

Images were built locally and pushed to Amazon ECR.

---

# 6. Jenkins CI/CD Pipeline

Jenkins is used to automate the Docker image build and ECR push process.

The pipeline performs:

```text
Git Checkout
     |
     v
Build Docker Images
     |
     v
Login to Amazon ECR
     |
     v
Tag Images
     |
     v
Push Images to ECR
     |
     v
Slack Notification
```

The Jenkins pipeline uses the AWS credential configured specifically for the StreamingApp project.

The AWS region is:

```text
ap-south-1
```

The pipeline builds all five application images.

---

# 7. Jenkins Pipeline Configuration

The main pipeline file is:

```text
Jenkinsfile
```

Important pipeline stages:

```groovy
stage('Checkout')
stage('Build Docker Images')
stage('Login to ECR')
stage('Push Images to ECR')
```

The pipeline uses:

```groovy
withAWS(
    credentials: 'Credentials_ID',
    region: 'ap-south-1'
)
```

The ECR login is performed using:

```bash
aws ecr get-login-password --region "${AWS_REGION}" |
    docker login \
        --username AWS \
        --password-stdin "${ECR_REGISTRY}"
```

---

# 8. Slack ChatOps Integration

Slack was integrated with Jenkins using an Incoming Webhook.

Slack channel:

```text
#streamingapp-devops
```

The webhook URL is stored securely in Jenkins Credentials.

Credential ID:

```text
streamingapp-slack-webhook
```

The webhook is accessed through:

```groovy
withCredentials([
    string(
        credentialsId: 'streamingapp-slack-webhook',
        variable: 'SLACK_WEBHOOK_URL'
    )
])
```

## Successful Pipeline Notification

A successful pipeline sends:

```text
🚀 StreamingApp CI/CD SUCCESS
```

## Failed Pipeline Notification

A failed pipeline sends:

```text
❌ StreamingApp CI/CD FAILED - Check Jenkins Console Output
```

Both success and failure notifications were tested successfully.

---

# 9. Amazon EKS Cluster

The application was deployed to:

```text
EKS Cluster:
streaming-app-cluster
```

AWS Region:

```text
ap-south-1
```

The EKS cluster uses managed worker nodes.

MongoDB storage requires the persistent volume to be scheduled in the Availability Zone containing its EBS volume.

The final cluster has worker nodes available across multiple Availability Zones.

---

# 10. Kubernetes Namespace

All application resources are deployed into:

```text
streamingapp
```

Example:

```bash
kubectl get pods -n streamingapp
```

---

# 11. Helm Deployment

The Kubernetes application is packaged as a Helm chart:

```text
streamingapp/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── admin-deployment.yaml
│   ├── admin-service.yaml
│   ├── auth-deployment.yaml
│   ├── auth-service.yaml
│   ├── chat-deployment.yaml
│   ├── chat-service.yaml
│   ├── configmap.yaml
│   ├── frontend-deployment.yaml
│   ├── frontend-service.yaml
│   ├── streaming-deployment.yaml
│   ├── streaming-service.yaml
│   ├── mongo-statefulset.yaml
│   ├── mongo-service.yaml
│   └── ...
└── ...
```

The Helm release is:

```text
streamingapp
```

Deployment command:

```bash
helm upgrade --install streamingapp ./streamingapp \
  -n streamingapp \
  --create-namespace
```

Helm upgrade example:

```bash
helm upgrade streamingapp ./streamingapp \
  -n streamingapp \
  --force-conflicts
```

---

## 8. Ingress and External Access

The application is exposed to users through a Kubernetes Ingress resource using the **AWS Load Balancer Controller**. The Ingress creates an AWS Application Load Balancer (ALB) that routes external HTTP traffic to the appropriate Kubernetes services.

### Ingress Configuration

The application uses the following hostname:

```text
imreading.xyz
```

Ingress resource:

```text
streamingapp-ingress
```

Ingress class:

```text
alb
```

The AWS Load Balancer Controller watches the Kubernetes Ingress resource and provisions/configures the AWS Application Load Balancer automatically.

The traffic flow is:

```text
User Browser
     |
     | HTTP
     v
imreading.xyz
     |
     v
Cloudflare DNS
     |
     v
AWS Application Load Balancer
     |
     | Kubernetes Ingress rules
     |
     +--------------------+
     |                    |
     v                    v
Frontend Service       API Services
     |                    |
     v                    +--> auth-service
Frontend Pods             +--> streaming-service
                          +--> admin-service
                          +--> chat-service
```

### Ingress Host-Based Routing

The Ingress uses the host `imreading.xyz` to route requests into the StreamingApp Kubernetes services.

API requests are exposed through the `/api` path and are routed to the corresponding backend services. The frontend application is served for normal web requests.

Examples:

```text
http://imreading.xyz/
    -> Frontend Service

http://imreading.xyz/api/login
    -> Auth Service

http://imreading.xyz/api/streaming/videos
    -> Streaming Service

http://imreading.xyz/api/admin/videos
    -> Admin Service

http://imreading.xyz/api/chat/...
    -> Chat Service
```

This allows the application to use a single public hostname instead of exposing each backend service separately.

### AWS Load Balancer Controller

The AWS Load Balancer Controller was installed in the `kube-system` namespace.

The controller creates and manages the AWS Application Load Balancer based on the Kubernetes Ingress resource.

Controller installation was performed using Helm:

```bash
helm upgrade --install aws-load-balancer-controller \
  eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=streaming-app-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=ap-south-1 \
  --set vpcId=$(aws eks describe-cluster \
    --name streaming-app-cluster \
    --region ap-south-1 \
    --query 'cluster.resourcesVpcConfig.vpcId' \
    --output text)
```

The controller pods were verified as running:

```bash
kubectl get pods -n kube-system \
  -l app.kubernetes.io/name=aws-load-balancer-controller
```

### Ingress Verification

The Ingress can be checked using:

```bash
kubectl get ingress -n streamingapp
```

Detailed configuration:

```bash
kubectl describe ingress streamingapp-ingress -n streamingapp
```

Expected hostname:

```text
imreading.xyz
```

The ALB DNS name provisioned for the application was:

```text
k8s-streamin-streamin-671c5478c2-188693131.ap-south-1.elb.amazonaws.com
```

### DNS Configuration

Cloudflare DNS is used to point the public domain to the AWS Application Load Balancer.

```text
imreading.xyz
      |
      | CNAME
      v
AWS Application Load Balancer
```

The DNS record is configured as DNS-only so that the hostname resolves to the AWS ALB.

### Application Verification

The application was verified through the public domain:

```bash
curl -I http://imreading.xyz
```

The application returned:

```text
HTTP/1.1 200 OK
```

The API endpoints were also tested through the Ingress:

```bash
curl -i http://imreading.xyz/api/verify
```

The endpoint returned an authentication response, confirming that the request reached the backend service through the Ingress.

The streaming API was also tested:

```bash
curl -i http://imreading.xyz/api/streaming/videos
```

The API returned successfully through the public hostname.

Therefore, the Ingress provides the external entry point for the StreamingApp and connects the public domain to the frontend and backend Kubernetes services.

---

# 12. Application Configuration

The application configuration is maintained through Helm `values.yaml` and Kubernetes ConfigMaps/Secrets.

The application services use the following ports:

```text
Auth       3001
Streaming  3002
Admin      3003
Chat       3004
Frontend   80
MongoDB    27017
```

The external application domain is:

```text
http://imreading.xyz
```

The frontend uses API paths such as:

```text
/api
/api/admin
/api/chat
```

---

# 13. Horizontal Pod Autoscaling

HPA is configured for the application services.

The normal baseline is one replica per application service.

Example:

```yaml
replicas: 1

autoscaling:
  enabled: true
  minReplicas: 1
  maxReplicas: 4
  targetCPUUtilizationPercentage: 70
```

The Streaming service was tested by scaling from:

```text
1 replica → 4 replicas
```

After the scaling test, the configuration was restored to:

```text
minReplicas: 1
maxReplicas: 4
```

With low CPU utilization, the Streaming service subsequently scaled back to one replica.

---

# 14. Scaling Validation

The Streaming deployment was manually scaled to four replicas:

```bash
kubectl scale deployment/streaming \
  -n streamingapp \
  --replicas=4
```

The resulting Deployment state was:

```text
streaming   4/4   4   4
```

Four Streaming pods were running and ready.

The HPA was then restored to its normal configuration:

```text
MINPODS = 1
MAXPODS = 4
```

The final Streaming Deployment returned to:

```text
streaming   1/1   1   1
```

---

# 15. Zero-Downtime Rolling Updates

All five application Deployments use:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1
```

Verified configuration:

```text
admin       RollingUpdate   0   1
auth        RollingUpdate   0   1
chat        RollingUpdate   0   1
frontend    RollingUpdate   0   1
streaming   RollingUpdate   0   1
```

This configuration means Kubernetes performs updates gradually while maintaining the required availability policy.

---

# 16. Auth Version Update

The Auth service was updated from:

```text
streaming-auth:v2
```

to:

```text
streaming-auth:1.0.1
```

The update was performed through Helm:

```bash
helm upgrade streamingapp ./streamingapp \
  -n streamingapp \
  --set services.auth.tag=1.0.1 \
  --force-conflicts
```

The resulting image was verified with:

```bash
kubectl get deployment auth -n streamingapp \
  -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'
```

Result:

```text
218014315198.dkr.ecr.ap-south-1.amazonaws.com/streaming-auth:1.0.1
```

The rollout completed successfully:

```text
deployment "auth" successfully rolled out
```

---

# 17. MongoDB StatefulSet and Persistent Storage

MongoDB is deployed using a StatefulSet:

```text
mongo-0
```

Persistent storage is provided using a PersistentVolumeClaim.

PVC:

```text
mongo-data-mongo-0
```

Storage:

```text
5 GiB
```

Storage class:

```text
gp2
```

MongoDB data is therefore stored on persistent EBS-backed Kubernetes storage instead of ephemeral container storage.

---

# 18. Amazon S3 Media Storage

Amazon S3 is used for:

* Video files
* Thumbnail files

Bucket:

```text
streamingapp-media-218014315198
```

The Admin service uses an IAM role through EKS service account integration.

Service account:

```text
streamingapp-s3
```

IAM role:

```text
StreamingAppS3Role
```

The role provides access to:

```text
s3:GetObject
s3:PutObject
s3:DeleteObject
s3:ListBucket
```

Access is restricted to the StreamingApp media bucket.

---

# 19. AWS Load Balancer Controller

The AWS Load Balancer Controller was installed in the EKS cluster.

It creates and manages the AWS Application Load Balancer from the Kubernetes Ingress resource.

The application Ingress uses:

```text
streamingapp-ingress
```

External hostname:

```text
imreading.xyz
```

The controller provides routing from the ALB to the Kubernetes Services.

---

# 20. Application Smoke Tests

The following application tests were performed.

## 20.1 Pod Health

```bash
kubectl get pods -n streamingapp
```

All application Pods were verified as:

```text
1/1 Running
```

including:

```text
admin
auth
chat
frontend
streaming
mongo-0
```

---

## 20.2 Registration and Login

The authentication API was tested through the application domain.

Login returned:

```text
HTTP 200
```

and a JWT was generated successfully.

This confirmed that the authentication service was accessible through the Ingress.

---

## 20.3 Admin Video Upload

A test video and thumbnail were uploaded through the Admin dashboard.

The upload was stored in Amazon S3 and the corresponding video metadata was stored in MongoDB.

The resulting video status was:

```text
ready
```

---

## 20.4 Video Playback

Video streaming was tested using HTTP Range requests.

Example:

```bash
curl -i \
  -H "Range: bytes=0-1023" \
  http://imreading.xyz/api/streaming/stream/<VIDEO_ID> \
  -o /tmp/video-test
```

The streaming service successfully returned video data.

---

## 20.5 Live Chat

Live chat was tested from the video playback page using two browser tabs.

Messages sent from one browser tab were received by the other tab.

This validated the live chat functionality.

---

## 20.6 Kubernetes Self-Healing

A Streaming pod was deliberately deleted:

```bash
kubectl delete pod <STREAMING_POD_NAME> \
  -n streamingapp
```

Kubernetes automatically created a replacement Pod.

The replacement Pod became:

```text
1/1 Running
```

This demonstrated Kubernetes self-healing.

---

# 21. CloudWatch Monitoring

Amazon CloudWatch Container Insights was enabled for the EKS cluster.

The CloudWatch Observability add-on collects:

* Cluster metrics
* Node metrics
* Pod metrics
* Container metrics
* Application logs
* Host logs
* Kubernetes dataplane logs

CloudWatch log groups include:

```text
/aws/containerinsights/streaming-app-cluster/application
/aws/containerinsights/streaming-app-cluster/dataplane
/aws/containerinsights/streaming-app-cluster/host
/aws/containerinsights/streaming-app-cluster/performance
```

---

# 22. CloudWatch Alarms

CPU utilization alarms were configured for EKS worker nodes.

Alarms:

```text
StreamingApp-EKS-Node1-HighCPU
StreamingApp-EKS-Node2-HighCPU
```

Threshold:

```text
70%
```

Evaluation period:

```text
300 seconds
```

The alarms were verified in the `OK` state during validation.

---

# 23. Useful Kubernetes Commands

### View all Pods

```bash
kubectl get pods -n streamingapp
```

### View Deployments

```bash
kubectl get deploy -n streamingapp
```

### View Services

```bash
kubectl get svc -n streamingapp
```

### View HPA

```bash
kubectl get hpa -n streamingapp
```

### View Ingress

```bash
kubectl get ingress -n streamingapp
```

### View rollout status

```bash
kubectl rollout status deployment/auth -n streamingapp
```

### View Deployment strategy

```bash
kubectl get deployments -n streamingapp \
  -o custom-columns='NAME:.metadata.name,STRATEGY:.spec.strategy.type,MAX_UNAVAILABLE:.spec.strategy.rollingUpdate.maxUnavailable,MAX_SURGE:.spec.strategy.rollingUpdate.maxSurge'
```

### View Helm releases

```bash
helm list -n streamingapp
```

### Upgrade Helm release

```bash
helm upgrade streamingapp ./streamingapp \
  -n streamingapp \
  --force-conflicts
```

---

# 24. Final Deployment State

The final application configuration uses one baseline replica for each application service.

```text
admin       1/1
auth        1/1
chat        1/1
frontend    1/1
streaming   1/1
```

MongoDB:

```text
mongo-0     1/1 Running
```

Streaming HPA:

```text
Min replicas: 1
Max replicas: 4
CPU target:   70%
```

All application Deployments use:

```text
RollingUpdate
maxUnavailable: 0
maxSurge: 1
```

---

# 25. Validation Summary

| Requirement                | Status     |
| -------------------------- | ---------- |
| Docker containerization    | Completed  |
| Five ECR repositories      | Completed  |
| Images pushed to ECR       | Completed  |
| Jenkins CI/CD pipeline     | Completed  |
| Slack success notification | Tested     |
| Slack failure notification | Tested     |
| EKS cluster deployment     | Completed  |
| Helm deployment            | Completed  |
| HPA configuration          | Completed  |
| Streaming scale 1 → 4      | Tested     |
| Streaming scale 4 → 1      | Tested     |
| RollingUpdate strategy     | Verified   |
| maxUnavailable = 0         | Verified   |
| maxSurge = 1               | Verified   |
| Auth updated to 1.0.1      | Completed  |
| Auth rollout               | Successful |
| MongoDB StatefulSet        | Completed  |
| Persistent storage         | Completed  |
| S3 media upload            | Tested     |
| Video playback             | Tested     |
| Live chat                  | Tested     |
| Pod self-healing           | Tested     |
| CloudWatch monitoring      | Completed  |
| CloudWatch logging         | Completed  |
| CloudWatch alarms          | Completed  |

---

# 26. Project Conclusion

StreamingApp was successfully containerized and deployed as a multi-service application on Amazon EKS.

Docker images were stored in Amazon ECR, Jenkins automated the image build and push process, Helm managed Kubernetes application configuration, HPA provided horizontal scaling, and RollingUpdate provided controlled application updates.

Amazon S3 was used for persistent media storage, MongoDB was deployed using a StatefulSet with persistent storage, and the AWS Load Balancer Controller provided external application access through an Application Load Balancer.

CloudWatch Container Insights provided monitoring and centralized logging, while Slack ChatOps notifications provided Jenkins pipeline success and failure visibility.

The application was validated through authentication, media upload, video playback, live chat, scaling, rolling updates, and Kubernetes self-healing tests.
