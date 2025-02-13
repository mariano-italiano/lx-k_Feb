# RBAC

## Utworzenie certyfikatów dla użytkownika

1. Generowanie klucza prywatnego i żądania podpisania certyfikatu:
```sh
openssl genrsa -out devops.key
openssl req -new -key devops.key -out devops.csr -subj "/CN=devops"
```

2. Kodowanie certyfikatu typu CertificateSigningRequest:
```sh
cat devops.csr | base64 | tr -d '\n'
```

3. Wygenerowanie obiekt CertificateSigningRequest w Kubernetes, aby podpisać certyfikat (w tym przykładzie wygaśnie on po 10 latach), vi user-request-devops.yml:
```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: user-request-devops
spec:
  groups:
  - system:authenticated
  request: LS0tLS1CRUdJTi...
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 315569260
  usages:
  - digital signature
  - key encipherment
  - client auth
```

```sh
kubectl create -f user-request-devops.yaml
kubectl certificate approve user-request-devops
kubectl get csr 
```

4. Teraz certyfikat powinien zostać podpisany. Możemy pobrać nowy podpisany klucz publiczny z zasobu csr:
```sh
kubectl get csr user-request-devops -o jsonpath='{.status.certificate}' | base64 -d > devops-user.crt
```

## Tworzenie pliku konfiguracyjnego

Utwórz nowy plik konfiguracyjny dla użytkownika `devops`:
```sh
kubectl --kubeconfig config-devops config set-cluster preprod --insecure-skip-tls-verify=true --server=https://192.168.1.100:6443
kubectl --kubeconfig config-devops config set-credentials devops --client-certificate=devops-user.crt --client-key=devops.key --embed-certs=true
kubectl --kubeconfig config-devops config set-context default --cluster=preprod --user=devops
kubectl --kubeconfig config-devops config use-context default
```

## Utworzenie namespace'u oraz obiektów RBAC dla Kubernetesa

1. Tworzenie dedykowanego namepsace'u dla roli:
```sh
kubectl create ns devops-ns
```

2. Utworzenie roli dla usera `devops` w namespace `devops-ns`:
```sh
kubectl create role devops-role --verb=get,list,watch --resource=pods -n devops-ns
kubectl describe role devops-role -n devops-ns
```

3. Utworzenie RoleBinding aby powiązać rolę z użytkownikiem `devops`: 
```sh
kubectl create rolebinding devops-rolebinding --role=devops-role --user=devops -n devops-ns
kubectl describe rolebindings.rbac.authorization.k8s.io -n devops-ns devops-rolebinding 
```

## Przetestowanie uprawnień

Przetestujmy dostęp, aby sprawdzić, czy możemy pomyślnie wylistować Pody, przeczytać logi, a np. usunięcie Poda jest niedozwolone:
```sh
kubectl config use-context default
kubectl -n devops-ns --kubeconfig config-devopstales get pods 
kubectl -n devops-ns --kubeconfig config-devopstales logs xxx
kubectl -n devops-ns --kubeconfig config-devopstales get deploy
kubectl -n devops-ns --kubeconfig config-devopstales delete pod xxx
```
