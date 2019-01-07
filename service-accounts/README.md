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


