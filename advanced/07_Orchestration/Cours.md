## Orchestration

--------

### Concepts

**Rappels**

- __Pod__ : unité de base de Kubernetes qui porte la charge de travail applicative. 
- __ReplicaSet__ : controller - supervise les Pods pour les maintenir dans l'état opérationnel voulu pour remplir les fonctions applicative : on parle d'orchestration.
- __Deployement__ : controller -  s'appuit sur les ReplicatSets pour l'orchestration et offre des options pour la gestion du Cycle de vie de application (updates, tests )
- __Service__ : expose les Pods derrières des Endpoints - les rendant consommables à la demande d'autres services. 

--------

#### Controller 

Ils gèrent l'orchestration suivant les stratégies décidées pour maintenir et faire évoluer les fonctions applicatives
- Stratégie de placement
- Stratégie de mise à jour


--------

#### Scheduler et strategie

- Kubernetes fournit un orchestrateur : __kube-scheduler__.
- Possibilité d'implémenter des "customs" schedulers.
- Stratégie « Un (et un seul) Pod sur chaque nœud » : **DaemonSet**
- Stratégie « N copies, un Pod par nœud si possible » : **ReplicatSet** (et Replication Controller), **Deployment** ( via leur ReplicatSet)

--------

### Stratégie de placement


La stratégie de placement par défaut : Trouver là où il y a de la place sur des nœuds éligibles, puis répartir les pods correspondant à un même service sur plusieurs nœuds. 


--------


#### Stratégie nodeSelector

Elle permet de filtrer les nœuds éligibles au placement

```
apiVersion: v1
kind: Pod
spec:
  nodeSelector:
    <key>: <value>  // Se base sur les labels
...
```

--------

#### Stratégie nodeAfinity

Comparer les Labels sur les Nodes et le Champ **NodeAffinity** dans la definition du Pod.
```
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution: // Definition de la Règle
        nodeSelectorTerms: // Type de règle : Correpondance Key/Value (liste de valeur)
        - matchExpressions:
          - key: e2e-az-NorthSouth                                               
            operator: In // labels du node : e2e-az-NorthSouth =e2e-az-North ou =e2e-az-South pour pouvoir lancer le Pod
            values:
            - e2e-az-North
            - e2e-az-South
```

--------

#### Stratégie de placement __podAffinity__ et __podAntiAffinity__

Un Pod peut être schédulé ou non sur un Noeud en fonction des Pods qui sont déjà présents ou non.

--------

#### Stratégie de placement __podAffinity__ et __podAntiAffinity__

Exemple : si un Node héberge déjà un Pod avec un Label " **security=S1** " alors le Pod defini ci-après pourra être orchestré :
```
apiVersion: v1
kind: Pod
metadata:
  name: with-pod-affinity
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: security
            operator: In
            values:
            - S1
        topologyKey: failure-domain.beta.kubernetes.io/zone
  containers:
```

--------

#### Stratégie de placement de Pods : __Taints & Tolerations__


- Une **Taint** permet à un noeud de refuser qu'un Pod soit schedulé si le pod ne possède les **Toleration** correpondantes

- **Taint** et **Toleration** consiste en une pair de Key/Value plus un "effect" (lorsque Key=Value)


--------

#### Stratégie de placement de Pods : __Taints & Tolerations__


Par exemple : on définit 3 "Taints" sur le Node 1 : 

```
$ kubectl taint nodes node1 key1=value1:NoSchedule // "NoSchedule" = effect 
$ kubectl taint nodes node1 key1=value1:NoExecute  // "NoExecute" = effect
$ kubectl taint nodes node1 key2=value2:NoSchedule
```

--------

#### Stratégie de placement de Pods : __Taints & Tolerations__

On définit pour un Pod les **Tolerations** suivantes :

```
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

__Le Pod no sera pas Schedulé__ car il n'y a pas de **Toleration** correpondant à la 3eme **Taint** du Noeud

--------

### Stratégie de mise à jour

Rappel : les Objets de L'API Kubernetes - Orchestration 
- Pod : unité de base de Kubernetes qui porte la charge de travail applicative. 
- ReplicaSet : supervise les Pods pour les maintenir dans l'état opérationnel voulu pour remplir les fonctions applicative : on parle d'orchestration.
- Deployement : s'appuit sur les ReplicatSets pour l'orchestration et offre des options pour la gestion du Cycle de vie de application (updates, tests )

--------

#### Stratégie de mise à jour : les techniques outillées dans Kubernetes

Kubernetes fournit grâce aux Controllers des outils permettant gerer le cycle de vie des applications. 
- Rolling Update : intégré au Deployement via le set de de commande ***kubectl apply***
- Canary Deployment : utiliser les fonctionalités des deployments et des services (bascule/routage) pour cette technique
- Bleu-Green Deployment : utiliser les fonctionalités des deployments et des services (bascule/routage) pour cette technique

--------

#### Stratégie de mise à jour : __Rolling Update__

Grâce au Deployment il est possible de mettre à jour __automatiquement__ chaque Pod :
`kubectl apply -f deployment_v2.yaml`

L'upgrade sera effectuée un Pod après l'autre de telle facon à ce que l'application ne soit pas indisponible. Ainsi au fur et à mesure de l'avancée de la mise à jour les utilisateurs se connecteront sur la nouvelle version. 

<img src="https://github.com/Treeptik/training-k8s-by-treeptik/blob/master/reveal.js/Slides/Img/Orchestration/RollingUpdate.jpg" width="100%" />

--------

#### Stratégie de mise à jour : __Canary Deployement__

Un déploiement Canary consiste à déployer une nouvelle version applicative en parralèle, sur les plateformes de production et de la distribuer à un échantillon choisi d'utilisateurs tout en gardant la version historique de l'application en production. Le nouveau deployment devra être labelisé de facon à ce que le service integre le(s) nouveau(x) Pod(s) Canary

Par exemple 
- 5% d'utilisateur seront basculés sur la version Canary. 
- Les autres 95% resteront sur la version historique : le temps de valider que la nouvelle version "Canary" convient. 

<img src="https://github.com/Treeptik/training-k8s-by-treeptik/blob/master/reveal.js/Slides/Img/Orchestration/canary.png" width="130%" />

--------

#### Stratégie de mise à jour : __Blue-Green Deployment__

Lorsque qu'une application est mise à jour, il est parfois nécessaire de mettre à jour la stack applicative entièrement, puis de router les flux vers la nouvelle version : on parle de "one shot update" qui est une autre approche que la Rolling Update 
Dans cette technique, le deployment :
- __Blue__ est la version à mettre à jour, qui restera en production jusqu'à la bascule complète vers : 
- __Green__ qui est la nouvelle version.  

Il s'agit alors de router le trafic vers la version "Green" en mettant à jour le Service correpondant. 

<img src="https://github.com/Treeptik/training-k8s-by-treeptik/blob/master/reveal.js/Slides/Img/Orchestration/blue-green.png" width="100%" />
