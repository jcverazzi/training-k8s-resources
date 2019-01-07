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



