# ☸️ Kubernetes & Helm 3 Cheatsheet

A production-ready reference guide for `kubectl` cluster management, object definitions, container debugging, and Helm 3 package management workflows.

---

## 🚀 Context & Namespace Management

```bash
# Display active context and cluster info
kubectl config get-contexts
kubectl config current-context
kubectl cluster-info

# Switch to a different cluster context
kubectl config use-context minikube

# Set default namespace for current context
kubectl config set-context --current --namespace=prod

# List all namespaces
kubectl get namespaces
```

---

## 📦 Pod & Workload Management

```bash
# List all pods in active namespace (or across all namespaces)
kubectl get pods
kubectl get pods -A -o wide

# Describe pod details (events, volume mounts, status)
kubectl describe pod <pod-name>

# Create deployment from command line
kubectl create deployment web-app --image=nginx:alpine --replicas=3

# Scale deployment replicas
kubectl scale deployment/web-app --replicas=5

# Rollout status and history
kubectl rollout status deployment/web-app
kubectl rollout history deployment/web-app
kubectl rollout undo deployment/web-app
```

---

## 🔍 Debugging & Log Inspection

```bash
# Print logs of a pod (and stream output live)
kubectl logs -f <pod-name>

# Print logs for a specific container in a multi-container pod
kubectl logs -f <pod-name> -c <container-name>

# View logs of a previously crashed container instance
kubectl logs -p <pod-name>

# Execute interactive shell inside running container
kubectl exec -it <pod-name> -- /bin/sh

# Port-forward a local port to a pod or service
kubectl port-forward pod/<pod-name> 8080:80
kubectl port-forward svc/<service-name> 8080:80

# Run ephemeral debug pod in cluster
kubectl run tmp-shell --rm -i --tty --image=busybox -- sh
```

---

## 🌐 Services, Ingress, & Config

```bash
# Expose deployment as a ClusterIP or NodePort service
kubectl expose deployment web-app --port=80 --target-port=8080 --type=ClusterIP

# List services, ingress rules, and endpoints
kubectl get svc
kubectl get ingress
kubectl get endpoints

# Create ConfigMap from literal values or files
kubectl create configmap app-config --from-literal=ENV=production --from-file=config.json

# Create Secret from literal values
kubectl create secret generic app-secret --from-literal=db_pass=s3cr3t
```

---

## ⛵ Helm 3 Package Management

```bash
# Add, list, and update Helm chart repositories
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo list
helm repo update

# Search for charts locally or on Artifact Hub
helm search repo nginx
helm search hub redis

# Install or upgrade a chart release
helm upgrade --install my-release bitnami/nginx \
  --namespace prod \
  --set service.type=ClusterIP \
  -f custom-values.yaml

# List installed chart releases
helm list -A

# Check status and rendered manifests of a release
helm status my-release
helm get manifest my-release

# Rollback release to a previous revision
helm rollback my-release 1

# Uninstall a release
helm uninstall my-release -n prod
```

---

## ⚡ Useful kubectl Imperative Shortcuts

```bash
# Generate manifest YAML without applying (Dry Run)
kubectl create deployment my-app --image=redis --dry-run=client -o yaml > deployment.yaml
kubectl expose deployment my-app --port=6379 --dry-run=client -o yaml > service.yaml

# Quick resource deletion (force delete stuck pod)
kubectl delete pod <pod-name> --grace-period=0 --force
```
