# Kubernetes End-to-End Project on AWS EKS

## 🎯 Project Overview 

Deploy a web application on **Amazon EKS** with **Ingress Controller** for external access.

**Architecture:** App → Docker → EKS → Ingress → ALB → Internet

## 📸 Project Result

<img width="1009" height="875" alt="Image" src="https://github.com/user-attachments/assets/a75816be-2183-4c24-8dc5-995a97f0dcec" />

_Successfully deployed application accessible via AWS Load Balancer with Kubernetes orchestration_

---

## 🛠️ Prerequisites

```bash
# Install required tools
brew install awscli kubectl eksctl helm
```

---

## 🚀 Step 1: Create EKS Cluster

```bash
# Create cluster
eksctl create cluster \
  --name my-eks-cluster \
  --region us-east-1 \
  --node-type t3.medium \
  --nodes 2 \
  --with-oidc \
  --managed

# Update kubeconfig
aws eks update-kubeconfig --region us-east-1 --name my-eks-cluster
```

---

## 📦 Step 2: Deploy Application

```yaml
# app.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: react-app
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
---
apiVersion: v1
kind: Service
metadata:
  name: react-app-service
spec:
  selector:
    app: react-app
  ports:
    - port: 80
      targetPort: 80
  type: ClusterIP
```

```bash
kubectl apply -f app.yaml
```

---

## 🌐 Step 3: Setup Ingress Controller

```bash
# Create IAM policy
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json

# Create service account
eksctl create iamserviceaccount \
  --cluster=my-eks-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --attach-policy-arn=arn:aws:iam::ACCOUNT-ID:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve

# Install controller
helm repo add eks https://aws.github.io/eks-charts
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=my-eks-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller
```

---

## 🔗 Step 4: Create Ingress

```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: react-app-ingress
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
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
kubectl apply -f ingress.yaml

# Get ALB URL (wait 2-3 minutes)
kubectl get ingress react-app-ingress
```

---

## 🧪 Testing

```bash
# Check deployment
kubectl get pods
kubectl get services
kubectl get ingress

# Get application URL
ALB_URL=$(kubectl get ingress react-app-ingress -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
echo "App URL: http://$ALB_URL"

# Test
curl http://$ALB_URL
```

---

## 🆘 Troubleshooting

| Issue             | Solution                                                  |
| ----------------- | --------------------------------------------------------- |
| **Pods pending**  | `kubectl describe nodes` - check capacity                 |
| **Ingress no IP** | `kubectl get pods -n kube-system` - verify ALB controller |
| **503 errors**    | `kubectl get endpoints` - check service endpoints         |

---

## 🧹 Cleanup

```bash
# Delete resources
kubectl delete ingress react-app-ingress
kubectl delete -f app.yaml
helm uninstall aws-load-balancer-controller -n kube-system

# Delete cluster
eksctl delete cluster --name my-eks-cluster
```

---

## 🎯 Architecture

```
Internet → ALB → EKS Nodes → Pods
```

**🚀 Result: Scalable web application accessible via AWS Load Balancer!**

happy leaning