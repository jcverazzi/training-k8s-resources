
# Stratégie de placement : Taints et Tolerations  

## 1. Quel est le resultat de la commande : `kubectl get nodes` ?

```
NAME                              STATUS     ROLES          AGE       VERSION
APHP-form-k8s-userX-master-1   Ready      master         8d        v1.10.2
APHP-form-k8s-userX-node-1     Ready   ingress,node   8d        v1.10.2
APHP-form-k8s-userX-node-2     Ready   ingress,node   8d        v1.10.2
```

## 2. 




# Créer un déploiement "Canary"

## 1. En ce basant sur le fichier précédent : écrire le fichier de configuration du deployment pour la version "Canary" 
`https://github.com/Treeptik/training-k8s-resources/blob/master/07_Orchestration/reponses/hello_deployment_canary_v2.yaml`

## 2. Ecrire le fichier de configuration du service en se basant sur le squelette :
`https://github.com/Treeptik/training-k8s-resources/blob/master/07_Orchestration/reponses/hello_service.yaml`

## 3. Lancer le deployment version 1.0.0 (cf fichier de configuration source)
`kubectl create -f $nom_fichier_1.0.0.yaml`

## 4. Lancer le deployment "canary" avec le fichier de configuration écrit précedemment
`kubectl create -f $nom_fichier_canary.yaml`

## 5. Lancer le service "hello-service"
`kubectl create -f $nom_fichier_service.yaml`

## 6. Obtenir la description des 2 deployments "1.0.0" et "Canary" : quelles commandes utilisez-vous ? 
`kubectl describe deployement $deployment`

## 7. Obtenir la description du Service : quels sont les endpoints associés à ce service ? 
`kubectl describe service $service`
```
Se reporter au champ "Endpoint de l'output"
```

# Créer un déploiement "Blue-Green"

## 1. En ce basant sur le fichier précédent : écrire le fichier de configuration du deployment pour la version "Green" 
`https://github.com/Treeptik/training-k8s-resources/blob/master/07_Orchestration/reponses/hello_deployment_green.yaml`
 
## 2. Lancer le deployment "Blue"
`kubectl create -f $nom_fichier_blue.yaml`

## 3. Lancer le deployment "Green"
`kubectl create -f $nom_fichier_green.yaml`

## 4. Ecrire le fichier de configuration du service qui routera vers la version 1.0.0 (Blue) en se basant sur le squelette 
`https://github.com/Treeptik/training-k8s-resources/blob/master/07_Orchestration/reponses/hello_service_blue.yaml`

## 5. Lancer le service "blue"
`kubectl create -f $nom_fichier_service_blue.yaml`

## 6. Quels sont les endpoints de ce service ?
`kubectl describe service $service`
```
Se reporter au champ "Endpoint de l'output"
```

## 7. Quel commande utiliser pour mettre à jour le Service pour router les flux vers la version "Green" : mettre à jour le service. 
`kubectl apply -f $nom_fichier_service_green.yaml`

## 8. vers quelles IPs le service "green" route t-il le flux ?
`kubectl describe service $service`
```
Se reporter au champ "Endpoint de l'output"
```