## Créer son premier chart

### Génération de fichiers de ressources K8S

Avant de créer son premier chart, il est nécessaire d'avoir des fichiers ressources pour K8S.
Une solution élégante est de les générer avec l'outil kubectl.

```
$ mkdir manifests
$ kubectl run example --image=nginx:1.13.5-alpine \
                      -o yaml > manifests/deployment.yaml
$ kubectl expose deployment example
                --port=80
                --type=NodePort \
                -o yaml > manifests/service.yaml
```

Il est possible d'accéder au déploiement via le NodePort exposé.
