## 1. Completer le fichier yaml

https://github.com/Treeptik/training-k8s-resources/blob/master/02_Pods/sources/auth_pod.yaml                  

## 2. Créer le Pod en fournissant le fichier de configuration .yaml précédent en paramètres

```
kubectl create -f auth_pod.yaml  
```

## 3. Quelle est l'adresse IP du Pod "auth" ?
`kubectl describe pod auth | grep -i ip`

## 4. Sur quel Node le Pod "auth" est-il lancé ?
`kubectl describe pod auth | grep -i node`

## 5. Quel est le nom du container lancé dans le Pod "auth"
`kubectl describe pod auth | grep -i 'container id'`

## 6. Quels sont les labels attachés au Pod
`kubectl describe pod auth `

## 7. Quels sont les labels attachés au Pod ?
https://github.com/Treeptik/training-k8s-resources/blob/master/02_Pods/sources/hello_pod.yaml

## 8. Créer le Pod en fournissant le fichier de configuration .yaml précédent en paramètres
```
kubectl create -f hello_pod.yaml  
```
## 9. Quelle est l'adresse IP du Pod "hello" ?
`kubectl describe pod hello | grep -i ip`

## 10. Sur quel Node le Pod "hello" est-il lancé ?
`kubectl describe pod hello | grep -i node`

## 11. Quel est le nom du container lancé dans le Pod "hello" ?
`kubectl describe pod hello | grep -i 'container id'`

## 12. Quels sont les labels attachés au Pod ?
`kubectl describe pod hello `
