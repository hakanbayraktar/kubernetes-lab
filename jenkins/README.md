Setup Jenkins On Kubernetes Cluster

Deploy and configuring the NFS server:
##on Master Server

apt update && apt -y upgrade
apt install -y nfs-server
mkdir /data
cat << EOF >> /etc/exports
/data 10.116.0.2(rw,no_subtree_check,no_root_squash)
EOF
systemctl enable --now nfs-server
exportfs -ar

Prepare Kubernetes worker nodes:
apt install -y nfs-common

mount 10.116.0.2:/data /data
or add /etc/fstab
10.116.0.2:/data  /data    nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0

1-Create a Namespace
kubectl apply -f namespace.yaml

2-Create a service account with Kubernetes admin permissions.
kubectl apply -f serviceAccount.yaml

3-Create local persistent volume for persistent Jenkins data on Pod restarts.
kubectl create -f volume.yaml

4-Create a deployment YAML and deploy it.
kubectl apply -f deployment.yaml

5-Create a service YAML and deploy it.
kubectl apply -f service.yaml

6-Access the Jenkins application on a Node Port.
http://<node-ip>:32000

kubectl get pods --namespace=devops-tools
kubectl logs jenkins-deployment-2539456353-j00w5 --namespace=jenkins

