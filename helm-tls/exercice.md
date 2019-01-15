## Installation de Helm

https://docs.helm.sh/using_helm/#installing-helm

## Initialisation 

```
helm init
```

**Helm** par défaut n'utilise aucune sécurité. Ou plutôt utilisez le modèle ABAC.
Depuis K8S 1.7, le modèle ABAC est remplacé par RBAC.
Il est donc nécessaire désormais de lui associer un ServiceAccount (sa) pour pouvoir l'utiliser.

```
kubectl create serviceaccount --namespace kube-system tiller 
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller 
kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":"{"spec":{"serviceAccount":"tiller"}}}}'
```

Nous reviendrons plus tard sur la manière de sécuriser Helm à l'aide de certificats TLS.

## Déploiement de Ghost

Ghost est une plateforme de microblogging

```
helm search ghost
```

Installer une version de l'outil
```
helm install stable/ghost
```

Vous allez avoir une erreur. Pourquoi ?

Il manque le fichier **values.yaml**.
Quand nous installons un chart, certaines options sont par défaut.
Ces valeurs peuvent être écrasées soit
* par le fameux fichier **values.yaml**
* par la commande **--set**

## Redéploiement

Identifiez le déploiement puis le supprimer

```
helm ls
helm delete ...
```

On va désormais le redéployer avec un serviceType de nature ClusterIp

``
helm install --set service.type=ClusterIP stable/ghost
```

## Création du Ingress

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: tailored-ocelot-ghost
spec:
  backend:
    serviceName: tailored-ocelot
    servicePort: 80
```
  
