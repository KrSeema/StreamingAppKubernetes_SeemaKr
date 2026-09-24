# StreamingApp — Kubernetes Container Orchestration & Scaling

A multi-service MERN/Streaming application containerized with Docker, packaged with Helm, deployed on Amazon EKS, exposed through an AWS Application Load Balancer, and verified with scaling, rolling updates, live chat, video upload/playback, and Kubernetes self-healing.

## Project Tasks

```text
TASK 1  Containerize Every Service
          ↓
TASK 2  Write the Kubernetes Manifests
          ↓
TASK 3  Package It as a Helm Chart
          ↓
TASK 4  Expose Traffic with Ingress
          ↓
TASK 5  Deploy, Scale & Update
          ↓
TASK 6  Verify & Smoke Test
```

## Architecture

```text
                         Internet
                            |
                            v
                 AWS Application Load Balancer
                            |
                     streamingapp.local
                            |
       +--------------------+---------------------+
       |          |          |          |          |
       v          v          v          v          v
   Frontend     Auth       Admin      Chat     Streaming
     :80        :3001      :3003      :3004       :3002
       |          |          |          |          |
       +----------+----------+----------+----------+
                            |
                            v
                     MongoDB StatefulSet
                            |
                            v
                       EBS Persistent
                          Volume
```

## Technology Stack

- Docker
- Docker Hub
- Kubernetes
- Amazon EKS
- Helm
- AWS Application Load Balancer
- AWS Load Balancer Controller
- MongoDB
- Amazon EBS CSI
- Amazon S3
- IRSA
- Socket.IO
- React
- Node.js / Express

---

# TASK 1 — Containerize Every Service

The application contains five containerized services:

| Service | Docker Image | Tag |
|---|---|---|
| Frontend | `seemakr/streaming-frontend` | `1.0.3` |
| Auth | `seemakr/streaming-auth` | `1.0.1` |
| Streaming | `seemakr/streaming-stream` | `1.0.0` |
| Admin | `seemakr/streaming-admin` | `1.0.0` |
| Chat | `seemakr/streaming-chat` | `1.0.0` |

MongoDB uses the official:

```text
mongo:6
```

## Build Docker Images

Example:

```bash
docker build -t seemakr/streaming-frontend:1.0.3 ./frontend
```

Build the remaining services similarly:

```bash
docker build -t seemakr/streaming-auth:1.0.1 ./backend/authService
docker build -t seemakr/streaming-stream:1.0.0 ./backend/streamingService
docker build -t seemakr/streaming-admin:1.0.0 ./backend/adminService
docker build -t seemakr/streaming-chat:1.0.0 ./backend/chatService
```

## Push Images

```bash
docker push seemakr/streaming-frontend:1.0.3
docker push seemakr/streaming-auth:1.0.1
docker push seemakr/streaming-stream:1.0.0
docker push seemakr/streaming-admin:1.0.0
docker push seemakr/streaming-chat:1.0.0
```

Docker Hub repositories:

- https://hub.docker.com/r/seemakr/streaming-frontend
- https://hub.docker.com/r/seemakr/streaming-auth
- https://hub.docker.com/r/seemakr/streaming-stream
- https://hub.docker.com/r/seemakr/streaming-admin
- https://hub.docker.com/r/seemakr/streaming-chat

## Local Docker Verification

Verify the images:

```bash
docker images | grep streaming
```

The application was also tested locally before Kubernetes deployment.

---

# TASK 2 — Write the Kubernetes Manifests

The Kubernetes resources include:

- Namespace
- ConfigMap
- Secret
- 5 Deployments
- 5 application Services
- MongoDB StatefulSet
- MongoDB Service
- PersistentVolumeClaim through `volumeClaimTemplates`
- ServiceAccount for AWS S3 access
- Ingress

## Kubernetes Resource Structure

```text
streamingapp/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── namespace.yaml
    ├── configmap.yaml
    ├── secret.yaml
    ├── serviceaccount.yaml
    ├── mongo-statefulset.yaml
    ├── mongo-service.yaml
    ├── frontend-deployment.yaml
    ├── frontend-service.yaml
    ├── auth-deployment.yaml
    ├── auth-service.yaml
    ├── admin-deployment.yaml
    ├── admin-service.yaml
    ├── chat-deployment.yaml
    ├── chat-service.yaml
    ├── streaming-deployment.yaml
    ├── streaming-service.yaml
    └── ingress.yaml
```

## Namespace

All application resources run in:

```text
streamingapp
```

Create it manually if needed:

```bash
kubectl create namespace streamingapp
```

The Helm installation can also create it automatically.

## Verify Kubernetes Cluster

AWS region:

```text
ap-south-1
```

EKS cluster:

```text
streaming-app-cluster
```

Configure kubectl:

```bash
aws eks update-kubeconfig \
  --region ap-south-1 \
  --name streaming-app-cluster
```

Verify:

```bash
kubectl get nodes
```

---

# TASK 3 — Package It as a Helm Chart

The Helm chart is:

```text
streamingapp
```

Chart metadata:

```yaml
apiVersion: v2
name: streamingapp
description: Helm chart for StreamingApp Kubernetes deployment
type: application
version: 1.0.0
```

## Validate the Chart

From the project directory:

```bash
cd /home/seema/StreamingApp/streamingapp
```

Run:

```bash
helm lint .
```

Expected:

```text
1 chart(s) linted, 0 chart(s) failed
```

Render the templates:

```bash
helm template streamingapp . \
  --namespace streamingapp
```

The rendered output can be reviewed before installation.

---

# TASK 4 — Expose Traffic with Ingress

The application uses the AWS Load Balancer Controller and an internet-facing AWS Application Load Balancer.

Check the Ingress:

```bash
kubectl get ingress -n streamingapp
```

Example:

```text
NAME                  CLASS   HOST                 ADDRESS
streamingapp-ingress  alb     streamingapp.local  <ALB-DNS>
```

Get the ALB DNS name:

```bash
kubectl get ingress streamingapp-ingress \
  -n streamingapp \
  -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

## Ingress Routing

```text
/                  → frontend-svc:80
/api               → auth:3001
/api/streaming     → streaming-svc:3002
/api/admin         → admin-svc:3003
/api/chat          → chat-svc:3004
/socket.io         → chat-svc:3004
```

The `/socket.io` route allows Socket.IO WebSocket traffic to reach the Chat service.

## Access the Application

The configured Ingress host is:

```text
http://streamingapp.local
```

For direct ALB testing:

```bash
curl -I \
  -H "Host: streamingapp.local" \
  http://<ALB-DNS>/
```

Expected:

```text
HTTP/1.1 200 OK
```

---

# TASK 5 — Deploy, Scale & Update

## 5.1 Install the Helm Release

From:

```bash
cd /home/seema/StreamingApp/streamingapp
```

Install:

```bash
helm install streamingapp . \
  --namespace streamingapp \
  --create-namespace
```

Verify:

```bash
helm list -n streamingapp
```

Check all resources:

```bash
kubectl get all -n streamingapp
```

Check Pods:

```bash
kubectl get pods -n streamingapp
```

---

## 5.2 Replica Scaling

Replica scaling was demonstrated live on the Auth Deployment.

The Auth service was initially running with:

```text
4 replicas
```

### Scale IN: 4 → 2

```bash
helm upgrade streamingapp ./streamingapp \
  --namespace streamingapp \
  --set services.auth.replicas=2
```

Verify:

```bash
kubectl get deployment auth -n streamingapp
```

Expected:

```text
auth   2/2   2   2
```

Verify the Pods:

```bash
kubectl get pods -n streamingapp -l app=auth
```

Two healthy Auth Pods were observed.

### Scale OUT: 2 → 4

```bash
helm upgrade streamingapp ./streamingapp \
  --namespace streamingapp \
  --set services.auth.replicas=4
```

Verify:

```bash
kubectl get deployment auth -n streamingapp
```

Expected:

```text
auth   4/4   4   4
```

This demonstrates Kubernetes scaling through Helm-managed desired state.

---

## 5.3 Rolling Updates

The application Deployments use Kubernetes RollingUpdate.

Verify the strategy:

```bash
kubectl get deployment auth -n streamingapp \
  -o jsonpath='{.spec.strategy.type}{"\n"}{.spec.strategy.rollingUpdate.maxUnavailable}{"\n"}{.spec.strategy.rollingUpdate.maxSurge}{"\n"}'
```

Expected:

```text
RollingUpdate
0
1
```

Meaning:

```text
maxUnavailable = 0
maxSurge        = 1
```

A rollout can be monitored with:

```bash
kubectl rollout status deployment/auth -n streamingapp
```

Expected:

```text
deployment "auth" successfully rolled out
```

Check rollout history:

```bash
kubectl rollout history deployment/auth -n streamingapp
```

---

# TASK 6 — Verify & Smoke Test

The following end-to-end tests were completed successfully.

## 6.1 All Pods Running and Ready

```bash
kubectl get pods -n streamingapp
```

All required application Pods reached:

```text
1/1 Running
```

and the Auth service was scaled to four replicas.

## 6.2 Register and Login

The application was accessed through the Ingress:

```text
http://streamingapp.local
```

User registration and login were verified successfully.

## 6.3 Admin Video Upload

An admin account was used to upload:

- Video
- Thumbnail
- Video metadata

The media was uploaded to Amazon S3 using the Kubernetes ServiceAccount/IRSA configuration.

The metadata was stored in MongoDB.

## 6.4 Video Playback

The uploaded video appeared in the Browse page.

Video playback was successfully verified through the Ingress.

## 6.5 Live Chat

Live chat was tested from the video player.

The Socket.IO route:

```text
/socket.io
```

was routed through the ALB to:

```text
chat-svc:3004
```

Messages were successfully exchanged.

## 6.6 Pod Self-Healing

A frontend Pod was deliberately deleted:

```bash
kubectl delete pod <frontend-pod-name> -n streamingapp
```

Kubernetes automatically created a replacement Pod.

The replacement reached:

```text
1/1 Running
```

The application remained accessible.

Final HTTP verification:

```bash
curl -I \
  -H "Host: streamingapp.local" \
  http://<ALB-DNS>/
```

Result:

```text
HTTP/1.1 200 OK
```

This demonstrated Kubernetes Deployment self-healing without application downtime observed during the test.

---

# MongoDB Persistent Storage

MongoDB runs as a StatefulSet.

Check:

```bash
kubectl get statefulset -n streamingapp
```

Check the PVC:

```bash
kubectl get pvc -n streamingapp
```

The MongoDB data volume uses the Amazon EBS CSI driver and `gp2` StorageClass.

Expected:

```text
STATUS   Bound
```

---

# Amazon S3 and IRSA

The Admin and Streaming services use an IAM role through Kubernetes ServiceAccount and EKS IRSA.

ServiceAccount:

```text
streamingapp-s3
```

Verify:

```bash
kubectl get serviceaccount streamingapp-s3 \
  -n streamingapp \
  -o yaml
```

The role provides access to the application S3 bucket for media operations.

The application bucket used in the deployment was:

```text
streamingapp-media-218014315198
```

---

# Useful Verification Commands

## Helm

```bash
helm status streamingapp -n streamingapp
```

```bash
helm history streamingapp -n streamingapp
```

## Pods

```bash
kubectl get pods -n streamingapp
```

## Deployments

```bash
kubectl get deployments -n streamingapp
```

## Services

```bash
kubectl get svc -n streamingapp
```

## StatefulSet

```bash
kubectl get statefulset -n streamingapp
```

## PVC

```bash
kubectl get pvc -n streamingapp
```

## Ingress

```bash
kubectl get ingress -n streamingapp
```

## Logs

```bash
kubectl logs <pod-name> -n streamingapp
```

## Describe a resource

```bash
kubectl describe pod <pod-name> -n streamingapp
```

---

# Production Improvements

For a production cluster, I would separate workloads into dedicated namespaces with appropriate RBAC and network policies, use HTTPS/TLS certificates through AWS Certificate Manager and the Application Load Balancer, configure Horizontal Pod Autoscalers for services based on resource or application metrics, store sensitive configuration in AWS Secrets Manager rather than plain Kubernetes values, define CPU/memory requests and limits, run multiple replicas across Availability Zones, add PodDisruptionBudgets, centralized logging and monitoring, automated backups, and disaster-recovery procedures for persistent data.

---

# Final Submission Checklist

```text
[✓] Task 1 — Containerize every service
[✓] Task 2 — Kubernetes manifests
[✓] Task 3 — Helm chart
[✓] Task 4 — AWS ALB Ingress
[✓] Task 5 — Deploy, scale and update
[✓] Task 6 — Verify and smoke test

[✓] Docker images pushed to Docker Hub
[✓] EKS deployment
[✓] MongoDB StatefulSet + persistent storage
[✓] Ingress routing
[✓] Replica scaling demonstrated
[✓] RollingUpdate strategy demonstrated
[✓] Video upload tested
[✓] Video playback tested
[✓] Live chat tested
[✓] Pod self-healing tested
```

## Cleanup

To remove the Helm release:

```bash
helm uninstall streamingapp \
  --namespace streamingapp
```

Verify:

```bash
kubectl get all -n streamingapp
```

> **Note:** Persistent storage should be reviewed separately before deleting the namespace or EBS volumes if the MongoDB data needs to be retained.