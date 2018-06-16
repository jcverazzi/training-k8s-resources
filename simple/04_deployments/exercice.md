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
        - name: URL_WS
          value: "https://foo.01.ws.com"
```

On peut voir toutes les ressources créées :

```
kubectl get all
NAME                                 READY     STATUS    RESTARTS   AGE
pod/tomcat-deploy-5777588498-lxm4q   1/1       Running   0          6s
pod/tomcat-deploy-5777588498-qrbvs   1/1       Running   0          6s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.233.0.1   <none>        443/TCP   14s

NAME                            DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/tomcat-deploy   2         2         2            2           6s

NAME                                       DESIRED   CURRENT   READY     AGE
replicaset.apps/tomcat-deploy-5777588498   2         2         2         6s
```

Editer le fichier "dep.yaml" pour y changer de la variable d'environnement URL_WS

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
        owner: nobody
    spec:
      containers:
      - name: sise
        image: tomcat:8.0.52-jre8
        ports:
        - containerPort: 8080
        env:
        - name: URL_WS
          value: "https://foo.02.ws.com"
```

Pour déployer la modification, vous ne devez pas utiliser la commande **create** mais bien **apply**.

```
kubectl apply -f dep.yaml
deployment.apps "tomcat-deploy" configured

kubectl get pods
NAME                             READY     STATUS              RESTARTS   AGE
tomcat-deploy-7d54877fdd-blrg2   1/1       Running             0          2s
tomcat-deploy-7d54877fdd-snnws   0/1       ContainerCreating   0          1s
tomcat-deploy-849cd86d55-8zmmb   1/1       Terminating         0          2m
tomcat-deploy-849cd86d55-jdm8h   1/1       Running             0          2m

kubectl get pods
NAME                             READY     STATUS        RESTARTS   AGE
tomcat-deploy-7d54877fdd-blrg2   1/1       Running       0          3s
tomcat-deploy-7d54877fdd-snnws   1/1       Running       0          2s
tomcat-deploy-849cd86d55-8zmmb   0/1       Terminating   0          2m
tomcat-deploy-849cd86d55-jdm8h   1/1       Terminating   0          2m

kubectl get pods
NAME                             READY     STATUS    RESTARTS   AGE
tomcat-deploy-7d54877fdd-blrg2   1/1       Running   0          1m
tomcat-deploy-7d54877fdd-snnws   1/1       Running   0          1m
```








