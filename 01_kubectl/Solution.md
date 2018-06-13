1. Lister toutes les options disponibles avec la commande kubect
```
alpha           auth            config          delete          explain         options         replace         set             
annotate        autoscale       convert         describe        expose          patch           rolling-update  taint           
api-versions    certificate     cordon          drain           get             plugin          rollout         top             
apply           cluster-info    cp              edit            label           port-forward    run             uncordon        
attach          completion      create          exec            logs            proxy           scale           version         
```
2. Lister tous les objets disponibles en option de kubectl get

```
certificatesigningrequest  deployment                 networkpolicy              podtemplate                statefulset
clusterrolebinding         endpoints                  node                       replicaset                 status
componentstatus            event                      persistentvolume           replicationcontroller      storageclass
configmap                  horizontalpodautoscaler    persistentvolumeclaim      rolebinding                
controllerrevision         ingress                    pod                        secret                     
cronjob                    job                        poddisruptionbudget        service                    
daemonset                  namespace                  podsecuritypolicy          serviceaccount   
```

3. Création du pod Nginx
`kubectl run mynginx --image=nginx:1.15 --port=80`

4. Quel est le résultat de la commande *kubectl get pods* ?
- La commande kubectl get pods renvoi la liste des pods actuellement sur le cluster (running ou non)
- On y retrouve notre pod nginx suffixé de son identifiant (UID) unique géneré par k8s
- On retrouve aussi son nombre d'occurance (ready or not..) , son status , le nombe de restart, et sa durée de vie depuis son dernier lancement.

*nginx-6d9cc98df6-2cg74   1/1       Running   0          18h*

5. Identifier la Colonne "NAME", à votre avis que représente la première ligne *nginx-.....* ?
- Identifiant (UID) unique géneré par k8s
https://kubernetes.io/docs/concepts/workloads/pods/pod/
``` Like individual application containers, pods are considered to be relatively ephemeral (rather than durable) entities. As discussed in life of a pod, pods are created, assigned a unique ID (UID), and scheduled to nodes where they remain until termination (according to restart policy) or deletion. If a node dies, the pods scheduled to that node are scheduled for deletion, after a timeout period. A given pod (as defined by a UID) is not “rescheduled” to a new node; instead, it can be replaced by an identical pod, with even the same name if desired, but with a new UID (see replication controller for more details).
```
6. Identifier la Colonne "READY", à votre avis à quoi correspondent les chiffres : "1/1" ?
- Ici nous avons 1 occurence "READY" sur 1 provisionnée précedement en mode déclaratif
- Dans un cas avec un replicaset à 2 nous aurrions pu avoir 2/2
- Les tests interne à k8s et/ou livenessProbe+readinessProbe définissent ce status READY

7. Trouver la liste de tous les *status* possible dans le cycle de vie d'un POD.

https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-phase

```
A Pod’s status field is a PodStatus object, which has a phase field.

The phase of a Pod is a simple, high-level summary of where the Pod is in its lifecycle. The phase is not intended to be a comprehensive rollup of observations of Container or Pod state, nor is it intended to be a comprehensive state machine.

The number and meanings of Pod phase values are tightly guarded. Other than what is documented here, nothing should be assumed about Pods that have a given phase value.

Here are the possible values for phase:

- Pending: The Pod has been accepted by the Kubernetes system, but one or more of the Container images has not been created. This includes time before being scheduled as well as time spent downloading images over the network, which could take a while.

- Running: The Pod has been bound to a node, and all of the Containers have been created. At least one Container is still running, or is in the process of starting or restarting.

- Succeeded: All Containers in the Pod have terminated in success, and will not be restarted.

- Failed: All Containers in the Pod have terminated, and at least one Container has terminated in failure. That is, the Container either exited with non-zero status or was terminated by the system.

- Unknown: For some reason the state of the Pod could not be obtained, typically due to an error in communicating with the host of the Pod.
```
8. Trouver la commande qui affiche les logs temps reels du pod "nginx"
*kubectl logs nginx-6d9cc98df6-2cg74 -f*

9. Est-il possible de voir les logs du conteneur executé dans le pod "nginx" - comment ? Trouver la commande.
*kubectl logs ${POD_NAME} ${CONTAINER_NAME}*
