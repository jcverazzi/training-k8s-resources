# Exercice : Travailler avec les Pods

Dans cet exercice nous allons manipuler l'objet Pod en utilisant les fichiers de configuration .yaml

Nous allons nous servir de l'application "App, A sample 12 Facter Application",  publiée par Kelsey Hightower (Google)

`https://github.com/kelseyhightower/app`

![Application APP](https://github.com/Treeptik/training-k8s-resources/blob/master/02_Pods/images/Treeptik-training-k8s-exo2-app.jpg?raw=true "Application APP")

Elle est composée : 

- d'un module d'authentification JWT : "auth" 

Image docker ( fournie en deux versions ) : `kelseyhightower/auth` 

- d'un module "hello" - un serveur web qui une fois l'étape d'authentification passée renvoit "hello" 

Image docker ( fournie en deux versions ) : `kelseyhightower/hello`  

- A ces deux modules, se rajoute un frontend Nginx qui route le traffic HTTP aux composants "auth" et "hello"


A la fin de l'exercice nous aurons : 
- Ecrit les fichiers de configuration de chaque Pod correspondant à "auth" et "hello". 
- Lancer les Pods. 
- Inspecter les Pods en utilisant **kubectl** .

Les Pods sont pour le moment indépendants. Dans les parties suivantes nous étudierons les objets qui permettrons d'orchestrer ces Pods entre eux et ainsi de remplir la fonction applicative : retourner "hello" à une requête HTTP d'un utilisateur.     

## Pod "auth"

Le container "auth" n'utilise pas de volume pour du stockage persistant. 

Comme vu pendant la formation, le fichier de configuration du Pod décrit : 
- Les métadatas associées au Pod (name, label)
- Les specifications du Pod : en particulier l'image du container, les ports du container à publier. Egalement il est utile de préciser les ressources (CPU,Memoire) allouable au container. 

Le squellette de fichier de configuration du Pod "auth" est donné ci-après 

```
apiVersion: v1
kind: 
metadata:
  name: 
  labels:
 spec:
  containers:
    - name: 
      image: 
      ports:
        - name: 
          containerPort: 
      resources:
        limits:
```


1. Completer le fichier afin de configurer le Pod tel que : 
- Le nom du Pod est "auth" 
- Le Pod aura les labels "app = auth" et "track = stable"
- L'image du container à déployer est : kelseyhightower/auth:1.0.0
- Le container ecoute HTTP sur le port 80
- BONUS : Le container utilisera un maximum de ressources telles que : 20% de 1 CPU et 20Mi de Mémoire

2. Créer le Pod en fournissant le fichier de configuration .yaml précédent en paramètres 
3. Quelle est l'adresse IP du Pod "auth" ?
4. Sur quel Node le Pod "auth" est-il lancé ? 
5. Quel est le nom du container lancé dans le Pod "auth"
6. Quels sont les labels attachés au Pod

## Pod "hello"

Le container "hello" n'utilise pas de volume pour du stockage persistant. 


7. Créer le fichier .yaml afin de configurer le Pod tel que : 
- Le nom du Pod est "hello" 
- Le Pod aura les labels "app = hello" et "track = stable"
- L'image du container à déployer est : kelseyhightower/hello:1.0.0
- Le container ecoute HTTP sur le port 80 
- BONUS : Le container utilisera un maximum de ressources telles que : 30% de 1 CPU et 50Mi de Mémoire

8. Créer le Pod en fournissant le fichier de configuration .yaml précédent en paramètres 
9. Quelle est l'adresse IP du Pod "hello" ?
10. Sur quel Node le Pod "hello" est-il lancé ? 
11. Quel est le nom du container lancé dans le Pod "hello" ?
12. Quels sont les labels attachés au Pod ?





