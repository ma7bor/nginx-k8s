# Nginx Kubernetes Deployment

This project contains Kubernetes configuration files to deploy an Nginx application using both ReplicaSets and Deployments. It includes configurations for the Nginx container, resource limits, and a NodePort service for external access.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Setup Instructions](#setup-instructions)
   - [Clone Repository](#clone-repository)
   - [Apply Configuration Files](#apply-configuration-files)
3. [Configuration Files](#configuration-files)
   - [ReplicaSet](#replicaset)
   - [Deployment](#deployment)
   - [Service](#service)
4. [Understanding Kubernetes Resources](#understanding-kubernetes-resources)
5. [Running the Project](#running-the-project)
   - [Step 1: Apply ReplicaSet Configuration](#step-1-apply-replicaset-configuration)
   - [Step 2: Access the Service](#step-2-access-the-service)
   - [Step 3: Clean Up ReplicaSet](#step-3-clean-up-replicaset)
   - [Step 4: Apply Deployment Configuration](#step-4-apply-deployment-configuration)
   - [Step 5: Compare Deployment and ReplicaSet](#step-5-compare-deployment-and-replicaset)
6. [Troubleshooting](#troubleshooting)
7. [Cleaning Up](#cleaning-up)

## Prerequisites

- Kubernetes cluster
- `kubectl` installed and configured
- `minikube` installed (for local development)
- Access to Docker Hub (or any other container registry)

## Setup Instructions

### Clone Repository

Clone the repository to your local machine:

```bash
git clone https://github.com/ma7bor/nginx-k8s.git
cd nginx-k8s
```

## Configuration Files

### ReplicaSet

`nginx.replicaset.yaml`:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
spec:
  replicas: 5
  selector:
    matchLabels: 
      app: mon-app
  template:
    metadata:
      labels:
        app: mon-app
    spec:
      containers:
        - name: nginx-container
          image: nginx
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "200m"
          ports:
            - containerPort: 80
```

### Deployment

`nginx.deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels: 
        app: nginx
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
        ports:
         - containerPort: 80
        resources:
          limits:
            memory: "256Mi"
            cpu: "200m"
          requests:
            memory: "128Mi"
            cpu: "100m"
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-servicev3
spec:
  selector:
    app: nginx
  type: NodePort
  ports:
  - nodePort: 31002
    port: 80
    targetPort: 80
```

### Service

The Service configuration is included within the Deployment file.

## Understanding Kubernetes Resources

### Deployments, ReplicaSets, Pods, and Containers

- **Containers**: These are the smallest deployable units in Kubernetes. Containers package your application code along with its dependencies and runtime environment. In this project, we use an Nginx container.

- **Pods**: A Pod is an abstraction over containers. It represents a single instance of a running process in your cluster. A Pod can contain one or more containers that are tightly coupled and share resources like network and storage.

- **ReplicaSets**: ReplicaSets ensure that a specified number of identical Pods are running at any given time. If a Pod crashes or is deleted, the ReplicaSet will automatically create a new one to maintain the desired number of replicas.

- **Deployments**: Deployments provide a higher-level abstraction for managing ReplicaSets and, by extension, Pods. They allow you to declaratively update your applications by managing ReplicaSets. When you create or update a Deployment, it automatically creates and manages the underlying ReplicaSet(s) for you.

### Why Use Deployments and ReplicaSets?

Using Deployments instead of directly managing ReplicaSets offers several advantages:

- **Declarative Updates**: Deployments provide a way to declaratively update your applications. You simply specify the desired state, and the Deployment controller handles the rest.

- **Rolling Updates**: Deployments support rolling updates, allowing you to update your application without downtime by incrementally replacing Pods with new ones.

- **Rollback**: Deployments keep a history of previous states, allowing you to roll back to a previous version if something goes wrong.

- **Scalability**: Deployments make it easy to scale your application up or down by adjusting the number of replicas.

In summary, Deployments manage ReplicaSets, and ReplicaSets manage Pods. Pods, in turn, are an abstraction over containers. This hierarchy allows Kubernetes to provide robust and flexible management of containerized applications.

## Running the Project

### Step 1: Apply ReplicaSet Configuration

First, apply the ReplicaSet configuration to create the Pods:

```bash
kubectl apply -f nginx.replicaset.yaml
```

### Step 2: Access the Service

Check the status of the Service:

```bash
kubectl get svc nginx-servicev3
```

For local development using `minikube`, you can open the service URL directly in your browser:

```bash
minikube service nginx-servicev3
```

This command will open the Nginx application in your default web browser.

### Step 3: Clean Up ReplicaSet

Delete the ReplicaSet and associated resources to prepare for the Deployment:

```bash
kubectl delete -f nginx.replicaset.yaml
```

### Step 4: Apply Deployment Configuration

Next, apply the Deployment configuration to create and manage the Pods via ReplicaSets:

```bash
kubectl apply -f nginx.deployment.yaml
```

### Step 5: Compare Deployment and ReplicaSet

Check the status of the Deployment:

```bash
kubectl get deployments nginx-deployment
```

Check the status of the ReplicaSet created by the Deployment:

```bash
kubectl get rs -l app=nginx
```

Check the status of the Pods:

```bash
kubectl get pods
```

Check the status of the Service:

```bash
kubectl get svc nginx-servicev3
```

Access the Nginx application using `minikube` as before:

```bash
minikube service nginx-servicev3
```

Observe how the Deployment manages the ReplicaSet and, by extension, the Pods. This demonstrates the benefits of using Deployments for managing application updates, scaling, and rollback.

## Troubleshooting

- If the Pods are not running correctly, check the Pod logs:

  ```bash
  kubectl logs <pod-name>
  ```

- If the Deployment or ReplicaSet is not creating the desired number of replicas, describe the resource to get more details:

  ```bash
  kubectl describe deployment nginx-deployment
  kubectl describe rs -l app=nginx
  ```

- Ensure that the nodes have sufficient resources to run the containers with the specified resource requests and limits.

## Cleaning Up

To delete the resources created by these configuration files, use the following commands:

```bash
kubectl delete -f nginx.replicaset.yaml
kubectl delete -f nginx.deployment.yaml
```