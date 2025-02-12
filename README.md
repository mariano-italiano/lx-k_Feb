# Instalacja Klastra K8s

## Master node / Control plane node

1. Przygotowanie wstępne systemu --> ip, hostname, /etc/hosts
2. Instalacja oraz konfiguracja systemu pod K8s  --> skrypt do momentu init, czyli wył. swapa, moduły, kernel config, containerd etc.
3. `kubectl init`
4. Skopiowanie configu do `.kube/config`
5. Instalacja pluginu sieciowego `kubectl apply -f <yaml>`

## Worker node / Data plane node

1. Przygotowanie wstępne systemu --> ip, hostname, /etc/hosts
2. Instalacja oraz konfiguracja systemu pod K8s  --> skrypt do momentu init, czyli wył. swapa, moduły, kernel config, containerd etc.
3. `kubectl join`
