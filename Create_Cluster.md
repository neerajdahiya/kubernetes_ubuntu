# Create VMs

**Using Virtualbox to create 3 VMs.**

**Some useful tips:**
- *Ensure to provide sufficient storage to each VM*
- *To preserve size choose minimal install option*
- *Name VMs so that it can be easily identified in Virtualbox UI. Like master01_cluster01, worker01_cluster01, etc.*
- *Provide username such that you can easily identify master / worker nodes. Ex: for master01_cluster01 VM give username as master01, and for worker01_cluster01 as worker01*
- ***Use 'Bridged Adapter' under Network options for all VMs. This would allow external DHCP server (ex: router on LAN) to assign IPs to Virtual Machine. Additionally HOST can also be used to deploy applications outside cluster (scenarios like CICD can easily be tested with this setup if HOST has limited resources)***
![image](https://user-images.githubusercontent.com/49958913/131646031-bfe32915-4ad4-496c-acd3-0fffdc18ed4d.png)

- ***Create VM snapshot immediately after clean OS install, this can act as base for clone in case there is need to spawn new VMs (to avoid OS installation steps)***

## Install net-tools
*Required to check interface ip and mac*

```
sudo apt install net-tools
```

## Enable SSH on all Virtual Machines
*Should be enabled for all VMs of cluster so that CLI can be accessed directly from HOST and login to individual VMs is not required. If not done then all CLI commands need to be completed by logging in to individual Virtual Machines*

### Install SSH server

```
sudo apt install openssh-server
```

- **Use ssh from host computer to connect to all machines and execute steps in next points.**
	-This is quite important as if each machine is configured via local terminal then it would be lot of manual overhead (as can't copy data between machines). Also worker node addition would require master node kubeadm command to be executed on all machines.

### Enable Password Authentication
```
vi sshd_config
```
- uncomment PasswordAuthentication yes

### Restart ssh service
```
sudo systemctl restart sshd.service
```

## (Optional) Enable FTP server
*FTP can be enabled on VMs in case file transfer is required between Host -- Guest / Guest -- Guest.*

### Install FTP server
```
sudo apt install vsftpd
```


# Install Docker
- Refer https://docs.docker.com/engine/install/ubuntu/ for official documentation. Install using the convenience script can be also opted (not covered here).
**- Execute "Install Docker" steps on all hosts / VMs that are planned to be added to cluster**

### Uninstall old versions
```
sudo apt-get remove docker docker-engine docker.io containerd runc
```

### Update the apt package index and install packages to allow apt to use a repository over HTTPS
```
 sudo apt-get update

 sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

### Add Docker’s official GPG key
```
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

### Set up the stable repository
```
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Update the apt package index, and install the latest version of Docker Engine and containerd
```
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

### Verify that Docker Engine is installed correctly
```
sudo docker version
sudo docker run hello-world
```

### (Optional) Manage Docker via non-root user
- Offical documentation at https://docs.docker.com/engine/install/linux-postinstall/
- Execute procedure from non-root user

#### Create Docker group
```
sudo groupadd docker
```

#### Add your user to the docker group
```
sudo usermod -aG docker $USER
```
- Log out and log back in so that your group membership is re-evaluated.

#### Verify that you can run docker commands without sudo
```
docker version
docker run hello-world
```

# Install Kubernetes
**- Execute below procedures on master node only unless specified otherwise**
- Start here https://kubernetes.io/docs/setup/production-environment/ for official documentation.
- Installing kubeadm https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/ 

### Disable swap
```
sudo sed -i '/swap/d' /etc/fstab
sudo swapoff -a
```

### Verify MAC and product_uuid uniqueness
- from all nodes in cluster
```
ifconfig -a
```
- execute on all cluster nodes, check mac address of enp0 interface (assuming only one network adapter is enabled for VM). Validate all have different MAC
```
sudo cat /sys/class/dmi/id/product_uuid
```
- execute on all cluster nodes, validate that all VMs have different id

### Let iptables see bridged traffic
  ```
  cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
  br_netfilter
  EOF

  cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
  net.bridge.bridge-nf-call-ip6tables = 1
  net.bridge.bridge-nf-call-iptables = 1
  EOF

  sudo sysctl --system
  ```
## Installing kubeadm, kubelet and kubectl

### Update the apt package index and install packages needed to use the Kubernetes apt repository
```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
```

### Download google public key
```
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```

### Add the Kubernetes apt repository
```
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

### Update apt package index, install kubelet, kubeadm and kubectl, and pin their version
```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

## Creating a cluster with kubeadm
- Official documentation page https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/

### Initializing control-plane node
```
#kubeadm init <args>
kubeadm init --pod-network-cidr=192.168.200.0/23 --ignore-preflight-errors=NumCPU
```
- Refer offical documentation page for various argument details.

### Setup kubeconfig on master node
- Execute as non-root user
```
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
### Install POD network add on
- Calico is being used as network provider.

#### Download Calico YAML file
```
curl https://docs.projectcalico.org/manifests/calico.yaml -o
```

#### Find CALICO_IPV4POOL_CIDR variable in yaml file and replace value with subnet used during kubeadm init command
```
vi calico.yaml
	
#	- find name: CALICO_IPV4POOL_CIDR and update its value attribute to #192.168.200.0/23
#		- name: CALICO_IPV4POOL_CIDR
#		  value: 192.168.200.0/23
```

#### Install Calico in cluster
```
kubectl apply -f calico.yaml
```

#### Verify cluster state
```
kubectl get nodes
```
- worker are yet to be added so output will show master node.

### Add Worker Nodes
- Execute the command shown in kubeadm init output on each worker node as root

```
#kubeadm join <control-plane-host>:<control-plane-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```
