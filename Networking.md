## Basic Commands

```sh
# Find necessary infos
$ ifconfig

cni0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.244.0.1  netmask 255.255.255.0  broadcast 0.0.0.0
        inet6 fe80::c03e:edff:feb7:ecf4  prefixlen 64  scopeid 0x20<link>
        ether c2:3e:ed:b7:ec:f4  txqueuelen 1000  (Ethernet)
        RX packets 2615  bytes 176507 (176.5 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2812  bytes 1118572 (1.1 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.18.0.1  netmask 255.255.255.0  broadcast 172.18.0.255
        ether 02:42:85:ce:3a:6d  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.37  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::42:acff:fe11:25  prefixlen 64  scopeid 0x20<link>
        ether 02:42:ac:11:00:25  txqueuelen 1000  (Ethernet)
        RX packets 41602  bytes 43327640 (43.3 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 19861  bytes 13196237 (13.1 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 211645  bytes 50235982 (50.2 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 211645  bytes 50235982 (50.2 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

veth707c2cbc: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::8ad:ccff:fef2:6ffc  prefixlen 64  scopeid 0x20<link>
        ether 0a:ad:cc:f2:6f:fc  txqueuelen 0  (Ethernet)
        RX packets 2615  bytes 213117 (213.1 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2827  bytes 1119642 (1.1 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

$ iplink show ens3

2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 02:42:ac:11:00:25 brd ff:ff:ff:ff:ff:ff

# network interface = ens3
# ip address = 172.18.0.1
# MAC address = 02:42:ac:11:00:25
# state = UP

# Find ip address of default gateway
$ ip route show default

default via 172.17.0.1 dev ens3

# Find kube-scheduler network info
$ netstat -punlat | grep scheduler

tcp        0      0 127.0.0.1:10259         0.0.0.0:*               LISTEN      2167/kube-scheduler 
tcp6       0      0 :::10251                :::*                    LISTEN      2167/kube-scheduler 
```

## CNI

```sh
# Get CNI plugin path
$ ps -ef | grep cni

root      4587     1  4 13:18 ?        00:00:18 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --cgroup-driver=systemd --network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.1 --cni-bin-dir=/opt/cni/bin # Path here

# CNI Config
$ cat /etc/cni/net.d/10-weave.conflist

{
    "cniVersion": "0.3.0",
    "name": "weave",
    "plugins": [
        {
            "name": "weave",
            "type": "weave-net",
            "hairpinMode": true
        },
        {
            "type": "portmap",
            "capabilities": {"portMappings": true},
            "snat": true
        }
    ]
}
```

## weave-net as CNI Provider

```sh
# Installation
$ k apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

# Check weave ip range for pod
$ ip addr show weave # 10.38.0.0/12

9: weave: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue state UP group default qlen 1000
    link/ether 06:a9:f6:0f:2d:d6 brd ff:ff:ff:ff:ff:ff
    inet 10.38.0.0/12 brd 10.47.255.255 scope global weave
       valid_lft forever preferred_lft forever
    inet6 fe80::4a9:f6ff:fe0f:2dd6/64 scope link 
       valid_lft forever preferred_lft forever
```

## Service Networking

```sh
# IP address range for nodes
$ ip addr show ens3 # 172.17.0.68/16

2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:42:ac:11:00:44 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.68/16 brd 172.17.255.255 scope global ens3
       valid_lft forever preferred_lft forever
    inet6 fe80::42:acff:fe11:44/64 scope link 
       valid_lft forever preferred_lft forever

# IP address range for pods
$ k logs weave-net-vbl87 weave -n kube-system | grep ipalloc-range

ipalloc-range:10.32.0.0/12

# IP address range for services
$ k describe po kube-apiserver-controlplane -n kube-system | grep ip

--service-cluster-ip-range=10.96.0.0/12

# Type of proxy of kube-proxy
$ k logs kube-proxy-qkr25 -n kube-system   

I0427 14:34:22.827695       1 node.go:136] Successfully retrieved node IP: 172.17.0.68
I0427 14:34:22.827872       1 server_others.go:111] kube-proxy node IP is an IPv4 address (172.17.0.68), assume IPv4 operation
W0427 14:34:23.292460       1 server_others.go:579] Unknown proxy mode "", assuming iptables proxy
I0427 14:34:23.292639       1 server_others.go:186] Using iptables Proxier.
```

## DNS in K8s

```
<POD-IP-ADDRESS>.<namespace-name>.pod.cluster.local
<service-name>.<namespace-name>.svc.cluster.local
```

## Ingress

> Beware nginx.ingress.kubernetes.io/rewrite-target: /