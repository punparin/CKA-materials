## Install K8s Cluster

```sh
# Install kubeadm, kubelet and kubectl in all nodes
$ sudo -i
$ sudo apt-get update
$ sudo apt-get install -y apt-transport-https ca-certificates curl
$ sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
$ echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | $ $ sudo tee /etc/apt/sources.list.d/kubernetes.list
$ sudo apt-get update
$ sudo apt-get install -y kubelet kubeadm kubectl
$ sudo apt-mark hold kubelet kubeadm kubectl

# Init controlplane node
$ kubeadm init

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

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.17.0.57:6443 --token bo6qxd.mx86vcrwkuh74zgv \
        --discovery-token-ca-cert-hash sha256:2998b4ab6e4a353df0a1b57f3ef2a2d0c9e4d099c857ff8b682e38c0316fbe43
        
# Setup kubeconfig for cluster
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Join worker nodes to the cluster (Run command on worker nodes)
$ kubeadm join 172.17.0.57:6443 --token bo6qxd.mx86vcrwkuh74zgv \
        --discovery-token-ca-cert-hash sha256:2998b4ab6e4a353df0a1b57f3ef2a2d0c9e4d099c857ff8b682e38c0316fbe43
        
# Install network plugin (weave-net)
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

