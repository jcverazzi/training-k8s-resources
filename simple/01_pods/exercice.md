## Démarrer une image Tomcat

Lancer une instance Tomcat sur le port standard.

`kubectl run mytomcat --image=tomcat:8.0.52-jre8 --port 8080`

Vérifier le status du pod :

`kubectl get pods`

Obtenir l'IP du pod :

`kubectl describe pod mytomcat-66948985cf-q92hq | grep IP:`


