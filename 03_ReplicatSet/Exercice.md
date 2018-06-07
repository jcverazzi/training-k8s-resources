# Exercice : le Controleur ReplicatSet 

Dans cet exercice nous allons créer des fichiers de configuraton de type "ReplicaSet" pour chaque service de l'application  "App, A sample 12 Facter Application", publiée par Kelsey Hightower (Google). 

`https://github.com/kelseyhightower/app`

A la fin de l'exercice nous aurons : 
- Ecrit les fichiers de configuration pour "auth" et "hello" ainsi que celui du frontend Nginx "frontend"
- Crée les : ReplicaSet pour chaque composant applicatif
- Abordé la scalabilité grâce aux ReplicatSets .

![Application APP](https://github.com/Treeptik/training-k8s-resources/blob/master/03_ReplicatSet/images/Treeptik-training-k8s-exo3-1.jpg?raw=true "Application APP")


A la fin de l'exercice l'Application APP ne sera pas encore utilisable. Les différentes parties ne seront pas exposées : 
- sur Internet pour le frontend, 
- en interne pour les composants "auth" et "hello". 

l'Application APP ne sera donc pas encore en mesure recevoir et de traiter les requêtes utlisateurs : cette thématique sera abordée dans les parties suivantes. 
  
## Introduction : 

Le système Kubernetes a connaissance du "status" du Pod, comme nous l'avons vu dans l'exercice 1, mais ne sait pas si ce status est le bon pour accomplir la fonction applicative. 

Il est donc necessaire d'expliquer à Kubernetes, en mode déclaratif, de quelle façon on souhaite que les Pods evoluent ensemble : on parle d'orchestration des Pods. Une fois l'orchestration décrite, il est aussi nécessaire de contrôler que les Pods continuent à fournir le service applicatif attendu. Kubernetes utilise des composants programmables dédiés à ces tâches : Les "Controllers". Les ReplicatSets et les Deployments en sont des exemples.  


## ReplicaSet Kubernetes

Le premier objet **controller** introduit pas Kubernetes a été le **ReplicationController** . Ce composant offre les possibilités de bases qui seront utilisées et améliorées avec les controllers plus récents, par exemple : 
- Superviser de multiples Pods sur de multiples Nodes,
- Utilser les labels pour mener à bien la supervision, 
- Utiliser les replicas comme référence sur le nombre de Pods à orchestrer .... 

L'objet **ReplicaSet** est la nouvelle génération de **ReplicationController**. Le ReplicatSet va superviser de multiples Pods sur de multiple Nodes en s'appuyant sur un nouveau champs : le **selector** - qui ajoute par rapport aux labels, des fonctionnalités avancées en terme de filtrage des Pods a orchestrer. Ci après 2 exemples déclaratifs utilisant le selector 

```
apiVersion: extensions/v1beta1
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
apiVersion: extensions/v1beta1
kind: ReplicaSet
...
spec:
   replicas: 3
   selector:
     matchExpressions:
      - {key: app, operator: In, values: [soaktestrs, soaktestrs, soaktest]}
      - {key: tier, operator: NotIn, values: [production]}
..
```


### Etude d'un fichier de configuration de ReplicatSet

On se donne le fichier suivant :

```
 apiVersion: extensions/v1beta1
 kind: ReplicaSet
 metadata:
   name: soaktestrs
 spec:
   replicas: 3
   selector:
     matchLabels:
       app: soaktestrs
   template:
     metadata:
       labels:
         app: soaktestrs
         environment: dev
     spec:
       containers:
       - name: soaktestrs
         image: nickchase/soaktest
         ports:
         - containerPort: 80
```

Comme tous les objets de l'API Kubernetes, un ReplicaSet est defini avec les champs **apiVersion**, **kind**, et **metadata**. A cela on ajoute le champs **spec** qui décrit les spécifications du status d'un Pod à atteindre et à maintenir lors de l'orchestration. 

- Une spécificité du controler ReplicatSet est le champs **.spec.selector.**. Le ReplicatSet gère les Pods dont le label correspond au selector défini dans ce champs.
- Le champ **.spec.template** décrit le template d'un Pod, qui est encapsulé dans la configuration du ReplicaSet. On remarque qu'il s'agit du même shéma que le fichier de configuration d'un Pod. En particulier on retrouve le champ **.spec.template.metadata.labels** qui sera comparé à **.spec.selector.** . Si ces deux champs ne sont pas identiques l'API rejetera la configuration. 


1. Donner le nombre de POD qui sera lancé lors de la creation du ReplicaSet avec ce fichier de configuration
2. Trouver la commande kubectl pour créer ce ReplicatSet. 
3. Quelle Image Docker est utilisée ? 
4. Quel sont les labels associés au Pod qui sera orchestré par ce ReplicaSet ? 
5. Quel est le selector pour ce ReplicaSet ? A comparer avec la réponse du 4. 
6. Ce ReplicatSet est-il destiné à la Production ? 


### Configuration du ReplicaSet pour le Pod "auth"

Le ReplicatSet orchestrera :
- Un replica de 3 PODs "auth"

On rappelle que 
- Dans le fichier de configuration d'un ReplicatSet, le champ **.spec.template** décrit le template d'un Pod

le fichier de configuration du Pod "auth" est disponible ici 

`https://github.com/Treeptik/training-k8s-resources/blob/master/02_Pods/sources/auth_pod.yaml`


7. En vous basant sur les données précédentes, completez le fichier de configuration du ReplicatSet pour le Pod "auth"

```
apiVersion: extensions/v1beta1
kind: ReplicatSet
metadata:
  name: auth
spec:
  replicas: 
  selector:
    matchLabels:
      app: auth
  template:
    metadata:
      labels:
        app: 
        track:
    spec:
      containers:
        - name: 
          image: 
          ports:
            - name: 
              containerPort: 
          resources:
            limits:
              cpu: 
              memory: 
```

8. Creér le ReplicatSet avec le fichier de configuration complété précedement  
9. Quel est le resultat de la commande : `kubectl get pods`
10. Quel est le resultat de la commande : `kubectl get replicasets`

### Configuration des ReplicaSets pour le Pod "hello" et le Pod "frontend"

Les deux ReplicatSets pour les Pods "hello" et le Pod "frontend" orchestrerons :
- Un replica de 3 PODs. 

le fichier de configuration du __Pod "auth"__ ,construit dans l'exercice précédent, est disponible ici :

`https://github.com/Treeptik/training-k8s-resources/blob/master/02_Pods/sources/auth_pod.yaml`

le fichier de configuration du __Pod "frontend"__ est donné ici :

`https://github.com/Treeptik/training-k8s-resources/blob/master/03_ReplicatSet_Deployment/sources/frontend_pod.yaml`

Vous pourrez remarquer des déclarations supplementaires dans le fichier de configuration du Pod "frontend" : le template du Pod décrit un volume persistant permettant de stocker 
- Le fichier de configuration de nginx pour ecouter sur le port 443 avec une couche SSL/TLS
- Les certificats SSL/TLS ( ! dangeureux sur un dépot public.. ! )

Cette partie sera détaillée dans la suite de la Formation 
 
11. Ecrire le fichier de configuration du ReplicatSet pour les Pods "hello" et le Pod "frontend"
12. Creér le ReplicatSet avec le fichier de configuration précedent 
13. Quel est le resultat de la commande : `kubectl get pods`
14. Quel est le resultat de la commande : `kubectl describe rs/hello` && `kubectl describe rs/frontend`