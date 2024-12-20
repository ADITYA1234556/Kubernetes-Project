# PROJECT

## **Phase 1: Initial Kubernetes cluster Setup and Deployment**
**Step 1: **

### 1. Launch EC2 (Ubuntu 22.04): 
- Provision an EC2 instance on AWS with Ubuntu 22.04.
- Connect to the instance using SSH.

### 2. Install Kubernetes cluster components [On Master & Worker Node]

- Use Script.sh script provided in the repo

### 3. Initialize Kubernetes Master Node [On MasterNode]

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

### 4. Join the kubernetes cluster [On WorkerNodes]

```bash
sudo kubeadm join 172.31.20.145:6443 --token mhi5wy.3385kwfnaytaa7mf \
        --discovery-token-ca-cert-hash sha256:ffce702b1a02ff38f47503e83b74d7ad188f49daa31bee1a98e3813b73234a87
```

### 9. Configure Kubernetes Cluster [On MasterNode]

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 10. Deploy Networking Solution (Calico) [On MasterNode]

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

### 11. Deploy Ingress Controller (NGINX) [On MasterNode]

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.49.0/deploy/static/provider/baremetal/deploy.yaml
```
