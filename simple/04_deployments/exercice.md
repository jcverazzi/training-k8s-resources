## Utiliser les Deployments

### Nettoyer l'environnement précédent

`kubectl delete daemonsets,replicasets,services,deployments,pods,rc --all`

## Pourquoi utiliser les Deployments ?

Un ReplicaSet ne peut fournir de service de rolling-update que les ReplicationController savent faire.
Pour faire une action de rolling-update ou de rollout sur un ReplicaSet, il est nécessaire d'utiliser un Deployment de façon déclarative et non pas impérative.

## 







