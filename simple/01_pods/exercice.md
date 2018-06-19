## Démarrer une image Tomcat simplement via la ligne de commande

Lancer une instance Tomcat sur le port standard.

`kubectl run mytomcat --image=tomcat:8.0.52-jre8`

Vérifier le status du pod :

`kubectl get pods`

Obtenir l'IP du pod :

`kubectl describe pod mytomcat-66948985cf-q92hq | grep IP:`

Vous pouvez remarquer qu'un **Deployment** vient également d'être créé:

`kubectl get deployment`

Vous pouvez accéder en local à l'application via 

`curl 10.233.69.80:8080`

Vous pouvez également rentrer dans le container et y accéder en local :

```
kubectl exec mytomcat-66948985cf-q92hq  -i -t -- bash
root@mytomcat-66948985cf-q92hq:/usr/local/tomcat# curl localhost:8080
```

Vous pouvez lister les variables d'environnement :

`kubectl exec mytomcat-66948985cf-q92hq -- printenv`

## Déployer un pod sur base d'un fichier 

Créer un fichier `pod-tomcat.yaml` sur base de ceci :

```
apiVersion: v1
kind: Pod
metadata:
  name: mytomcat
spec:
  containers:
  - name: rawtomcat
    image: tomcat:8.0.52-jre8
  - name: shell
    image: centos:7
    command:
      - "bin/bash"
      - "-c"
      - "sleep 10000"
```

Vous remarquerez que l'on crée désormais deux containers au sein du même pod.

```
kubectl exec mytomcat -c shell -i -t -- bash
[root@mytomcat /]# ps -ef
[root@mytomcat /]# curl localhost:8080
```

On remarque bien que l'on est dans le container shell mais que l'on peut accéder en localhost au tomcat.






