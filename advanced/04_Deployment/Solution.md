## 1. Donner le nombre de POD qui sera lancé lors de la creation du Deployment avec ce fichier de configuration
`5`
## 2. Trouver la commande kubectl pour créer ce Deployment.
`kubectl create -f  fichier.yml`
## 3. Quelle Image Docker est utilisée  ?
`nickchase/soaktest`
## 4. Quel sont les labels associés au Pod qui sera orchestré par ce Deployment ?
`soaktest`
## 5. Quel est le selector pour ce Deployment ? A comparer avec la réponse du 4.
`soaktest | est identique au Label du Pod - obligatoire pour que le deployment puisse être crée. Au préalable le Deployement interrogera le Cluster pour vois si un ou plusieurs pod possède(nt) déjà un tel label`
## 7. Contruire le fichier de configuration du Deployment pour le Pod "auth"
`cf :  ../reponses/auth_deployment_v1.yaml`
## 8. Contruire le fichier de configuration du Deployment pour le Pod "hello"
`cf :  ../reponses/auth_deployment_v1.yaml`
## 9. Contruire le fichier de configuration du Deployment pour le Pod "frontend"
`cf :  ../reponses/hello_deployment_v1.yaml`

## 10. Creér les 3 Deployments avec les fichiers de configurations complétés précedement.   
`kubectl create -f auth_deployment_v1.yaml`
`kubectl create -f hello_deployment_v1.yaml`
`kubectl create -f hello_deployment_v1.yaml`

## 11. Quel est le resultat de la commande : `kubectl get deployments`

```
NAME                    DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
authv1-deployment       1         1         1            0           26m
frontendv1-deployment   1         1         1            0           17m
hellov1-deployment      1         1         1            0           18m
```

## 12. Quel est les resultats des commandes : `kubectl describe rs/auth` && `kubectl describe rs/hello` && `kubectl describe rs/frontend`

## 13. Mettre à jour les fichiers de Configurations pour les 3 Deployments
`cf :  ../reponses/auth_deployment_v2.yaml`
`cf :  ../reponses/hello_deployment_v2.yaml`
`cf :  ../reponses/frontend_deployment_v2.yaml`
## 14. Que faut-il faire pour que les nouvelles valeures cibles de replicas soient prises en compte ?
`kubectl apply -f  fichier.yml`
## 15. Decrivez précisement les nouveaux groupes de PODs : quel est le nom des pods, sur quels noeuds sont-ils executés ?  
`kubectl get pod -o wide |grep deployment`
## 16. Quel est le résultat de la commande : `kubectl rollout history deployment` ? 
## 17. Mettre à jour les Deployments vers la v2 
`kubectl apply -f auth_deployment_v2.yaml --record`
## 18. Quelle commande pouvez-vous utiliser pour vérifier le déroulement des mises à jour ( rollout status ) ?
```
kubectl rollout status deployment auth-deployment
Waiting for rollout to finish: 3 out of 5 new replicas have been updated...
```