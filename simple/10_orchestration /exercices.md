# Stratégies de placement

## Node Selector 

Obtenez la liste des Nodes 
`kubectl get nodes`
NAME                           STATUS     ROLES          AGE       VERSION
APHP-form-k8s-userX-master-1   Ready      master         8d        v1.10.2
APHP-form-k8s-userX-node-1     Ready   ingress,node   	 8d        v1.10.2
APHP-form-k8s-userX-node-2     Ready   ingress,node      8d        v1.10.2

Comme toutes ressources il est possible de la labeliser. Ce ou Ces labels serviront au kube-scheduler pour lancer un Pod porteur du label. 
Ajoutons le Label "schedulePodName"="hello-pod" . 

```
kubectl label nodes APHP-form-k8s-userX-node-1 schedulePodName=hello-pod`
node "kevindp-form-k8s-user1-node-1" labeled
```

Vérifier les labels : ( à la fin ) 
```
kubectl get nodes --show-labels
kevindp-form-k8s-user1-node-1     NotReady   ingress,node   8d        v1.10.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=kevindp-form-k8s-user1-node-1,node-role.kubernetes.io/ingress=true,node-role.kubernetes.io/node=true,schedulePodName=hello-pod
```

Il s'agit maintenant de configurer le Pod pour avoir le label correpondant dans le champs qui specifiera le **NodeSelector** :

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
  nodeSelector:
  	schedulePodName: hello-pod
```
