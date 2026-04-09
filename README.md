# Kubernetes Hello World

A simple demonstration of deploying a static "Hello World" page to a local Kubernetes cluster using kind.

## Overview

This project demonstrates core Kubernetes concepts by:
- Setting up a local Kubernetes cluster with [kind](https://kind.sigs.k8s.io/) (Kubernetes IN Docker)
- Deploying an nginx Pod using a YAML manifest
- Exposing the Pod to the host machine via `kubectl port-forward`
- Accessing the served page in a web browser

## Architecture

```
Host machine (Windows)
    ↓
Docker Desktop
    ↓
kind cluster (Docker container running Kubernetes)
    ↓
Pod: hello-world-pod
    ↓
Container: nginx (serving on port 80)

Browser → localhost:8080 → (port-forward) → Pod:80 → nginx → HTML
```

## Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)

## Quick Start

### 1. Create the cluster

```powershell
kind create cluster --name hello-world
```

### 2. Deploy the Pod

```powershell
kubectl apply -f hello-world-pod.yaml
```

### 3. Wait for the Pod to be ready

```powershell
kubectl get pods
```

Wait until STATUS shows `Running`.

### 4. Port forward

```powershell
kubectl port-forward pod/hello-world-pod 8080:80
```

Leave this terminal window open.

### 5. Open in browser

Navigate to: http://localhost:8080

You should see the "Welcome to nginx!" page.

## Cleanup

```powershell
# Stop port-forward with Ctrl+C, then:
kubectl delete -f hello-world-pod.yaml
kind delete cluster --name hello-world
```

## File structure

- `hello-world-pod.yaml` — Kubernetes Pod manifest
- `README.md` — this file
- `GUIDE.md` — detailed step-by-step walkthrough in Norwegian (optional)

## What I learned

- **Kubernetes fundamentals**: Pods, manifests, cluster, kubectl
- **Declarative infrastructure**: Describing desired state in YAML
- **Container networking**: Port-forward for local development
- **Local Kubernetes with kind**: Fast feedback loop without cloud costs

## Next steps

- Replace `kubectl port-forward` with a Kubernetes Service
- Add an Ingress controller for HTTP routing
- Deploy to Azure Kubernetes Service (AKS) using Terraform
- Add a Deployment resource for multiple replicas and self-healing

## Notes

This is a learning project demonstrating basic Kubernetes concepts. For production workloads, use proper Services, Ingress, and managed Kubernetes (AKS, EKS, GKE) instead of kind and port-forward.
