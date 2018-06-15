# Exercice : les Services 

Dans cet exercice nous allons créer des fichiers de configuraton de type "Service" pour mettre en ligne  une application appellée "pokemon" - en réalité qui permet de présenter une démo de Sozu Proxy 

`https://github.com/sozu-proxy`

A la fin de l'exercice nous aurons : 
- Ecrit les fichiers de configuration pour les services exposant "pikachu, mewtwo, nidoqueen" à une requete HTTP depuis l'internet. 
- Crée les services.
- Abordé les controllers Ingress qui permetront dexpposer les services sur Internet.

On se donne le fichier de deployments suivant : 

1. Combien de Deployments vont être lancé ? Détaillez votre réponse. 
2. Combien de pod vont être lancé ? Détaillez votre réponse.
3. En analysant les champs {spec.template.spec.container.port} pour chaque type de deployment, déterminer quelle sera la réponse de chaque type de Pod à une requête HTTP port 80 ? 
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


On se donne le fichier de Ingress controller suivant : 

``` 
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: pokemon
spec:
  rules:
  - host: pikachu.local
    http:
      paths:
      - path: /
        backend:
          serviceName: pikachu
          servicePort: 80
  - host: mewtwo.local
    http:
      paths:
      - path: /
        backend:
          serviceName: mewtwo
          servicePort: 80
  - host: nidoqueen.local
    http:
      paths:
      - path: /
        backend:
          serviceName: nidoqueen
          servicePort: 80
```

9. Lancer le Ingress Controller 
10. Déterminer sur quelle IP publique l'application est lancée 
11. Connectez vous sur http://ip_publique/pikachu, http://ip_publique/mewtwo, http://ip_publique/nidoqueen


