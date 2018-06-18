# Cours : Orchestration avec Kubernetes 

## Introduction 

Dans les parties précedentes nous avons manipulé les Objets de L'API Kubernetes. 
- Pod : unité de base de Kubernetes qui porte la charge de travail applicative. 
- ReplicaSet : supervise les Pods pour les maintenir dans l'état opérationnel voulu pour remplir les fonctions applicative : on parle d'orchestration.
- Deployement : s'appuit sur les ReplicatSets pour l'orchestration et offre des options pour la gestion du Cycle de vie de application (updates, tests )

Dans la suite nous allons aborder tois techniques qui s'appliquent lors de la montée en version d'une application : 
- Rolling Updates
- Canary Deployment 
- Blue-Green Deployment


## Rolling Updates 1/3 : 

Grâce au Deployment il est possible de mettre à jour __automatiquement__ chaque Pod. L'upgrade sera effectuée un Pod après l'autre de telle facon à ce que l'application ne soit pas indisponible. Ainsi au fur et à mesure de l'avancée de la mise à jour les utilisateurs se connecteront sur la nouvelle version. 


![Rolling_Update]( "Rolling_Update")


## Rolling Updates 1/3

