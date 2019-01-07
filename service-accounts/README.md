## Service Accounts and Auditing in Kubernetes

ServiceAccounts et Namespaces vous permettent de limiter les pods et définir les permissions utilisateurs dans K8S.
Les logs d'audit donnent une idée sur les activités des comptes sur les ressources.

### Créer le Namespace

Créer le fichier **api-reader-dev-namespace.yaml**

```
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

Puis créer la ressource

```
kubectl create -f api-reader-dev-namespace.yaml 
```

Puis changer le namespace par défaut sur le contexte courant.

```
kubectl config set-context $(kubectl config current-context) --namespace=dev
```

### Créer un secret

Rappel: un secret est encodé sur base64.

Créer le fichier **api-reader-secret.yaml**

```
---
apiVersion: v1
kind: Secret
metadata:
  name: api-access-secret
type: Opaque
data:
  username: YWRtaW4=
  password: cGFzc3dvcmQ=
```

Puis créer le secret:

```
kubectl create -f api-reader-secret.yaml
```

Vérifiez que le secret existe bien.

```
kubectl get secrets api-access-secret
```

### Créer le ServiceAccount

Vous pouvez attacher des comptes de service aux pods et les utiliser pour accéder à l'API Kubernetes. Si aucun compte de service n'est défini dans la définition du pod, celui-ci utilise le compte de service par défaut pour l'espace-noms. 

Les fichiers nommés token, ca.crt et namespace sont automatiquement montés dans le répertoire /var/run/secrets/kubernetes.io/serviceaccount/ de chaque conteneur. 
Leur contenu est basé sur le nom du compte de service que vous avez fourni.

Remarque: les secrets affichés dans le répertoire /var/run/secrets/kubernetes.io/serviceaccount/ sont des secrets spécifiques au compte de service montés par le système Kubernetes, et non le secret que vous avez créé. L'accès à ce secret n'indique pas que le pod peut accéder à d'autres secrets avec ce jeton.

```
---
# Service account for preventing API access
apiVersion: v1
kind: ServiceAccount
metadata:
  name: no-access-sa
---
# Service account for accessing secrets API
apiVersion: v1
kind: ServiceAccount
metadata:
  name: secret-access-sa
```

Puis créer les : 

```
kubectl create -f api-reader-service-accounts.yaml 
kubectl get serviceaccounts
```

### Créer les Cluster Roles

Un ClusterRole définit un ensemble d'autorisations utilisé pour accéder aux ressources, telles que les pods et les secrets. Les ClusterRole sont étendus au cluster. Les ClusterRole définis ici sont attachés aux comptes de service via une liaison de rôle dans les étapes suivantes. 

L'utilisation d'un RoleBinding au lieu d'un ClusterRoleBinding étend les autorisations à un namespace. 

Créer le fichier **api-reader-cluster-roles.yaml**

```
---
# A role with no access
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: no-access-cr
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: [""]
  verbs: [""]

---
# A role for reading/listing secrets
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: secret-access-cr
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
```  

Puis créer les ressources 

```
kubectl create -f api-reader-cluster-roles.yaml
kubectl get clusterroles
```

### Créer les liaisons des rôles

Pour appliquer des rôles de cluster aux comptes de service, créez des liaisons de rôle qui les connectent. Lorsque vous liez un rôle de serveur à un compte de service, les autorisations que vous avez définies dans un rôle sont accordées au compte.

```
---
# The role binding to combine the no-access service account and role
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: no-access-rb
subjects:
- kind: ServiceAccount
  name: no-access-sa
roleRef:
  kind: ClusterRole
  name: no-access-cr
  apiGroup: rbac.authorization.k8s.io

---
# The role binding to combine the secret-access service account and role
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: secret-access-rb
subjects:
- kind: ServiceAccount
  name: secret-access-sa
roleRef:
  kind: ClusterRole
  name: secret-access-cr
  apiGroup: rbac.authorization.k8s.io
```

Créer les ressources 

```
kubectl create -f api-reader-role-bindings.yaml 
kubectl get rolebindings
```

### Créer les pods

Créer le fichier **api-reader-pods.yaml**

```
---
# Create a pod with the no-access service account
kind: Pod
apiVersion: v1
metadata:
 name: no-access-pod
spec:
 serviceAccountName: no-access-sa
 containers:
 - name: no-access-container
   image: ubuntu
---
# Create a pod with the secret-access service account
kind: Pod
apiVersion: v1
metadata:
 name: secret-access-pod
spec:
 serviceAccountName: secret-access-sa
 containers:
 - name: secret-access-container
   image: ubuntu
```

Créer les pods:
```
kubectl create -f api-reader-pods.yaml
kubectl get pods
```

