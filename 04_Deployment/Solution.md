## 1. Donner le nombre de POD qui sera lancé lors de la creation du Deployment avec ce fichier de configuration

## 2. Trouver la commande kubectl pour créer ce Deployment.

## 3. Quelle Image Docker est utilisée  ?

## 4. Quel sont les labels associés au Pod qui sera orchestré par ce Deployment ?

## 5. Quel est le selector pour ce Deployment ? A comparer avec la réponse du 4.

## 7. Contruire le fichier de configuration du Deployment pour le Pod "auth"
## 8. Contruire le fichier de configuration du Deployment pour le Pod "hello"
## 9. Contruire le fichier de configuration du Deployment pour le Pod "frontend"

## 10. Creér les 3 Deployments avec les fichiers de configurations complétés précedement.   
## 11. Quel est le resultat de la commande : `kubectl get pods --show-labels`
## 12. Quel est le resultat de la commande : `kubectl get deployments`

## 13. Quel est les resultats des commandes : `kubectl describe rs/auth` && `kubectl describe rs/hello` && `kubectl describe rs/frontend`

## 14. Mettre à jour les fichiers de Configurations pour les 3 Deployments

## 15. Que faut-il faire pour que les nouvelles valeures cibles de replicas soient prises en compte ?

## 16. Decrivez précisement les nouveaux groupes de PODs : quel est le nom des pods, sur quels noeuds sont-ils executés ?  

## 17. Quel est les resultats des commandes : `kubectl describe rs/auth` && `kubectl describe rs/hello` && `kubectl describe rs/frontend`. Combien de ReplicaSets chaque Deployment a t-il crée ?
## 18. Mettre à jour les Deployments
## 19. Quelle commande pouvez-vous utiliser pour vérifier le déroulement des mises à jour ( rollout status ) ? 
