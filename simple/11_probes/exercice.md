## S'assurer que les applications travaillent bien

### Nettoyer l'environnement précédent

`kubectl delete daemonsets,replicasets,services,deployments,pods,rc --all`

### Créer un pod avec une sonde liveness

```
apiVersion: v1
kind: Pod
metadata:
  name: hc
spec:
  containers:
  - name: sonde
    image: treeptik/probe
    ports:
    - containerPort: 9876
    livenessProbe:
      initialDelaySeconds: 2
      periodSeconds: 5
      httpGet:
        path: /health
        port: 9876
```

Lancer la commande:
```
kubectl create -f probe-live.yml

kubectl create -f probe.yml
pod "hc" created
root@user1:~/probes# kubectl describe pod hc
Name:         hc
Namespace:    default
Node:         user1/167.99.204.171
Labels:       <none>
Annotations:  cni.projectcalico.org/podIP=10.42.2.14/32
Status:       Running
IP:           10.42.2.14
Containers:
  sonde:
    Container ID:   docker://c2103cf41691565a6ed51fd528245b44e3f7d4c5df17c0216c61e39a3ed72c44
    Image:          treeptik/probe
    Image ID:       docker-pullable://treeptik/probe@sha256:33f589ea71cc8a0e37a2f4b17c2e3c207a69c2df525aa62ef5451b75638fd966
    Port:           9876/TCP
    Host Port:      0/TCP
    State:          Running
    Ready:          True
    Restart Count:  0
    Liveness:       http-get http://:9876/health delay=2s timeout=1s period=5s #success=1 #failure=3
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-zkwpx (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-zkwpx:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-zkwpx
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason                 Age   From               Message
  ----    ------                 ----  ----               -------
  Normal  Scheduled              59s   default-scheduler  Successfully assigned hc to user1
  Normal  SuccessfulMountVolume  59s   kubelet, user1     MountVolume.SetUp succeeded for volume "default-token-zkwpx"
  Normal  Pulling                58s   kubelet, user1     pulling image "treeptik/probe"
  Normal  Pulled                 35s   kubelet, user1     Successfully pulled image "treeptik/probe"
  Normal  Created                35s   kubelet, user1     Created container
  Normal  Started                34s   kubelet, user1     Started container
```

## Lancer un port qui bogue

Créer le fichier **nitro.yml**

```
apiVersion: v1
kind: Pod
metadata:
  name: 
specquiaimelerisque:
  containers:
  - name: nitro
    image: treeptik/probe
    ports:
    - containerPort: 9876
    env:
    - name: HEALTH_MIN
      value: "1000"
    - name: HEALTH_MAX
      value: "4000"
    livenessProbe:
      initialDelaySeconds: 2
      periodSeconds: 5
      httpGet:
        path: /health
        port: 9876
```

Lancer le pod : 

```
kubectl create -f nitro.yml

kubectl describe pod nitropod

kubectl get pods

```


