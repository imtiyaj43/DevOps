**Kubernetes Nodes** :
In a Kubernetes cluster, you will encounter two categories of nodes:

**Master Nodes** : These nodes play a crucial role in managing the control API calls for various components within the Kubernetes cluster. This includes overseeing pods, replication controllers, services, nodes, and more.

**Worker Nodes** : Worker nodes are responsible for providing runtime environments for containers. It’s worth noting that a group of container pods can extend across multiple worker nodes, ensuring optimal resource allocation and management.

## Step 0 : System Requirements :

### 1. OS : Ubuntu 24.04 LTS Server

### 2. RAM : 2GB or more

### 3. Disk : 20GB+

### 4. Swap Memory : Disable

### 5. Instance Type : t2.medium

### 6. Create 1 Master node and 2 Worker node (based on your usage requirements.)

### 7. Security Ports : Master Node {API Server(6443), Kubelet(10250), ETCD(2379), Scheduler(10251), Controller-Manager(10252)}, Worker Node{Kubelet(10250), Services(30000-32767)}

## Step 1 : Disable Swap Temporarily

```
sudo swapoff -a
```

## Step 2 : Disable Swap Permanently

```
sudo vi /etc/fstab
```

Comment the line : UUID=<some-uuid> none swap sw 0
(If Not show above line, No problem)

## Step 3 : Run on Both Master and Worker Nodes

### 1. Update and upgrade packages:

```
sudo apt update && sudo apt upgrade -y
```

### 2. Check if a reboot is required:

```
sudo cat /var/run/reboot-required
```

If required, reboot the system(sudo systemctl reboot).

### 3. Load necessary kernel modules:

```
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
```

```
sudo modprobe overlay
sudo modprobe br_netfilter
```

### 3.1. Configure Networking:

```
sudo tee /etc/sysctl.d/99-kubernetes-cri.conf <<EOF
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system
```

## Step 4 : Install containerd on both Master and worker

### 4.1. Set up Docker's apt repository:

```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```
```
# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

### 4.2. Install the Docker packages:

```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

### 4.3. Verify installation :

```
sudo systemctl status containerd
```

### 4.4. Configure containerd:

### 4.4.1 : Create the /etc/containerd directory:
```
sudo mkdir -p /etc/containerd
```
### 4.4.2 : Generate the default configuration:

```
sudo containerd config default | sudo tee /etc/containerd/config.toml > /dev/null
```
### 4.5. Update configuration:

```
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
```
### 4.5.1 :Restart containerd
```
sudo systemctl restart containerd
sudo systemctl enable containerd
```

### 4.6. Create crictl configuration file:

```
sudo tee /etc/crictl.yaml <<EOF
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 2
EOF
```

## Step 5 : Install Kubernetes Packages on All Nodes

### 5.1 : Update the apt package index and install packages needed to use the Kubernetes apt repository:

```
sudo apt-get update
# apt-transport-https may be a dummy package; if so, you can skip that package
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```

### 5.2 : Download the public signing key for the Kubernetes package repositories.

```
sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

### 5.3 : Add the appropriate Kubernetes apt repository. 

```
# This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

### 5.4 : Update the apt package index, install kubelet, kubeadm and kubectl, and pin their version:
```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
### 5.5 : This is installation link of Kubernetes package
```
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
```
You can choose kubeadm version from the above official documentation.

## Step 6 : Run on Master Node Only

### 6.1 : Enable IP Forwarding
### 6.1.1 :Check the current value of ip forward:
```
sudo cat /proc/sys/net/ipv4/ip_forward
```
### 6.1.2 : Enable IP forwarding: Edit the /etc/sysctl.conf file:
```
sudo vi /etc/sysctl.conf
```
Find the following line and uncomment it (or add it if it doesn't exist):
```
net.ipv4.ip_forward=1
```
Save and exit the file.
### 6.1.3 : Apply the changes:
```
sudo sysctl -p
```
### 6.2 : Initialize the Kubernetes cluster:
```
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --kubernetes-version=stable-1.30
```
(Save the generated token for worker node joining)(It is simply convenient to reuse the given command, although you can regenerate it and create new tokens using the kubeadm token command. The join tokens expire after 24 hours by default.)

### 6.3 : Deploy Calico Network for pod networking:
```
sudo kubectl create -f https://docs.projectcalico.org/manifests/tigera-operator.yaml
sudo wget https://docs.projectcalico.org/manifests/custom-resources.yaml
sudo kubectl apply -f custom-resources.yaml
```

## Step 7 : Join Worker node
```
sudo <token-generated-from master-node> --v=5
```

## Step 8 : Initialize your user's default kubectl configuration using the admin kubeconfig file generated by kubeadm:
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 8.1 : Run below command on the control-plane to see this node join the cluster
```
kubectl get nodes
```
