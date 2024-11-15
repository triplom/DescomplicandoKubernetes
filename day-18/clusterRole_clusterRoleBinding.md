# ClusterRole e ClusterRoleBinding

Até agora nós criamos uma Role e uma RoleBinding, que são recursos Namespaced, ou seja, eles só podem ser criados dentro de um namespace, mas e se você quiser criar um usuário que tenha acesso a todos os namespaces do cluster? Para isso, você precisa criar um ClusterRole e um ClusterRoleBinding.

Basicamente eles são iguais a Role e RoleBinding, porém com maior escopo, pois eles refletem em todo o cluster, e não apenas em um namespace específico.

Agora, precisamos criar um novo usuário, que será do time de Platform, e que terá acesso a todos os namespaces do cluster, pois ele precisa criar e gerenciar recursos em todos os namespaces.

Ele precisa ter acesso a `Deployments`, `Services`, `ConfigMaps`, `Secrets`, `Ingress` e `Pods`, por enquanto.

Então vamos começar criando a chave privada e o CSR do usuário:

```bash
openssl genrsa -out platform.key 2048
openssl req -new -key platform.key -out platform.csr -subj "/CN=platform/O=platform"
```

Pegue o conteúdo do arquivo platform.csr, use o base64 para codificar o conteúdo, e cole o resultado no campo request do arquivo platform-csr.yaml:

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: platform
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ2F6Q0NBVk1DQVFBd0pqRVJNQThHQTFVRUF3d0ljR3hoZEdadmNtMHhFVEFQQmdOVkJBb01DSEJzWVhSbQpiM0p0TUlJQklqQU5CZ2txaGtpRzl3MEJBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUFyd3ZaT1NNeUlsTFlUYlR0CjEzYmNMOWtqK0hyOXl5ekpPQnU1Z1hLSUljM3NhdGg1Q1BMOHNvRFN4dzNrbkVOQklsQlE5YlYxNWRJNDQwSnoKdDRDT0ZpR1lTSGt2WGNuVFRGTlpHSlFxemwxZ2JvZ1VldlZxS0tYM0c2ZFhUZ3BrYjNaNTZQRHV2MlF3VlZ6cApYRWE1VlpLZFF6Nmd2LzdFNzV2YWdDOUhxdXVYWWtQYWN4ajRvTERZL2xxNzRpMlBlb1B2L0VNd2NsVEpSV01NCk5ESmNDZmIyLzl3bU1vVEg3N3BpYUNaRlBGWG1XOStHcmhrcGY5VDNua1dST2doL2s3b2RBcjc4ekNUVTZ3a0MKa0p3QWJNR1lkaEpaZHpJLzZWTVJPUHhxV01RckVzR005MzkzNUNPSmZVYlRzOEoreEZQb2l2eWJPbXdzQWFYMgpNM0U4clFJREFRQUJvQUF3RFFZSktvWklodmNOQVFFTEJRQURnZ0VCQUdTUlRtVEU1bjBIK2VFZ2ZFdmdqWFRxCitucmdlSEJka3JvK3BlMTFOM1pONjVubVhaWEQ3cFJ0U3RZamtma2p1RG1OdDcrN0Q0NW1hOXEyeDZ1aGV2eFkKMWhCNEcxOWNCS0dTVWM1QzlBbnJNemtsY29Ic1JTYjhwZkhXcmNpak4rVEQ1MkNZWC9XMVEyY28xQ2pTMWI1bgpTWG8rRE1vdkZRVHhDbnFGdTVCMVlsVXdGWkhDVW5qOXp6SUJjQmRBekUzT3VZSVdyYWFFOXNNZUU5ZC9OVE11CkVvL3lSVVJ0THlaQUtWc3NTQ2k2NEk0dVlGV3Y3WXlKbG00RTVuUDQ1YUhJd1pzdVFlZ2RwOEZhOVVZVFBLaksKbmVjWEJGdVlhekRxYlRiWHhRN1QvUW44VWhZbGIybVZ3UUpsbUpoeTEweXYwZW1DeTVPVGFDZ2ZiUlh5bk9rPQotLS0tLUVORCBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0K
 signerName: kubernetes.io/kube-apiserver-client
 expirationSeconds: 31536000 # 1 year
 usages:
 - client auth
```

Agora vamos aplicar:

```bash
kubectl apply -f platform-csr.yaml
```

Para verificar se o CSR foi criado com sucesso, vamos listar os CSR's do cluster:

```bash
kubectl get csr
```

```text
A saída:

NAME        AGE   SIGNERNAME                                    REQUESTOR                 REQUESTEDDURATION   CONDITION
csr-648l9   82m   kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:abcdef   <none>              Approved,Issued
csr-hn4d4   82m   kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:abcdef   <none>              Approved,Issued
csr-qd72l   82m   kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:abcdef   <none>              Approved,Issued
csr-r24pf   82m   kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:abcdef   <none>              Approved,Issued
csr-rswk4   82m   kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:abcdef   <none>              Approved,Issued
developer   79m   kubernetes.io/kube-apiserver-client           kubernetes-admin          365d                Approved,Issued
platform    87s   kubernetes.io/kube-apiserver-client           kubernetes-admin          365d                Pending
```

Vamos aprovar o CSR:

```bash
kubectl certificate approve platform
```

Agora vamos pegar o certificado do usuário:

```bash
kubectl get csr platform -o jsonpath='{.status.certificate}' | base64 --decode > platform.crt
```

Eu não vou explicar novamente tudo o que estamos fazendo, pois boa parte é igual ao que já fizemos anteriormente, então se você não se lembra, volte e leia novamente. 😄

Pronto, o que está faltando agora são os recursos ClusterRole e ClusterRoleBinding, então vamos criar o arquivo platform-clusterrole.yaml com o seguinte conteúdo:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: platform
rules:
- apiGroups: ["", "apps"] # "" indicates the core API group
  resources: ["deployments", "pods"]
  verbs: ["get", "list", "create"]
```

Como pode ver, a configuração é muito parecida com a Role, porém agora estamos criando um ClusterRole. A diferença na definção da Role e do ClusterRole é que o ClusterRole o Kind é ClusterRole, e também não tem o campo namespace, pois o ClusterRole não é Namespaced.

Agora vamos criar o arquivo platform-clusterrolebinding.yaml com o seguinte conteúdo:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: PlatformClusterRoleBinding
subjects:
- kind: User
  name: platform
roleRef:
  kind: ClusterRole
  name: platform
  apiGroup: rbac.authorization.k8s.io
```

Mesma coisa aqui, a configuração é super parecida com a RoleBinding, mudando somente o Kind, que agora é ClusterRoleBinding, e também não tem o campo namespace.

Bora aplicar os arquivos:

```bash
kubectl apply -f platform-clusterrole.yaml
kubectl apply -f platform-clusterrolebinding.yaml
```

Para verificar se os recursos foram criados com sucesso, vamos listar os ClusterRoles e ClusterRoleBindings do cluster:

```bash
kubectl get clusterroles
kubectl get clusterrolebindings
```

Estão criados!

Agora vamos adicionar o certificado do usuário no kubeconfig:

```bash
kubectl config set-credentials platform --client-certificate=platform.crt --client-key=platform.key --embed-certs=true
```

Falta agora criar o contexto para o usuário:

```bash
kubectl config set-context platform --cluster=NOME-DO-CLUSTER --user=platform
```

E já era! Agora vamos testar o acesso ao cluster:

```bash
kubectl config use-context platform
kubectl get pods --all-namespaces
```

Tudo funcionando lindamente!

Agora vamos tentar listar os Nodes do cluster:

```bash
kubectl get nodes
```

O resultado será algo parecido com isso:

```text
Error from server (Forbidden): nodes is forbidden: User "platform" cannot list resource "nodes" in API group "" at the cluster scope
Ou seja, o usuário platform não tem permissão para listar os Nodes do cluster, pois ele só tem permissão para listar os recursos deployments e pods conforme definido na nossa ClusterRole.
```

Simples como voar!

## ClusterRole e ClusterRoleBinding para o usuário admin

Agora vamos criar um usuário que terá acesso total ao cluster, ou seja, ele terá acesso a todos os recursos do cluster, e a todos os namespaces do cluster.

Primeiro vamos criar a chave privada e o CSR do usuário:

```bash
openssl genrsa -out admin.key 2048
openssl req -new -key admin.key -out admin.csr -subj "/CN=admin/O=admin"
```

Converta o conteúdo do arquivo admin.csr para base64, e cole o resultado no campo request do arquivo admin-csr.yaml:

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
 name: admin
spec:
 request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1pUQ0NBVTBDQVFBd0lERU9NQXdHQTFVRUF3d0ZZV1J0YVc0eERqQU1CZ05WQkFvTUJXRmtiV2x1TUlJQgpJakFOQmdrcWhraUc5dzBCQVFFRkFBT0NBUThBTUlJQkNnS0NBUUVBb2NtUUxVdVc1TEpKM0p4YmZFdmY1a2xGClRCdWZrb2h3SFBiS1FVYVdmNWErc3FrRzBlS2NSMXplVmhwZXNnYUh3WG9uRnVYakdzc3BHaHJnL21XcWFQSG4KNU14SE1uc1lmWE9VV2diOWIrT3VtRFV1Qmh3eE5EL0c1eSswMmhORUIybzdaTzltcUlCNkdZWlo2d3dHQ01vSgpVVGFzM2tCNVBXaUg4eXFISDZhdFhPNlBteG5Ba0Iyb3praHQ0NWp0ZlNVeDI1UzQvN00zVTVwSjhOTG9jU3N1CjE3d0wvRWdhQitpaThVa0t3MnNvZlZTM2hwQlNxNVNNbzNqZ2dCQ3R3eWN0UzhOWG9wb2M4Rlg5SzFWeFBGMXEKSnpGK2N2UFZVZHNWUUNJWm45Vkt6ZTV4T2JBT2ZYL2hrallrdFhJS0VIU2dIYTZvTXdOT0hXWlBxQVUrVndJRApBUUFCb0FBd0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFJeFJJeVN6TzY0Vi95ZHRoNVVSS09IcXdBM3JKb2U0CnJFc3A3Nkp2VTNjcVFtUVVFem9nR2RzdHMxWEFGVzA5cGJxZTdJa1pXTktOemp2NjZyTThkeDBnazRpSWdCdGkKeXovWDI5ZE5KUEVqRCthTW1sSWJ1VEpnZkpSYmx0a1lDYVpPN2tqWUJlMXRNNWE5U0dCQXg3bkt0RnlTNzArMwpTN2VaQmhYNlY2K080Z1FsZlEzT1ZxMzlzSzN5QnBPallYTDQxak5HZmh0TkRTcjNMWExodzd2MjhJb2xUZXQ5CnZJSUw0S3hjQ3kvUTROUTFXSnlKdkpWODZJS01BT2tabjJtUnY0UWFoUEVRM1grS21RbmxpQ01QYnFic2l3czUKL1pTKzA1QUp2QytLalhjcXpHZ016V21rbDlZOXV4RENVVE5paXdmQkJiOGhvRW9PV2lzcENHcz0KLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==
 signerName: kubernetes.io/kube-apiserver-client
 expirationSeconds: 31536000 # 1 year
 usages:
 - client auth
```

Agora vamos aplicar:

```bash
kubectl apply -f admin-csr.yaml
```

Hora de aprovar o CSR:

```bash
kubectl certificate approve admin
```

Agora vamos pegar o certificado do usuário:

```bash
kubectl get csr admin -o jsonpath='{.status.certificate}' | base64 --decode > admin.crt
```

Agora bora criar o arquivo admin-clusterrole.yaml com o seguinte conteúdo:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: admin
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["*"]
  verbs: ["*"]
```

Perceba que nesse caso, nós estamos utilizando o * para indicar que o usuário terá acesso a todos os recursos do cluster, e a todos os namespaces do cluster.

Agora vamos criar o arquivo admin-clusterrolebinding.yaml com o seguinte conteúdo:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: AdminClusterRoleBinding
subjects:
- kind: User
  name: admin
roleRef:
  kind: ClusterRole
  name: admin
  apiGroup: rbac.authorization.k8s.io
```

Agora vamos aplicar os arquivos:

```bash
kubectl apply -f admin-clusterrole.yaml
kubectl apply -f admin-clusterrolebinding.yaml
```

Para verificar se os recursos foram criados com sucesso, vamos listar os ClusterRoles e ClusterRoleBindings do cluster:

```bash
kubectl get clusterroles
kubectl get clusterrolebindings
```

Estão lá! Hora de adicionar o certificado do usuário no kubeconfig:

```bash
kubectl config set-credentials admin --client-certificate=admin.crt --client-key=admin.key --embed-certs=true
```

E criar o contexto para o usuário:

```bash
kubectl config set-context admin --cluster=NOME-DO-CLUSTER --user=admin
```

Fim! hahahah

Agora é só testar, então vamos:

```bash
kubectl config use-context admin
kubectl get pods --all-namespaces
```

Você pode usar um comando super útil para saber tudo o que o usuário pode fazer no cluster:

```bash
kubectl auth can-i --list
```

A saída é gigante, então vou colocar somente o começo dela:

```text
Resources                                        Non-Resource URLs   Resource Names   Verbs
leases.coordination.k8s.io                       []                  []               [create delete deletecollection get list patch update watch]
rolebindings.rbac.authorization.k8s.io           []                  []               [create delete deletecollection get list patch update watch]
roles.rbac.authorization.k8s.io                  []                  []               [create delete deletecollection get list patch update watch]
configmaps                                       []                  []               [create delete deletecollection patch update get list watch]
events                                           []                  []               [create delete deletecollection patch update get list watch]
persistentvolumeclaims                           []                  []               [create delete deletecollection patch update get list watch]
...
```

Um bom teste é você comparar com o usuário developer e com o usuário platform, assim você poderá ver a diferença entre eles do que eles podem fazer no cluster.

Acho que deu pra entender bem como funciona o RBAC, e como criar usuários com diferentes níveis de acesso ao cluster, e como criar Roles e ClusterRoles para diferentes perfis de usuários.

Agora é só praticar!

## Removendo o usuário

Para remover o usuário é super simples, basta remover o CSR e o RoleBinding relacionado ao usuário.

```bash
kubectl delete csr NOME-DO-CSR
kubectl delete rolebinding NOME-DO-ROLEBINDING
```

E para remover do seu kubeconfig, basta utilizar o comando kubectl config unset:

```bash
kubectl config unset users.NOME-DO-USUARIO
```

Pronto, usuário removido!
