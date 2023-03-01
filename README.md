# SetupK8

This project is seting up a Kubernetes Cluster using VirtualBox VM of Ubuntu 20.04. 

### 1. Setup VirtualBox VM image based on Ubuntu 20.04 LTS
### 2. Prepare the VMs for the installation of kubeadm, 

#### * Install SSH 
```
sudo apt update
sudo apt install openssh-server
```
```
sudo systemctl status ssh
```
Ubuntu ships with a firewall configuration tool called UFW. If the firewall is enabled on your system, make sure to open the SSH port:
```
sudo ufw allow ssh
```
```
ip a
```
```
ssh linuxize@10.0.2.15
```
#### * Set up Docker
```    
sudo apt update  
sudo apt install docker.io -y  
```
```
sudo systemctl enable docker  
sudo systemctl status docker  
sudo systemctl start docker  
```   
### 3. Installation of Kubernetes

    https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
  
#### * Update the apt package index and install packages needed to use the Kubernetes apt repository:
```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
```
#### * Download the Google Cloud public signing key:
```
sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```
#### * Add the Kubernetes apt repository:
```
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

#### * Update apt package index, install kubelet, kubeadm and kubectl, and pin their version:
```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
Note: In releases older than Debian 12 and Ubuntu 22.04, /etc/apt/keyrings does not exist by default. You can create this directory if you need to, making it world-readable but writeable only by admins.

|Hostname|IP	            |MAC	        |Product_uuid	                       |Type	|Memory	|Disk	|CPUs	|open port	|SWAP |
|--------|---------------|-------------|-------------------------------------|-----|-------|-----|-----|-----------|-----|
|kube01	 |192.168.11.150 |080027022924 |29ce83a0-c636-da4e-a813-fec411483eff |     |4G	    |25G	|2		|OFF        | OFF |
|kube02	 |192.168.11.151 |08002772E42B |d4f264b1-111a-7748-9b8c-de41a7c4fc9e |					
|kube03	 |192.168.11.152 |080027EA5479 |cafcc210-fd7d-2244-a94c-707c928d2028 |					
|kube04	 |192.168.11.153 |								
|kube05	 |192.168.11.154 |								
|kube06	 |192.168.11.155 |								

#### Verity Ubuntu Setup for running Kebenetes

#### * Map VM network adaptor
#### * Turn SWAP off, check SWAP by running below
```
sudo swapoff -a
```
#### *  Check SWAP usage
```
free -g
```
#### *  check product_uuid
```
sudo cat /sys/class/dmi/id/product_uuid
```
#### *  Check hostname
```
sudo vi /etc/hostname
sudo vi /etc/hosts
```
#### *  Check required ports:
```
nc 127.0.0.16443
```  
#### *  Installing a container runtime

https://github.com/Mirantis/cri-dockerd 
Install cri-dockerd


Forwarding IPv4 and letting iptables see bridged traffic
https://kubernetes.io/docs/setup/production-environment/container-runtimes/
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```
```
sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```
Verify that the br_netfilter, overlay modules are loaded by running below instructions:
```
	lsmod | grep br_netfilter
	lsmod | grep overlay
```
Verify that the net.bridge.bridge-nf-call-iptables, net.bridge.bridge-nf-call-ip6tables, net.ipv4.ip_forward system variables are set to 1 in your sysctl config by running below instruction:
```
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```
