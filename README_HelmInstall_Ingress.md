# StreamingApp — Kubernetes Deployment with Helm

A production-style MERN/Streaming application deployed on Amazon EKS using Kubernetes and Helm.

## Architecture

```text
                         Internet
                            |
                            v
                +----------------------+
                |   AWS ALB Ingress    |
                |  streamingapp.local  |
                +----------+-----------+
                           |
        +------------------+------------------+
        |                  |                  |
        v                  v                  v
   Frontend            Auth/Admin/Chat    Streaming
   Service              Services          Service
        |                  |                  |
        +------------------+------------------+
                           |
                           v
                    MongoDB StatefulSet
                           |
                           v
                      EBS Persistent
                         Volume
```

## Services

| Service   | Container Port | Kubernetes Service |
| --------- | -------------: | ------------------ |
| Frontend  |             80 | `frontend-svc`     |
| Auth      |           3001 | `auth`             |
| Streaming |           3002 | `streaming-svc`    |
| Admin     |           3003 | `admin-svc`        |
| Chat      |           3004 | `chat-svc`         |
| MongoDB   |          27017 | `mongo`            |

## Prerequisites

The following tools are required:

* AWS CLI
* kubectl
* Helm
* eksctl
* An existing Amazon EKS cluster
* Docker images available from Docker Hub

Docker Hub repositories:

* `seemakr/streaming-frontend`
* `seemakr/streaming-auth`
* `seemakr/streaming-stream`
* `seemakr/streaming-admin`
* `seemakr/streaming-chat`

## AWS Configuration

AWS region used for this project:

```bash
export AWS_REGION=ap-south-1
```

Verify AWS credentials:

```bash
aws sts get-caller-identity
```

Connect kubectl to the EKS cluster:

```bash
aws eks update-kubeconfig \
  --region ap-south-1 \
  --name streaming-app-cluster
```

Verify the cluster:

```bash
kubectl get nodes
```

Expected result:

```text
NAME                                             STATUS   ROLES    AGE
ip-192-168-27-47.ap-south-1.compute.internal    Ready    <none>   ...
ip-192-168-35-155.ap-south-1.compute.internal   Ready    <none>   ...
ip-192-168-80-184.ap-south-1.compute.internal   Ready    <none>   ...
```

## Project Directory

Clone or copy the project and move into the Helm chart directory:

```bash
cd /home/seema/StreamingApp
```

The Helm chart is located at:

```text
StreamingApp/
└── streamingapp/
    ├── Chart.yaml
    ├── values.yaml
    └── templates/
        ├── namespace.yaml
        ├── configmap.yaml
        ├── secret.yaml
        ├── serviceaccount.yaml
        ├── mongo-service.yaml
        ├── mongo-statefulset.yaml
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

Move into the chart:

```bash
cd /home/seema/StreamingApp/streamingapp
```

## Validate the Helm Chart

Run Helm lint:

```bash
helm lint .
```

Expected:

```text
1 chart(s) linted, 0 chart(s) failed
```

Render the Kubernetes manifests:

```bash
helm template streamingapp . \
  --namespace streamingapp
```

## Install the Application

Create the namespace and install the Helm release:

```bash
helm install streamingapp . \
  --namespace streamingapp \
  --create-namespace
```

Verify the Helm release:

```bash
helm list -n streamingapp
```

Expected status:

```text
STATUS: deployed
```

## Verify Kubernetes Resources

Check all pods:

```bash
kubectl get pods -n streamingapp
```

All application pods should eventually show:

```text
Running
```

Check services:

```bash
kubectl get svc -n streamingapp
```

Check deployments:

```bash
kubectl get deployments -n streamingapp
```

Check MongoDB StatefulSet:

```bash
kubectl get statefulset -n streamingapp
```

Check MongoDB PVC:

```bash
kubectl get pvc -n streamingapp
```

The MongoDB PVC should show:

```text
STATUS   Bound
```

## Ingress

The application uses the AWS Load Balancer Controller and an AWS Application Load Balancer.

Check the Ingress:

```bash
kubectl get ingress -n streamingapp
```

Example:

```text
NAME                  CLASS   HOST                 ADDRESS
streamingapp-ingress  alb     streamingapp.local  k8s-streamin-streamin-671c5478c2-1060661347.ap-south-1.elb.amazonaws.com
```

Get the ALB hostname directly:

```bash
kubectl get ingress streamingapp-ingress \
  -n streamingapp \
  -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

The application is exposed through:

```text
http://streamingapp.local
```

### Local DNS / Hosts File

Because `streamingapp.local` is used as the Ingress host, add the ALB hostname to your local hosts/DNS configuration if required.

For testing directly against the ALB, use the Host header:

```bash
curl -I \
  -H "Host: streamingapp.local" \
  http://k8s-streamin-streamin-671c5478c2-1060661347.ap-south-1.elb.amazonaws.com/
```

Expected:

```text
HTTP/1.1 200 OK
```

## Ingress Routing

The ALB routes requests to the appropriate Kubernetes Service.

```text
/                  → frontend-svc:80
/api               → auth:3001
/api/streaming     → streaming-svc:3002
/api/admin         → admin-svc:3003
/api/chat          → chat-svc:3004
/socket.io         → chat-svc:3004
```

The `/socket.io` route is required for Socket.IO live chat.

## Access the Application

Open:

```text
http://streamingapp.local
```

The application provides:

* User registration
* User login
* Video browsing
* Video playback
* Admin dashboard
* Video and thumbnail upload
* Live chat
* MongoDB-backed application data

## Scaling

Application replicas are controlled through Helm values.

For example, scale the Auth service to four replicas:

```bash
helm upgrade streamingapp . \
  --namespace streamingapp \
  --set services.auth.replicas=4
```

Verify:

```bash
kubectl get pods -n streamingapp -l app=auth
```

The Auth deployment uses a rolling update strategy.

## Rolling Update

The application deployments use:

```text
strategy: RollingUpdate
maxUnavailable: 0
maxSurge: 1
```

This allows new Pods to become available before old Pods are removed.

Check the deployment:

```bash
kubectl get deployment frontend -n streamingapp -o wide
```

## Pod Self-Healing

Kubernetes automatically recreates Pods managed by a Deployment.

Example:

```bash
kubectl get pods -n streamingapp -l app=frontend
```

Delete the current frontend Pod:

```bash
kubectl delete pod <frontend-pod-name> -n streamingapp
```

Watch Kubernetes create the replacement:

```bash
kubectl get pods -n streamingapp -l app=frontend -w
```

The replacement Pod should eventually become:

```text
1/1   Running
```

Verify that the application is still accessible:

```bash
curl -I \
  -H "Host: streamingapp.local" \
  http://k8s-streamin-streamin-671c5478c2-1060661347.ap-south-1.elb.amazonaws.com/
```

Expected:

```text
HTTP/1.1 200 OK
```

## Health Probes

The application Deployments include Kubernetes readiness and liveness probes.

Check probes on a Deployment:

```bash
kubectl describe deployment frontend -n streamingapp
```

Look for:

```text
Liveness
Readiness
```

These probes allow Kubernetes to determine whether a container is healthy and ready to receive traffic.

## Smoke Tests

### 1. Check Pods

```bash
kubectl get pods -n streamingapp
```

All application Pods should be `Running` and `Ready`.

### 2. Register/Login

Open:

```text
http://streamingapp.local
```

Register a user and log in.

### 3. Admin Upload

Log in using an admin account.

Open the Admin Dashboard and upload:

* Video
* Thumbnail
* Title
* Description
* Genre
* Release year

The uploaded media is stored in Amazon S3 and metadata is stored in MongoDB.

### 4. Video Playback

Open the Browse page.

Select the uploaded video.

Verify that the video plays successfully.

### 5. Live Chat

Open the video player.

Open the Chat panel.

Send a message and verify that it is delivered.

The Socket.IO endpoint is routed through:

```text
/socket.io → chat-svc:3004
```

### 6. Self-Healing

Delete a frontend Pod and verify that Kubernetes creates a replacement Pod.

Then verify the application still returns:

```text
HTTP/1.1 200 OK
```

## Useful Troubleshooting Commands

Check all resources:

```bash
kubectl get all -n streamingapp
```

Check Ingress:

```bash
kubectl describe ingress streamingapp-ingress -n streamingapp
```

Check a Deployment:

```bash
kubectl describe deployment frontend -n streamingapp
```

Check Pod logs:

```bash
kubectl logs <pod-name> -n streamingapp
```

Check previous container logs:

```bash
kubectl logs <pod-name> -n streamingapp --previous
```

Check Helm release history:

```bash
helm history streamingapp -n streamingapp
```

Check Helm status:

```bash
helm status streamingapp -n streamingapp
```

## Uninstall

To remove the Helm release:

```bash
helm uninstall streamingapp \
  --namespace streamingapp
```

Check the namespace:

```bash
kubectl get namespace streamingapp
```

MongoDB uses persistent storage, so verify PVCs before deleting storage-related resources.

## Final Deployment Summary

```text
Application:
StreamingApp

Platform:
Amazon EKS

Region:
ap-south-1 (Mumbai)

Namespace:
streamingapp

Package Manager:
Helm

Ingress:
AWS Application Load Balancer

Ingress Host:
streamingapp.local

Database:
MongoDB StatefulSet

Persistent Storage:
Amazon EBS CSI / gp2

Object Storage:
Amazon S3

Container Registry:
Docker Hub

Live Chat:
Socket.IO

Self-Healing:
Kubernetes Deployment

Scaling:
Helm-managed replica configuration
```

## Final Verification

Before submission, run:

```bash
helm status streamingapp -n streamingapp
```

```bash
kubectl get pods -n streamingapp
```

```bash
kubectl get svc -n streamingapp
```

```bash
kubectl get ingress -n streamingapp
```

```bash
kubectl get pvc -n streamingapp
```

```bash
kubectl get deployments -n streamingapp
```

The application should be accessible through the Ingress hostname and all required services should be healthy.
