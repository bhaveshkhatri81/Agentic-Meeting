

# 🚀 Deployment Steps

This project demonstrates how to deploy a **Streamlit** web app using **Docker**, **GitHub Actions**, **Amazon ECR**, and a **self-hosted runner on AWS EC2**. It uses `crewai`, `langchain`, and other modern libraries.

---

## 📁 Project Structure

streamlit-app/
├── app.py
├── requirements.txt
├── Dockerfile
├── .dockerignore
├── scripts/
│   └── deploy_container.sh
├── nginx/
│   └── app.conf
└── .github/
    └── workflows/
        └── deploy.yml

---

## 🔧 Prerequisites

- AWS account
- EC2 instance (Ubuntu 20.04)
- ECR repository (`streamlit-app`)
- GitHub repo
- GitHub Actions self-hosted runner (on EC2)
- Docker installed on EC2
- IAM user with ECR permissions

---

## 🐳 Dockerfile (Python 3.11 Compatible)

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --upgrade pip
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8501

HEALTHCHECK CMD curl --fail http://localhost:8501/_stcore/health || exit 1

ENTRYPOINT ["streamlit", "run", "app.py", "--server.port=8501", "--server.address=0.0.0.0"]
```
---
## 📦 requirements.txt
```
streamlit>=1.22.0
crewai>=0.28.5
langchain>=0.0.267
langchain-openai>=0.0.2
langchain-community>=0.0.13
faiss-cpu>=1.7.4
pandas>=2.0.0
openai>=1.1.1
python-dotenv>=1.0.0
pypdf>=3.8.1
```

---

## 🔁 GitHub Actions Workflow
.github/workflows/deploy.yml

```
name: Deploy Streamlit App

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: self-hosted

    steps:
    - name: Checkout Code
      uses: actions/checkout@v2

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ap-south-1

    - name: Login to Amazon ECR
      uses: aws-actions/amazon-ecr-login@v1

    - name: Build & Push Docker Image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        ECR_REPOSITORY: ${{ secrets.ECR_REPOSITORY }}
        IMAGE_TAG: ${{ github.sha }}
      run: |
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
        docker tag $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG $ECR_REGISTRY/$ECR_REPOSITORY:latest
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

    - name: Run Deployment Script
      run: ./scripts/deploy_container.sh

```
---
## 🔐 GitHub Secrets

Set these in Settings → Secrets → Actions:
```
Secret Name | Description
AWS_ACCESS_KEY_ID | IAM user access key
AWS_SECRET_ACCESS_KEY | IAM user secret key
ECR_REPOSITORY | ECR repo name (e.g., streamlit-app)
EC2_HOST | Public IP/DNS of EC2 instance
EC2_SSH_KEY | Contents of your .pem private key

```
---
## 🖥️ Self-Hosted Runner on EC2

```
mkdir actions-runner && cd actions-runner
curl -o actions-runner-linux-x64.tar.gz -L <GitHub link>
tar xzf actions-runner-linux-x64.tar.gz
./config.sh --url https://github.com/<your-user>/<your-repo> --token <token>
sudo ./svc.sh install
sudo ./svc.sh start

```
---

## 📜 deploy_container.sh Script Highlights

- Installs Docker & AWS CLI

- Logs into Amazon ECR

- Pulls and runs Docker image

- Configures Nginx if not already set

✅ Full script is in scripts/deploy_container.sh

---

## 🌐 Access the Application

```
Type | URL
Direct | http://<EC2-IP>:8501
Via Nginx | http://<EC2-IP>/

```

---

## 🧠 Architecture

```
Developer → GitHub → GitHub Actions → Amazon ECR → EC2 Runner → Docker Container → Nginx → Browser

```

---

## ✅ Quick Deployment

```
git add .
git commit -m "Deploy Streamlit App"
git push origin main

```
## 🧪 Verify Deployment
```
# On EC2:
docker ps
sudo nginx -t
curl http://localhost:8501

```
