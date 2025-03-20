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


sudo modprobe overlay
sudo modprobe br_netfilter
```

### 4. Configure Networking:

```
sudo tee /etc/sysctl.d/99-kubernetes-cri.conf <<EOF
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system
```

### 5. Install containerd on All Nodes

### 5.1. Set up Docker's apt repository:

```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

### 5.2. Install the Docker packages:

```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### 5.3. Verify installation :

```
sudo systemctl status containerd
```
