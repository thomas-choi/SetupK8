# Setup HA Kubernetes Cluster

This section is talking about the setup of a HA Kubernetes Cluster. It is based on the instructions in the previous section. You can follow the same step of **1. Setup a baseline of Ubuntu VM using VirtualBox** and **2. Installation of Kubernetes steps for each Node**. And here is the steps of creating the HA cluster.

## 3. Use HAProxy as Load Balancer
A HAproxy is setup to act as the entry of the Kubernetes and dispatches the messages to kubeM1 and kubeM2.

 - Install and configure HAProxy
 ```Shell
 apt update && apt install -y haproxy
 ```
Append the below to the configure file */etc/haproxy/haproxy.cfg*
 ```Shell
frontend kubernetes
        bind 192.168.11.156:6443
        option tcplog
        mode tcp
        default_backend kubernetes-master-nodes

backend kubernetes-master-nodes
        mode tcp
        balance roundrobin
        option tcp-check
        server kubeM1 192.168.11.150:6443 check fall 3 rise 2
        server kubeM2 192.168.11.151:6443 check fall 3 rise 2

 ```
Start and enable the HAProxy service:
```Shell
systemctl restart haproxy
systemctl enable haproxy
```

## 4. Initializing Kubernetes

-**kubeadm init the first Master Node**
```Shell
sudo kubeadm init --control-plane-endpoint="192.168.11.156:6443" --upload-certs --apiserver-advertise-address=192.168.11.150 --cri-socket=unix:///var/run/cri-dockerd.sock
```
The output of the command should be kept such that you can use the information to add another Master and Worker nodes. :
```Shell
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

 sudo kubeadm join 192.168.11.156:6443 --token yrmgbu.sccns93idid4oskq \
        --discovery-token-ca-cert-hash sha256:413095cc58ec83ebab13e62806ada8634e8bd8fee479aa485a3e0d06712c707f \
        --control-plane --certificate-key 3d687422431f04c1a4bab69ae1cd302c0664dcb0d20dc01be3896fae4e77bdf1

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

sudo kubeadm join 192.168.11.156:6443 --token yrmgbu.sccns93idid4oskq \
        --discovery-token-ca-cert-hash sha256:413095cc58ec83ebab13e62806ada8634e8bd8fee479aa485a3e0d06712c707f
```

-**Setup the second Master Node**
The command used is from the output of **kubeadm init** on kubeM1. Just remember to add the *--cri-socket* options as cri-dockerd is used.
```Shell
sudo kubeadm join 192.168.11.156:6443 --token yrmgbu.sccns93idid4oskq \
        --discovery-token-ca-cert-hash sha256:413095cc58ec83ebab13e62806ada8634e8bd8fee479aa485a3e0d06712c707f \
        --control-plane --certificate-key 3d687422431f04c1a4bab69ae1cd302c0664dcb0d20dc01be3896fae4e77bdf1  --cri-socket=unix:///var/run/cri-dockerd.sock
```

-**Setup other Worker Node**

```Shell
sudo kubeadm join 192.168.11.156:6443 --token yrmgbu.sccns93idid4oskq \
        --discovery-token-ca-cert-hash sha256:413095cc58ec83ebab13e62806ada8634e8bd8fee479aa485a3e0d06712c707f  --cri-socket=unix:///var/run/cri-dockerd.sock
```

-**Expired Token for joining the Cluster**
I have into a problem when try to add another worker node. I use the below command to check the token in a Master Node.
```Shell
kubeadm token list
```
If the token is not displayed, then it is expired and a new token is needed to add nodes to the cluster. I run the below command in a master node and used the option *--ttl 0* to create a never expired token.
```Shell
kubeadm token create --print-join-command  --ttl 0
```
And the below is output of the command that you can copy the token for the add Master Node and Worker Node
```Shell
kubeadm join 192.168.11.156:6443 --token hslg4k.4vf2e4dn580lw2dr --discovery-token-ca-cert-hash sha256:413095cc58ec83ebab13e62806ada8634e8bd8fee479aa485a3e0d06712c707f
```
Here is the modified command with new Token, for adding new Master Node

```Shell
sudo kubeadm join 192.168.11.156:6443 --token hslg4k.4vf2e4dn580lw2dr \
        --discovery-token-ca-cert-hash sha256:413095cc58ec83ebab13e62806ada8634e8bd8fee479aa485a3e0d06712c707f \
        --control-plane --certificate-key 3d687422431f04c1a4bab69ae1cd302c0664dcb0d20dc01be3896fae4e77bdf1  --cri-socket=unix:///var/run/cri-dockerd.sock
```
and command for adding Worker Node.
```Shell
sudo kubeadm join 192.168.11.156:6443 --token hslg4k.4vf2e4dn580lw2dr\
        --discovery-token-ca-cert-hash sha256:413095cc58ec83ebab13e62806ada8634e8bd8fee479aa485a3e0d06712c707f  --cri-socket=unix:///var/run/cri-dockerd.sock
```