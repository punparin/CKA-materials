## Drain node

```sh
# Drain node (+ cordon)
k drain node01 --ignore-daemonsets

# Cordon node (mark as unschedulable)
k cordon node01

# Uncordon node (mark as schedulable)
k uncordon node01
```

## Upgrade Controlplane

```sh
# Check version to upgrade to
kubeadm upgrade plan

# Drain controlplane
k drain controlplane --ignore-daemonsets

# Upgrade kubeadm
apt-get update && \
apt-get install -y --allow-change-held-packages kubeadm=1.19.0-00

# Verify version
kubeadm version

# Upgrade node's version
sudo kubeadm upgrade apply v1.19.0

# Upgrade kubelet & kubectl
apt-get update && \
apt-get install -y --allow-change-held-packages kubelet=1.19.0-00 kubectl=1.19.0-00

# Restart kubelet
sudo systemctl daemon-reload
sudo systemctl restart kubelet

# Mark as schedulable
k uncordon controlplane
```

## Upgrade Worker node

```sh
# SSH
ssh node01

# Upgrade kubeadm
apt-get update && \
apt-get install -y --allow-change-held-packages kubeadm=1.19.0-00

# Upgrade local kubelet config
sudo kubeadm upgrade node

# Upgrade kubelet & kubectl
apt-get update && \
apt-get install -y --allow-change-held-packages kubelet=1.19.0-00 kubectl=1.19.0-00

# Restart kubelet
sudo systemctl daemon-reload
sudo systemctl restart kubelet

# Exit node01
exit

# Mark as schedulable
k uncordon node01
```

## ETCD Backup and Restore

```sh
# Check necessary infos
k describe po etcd-controlplane -n kube-system

# endpoints= --listen-client-urls
# cert= --cert-file
# cacert= --trusted-ca-file
# key= --key-file

# Backup ETCD's snapshot
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /opt/snapshot-pre-boot.db
  
# Restore ETCD's snapshot
ETCDCTL_API=3 etcdctl --data-dir /backup \
	snapshot restore /opt/snapshot-pre-boot.db
	
# Find static pods location
ps -ef | grep -i kubelet #look for --config
cat /var/lib/kubelet/config.yaml # look for staticPodPath:

# Change dir to static pod path
cd /etc/kubernetes/manifests

# Modify etcd.yaml
vim etcd.yaml

​```
  volumes:
  - hostPath:
      path: /backup
      type: DirectoryOrCreate
    name: etcd-data
​```
```

