# DevOps CI/CD Starter Project

This project is designed for:
- Windows Laptop
- Kubernetes via Docker Desktop
- Jenkins running on Ubuntu WSL
- Helm deployments
- ArgoCD GitOps CD
- Multi environment setup (dev, stage, prod)

## Project Structure

```text
devops-cicd-project/
├── app/
├── helm/
├── k8s/
├── argocd/
├── Jenkinsfile
└── docker-compose.yml
```

## Prerequisites

- Docker Desktop with Kubernetes enabled
- WSL Ubuntu
- Jenkins installed in WSL
- kubectl
- Helm
- ArgoCD CLI

## Jenkins Pipeline Flow

1. Pull code from GitHub
2. Run tests
3. Build Docker image
4. Push image to DockerHub
5. Update Helm values
6. ArgoCD syncs automatically

## Namespaces

- dev
- stage
- prod

## Commands

### Create namespaces

```bash
kubectl create ns dev
kubectl create ns stage
kubectl create ns prod
```

### Install ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Deploy Helm Chart

```bash
helm install webapp ./helm/webapp -n dev
```
