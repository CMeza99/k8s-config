
## Created

As root on Master:
```
kubeadm init --pod-network-cidr=192.168.0.0/16
```

## Usage

On Master:

Set api_ip:
```
for f in $(grep -lrF '<api_ip>' ./etc/); do sed -i 's/\<api_ip\>/###\.###\.###\.###/' ${f}; done
```

To start using your cluster, you need to run (as a regular user):
```
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
```
  kubeadm join --token <token> <api_ip>:6443 --discovery-token-ca-cert-hash sha256:<token_hash>
```
