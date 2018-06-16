## Démarrer une image Tomcat

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
kubectl exec mytomcat-66948985cf-q92hq -c shell -i -t -- bash
curl localhost:8080
```

Vous pouvez lister les variables d'environnement :

`kubectl exec mytomcat-66948985cf-q92hq -- printenv`







