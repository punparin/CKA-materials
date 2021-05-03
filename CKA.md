## Aliases

```sh
echo “source <(kubectl completion bash)” >> ~/.bashrc
alias k=kubectl
complete -F __start_kubectl k
alias kn='kubectl config set-context --current --namespace'
alias knc='kubectl config view --minify | grep namespace:'
alias kx='kubectl config use-context'
alias kd='kubectl delete --grace-period=0 --force'
alias kdc='kubectl describe'
alias dk='docker'
export do="-o yaml --dry-run=client"

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

# List running process
ps -ef

# List running containers
dk ps

# Kubeconfig
cat $HOME/.kube/config

# Count the number of po
k get po --no-headers | wc -l

# Find api group of api resources
k api-resources
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

## Formatting

```sh
$ k get deploy -n admin2406 --output=custom-columns="DEPLOYMENT:.metadata.name,CONTAINER_IMAGE:.spec.template.spec.containers.*.image,READY_REPLICAS:.spec.replicas,NAMESPACE:.metadata.namespace"$ 

DEPLOYMENT   CONTAINER_IMAGE   READY_REPLICAS   NAMESPACE
deploy1      nginx             1                admin2406
deploy2      nginx:alpine      1                admin2406
deploy3      nginx:1.16        1                admin2406
deploy4      nginx:1.17        1                admin2406
deploy5      nginx:latest      1                admin2406
```

