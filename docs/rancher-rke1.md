# Rancher on RKE
Steps for configuring lab Rancher on RKE

_Update 2024-07-19_

## Host
192.168.0.20 - rancher2.myrancher.lab

## Install Docker
```
zypper install docker
```
```
systemctl enable --now docker
```

## Access on Docker
```
export RKE_ADMIN_USER=rke-admin; useradd -m -G docker -s /usr/bin/bash -c "Rancher Kubernetes Admin user" $RKE_ADMIN_USER
```
```
su - rke-admin -c "docker version"
```

## Create Keys
```
export RKE_ADMIN_KEY=rke-admin-key
```
```
ssh-keygen -f $RKE_ADMIN_KEY
```
```
su -c "umask 077; mkdir -p ~$RKE_ADMIN_USER/.ssh; cat ~`whoami`/$RKE_ADMIN_KEY.pub >> ~$RKE_ADMIN_USER/.ssh/authorized_keys; chown -R $RKE_ADMIN_USER:users ~$RKE_ADMIN_USER/.ssh" 
```

## Check compatibility Docker and RKE
https://github.com/rancher/rke/releases/

https://www.suse.com/suse-rke1/support-matrix/all-supported-versions/rke1-v1-27/

## Install RKE
```
mkdir -p ~/bin; cd ~/bin; wget https://github.com/rancher/rke/releases/download/v1.4.20-rc3/rke_linux-amd64; chmod +x ~/bin/rke_linux-amd64; ln -s ./rke_linux-amd64 ~/bin/rke; cd ~
```
```
rke config - name cluster.yml

[+] Cluster Level SSH Private Key Path [~/.ssh/id_rsa]: ./rke-admin-key
[+] Number of Hosts [1]:
[+] SSH Address of host (1) [none]: 192.168.0.20
[+] SSH Port of host (1) [22]:
[+] SSH Private Key Path of host (192.168.0.20) [none]: ./rke-admin-key
[+] SSH User of host (192.168.0.20) [ubuntu]: rke-admin
[+] Is host (192.168.0.20) a Control Plane host (y/n)? [y]:
[+] Is host (192.168.0.20) a Worker host (y/n)? [n]: y
[+] Is host (192.168.0.20) an etcd host (y/n)? [n]: y
[+] Override Hostname of host (192.168.0.20) [none]: rancher2.myrancher.lab
[+] Internal IP of host (192.168.0.20) [none]:
[+] Docker socket path on host (192.168.0.20) [/var/run/docker.sock]:
[+] Network Plugin Type (flannel, calico, weave, canal, aci) [canal]: flannel
[+] Authentication Strategy [x509]:
[+] Authorization Mode (rbac, none) [rbac]:
[+] Kubernetes Docker image [rancher/hyperkube:v1.21.5-rancher1]:
[+] Cluster domain [cluster.local]: rancher2.myrancher.lab
[+] Service Cluster IP Range [10.43.0.0/16]:
[+] Enable PodSecurityPolicy [n]:
[+] Cluster Network CIDR [10.42.0.0/16]:
[+] Cluster DNS Service IP [10.43.0.10]:
[+] Add addon manifest URLs or YAML files [no]:
```

## Deploy Rancher
```
rke up
...
INFO[0221] Finished building Kubernetes cluster successfully
```

## Configure kubectl
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"; chmod +x kubectl; mv kubectl /usr/local/bin/
```
```
mkdir -p ~/.kube; cp -p ./kube_config_cluster.yml ~/.kube/config
```
```
kubectl version
```
```
kubectl get nodes
```

## Install Helm
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
```
```
chmod 700 get_helm.sh
```
```
./get_helm.sh
```
```
export CHANNEL=stable
```
```
helm repo add rancher-$CHANNEL https://releases.rancher.com/server-charts/$CHANNEL
```
```
helm repo update
```
```
helm repo list
```

## Create Namespace
```
export RANCHER_NS=cattle-system
```
```
kubectl create ns ${RANCHER_NS}
```
```
kubectl get namespaces
```

## Install Cert-manager
```
export CERT_MANAGER_VERSION=v1.15.1
```
```
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/${CERT_MANAGER_VERSION}/cert-manager.crds.yaml
```
```
helm repo add jetstack https://charts.jetstack.io
```
```
helm repo update
```
```
helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --version ${CERT_MANAGER_VERSION}
```
```
kubectl get pods --namespace cert-manager
```

# Install Rancher
```
export RKE_HOSTNAME=rancher2.myrancher.lab
```
```
export RKE_ADMIN_PASSWORD=YOUR_PASSWORD
```
```
helm install rancher rancher-stable/rancher --version v2.8.1 --namespace cattle-system --set hostname=${RKE_HOSTNAME} --set bootstrapPassword=${RKE_ADMIN_PASSWORD}
```
```
kubectl get pods -n cattle-system
```

Wait a few minutes and click this link https://rancher2.myrancher.lab

# Sources
https://medium.com/@enterco-0/installing-rancher-kubernetes-on-a-single-node-cluster-969584274750

https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

https://helm.sh/docs/intro/install/

https://github.com/cert-manager/cert-manager/releases
