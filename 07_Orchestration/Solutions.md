## 1. En ce basant sur le fichier précédent : écrire le fichier de configuration du deployment pour la version "Canary" 
`ss`
## 2. Ecrire le fichier de configuration du service en se basanr sur le squelette :

## 3. Lancer le deployment version 1.0.0 (cf fichier de configuration source)

## 4. Lancer le deployment "canary" avec le fichier de configuration écrit précedemment

## 5. Lancer le service "hello-service"

## 6. Obtenir la description des 2 deployments "1.0.0" et "Canary" : quelles commandes utilisez-vous ? 

## 7. Obtenir la description du Service : quels sont les endpoints associés à ce service ? 

## 8. En ce basant sur le fichier précédent : écrire le fichier de configuration du deployment pour la version "Green" 
 
## 9. Lancer le deployment "Blue"

## 10. Lancer le deployment "Green"

## 11. Ecrire le fichier de configuration du service qui routera vers la version 1.0.0 (Blue) en se basant sur le squelette : 

## 12. Lancer le service "hello-service-blue"

## 13. Quels sont les endpoints de ce service ?

## 14. Verfier la description des deployments "Blue" et "Green" : vers quelles IPs le service route t-il le flux ?

## 15. Quel commande utiliser pour mettre à jour le Service pour router les flux vers la version "Green" 
kubectl apply -f hello_service_green.yaml

## 16. Mettre à jour le service 

## 17. vers quelles IPs le service "hello-service-blue" route t-il le flux ?