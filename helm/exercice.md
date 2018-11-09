## Utiliser Helm pour déployer une Application Kubernetes

### Nettoyer l'environnement précédent

`kubectl delete daemonsets,replicasets,services,deployments,pods,rc --all`

## Créer un chart

`helm create chart-exercice`

## Supprimer l'application demonstration

Supprimer les fichiers :
- templates/deployment.yaml
- templates/ingress.yaml
- templates/service.yaml

## Récupérer les fichiers de configurations de l'Application Wordpress

`git clone https://github.com/Treeptik/training-k8s-resources.git`
Récupérer tout les fichiers dans le dossier : wordpress-mysql/solution
Les copier dans notre chart : chart-exercice/templates

## Lancer le déploiement

`helm install --name chart-exercice chart-exercice/`

## Exposer le frontend sur votre localhost

`kubectl port-forward <nom du pod> 8080:80`

Vous devriez pouvoir accéder à wordpress à l'adresse : 127.0.0.1:8080

## Utiliser les variables de helm dans les templates

Supprimer le contenus du fichier values.yaml
Le remplacer par :
mysql:
  image:
    repository: mysql
    tag: 5.6

wordpress:
  image:
    repository: wordpress
    tag: 4.8

Dans le fichier wordpress-deployment.yaml remplacer l'image du container par :
"{{ .Values.wordpress.image.repository }}:{{ .Values.wordpress.image.tag }}"

Dans le fichier mysql-deployment.yaml remplacer l'image du container par :
"{{ .Values.mysql.image.repository }}:{{ .Values.mysql.image.tag }}"

## Relancer le déploiement

`helm del chart-exercice --purge`
`helm install --name chart-exercice chart-exercice/`

`kubectl port-forward <nom du pod> 8080:80`


