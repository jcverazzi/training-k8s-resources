# Stratégies de placement

## Node Selector 

Obtenez la liste des Nodes 
`kubectl get nodes`
```
NAME                           STATUS     ROLES          AGE       VERSION
APHP-form-k8s-userX-master-1   Ready      master         8d        v1.10.2
APHP-form-k8s-userX-node-1     Ready   ingress,node   	 8d        v1.10.2
APHP-form-k8s-userX-node-2     Ready   ingress,node      8d        v1.10.2
```

Comme toutes ressources il est possible de la labeliser. Ce ou Ces labels serviront au kube-scheduler pour lancer un Pod porteur du label. 
Ajoutons le Label "schedulePodName"="hello-pod" . 

```
kubectl label nodes APHP-form-k8s-userX-node-1 schedulePodName=hello-pod`
node "kevindp-form-k8s-user1-node-1" labeled
```

Vérifier les labels : 
```
kubectl get nodes --show-labels
kevindp-form-k8s-user1-node-1     NotReady   ingress,node   8d        v1.10.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=kevindp-form-k8s-user1-node-1,node-role.kubernetes.io/ingress=true,node-role.kubernetes.io/node=true,schedulePodName=hello-pod
```

Il s'agit maintenant de configurer le Pod pour avoir le label correpondant dans le champs qui specifiera le **NodeSelector** :

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
          cpu: 50m
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

Ce qui amènera la sortie suivante : On vérifie bien que le Pod a été crée sur le Node 

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
      cpu:     50m
      memory:  50Mi
    Requests:
      cpu:        50m
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





