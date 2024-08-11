# Installation (Ubuntu 22.04)

```bash
# confirm virtualization enabled
cat /proc/cpuinfo | grep svm
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```

# Run / Configure

```bash
minikube start --driver=docker
minikube start --kubernetes-version=xyz --profile xyz --nodes=2 --disk-size=10g --cpus=2 --memory=6g --cni=calico --container-runtime=cri-o
minikube status
minikube profile list
minikube stop
minikube delete
```

# Cluster Management

```bash
minikube node list
minikube ip
```