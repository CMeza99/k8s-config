# k8s config

These are my test configs for Kubernetes.

The configs were originally obtain by:
```sh
kubeadm init --pod-network-cidr=192.168.0.0/16
```

# Usage

## Prep

```sh
echo 'br_netfilter' >> /etc/modules
modprobe br_netfilter
sysctl net.bridge.bridge-nf-call-iptables=1

swapoff -a

apt-get update && apt-get install -y apt-transport-https ca-certificates
apt install -y ebtables ethtool curl
apt-get install -y linux-image-extra-$(uname -r) linux-image-extra-virtual software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -

add-apt-repository "deb http://apt.kubernetes.io/ kubernetes-$(lsb_release -cs) main"
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

apt-get update && apt-get install -y docker-ce=$(apt-cache madison docker-ce | grep 17.03 | head -1 | awk '{print $3}') kubelet
apt-mark hold docker-ce

usermod -aG docker <docker user>
hostnamectl set-hostname <hostname>

# optional
curl -sS https://s3.amazonaws.com/download.draios.com/DRAIOS-GPG-KEY.public | apt-key add -
curl -sS -o /etc/apt/sources.list.d/draios.list https://s3.amazonaws.com/download.draios.com/stable/deb/draios.list

apt update && apt-get -y install falco sysdig

reboot
```

## Start

On Master:

Get and set configs:
```sh
# copy configs
git clone --depth 1 https://github.com/CMeza99/k8s-config.git
cp -a k8s-config/etc/kubernetes /etc/

# set ip for api
for f in $(grep -lrF '<api_ip>' /etc/kubernetes); do sed -i 's/\<api_ip\>/###\.###\.###\.###/' ${f}; done

###
# TO DO #
###
# create ca
# create certs and keys
# set tokens
  # admin.conf scheduler.conf controller-manager.conf kubelet.conf
## ca
txt=certificate-authority-data; for f in $(grep -lrF ${txt} /etc/kubernetes/); do sed -i "s/${ttt}.*/${txt}:<CERTIFICATE-AUTHORITY-DATA>/" ${f}; done
```

To start using your cluster, you need to run (as a regular user):
```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
echo 'source <(kubectl completion bash)' >> ~/.bashrc
```

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  http://kubernetes.io/docs/admin/addons/

You can now join any number of machines by running the following on each node
as root:
```sh
kubeadm join --token <token> <api_ip>:6443 --discovery-token-ca-cert-hash sha256:<token_hash>
```

## Testing

### ectd

```sh
kubectl exec etcd-k8s-0 etcdctl member list --namespace=kube-system
kubectl exec etcd-k8s-0 etcdctl cluster-health --namespace=kube-system
```

### api

```sh
docker container ls | grep -F apiserver
echo $(curl -ks http://localhost:6443/healthz)
curl -ks http://localhost:6443/api
```

### Validation

```sh
git clone --branch <k8s tag> --depth 1 https://github.com/kubernetes/kubernetes.git cd kubernetes
KUBECTL_PATH=$(which kubectl) NUM_NODES=3 KUBERNETES_PROVIDER=local cluster/validate-cluster.sh
```
