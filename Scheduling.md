## Deploy to Specific Node

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

