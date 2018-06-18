## Se protéger avec les secrets 

### Nettoyer l'environnement précédent

`kubectl delete daemonsets,replicasets,services,deployments,pods,rc --all`

### Creer son premier secret

Nous allons écrire notre secret dans un fichier texte.
       
```
echo -n "Radiohead - Karma Police" > ./chanson.txt
```

Ensuite nous allons créer un secret sur base de ce fichier (1 Mo Max pour rappel)
```
kubectl create secret generic chanson --from-file=./chanson.txt
secret "chanson" created
```






