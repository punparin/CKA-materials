## Application

- Check SVC
  - port / targetPort / nodePort
  - selector
- Check deploy / pod
  - volume 
  - env
  - svc domain name
- Use `k logs` to debug error

## Controlplane

```sh
# Check nodes
k get node

# Check kube-system pods
k get po -n kube-system

# Check service status
service kube-apiserver status
service kube-controller-manager status
service kube-scheduler status

## Check service logs
k logs <POD> -n kube-system # Option 1
sudo journalctl -u kube-apiserver # Option 2
```

## Worker Nodes

```sh
# Check master node info
k cluster-info

# Check nodes
k get node

# Check node status
k describe node01

# Check process
top

# check disk space
df

# Check service status
service kubelet status
service kube-proxy status

## Check service logs
k logs <POD> -n kube-system # Option 1
sudo journalctl -u kube-apiserver # Option 2

## Check kubelet config info
cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

# If service stops, start again
service <SERVICE> start

# If service fixs, restart
service <SERVICE> restart

# Check kubelet's certificate
openssl x509 -in /var/lib/kubelet/worker-1.crt -text -out
```

