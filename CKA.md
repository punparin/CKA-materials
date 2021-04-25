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
alias dk='docker'

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

