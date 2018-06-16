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

