
# Kubernetes Cluster Setup Guide

## Networking Requirements

1. Assign a unique hostname to all machines.
2. Ensure all VMs have different UUIDs (especially if cloned).
3. Allow required ports between master and worker nodes. https://kubernetes.io/docs/reference/networking/ports-and-protocols/
4. Ensure outbound internet access on all nodes.
5. Make sure SSH port TCP/22 is open (only from trusted ips) to manage nodes remotely

---

## Setup Steps

1. Disable SWAP
2. Install `kubelet`
3. Enable IP Forwarding / Kernel parameters
4. Install Container Runtime
5. Initialize master/control plane services (MASTER only)
6. Prepare kubeconfig file to interact with cluster
7. Install CNI - Calico plugin (MASTER only)
8. Create Token if missed: `kubeadm token create --print-join-command`
9. Add workers to the cluster
10. Validation
11. Deploy a test pod to verify setup

---

## Commands and Instructions

### Disable SWAP
```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

### Install kubectl
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

### Enable IP Forwarding and Kernel Parameters
```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```

### Verify Kernel Modules and Sysctl Settings
```bash
lsmod | grep br_netfilter
lsmod | grep overlay
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```

### Install Container Runtime (Docker)
Follow official Docker documentation:
https://docs.docker.com/engine/install/


**After DOCKER installation:**
Add Your User to the Docker Group:
Docker runs its processes as the docker group, and adding your user to this group allows you to run Docker commands without sudo.

Run the following command to add your user to the docker group:

```bash
sudo usermod -aG docker $USER
```
apply the group changes:

```bash
newgrp docker

```
 you're now able to interact with Docker as a non-root user. use docker commands without sudo i.e #docker ps (instead of #sudo docker ps)

 
### Install CRI for Docker
Follow instructions:
https://mirantis.github.io/cri-dockerd/usage/install/

 steps:
```bash
wget <download link for cri-dockerd>
chmod +x v0.4.0
sudo chmod o+r ./cri-dockerd_0.4.0.3-0.ubuntu-jammy_amd64.deb
sudo apt-get install ./cri-dockerd_0.4.0.3-0.ubuntu-jammy_amd64.deb
```

### Install Kubernetes Packages
```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

sudo systemctl enable --now kubelet
```

NOTE : you can specify version if dont want to install latest version #apt-get install -y kubelet="1.33.0-*" kubectl="1.33.0-*" kubeadm="1.33.0-*"

### Check Versions
```bash
kubeadm version
kubelet --version
kubectl version --client
```

### Initialize Master Node
Example (if you have one/single container runtime):
```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=<master-ip> --node-name=kube-master
```

For Docker with cri-dockerd:
```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --cri-socket unix:///var/run/cri-dockerd.sock
```

For containerd runtime:
```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --cri-socket unix:///var/run/containerd/containerd.sock
```



**Note:** Save the `kubeadm join` command output. you need token to join worker with the master / to add worker node in the cluster

### Prepare kubeconfig
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Install Calico Network Plugin (manage networking/routing in kubernetes)
**Modern method (Recommended):**
```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.0/manifests/tigera-operator.yaml
curl -O https://raw.githubusercontent.com/projectcalico/calico/v3.29.0/manifests/custom-resources.yaml
kubectl apply -f custom-resources.yaml
```

**Old Method**: kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.0/manifests/calico.yaml --> **not recommended**
you can try old method if you having issues with the Modren method (using tigera operator) installation  --> but only for non-production environment.


### Join Worker Nodes (Worker node only)
Use the `kubeadm join` command from the master node output.
Example:
```bash
kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

### Validation
```bash
kubectl get nodes
kubectl get pods -A
```

### Quickly Deploy a POD to see if everything is working and are we able to deploy actual workload? :
```bash
#kubectl run mytestpod --image=httpd --restart=Never
```
```bash
#kubectl get pods
```
```bash
#kubectl describe pod mytestpod
```

### Troubleshooting
Reset the cluster if needed:
this command will delete all kubernetes resources/destroy whole cluster, after fixing the issue you can run it to create a cluster from scratch.

```bash
kubeadm reset
```


**check logs and services status:**


- systemctl status kubelet
- systemctl status contaienrd
- systemctl status docker
- journalctl -xeu kubelet      --> #check if there is any errors while starting kubelet
- see if any pod failing i.e coredns, calico etc #kubectl get pods -A
- make sure kubelet service i sup and running before running kubeadm - check its health via endpoint http://127.0.0.1:10248/healthz
- make sure containerd,docker both services are up and running
- make sure worker node can connect to master node on port TCP/6443 ( allow in NSG,OS level Firewall, - use telnet, tracetcp to check connectivity)

```


### Access Cluster from Root User

```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
```


### Set Alias for kubectl command
```bash
alias k='kubectl'
```


### Use Master as a worker node ( remove taint)
```bash
kubectl taint nodes <master node name> node-role.kubernetes.io/control-plane:NoSchedule-
```

## Optional: Use containerd as Container Runtime

### Install containerd
```bash
curl -LO https://github.com/containerd/containerd/releases/download/v1.7.14/containerd-1.7.14-linux-amd64.tar.gz
sudo tar Cxzvf /usr/local containerd-1.7.14-linux-amd64.tar.gz
curl -LO https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
sudo mkdir -p /usr/local/lib/systemd/system/
sudo mv containerd.service /usr/local/lib/systemd/system/
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
sudo systemctl daemon-reload
sudo systemctl enable --now containerd
```

Check containerd status:
```bash
systemctl status containerd
```

### Install runc
```bash
curl -LO https://github.com/opencontainers/runc/releases/download/v1.1.12/runc.amd64
sudo install -m 755 runc.amd64 /usr/local/sbin/runc
```


### POST Validation Steps - quickly deploy a test app and see if everything is working fine.

**Create a Service and see if its reachable over the network:**
```bash
kubectl expose pod mytestpod --type=NodePort --port=8090 --target-port=80 --name=mytestpod-service --selector=run=mytestpod
```


**Test TAINT: After removing -NoSchedule from master node:**
create a deployment ( multiple redundant pods/containers) and see if there is pod scheduled on master node?

```bash
kubectl create deployment my-deployment --image=httpd
kubectl scale deployment my-deployment --replicas=3
```





