## Openssl

```sh
openssl x509 -in <cert> -text -noout
```

## Debug Certificate Issues

> In case communication between api-server and etcd fails

```sh
# Locate api-server
dk ps | grep kube-apiserver

# View logs
dk logs 5b1ec0f4edf9

# Find static pods location
ps -ef | grep -i kubelet #look for --config
cat /var/lib/kubelet/config.yaml # look for staticPodPath:

# Fix based on certificate issue shown in logs
```

## CSR

```sh
# Get csr as base64
cat akshay.csr | base64 -w 0

# Create CSR
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: akshay
spec:
  groups:
  - system:authenticated
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZqQ0NBVDRDQVFBd0VURVBNQTBHQTFVRUF3d0dZV3R6YUdGNU1JSUJJakFOQmdrcWhraUc5dzBCQVFFRgpBQU9DQVE4QU1JSUJDZ0tDQVFFQXlnVVFYbnBodERuUlNCd1dyZnhSNDdRS1hoZUdVbm1nZFgrTVUzNnRtTis4CkZYeGlLL2hCK3pXUHMrUG9CeEZsZHVEMTZQMWNZOTRNcXhRRDM1LzVROUZ2ZWRpWkZ0dFpXaU1nR2h5QW40WEEKWHJDcXpQOUVYK1pHNlNQSklDbVlpRGRuaGFVQXdsTytjTVNuUnh0eFpvV2xxbWpvQVRVZzRxdCtaMURkOVdsawpWNGZscFY5amRsdXNDMVlIdWZBa0dVa1JoejVRTHpQMFhmb1ZTRFNZdjBzNmM0dERzN2V4ZUhVdWhKcWdhN2hWCno2dXRYS2diZUFHLzM1YjdqZ3Q2VTRWU0VsMXZSeXFuQVZ3K0htbHZBQXZ0SjZZS2pkVEtuM2RoQ25YQWhPWnUKcytyM3VZcHk1OVdFQmx3cFBmTDdXbTg0dXJyemhPaHY0VkptaVhLN2FRSURBUUFCb0FBd0RRWUpLb1pJaHZjTgpBUUVMQlFBRGdnRUJBTG15RDF3WXNtRWxwTDlhdG9FQVE5ZEVyaGkyTkRKRWFCM3JHWnNjWFZCSmF4WVFJM2h4CnZ6aFJJU0c1N0NhTVNNWkRFRkFzd3kvWkVOS2dMSzRFVTVZa1Voc0lSTDZoUnhkSzRyTXRFSkxBRit3bGRGNDMKUUc1eFNSUmw3MWY4a1EvSnBlVDdQT2l6VVcxbk1OR3VFNHdueGpQd1d5eW5OMThuVkxTOVJuOWZTMExyRmhUQgo4akV0cDJkdXhLaE1XRkFHWWVIcUVuZW9wOFFTcmVJLzhsQW84WngyNkV6WmxRTFZQWHErMDJXRWVSWVZ0YVovCk9ibno0RHFPR0NYY05ib2dtMWZoSlg5MEVKQXhSVmowbGFMOW04dXQwQjNpaG9vR1FBcGZCRllRVkZlT092RU4KS0dhZ2FWV294UEgxSXdxZ2FtTGpWT2NlT0J4OHZlTkptL2M9Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
EOF

## To approve CSR
k certificate approve akshay

## To deny CSR
k certificate deny agent-smith
```

## RBAC

```sh
# Run command as another user
k get po --as dev-user

# Check access
k auth can-i create deployments --as dev-user
```

## Role & RoleBinding

> Role & RoleBinding are namespaced objects

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["create", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-user-binding
  namespace: default
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

## ClusterRole & ClusterRoleBinding

> ClusterRole & ClusterRoleBinding are not namespaced objects

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: role-grantor
rules:
- apiGroups: ["rbac.authorization.k8s.io"]
  resources: ["rolebindings"]
  verbs: ["create"]
- apiGroups: ["rbac.authorization.k8s.io"]
  resources: ["clusterroles"]
  verbs: ["bind"]
  # omit resourceNames to allow binding any ClusterRole
  resourceNames: ["admin","edit","view"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: role-grantor-binding
  namespace: user-1-namespace
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: role-grantor
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: user-1
```

## Private Registry

```sh
# Create secret as credentials
k create secret docker-registry private-reg-cred --docker-server=myprivateregistry.com:5000 --docker-username=dock_user --docker-password=dock_password --docker-email=dock_user@myprivateregistry.com
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: private-reg
spec:
  containers:
  - name: private-reg-container
    image: myprivateregistry.com:5000/nginx:alpine
  imagePullSecrets:
  - name: private-reg-cred
```

## Security Context

```sh
# Check user in container
k exec -it ubuntu-sleeper -- whoami
```

```yaml
# Give root permission for specific capabilities
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo-4
spec:
  containers:
  - name: sec-ctx-4
    image: gcr.io/google-samples/node-hello:1.0
    securityContext:
      capabilities:
        add: ["NET_ADMIN", "SYS_TIME"]
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo-2
spec:
  securityContext:
    runAsUser: 1000
  containers:
  - name: sec-ctx-demo-2
    image: gcr.io/google-samples/node-hello:1.0
    securityContext:
      runAsUser: 2000 # This overrides 1000
      allowPrivilegeEscalation: false
```

## Network Policy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978
```



