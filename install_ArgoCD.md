## Create Kubernetes Cluster
- use Create_Cluster_md

## Install ArgoCD
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## Access ArgoCD UI
- using port-forwarding
```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

## 
