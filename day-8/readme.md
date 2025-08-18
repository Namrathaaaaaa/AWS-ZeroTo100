# Kubernetes End-to-End Project on AWS EKS

## 🎯 Project Overview

Deploy a complete web application on **Amazon EKS** (Elastic Kubernetes Service) with **Ingress Controller** for external access. This project demonstrates containerized application deployment, Kubernetes orchestration, and cloud-native networking.

**Architecture:** React App → Docker → EKS Cluster → Ingress → Load Balancer → Internet

---

## 🛠️ Prerequisites

```bash
# Required tools
aws-cli      # AWS command line
kubectl      # Kubernetes CLI
eksctl       # EKS cluster management
docker       # Container runtime
helm         # Kubernetes package manager
```

**AWS Requirements:**
- AWS Account with EKS permissions
- IAM user with appropriate policies
- VPC with public/private subnets

---

## 🚀 Step 1: EKS Cluster Setup

### Install eksctl
```bash
# Install eksctl (macOS)
brew tap weaveworks/tap
brew install weaveworks/tap/eksctl

# Verify installation
eksctl version
```

### Create EKS Cluster
```bash
# Create cluster with worker nodes
eksctl create cluster \
  --name my-eks-cluster \
  --region us-east-1 \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 4 \
  --with-oidc \
  --ssh-access \
  --ssh-public-key my-key \
  --managed

# Update kubeconfig
aws eks update-kubeconfig --region us-east-1 --name my-eks-cluster

# Verify cluster
kubectl get nodes
```

---

## 📦 Step 2: Application Deployment

### Sample React App Deployment
```yaml
# app-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: react-app
  labels:
    app: react-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: react-app
  template:
    metadata:
      labels:
        app: react-app
    spec:
      containers:
      - name: react-app
        image: nginx:alpine
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
---
apiVersion: v1
kind: Service
metadata:
  name: react-app-service
spec:
  selector:
    app: react-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

```bash
# Deploy application
kubectl apply -f app-deployment.yaml

# Verify deployment
kubectl get pods
kubectl get services
```

---

## 🌐 Step 3: Ingress Controller Setup

### Install AWS Load Balancer Controller
```bash
# Download IAM policy
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json

# Create IAM policy
aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json

# Create IAM service account
eksctl create iamserviceaccount \
  --cluster=my-eks-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::ACCOUNT-ID:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve

# Install controller via Helm
helm repo add eks https://aws.github.io/eks-charts
helm repo update

helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=my-eks-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller

# Verify installation
kubectl get deployment -n kube-system aws-load-balancer-controller
```

---

## 🔗 Step 4: Ingress Configuration

### Create Ingress Resource
```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: react-app-ingress
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/healthcheck-path: /
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}]'
spec:
  ingressClassName: alb
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: react-app-service
            port:
              number: 80
```

```bash
# Apply ingress
kubectl apply -f ingress.yaml

# Get ingress details
kubectl get ingress
kubectl describe ingress react-app-ingress

# Get ALB URL (takes 2-3 minutes)
kubectl get ingress react-app-ingress -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

---

## 🧪 Step 5: Testing & Verification

### Check Application Access
```bash
# Get ALB endpoint
ALB_URL=$(kubectl get ingress react-app-ingress -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
echo "Application URL: http://$ALB_URL"

# Test connectivity
curl -I http://$ALB_URL

# Check pod status
kubectl get pods -o wide
kubectl logs -l app=react-app

# Verify ingress controller
kubectl get pods -n kube-system -l app.kubernetes.io/name=aws-load-balancer-controller
```

### Application Scaling
```bash
# Scale application
kubectl scale deployment react-app --replicas=5

# Check auto-scaling
kubectl get hpa
kubectl top nodes
kubectl top pods
```

---

## 📊 Monitoring & Troubleshooting

### Essential Commands
```bash
# Cluster info
kubectl cluster-info
eksctl get cluster

# Resource monitoring
kubectl get all
kubectl describe nodes
kubectl get events --sort-by=.metadata.creationTimestamp

# Logs
kubectl logs -f deployment/react-app
kubectl logs -n kube-system -l app.kubernetes.io/name=aws-load-balancer-controller

# Troubleshooting ingress
kubectl describe ingress react-app-ingress
aws elbv2 describe-load-balancers
```

### Common Issues
| Issue | Solution |
|-------|----------|
| **Pods pending** | Check node capacity: `kubectl describe nodes` |
| **Ingress no IP** | Verify ALB controller: `kubectl get pods -n kube-system` |
| **503 errors** | Check service endpoints: `kubectl get endpoints` |
| **Timeout** | Verify security groups and target groups |

---

## 🔐 Security Best Practices

### Network Security
```bash
# Update security groups for ALB
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxxxxxx \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Enable pod security standards
kubectl label namespace default pod-security.kubernetes.io/enforce=baseline
```

### RBAC Configuration
```yaml
# rbac.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-reader
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-reader-binding
subjects:
- kind: User
  name: developer
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: app-reader
  apiGroup: rbac.authorization.k8s.io
```

---

## 💰 Cost Optimization

### Cluster Management
```bash
# Use Spot instances for cost savings
eksctl create cluster \
  --name spot-cluster \
  --spot \
  --instance-types=t3.medium,t3.large

# Auto-scaling configuration
kubectl apply -f https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml

# Monitor costs
aws ce get-cost-and-usage \
  --time-period Start=2025-08-01,End=2025-08-18 \
  --granularity MONTHLY \
  --metrics BlendedCost
```

---

## 🧹 Cleanup

```bash
# Delete ingress (removes ALB)
kubectl delete ingress react-app-ingress

# Delete application
kubectl delete -f app-deployment.yaml

# Delete ALB controller
helm uninstall aws-load-balancer-controller -n kube-system

# Delete cluster (THIS WILL DELETE EVERYTHING)
eksctl delete cluster --name my-eks-cluster --region us-east-1
```

---

## 🎯 Project Architecture

```
Internet → ALB (Ingress) → EKS Worker Nodes → Pods
                     ↓
                Target Groups → Service → Deployment
```

**Components:**
- **EKS Cluster**: Managed Kubernetes control plane
- **Worker Nodes**: EC2 instances running containers
- **ALB**: Application Load Balancer for external access
- **Ingress Controller**: Routes traffic to services
- **Services**: Internal load balancing to pods
- **Deployments**: Manages pod replicas and updates

---

## 📖 Key Learnings

**✅ Achieved:**
- Scalable containerized application deployment
- Cloud-native load balancing with ALB
- Kubernetes networking and service mesh
- Infrastructure as Code with YAML manifests
- Production-ready monitoring and logging

**🚀 Next Steps:**
- Add HTTPS/TLS certificates
- Implement CI/CD pipeline
- Set up monitoring with Prometheus/Grafana
- Add horizontal pod autoscaling
- Implement blue-green deployments

---

**🎉 Success! Your application is now running on EKS with external access through Ingress!**