
## 1. Combien de Deployment vont être lancés ? 
- 1er  deployment : "pikachu"
- 2eme deployment : "mewtwo" 
- 3eme deployment : "nidoqueen"

## 2. Combien de pod vont être lancés pour chaque Deployment ?
- 1er  deployment : 2
- 2eme deployment : 2
- 3eme deployment : 2

## 3. Lancer les deployments
`kubectl create -f https://github.com/sozu-proxy/sozu-demo/blob/master/kubernetes-using-tube-cheese/pokemon-deployments.yaml`

## 4. Decrivez chacun des deployments 
`kubectl describe deployments`

## 5. Ecrire le fichier de configuration pour le service pour **mewtwo**  et lancer le service 
`kubectl create -f https://github.com/sozu-proxy/sozu-demo/blob/master/kubernetes-using-tube-cheese/pokemon-deployments.yaml`

## 6. Ecrire le fichier de configuration pour le service pour **nidoqueen** et lancer le service 
`kubectl create -f https://github.com/sozu-proxy/sozu-demo/blob/master/kubernetes-using-tube-cheese/pokemon-deployments.yaml`

## 7. Quelle commande utilisez vous pour décrire l'état et les détails du service ? 
`kubectl describe service`

## 8. Lancer le Ingress Controller 
`kubectl create -f https://github.com/sozu-proxy/sozu-demo/blob/master/kubernetes-using-tube-cheese/pokemon-ingress.yaml`

## 9. Déterminer sur quelle IP publique l'application est lancée 

## 10. Connectez vous sur http://ip_publique/pikachu, http://ip_publique/mewtwo, http://ip_publique/nidoqueen
=======
## 1. Combien de Deployment vont être lancés ? 
- 1er  deployment : "pikachu"
- 2eme deployment : "mewtwo" 
- 3eme deployment : "nidoqueen"

## 2. Combien de pod vont être lancés pour chaque Deployment ?
- 1er  deployment : 2
- 2eme deployment : 2
- 3eme deployment : 2

## 3. En analysant les champs {spec.template.spec.container.port} pour chaque type de deployment, déterminer quelle sera la réponse de chaque type de Pod à une requête HTTP port 80 ? 
- 1er  deployment : "pikachu" - deploie une image **clevercloud/demo-pokemon** et parametise la réponse HTTP:80 pour renvoyer POKEMON_NUMBER=25 -> "pikachu"
- 2eme deployment : "mewtwo" - deploie une image **clevercloud/demo-pokemon** et parametise la réponse HTTP:80 pour renvoyer POKEMON_NUMBER=150 -> "mewtwo"
- 3eme deployment : "nidoqueen" - deploie une image **clevercloud/demo-pokemon** et parametise la réponse HTTP:80 pour renvoyer POKEMON_NUMBER=31 -> "nidoqueen"


## 4. Lancer les deployments
`kubectl create -f https://github.com/sozu-proxy/sozu-demo/blob/master/kubernetes-using-tube-cheese/pokemon-deployments.yaml`

## 5. Decrivez chacun des deployments 
`kubectl describe deployments`

## 6. Ecrire le fichier de configuration pour le service pour **mewtwo**  et lancer le service 
`kubectl create -f https://github.com/sozu-proxy/sozu-demo/blob/master/kubernetes-using-tube-cheese/pokemon-deployments.yaml`

## 7. Ecrire le fichier de configuration pour le service pour **nidoqueen** et lancer le service 
`kubectl create -f https://github.com/sozu-proxy/sozu-demo/blob/master/kubernetes-using-tube-cheese/pokemon-deployments.yaml`

## 8. Quelle commande utilisez vous pour décrire l'état et les détails du service ? 
`kubectl describe service`

## 9. Lancer le Ingress Controller 
`kubectl create -f https://github.com/sozu-proxy/sozu-demo/blob/master/kubernetes-using-tube-cheese/pokemon-ingress.yaml`
>>>>>>> e66179ec625094fb031e7ae99e92cb0dbf230253

## 10. Déterminer sur quelle IP publique l'application est lancée 

<<<<<<< HEAD
## 11. __OPTIONNEL__ : En analysant les champs {spec.template.spec.container.port} pour chaque type de deployment, déterminer quelle sera la réponse de chaque type de Pod à une requête HTTP port 80 ? 
- 1er  deployment : "pikachu" - deploie une image **clevercloud/demo-pokemon** et parametrise la réponse HTTP:80 pour renvoyer POKEMON_NUMBER=25 -> "pikachu"
- 2eme deployment : "mewtwo" - deploie une image **clevercloud/demo-pokemon** et parametrise la réponse HTTP:80 pour renvoyer POKEMON_NUMBER=150 -> "mewtwo"
- 3eme deployment : "nidoqueen" - deploie une image **clevercloud/demo-pokemon** et parametrise la réponse HTTP:80 pour renvoyer POKEMON_NUMBER=31 -> "nidoqueen"
=======
## 11. Connectez vous sur http://ip_publique/pikachu, http://ip_publique/mewtwo, http://ip_publique/nidoqueen
>>>>>>> e66179ec625094fb031e7ae99e92cb0dbf230253
