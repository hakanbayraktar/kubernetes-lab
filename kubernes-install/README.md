#######kubernetes install on Ubuntu

#####Master and Worker Node
1-Containerd install
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf

net.bridge.bridge-nf-call-iptables  = 1

net.ipv4.ip_forward                 = 1

net.bridge.bridge-nf-call-ip6tables = 1

EOF

sudo sysctl --system

apt-get update && sudo apt-get install -y containerd 

mkdir -p /etc/containerd

containerd config default | sudo tee /etc/containerd/config.toml

systemctl restart containerd

2-Disabling swap

swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

3-Install kubeadm, kubelet, kubectl

apt-get update && sudo apt-get install -y apt-transport-https curl

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -

cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list

deb https://apt.kubernetes.io/ kubernetes-xenial main

EOF

sudo apt-get update

sudo apt-get install -y kubelet kubeadm kubectl  

sudo apt-get install net-tools

sudo apt-mark hold kubelet kubeadm kubectl

###Only Master Node

4-Initiate the cluster

sudo kubeadm init --pod-network-cidr 192.168.0.0/16

mkdir -p $HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config

5-Install Calico network add-on.

kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
kubectl get pods -n kube-system

6-Create a join token 
kubeadm token create --print-join-command

###Only worker Node
Adding a worker to the cluster

Copy the join command from the previous step and run it as sudo or root on the worker host

###Only Master Node for verify cluster

kubectl get nodes

