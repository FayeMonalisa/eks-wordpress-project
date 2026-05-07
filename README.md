#  EKS WordPress Deployment with Autoscaling

##  Project Overview
This project demonstrates deploying a scalable WordPress application on **Amazon EKS (Elastic Kubernetes Service)** using **Kubernetes, Helm, and AWS tools**. The application is exposed via a LoadBalancer and configured with **Horizontal Pod Autoscaling (HPA)** based on CPU utilization.

---

##  Tech Stack
- AWS EKS – Managed Kubernetes cluster  
- EC2 (Ubuntu) – Deployment environment  
- kubectl – Kubernetes CLI  
- eksctl – EKS cluster management  
- Helm – Kubernetes package manager  
- Docker (WordPress image) – Application container  
- Git & GitHub – Version control  
- Metrics Server – Resource monitoring  
- HPA (Horizontal Pod Autoscaler) – Autoscaling  
- Siege – Load testing tool  

---

##  Architecture

            User
              │
              ▼
    AWS LoadBalancer
              │
              ▼
    Kubernetes Service
              │
              ▼
     Pods (WordPress)
              │
              ▼
    HPA (Auto Scaling)
              │
              ▼
 Metrics Server (CPU Monitoring)

---

## Setup & Deployment Steps

### 1. Clone Repository
```bash
git clone https://github.com/FayeMonalisa/eks-wordpress-project.git
cd eks-wordpress-project
```

### 2. Install Required Tools
```bash
aws --version
eksctl version
kubectl version --client
helm version
```

### 3. Create EKS Cluster
```bash
eksctl create cluster \
--name enterprise-cluster \
--region us-east-2 \
--nodegroup-name standard-workers \
--node-type t3.medium \
--nodes 3 \
--nodes-min 3 \
--nodes-max 6 \
--managed
```

### 4. Configure kubectl
```bash
aws eks --region us-east-2 update-kubeconfig --name enterprise-cluster
kubectl get nodes
```
### 5. Deploy WordPress with Helm
``` bash
helm create my-microservice
helm install my-microservice ./my-microservice
```

### 6. Expose Application (LoadBalancer)
```bash
kubectl get svc
```
Access the application:
http://<EXTERNAL-LOADBALANCER-URL>

### 7. Install Metrics Server
```
aws eks update-addon \
  --cluster-name enterprise-cluster \
  --addon-name metrics-server \
  --region us-east-2 \
  --resolve-conflicts OVERWRITE
```
Verify:
```
kubectl top nodes
kubectl top pods
```
### 8. Configure Horizontal Pod Autoscaler
```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: wordpress-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-microservice
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```
Apply:
```
kubectl apply -f hpa.yaml
```
### 9. Load Testing with Siege
```bash
kubectl run siege --image=jstarcher/siege --restart=Never \
-- /bin/sh -c "siege -c 100 -t 3m http://<LOADBALANCER-URL>"
```
Monitor scaling:
```
kubectl get hpa -w
kubectl get pods -w
```
Results
- Successfully deployed WordPress on EKS
- Exposed application via AWS LoadBalancer
- Enabled autoscaling using HPA
- Verified CPU-based scaling logic
- Troubleshot and resolved:
    * Metrics Server issues
    * YAML configuration errors
    * Resource request misconfigurations

Key Learnings
- Importance of resource requests for HPA functionality
- Debugging Kubernetes metrics pipeline
- Managing infrastructure via CLI tools
- Real-world DevOps troubleshooting

Cleanup (Cost Optimization)
```
eksctl delete cluster --name enterprise-cluster --region us-east-2
```

Then terminate EC2 instance from AWS Console.
