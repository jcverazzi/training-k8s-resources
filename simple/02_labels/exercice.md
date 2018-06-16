## Travail sur les labels

### Nettoyer l'environnement précédent

`kubectl delete daemonsets,replicasets,services,deployments,pods,rc --all`

### Création d'un nouveau tomcat

Créer un nouveau fichier **pod-tomcat-label.dev.yaml**

````
apiVersion: v1
kind: Pod
metadata:
  name: mytomcat
  labels:
    env: development
spec:
  containers:
  - name: rawtomcat
    image: tomcat:8.0.52-jre8
````

Afficher désormais les labels associés aux pods :

````
kubectl get pods mytomcat --show-labels
NAME       READY     STATUS    RESTARTS   AGE       LABELS
mytomcat   1/1       Running   0          35s       env=development
````

Ajouter un nouvel label au pod 

```
kubectl label pods mytomcat owner=nicolas

kubectl get pods mytomcat --show-labels
NAME       READY     STATUS    RESTARTS   AGE       LABELS
mytomcat   1/1       Running   0          2m        env=development,owner=nicolas

```

Afficher seulement les pods ayant un certain label :

````
kubectl get pods -l env=development
NAME       READY     STATUS    RESTARTS   AGE
mytomcat   1/1       Running   0          3m
```

