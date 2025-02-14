# K3s

## Instalacja

```sh
# PREREQUISITES 
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg 
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server" INSTALL_K3S_VERSION="v1.29.2+k3s1" sh -s - --server "https://$1" --bind-address "$1" --write-kubeconfig-mode 644 --disable traefik,servicelb --node-name k3s-master

```

# ArgoCD

```sh
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl wait --namespace argocd --for=condition=ready pod --selector=app.kubernetes.io/name=argocd-server --timeout=90s

```
