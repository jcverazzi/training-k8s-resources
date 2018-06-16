# Exercice : le Controleur ReplicatSet 

Dans cet exercice nous allons créer des fichiers de configuraton de type "ReplicaSet" pour chaque service de l'application  "App, A sample 12 Facter Application", publiée par Kelsey Hightower (Google). 

`https://github.com/kelseyhightower/app`

A la fin de l'exercice nous aurons : 
- Ecrit les fichiers de configuration pour "auth" et "hello" ainsi que celui du frontend Nginx "frontend"
- Crée les : ReplicaSet pour chaque composant applicatif
- Abordé la scalabilité grâce aux ReplicatSets .

![Application APP](https://github.com/Treeptik/training-k8s-resources/blob/master/advanced/03_ReplicatSet/images/Treeptik-training-k8s-exo3-1.jpg?raw=true "Application APP")


Les différentes parties ne seront pas exposées : 
- sur Internet pour le frontend, 
- en interne pour les composants "auth" et "hello". 

L'Application APP n'est pas encore en mesure reçevoir et de traiter les requêtes utlisateurs : cette thématique sera abordée dans la partie dédiée aux Services
  
## Introduction : 

Le système Kubernetes a connaissance du "status" du Pod, comme nous l'avons vu dans l'exercice 1, mais ne sait pas si ce status est le bon pour accomplir la fonction applicative. 

Il est donc necessaire d'expliquer à Kubernetes, en mode déclaratif, de quelle façon on souhaite que les Pods evoluent ensemble : on parle d'orchestration des Pods. Une fois l'orchestration décrite, il est aussi nécessaire de contrôler que les Pods continuent à fournir le service applicatif attendu. Kubernetes utilise des composants programmables dédiés à ces tâches : Les "Controllers". Les ReplicatSets et les Deployments en sont des exemples.  


## ReplicaSet Kubernetes

Le premier objet **controller** introduit pas Kubernetes a été le **ReplicationController** . Ce composant offre les possibilités de bases qui seront utilisées et améliorées avec les controllers plus récents, par exemple : 
- Superviser de multiples Pods sur de multiples Nodes,
- Utilser les labels pour identifier les Pod à controler,  
- Utiliser les replicas comme référence sur le nombre de Pods à orchestrer .... 

L'objet **ReplicaSet** est la nouvelle génération de **ReplicationController**. 


- Le ReplicatSet va superviser de multiples Pods sur de multiples Nodes en s'appuyant sur un nouveau champs : le **selector** qui ajoute - par rapport aux labels - des fonctionnalités avancées en terme de filtrage des Pods a orchestrer. Ci après 2 exemples déclaratifs utilisant le selector 

```
apiVersion: apps/v1
kind: ReplicaSet
...
spec:
   replicas: 3
   selector:
     matchLabels:
       app: soaktestrs
...
```

```
apiVersion: apps/v1
kind: ReplicaSet
...
spec:
   replicas: 3
   selector:
     matchExpressions:
      - {key: app, operator: In, values: [soaktestrs, soaktestrcs, soaktest]}
      - {key: track, operator: NotIn, values: [production]}
..
```


### Etude d'un fichier de configuration de ReplicatSet

On se donne le fichier suivant :

```
 apiVersion: apps/v1
 kind: ReplicaSet
 metadata:
   name: soaktestrs
 spec:
   replicas: 3
   selector:
     matchExpressions:
      - {key: app, operator: In, values: [soaktest1, soaktest2, soaktest]}
      - {key: track, operator: NotIn, values: [production, stagging]}
   template:
     metadata:
       labels:
         app: soaktest1
         track: dev
     spec:
       containers:
       - name: soaktestrs
         image: nickchase/soaktest
         ports:
         - containerPort: 80
```

Comme tous les objets de l'API Kubernetes, un ReplicaSet est defini avec les champs **apiVersion**, **kind**, et **metadata**. A cela on ajoute le champs **spec** qui décrit les spécifications du status d'un Pod à atteindre et à maintenir lors de l'orchestration. 

- Une spécificité introduite avec le ReplicatSet est le champs **.spec.selector.**. Le ReplicatSet gère les Pods dont le label correspond au selector défini dans ce champs : il est essentiel de noter que lors de la création du ReplicatSet, l'API sera interrogée pour savoir si des Pods déjà existants sont déjà labelisés suivant le valeur du Selector. Si de tel Pod existent, alors le Replicaset en prendra le contrôle. 
- La bonne pratique est d'utiliser ce "Label Selector" et de le préférer au "Label" uniquement. En effet, l'usine logicielle est succeptible d'utiliser de nombreux pipelines et il est essentiel de pouvoir filtrer/selectionner les labels précisement pour l'orchestration de Pod. La figure suivante permet d'illuster un pipeline complexe et la nécessite de selectionner judicieusement les Labels : 

![Label Selector](https://github.com/Treeptik/training-k8s-resources/blob/master/advanced/03_ReplicatSet/images/Labels_Selector.png?raw=true "Label Selector")

- Le champ **.spec.template** décrit le template d'un Pod, qui est encapsulé dans la configuration du ReplicaSet. On remarque qu'il s'agit du même shéma que le fichier de configuration d'un Pod. 
- En particulier on retrouve le champ **.spec.template.metadata.labels** . Le selector recherche dans ce champs les valeurs qui correpondent à celle du **.spec.selector.**. Si aucune correpondance n'est trouvée, l'API rejetera la configuration.
- Si le champ **.spec.selector.** n'est pas précisé alors c'est le champ **.spec.template.metadata.labels** qui sera utilisé pour reconnaitre les Pods à orchestrer.


1. Donner le nombre de POD qui sera lancé lors de la creation du ReplicaSet avec ce fichier de configuration
2. Trouver la commande kubectl pour créer ce ReplicatSet. 
3. Quelle Image Docker est utilisée ? 
4. Quel sont les labels associés au Pod qui sera orchestré par ce ReplicaSet ? 
5. Expliquer le selector pour ce ReplicaSet ? A comparer avec la réponse du 4. 
6. Ce ReplicatSet est-il destiné à la Production ? 


### Configuration du ReplicaSet pour les Pods "auth", "hello" et "frontend". 

Pour construire les fichiers de configuration des 3 ReplicatSets nous allons nous appuyer sur les fichiers de configuration des Pod "auth", "hello" et "frontend". 

le fichier de configuration du __Pod "auth"__ est disponible ici 

`https://github.com/Treeptik/training-k8s-resources/blob/master/advanced/00_Treeptik/Sources/auth_pod.yaml`

le fichier de configuration du __Pod "hello"__ ,construit dans l'exercice précédent, est disponible ici :

`https://github.com/Treeptik/training-k8s-resources/blob/master/advanced/00_Treeptik/Sources/hello_pod.yaml`

le fichier de configuration du __Pod "frontend"__ est donné ici :

`https://github.com/Treeptik/training-k8s-resources/blob/master/advanced/00_Treeptik/Sources/frontend_pod.yaml`

 
Les ReplicatSets orchestrerons pour commencer:
- Un replica de 1 POD "frontend"
- Un replica de 1 POD "auth"
- Un replica de 1 POD "hello"

On rappelle que 
- Dans le fichier de configuration d'un ReplicatSet, le champ **.spec.template** décrit le template d'un Pod (celui décrit dans le fichier de configuration du Pod)


7. Contruire le fichier de configuration du ReplicatSet pour le Pod "auth"
8. Contruire le fichier de configuration du ReplicatSet pour le Pod "hello"
9. Contruire le fichier de configuration du ReplicatSet pour le Pod "frontend"

10. Creér les 3 ReplicatSets avec les fichiers de configurations complétés précedement.   

11. Quel est le resultat de la commande : `kubectl get pods`
12. Quel est le resultat de la commande : `kubectl get replicasets`

13. Quel est les resultats des commandes : `kubectl describe rs/auth` && `kubectl describe rs/hello` && `kubectl describe rs/frontend`


### Scalabilité 

Il est facile de re-dimensionner à la volée (scale-up ou scale-down) un ReplicatSet. 
Puisque les ReplicatSets ont été crée en utilisant des fichiers de configuration il s'agit de mettre à jour le champ **.spec.replicas** avec la nouvelle valeur cible. 

Les ReplicatSets seront scalés tels que :
- Un replica de 5 POD "frontend"
- Un replica de 5 POD "auth"
- Un replica de 5 POD "hello"

14. Mettre à jour les fichiers de Configurations pour les 3 ReplicatSets
15. Que faut il faire pour que les nouvelles valeures cibles de replicas soient prises en compte ? 
16. Decrivez précisement les nouveaux groupes de PODs : quel est le nom des pods, sur quels noeuds sont-ils executés ?  
17. A votre avis, que ce passe t'il si un Node tombe en panne ? //
18. Fin de l'exercice : veiller à bien effacer les ReplicatSets !  


__Perpectives__ : 

Cette approche de la scalabilité est limitée : même si le système Kubernetes se charge de toutes les opérations au niveau du cluster pour re-dimensionner à la volée un ReplicatSet de Pod(s), il faut qu'un opérateur, en mode déclaratif, intervienne manuellement pour renseigner les nouveaux fichiers de configuration 

Les __Horizontal Pod Autoscaler (HPA)__ est une ressource de l'API qui permet de re-dimmensionner un ReplicaSet ( ou un Deployment ) de Pods en fonction de l'utilisation d'une métrique. Par défaut, le re-dimmensionnement en fonction de l'utilisation du CPU est implémentée dans la version autoscaling/v1 de Kubernetes. La prise en charge de "customs metrics" est disponible dans la version autoscaling/v2beta1 de l'API. 




