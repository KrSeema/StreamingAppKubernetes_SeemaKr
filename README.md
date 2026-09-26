# StreamingApp — Container Orchestration and Scaling

A production-style DevOps deployment of a multi-service MERN StreamingApp using Docker, Amazon ECR, Jenkins CI/CD, Kubernetes, Amazon EKS, Helm, AWS Application Load Balancer, Amazon S3, IRSA, CloudWatch monitoring/logging, and Slack ChatOps.

---

## Project Overview

StreamingApp is a multi-service MERN application consisting of:

* React frontend
* Authentication service
* Streaming service
* Admin service
* Chat service
* MongoDB database

The application is containerized using Docker and deployed to Amazon EKS using Helm.

The project demonstrates:

* Containerization
* Docker image management with Amazon ECR
* Jenkins CI/CD
* Kubernetes orchestration
* Helm-based deployment
* Horizontal Pod Autoscaling
* Rolling updates with zero unavailable replicas
* Persistent MongoDB storage
* AWS Application Load Balancer Ingress
* Amazon S3 media storage
* IAM Roles for Service Accounts (IRSA)
* CloudWatch monitoring and logging
* Slack-based ChatOps notifications
* Kubernetes self-healing

---

# Architecture

```text
                                      Internet
                                         |
                                         v
                                +----------------+
                                |  Cloudflare DNS |
                                |  imreading.xyz  |
                                +--------+-------+
                                         |
                                         v
                              +-----------------------+
                              | AWS Application       |
                              | Load Balancer (ALB)   |
                              +-----------+-----------+
                                          |
                                          v
                              +-----------------------+
                              | Kubernetes Ingress    |
                              | streamingapp-ingress   |
                              +-----------+-----------+
                                          |
              +---------------------------+---------------------------+
              |                           |                           |
              v                           v                           v
       Frontend Service            Backend Services              Chat Service
              |                           |                           |
              v                           v                           v
       Frontend Pods          +-----------+-----------+          Chat Pods
                              |           |           |
                              v           v           v
                            Auth      Streaming      Admin
                            Pods        Pods          Pods
                              |           |           |
                              +-----------+-----------+
                                          |
                                          v
                                   +--------------+
                                   |   MongoDB     |
                                   | StatefulSet   |
                                   +------+---------+
                                          |
                                          |
                              Admin Service uses IRSA
                                          |
                                          v
                                   +--------------+
                                   |  Amazon S3    |
                                   | Media Storage |
                                   +--------------+


 CI/CD FLOW

 Developer
     |
     v
   GitHub
     |
     v
  Jenkins
     |
     +--------------------+
     |                    |
     v                    v
 Docker Build          Tests/Checks
     |
     v
 Amazon ECR
     |
     v
 Helm
     |
     v
 Amazon EKS
```

The AWS Load Balancer Controller watches the Kubernetes Ingress resource and manages the AWS Application Load Balancer. It is not part of the normal application request path.

---

# Application Components

| Component | Technology    |   Port | Kubernetes Resource         |
| --------- | ------------- | -----: | --------------------------- |
| Frontend  | React + Nginx |     80 | Deployment + Service        |
| Auth      | Node.js       |   3001 | Deployment + Service        |
| Streaming | Node.js       |   3002 | Deployment + Service        |
| Admin     | Node.js       |   3003 | Deployment + Service        |
| Chat      | Node.js       |   3004 | Deployment + Service        |
| MongoDB   | MongoDB       |  27017 | StatefulSet + Service + PVC |
| Ingress   | AWS ALB       | 80/443 | Kubernetes Ingress          |

---

# Technology Stack

## Application

* React
* Node.js
* Express
* MongoDB
* Socket.IO

## Containers

* Docker
* Docker Compose
* Amazon Elastic Container Registry (ECR)

## CI/CD

* GitHub
* Jenkins
* Slack ChatOps

## Kubernetes

* Kubernetes
* Amazon EKS
* Helm
* Kubernetes Deployments
* Services
* ConfigMaps
* Secrets
* StatefulSets
* PersistentVolumeClaims
* Horizontal Pod Autoscaler
* Readiness probes
* Liveness probes
* Rolling updates

## AWS

* Amazon EKS
* Amazon ECR
* Amazon S3
* Application Load Balancer
* AWS Load Balancer Controller
* IAM
* IAM Roles for Service Accounts
* CloudWatch
* CloudWatch Container Insights

---

# Repository Structure

```text
StreamingApp/
│
├── backend/
│   ├── adminService/
│   │   └── Dockerfile
│   ├── authService/
│   │   └── Dockerfile
│   ├── chatService/
│   │   └── Dockerfile
│   └── streamingService/
│       └── Dockerfile
│
├── frontend/
│   ├── Dockerfile
│   └── ...
│
├── k8s/
│   └── Kubernetes manifests
│
├── streamingapp/
│   ├── Chart.yaml
│   ├── values.yaml
│   ├── templates/
│   └── ...
│
├── Jenkinsfile
├── docker-compose.yml
├── ingress.yaml
├── README.md
└── README_HelmInstall_Ingress.md
```

---

# 1. Local Application Validation

Before deploying to Kubernetes, the application was tested locally using Docker Compose.

Build and start the application:

```bash
docker compose up --build -d
```

Check containers:

```bash
docker compose ps
```

The five application services were successfully containerized:

```text
streaming-auth
streaming-stream
streaming-admin
streaming-chat
streaming-frontend
```

---

# 2. Docker Images

Separate Docker images are created for each application component.

Example:

```bash
docker build \
  -f backend/authService/Dockerfile \
  -t streaming-auth:v1 \
  ./backend
```

Frontend:

```bash
docker build \
  -f frontend/Dockerfile \
  -t streaming-frontend:v1 \
  ./frontend
```

The backend services use Node.js containers, while the frontend uses a multi-stage Docker build with Nginx serving the production React application.

---

# 3. Amazon ECR

An Amazon ECR repository was created for each application component.

```text
streaming-admin
streaming-auth
streaming-chat
streaming-frontend
streaming-stream
```

ECR registry:

```text
218014315198.dkr.ecr.ap-south-1.amazonaws.com
```

Example image:

```text
218014315198.dkr.ecr.ap-south-1.amazonaws.com/streaming-auth:1.0.1
```

The Helm chart references the ECR images through `values.yaml`.

---

# 4. Jenkins CI/CD

Jenkins is used to automate the Docker image build and ECR push process.

The pipeline performs:

```text
GitHub
   |
   v
Jenkins
   |
   v
Checkout Source
   |
   v
Build Docker Images
   |
   v
Authenticate with ECR
   |
   v
Push Images
   |
   v
Slack Notification
```

The Jenkins pipeline includes:

* Source checkout
* Docker image builds
* AWS authentication
* ECR authentication
* ECR image pushes
* Success notification
* Failure notification

The AWS region used by the pipeline is:

```text
ap-south-1
```

---

# 5. Slack ChatOps

Jenkins is integrated with Slack for CI/CD notifications.

Successful pipeline:

```text
🚀 StreamingApp CI/CD SUCCESS
```

Failed pipeline:

```text
❌ StreamingApp CI/CD FAILED - Check Jenkins Console Output
```

The Slack webhook is stored in Jenkins Credentials rather than being hard-coded into the repository.

---

# 6. Amazon EKS

The application was deployed to an Amazon EKS cluster:

```text
Cluster: streaming-app-cluster
Region: ap-south-1
```

The Kubernetes namespace is:

```text
streamingapp
```

The cluster was configured with worker nodes across multiple Availability Zones.

---

# 7. Kubernetes Resources

The deployment contains:

### Deployments

Five application Deployments:

```text
auth
streaming
admin
chat
frontend
```

### Services

Five application Services:

```text
auth
streaming-svc
admin-svc
chat-svc
frontend-svc
```

MongoDB also has a Kubernetes Service.

### Other resources

```text
ConfigMap
Secret
StatefulSet
PersistentVolumeClaim
HorizontalPodAutoscalers
Ingress
ServiceAccount
```

---

# 8. Helm

The complete Kubernetes application is packaged as a Helm chart.

Chart location:

```text
streamingapp/
```

Validate the chart:

```bash
helm lint ./streamingapp
```

Render the manifests:

```bash
helm template streamingapp ./streamingapp \
  -n streamingapp
```

Install:

```bash
helm install streamingapp ./streamingapp \
  -n streamingapp
```

Upgrade:

```bash
helm upgrade streamingapp ./streamingapp \
  -n streamingapp
```

Detailed installation and Ingress instructions are available in:

**[README_HelmInstall_Ingress.md](README_HelmInstall_Ingress.md)**

---

# 9. Application Configuration

Application configuration is managed using Kubernetes ConfigMaps and Secrets.

Examples include:

```text
MongoDB connection
JWT configuration
Application URLs
Frontend API URLs
Streaming URLs
Chat configuration
```

The frontend supports environment variables such as:

```text
REACT_APP_AUTH_API_URL
REACT_APP_STREAMING_API_URL
REACT_APP_STREAMING_PUBLIC_URL
REACT_APP_ADMIN_API_URL
REACT_APP_CHAT_API_URL
REACT_APP_CHAT_SOCKET_URL
```

The production application domain used during validation was:

```text
http://imreading.xyz
```

---

# 10. MongoDB StatefulSet and Persistent Storage

MongoDB is deployed as a Kubernetes StatefulSet.

```text
MongoDB
   |
   v
StatefulSet
   |
   v
PersistentVolumeClaim
   |
   v
Amazon EBS
```

The MongoDB connection used by the application is:

```text
mongodb://mongo:27017/streamingapp
```

The PVC was configured with:

```text
Storage: 5Gi
Access Mode: ReadWriteOnce
Storage Class: gp2
```

The StatefulSet provides stable identity and persistent storage for MongoDB.

---

# 11. Amazon S3 Media Storage

Uploaded video and thumbnail files are stored in Amazon S3.

The architecture separates:

```text
MongoDB
  |
  +-- Video metadata
  +-- Title
  +-- Description
  +-- Genre
  +-- Duration
  +-- S3 object keys

Amazon S3
  |
  +-- Video files
  +-- Thumbnail files
```

This prevents large media files from being stored directly inside MongoDB.

---

# 12. IAM Roles for Service Accounts

The Admin service accesses S3 using an IAM role associated with its Kubernetes ServiceAccount.

```text
Admin Pod
    |
    v
Kubernetes ServiceAccount
    |
    v
IRSA
    |
    v
IAM Role
    |
    v
Amazon S3
```

The Kubernetes ServiceAccount is:

```text
streamingapp-s3
```

The IAM role is:

```text
StreamingAppS3Role
```

The role provides scoped access to the StreamingApp S3 bucket.

---

# 13. AWS Load Balancer Controller and Ingress

The application uses a Kubernetes Ingress with the AWS ALB ingress class.

```yaml
ingressClassName: alb
```

The AWS Load Balancer Controller manages the ALB based on the Kubernetes Ingress definition.

Request flow:

```text
Browser
   |
   v
imreading.xyz
   |
   v
Cloudflare DNS
   |
   v
AWS ALB
   |
   v
Kubernetes Ingress
   |
   +----> Frontend Service
   |
   +----> Auth Service
   |
   +----> Streaming Service
   |
   +----> Admin Service
   |
   +----> Chat Service
```

The application was successfully accessed through:

```text
http://imreading.xyz
```

---

# 14. Ingress Routing

The main routes are:

| Path                 | Destination            |
| -------------------- | ---------------------- |
| `/`                  | Frontend               |
| `/api/login`         | Authentication service |
| `/api/register`      | Authentication service |
| `/api/streaming/...` | Streaming service      |
| `/api/admin/...`     | Admin service          |
| `/api/chat/...`      | Chat service           |

The Ingress configuration is maintained in the Helm chart.

---

# 15. Horizontal Pod Autoscaling

HPA was configured for the application services.

Example configuration:

```yaml
autoscaling:
  enabled: true
  minReplicas: 1
  maxReplicas: 4
  targetCPUUtilizationPercentage: 70
```

The streaming service was configured with:

```text
Minimum replicas: 1
Maximum replicas: 4
CPU target: 70%
```

The scaling test successfully demonstrated that the Streaming deployment could scale to four replicas.

Example:

```bash
kubectl scale deployment/streaming \
  -n streamingapp \
  --replicas=4
```

HPA configuration was then restored to the Helm-defined baseline.

---

# 16. Zero-Downtime Rolling Updates

All five application Deployments use:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1
```

This means Kubernetes maintains the existing available pod while starting the replacement pod during an update.

Verified deployments:

```text
admin       RollingUpdate   0   1
auth        RollingUpdate   0   1
chat        RollingUpdate   0   1
frontend    RollingUpdate   0   1
streaming   RollingUpdate   0   1
```

---

# 17. Authentication Version Update

The authentication image was updated to:

```text
streaming-auth:1.0.1
```

Using Helm:

```bash
helm upgrade streamingapp ./streamingapp \
  -n streamingapp \
  --set services.auth.tag=1.0.1
```

The rollout was verified successfully:

```bash
kubectl rollout status deployment/auth \
  -n streamingapp
```

The running deployment used:

```text
218014315198.dkr.ecr.ap-south-1.amazonaws.com/streaming-auth:1.0.1
```

---

# 18. Self-Healing

Kubernetes self-healing was tested by deleting a running Streaming pod.

Example:

```bash
kubectl delete pod <STREAMING_POD_NAME> \
  -n streamingapp
```

Kubernetes automatically created a replacement pod.

This demonstrated the Deployment controller's ability to maintain the desired replica count.

---

# 19. Application Smoke Tests

The following application tests were performed.

### 1. Pod health

Verified that application pods were:

```text
Running
Ready
```

### 2. Authentication

Registration/login API was tested through the Ingress.

Example:

```bash
curl -i -X POST \
  http://imreading.xyz/api/login \
  -H "Content-Type: application/json" \
  -d '{"email":"seema.test@example.com","password":"Test@12345"}'
```

Authentication returned a successful response with JWT authentication information.

### 3. Video upload

A small test video and thumbnail were uploaded through the Admin dashboard.

### 4. Playback

The uploaded video was successfully retrieved through the streaming API.

The streaming endpoint was tested using an HTTP Range request:

```bash
curl -i \
  -H "Range: bytes=0-1023" \
  http://imreading.xyz/api/streaming/stream/<VIDEO_ID> \
  -o /tmp/video-test
```

### 5. Live chat

Live chat was tested using two browser tabs.

Messages sent from one tab were received by the other tab.

### 6. Kubernetes self-healing

A Streaming pod was deleted manually and Kubernetes created a replacement.

---

# 20. CloudWatch Monitoring and Logging

Amazon CloudWatch Container Insights was configured for the EKS cluster.

Monitoring components included:

```text
CloudWatch Observability
CloudWatch Agent
Fluent Bit
Container Insights
```

Container Insights provided cluster, node, pod, and container metrics.

Application logs were collected into CloudWatch Logs.

Example log groups:

```text
/aws/containerinsights/<cluster>/application
/aws/containerinsights/<cluster>/dataplane
/aws/containerinsights/<cluster>/host
/aws/containerinsights/<cluster>/performance
```

EC2 CPU alarms were also configured with a 70% threshold.

---

# 21. Monitoring Architecture

```text
Kubernetes Pods
      |
      v
CloudWatch Agent / Fluent Bit
      |
      +--------------------+
      |                    |
      v                    v
CloudWatch Metrics     CloudWatch Logs
      |
      v
CloudWatch Alarms
```

This provides visibility into cluster performance and application/container logs.

---

# 22. Useful Kubernetes Commands

Check pods:

```bash
kubectl get pods -n streamingapp
```

Check deployments:

```bash
kubectl get deployments -n streamingapp
```

Check services:

```bash
kubectl get services -n streamingapp
```

Check Ingress:

```bash
kubectl get ingress -n streamingapp
```

Check HPA:

```bash
kubectl get hpa -n streamingapp
```

Check PVC:

```bash
kubectl get pvc -n streamingapp
```

Check StatefulSet:

```bash
kubectl get statefulset -n streamingapp
```

Check events:

```bash
kubectl get events \
  -n streamingapp \
  --sort-by=.lastTimestamp
```

---

# 23. Useful Helm Commands

List releases:

```bash
helm list -n streamingapp
```

Check release status:

```bash
helm status streamingapp \
  -n streamingapp
```

View values:

```bash
helm get values streamingapp \
  -n streamingapp
```

View release history:

```bash
helm history streamingapp \
  -n streamingapp
```

Upgrade:

```bash
helm upgrade streamingapp ./streamingapp \
  -n streamingapp
```

Uninstall:

```bash
helm uninstall streamingapp \
  -n streamingapp
```

---

# 24. Project Validation Summary

| Requirement                    | Status |
| ------------------------------ | ------ |
| MERN application containerized | ✅      |
| Docker images created          | ✅      |
| ECR repositories created       | ✅      |
| Images pushed to ECR           | ✅      |
| Jenkins CI pipeline            | ✅      |
| Jenkins ECR authentication     | ✅      |
| Jenkins ECR image push         | ✅      |
| Slack success notification     | ✅      |
| Slack failure notification     | ✅      |
| EKS deployment                 | ✅      |
| Helm chart                     | ✅      |
| Five Deployments               | ✅      |
| Five Services                  | ✅      |
| ConfigMap                      | ✅      |
| Secret                         | ✅      |
| MongoDB StatefulSet            | ✅      |
| Persistent storage             | ✅      |
| Readiness probes               | ✅      |
| Liveness probes                | ✅      |
| HPA                            | ✅      |
| RollingUpdate                  | ✅      |
| `maxUnavailable: 0`            | ✅      |
| `maxSurge: 1`                  | ✅      |
| Auth `1.0.1` rollout           | ✅      |
| ALB Ingress                    | ✅      |
| Custom domain                  | ✅      |
| S3 media storage               | ✅      |
| IRSA                           | ✅      |
| CloudWatch monitoring          | ✅      |
| CloudWatch logging             | ✅      |
| Self-healing test              | ✅      |
| Video upload test              | ✅      |
| Video playback test            | ✅      |
| Live chat test                 | ✅      |

---

# 25. Project Outcome

The project demonstrates a complete container orchestration workflow:

```text
Developer
    |
    v
GitHub
    |
    v
Jenkins CI/CD
    |
    v
Docker Build
    |
    v
Amazon ECR
    |
    v
Helm
    |
    v
Amazon EKS
    |
    +---------------------+
    |                     |
    v                     v
Kubernetes             AWS Services
Workloads              |
    |                   +-- ALB
    |                   +-- S3
    |                   +-- CloudWatch
    |
    +-- Frontend
    +-- Auth
    +-- Streaming
    +-- Admin
    +-- Chat
    +-- MongoDB
```

The application was validated for deployment, scaling, rolling updates, authentication, media upload, media playback, live chat, persistence, self-healing, monitoring, logging, and CI/CD notifications.

---

# 26. Detailed Deployment Guide

For the complete step-by-step Helm installation process and instructions for accessing the application through the AWS ALB Ingress, see:

**[Helm Installation and Ingress Guide]([Helm Installation and Ingress Guide](https://github.com/KrSeema/StreamingAppKubernetes_SeemaKr/blob/main/README_HelmInstall_Ingress.md))**

---

# 27. Production Considerations

For a production environment, the following improvements should be considered:

* Separate application, monitoring, and platform namespaces.
* Apply appropriate RBAC policies.
* Configure resource requests and limits.
* Enable HTTPS using AWS Certificate Manager or cert-manager.
* Use carefully tuned HPA CPU and memory targets.
* Configure cluster autoscaling.
* Use Kubernetes NetworkPolicies where appropriate.
* Store sensitive configuration in a dedicated secrets-management solution.
* Configure regular MongoDB backups.
* Deploy critical workloads across multiple Availability Zones.
* Configure stronger observability, alerting, and incident-response procedures.
* Use appropriate backup and disaster-recovery strategies.
* Apply least-privilege IAM policies.
* Use production DNS and TLS configuration.

---

# 28. Cleanup

When the temporary EKS environment is no longer required, remove the Kubernetes infrastructure and other AWS resources created specifically for the project.

Example EKS cleanup:

```bash
eksctl delete cluster \
  --name streaming-app-cluster \
  --region ap-south-1 \
  --wait
```

Verify:

```bash
eksctl get cluster \
  --region ap-south-1
```

Before deleting resources, verify that they belong to this project and are not used by another application.

---

# License

This project is intended for educational and DevOps demonstration purposes.
