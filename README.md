# Kubernetes Kind Cluster Lab

A beginner Kubernetes lab that runs locally using Kind and Docker Desktop.

## What I Practiced

- Creating a local Kubernetes cluster with Kind
- Deploying a standalone Nginx Pod
- Creating and scaling a Deployment
- Testing Kubernetes self-healing
- Exposing Pods through a ClusterIP Service
- Accessing the application using port forwarding

## Project Files

- `kind-config.yaml` — Defines the Kind cluster
- `pod.yaml` — Creates a standalone Nginx Pod
- `deployment.yaml` — Manages multiple Nginx Pods
- `service.yaml` — Provides one stable endpoint for the Pods

## Requirements

- Docker Desktop
- Kind
- kubectl

## Create the Cluster

```bash
kind create cluster --name kind-lab
```

## Deploy the Application

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

## Check the Resources

```bash
kubectl get nodes
kubectl get deployments
kubectl get pods
kubectl get services
```

## Access Nginx

```bash
kubectl port-forward service/nginx-service 8080:80
```

Open:

```text
http://localhost:8080
```

## Test Self-Healing

Delete one of the Pods:

```bash
kubectl delete pod <pod-name>
```

Kubernetes automatically creates a replacement because the Deployment maintains the desired number of replicas.

## Scale the Application

```bash
kubectl scale deployment nginx-deployment --replicas=3
```

## Clean Up

```bash
kubectl delete -f service.yaml
kubectl delete -f deployment.yaml
kind delete cluster --name kind-lab
```


## Kubernetes API Resources Lab

This lab explores how Kubernetes resources are defined, validated, and managed through the API server.

### Concepts Practiced

- Kubernetes resource structure: `apiVersion`, `kind`, `metadata`, and `spec`
- Discovering resources with `kubectl api-resources`
- Exploring YAML fields with `kubectl explain`
- Namespaced versus cluster-wide resources
- Server-side YAML validation with `--dry-run=server`
- Creating and inspecting a standalone Nginx Pod

### Commands

```bash
kubectl api-resources
kubectl explain pod
kubectl explain pod.spec.containers
kubectl apply -f api-resources/pod.yaml
kubectl get pods
kubectl describe pod api-demo
kubectl apply --dry-run=server -f api-resources/pod.yaml