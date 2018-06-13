## 1. Donner le nombre de POD qui sera lancé lors de la creation du ReplicaSet avec ce fichier de configuration
- 3
## 2. Trouver la commande kubectl pour créer ce ReplicatSet.
`kubectl create -f soaktest_replicat.yaml`
## 3. Quelle Image Docker est utilisée ?
`nickchase/soaktest`
## 4. Quel sont les labels associés au Pod qui sera orchestré par ce ReplicaSet ?
`app&track`
## 5. Expliquer le selector pour ce ReplicaSet ? A comparer avec la réponse du 4.
- Le selector va matcher les label *app* ayant pour valeur *soaktest1, soaktest2, soaktest*
- Cependant on exclu les objets ayant le label track avec les valeures production & staging
## 6. Ce ReplicatSet est-il destiné à la Production ?
- Non, Le selector exlu la production et notre pod a le label : track: dev.
## 7. Contruire le fichier de configuration du ReplicatSet pour le Pod "auth" 
- cf ../reponses/auth_replicatset.yaml
## 8. Contruire le fichier de configuration du ReplicatSet pour le Pod "hello"
- cf ../reponses/hello_replicatset.yaml
## 9. Contruire le fichier de configuration du ReplicatSet pour le Pod "frontend"
- cf ../reponses/frontend_replicatset.yaml
