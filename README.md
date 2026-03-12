# Marketplace Project – Deployment Guide

This repository contains the complete guide for setting up, building, and deploying the **Marketplace** project in local environments using **Docker Compose** and **Kubernetes (Minikube)**.

---

## Prerequisites

Before getting started, make sure the following tools are installed:

- Docker Desktop  
- Minikube  
- Kubectl  

---

# 1. Clone the Repository
Start by cloning the repository to your local machine:

```bash
git clone https://github.com/RobisonTorres/Marketplace.git
cd Marketplace
```

# 2. Running with Docker Compose (Local Environment)

The `docker-compose.yml` file located in the root directory is configured to create all necessary containers for running the project locally. This includes:

- Database  
- Backend  
- Frontend  

## Start the Application

```bash
docker compose up --build
```

This command:

- Builds the images if they do not already exist  
- Pulls required base images if necessary  
- Starts all defined services  

## Access the Application

The frontend uses an **Nginx** image and is exposed on port **8081**, according to the Docker Compose configuration.

Open your browser and access:
```bash
http://localhost:8081
```

---

# 3. Running with Kubernetes (Minikube)

## 3.1 Start and Configure Minikube

Start the Minikube cluster:

```bash
minikube start
```

This command:

- Creates a virtual machine  
- Initializes a local Kubernetes cluster  
- Configures the kubectl context automatically  

The project uses the YAML files located in the `k8s/` directory to apply Kubernetes resources such as:

- Namespace  
- Deployments  
- Services  
- ConfigMaps  
- Secrets  

---

## 3.2 Build Docker Images Locally

Connect local Docker to Minikube:

```bash
& minikube -p minikube docker-env --shell powershell | iex
```

Build the backend image:

```bash
docker build -t marketplace:1.0.0 ./backend
```

Build the frontend image:

```bash
docker build -t marketplace-frontend:1.0.0 ./frontend
```

## 3.3 Apply Kubernetes Configurations

1 - Apply the namespace:

```bash
kubectl apply -f k8s/namespace.yaml
```

2 - Apply database:

```bash
kubectl apply -f k8s/mysql/
```

3 - Apply backend:

```bash
kubectl apply -f k8s/backend/
```

4 - Apply frontend:

```bash
kubectl apply -f k8s/frontend/
```

### Note

The backend's deployment is configured to wait for the database to be ready before starting, ensuring proper initialization.

---

## 3.4 Access the Application

To open the frontend service in your browser:

```bash
minikube service nginx -n marketplace
```

This command will automatically open the application URL.

---

# 4. Monitoring and Scaling the Application

## 4.1 Generate Load for Testing (HPA)

To test Horizontal Pod Autoscaler (HPA) behavior, create a temporary load generator pod:

```bash
kubectl run load-generator -n marketplace --rm -it --image=busybox -- /bin/sh
```

Inside the container, execute:

```bash
while true; do wget -q -O- http://marketplace:8080/actuator/health; done
```

This continuously sends requests to the backend service, generating CPU load.

---

## 4.2 Monitor the HPA

In a separate terminal, monitor scaling activity:

```bash
kubectl get hpa -n marketplace -w
```

This command watches the HPA metrics in real time and displays scaling events as they occur.

---

The application should now be fully operational in both Docker Compose and Kubernetes environments.

# 5. CI/CD Pipeline

The project includes a GitHub Actions workflow defined in `.github/workflows/ci-cd.yml` that automates the build and deployment process in Github actions. This workflow is triggered on pushes to the `main` branch and performs all the tests and deployment steps automatically.
