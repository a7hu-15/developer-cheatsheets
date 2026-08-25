# ☸️ Kubernetes & Helm Cheatsheet

A production-grade reference guide for Kubernetes (`kubectl`) cluster administration, workload management, debugging, and Helm 3 chart management.

---

## 🛠️ Essential `kubectl` Commands

### Cluster & Context Management
```bash
# Check cluster info and component status
kubectl cluster-info
kubectl get componentstatuses

# Switch contexts and namespaces
kubectl config get-contexts
kubectl config use-context <context-name>
kubectl config set-context --current --namespace=<namespace>
```

### Resource Overview (`get`, `describe`)
```bash
# List all pods across all namespaces
kubectl get pods -A -o wide

# Get deployment details with label selector
kubectl get deploy -l app=frontend -o yaml

# Detailed diagnostics for a specific resource
kubectl describe pod <pod-name> -n <namespace>
```

---

## 🔍 Troubleshooting & Debugging

```bash
# Stream logs from a container (with tail & follow)
kubectl logs -f <pod-name> -c <container-name> --tail=100

# Stream logs from multi-container deployment
kubectl logs -f deployment/<deploy-name> --all-containers=true

# Execute interactive shell inside pod
kubectl exec -it <pod-name> -- /bin/sh

# Resource utilization (CPU/Memory)
kubectl top node
kubectl top pod --sort-by=cpu
```

---

## ⚓ Helm 3 Chart Operations

```bash
# Manage repositories
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Install / Upgrade releases
helm upgrade --install my-app ./my-chart --namespace prod --set replicaCount=3 -f values.prod.yaml

# Rollback release to previous revision
helm rollback my-app 1 -n prod

# List releases and status
helm list -n prod
```
