## Aliases

```sh
echo “source <(kubectl completion bash)” >> ~/.bashrc
alias k=kubectl
complete -F __start_kubectl k
alias kn='kubectl config set-context --current'
alias knc='kubectl config view --minify | grep namespace:'
alias kx='kubectl config use-context'
alias kd='kubectl delete --grace-period=0 --force'
alias kdc='kubectl describe'

vim ~/.vimrc
set nu
set ic
set expandtab
set shiftwidth=2
set tabstop=2

# Go to end of current like → $
# Go to start of first of current line → 0
# Go to the last line of the file → G
# Go to line number → :<desire_line_number>
# Undo the last change → u
# Undo the two last changes → 2u
# Delete contents from cursor to end of file → dG
# Save and quit quickly → ZZ
# Quit without saving → ZQ
# search for “string” → /string
```

## Useful Commands

```sh
# To find sth in a file (not case-sensitive)
grep -i staticpod /var/lib/kubelet/config.yaml
```

## Object Short Name

| Short name | Full name              |
| ---------- | ---------------------- |
| cs         | componentstatuses      |
| ds         | daemonsets             |
| deploy     | deployments            |
| ep         | endpoints              |
| ev         | events                 |
| ns         | namespaces             |
| no         | nodes                  |
| pvc        | persistentvolumeclaims |
| pv         | persistentvolumes      |
| po         | pods                   |
| rs         | replicasets            |
| svc        | services               |

## Manual Scheduling

- Delete pod
- Specify nodeName in YAML
- Create pod again

## Taint

```shell
# Taint
k taint node node1 key1=value1:NoSchedule

# Untaint
k taint node foo dedicated:NoSchedule-
```

## Label

```shell
k label node node01 color=blue
```

## Node Selector

> To place pods in nodes with matched label (only support single label)

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: myapp-pod
spec:
 containers:
 - name: data-processor
   image: data-processor
 nodeSelector:
  size: Large
```

## Node Affinity

> To place pods in a right node specified as conditions in affinity

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: myapp-pod
spec:
 containers:
 - name: data-processor
   image: data-processor
 affinity:
   nodeAffinity:
     requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: In
            values: 
            - Large
            - Medium
```

## DaemonSet

- Create YAML file from Deployment and change to DeamonSet
- Remove startegy in YAML file

## Static Pod

- Ends with -<NODENAME>
- To find static pod path
  - `ps -ef | grep /usr/bin/kubelet ` and find --config=<PATH>
  - staticPodPath: <VALUE>
- Default definition files is located at /etc/kubernetes/manifests

## Configure Multiple Schedulers

- Duplicate /etc/kubernetes/manifests/kube-scheduler.yaml (not as static pod)

- Modify configs in YAML file

  ```yaml
  - --leader-elect=false
  - --scheduler-name=my-scheduler
  ```

- To use custom scheduler

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: annotation-default-scheduler
    labels:
      name: multischeduler-example
  spec:
    schedulerName: default-scheduler
    containers:
    - name: pod-with-default-annotation-container
      image: k8s.gcr.io/pause:2.0
  ```

