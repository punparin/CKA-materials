```sh
endpoint: https://127.0.0.1:2379
key: /etc/kubernetes/pki/etcd/server.key
cert: /etc/kubernetes/pki/etcd/server.crt
ca: /etc/kubernetes/pki/etcd/ca.crt

ETCDCTL_API=3 etcdctl --endpoints 127.0.0.1:2379 \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  snapshot save /opt/etcd-backup.db

ETCDCTL_API=3 etcdctl --write-out=table snapshot status /opt/etcd-backup.db

apiVersion: v1
kind: Pod
metadata:
  name: redis-storage
spec:
  containers:
  - image: redis:alpine
    name: redis-storage
    volumeMounts:
    - mountPath: /data/redis
      name: volume
  volumes:
  - name: volume
    emptyDir: {}
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: use-pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 9Mi
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: use-pv
  name: use-pv
spec:
  containers:
  - image: nginx
    name: use-pv
    volumeMounts:
        - mountPath: "/data"
          name: my-pvc
  volumes:
    - name: my-pvc
      persistentVolumeClaim:
        claimName: use-pv-claim
```

```sh
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: john-developer
spec:
  groups:
  - system:authenticated
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZEQ0NBVHdDQVFBd0R6RU5NQXNHQTFVRUF3d0VhbTlvYmpDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRApnZ0VQQURDQ0FRb0NnZ0VCQUxGejdQSXQrQnArVWwrY3VYbHhCcWRneDM3VHpONXNlbjBLOWprNTRGVTJvcWQzCnR1R25jaEJaRUV6ck0rMjh2YituZlhEeFNOaFFnQW5QNnhhbFY0dFdnNEo4NDAvdmZXeHpQNHdRRVd0YmRyRVgKN0tPUWlwY3I5ZUx4bllFd1dKZ2R6TGdqbnU1MUJWY0lHK1dQN3J0NDI1alBTeDd5MHpKbXB4ZllGM0E5Q21aTgpoNEVkZG9uenhJWGlyV3pUaEdBMkVwWmxqSUVaR1ZJWmhUU1V2WHcwaFBzcGhUZXlJMDV5enVQTDRaSGc0aVdnCmFPbHhmb0FPYTh2Z3lwNExZRmxJK29WeStCVVltd0xCNnhnNlNKS21TdmcwS0xKY29rUHFpei9hOTBNV25BQ0UKTXBRK2Flcy9sVllEQzAzMHhxV3czUUMrQW1Qa2dPRFZnaG1MYTZrQ0F3RUFBYUFBTUEwR0NTcUdTSWIzRFFFQgpDd1VBQTRJQkFRQ3dNbnBsT2tFTFpkZWU1K1VEUVE1Nm5ES3Nzc3V4Q3BoQWtSYkZ4QVhVYkVxNWZhSHpZb3hWClJUTTEyMnVuVGdNRkFWVmJaK0VmelYrZm9yUTlYNjYxd1B5Uit1dU1OTDlzWmdzRnloWFIrYktoUHRYd3lwN0wKVWxHMjZjVktOUExHUnIrdll1RmVQNDU4RjJwK2JBTVRabDNxSlg1WlZkaGsvWkZjc0ptTzBXR0x2bnBlbUVmZgpkTzlLRXVLZUtMd1NPN0g3S04wUktGOXF2dEFKVkxoUHVSSTdsNkhZMU1HZXhrSDNUb0RFTVJPNjRKa2RkbVNqCkRsUlkrMlFuVDlTS1Q5R043TEx3MUcyb2NjVXJXVXgyM05hc1lSeGJYUVdTUVhBL0VFOHZSQU1LREhMN3dReWsKNXc2WGZ6Mmk2NngwQTRrNXA5Nndaa2xHWStoU1NmUysKLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
EOF
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pvviewer-role
rules:
- apiGroups: [""]
  resources: ["persistentvolumes"]
  verbs: ["list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: pvviewer-role-binding
subjects:
- kind: User
  name: pvviewer
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: pvviewer-role
  apiGroup: rbac.authorization.k8s.io
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: john-rolebinding
  namespace: development
subjects:
- kind: User
  name: john
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

```sh
k expose pod nginx-resolver --port=80 --name=nginx-resolver-service
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: multi-pod
  name: multi-pod
spec:
  containers:
  - image: nginx
    name: alpha
    env:
    - name: name
      value: "alpha"
  - image: busybox
    name: beta
    args:
    - sleep
    - "4800"
    env:
    - name: name
      value: "beta"
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: non-root-pod
  name: non-root-pod
spec:
  securityContext:
    runAsUser: 1000
    fsGroup: 2000
  containers:
  - image: redis:alpine
    name: non-root-pod
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ingress-to-nptest
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: np-test-1
  policyTypes:
  - Ingress
  ingress:
  - from:
    ports:
    - protocol: TCP
      port: 80
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: prod-redis
  name: prod-redis
spec:
  nodeName: node01
  containers:
  - image: redis:alpine
    name: prod-redis
  tolerations:
  - key: "env_type"
    operator: "Equal"
    value: "production"
    effect: "NoSchedule"
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-alpha-pvc
spec:
  storageClassName: slow
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-analytics
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: "/pv/data-analytics"
```

```sh
ETCDCTL_API=3 etcdctl --endpoints https://127.0.0.1:2379 \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  snapshot save /opt/etcd-backup.db
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: redis-storage
  name: redis-storage
spec:
  volumes:
  - name: volume
    emptyDir: {}
  containers:
  - image: redis:alpine
    name: redis-storage
    volumeMounts:
    - mountPath: "/data/redis"
      name: volume
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: super-user-pod
  name: super-user-pod
spec:
  containers:
  - args:
    - sleep
    - "4800"
    image: busybox:1.28
    name: super-user-pod
    securityContext:
      capabilities:
        add: ["SYS_TIME"]
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-1
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Mi
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: use-pv
  name: use-pv
spec:
  volumes:
    - name: vol
      persistentVolumeClaim:
        claimName: my-pvc
  containers:
  - image: nginx
    name: use-pv
    volumeMounts:
        - mountPath: "/data"
          name: vol
```

```sh
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: john-developer
spec:
  groups:
  - system:authenticated
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZEQ0NBVHdDQVFBd0R6RU5NQXNHQTFVRUF3d0VhbTlvYmpDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRApnZ0VQQURDQ0FRb0NnZ0VCQUxFWWI5ak9QMFBUSGVNS1kxaFN1aE1DbXBFWXA1SU42ZU5hb211UGVXSTErZ1h6CkV4TVVYZ3RKaHBVSGk0emhXTzc1LzlhTkZ2V2ZreXlMSTFRSURhS1F0Qmo5SVBOS0NLbXRnYStGcTRGYi9HRVQKM1ppa0x0Zkc3eVU0QkRZeHVTa3V3cFFEa1d2TERxa24wdTFVczhUdEZmL213UC9NeGpUd1hhUWVYLzlPNWVuSQppL1YyRmtNdjduUEpGZ2NOZFZnNFJ1OUtldlZkeU9PcitYSjdDU1pRZDBDWTMvZ2FnOC9OdVdTdlh4ajU1UDYvCjFKQWR1YnJDWjl5SWNQUHBXZ3lXK1RTR2FFTkdFcnRGNTZMM1hZbEdPZVRIc0lDZ1AzZUx0bCtFWlNEQXFLemEKMi9IMDlYemdoajhKYXpxY0RmbkdyeW5pNmk3VVNEMnk2TDhBZi9rQ0F3RUFBYUFBTUEwR0NTcUdTSWIzRFFFQgpDd1VBQTRJQkFRQnpTc1A2QUdmTjcxSWdOTCtnMkxoTlUyNkM0VzdRQ21GZkZ6NnZxb2d2K1NYeEZIdGlBWkZxCmJNYjRtR2dpOURYZHFZUFdqVmxrY0xBcHo4TzdJRGJJNkV5cDFoQitraDlFekZVT1FxKzg0OEM3YWpuN3BtZWYKdWtaNndvdnJqU3BZMUVpVXcrRWt5NU5UNTNtaXU3VUpod3ZGeEJhTys2bCsvaU1tc2NHRlQrN082Q1hLanZjRApzczZxZWlFV0UySnlsSlRtQXltMlhld0pjbG9zYk5XdlFaZzh5QnhGYjBXRzVMdU9STG1CWHIyS0FhVXF4OGxlClFHdHpLcGxNT0djWnNLSmp2aHVKWitHUFNhZFB3MFR1c2xvWlgrMTEzTEpqVk9MSGZpSHh2TU9EcGFhSTluY0IKUnRnYy9Xd1d4UnBPcWVjVDJlcllScUJ4QkJBTjVsZjkKLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
EOF

k create role developer -n development --verb create --verb list --verb get --verb update --verb delete --resource pods
kubectl create rolebinding developer --role=developer --user=john
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: pvviewer
  name: pvviewer
spec:
  serviceAccountName: pvviewer
  containers:
  - image: redis
    name: pvviewer
```

```sh
k get node -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}'
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: multi-pod
  name: multi-pod
spec:
  containers:
  - args:
    - sleep
    - "4800"
    image: busybox
    name: beta
    env:
    - name: name
      value: "beta"
  - image: nginx
    name: alpha
    env:
    - name: name
      value: "alpha"
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: non-root-pod
  name: non-root-pod
spec:
  securityContext:
    runAsUser: 1000
    fsGroup: 2000
  containers:
  - image: redis:alpine
    name: non-root-pod
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ingress-to-nptest
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: np-test-1
  policyTypes:
  - Ingress
  ingress:
  - from:
    ports:
    - protocol: TCP
      port: 80
```

```sh
kubectl taint nodes node01 env_type=production:NoSchedule
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: prod-redis
  name: prod-redis
spec:
  containers:
  - image: redis:alpine
    name: prod-redis
  tolerations:
- key: "env_type"
  operator: "Equal"
  value: "production"
  effect: "NoSchedule"
```

