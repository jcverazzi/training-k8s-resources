# Exercice : le Controleur Deployment

Dans cet exercice nous allons créer des fichiers de configuraton de type "Deployment" pour chaque service de l'application  "App, A sample 12 Facter Application", publiée par Kelsey Hightower (Google). 

`https://github.com/kelseyhightower/app`


A la fin de l'exercice nous aurons : 
- Ecrit les fichiers de configuration pour "auth" et "hello" ainsi que celui du frontend Nginx "frontend"
- Crée le Deployment pour tous les composants le l'application APP dans sa version 1.0.0
- Scalé le Deployment de l'application APP version 1.0.0 
- Mettre a jour - update - de la version de l'Application APP dans sa version 2.0.0 

![Application APP](https://github.com/Treeptik/training-k8s-resources/blob/master/04_Deployment/images/Treeptik-training-k8s-exo4-1.jpg?raw=true "Application APP")


Les différentes parties ne seront pas exposées : 
- sur Internet pour le frontend, 
- en interne pour les composants "auth" et "hello". 

L'Application APP n'est pas encore en mesure reçevoir et de traiter les requêtes utlisateurs : cette thématique sera abordée dans la partie dédiée aux Services.  

  
## Introduction : 

Les pratiques DevOps impliquent de pouvoir déployer de nouvelles versions applicatives à la volée c'est à dire sans coupure de service. 

L'objet Deployment se positionne pour répondre à ce besoin, en adoptant le point de vue suivant : 
- La stack applicative est composées de Pods, 
- L'update de la stack applicative consiste à l'update des containers des Pods, 
- Un ReplicatSet supervise les Pods, 
- L'Objet Deployment pilote le ReplicatSet des Pods - notamment pendant la mise à jour de la stack applicative. Cette partie est importante : le Deployment utilise son ReplicatSet pour transmettre les ordres à l'API Kubernetes. 

En definisant un objet Deployment avec son fichier de configuration .yaml, on définit l'état du Pod à atteindre, en particulier au niveau de versionning. Le  Deployment assure la transition vers le nouvel état de telle manière à ne pas couper le service applicatif. 

Grâce à l'objet Deployment on peut par exemple : 
- Créer from scracth une stack applicative dans sa version 1
- Scaler le deployment version 1 pour s'adapter à la charge applicative. 
- Mettre à jour cette stack applicative dans une version 2 
- Mettre en pause / reprendre un deployement.... 


### Etude d'un fichier de configuration de Deployment

On se donne le fichier suivant :

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: soaktest
spec:
  replicas: 5
  selector:
    matchLabels:
      app: soaktest
  template:
    metadata:
      labels:
        app: soaktest
    spec:
      containers:
      - name: soaktest
        image: nickchase/soaktest
        ports:
        - containerPort: 80
```

Comme tous les objets de l'API Kubernetes, un Deployment est defini avec les champs **apiVersion**, **kind**, et **metadata**. A cela on ajoute le champs **spec** qui décrit les spécifications d'un Pod à déployer. 

- On retrouve le champs **.spec.selector.** introduit avec l'objet ReplicatSet. Ce champ sera transmit au ReplicatSet associé à ce Deployment. 
- Comme pour le ReplicaSet la bonne pratique est d'utiliser le "Label Selector" et de le préférer aux "Labels" simple présent dans le champs **.spec.template.metadata.labels** . 
- Enfin, l'objet Deployment est à préférer au ReplicatSet car il permet de gérer le cycle de vie de l'application au sein de l'usine logielle : update, rollback de versions... 

1. Donner le nombre de POD qui sera lancé lors de la creation du Deployment avec ce fichier de configuration
2. Trouver la commande kubectl pour créer ce Deployment. 
3. Quelle Image Docker est utilisée  ? 
4. Quel sont les labels associés au Pod qui sera orchestré par ce Deployment ? 
5. Quel est le selector pour ce Deployment ? A comparer avec la réponse du 4. 


### Configuration du Deployment pour les Pods "auth", "hello" et "frontend". 

Pour construire les fichiers de configuration des 3 Deployments nous allons nous appuyer sur les fichiers de configuration des Pod "auth", "hello" et "frontend". 
Ces fichiers ont été explicité dans l'exercice 03_ReplicatSet. On les rappelle :  

le fichier de configuration du __Pod "auth"__ est disponible ici 

`https://github.com/Treeptik/training-k8s-resources/blob/master/00_Treeptik/Sources/auth_pod.yaml`

le fichier de configuration du __Pod "hello"__ ,construit dans l'exercice précédent, est disponible ici :

`https://github.com/Treeptik/training-k8s-resources/blob/master/00_Treeptik/Sources/hello_pod.yaml`

le fichier de configuration du __Pod "frontend"__ est donné ici :

`https://github.com/Treeptik/training-k8s-resources/blob/master/00_Treeptik/Sources/frontend_pod.yaml`


On rappelle que 
- Dans le fichier de configuration d'un Deployment, le champ **.spec.template** décrit le template d'un Pod (celui décrit dans le fichier de configuration du Pod)


7. Contruire le fichier de configuration du Deployment pour le Pod "auth" - Avec Replica: 1  
8. Contruire le fichier de configuration du Deployment pour le Pod "hello" - Avec Replica: 1  
9. Contruire le fichier de configuration du Deployment pour le Pod "frontend" - Avec Replica: 1  

10. Creér les 3 Deployments avec les fichiers de configurations complétés précedement.   
11. Quel est le resultat de la commande : `kubectl get deployments`

12. Quel est le resultat de la commande : `kubectl describe deployments` pour chaque deployment précedemment lancé ? Trouver le nom du PrelicatSet associé au Deployment. 


### Scalabilité 

Il est facile de re-dimensionner à la volée (scale-up ou scale-down) un Deployment. 
Puisque les Deployments ont été crée en utilisant des fichiers de configuration il s'agit de mettre à jour le champ **.spec.replicas** avec la nouvelle valeur cible. 

Les Deployments pour la version 1.0.0 de la stack applicative seront scalés tels que :
- Un replica de 3 POD "frontend"
- Un replica de 5 POD "auth"
- Un replica de 5 POD "hello"

13. Mettre à jour les fichiers de Configurations pour les 3 Deployments
14. Que faut-il faire pour que les nouvelles valeures cibles de replicas soient prises en compte ? 
15. Decrivez précisement les nouveaux groupes de PODs : quel est le nom des pods, sur quels noeuds sont-ils executés ?  
16. Combien de ReplicaSet(s) chaque Deployment a t-il crée ? 

### Montée en version Applicative - Rolling Update 

Il est également facile de faire une rolling update à la volée - montée en version - dans notre cas avec un Deployment.

La version 1.0.0 de APP est lancée. On veut déployer la __version 2.0.0__  à la volée avec les Deployments suivants :
- Un replica de 3 POD "frontend"
- Un replica de 5 POD "auth"
- Un replica de 5 POD "hello"

C'est à dire mettre à jour les container auth et hello qui actuellement tournent en **version 1.0.0** vers la **version 2.0.0**
- auth v2.0.0 image: "kelseyhightower/auth:2.0.0"
- hello v2.0.0 image: "kelseyhightower/hello:2.0.0"

17. Mettre à jour les Deployments et les composants applicatifs 
18. Quelle commande pouvez-vous utiliser pour vérifier le déroulement des mises à jour ( rollout status ) ? 

