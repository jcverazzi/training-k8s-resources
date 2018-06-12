# Exercice : le CLI "kubectl" 

Dans cet exercice nous allons introduire et manipuler les composants de bases de Kubernetes avec le **CLI "kubectl"**.
Ces composants seront décrits en détails dans la suite de la formation. 

__Les applications sont au coeur de Kubernetes__.

Chaque fonction de l'application (frontend, backend... ) peut avantageusement utiliser la containeurisation - avec Docker par exemple. 

C'est là que le système Kubernetes rentre en scène. 
Son rôle: piloter harmonieusement l'ensemble des containers formant l'application. 

Pour maintenir le cap tout au long du cycle de vie de l'application, Kubernetes utilise des composants :
* Pods
* ReplicatSet 
* Deployments... 

Ces composants permettent via une syntaxe déclarative de fournir les capacités d'automatisation et d'orchestration. Cette approche en parfaite correspondance avec les bonnes pratiques DevOps. 

Cet exercice se focalise sur le set de commandes *kubectl* et nous soumettrons directement via le CLI les options a passer au système K8s. Plus tard nous envisagerons le provisionning d'applications, qui consiste à automatiser le déploiement des *Pods* et des objects nécessaires à leur orchestration, au travers de fichiers de configurations *.yaml* . 

## Objectifs 

A la fin de l'exercice nous aurons déployé __en ligne de commande via *kubectl*__ un groupe de containers Nginx sur Kubernetes telle que :

![Architecture Cible de l'exercice](https://github.com/Treeptik/training-k8s-resources/blob/master/01_kubectl/images/Treeptik-training-k8s-exo1.jpg?raw=true "Architecture Cible de l'exercice")


## 1 - Cluster, Pod, Deployment sur Kubernetes 

Se connecter au Cluster K8s de votre environnement de lab provisionné précedemment. 

- En vous aidant de la completion venant avec CLI "kubectl" et de la documentation de *kubetcl* :
1. Lister toutes les options disponibles avec la commande kubectl. 
2. Lister tous les objets disponibles en option de `kubectl get`
3. Quel est le résultat de la commande ? Commentez : 
 
`kubectl get pods `

4. Utiliser *kubectl* pour lancer un *Pod* avec les options : 
* nom du POD : nginx_pod
* image : nginx
* port : 80

La sortie de la commande devrait afficher : 

`deployment.apps "nginx" created `

Kubernetes a crée automatiquement un "Deployment". Cet objet, qui sera etudié plus tard, est utilisé pour orchestrer le POD nginx tout au long de son cycle de vie : create, scale... .

5. Quel est le résultat de la commande :  
 
`kubectl get pods `

Etudions les informations données par la sortie de ces commandes - en particulier : 

6. Identifier la Colonne "NAME", à votre avis que représente la première ligne *nginx-.....* ?
7. Identifier la Colonne "READY", à votre avis à quoi correspondent les chiffres : "1/1" ? 

- Le Status du POD est normalement en mode '"Running". En vous aidant de la documentation (trouvée sur Internet)
8. Trouver la liste de tous les *status* possible dans le cycle de vie d'un POD. 
9. Trouver la commande qui affiche les logs temps reels du pod "nginx"
10. Est-il possible de voir les logs du conteneur executé dans le pod "nginx" - comment ? Trouver la commande. 





