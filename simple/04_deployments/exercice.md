## Utiliser les Deployments

### Nettoyer l'environnement précédent

`kubectl delete daemonsets,replicasets,services,deployments,pods,rc --all`

## Pourquoi utiliser les Deployments ?

Un ReplicaSet ne peut fournir de service de rolling-update que les ReplicationController savent faire.
Pour faire une action de rolling-update ou de rollout sur un ReplicaSet, il est nécessaire d'utiliser un Deployment de façon déclarative.

## Créer son premier Deployment

Copier le contenu dans le fichier **dep.yaml**

```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: tomcat-deploy
spec:
  replicas: 2
  template:
    metadata:
      labels:
        env: dev
        owner: nicolas
    spec:
      containers:
      - name: sise
        image: tomcat:8.0.52-jre8
        ports:
        - containerPort: 8080
        env:
        - name: TOMCAT_VERSION
          value: "8.0.52"
```

On peut voir toutes les ressources créées :

```
kubectl get all
NAME                                 READY     STATUS    RESTARTS   AGE
pod/tomcat-deploy-7487b7bf6b-7k2cl   1/1       Running   0          18s
pod/tomcat-deploy-7487b7bf6b-md98v   1/1       Running   0          18s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.233.0.1   <none>        443/TCP   1m

NAME                            DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/tomcat-deploy   2         2         2            2           18s

NAME                                       DESIRED   CURRENT   READY     AGE
replicaset.apps/tomcat-deploy-7487b7bf6b   2         2         2         18s
```







