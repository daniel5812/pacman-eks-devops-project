# Pac-Man DevOps Project on AWS EKS

## Project Overview

This project demonstrates a complete DevOps deployment workflow for the Pac-Man application on AWS EKS.

The application was containerized with Docker, pushed to Amazon ECR, deployed to an EKS Auto Mode cluster, exposed externally using an AWS Network Load Balancer, and connected to a MongoDB database deployed as a Kubernetes StatefulSet with persistent storage.

The project also includes a GitHub Actions CI/CD pipeline that automatically builds the Docker image, pushes it to Amazon ECR, and deploys the Kubernetes manifests to EKS.

---

## Requirements Covered

* Infrastructure provisioned using `eksctl`
* EKS Auto Mode cluster
* Dockerized Pac-Man application
* CI/CD pipeline using GitHub Actions
* Docker image stored in Amazon ECR
* Application deployed to AWS EKS
* Application exposed using AWS Load Balancer / NLB
* MongoDB deployed as StatefulSet
* Persistent storage using PVC and AWS EBS
* Kubernetes manifest files included
* Deployment evidence and screenshots included

---

## Architecture

### Runtime Architecture

```text
User Browser
   ↓
AWS Network Load Balancer
   ↓
Kubernetes Service: pacman
   ↓
Pac-Man Deployment / Pod
   ↓
Kubernetes Service: mongo
   ↓
MongoDB StatefulSet: mongo-0
   ↓
PersistentVolumeClaim
   ↓
AWS EBS Volume
```

### CI/CD Pipeline

```text
Developer Push to GitHub
   ↓
GitHub Actions
   ↓
Docker Build
   ↓
Push Image to Amazon ECR
   ↓
Update kubeconfig for EKS
   ↓
kubectl apply Kubernetes manifests
   ↓
Restart Pac-Man Deployment
   ↓
Application updated on EKS
```

---

## Technologies Used

* Docker
* Docker Compose
* Kubernetes
* Minikube
* AWS EKS
* EKS Auto Mode
* Amazon ECR
* AWS Network Load Balancer
* MongoDB
* StatefulSet
* PersistentVolumeClaim
* AWS EBS
* GitHub Actions
* eksctl
* AWS CLI

---

## Repository Structure

```text
.
├── .github/
│   └── workflows/
│       └── build-and-push.yml
├── docs/
│   └── evidence/
│       ├── eks-deployment-results.txt
│       └── github-actions-deployment-results.txt
├── infra/
│   └── eks-auto-cluster.yaml
├── k8s/
│   ├── mongo-service.yaml
│   ├── mongo-statefulset.yaml
│   ├── pacman-deployment.yaml
│   ├── pacman-deployment-eks.yaml
│   ├── pacman-service.yaml
│   ├── pacman-service-eks.yaml
│   └── storage-class-eks.yaml
├── screenshots/
├── Dockerfile
├── docker-compose.yml
├── package.json
├── package-lock.json
├── README.md
└── README-original.md
```

---

## Docker

The application was containerized using a custom Dockerfile.

Build the Docker image locally:

```bash
docker build -t pacman-app:local .
```

Run locally with Docker Compose:

```bash
docker compose up -d --build
```

Check the running containers:

```bash
docker ps
docker compose logs pacman-app
```

---

## Kubernetes on Minikube

Before deploying to AWS, the application was tested locally on Minikube.

Load the local image into Minikube:

```bash
minikube image load pacman-app:local
```

Apply the Kubernetes manifests:

```bash
kubectl apply -f k8s/mongo-service.yaml
kubectl apply -f k8s/mongo-statefulset.yaml
kubectl apply -f k8s/pacman-deployment.yaml
kubectl apply -f k8s/pacman-service.yaml
```

Validate the deployment:

```bash
kubectl get pods
kubectl get svc
kubectl get statefulset
kubectl get pvc
kubectl logs deployment/pacman
```

---

## Amazon ECR

The Docker image was pushed to Amazon ECR.

ECR repository:

```text
pacman-app
```

Image tag:

```text
latest
```

Example image URI:

```text
696223520711.dkr.ecr.eu-central-1.amazonaws.com/pacman-app:latest
```

---

## EKS Infrastructure

The EKS cluster was provisioned using `eksctl` with EKS Auto Mode.

Cluster configuration file:

```text
infra/eks-auto-cluster.yaml
```

Create the cluster:

```bash
eksctl create cluster -f infra/eks-auto-cluster.yaml
```

Verify the cluster:

```bash
kubectl get nodes
aws eks describe-cluster \
  --name pacman-eks \
  --region eu-central-1 \
  --query "cluster.status"
```

Expected result:

```text
ACTIVE
```

---

## EKS Application Deployment

The application and database were deployed using Kubernetes manifests.

Apply the manifests:

```bash
kubectl apply -f k8s/storage-class-eks.yaml
kubectl apply -f k8s/mongo-service.yaml
kubectl apply -f k8s/mongo-statefulset.yaml
kubectl apply -f k8s/pacman-deployment-eks.yaml
kubectl apply -f k8s/pacman-service-eks.yaml
```

Validate the deployment:

```bash
kubectl get pods -o wide
kubectl get svc
kubectl get statefulset
kubectl get pvc
kubectl logs deployment/pacman
```

Expected results:

```text
mongo-0                   1/1 Running
pacman-...                1/1 Running
mongo                     1/1
mongo-data-mongo-0        Bound
Connected to database server successfully
```

---

## Load Balancer Access

The Pac-Man application is exposed using a Kubernetes Service of type `LoadBalancer`.

Get the Load Balancer DNS:

```bash
kubectl get svc pacman
```

Open the application in a browser:

```text
http://<LOAD_BALANCER_DNS_NAME>
```

---

## MongoDB StatefulSet and Persistent Storage

MongoDB is deployed using a Kubernetes StatefulSet.

The StatefulSet uses:

* MongoDB container image
* Kubernetes ClusterIP Service
* PersistentVolumeClaim
* EKS Auto Mode StorageClass
* AWS EBS persistent volume

Validate MongoDB:

```bash
kubectl exec -it mongo-0 -- mongo
```

Inside MongoDB:

```javascript
use pacman
show collections
db.highscore.find().pretty()
db.userstats.find().pretty()
exit
```

---

## GitHub Actions CI/CD

The CI/CD workflow is located at:

```text
.github/workflows/build-and-push.yml
```

The pipeline performs the following steps:

1. Checkout source code
2. Configure AWS credentials
3. Login to Amazon ECR
4. Build Docker image
5. Tag Docker image
6. Push Docker image to ECR
7. Update kubeconfig for EKS
8. Apply Kubernetes manifests
9. Restart the Pac-Man deployment
10. Verify rollout status

Required GitHub repository secrets:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_ACCOUNT_ID
```

The IAM user used by GitHub Actions was granted access to the EKS cluster using EKS Access Entries and an EKS access policy.

---

## Evidence

Deployment evidence is stored under:

```text
docs/evidence/
```

Screenshots are stored under:

```text
screenshots/
```

Recommended screenshots:

* GitHub Actions successful workflow
* ECR repository with `pacman-app:latest`
* EKS cluster status `ACTIVE`
* `kubectl get pods -o wide`
* `kubectl get svc` showing Load Balancer DNS
* `kubectl get pvc` showing PVC status `Bound`
* Pac-Man logs showing MongoDB connection
* Pac-Man application running in the browser through the Load Balancer
* MongoDB records after playing the game

---

## Cleanup

To avoid unnecessary AWS charges, delete the EKS cluster after finishing the project:

```bash
eksctl delete cluster -f infra/eks-auto-cluster.yaml
```

Verify deletion:

```bash
aws eks describe-cluster \
  --name pacman-eks \
  --region eu-central-1
```

Optional ECR cleanup:

```bash
aws ecr delete-repository \
  --repository-name pacman-app \
  --region eu-central-1 \
  --force
```

---

## Notes

* The application source code is located in the root of the repository.
* MongoDB is deployed separately from the application.
* The application connects to MongoDB through an internal Kubernetes Service named `mongo`.
* Persistent storage is handled by Kubernetes PVC and AWS EBS.
* The application is exposed publicly through an AWS Load Balancer.
* For a production environment, GitHub Actions should use AWS OIDC instead of long-term access keys.
