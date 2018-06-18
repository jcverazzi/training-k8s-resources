# Exercice : les Services 

Dans cet exercice nous allons créer des fichiers de configuration de type "Service" pour mettre en ligne une application appelée "pokemon".

`https://github.com/sozu-proxy`

A la fin de l'exercice nous aurons : 
- Ecrit les fichiers de configuration pour les services exposant "pikachu, mewtwo, nidoqueen" à une requete HTTP depuis l'internet. 
- Crée les services.
- Abordé les controllers Ingress qui permetront d'exposer les services sur Internet.

On se donne le fichier de deployment suivant : 

`https://github.com/sozu-proxy/sozu-demo/blob/master/kubernetes-using-tube-cheese/pokemon-deployments.yaml`

1. Combien de Deployment vont être lancés ? 
2. Combien de pod vont être lancés pour chaque Deployment? 
3. En analysant les champs {spec.template.spec.container.port} pour chaque type de deployment, déterminer quelle sera la réponse de chaque type de Pod à une requête HTTP port 80. 
4. Lancer les deployments
5. Decrivez chacun des deployments 

En se basant sur le squelette de fichier de configuration du service :  écrire les services pour **mewtwo** et **nidoqueen**

``` 
apiVersion: v1
kind: Service
metadata:
  name: pikachu
spec:
  ports:
  - name: http
    targetPort: 80
    port: 80
  selector:
    app: pokemon
    task: pikachu
```

6. Ecrire le fichier de configuration pour le service pour **mewtwo**  et lancer le service 
7. Ecrire le fichier de configuration pour le service pour **nidoqueen** et lancer le service 
8. Quelle commande utilisez vous pour décrire l'état et les détails du service ? 


__Partie OPTIONNELLE__ : Utilise le "Nginx INGRESS Controler" 


On se donne le fichier de Ingress controller suivant : 

``` 
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: pokemon
spec:
  rules:
  - host: 207.154.253.118.xip.io
    http:
      paths:
      - path: /
        backend:
          serviceName: pikachu
          servicePort: 80
  - host: mewtwo.local
    http: 207.154.253.118.xip.io
      paths:
      - path: /
        backend:
          serviceName: mewtwo
          servicePort: 80
  - host: nidoqueen.local
    http: 207.154.253.118.xip.io
      paths:
      - path: /
        backend:
          serviceName: nidoqueen
          servicePort: 80
```

9. Lancer le Ingress Controller 
10. Quelle commande utilisez vous pour décrire l'état et les détails du Ingress
kubectl describe ing
11. Connectez vous sur http://207.154.253.118.xip.io/pikachu, http://207.154.253.118.xip.io/mewtwo, http://207.154.253.118.xip.io/nidoqueen


