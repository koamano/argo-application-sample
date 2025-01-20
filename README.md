# argo-application-sample
Follow the instructions below to install on ArgoCD on your cluster and use the App of App pattern to deploy multiple applications on your AKS cluster. Nginx Ingress Controller (not the "official" Kubernetes ingress controller) is installed and used to route requests to all of the applications. 

## Pre-Requisites
Deploy an AKS or EKS K8s cluster
### For EKS
eksctl can be used for simple creation

eksctl create cluster \
> --name koamano-test \
> --region us-east-1 \
> --nodegroup-name linux-nodes \
> --node-type t3.medium

## Repo Dependencies
The App and App pattern uses three repositories
### Artifactory sample repository
Subchart using JFrog Platform Helm Chart. [Link to Repo](https://github.com/koamano/artifactory-sample)
### Sonarqube sample repository
Subchart using Sonarqube Helm Chart. [Link to Repo](https://github.com/koamano/sonarqube-sample)
### Sample calc repository
Sample calculator application with Front end and API, deployed with Kustomization. [Link to Repo](https://github.com/koamano/calc-sample)

## Set up Steps
### Get Credentials for the deployed AKS cluster (already set up if eksctl used for EKS)
az aks get-credentials --resource-group dev-aks-argo-sample --name aks-argo-cluster

### Create namespace for ArgoCD
kubectl create namespace argocd

### Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

### Expose external IP for ArgoUI (for development purposes)
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

### Install ArgoCD CL locally
brew install argocd

### Get initial password for ArgoCD (username is "admin")
argocd admin initial-password -n argocd

### Install Nginx Ingress Controller
helm install my-release oci://ghcr.io/nginxinc/charts/nginx-ingress --version 1.0.2

### To upgrade to Nginx Plus
The Nginx Plus image must be used. There are three options to use the Nginx Plus image.
1. Pull directly from the F5 container registry.
2. Pull the image and push it to your private registry. 
3. build the image and push to your private registry. 
<br>
[Link to more information]https://docs.nginx.com/nginx-ingress-controller/installation/installing-nic/installation-with-helm/

### Apply App of App application
Clone argo-application-sample repo and navigate root.
kubectl apply -f ./app-of-apps-dev.yaml

### Set CNAM in route53
kubectl get service -n sample

copy EXTERNAL-IP of myapp-service and paste into CNAM record

### Test
Change replica number in calc-sample overlay

### Destroy cluster
eksctl delete cluster --name koamano-test

# Appendix
### Install each application separately from argo-application-sample
k apply -f ./applications/dev/sonarqube-application-dev.yaml
