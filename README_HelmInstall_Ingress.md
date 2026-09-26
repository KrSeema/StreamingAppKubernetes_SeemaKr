# StreamingApp — Helm Installation and Ingress Access

This document explains how to deploy the StreamingApp Helm chart to an Amazon EKS cluster and access the application through an AWS Application Load Balancer (ALB) Ingress.

---

## 1. Prerequisites

Before starting, make sure the following tools are installed:

* AWS CLI
* kubectl
* Helm
* eksctl

Verify:

```bash
aws --version
kubectl version --client
helm version
eksctl version
```

The commands in this document use the AWS Mumbai region:

```text
ap-south-1
```

---

# 2. Configure AWS CLI

Configure AWS credentials if required:

```bash
aws configure
```

Verify the AWS identity:

```bash
aws sts get-caller-identity
```

Set the AWS region:

```bash
export AWS_REGION=ap-south-1
```

---

# 3. Create or Use an EKS Cluster

If the EKS cluster already exists, skip this section.

Example:

```bash
eksctl create cluster \
  --name streaming-app-cluster \
  --region ap-south-1 \
  --nodes 2 \
  --node-type t3.medium
```

Check the cluster:

```bash
eksctl get cluster --region ap-south-1
```

Update the local kubeconfig:

```bash
aws eks update-kubeconfig \
  --name streaming-app-cluster \
  --region ap-south-1
```

Verify connectivity:

```bash
kubectl get nodes
```

Expected:

```text
NAME                         STATUS   ROLES    AGE   VERSION
ip-192-168-xx-xx...          Ready    <none>   ...   ...
```

---

# 4. Create the Application Namespace

Create the namespace:

```bash
kubectl create namespace streamingapp
```

If it already exists, use:

```bash
kubectl get namespace streamingapp
```

---

# 5. Install AWS Load Balancer Controller

The StreamingApp uses a Kubernetes Ingress with:

```yaml
ingressClassName: alb
```

Therefore, the AWS Load Balancer Controller must be installed in the EKS cluster.

Check whether it is already installed:

```bash
kubectl get deployment \
  aws-load-balancer-controller \
  -n kube-system
```

Check its pods:

```bash
kubectl get pods -n kube-system \
  -l app.kubernetes.io/name=aws-load-balancer-controller
```

The controller should have Running pods.

If the controller is not installed, install it according to the AWS Load Balancer Controller setup for the EKS cluster.

---

# 6. Verify the Helm Chart

From the project root:

```bash
cd ~/StreamingApp
```

Check the chart:

```bash
helm lint ./streamingapp
```

Expected:

```text
1 chart(s) linted, 0 chart(s) failed
```

Render the manifests locally before installation:

```bash
helm template streamingapp ./streamingapp \
  -n streamingapp
```

This allows the generated Kubernetes resources to be inspected without installing them.

---

# 7. Install the StreamingApp Helm Chart

Install the application:

```bash
helm install streamingapp ./streamingapp \
  -n streamingapp
```

If the namespace does not exist, use:

```bash
helm install streamingapp ./streamingapp \
  --create-namespace \
  -n streamingapp
```

Check the Helm release:

```bash
helm list -n streamingapp
```

Expected:

```text
NAME           NAMESPACE      STATUS
streamingapp   streamingapp   deployed
```

---

# 8. Check Kubernetes Resources

Check all resources:

```bash
kubectl get all -n streamingapp
```

Check deployments:

```bash
kubectl get deployments -n streamingapp
```

Check pods:

```bash
kubectl get pods -n streamingapp -o wide
```

All application pods should eventually show:

```text
STATUS    Running
READY     1/1
```

Check services:

```bash
kubectl get services -n streamingapp
```

---

# 9. Check MongoDB

The application uses MongoDB deployed inside Kubernetes as a StatefulSet.

Check the StatefulSet:

```bash
kubectl get statefulset -n streamingapp
```

Check the MongoDB pod:

```bash
kubectl get pods -n streamingapp -l app=mongo
```

Check the PersistentVolumeClaim:

```bash
kubectl get pvc -n streamingapp
```

The MongoDB PVC should be:

```text
STATUS: Bound
```

---

# 10. Check the Ingress

Check the StreamingApp Ingress:

```bash
kubectl get ingress -n streamingapp
```

Example:

```text
NAME                  CLASS   HOSTS         ADDRESS
streamingapp-ingress  alb     imreading.xyz k8s-streamin-....
```

For more details:

```bash
kubectl describe ingress streamingapp-ingress \
  -n streamingapp
```

The `ADDRESS` field should contain the AWS Application Load Balancer DNS name.

---

# 11. How the Ingress Works

The request flow is:

```text
User Browser
     |
     v
imreading.xyz
     |
     v
Cloudflare DNS
     |
     v
AWS Application Load Balancer
     |
     v
Kubernetes Ingress
     |
     +-------------------+
     |        |          |
     v        v          v
Frontend   Backend     Chat
Service    Services    Service
     |        |          |
     v        v          v
   Pods      Pods       Pods
```

The AWS Load Balancer Controller watches the Kubernetes Ingress resource and creates/manages the AWS Application Load Balancer.

---

# 12. Ingress Host

The application uses:

```text
imreading.xyz
```

The Ingress configuration contains:

```yaml
spec:
  ingressClassName: alb

  rules:
    - host: imreading.xyz
```

Therefore, requests should be made using the configured domain.

---

# 13. Application Routes

The Ingress routes requests to the appropriate Kubernetes Services.

| Path                 | Application            |
| -------------------- | ---------------------- |
| `/`                  | Frontend               |
| `/api/login`         | Authentication service |
| `/api/register`      | Authentication service |
| `/api/streaming/...` | Streaming service      |
| `/api/admin/...`     | Admin service          |
| `/api/chat/...`      | Chat service           |

The exact routing rules are defined in:

```text
streamingapp/templates/ingress.yaml
```

---

# 14. Access the Application

After the Ingress receives an AWS ALB address, verify it:

```bash
kubectl get ingress streamingapp-ingress \
  -n streamingapp
```

Example:

```text
ADDRESS: k8s-streamin-streamin-671c5478c2-188693131.ap-south-1.elb.amazonaws.com
```

Open the configured application domain in a browser:

```text
http://imreading.xyz
```

The StreamingApp frontend should load.

---

# 15. Verify the ALB Directly

You can also test the ALB using its DNS name.

First retrieve it:

```bash
kubectl get ingress streamingapp-ingress \
  -n streamingapp \
  -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

Example output:

```text
k8s-streamin-streamin-671c5478c2-188693131.ap-south-1.elb.amazonaws.com
```

Because the Ingress uses host-based routing, the `Host` header should be supplied:

```bash
curl -I \
  -H "Host: imreading.xyz" \
  http://<ALB-DNS-NAME>
```

Expected:

```text
HTTP/1.1 200 OK
```

---

# 16. Verify the Domain

Test the application domain:

```bash
curl -I http://imreading.xyz
```

Expected:

```text
HTTP/1.1 200 OK
```

You can also test the authentication API:

```bash
curl -i -X POST \
  http://imreading.xyz/api/login \
  -H "Content-Type: application/json" \
  -d '{"email":"seema.test@example.com","password":"Test@12345"}'
```

A successful login should return an HTTP success response and authentication information.

---

# 17. Verify Streaming

The streaming API requires an HTTP Range request.

Example:

```bash
curl -i \
  -H "Range: bytes=0-1023" \
  http://imreading.xyz/api/streaming/stream/<VIDEO_ID> \
  -o /tmp/video-test
```

Replace:

```text
<VIDEO_ID>
```

with the MongoDB video document ID.

A successful response should contain:

```text
HTTP/1.1 206 Partial Content
```

---

# 18. Verify Chat

Open the application in two browser tabs.

1. Login in both tabs.
2. Open a video.
3. Open the chat section.
4. Send a message from the first tab.
5. Confirm that the message appears in the second tab.

This validates the chat API/WebSocket functionality.

---

# 19. Check Application Logs

Check logs for each service:

```bash
kubectl logs \
  deployment/auth \
  -n streamingapp
```

```bash
kubectl logs \
  deployment/streaming \
  -n streamingapp
```

```bash
kubectl logs \
  deployment/admin \
  -n streamingapp
```

```bash
kubectl logs \
  deployment/chat \
  -n streamingapp
```

Frontend:

```bash
kubectl logs \
  deployment/frontend \
  -n streamingapp
```

---

# 20. Check Rollout Status

Verify all deployments:

```bash
kubectl rollout status deployment/auth -n streamingapp
kubectl rollout status deployment/streaming -n streamingapp
kubectl rollout status deployment/admin -n streamingapp
kubectl rollout status deployment/chat -n streamingapp
kubectl rollout status deployment/frontend -n streamingapp
```

All deployments should report successful rollout.

---

# 21. Verify Rolling Update Strategy

The application deployments use:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1
```

Verify:

```bash
kubectl get deployments -n streamingapp \
  -o custom-columns='NAME:.metadata.name,STRATEGY:.spec.strategy.type,MAX_UNAVAILABLE:.spec.strategy.rollingUpdate.maxUnavailable,MAX_SURGE:.spec.strategy.rollingUpdate.maxSurge'
```

Expected:

```text
NAME        STRATEGY        MAX_UNAVAILABLE   MAX_SURGE
auth        RollingUpdate   0                 1
admin       RollingUpdate   0                 1
chat        RollingUpdate   0                 1
frontend    RollingUpdate   0                 1
streaming   RollingUpdate   0                 1
```

---

# 22. Scale Streaming Service

Scale the Streaming deployment manually:

```bash
kubectl scale deployment/streaming \
  -n streamingapp \
  --replicas=4
```

Verify:

```bash
kubectl get deployment streaming \
  -n streamingapp
```

Check the pods:

```bash
kubectl get pods \
  -n streamingapp \
  -l app=streaming
```

The deployment should show four replicas.

The Helm chart also contains an HPA for the streaming service.

Check it:

```bash
kubectl get hpa -n streamingapp
```

---

# 23. Upgrade Auth Image to 1.0.1

The Helm chart supports changing the image tag.

Run:

```bash
helm upgrade streamingapp ./streamingapp \
  -n streamingapp \
  --set services.auth.tag=1.0.1
```

Check the rollout:

```bash
kubectl rollout status deployment/auth \
  -n streamingapp
```

Verify the image:

```bash
kubectl get deployment auth \
  -n streamingapp \
  -o jsonpath='{.spec.template.spec.containers[0].image}'
```

Expected:

```text
218014315198.dkr.ecr.ap-south-1.amazonaws.com/streaming-auth:1.0.1
```

---

# 24. Verify Helm Release

Check the release:

```bash
helm status streamingapp \
  -n streamingapp
```

Check Helm history:

```bash
helm history streamingapp \
  -n streamingapp
```

---

# 25. Troubleshooting

## Ingress has no ADDRESS

Check:

```bash
kubectl describe ingress streamingapp-ingress \
  -n streamingapp
```

Check the AWS Load Balancer Controller:

```bash
kubectl get pods \
  -n kube-system \
  -l app.kubernetes.io/name=aws-load-balancer-controller
```

Check controller logs:

```bash
kubectl logs \
  -n kube-system \
  deployment/aws-load-balancer-controller
```

---

## Pods are not Running

Check:

```bash
kubectl get pods -n streamingapp
```

Then:

```bash
kubectl describe pod <POD_NAME> \
  -n streamingapp
```

Check events:

```bash
kubectl get events \
  -n streamingapp \
  --sort-by=.lastTimestamp
```

---

## ImagePullBackOff

Check the image configured in Helm:

```bash
helm get values streamingapp \
  -n streamingapp
```

Verify that the image exists in Amazon ECR.

---

## MongoDB Pod is Pending

Check:

```bash
kubectl describe pod mongo-0 \
  -n streamingapp
```

Check PVC:

```bash
kubectl get pvc -n streamingapp
```

Check PV:

```bash
kubectl get pv
```

---

# 26. Uninstall the Application

To remove the Helm release:

```bash
helm uninstall streamingapp \
  -n streamingapp
```

Verify:

```bash
kubectl get all -n streamingapp
```

If the namespace is no longer required:

```bash
kubectl delete namespace streamingapp
```

> Note: Persistent resources such as storage may require separate cleanup depending on the configured Kubernetes storage and reclaim policy.

---

# 27. Complete Deployment Verification

Run:

```bash
helm list -n streamingapp
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

The expected architecture is:

```text
                         Internet
                            |
                            v
                    imreading.xyz
                            |
                            v
                     Cloudflare DNS
                            |
                            v
              AWS Application Load Balancer
                            |
                            v
                   Kubernetes Ingress
                            |
          +-----------------+-----------------+
          |                 |                 |
          v                 v                 v
      Frontend           Backend             Chat
       Service           Services           Service
          |                 |                 |
          v                 v                 v
      Frontend        Auth / Streaming /    Chat Pods
        Pods           Admin Pods
                            |
                +-----------+-----------+
                |                       |
                v                       v
             MongoDB                  Amazon S3
             StatefulSet              Media Storage
```

The AWS Load Balancer Controller manages the AWS ALB based on the Kubernetes Ingress resource.

---

## Quick Deployment Commands

For a quick deployment after the EKS cluster and prerequisites are ready:

```bash
aws eks update-kubeconfig \
  --name streaming-app-cluster \
  --region ap-south-1
```

```bash
kubectl create namespace streamingapp
```

```bash
helm lint ./streamingapp
```

```bash
helm install streamingapp ./streamingapp \
  -n streamingapp
```

```bash
kubectl get pods -n streamingapp
```

```bash
kubectl get ingress -n streamingapp
```

Once the ALB is provisioned and DNS is configured, access:

```text
http://imreading.xyz
```

---

# 28. Useful Commands

### Helm

```bash
helm list -n streamingapp
helm status streamingapp -n streamingapp
helm history streamingapp -n streamingapp
helm get values streamingapp -n streamingapp
```

### Kubernetes

```bash
kubectl get pods -n streamingapp
kubectl get deployments -n streamingapp
kubectl get services -n streamingapp
kubectl get ingress -n streamingapp
kubectl get pvc -n streamingapp
kubectl get hpa -n streamingapp
```

### Logs

```bash
kubectl logs deployment/auth -n streamingapp
kubectl logs deployment/streaming -n streamingapp
kubectl logs deployment/admin -n streamingapp
kubectl logs deployment/chat -n streamingapp
kubectl logs deployment/frontend -n streamingapp
```

### Rollouts

```bash
kubectl rollout status deployment/auth -n streamingapp
kubectl rollout status deployment/streaming -n streamingapp
kubectl rollout status deployment/admin -n streamingapp
kubectl rollout status deployment/chat -n streamingapp
kubectl rollout status deployment/frontend -n streamingapp
```

---

# Conclusion

The StreamingApp Helm chart provides a repeatable way to deploy the complete multi-service application to Kubernetes.

The AWS Load Balancer Controller integrates the Kubernetes Ingress with an AWS Application Load Balancer, allowing users to access the application through:

```text
http://imreading.xyz
```

Helm can then be used to manage application versions, image tags, replica counts, autoscaling configuration, and rolling updates.
