## Pod

```sh
# Create new pod
k run nginx --image=nginx --labels="app=hazelcast,env=prod" --port=5701

# Create a pod and expose to svc
k run nginx --image=nginx --labels="app=hazelcast,env=prod" --port=5701 --expose

# Create a pod with commands
kubectl run nginx --image=nginx -- sleep 1000

# Force delete pod
k delete po po --grace-period=0 --force

# Use selector to query pods
k get po -l "env=dev"

# Execute command in pod
k exec -it app -n elastic-stack -- sh
```

## Replica Set

```sh
# Scale replicas
k scale --replicas=6 replicaset myapp-replicaset
```

## Deployment

```sh
k create deploy httpd-frontend --replicas=3 --image=httpd:2.4-alpine
```

## Service

```sh
k expose po redis --port=6379 --name redis-service
```

## ConfigMap

```sh
k create configmap webapp-config-map --from-literal=APP_COLOR=darkblue
```

