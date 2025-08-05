# 🚀 Kubernetes Nginx Deployment Tutorial

This repository demonstrates how to deploy an Nginx application on Kubernetes and expose it using a NodePort service.

## 📦 Clone the Repository

```bash
git clone <thisRepoUrl>
cd k8s-learning
```

## 📁 Files Overview

- `nginx-deployment.yaml`: Defines a Deployment with 3 replicas of the Nginx container.
- `nginx-service.yaml`: Defines a NodePort service to expose the Nginx pods externally.

## 🛠️ Prerequisites

Ensure you have the following installed:
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- A running Kubernetes cluster (e.g., Minikube, Docker Desktop, or cloud provider)

## 🚀 Step-by-Step Guide

### 1. Create the Deployment

```bash
kubectl apply -f nginx-deployment.yaml
```

This will:
- Deploy 3 replicas of Nginx containers
- Use the official `nginx:1.14.2` image
- Expose port 80 inside the container

### 2. Expose the Deployment via NodePort

```bash
kubectl apply -f nginx-service.yaml
```

This will:
- Create a NodePort service named `nginx-nodeport-service`
- Forward traffic from a static port (e.g., 30080) on each node to port 80 on the pods

### 3. Verify the Setup

```bash
kubectl get pods
kubectl get svc nginx-nodeport-service
```

### 4. Access the Application

Open your browser or use `curl` to access:
```
http://<NodeIP>:30080
```

> Replace `<NodeIP>` with the IP address of your Kubernetes node. For Minikube, use:
```bash
minikube ip
```

## 🧹 Cleanup

To remove all resources:
```bash
kubectl delete -f nginx-deployment.yaml
kubectl delete -f nginx-service.yaml
```

## 📚 Learn More

- [Kubernetes Services](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Expose Your App](https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/)
