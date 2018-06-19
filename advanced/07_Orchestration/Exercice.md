# Exercice : Orchestration 


## Stratégie de placement

1. Quel est le resultat de la commande : `kubectl get nodes` ?
2. Créer les taints suivantes sur le node "APHP-form-k8s-userX-node-1" telles que : 
```
kubectl taint nodes APHP-form-k8s-userX-node-1 key1=value1:NoSchedule  
kubectl taint nodes APHP-form-k8s-userX-node-1 key1=value1:NoExecute  
kubectl taint nodes APHP-form-k8s-userX-node-1 key2=value2:NoSchedule
```

Vérifier que la sortie est bien `node "APHP-form-k8s-userX-node-1" tainted`

3. Quels est le résulat de la commande : `kubectl describe node APHP-form-k8s-userX-node-1` ? Vérifier que les Taints sont été prises en compte. 
4. Créer le fichier pod_toleration_1.yaml en utilisant le contenu ci-dessous :  

```
apiVersion: v1
kind: Pod
metadata:
  name: hello-toleration-1
  labels:
    app: hello
    realease: stable
    tier: webserver
    environement: dev
    partition: training-k8s
spec:
  containers:
    - name: hello
      image: "kelseyhightower/hello:1.0.0"
      ports:
      - name: http
        containerPort: 80
      - name: health
        containerPort: 81
      resources:
        limits:
          cpu: 50m
          memory: 50Mi
  tolerations:
  - key: "key1"
    operator: "Equal"
    value: "value1"
    effect: "NoSchedule"
  - key: "key1"
    operator: "Equal"
    value: "value1"
    effect: "NoExecute"
```

5. Lancer le Pod suivant avec la commande : `kubectl create -f pod_toleration_1.yaml`
6. Quel est le résulat de la commande : `kubectl get pods | grep hello-toleration-1` ? 
Indication : le scheduler ne trouve pas de Node avec les Taints correspondant aux Tolerations - le Pod n'est pas shédulé, en PENDING . 
7. Créer le fichier pod_toleration_2.yaml en utilisant le contenu ci-dessous 

```
apiVersion: v1
kind: Pod
metadata:
  name: hello-toleration-2
  labels:
    app: hello
    realease: stable
    tier: webserver
    environement: dev
    partition: training-k8s
spec:
  containers:
    - name: hello
      image: "kelseyhightower/hello:1.0.0"
      ports:
      - name: http
        containerPort: 80
      - name: health
        containerPort: 81
      resources:
        limits:
          cpu: 30m
          memory: 50Mi
  tolerations:
  - key: "key1"
    operator: "Equal"
    value: "value1"
    effect: "NoSchedule"
  - key: "key1"
    operator: "Equal"
    value: "value1"
    effect: "NoExecute"
  - key: "key2"
    operator: "Equal"
    value: "value2"
    effect: "NoExecute"
```

8. Quel est le résulat de la commande : `kubectl get pods | grep hello-toleration-2` ? 



## Créer un déploiement "Canary"

Un déploiement Canary consiste à déployer une nouvelle version applicative en parralèle,sur les plateformes de production et de la distribuer à un échantillon choisi d'utilisateurs tout en gardant la version historique de l'applicative en production.  

Par exemple 
- 5% d'utilisateur seront basculés sur la version Canary. 
- Les autres 95% resteront sur la version historique : le temps de valider que la nouvelle version "Canary" convient. 

Kubernetes permet de mettre en oeuvre facilement cette techique d'échantillonage grâce aux Deployments 
- Le 1er Deployment de 3 pods representera la version "historique" : en version 1.0.0 
- Le 2nd Deployment de 1 pod representera la version "canary" : en version 2.0.0 

![Canary](https://github.com/Treeptik/training-k8s-resources/blob/master/advanced/07_Orchestration/images/canary.png "Canary")

Nous utiliserons dans cet exercice le Pod "hello" : 

Pour la version "historique" nous nous baserons sur ce deployment : 

```
https://raw.githubusercontent.com/Treeptik/training-k8s-resources/master/advanced/07_Orchestration/sources/hello_deployment_v1.yaml
```

1. En ce basant sur le fichier précédent : écrire le fichier de configuration du deployment pour la version "Canary" 
__Indications__ : 
- Remplacer le label 'release: stable' avec la valeur "canary" partout
- La version Canary lancera 1 pod unique 

Le Service Kubernetes est en charge de distribuer les requêtes utilisateurs sur les 2 versions applicatives qui coexistent en production. 
Nous allons écrire le fichier de configuration de ce service de telle facon à ce que la selection soit à la fois sur les Pods de l'application dans sa version 1.0.0 et sur le pod de la version Canary . 

Rappel : Un service utilise les **selectors** pour identifier les pods et monter les endpoints pour router les flux vers ces Pods selectionner. 

2. Ecrire le fichier de configuration du service en se basant sur le squelette : 

```
apiVersion: v1
kind: Service
metadata:
  name: hello-service
spec:
  selector:
  	...
  	...// Déterminer le Selector 
  	...// Pour filtrer les Pod version 1.0.0 et "canary"
  	...
  type: ClusterIP
  ports:
    - name: http
      protocol: "TCP"
      port: 80
      targetPort: 80

```

3. Lancer le deployment version 1.0.0 (cf fichier de configuration source)
4. Lancer le deployment "canary" avec le fichier de configuration écrit précedemment
5. Lancer le service "hello-service"
6. Obtenir la description des 2 deployments "1.0.0" et "Canary" : quelles commandes utilisez-vous ? 
7. Obtenir la description du Service : quels sont les endpoints associés à ce service ? 

## Créer un déploiement "Blue-Green"

Lorsque qu'une application est mise à jour, il est parfois nécessaire de mettre à jour la stack applicative entièrement, puis de router les flux vers la nouvelle version : on parle de "one shot update" qui est une autre approche que la Rolling Update ( gérée également par l'Objet Deployment)
Dans cette technique, le deployment :
- __Blue__ est la version à mettre à jour, qui restera en production jusqu'à la bascule complète vers : 
- __Green__ qui est la nouvelle version.  

![Blue_Green](https://github.com/Treeptik/training-k8s-resources/blob/master/advanced/07_Orchestration/images/blue-green.png "Blue_Green")

Il s'agit alors de router le trafic vers la version "Green" en mettant à jour le Service correpondant. 

On se base pour la version "Blue" à mettre à jour sur le fichier de configuration suivant : 

`https://github.com/Treeptik/training-k8s-resources/blob/master/advanced/07_Orchestration/sources/hello_deployment_blue.yaml`

1. En ce basant sur le fichier précédent : écrire le fichier de configuration du deployment pour la version "Green" 
__Indications__ : 
- Remplacer le label 'version: 1.0.0' avec la valeur "2.0.0"
- Bien vérifier que l'image du container dans le template est la version 2.0.0. 

2. Lancer le deployment "Blue"
3. Lancer le deployment "Green"

4. Ecrire le fichier de configuration du service qui routera vers la version 1.0.0 (Blue) en se basant sur le squelette : 

```
apiVersion: v1
kind: Service
metadata:
  name: hello-service-blue
spec:
  selector:
  	...
  	...// Déterminer le Selector 
  	...// Pour filtrer les Pods version 1.0.0 UNIQUEMENT ( = Blue ) 
  	...
  type: ClusterIP
  ports:
    - name: http
      protocol: "TCP"
      port: 80
      targetPort: 80

```


5. Lancer le service "hello-service-blue"
6. Quels sont les endpoints de ce service ?
7. Quel commande utiliser pour mettre à jour le Service pour router les flux vers la version "Green" : faire la mise à jour . 
8. Vers quelles IPs le service "hello-service-blue" route t-il le flux ?

 


