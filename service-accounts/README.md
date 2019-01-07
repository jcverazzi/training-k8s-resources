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
````

Puis créer la ressource

```
kubectl create -f api-reader-dev-namespace.yaml 
```


