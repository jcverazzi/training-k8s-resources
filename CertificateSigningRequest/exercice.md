## CertificateSigningRequest

### Nettoyer l'environnement précédent

`kubectl delete daemonsets,replicasets,services,deployments,pods,rc --all`

### Installer cfssl

```
mkdir ~/bin
curl -s -L -o ~/bin/cfssl https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
curl -s -L -o ~/bin/cfssljson https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
chmod +x ~/bin/{cfssl,cfssljson}
export PATH=$PATH:~/bin
```

### Créer une clés et une demande de certificat

```
echo '{"CN":"treeptik.student","hosts":[""],"key":{"algo":"rsa","size":2048}}' | cfssl genkey  - | cfssljson -bare treeptik.student
```

### Créer la demande de signature du certificat

Créer un fichier certsingnrequest.yaml :
```
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: user-request-treeptik-student
spec:
  groups:
  - system:authenticated
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ2ZqQ0NBV1lDQVFBd0d6RVpNQmNHQTFVRUF4TVFkSEpsWlhCMGFXc3VjM1IxWkdWdWREQ0NBU0l3RFFZSgpLb1pJaHZjTkFRRUJCUUFEZ2dFUEFEQ0NBUW9DZ2dFQkFNcUtSbys3WFZNV3o0bmZ3bVhobHBodDJ1NUdXbitDCkFnbVVMV1hCS2Yyclo3dGh6YjB0VVJIVkdGQlZ1N2R1dTROY1dlWUhBRnoyeWlzVkhXbkdxV2ZrS1RZTEppeGsKUWMzN1h6dW9YMmF6bFZBOGhvUEROMTZYNlZsV1d3K0tyeGpYMTd2Qmp1V05YYmZvNkRldE5TL0RhZTJqTERQUgpWekN4ajVwYWdScVNzMlBQanhjNzVUdkZKZTN3TUxLSmxZZE9kd2gwWHBNY3FwbEg1S2E3c2E5YjNVVHh3SitnCnorWDgxVEVGMGQ3RVJ5RWNEQ3F4dThVb2tuMmQ3NGJjbXJlMk1TVFZENTZLOXFzbGZXZFVnMUNVNUwzN1lBb2wKc25KeEpqajQ2YWhieWUvU0s1R1czRmQ2dFFSblUveklCVlQ1VmlWTk5uNk9USzJrTmxkN3hGOENBd0VBQWFBZQpNQndHQ1NxR1NJYjNEUUVKRGpFUE1BMHdDd1lEVlIwUkJBUXdBb0lBTUEwR0NTcUdTSWIzRFFFQkN3VUFBNElCCkFRQmw2M0JIVVNNbkRsQlROQ3VrZjVNU3hONUxOdHRzcVJSSy9TVEQyc0p6SDZxTnNzaHl3NlFPR0c0V0xCYWQKclExelVXQlVyejZ4b25uVHBTV3psM1NyMUU4TERnZmVhS0JWanpQYVRlRDljSGlINXZLY3EwU3BkOUVSUDRRMQpBQ2lGTFAwZElRL0diWmc4a296NXJvc3U4akpKeWRIYTBIQW9TUXVWNWEyYnNNRi8vK0hRT2RrcGFTNGt0MjdhClpINEpuenRpM0xpemNCOTlIV2ZzV3EvV0V6TmRZV2hMRU5aTkU3NUx5RnJqRjh6eXorcERiemdldXN2T200WVIKdFRoY3czcHlPUXE5dm90dW8wajRGWSsrTnI1TTUxZC9wNFp5VHFKOEFDQVdVcTFISHh2aFBkc0tTdXZ3T2pLTwp0NWEwajBEYVNaTzhpd0hSL0lJNHNQdGgKLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==
  usages:
  - digital signature
  - key encipherment
  - client auth


```

Remplacer la valeur du champs request par le résultat de la commande : 
```
cat treeptik.student.csr | base64 | tr -d '\n'
```

Créer la ressource :
```
kubectl create -f certsignrequest.yaml
```

### Valider le certificat

```
kubectl certificate approve user-request-treeptik-student
```

### Créer un namespace

```
kubectl create namespace treeptik-namespace
```

### Créer un role

Créer un fichier role.yaml :
```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: treeptik-namespace
  name: pod-reader
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["deployments", "replicasets", "pods"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```

Créer la ressource :
```
kubectl create -f role.yaml
```

### Créer un rolebinding

Créer un fichier rolebinding.yaml :
```
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-pod
  namespace: treeptik-namespace # Nom du namespace
subjects:
- kind: User
  name: treeptik.student
  apiGroup: ""
roleRef:
  kind: Role 
  name: pod-reader
  apiGroup: ""
```

Créer la ressource :
```
kubectl create -f rolebinding.yaml
```

### Récuperer le certificat de votre nouvel utilisateur

```
kubectl get csr user-request-treeptik-student -o jsonpath='{.status.certificate}' | base64 -d > treeptik.student.pem
```

### Créer l'utilisateur 

```
kubectl config set-credentials treeptik.student --embed-certs=true --client-certificate=treeptik.student.pem --client-key=treeptik.student-key.pem
```

### Créer le contexte de travail

Récupérer le nom du cluster :
```
kubectl config get-clusters
```

Créer le contexte :
```
kubectl config set-context treeptik-context --cluster=<nom du cluster> --user=treeptik.student --namespace=treeptik-namespace
```

### Tester le nouvel utilisateur 

Changer de contexte :
```
kubectl config use-context treeptik-context
```

Tester si l'utilisateur peut travaillé dans le namespace default ( la réponse devrait être non) :
```
kubectl auth can-i get deployments --namespace default
```

Tester si l'utilisateur peut travaillé dans le namespace treeptik-namespace ( la réponse devrait être oui) :
```
kubectl auth can-i get deployments --namespace treeptik-namespace
```

### Pour créer un fichier kubeconfig personalisé

```
kubectl config set-credentials treeptik.student --kubeconfig treeptik-student-config --embed-certs=true --client-certificate=treeptik.student.pem --client-key=treeptik.student-key.pem
```

```
kubectl config set-context --kubeconfig treeptik-student-config treeptik-context --cluster=<nom du cluster> --user=treeptik.student --namespace=treeptik-namespace
```

```
kubectl config use-context --kubeconfig treeptik-student-config treeptik-context
```

```
kubectl config set-cluster --kubeconfig treeptik-student-config <nom du cluster> --insecure-skip-tls-verify=true --server=<adresse du serveur>
```




