# Exercice : Orchestration 

## Créer un déploiement "Canary"

Un déploiement Canary consiste à déployer une nouvelle version applicative en parralèle,sur les plateformes de production et de la distribuer à un echantillion choisi d'utilisateurs tout en gardant la version historique de l'applicative en production.  

Par exemple 
- 5% d'utilisateur seront basculés sur la version Canary. 
- Les autres 95% resteront sur la version historique : le temps de valider que la nouvelle version "Canary" convient. 

Kubernetes permet de mettre en oeuvre facilement cette techique d'échantillonage grâce aux Deployment 
- Le 1er Deployement de 3 pods representera la version "historique" : en version 1.0.0 
- Le 2nd Deployement de 1 pod representera la version "canary" : en version 2.0.0 

![Canary](https://github.com/Treeptik/training-k8s-resources/blob/master/07_Deployment/images/canary.png?raw=true "Canary")

Nous utiliserons dans cet exercice le Pod "hello" : 

Pour la version "historique" : 

`https://github.com/Treeptik/training-k8s-resources/blob/master/00_Treeptik/Sources/auth_pod.yaml`

1. Ecrire le fichier de configuration du deployment pour la version "Canary" 
- Indications : Rajouter un label 'track: canary'

