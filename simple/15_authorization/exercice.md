#### Exercice

Nous allons maintenant créer un "espace étanche" sur votre cluster pour l'utilisateur "treeptik". 

- Se connecter en ssh au node master du cluster:
~~~
ssh -i <path du certificat> <nom d'utilisateur>@<ip du node master>
~~~

- Générez la clé privée du nouvel utilisateur:
~~~bash
openssl genrsa -out treeptik.key 2048
~~~

- Créez une demande de certificat (attribut CN : group et O : utilisateur) :
~~~bash
openssl req -new -key treeptik.key -out treeptik.csr \
-subj "/CN=treeptik-group/O=treeptik"
~~~


- Validez la demande de certificat avec le certificat administrateur (ca.pem, ca-key.pem) :
~~~bash
sudo openssl x509 -req -in treeptik.csr -CA /etc/kubernetes/ssl/ca.pem \
-CAkey /etc/kubernetes/ssl/ca-key.pem -CAcreateserial -out treeptik.crt -days 500
~~~

- Créez le namespace sur lequel l'utilisateur treeptik pourra agir:
~~~bash
kubectl create namespace treeptik-namespace
~~~

- Créez le nouvel utilisateur dans kubernetes:
~~~bash
kubectl config set-credentials treeptik --client-certificate=/home/centos/treeptik.crt \
--client-key=/home/centos/treeptik.key
~~~

- Créez le contexte associé à notre nouvel utilisateur:
~~~bash
kubectl config set-context treeptik-context \
--namespace=treeptik-namespace --user=treeptik
~~~

- Changez de context pour celui nouvellement créez:
~~~bash
kubectl config use-context treeptik-context
~~~

- Exportez la config dans un fichier :
~~~bash
kubectl config view --flatten > kubeconfig.cfg
~~~

- Récupérez l'adresse ip du serveur et le certificat administrateur:
~~~bash
sudo cat /etc/kubernetes/admin.conf | grep server
sudo cat /etc/kubernetes/admin.conf | grep certificate-authority-data
~~~

- Dans le fichier kubeconfig.cfg remplacer la section clusters par :

~~~bash
clusters:
- cluster:
    certificate-authority-data: <le certificat administrateur>
    server: <adresse du serveur>
~~~

- Créez le role :

~~~bash
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: treeptik-namespace
  name: pod-reader
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["deployments", "replicasets", "pods"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
~~~

- Créez le roleBinding:

~~~bash
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-pod
  namespace: treeptik-namespace # Nom du namespace
subjects:
- kind: User
  name: treeptik
  apiGroup: ""
roleRef:
  kind: Role 
  name: pod-reader
  apiGroup: ""
~~~

- Ajoutez les au cluster:

~~~bash
kubectl create -f role.yaml -f role-binding.yaml
~~~

- Récupérez le fichier kubeconfig.cfg sur votre machine.
- Configurez la variable d'environnement KUBECONFIG pour la faire pointer sur kubeconfig.cfg: 
~~~bash
export KUBECONFIG=<path>/kubeconfig.cfg
~~~
- Lancez un deployment depuis votre machine:
~~~bash
kubectl run --image bitnami/dokuwiki mydokuwiki
~~~
- Affichez les ressources créees:
~~~bash
kubectl get pods,deployment
~~~
- Vérifiez que vous ne pouvez pas afficher de ressources du namespace "default":
~~~bash
kubectl auth can-i get deployments --namespace default
~~~
