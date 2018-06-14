# Exercice : Les Services 

Dans les parties précedentes nous avons lancé des ReplicatSets et des Deployments (qui eux-mêmes s'appuis sur des ReplicatSets ) pour executer les Pods de containers qui constituent les composants de l'application. 

Dans cet exercice nous allons __mettre en ligne l'application APP__ en utilisant les Services Kubernetes pour exposer chaque Pod : 
- Sur Internet pour le Pod "frontend", 
- En interne pour les Pods "auth" et "hello". 

A la fin de cet exercice nous aurons 
- Ecrit les fichier de configuration des Services 
- ..
- ..


## Introduction 

Nous avons vu lors des exercices précédents que chaque Pod possède une adresse IP. 

1. Rappeler la commande pour obtenir l'adresse IP d'un POD 
//  kubectl get pods -l run=**"podname"** -o yaml | grep podIP

Il s'agit ici de décrire les mécanismes mis en oeuvre lors du lancement de chaque deployment afin de mettre en evidence la nécéssité des Services Kubernetes. 

![POD](https://github.com/Treeptik/training-k8s-resources/blob/master/05_Services/images/Treeptik-training-k8s-exo5-workflow_pod.jpg?raw=true "POD")

- 1 L'utililsateur invoque kubectl pour lancer un deployment via un fichier de configuration,
- 2 kubectl transmet la requete à l'API kube-apiserver
- 3 kube-apiserver sauvegarde les informations liées au Pod à deployer dans etcd
- 4 kube-scheduler recherche dans etcd (via kube-apiserver) si des Pods correpondant sont déjà lancés : il n'en trouve pas. 
- 5 kube-scheduler décide sur quel Node le Pod sera lancé. Pour cela kube-scheduler se base :  
- 5.1 : sur les spécifications du Pod lui même - par exemple si un container du Pod a besoin de disque SSD pour son volume persistant, kube-scheduler selectionnera un Node avec des disques SSD. 
- 5.2 : sur les Pods déjà existants dans le cluster : y a t'il des affinités avec le Pods à lancer ? Etc etc. kube-scheduler est "topology-aware"
- 6 kube-scheduler demande à kube-api de lancer le Pod - sur le Node 2 par exemple 
- 7 kube-apiserver invoque kubelet et kube-proxy sur le Node 2 : 
- 7.1 kubelet pour lancer le Pod de container : 
- 7.1.1 - kubelet demande à Docker d'executer le(s) container(e) du Pod 
- 7.1.2 - Docker lance le(s) container(s) 
- 7.2 kube-proxy pour la partie (proxy-)networking :
- 7.2.1 - kube-proxy attribue l'adresse IP au Pod ( DHCP )
- 7.2.2 - kube-proxy utilise netflilter pour autoriser le forward des flux vers cet IP en invoquant le set de commandes iptables. En

Le Pod est mainteant lancé avec un adresse IP. Aussi et afin de maintenir l'état décrit dans le Deployement : 

8 - kube-apiserver commande à kube-controller-manger de lancer un Deployement Controler : une boucle qui va s'assurer que le Deployment reste tel que décrit dans le fichier de configuration. 


### Exposer les Pods sur le Cluster 

Même si les Pods viennent avec leurs IPs, l'application ne peut pas s'appuyer dessus de manière continue : en effet, les controleurs de type ReplicatSet ou Deployment créent et détruisent les PODs dynamiquement (scaling Up, scaling Down, rolling upgrade....) ou bien les Nodes eux-mêmes peuvent tomber en panne. Il est donc nécessaire d'introduire un mécanisme qui va mapper les IPs des Pods à une entité logique plus stable, qui demeurera même si le Pod est détruit et qui sera en mesure de mapper les flux sur la nouvelle IP du nouveau Pod.  

__Les services__ Kubernetes sont ces objets kubernetes qui permettent d'identifier un groupe logique de Pods ainsi que les politiques pour y acceder. Pour définir ce groupe logique de Pods on utilise à nouveau le Label Selector. 

Services : 
- ClusterIP : service par defaut 
- NodePort : 
- Load Balancer :  


La spécification suivante créera un __Service__ :  

```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  type: ClusterIP
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 9080
  - name: https
    protocol: TCP
    port: 443
    targetPort: 9433
```

- Ce service s'appelera "my-service"
- Il sélectionnera tous les Pods qui ont le label "MyApp"
- Le service est de type "ClusterIP" : il aura une adresse IP virtuelle (VIP) distribuée sur tout le Cluster. 
- Le service dont l'IP est ClusterIP ecoutera sur les ports TCP:80 ( HTTP ) et sur TCP:443( HTTPS ) 
- Au niveau des Nodes cela signifie : kuybe-api installera les règles iptables via kube-proxy sur chaque noeud qui captureront le trafic à destination des ports 80 et 443 et qui redirigera vers les ports 9080 et 9430 
- Un object de type Endpoint appelé 'my-service" sera implicitement crée. Il exposera les IPs des Pods au service.  Autrement dit il sera possible de joindre les Pods au travers de la VIP "ClusterIP" grâce à ce Endpoint.    
- Endpoint evaluera continuellement quels sont les Pods taggués MyApp et donc leurs IPs, et tiendra à jour un inventaire des IPs des Pods. 
- Lorsqu'un requête utilisateur arrivera sur la ClusterIP, le Service interrogera le Enpoint pour savoir sur quel Pod router la requete et donc sur quelle IP 
- Le Endpoint choisira en mode random le Pod à contacter, les règles iptables sur chaque Noeud permettant d'intercepter le trafic en 80/443 pour le rediriger sur 9080/9433  

### Pods "auth" et "hello"

En se basant sur les fichiers de configuration des Pods 

le fichier de configuration du __Pod "auth"__ est disponible ici 

`https://github.com/Treeptik/training-k8s-resources/blob/master/05_Services/sources/auth_pod.yaml`

le fichier de configuration du __Pod "hello"__ ,construit dans l'exercice précédent, est disponible ici :

`https://github.com/Treeptik/training-k8s-resources/blob/master//05_Services/sources/hello_pod.yaml`


2. Ecrire le fichier de configuration du Service pour le Pod "auth"
3. Ecrire le fichier de configuration du Service pour le Pod "hello"



