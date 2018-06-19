# Stratégies de placement

## Node Selector : nodeSelector

Obtenez la liste des Nodes 
`kubectl get nodes`
```
NAME                           STATUS     ROLES          AGE       VERSION
APHP-form-k8s-userX-master-1   Ready      master         8d        v1.10.2
APHP-form-k8s-userX-node-1     Ready   ingress,node   	 8d        v1.10.2
APHP-form-k8s-userX-node-2     Ready   ingress,node      8d        v1.10.2
```

Comme toutes ressources il est possible de la labeliser. Ce ou Ces labels serviront au kube-scheduler pour lancer un Pod porteur du label. 
Ajoutons le Label **"schedulePodName"="hello-pod"** . 

```
kubectl label nodes APHP-form-k8s-userX-node-1 schedulePodName=hello-pod`
node "kevindp-form-k8s-user1-node-1" labeled
```

Vérifier les labels : (regarder à la fin de la liste)
```
kubectl get nodes --show-labels
kevindp-form-k8s-user1-node-1     NotReady   ingress,node   8d        v1.10.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=kevindp-form-k8s-user1-node-1,node-role.kubernetes.io/ingress=true,node-role.kubernetes.io/node=true,schedulePodName=hello-pod
```

Il s'agit maintenant de configurer le Pod pour avoir le label correpondant dans le champ qui specifiera le **NodeSelector** :

Créer un fichier de configuration Pod "hello-nodeselector.yaml"

```
apiVersion: v1
kind: Pod
metadata:
  name: hello-nodeselector
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
          cpu: 25m
          memory: 50Mi
  nodeSelector:
  	schedulePodName: hello-pod
```

Lancer le Pod 

```
kubectl create -f hello-nodeselector.yaml
pod "hello-nodeselector" created
```

Observer sur quel Node le Pod a été orchestré : 
```
kubectl describe pod hello-nodeselector
```

Ce qui amènera la sortie suivante : On vérifie bien que le Pod a été crée sur le Node **APHP-form-k8s-userX-node-1**

```
Name:         hello-nodeselector
Namespace:    default
Node:         APHP-form-k8s-userX-node-1/207.154.237.15
Start Time:   Tue, 19 Jun 2018 19:43:49 +0000
Labels:       app=hello
              environement=dev
              partition=training-k8s
              realease=stable
              tier=webserver
Annotations:  kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"app":"hello","environement":"dev","partition":"training-k8s","realease":"stable...
Status:       Running
IP:           10.233.80.2
Containers:
  hello:
    Container ID:   docker://d227367257a72649b5ff50912f7ed71769449b36f3cf3751d839200cb6ae5268
    Image:          kelseyhightower/hello:1.0.0
    Image ID:       docker-pullable://kelseyhightower/hello@sha256:6d60ae5cf957ee1d87ac7e93bbe29e991eab18d18c29385bf9a5bf791a8d82e2
    Ports:          80/TCP, 81/TCP
    Host Ports:     0/TCP, 0/TCP
    State:          Running
      Started:      Tue, 19 Jun 2018 19:43:53 +0000
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     25m
      memory:  50Mi
    Requests:
      cpu:        25m
      memory:     50Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-gbtnw (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-gbtnw:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-gbtnw
    Optional:    false
QoS Class:       Guaranteed
Node-Selectors:  schedulePodName=hello-pod
Tolerations:     <none>
Events:
  Type     Reason                 Age                 From                                      Message
  ----     ------                 ----                ----                                      -------
  Normal   SuccessfulMountVolume  9s                  kubelet, APHP-form-k8s-userX-node-1  MountVolume.SetUp succeeded for volume "default-token-gbtnw"
  Normal   Pulling                8s                  kubelet, APHP-form-k8s-userX-node-1  pulling image "kelseyhightower/hello:1.0.0"
  Normal   Pulled                 5s                  kubelet, APHP-form-k8s-userX-node-1  Successfully pulled image "kelseyhightower/hello:1.0.0"
  Normal   Created                5s                  kubelet, APHP-form-k8s-userX-node-1  Created container
  Normal   Started                5s                  kubelet, APHP-form-k8s-userX-node-1  Started container
```


## Node Afinity : nodeAfinity 

Obtenez la liste des Nodes 
`kubectl get nodes`
```
NAME                           STATUS     ROLES          AGE       VERSION
APHP-form-k8s-userX-master-1   Ready      master         8d        v1.10.2
APHP-form-k8s-userX-node-1     Ready   ingress,node   	 8d        v1.10.2
APHP-form-k8s-userX-node-2     Ready   ingress,node      8d        v1.10.2
```

Ajoutons les labels suivants aux 2 Nodes : 
- APHP-form-k8s-userX-node-1 : "AvailZone=az-North"
- APHP-form-k8s-userX-node-2 : "AvailZone=az-South"

**Node 1**

```
kubectl label nodes APHP-form-k8s-userX-node-1 AvailZone=az-North`
node "kevindp-form-k8s-user1-node-1" labeled
```

**Node 2**

```
kubectl label nodes APHP-form-k8s-userX-node-2 AvailZone=az-South`
node "kevindp-form-k8s-user1-node-2" labeled
```


Vérifier les labels : (regarder à la fin de la liste)
```
kubectl get nodes --show-labels
kevindp-form-k8s-user1-node-1     NotReady   ingress,node   8d        v1.10.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=kevindp-form-k8s-user1-node-1,node-role.kubernetes.io/ingress=true,node-role.kubernetes.io/node=true
```

Il s'agit maintenant de configurer 3 Pods
- Pod-North qui devra se lancer sur un Node avec le Label az-North - Affinité = Required 
- Pod-South qui devra se lancer sur un Node avec le Label az-South - Affinité = Required 
- Pod-Middle qui se lancera sur un Node Présentant soit le Label az-North soit le Label az-South - Affinité = Prefered ( le kube-scheduler choisit le Node en se basant sur un score calculé en fonction de plusieurs paramètres )

Créer les fichiers de configuration des Pods suivants 

**pod-north.yaml**

```
apiVersion: v1
kind: Pod
metadata:
  name: hello-pod-north
  labels:
    app: hello-pod-north
    realease: stable
    tier: webserver
    environement: dev
    partition: training-k8s
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: AvailZone
            operator: In
            values:
            - az-North
            - az-North-East 
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
          cpu: 25m
          memory: 50Mi
```

Lancer les Pods 

`kubectl create -f pod-north.yaml `
`kubectl create -f pod-south.yaml `
`kubectl create -f pod-middle.yaml `

**pod-south.yaml**

```
apiVersion: v1
kind: Pod
metadata:
  name: hello-pod-south
  labels:
    app: hello-pod-south
    realease: stable
    tier: webserver
    environement: dev
    partition: training-k8s
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: AvailZone
            operator: In
            values:
            - az-South
            - az-South-East 
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
          cpu: 25m
          memory: 50Mi
```

**pod-middle.yaml**

```
apiVersion: v1
kind: Pod
metadata:
  name: hello-pod-middle
  labels:
    app: hello
    realease: stable
    tier: webserver
    environement: dev
    partition: training-k8s
spec:
  affinity:
    nodeAffinity: 
      preferredDuringSchedulingIgnoredDuringExecution: 
      - weight: 1 
        preference:
          matchExpressions:
          - key: AvailZone
            operator: In 
            values:
            - az-North 
            - az-South
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
          cpu: 25m
          memory: 50Mi
```

Lancer les Pods 

```
kubectl create -f pod-north.yaml 
pod "hello-pod-north" created

kubectl create -f pod-south.yaml
pod "hello-pod-south" created

kubectl create -f pod-middle.yaml
pod "hello-pod-middle" created
```

Obtenir la description des Pods : vérfier le placement des Pods 

```
kubectl describe pod  hello-pod-north
...
Node:         APHP-form-k8s-userX-node-1/207.154.237.15
...

kubectl describe pod  hello-pod-south
...
Node:         APHP-form-k8s-userX-node-2/207.154.237.16
...

kubectl describe pod  hello-pod-middle
...
Node:         APHP-form-k8s-userX-node-2/207.154.237.16 // Ici le scheduler a choisit "Node2"
...
```

## Pod Afinity : podAfinity & podAntiAffinity

Pour commencer vérifier que les 2 Pods de l'exercice précendent sont bien lancés : 
- Pod : "hello-pod-north" 
- Pod : "hello-pod-south" 

Dans cet exerice nous allons configurer 2 nouveaux Pod qui se lanceront en fonction de leur affinité (ou non) avec les Pods précedents
- Le Pod hello-pod-north-east aura une affinity avec le pod "hello-pod-north" et se lancera donc sur le même Node
- Le Pod hello-pod-south-east aura une "anti"-affinity avec les pod du nord "hello-pod-north" "hello-pod-north-east" et se lancera donc un autre Node - ici avec le mecanisme preferred et la pondération "weight"

Créer le fichier de configuration de ces 2 pods tels que : 

**pod-affinity-north.yaml**

```
apiVersion: v1
kind: Pod
metadata:
  name: hello-pod-north-east
  labels:
    app: hello
    realease: stable
    tier: webserver
    environement: dev
    partition: training-k8s
spec:
  affinity:
    podAffinity: 
      requiredDuringSchedulingIgnoredDuringExecution: 
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In 
            values:
            - hello-pod-north 
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
          cpu: 25m
          memory: 50Mi
```

**pod-anti-affinity-north.yaml** 

```
apiVersion: v1
kind: Pod
metadata:
  name: hello-pod-south-east
  labels:
    app: hello-pod-north-east
    realease: stable
    tier: webserver
    environement: dev
    partition: training-k8s
spec:
  affinity:
    podAntiAffinity: 
      preferredDuringSchedulingIgnoredDuringExecution: 
      - weight: 100 
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: app 
              operator: In 
              values:
              - hello-pod-north
              - hello-pod-north-east
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
          cpu: 25m
          memory: 50Mi
```

Lancer les Pods 

```
kubectl create -f pod-affinity-north.yaml 
pod "hello-pod-north-east" created

kubectl create -f pod-anti-affinity-north.yaml
pod "hello-pod-south-east" created
```

Obtenir la description des Pods : vérfier le placement des Pods 

```
kubectl describe pod hello-pod-north-east 
...
Node:         APHP-form-k8s-userX-node-1/207.154.237.15
...

kubectl describe pod  hello-pod-south-east
...
Node:         APHP-form-k8s-userX-node-2/207.154.237.16
...
```


