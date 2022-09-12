# Creating K8S Cluster

## Install kubeadm

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl

sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

## Configure cgroup driver as systemd

```
root@instance-1:~/kubeadm# pwd
/root/kubeadm
root@instance-1:~/kubeadm# ls
kubeadm-config.yaml

kind: ClusterConfiguration
apiVersion: kubeadm.k8s.io/v1beta3
kubernetesVersion: v1.25.0
---
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
cgroupDriver: systemd

We will pass it like so : kubeadm init --config kubeadm-config.yaml
```

## Installed containerd

```
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
echo "deb [arch=amd64] https://download.docker.com/linux/debian buster stable" |sudo tee /etc/apt/sources.list.d/docker.list
sudo apt update
sudo apt install containerd

service containerd restart
```

## Setting networking stuff

```
After editing the /etc/sysctl.conf
add the line below
net.bridge.bridge-nf-call-iptables = 1
root@instance-1:/proc/sys/net/core# modprobe br_netfilter
root@instance-1:/proc/sys/net/core# echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
root@instance-1:/proc/sys/net/core# echo 1 > /proc/sys/net/bridge/bridge-nf-call-ip6tables
root@instance-1:/proc/sys/net/core# sudo sysctl -p
net.bridge.bridge-nf-call-iptables = 1
root@instance-1:~/kubeadm# echo 1 > /proc/sys/net/ipv4/ip_forward

```

## Start kubeadm

```
root@instance-1:~/kubeadm# kubeadm init --config kubeadm-config.yaml
```

## Installing calico

https://projectcalico.docs.tigera.io/getting-started/kubernetes/self-managed-onprem/onpremises

```
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.1/manifests/tigera-operator.yaml
curl https://raw.githubusercontent.com/projectcalico/calico/v3.24.1/manifests/custom-resources.yaml -O
kubectl create -f custom-resources.yaml
```
