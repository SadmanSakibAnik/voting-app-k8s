# 📦 K8s Kind, ArgoCD, Grafana, Promethus Commands


---

## Install `kind`

```bash
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo cp ./kind /usr/local/bin/kind
rm -rf kind
```

---

##  Install `kubectl`

```bash
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

---


## Create & Manage Kubernetes Cluster (Kind)

```bash
# Clear terminal
clear

# Create 3-node Kubernetes cluster
kind create cluster --config=config.yml --name="your_cluster_name"

# Cluster info & nodes
kubectl cluster-info --context kind-kind
kubectl get nodes
kind get clusters
```

---


## Docker & Kubernetes Pods

```bash
# Docker container list
docker ps

# All Kubernetes pods
kubectl get pods -A
```

---

## Run Example Voting App

```bash
# Clone project
git clone https://github.com/dockersamples/example-voting-app.git
cd example-voting-app/

# Apply YAML files
kubectl apply -f k8s-specifications/

# View all resources
kubectl get all

# Port forwarding
kubectl port-forward service/vote 5000:5000 --address=0.0.0.0 &
kubectl port-forward service/result 5001:5001 --address=0.0.0.0 &
```

---

## Install Argo CD

```bash
# Namespace for Argo CD
kubectl create namespace argocd

# Install Argo CD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Check services
kubectl get svc -n argocd

# NodePort for UI access
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'

# Port forward for browser
kubectl port-forward -n argocd service/argocd-server -n argocd 8443:443 &
```

---

## Delete Cluster

```bash
kind delete cluster --name=kind
```

---



## Argo CD Admin Password

```bash
kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
```

---

## Install Helm

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

---

## Install Kube Prometheus Stack

```bash
# Add repos
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add stable https://charts.helm.sh/stable
helm repo update

# Create monitoring namespace
kubectl create namespace monitoring

# Install Prometheus stack
helm install kind-prometheus prometheus-community/kube-prometheus-stack   --namespace monitoring   --set prometheus.service.nodePort=30000   --set prometheus.service.type=NodePort   --set grafana.service.nodePort=31000   --set grafana.service.type=NodePort   --set alertmanager.service.nodePort=32000   --set alertmanager.service.type=NodePort   --set prometheus-node-exporter.service.nodePort=32001   --set prometheus-node-exporter.service.type=NodePort

# Check services
kubectl get svc -n monitoring
kubectl get namespace
```

### Port Forward

```bash
kubectl port-forward svc/kind-prometheus-kube-prome-prometheus -n monitoring 9090:9090 --address=0.0.0.0 &
kubectl port-forward svc/kind-prometheus-grafana -n monitoring 31000:80 --address=0.0.0.0 &
```

---

## Prometheus Queries

```promql
# CPU usage (%)
sum(rate(container_cpu_usage_seconds_total{namespace="default"}[1m])) / sum(machine_cpu_cores) * 100

# Memory usage by pod
sum(container_memory_usage_bytes{namespace="default"}) by (pod)

# Network in/out
sum(rate(container_network_receive_bytes_total{namespace="default"}[5m])) by (pod)
sum(rate(container_network_transmit_bytes_total{namespace="default"}[5m])) by (pod)
```

---