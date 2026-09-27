# StreamingApp — Project Documentation

## 1. Project Overview

StreamingApp is a MERN-based multi-service application deployed using Docker, Amazon ECR, Jenkins CI/CD, Kubernetes, Amazon EKS, Helm, AWS Load Balancer Controller, Amazon S3, IRSA, HPA, CloudWatch, and Slack ChatOps.

### Services

| Service           |  Port |
| ----------------- | ----: |
| Frontend          |    80 |
| Auth Service      |  3001 |
| Streaming Service |  3002 |
| Admin Service     |  3003 |
| Chat Service      |  3004 |
| MongoDB           | 27017 |

AWS Region:

```bash
ap-south-1
```

---

# 2. Clone / Project Setup

```bash
git clone <Repo-link>

cd StreamingAppKubernetes_SeemaKr
```

Check project files:

```bash
ls
```

Check Git status:

```bash
git status
```

---

# 3. Local Application Validation

Check Docker:

```bash
docker --version
docker compose version
```

Start the application locally:

```bash
docker compose up -d
```

Check containers:

```bash
docker compose ps
```

Check logs:

```bash
docker compose logs
```

Test the frontend in the browser:

```text
http://localhost
```

Stop the application:

```bash
docker compose down
```

---

# 4. Docker Images

## 4.1 Build Auth Service

```bash
docker build \
  -f backend/authService/Dockerfile \
  -t streaming-auth:1.0.0 \
  ./backend
```

## 4.2 Build Streaming Service

```bash
docker build \
  -f backend/streamingService/Dockerfile \
  -t streaming-stream:1.0.0 \
  ./backend
```

## 4.3 Build Admin Service

```bash
docker build \
  -f backend/adminService/Dockerfile \
  -t streaming-admin:1.0.0 \
  ./backend
```

## 4.4 Build Chat Service

```bash
docker build \
  -f backend/chatService/Dockerfile \
  -t streaming-chat:1.0.0 \
  ./backend
```

## 4.5 Build Frontend

```bash
docker build \
  -f frontend/Dockerfile \
  -t streaming-frontend:1.0.0 \
  ./frontend
```

Check images:

```bash
docker images
```

---

# 5. Amazon ECR

AWS account:

```text
aws_accountID
```

Region:

```text
ap-south-1
```

Login to ECR:

```bash
aws ecr get-login-password \
  --region ap-south-1 |
docker login \
  --username AWS \
  --password-stdin \
  <aws_accountID>.dkr.ecr.ap-south-1.amazonaws.com
```

Create repositories:

```bash
aws ecr create-repository \
  --repository-name streaming-auth \
  --region ap-south-1

aws ecr create-repository \
  --repository-name streaming-stream \
  --region ap-south-1

aws ecr create-repository \
  --repository-name streaming-admin \
  --region ap-south-1

aws ecr create-repository \
  --repository-name streaming-chat \
  --region ap-south-1

aws ecr create-repository \
  --repository-name streaming-frontend \
  --region ap-south-1
```

Check repositories:

```bash
aws ecr describe-repositories \
  --region ap-south-1
```

---

# 6. Tag Docker Images for ECR

```bash
docker tag streaming-auth:1.0.0 \
<aws_accountID>.dkr.ecr.ap-south-1.amazonaws.com/streaming-auth:v1

docker tag streaming-stream:1.0.0 \
<aws_accountID>.dkr.ecr.ap-south-1.amazonaws.com/streaming-stream:v1

docker tag streaming-admin:1.0.0 \
<aws_accountID>.dkr.ecr.ap-south-1.amazonaws.com/streaming-admin:v1

docker tag streaming-chat:1.0.0 \
<aws_accountID>.dkr.ecr.ap-south-1.amazonaws.com/streaming-chat:v1

docker tag streaming-frontend:1.0.0 \
<aws_accountID>.dkr.ecr.ap-south-1.amazonaws.com/streaming-frontend:v1
```

Push images:

```bash
docker push \
<aws_accountID>.dkr.ecr.ap-south-1.amazonaws.com/streaming-auth:v1

docker push \
<aws_accountID>.dkr.ecr.ap-south-1.amazonaws.com/streaming-stream:v1

docker push \
<aws_accountID>.dkr.ecr.ap-south-1.amazonaws.com/streaming-admin:v1

docker push \
<aws_accountID>.dkr.ecr.ap-south-1.amazonaws.com/streaming-chat:v1

docker push \
<aws_accountID>.dkr.ecr.ap-south-1.amazonaws.com/streaming-frontend:v1
```

---

# 7. Jenkins CI/CD

Jenkins:

```text
https://jenkinsurl.com/
```

Pipeline job:

```text
StreamingApp-CI-CD_seemaKr
```

AWS credential:

```text
StreamingApp-CI-CD_seemaKr
```

Slack credential:

```text
streamingapp-slack-webhook
```

The Jenkins pipeline performs:

```text
Checkout
   ↓
Build Docker Images
   ↓
Login to AWS ECR
   ↓
Tag Images
   ↓
Push Images to ECR
   ↓
Slack Success/Failure Notification
```

Jenkins AWS authentication:

```groovy
withAWS(
    credentials: 'CredentialsID',
    region: 'ap-south-1'
) {
    sh '''
        aws sts get-caller-identity

        aws ecr get-login-password \
          --region "${AWS_REGION}" |
        docker login \
          --username AWS \
          --password-stdin \
          "${ECR_REGISTRY}"
    '''
}
```

Check Jenkinsfile:


cat [Jenkinsfile](https://github.com/KrSeema/StreamingAppKubernetes_SeemaKr/blob/main/Jenkinsfile)


After modifying Jenkinsfile:

```bash
git add Jenkinsfile
git commit -m "Update Jenkins CI/CD pipeline"
git push origin main
```

---

# 8. Slack ChatOps

Slack channel:

```text
#streamingapp-devops
```

Jenkins success notification:

```text
🚀 StreamingApp CI/CD SUCCESS
```

Jenkins failure notification:

```text
❌ StreamingApp CI/CD FAILED - Check Jenkins Console Output
```

Webhook is stored in Jenkins credentials and is not committed to GitHub.

---

# 9. Kubernetes Manifests

Check Kubernetes files:

```bash
ls k8s
```

Required Kubernetes resources:

```text
5 Deployments
5 Services
ConfigMap
Secret
MongoDB StatefulSet
PVC
Readiness Probes
Liveness Probes
RollingUpdate Strategy
```

Apply namespace:

```bash
kubectl apply -f k8s/namespace.yaml
```

Apply ConfigMap:

```bash
kubectl apply -f k8s/configmap.yaml
```

Apply Secret:

```bash
kubectl apply -f k8s/secret.yaml
```

Apply MongoDB:

```bash
kubectl apply -f k8s/mongo-statefulset.yaml
```

Apply application resources:

```bash
kubectl apply -f k8s/
```

Check resources:

```bash
kubectl get all -n streamingapp
```

---

# 10. Amazon EKS Cluster

AWS region:

```bash
ap-south-1
```

Create EKS cluster using eksctl:

```bash
eksctl create cluster \
  --name streaming-app-cluster \
  --region ap-south-1
```

Check cluster:

```bash
eksctl get cluster \
  --region ap-south-1
```

Update kubeconfig:

```bash
aws eks update-kubeconfig \
  --region ap-south-1 \
  --name streaming-app-cluster
```

Check Kubernetes context:

```bash
kubectl config current-context
```

Check nodes:

```bash
kubectl get nodes
```

Check detailed node information:

```bash
kubectl get nodes -o wide
```

---

# 11. Kubernetes Namespace

Create namespace:

```bash
kubectl create namespace streamingapp
```

Check namespace:

```bash
kubectl get namespaces
```

---

# 12. Amazon EBS CSI Driver

Check add-ons:

```bash
aws eks list-addons \
  --cluster-name streaming-app-cluster \
  --region ap-south-1
```

Create EBS CSI add-on:

```bash
aws eks create-addon \
  --cluster-name streaming-app-cluster \
  --addon-name aws-ebs-csi-driver \
  --region ap-south-1
```

Check status:

```bash
aws eks describe-addon \
  --cluster-name streaming-app-cluster \
  --addon-name aws-ebs-csi-driver \
  --region ap-south-1
```

Check CSI pods:

```bash
kubectl get pods \
  -n kube-system \
  -l app.kubernetes.io/name=aws-ebs-csi-driver
```

---

# 13. MongoDB StatefulSet and PVC

Check StatefulSet:

```bash
kubectl get statefulset \
  -n streamingapp
```

Check MongoDB pod:

```bash
kubectl get pods \
  -n streamingapp \
  -l app=mongo
```

Check PVC:

```bash
kubectl get pvc \
  -n streamingapp
```

Check PV:

```bash
kubectl get pv
```

Check MongoDB service:

```bash
kubectl get svc mongo \
  -n streamingapp
```

MongoDB connection string:

```text
mongodb://mongo:27017/streamingapp
```

---

# 14. Helm Chart

Check Helm:

```bash
helm version
```

Check chart:

```bash
ls streamingapp
```

Lint chart:

```bash
helm lint ./streamingapp
```

Render templates:

```bash
helm template \
  streamingapp \
  ./streamingapp \
  -n streamingapp
```

Save rendered output:

```bash
helm template \
  streamingapp \
  ./streamingapp \
  -n streamingapp \
  > rendered.yaml
```

Check Helm releases:

```bash
helm list \
  -n streamingapp
```

---

# 15. Helm Installation

Install the application:

```bash
helm install \
  streamingapp \
  ./streamingapp \
  -n streamingapp \
  --create-namespace
```

Check release:

```bash
helm status \
  streamingapp \
  -n streamingapp
```

Check resources:

```bash
kubectl get all \
  -n streamingapp
```

---

# 16. Helm Upgrade

Upgrade application:

```bash
helm upgrade \
  streamingapp \
  ./streamingapp \
  -n streamingapp \
  --force-conflicts
```

Check release:

```bash
helm status \
  streamingapp \
  -n streamingapp
```

---

# 17. AWS Load Balancer Controller

Create IAM policy:

```bash
aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json
```

Create IAM service account using eksctl:

```bash
eksctl create iamserviceaccount \
  --cluster streaming-app-cluster \
  --namespace kube-system \
  --name aws-load-balancer-controller \
  --attach-policy-arn arn:aws:iam::<aws_accountID>:policy/AWSLoadBalancerControllerIAMPolicy \
  --override-existing-serviceaccounts \
  --region ap-south-1 \
  --approve
```

Add Helm repository:

```bash
helm repo add eks \
  https://aws.github.io/eks-charts
```

Update Helm repositories:

```bash
helm repo update
```

Install controller:

```bash
helm install aws-load-balancer-controller \
  eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=streaming-app-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller
```

Check controller:

```bash
kubectl get pods \
  -n kube-system \
  -l app.kubernetes.io/name=aws-load-balancer-controller
```

---

# 18. Ingress

Check Ingress:

```bash
kubectl get ingress \
  -n streamingapp
```

Describe Ingress:

```bash
kubectl describe ingress \
  streamingapp-ingress \
  -n streamingapp
```

Get ALB DNS:

```bash
kubectl get ingress \
  streamingapp-ingress \
  -n streamingapp \
  -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

Ingress host:

```text
imreading.xyz
```

Ingress paths:

```text
/
/api
/api/streaming
/api/admin
/api/chat
```

Test domain:

```bash
curl -I http://imreading.xyz
```

Test ALB using Host header:

```bash
curl -I \
  -H "Host: imreading.xyz" \
  http://<ALB-DNS>
```

---

# 19. Cloudflare DNS

DNS configuration:

```text
imreading.xyz
        ↓
ALB DNS
        ↓
AWS Load Balancer
```

CNAME:

```text
imreading.xyz → ALB DNS
```

DNS mode:

```text
DNS Only
```

---

# 20. Application Login Test

Register user:

```bash
curl -i -X POST \
  http://imreading.xyz/api/register \
  -H "Content-Type: application/json" \
  -d '{
    "email":"seema.test@example.com",
    "password":"Test@12345"
  }'
```

Login:

```bash
curl -i -X POST \
  http://imreading.xyz/api/login \
  -H "Content-Type: application/json" \
  -d '{
    "email":"seema.test@example.com",
    "password":"Test@12345"
  }'
```

---

# 21. Streaming Test

Use HTTP Range request:

```bash
curl -i \
  -H "Range: bytes=0-1023" \
  http://imreading.xyz/api/streaming/stream/<VIDEO_ID> \
  -o /tmp/video-test
```

Check downloaded data:

```bash
ls -lh /tmp/video-test
```

---

# 22. Live Chat Test

Open the application in two browser tabs.

```text
Tab 1 → Login → Open video → Join chat
Tab 2 → Login → Open same video → Join chat
```

Send a message from Tab 1.

Verify that it appears in Tab 2.

---

# 23. Kubernetes Health Checks

Check pods:

```bash
kubectl get pods \
  -n streamingapp
```

Check deployments:

```bash
kubectl get deployments \
  -n streamingapp
```

Check services:

```bash
kubectl get services \
  -n streamingapp
```

Check StatefulSet:

```bash
kubectl get statefulset \
  -n streamingapp
```

Check events:

```bash
kubectl get events \
  -n streamingapp \
  --sort-by=.lastTimestamp
```

---

# 24. Readiness and Liveness Probes

Check deployment configuration:

```bash
kubectl describe deployment \
  streaming \
  -n streamingapp
```

Verify probes:

```bash
kubectl get deployment \
  streaming \
  -n streamingapp \
  -o yaml
```

The application deployments use:

```yaml
readinessProbe
livenessProbe
```

---

# 25. Rolling Update Strategy

Check deployment strategy:

```bash
kubectl get deployments \
  -n streamingapp
```

Verify one deployment:

```bash
kubectl get deployment streaming \
  -n streamingapp \
  -o yaml
```

Required strategy:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1
```

Check all deployments:

```bash
for d in admin auth chat frontend streaming
do
  kubectl get deployment "$d" \
    -n streamingapp \
    -o jsonpath="{.metadata.name}{' '}{.spec.strategy.type}{' '}{.spec.strategy.rollingUpdate.maxUnavailable}{' '}{.spec.strategy.rollingUpdate.maxSurge}{'\n'}"
done
```

---

# 26. Scaling Test

Scale Streaming Service to 4 replicas:

```bash
kubectl scale deployment/streaming \
  -n streamingapp \
  --replicas=4
```

Check pods:

```bash
kubectl get pods \
  -n streamingapp \
  -l app=streaming
```

Check deployment:

```bash
kubectl get deployment streaming \
  -n streamingapp
```

---

# 27. Horizontal Pod Autoscaler

Check HPA:

```bash
kubectl get hpa \
  -n streamingapp
```

Detailed HPA:

```bash
kubectl describe hpa \
  streaming \
  -n streamingapp
```

Example configuration:

```yaml
minReplicas: 1
maxReplicas: 4
targetCPUUtilizationPercentage: 70
```

---

# 28. Auth Service Version Upgrade

Update Auth image tag:

```bash
helm upgrade streamingapp \
  ./streamingapp \
  -n streamingapp \
  --set services.auth.tag=1.0.1 \
  --force-conflicts
```

Check rollout:

```bash
kubectl rollout status \
  deployment/auth \
  -n streamingapp
```

Check image:

```bash
kubectl get deployment auth \
  -n streamingapp \
  -o jsonpath='{.spec.template.spec.containers[0].image}'
```

Expected:

```text
<aws_accountID>.dkr.ecr.ap-south-1.amazonaws.com/streaming-auth:1.0.1
```

---

# 29. Rollout Testing

Check rollout status for all services:

```bash
kubectl rollout status deployment/auth -n streamingapp
kubectl rollout status deployment/streaming -n streamingapp
kubectl rollout status deployment/admin -n streamingapp
kubectl rollout status deployment/chat -n streamingapp
kubectl rollout status deployment/frontend -n streamingapp
```

Check rollout history:

```bash
kubectl rollout history deployment/auth \
  -n streamingapp
```

---

# 30. Self-Healing Test

Delete a running pod:

```bash
kubectl delete pod \
  <POD_NAME> \
  -n streamingapp
```

Watch pods:

```bash
kubectl get pods \
  -n streamingapp \
  -w
```

Verify that Kubernetes creates a replacement pod.

---

# 31. Amazon S3

Create S3 bucket:

```bash
aws s3 mb \
  s3://streamingapp-media-<aws_accountID> \
  --region ap-south-1
```

Check bucket:

```bash
aws s3 ls
```

Upload test object:

```bash
aws s3 cp \
  <file> \
  s3://streamingapp-media-<aws_accountID>/
```

List objects:

```bash
aws s3 ls \
  s3://streamingapp-media-<aws_accountID>/ \
  --recursive
```

---

# 32. IRSA for S3

IAM role:

```text
StreamingAppS3Role
```

ServiceAccount:

```text
streamingapp-s3
```

Check ServiceAccount:

```bash
kubectl get serviceaccount \
  streamingapp-s3 \
  -n streamingapp
```

Check annotation:

```bash
kubectl get serviceaccount \
  streamingapp-s3 \
  -n streamingapp \
  -o yaml
```

Expected annotation:

```yaml
eks.amazonaws.com/role-arn: arn:aws:iam::<aws_accountID>:role/StreamingAppS3Role
```

---

# 33. CloudWatch Monitoring

Check CloudWatch Observability add-on:

```bash
aws eks describe-addon \
  --cluster-name streaming-app-cluster \
  --addon-name amazon-cloudwatch-observability \
  --region ap-south-1
```

Check CloudWatch pods:

```bash
kubectl get pods \
  -n amazon-cloudwatch
```

Check CloudWatch log groups:

```bash
aws logs describe-log-groups \
  --region ap-south-1
```

Example application log group:

```text
/aws/containerinsights/streaming-app-cluster/application
```

---

# 34. CloudWatch CPU Alarms

List alarms:

```bash
aws cloudwatch describe-alarms \
  --region ap-south-1
```

Example alarms:

```text
StreamingApp-EKS-Node1-HighCPU
StreamingApp-EKS-Node2-HighCPU
```

Check alarm state:

```bash
aws cloudwatch describe-alarms \
  --alarm-names StreamingApp-EKS-Node1-HighCPU \
  --region ap-south-1
```

---

# 35. Application Logs

Check pod logs:

```bash
kubectl logs \
  <POD_NAME> \
  -n streamingapp
```

Follow logs:

```bash
kubectl logs \
  -f <POD_NAME> \
  -n streamingapp
```

Previous container logs:

```bash
kubectl logs \
  <POD_NAME> \
  -n streamingapp \
  --previous
```

---

# 36. Useful Kubernetes Commands

All resources:

```bash
kubectl get all \
  -n streamingapp
```

Pods:

```bash
kubectl get pods \
  -n streamingapp
```

Services:

```bash
kubectl get svc \
  -n streamingapp
```

Deployments:

```bash
kubectl get deploy \
  -n streamingapp
```

ConfigMaps:

```bash
kubectl get configmap \
  -n streamingapp
```

Secrets:

```bash
kubectl get secrets \
  -n streamingapp
```

Ingress:

```bash
kubectl get ingress \
  -n streamingapp
```

PVC:

```bash
kubectl get pvc \
  -n streamingapp
```

---

# 37. Useful Helm Commands

List releases:

```bash
helm list \
  -n streamingapp
```

Show values:

```bash
helm get values \
  streamingapp \
  -n streamingapp
```

Show manifest:

```bash
helm get manifest \
  streamingapp \
  -n streamingapp
```

Upgrade:

```bash
helm upgrade \
  streamingapp \
  ./streamingapp \
  -n streamingapp
```

Rollback:

```bash
helm rollback \
  streamingapp \
  <REVISION> \
  -n streamingapp
```

Uninstall:

```bash
helm uninstall \
  streamingapp \
  -n streamingapp
```

---

# 38. Final Application Validation

Check all pods:

```bash
kubectl get pods \
  -n streamingapp
```

Check all deployments:

```bash
kubectl get deployments \
  -n streamingapp
```

Check services:

```bash
kubectl get services \
  -n streamingapp
```

Check HPA:

```bash
kubectl get hpa \
  -n streamingapp
```

Check Ingress:

```bash
kubectl get ingress \
  -n streamingapp
```

Test application:

```bash
curl -I http://imreading.xyz
```

Test login:

```bash
curl -i -X POST \
  http://imreading.xyz/api/login \
  -H "Content-Type: application/json" \
  -d '{"email":"seema.test@example.com","password":"Test@12345"}'
```

---

# 39. AWS Resource Cleanup

After completing the assignment, delete the EKS cluster:

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

Expected:

```text
No clusters found
```

Check EC2 instances:

```bash
aws ec2 describe-instances \
  --region ap-south-1 \
  --filters \
  "Name=instance-state-name,Values=running,stopped,pending,stopping" \
  --query 'Reservations[].Instances[].InstanceId' \
  --output text
```

Check NAT Gateways:

```bash
aws ec2 describe-nat-gateways \
  --region ap-south-1 \
  --filter Name=state,Values=available,pending,deleting \
  --query 'NatGateways[].NatGatewayId' \
  --output text
```

Check Elastic IPs:

```bash
aws ec2 describe-addresses \
  --region ap-south-1 \
  --query 'Addresses[].AllocationId' \
  --output text
```

Check EBS volumes:

```bash
aws ec2 describe-volumes \
  --region ap-south-1 \
  --filters Name=status,Values=available \
  --query 'Volumes[].VolumeId' \
  --output text
```

---

# 40. Delete S3 Test Data / Bucket

List bucket:

```bash
aws s3 ls \
  s3://streamingapp-media-<aws_accountID>/ \
  --recursive
```

Delete objects:

```bash
aws s3 rm \
  s3://streamingapp-media-<aws_accountID>/ \
  --recursive
```

Delete bucket:

```bash
aws s3 rb \
  s3://streamingapp-media-<aws_accountID>
```

Verify:

```bash
aws s3api head-bucket \
  --bucket streamingapp-media-<aws_accountID>
```

---

# 41. Delete ECR Repositories

Check ECR repositories:

```bash
aws ecr describe-repositories \
  --region ap-south-1 \
  --query 'repositories[].repositoryName' \
  --output table
```

Delete project repositories:

```bash
for repo in streaming-admin streaming-auth streaming-chat streaming-frontend streaming-stream
do
  aws ecr delete-repository \
    --repository-name "$repo" \
    --region ap-south-1 \
    --force
done
```

Verify:

```bash
aws ecr describe-repositories \
  --region ap-south-1 \
  --output table
```

Expected:

```text
----------------------
|DescribeRepositories|
+--------------------+
```

---

# 42. Final AWS Cleanup Verification

Check EKS:

```bash
eksctl get cluster \
  --region ap-south-1
```

Check EC2:

```bash
aws ec2 describe-instances \
  --region ap-south-1 \
  --filters \
  "Name=instance-state-name,Values=running,stopped,pending,stopping" \
  --output table
```

Check NAT:

```bash
aws ec2 describe-nat-gateways \
  --region ap-south-1 \
  --filter Name=state,Values=available,pending,deleting \
  --output table
```

Check ECR:

```bash
aws ecr describe-repositories \
  --region ap-south-1 \
  --output table
```

Check S3:

```bash
aws s3 ls
```

---

# 43. Git Commit and Push Documentation

Check changed files:

```bash
git status
```

Add documentation:

```bash
git add Documentation.md
```

Commit:

```bash
git commit -m "Add complete project documentation"
```

Push:

```bash
git push origin main
```

---

# 44. Related Documentation

Detailed Helm installation and Ingress guide:

[Helm Installation and Ingress Guide](https://github.com/KrSeema/StreamingAppKubernetes_SeemaKr/blob/main/README_HelmInstall_Ingress.md)

Main project README:

[README.md](https://github.com/KrSeema/StreamingAppKubernetes_SeemaKr/blob/main/README.md)

---

# 45. Final Project Flow

```text
Developer
   |
   v
GitHub
   |
   v
Jenkins
   |
   +---- Build Docker Images
   |
   +---- Push Images
   |
   v
Amazon ECR
   |
   v
Amazon EKS
   |
   v
Helm
   |
   +---- Frontend
   +---- Auth
   +---- Streaming
   +---- Admin
   +---- Chat
   |
   +---- MongoDB StatefulSet
   |
   v
AWS Load Balancer Controller
   |
   v
Application Load Balancer
   |
   v
Cloudflare
   |
   v
imreading.xyz
```

Supporting AWS services:

```text
Amazon S3
   |
   +---- Videos
   +---- Thumbnails

IRSA
   |
   +---- Secure S3 access

CloudWatch
   |
   +---- Metrics
   +---- Logs
   +---- Alarms

Slack
   |
   +---- Jenkins CI/CD notifications
```

---

# 46. Project Completion

The project demonstrated:

* Docker containerization
* Multi-service application deployment
* Amazon ECR
* Jenkins CI/CD
* Slack ChatOps
* Kubernetes deployments and services
* MongoDB StatefulSet
* Persistent Volume
* Helm
* Amazon EKS
* AWS Load Balancer Controller
* Application Load Balancer
* Cloudflare DNS
* Amazon S3
* IAM Roles for Service Accounts
* Horizontal Pod Autoscaling
* Rolling Updates
* Readiness and Liveness Probes
* Kubernetes self-healing
* CloudWatch monitoring
* CloudWatch logging
* CloudWatch alarms
* Application smoke testing
* AWS resource cleanup


# 47. Production Best Practices

For a production Kubernetes cluster, separate workloads into dedicated namespaces such as application, monitoring, and platform namespaces, with appropriate RBAC policies and resource quotas to improve isolation and security. I would enable TLS for external traffic using HTTPS with certificates managed through AWS Certificate Manager or cert-manager, and enforce secure communication between services where required. For scaling. Use HPA with carefully tuned CPU and memory targets and minimum/maximum replica counts based on actual workload patterns, and consider cluster autoscaling so worker nodes can scale with demand. Also use production-grade observability, network policies, secrets management, backups, and multi-AZ workloads to improve reliability and security.