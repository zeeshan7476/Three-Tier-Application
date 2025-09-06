# #Three Tier Application

# 🧱 Three-Tier Application with React, Node.js, MongoDB, Docker, Kubernetes, and AWS

This project contains a complete **Three-Tier Application** with the following architecture:

- **Presentation Layer**: React.js  
- **Business Logic Layer**: Node.js (Express)  
- **Data Layer**: MongoDB  

The project is containerized with **Docker**, deployed to **AWS ECR**, and orchestrated using **Kubernetes**. It uses **Ingress Controller** for internal routing and **AWS Application Load Balancer (ALB)** for external access. **Helm charts** are used for installing the ingress controller.

---

## 🧰 Tech Stack

| Layer             | Technology                        |
|------------------|-----------------------------------|
| Frontend         | React.js                          |
| Backend          | Node.js (Express)                 |
| Database         | MongoDB                           |
| Containerization | Docker                            |
| Orchestration    | Kubernetes                        |
| Cloud Registry   | AWS ECR                           |
| Ingress Routing  | NGINX Ingress Controller, ALB     |
| Helm Charts      | For ingress controller deployment |
| Version Control  | GitHub                            |


## 📌 Architecture Diagram
```plaintext

                 ┌────────────────────┐
                 │     GitHub Repo    │
                 └────────┬───────────┘
                          │
                          ▼
               Clone Source Code (React, Node.js, MongoDB)
                          │
                          ▼
 ┌────────────────────────────────────────────┐
 │       Build Docker Images (DF = Dockerfile)│
 │  React ➝ Dockerfile ➝ Image ➝ AWS ECR       │
 │  Node.js ➝ Dockerfile ➝ Image ➝ AWS ECR     │
 │  MongoDB ➝ Dockerfile ➝ Image ➝ AWS ECR     │
 └────────────────────────────────────────────┘
                          │
                          ▼
              Push Docker Images to AWS ECR
                          │
                          ▼
           Deploy Images in Kubernetes Cluster (EKS)
                          │
                          ▼
           ┌──────────────┴──────────────┐
           ▼                             ▼
┌────────────────────┐      ┌────────────────────┐
│  Frontend Pod      │      │  Backend Pod        │
└────────────────────┘      └────────────────────┘
           ▼                             ▲
           ▼                             │
 ┌────────────────────┐     ┌────────────────────┐
 │ Ingress Controller │────▶│  MongoDB Pod        │
 └────────────────────┘     └────────────────────┘
                          │
                          ▼
               AWS ALB Ingress Controller
                          │
                          ▼
                     🌐 External User

```

> 🧠 **Note**: Ingress Controller handles internal routing between services. ALB (Application Load Balancer) exposes the app to external users securely.

---

## 📂 Folder Structure

```plaintext
three-tier-app/
├── client/                   # React frontend
│   └── Dockerfile
├── server/                   # Node.js backend
│   └── Dockerfile
├── database/                 # MongoDB setup (optional)
│   └── Dockerfile
├── K8s-Manifests-file/                      # Kubernetes YAML files
│   ├── client-deployment.yaml
│   ├── server-deployment.yaml
│   ├── mongo-deployment.yaml
│   ├── ingress.yaml                     
├── README.md

```

## 🚀 Project Workflow

### 1.Clone the Repository
``` shell
 git clone https://github.com/your-username/three-tier-app.git
 cd three-tier-app
``` 
### 2.Build Docker Images
 # Frontend (React)
 ``` shell
 cd Frontend
 docker build -t your-ecr-url/client .
``` 
# Backend (Node.js)
``` shell
cd ../Backend
docker build -t your-ecr-url/server .
```
# MongoDB (optional if not using managed DB)
``` shell
cd ../database
docker build -t your-ecr-url/mongo .
```
### 3.Push Docker Images to AWS ECR
``` shell
aws ecr get-login-password --region <your-region> | docker login --username AWS --password-stdin <your-ecr-url>

docker push <your-ecr-url>/client
docker push <your-ecr-url>/server
docker push <your-ecr-url>/mongo
```
### 4.Create a Kubernetes Cluster
``` shell
eksctl create cluster --name three-tier-cluster --region <your-region> --nodes 3
```
### 5.Install Ingress Controller via Helm
``` shell
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx
```
### 6.Deploy Application to Kubernetes
``` shell
kubectl apply -f k8s/mongo-deployment.yaml
kubectl apply -f k8s/server-deployment.yaml
kubectl apply -f k8s/client-deployment.yaml
kubectl apply -f k8s/ingress.yaml
```
### 7.Configure External Access with AWS ALB Ingress Controller

Install ALB Ingress Controller (via Helm or YAML)

Annotate the ingress resources with ALB-specific configurations

Ensure proper IAM roles and permissions are attached to your cluster


## 🌐 Application Access

Once the Ingress Controller and ALB are configured correctly, access your application via:
http://<your-alb-dns-name>

Or configure DNS to point to the ALB.

## 📦 Environment Variables

Example .env for backend:
MONGO_URI=mongodb://mongo-service:27017/mydb
PORT=5000

---

## 🚀 Deployment Guide (Step by Step)
### Step 1: IAM Configuration

Create a user eks-admin with AdministratorAccess.
Generate Access Key and Secret Access Key.

### Step 2: EC2 Setup

Launch Ubuntu instance in us-west-2 (or your region).
SSH into the instance.

### Step 3: Install AWS CLI v2
``` shell
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update
aws configure
```
### Step 4: Install Docker
``` shell
sudo apt-get update
sudo apt install docker.io
docker ps
sudo chown $USER /var/run/docker.sock
```
### Step 5: Install kubectl
``` shell
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```
### Step 6: Install eksctl
``` shell
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```
### Step 7: Setup EKS Cluster
``` shell
eksctl create cluster --name three-tier-cluster --region us-west-2 --node-type t2.medium --nodes-min 2 --nodes-max 2
aws eks update-kubeconfig --region us-west-2 --name three-tier-cluster
kubectl get nodes
```
### Step 8: Run Manifests
``` shell
kubectl create namespace workshop
kubectl apply -f .
kubectl delete -f .
```
### Step 9: Install AWS Load Balancer
``` shell
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
eksctl utils associate-iam-oidc-provider --region=us-west-2 --cluster=three-tier-cluster --approve
eksctl create iamserviceaccount --cluster=three-tier-cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::626072240565:policy/AWSLoadBalancerControllerIAMPolicy --approve --region=us-west-2
```
### Step 10: Deploy AWS Load Balancer Controller
``` shell
sudo snap install helm --classic
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=my-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
kubectl get deployment -n kube-system aws-load-balancer-controller
kubectl apply -f full_stack_lb.yaml
kubectl get deployment -n kube-system aws-load-balancer-controller
kubectl apply -f full_stack_lb.yaml
```
## 🧹 Cleanup
``` shell
eksctl delete cluster --name three-tier-cluster --region us-west-2
```
Terminate the EC2 instance from step 2.

Delete the Load Balancer from steps 9–10.

Delete Security Groups in EC2 console.


## 📁 Future Improvements

CI/CD with GitHub Actions.

Monitoring (Prometheus + Grafana).

Managed MongoDB (Atlas).

SSL/TLS via cert-manager.

## 📝 License

MIT License












