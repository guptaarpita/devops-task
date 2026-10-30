# 🐳 Dockerized Web Application on AWS (EC2 & ECS Fargate)

**Stack:** `Docker` · `Nginx` · `AWS EC2` · `ECS Fargate` · `Amazon ECR` · `Ubuntu`

## 📌 Project Overview

This project demonstrates how to containerize a static web application with **Docker** and **Nginx**, then deploy the same image to AWS in two different ways:

1. **Amazon EC2** – run the container directly on an Ubuntu virtual machine.
2. **Amazon ECS on Fargate** – run the container serverlessly, pulling the image from a private **Amazon ECR** registry.



**How to read the diagram**

| Stage | Left path (EC2) | Right path (ECS Fargate) |
|-------|-----------------|--------------------------|
| Source | Code pushed to a public GitHub repository | Code built into `task-app:1.0` |
| Image | Image built/run on the EC2 host | Image pushed to Amazon ECR (private) |
| Compute | EC2 Ubuntu `t3.micro` (1 GiB RAM) | ECS Cluster → Fargate Task (0.25 vCPU, 0.5 GB) |
| Runtime | Docker container (Nginx, port 8081) | ECS Service → running Fargate task (Nginx, port 8081) |
| Network | EC2 Security Group allows TCP 8081 | ECS Security Group allows TCP 8081 |
| Result | `http://<EC2_PUBLIC_IP>:8081` | `http://<FARGATE_PUBLIC_IP>:8081` |

## 🧰 Technologies

- **AWS EC2** – virtual machine hosting
- **AWS ECR** – private Docker image registry
- **AWS ECS + Fargate** – serverless container orchestration
- **Ubuntu Linux** – host operating system
- **Docker** – containerization
- **Nginx** – web server
- **Git & GitHub** – version control and source hosting

## 📁 Project Structure

```
.
├── Dockerfile      # Docker image configuration
├── index.html      # Web application
├── nginx.conf      # Nginx server configuration (listens on 8081)
└── README.md
```

## ⚙️ Infrastructure Summary

| Setting | EC2 | ECS Fargate |
|---------|-----|-------------|
| Compute | `t3.micro` | 0.25 vCPU |
| Memory | 1 GiB RAM | 0.5 GB RAM |
| Storage | 8 GiB | Ephemeral (task-level) |
| App Port | 8081 | 8081 |
| Image Source | Built locally on host | Amazon ECR |
| Inbound Rule | TCP 8081 | TCP 8081 |

## ✅ Prerequisites

- An AWS account
- Docker installed (locally and/or on the EC2 instance)
- AWS CLI configured (`aws configure`) – required for the ECR / Fargate path
- Git

## 🚀 Quick Start (Local)

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>

docker build -t task-app:1.0 .
docker run -d --name task-app-container -p 8081:8081 task-app:1.0
```

Open <http://localhost:8081>.

## ☁️ Deployment Option 1: AWS EC2

### 1. Launch the instance
- AMI: **Ubuntu**
- Instance type: **t3.micro**
- Storage: **8 GiB**
- Security Group: add an inbound rule for **TCP 8081**

### 2. Install Docker

```bash
sudo apt update
sudo apt install -y docker.io git
sudo systemctl enable --now docker
```

### 3. Clone, build and run

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>

sudo docker build -t task-app:1.0 .
sudo docker run -d \
  --name task-app-container \
  --restart unless-stopped \
  -p 8081:8081 \
  task-app:1.0
```

### 4. Access the application

```
http://<YOUR-EC2-PUBLIC-IP>:8081
```

## ☁️ Deployment Option 2: AWS ECS Fargate

### 1. Create an ECR repository and push the image

```bash
AWS_REGION=<your-region>
ACCOUNT_ID=<your-account-id>

aws ecr create-repository --repository-name task-app --region $AWS_REGION

aws ecr get-login-password --region $AWS_REGION | \
  docker login --username AWS --password-stdin $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com

docker tag task-app:1.0 $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/task-app:1.0
docker push $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/task-app:1.0
```

### 2. Create the ECS resources

1. **ECS Cluster** – create a cluster using the Fargate launch type.
2. **Task Definition** – Fargate, **0.25 vCPU / 0.5 GB RAM**, container image from ECR, container port **8081**.
3. **ECS Service** – run 1 desired task, assign a **public IP**, and attach a security group allowing **TCP 8081**.

### 3. Access the application

Open the running task in the ECS console, copy its public IP, and visit:

```
http://<FARGATE-PUBLIC-IP>:8081
```

## 🔍 Useful Commands

```bash
# Check running containers
sudo docker ps

# View logs
sudo docker logs task-app-container

# Restart / stop / remove
sudo docker restart task-app-container
sudo docker stop task-app-container
sudo docker rm task-app-container

# Rebuild after code changes
sudo docker build -t task-app:1.0 .
sudo docker rm -f task-app-container
sudo docker run -d --name task-app-container --restart unless-stopped -p 8081:8081 task-app:1.0
```

## 🛠️ Troubleshooting

| Problem | Possible Cause | Fix |
|---------|----------------|-----|
| Site not reachable | Security group missing rule | Allow inbound **TCP 8081** |
| Container exits immediately | Nginx config error | Check `docker logs task-app-container` |
| `port is already allocated` | Old container still running | `docker rm -f task-app-container` |
| Fargate task can't pull image | Missing ECR permissions or no public IP/NAT | Check the task execution role and network settings |
| Fargate IP changed | Task was restarted | Use a load balancer or Elastic IP-style setup for a stable address |

## 🔐 Security Notes

- Opening port 8081 to `0.0.0.0/0` is fine for a demo; restrict the source CIDR for anything beyond that.
- Never commit AWS credentials or `.pem` key files to the repository.
- Keep the Docker image and base OS updated.

## 🧹 Cleanup

To avoid unexpected AWS charges:

- Terminate the EC2 instance.
- Delete the ECS service, cluster and task definitions.
- Delete the ECR repository and any unused images.

## 🔮 Future Improvements

- CI/CD with GitHub Actions (build, push to ECR, deploy)
- HTTPS with an Application Load Balancer and ACM certificate
- Custom domain with Route 53
- Infrastructure as Code (Terraform or CloudFormation)

## 👤 Author

**Your Name** – [GitHub](https://github.com/<your-username>)
