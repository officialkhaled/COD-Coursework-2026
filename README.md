# Container Orchestration for Dev-Ops | Coursework

# 🚀 DevOps CI/CD Pipeline with Docker, Jenkins & Kubernetes (AWS EC2)

## 📌 Overview

This project demonstrates a complete **CI/CD pipeline** using:

* GitHub (Version Control)
* Jenkins (CI/CD Automation)
* Docker (Containerisation)
* Kubernetes - Minikube (Orchestration)
* AWS EC2 (Infrastructure)

The application is a simple **portfolio website served via Nginx**, containerised using Docker and deployed using Kubernetes.

---

## 🏗️ Architecture

The system uses **two AWS EC2 instances**:

### 🔹 1. Jenkins Instance

* Runs Jenkins server
* Builds Docker image
* Pushes image to Docker Hub
* Deploys application to Kubernetes (via SSH)

### 🔹 2. Worker Instance (Minikube)

* Runs Docker
* Hosts Kubernetes cluster (Minikube)
* Deploys and runs application containers

### 🔄 Workflow

```
GitHub → Jenkins → Docker Hub → Worker EC2 (Kubernetes)
```

---

## ⚙️ Prerequisites

Before starting, ensure:

* AWS account
* Docker Hub account
* GitHub repository created
* SSH key pair for EC2

---

# ☁️ Step 1: Setup AWS EC2 Instances

Create **2 EC2 instances (Amazon Linux)**:

### Jenkins Instance

* Install:

  * Java
  * Jenkins
  * Git
  * Docker

### Worker Instance

* Install:

  * Docker
  * Minikube
  * kubectl
  * Git

---

# 🔧 Step 2: Setup Worker EC2 (Kubernetes)

SSH into Worker EC2:

```bash
ssh ec2-user@<WORKER_PUBLIC_IP>
```

Install Docker:

```bash
sudo yum install docker -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ec2-user
newgrp docker
```

Install Minikube:

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

Start Minikube:

```bash
minikube start --driver=docker
```

Install kubectl:

```bash
curl -LO "https://dl.k8s.io/release/latest/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

Verify:

```bash
kubectl get nodes
```

---

# 🧰 Step 3: Setup Jenkins EC2

SSH into Jenkins EC2:

```bash
ssh ec2-user@<JENKINS_PUBLIC_IP>
```

Install Java:

```bash
sudo yum install java-17-amazon-corretto -y
```

Install Jenkins:

```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo \
https://pkg.jenkins.io/redhat-stable/jenkins.repo

sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum install jenkins -y
```

Start Jenkins:

```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

Install Docker:

```bash
sudo yum install docker -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker jenkins
```

Restart Jenkins:

```bash
sudo systemctl restart jenkins
```

---

# 🔐 Step 4: Configure SSH (Jenkins → Worker)

On Jenkins EC2:

```bash
sudo mkdir -p /var/lib/jenkins/.ssh
sudo ssh-keygen -t rsa -b 4096 -f /var/lib/jenkins/.ssh/id_rsa
```

Copy public key:

```bash
sudo cat /var/lib/jenkins/.ssh/id_rsa.pub
```

On Worker EC2:

```bash
nano ~/.ssh/authorized_keys
```

Paste key and set permissions:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

---

# 📦 Step 5: Clone Repository (Worker EC2)

```bash
git clone https://github.com/<YOUR_USERNAME>/<YOUR_REPO>.git
cd <YOUR_REPO>
```

---

# 🐳 Step 6: Docker Setup

Build image manually (test):

```bash
docker build -t your-dockerhub-username/portfolio-app:v1 .
```

Run container:

```bash
docker run -d -p 8081:80 your-dockerhub-username/portfolio-app:v1
```

---

# ☸️ Step 7: Kubernetes Deployment

Apply configs:

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

Check:

```bash
kubectl get pods
kubectl get svc
```

---

# ⚙️ Step 8: Jenkins Pipeline Setup

### Create Pipeline Job

* New Item → Pipeline
* Select:

  * Pipeline script from SCM
  * Git repository URL
  * Branch: `main`
  * Script path: `Jenkinsfile`

---

### Add Credentials (Docker Hub)

```
Manage Jenkins → Credentials → Add
```

* ID: `dockerhub-creds`
* Username: Docker Hub username
* Password: Docker Hub password/token

---

# 🔁 Step 9: Enable GitHub Webhook

### In Jenkins:

✔ Enable:

```
GitHub hook trigger for GITScm polling
```

### In GitHub:

```
Settings → Webhooks → Add Webhook
```

* Payload URL:

```
http://<JENKINS_IP>:8080/github-webhook/
```

* Content type:

```
application/json
```

* Event:

```
Push
```

---

# 🚀 Step 10: Run the Pipeline

Push code to `main` branch:

```bash
git add .
git commit -m "update"
git push origin main
```

👉 This will automatically:

* Trigger Jenkins
* Build Docker image
* Push to Docker Hub
* Deploy to Kubernetes

---

# 🌐 Step 11: Access the Application

## Option 1: Docker (Recommended for browser)

```bash
docker run -d -p 80:80 your-dockerhub-username/portfolio-app:latest
```

Open:

```
http://<WORKER_PUBLIC_IP>
```

---

## Option 2: Kubernetes (NodePort)

```bash
kubectl port-forward --address 0.0.0.0 service/portfolio-service 30007:80
```

Open:

```
http://<WORKER_PUBLIC_IP>:30007
```

---

# 📊 Verification Commands

```bash
kubectl get pods
kubectl get services
kubectl get deployment
docker ps
```

---

# ⚠️ Notes

* Minikube on EC2 has networking limitations
* NodePort may not be directly accessible externally
* Docker port mapping is more stable for demo

---

# 🎯 Features

* CI/CD pipeline automation
* Docker containerisation
* Kubernetes deployment (2 replicas, rolling updates)
* Secure SSH-based deployment
* Webhook-triggered builds

---

# 👨‍💻 Author

Khaled Hossain

---

# 📄 License

For academic use only
