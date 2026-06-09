# Pac-Man DevOps Project on AWS EKS

## Project Overview

This project demonstrates a complete DevOps deployment workflow for the Pac-Man application using Docker, Kubernetes, Amazon ECR, Amazon EKS, an AWS Network Load Balancer, and MongoDB with persistent storage.

The main goal of the project is to containerize the application, deploy it to Kubernetes, expose it publicly through a Load Balancer, and manage the database using a StatefulSet with Persistent Volumes.

## Architecture

The deployed architecture is:

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
PVC
   ↓
AWS EBS Volume
```

The CI/CD flow is:

```text
Developer Push to GitHub
   ↓
GitHub Actions
   ↓
Docker Build
   ↓
Push Image to Amazon ECR
   ↓
Deploy to Amazon EKS
```

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
* Kubernetes StatefulSet
* PersistentVolumeClaim
* GitHub Actions
* eksctl
* AWS CLI

## Repository Structure

```text
.
├── .github/
│   └── workflows/
│       └── build-and-push.yml
├── docker/
├── docs/
│   └── evidence/
│       └── eks-deployment-results.txt
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
├── public/
├── routes/
├── views/
├── Dockerfile
├── docker-compose.yml
├── package.json
├── package-lock.json
└── README.md
```

## Local Docker Deployment

Build the Docker image:

```bash
docker build -t pacman-app:local .
```

Run MongoDB:

```bash
docker run -d \
  --name pacman-mongo \
  -p 27017:27017 \
  mongo:3.6
```

Run the Pac-Man application:

```bash
docker run -d \
  --name pacman-app \
  --network pacman-net \
  -p 8080:8080 \
  -e MONGO_SERVICE_HOST=pacman-mongo \
  -e MONGO_DATABASE=pacman \
  -e MY_MONGO_PORT=27017 \
  pacman-app:local
```

## Docker Compose

The project also includes a `docker-compose.yml` file for running the application and MongoDB together locally.

Run:

```bash
docker compose up -d --build
```

Check containers:

```bash
docker ps
docker compose logs pacman-app
```

## Kubernetes Deployment on Minikube

Load the local Docker image into Minikube:

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

Check the deployment:

```bash
kubectl get pods
kubectl get svc
kubectl get statefulset
kubectl get pvc
kubectl logs deployment/pacman
```

Access the application locally:

```bash
kubectl port-forward --address 0.0.0.0 service/pacman 8080:8080
```

## Amazon ECR

Create an ECR repository:

```bash
aws ecr create-repository \
  --repository-name pacman-app \
  --region eu-central-1
```

Authenticate Docker to ECR:

```bash
aws ecr get-login-password --region eu-central-1 | \
docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.eu-central-1.amazonaws.com
```

Tag and push the image:

```bash
docker tag pacman-app:local <AWS_ACCOUNT_ID>.dkr.ecr.eu-central-1.amazonaws.com/pacman-app:latest

docker push <AWS_ACCOUNT_ID>.dkr.ecr.eu-central-1.amazonaws.com/pacman-app:latest
```

## EKS Cluster Creation

The EKS cluster was created using `eksctl` with EKS Auto Mode.

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
aws eks describe-cluster --name pacman-eks --region eu-central-1 --query "cluster.status"
```

## EKS Application Deployment

Apply the EKS Kubernetes manifests:

```bash
kubectl apply -f k8s/storage-class-eks.yaml
kubectl apply -f k8s/mongo-service.yaml
kubectl apply -f k8s/mongo-statefulset.yaml
kubectl apply -f k8s/pacman-deployment-eks.yaml
kubectl apply -f k8s/pacman-service-eks.yaml
```

Verify the deployment:

```bash
kubectl get pods -o wide
kubectl get svc
kubectl get statefulset
kubectl get pvc
kubectl logs deployment/pacman
```

Expected result:

```text
mongo-0                   1/1 Running
pacman-...                1/1 Running
mongo                     1/1
mongo-data-mongo-0        Bound
Connected to database server successfully
```

## Load Balancer Access

The Pac-Man application is exposed using a Kubernetes Service of type `LoadBalancer`.

Get the external Load Balancer address:

```bash
kubectl get svc pacman
```

Open the application in a browser:

```text
http://<LOAD_BALANCER_DNS_NAME>
```

## MongoDB Validation

Connect to the MongoDB pod:

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

This validates that the Pac-Man application is connected to MongoDB and that data is stored in the database.

## CI/CD Pipeline

The project includes a GitHub Actions workflow:

```text
.github/workflows/build-and-push.yml
```

The workflow performs the following steps:

1. Checkout the source code
2. Configure AWS credentials
3. Login to Amazon ECR
4. Build the Docker image
5. Tag the Docker image
6. Push the image to Amazon ECR

Required GitHub Secrets:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_ACCOUNT_ID
```

## Evidence

Deployment evidence is stored under:

```text
docs/evidence/
```

Recommended screenshots are stored under:

```text
screenshots/
```

Important evidence includes:

* EKS cluster status: ACTIVE
* ECR image with `latest` tag
* Running Kubernetes pods
* LoadBalancer service with external DNS
* MongoDB StatefulSet ready
* PVC status: Bound
* Pac-Man logs showing successful MongoDB connection
* Pac-Man application running in the browser

## Cleanup

To avoid unnecessary AWS charges, delete the EKS cluster when finished:

```bash
eksctl delete cluster -f infra/eks-auto-cluster.yaml
```

Verify that resources were deleted:

```bash
aws eks describe-cluster --name pacman-eks --region eu-central-1
```

Optional ECR cleanup:

```bash
aws ecr delete-repository \
  --repository-name pacman-app \
  --region eu-central-1 \
  --force
```

## Notes

* The Pac-Man application uses MongoDB as its database.
* MongoDB is deployed as a Kubernetes StatefulSet.
* Persistent storage is provided through a PVC and AWS EBS.
* The application is exposed externally through an AWS Load Balancer.
* In production, GitHub Actions should use AWS OIDC instead of long-term AWS access keys.
