## RBAC avec Ansible

### Fichiers de configurations

Créer un fichier ansible.cfg
```
[defaults]
inventory = hosts
host_key_checking = False
retry_files_enabled = False
gather_facts = True
forks = 100
```

Créer un fichier host
```
cluster-master-1 ansible_host=<ip master>
cluster-node-1 ansible_host=<ip node 1>
cluster-node-2 ansible_host=<ip node 2>

[user1]
cluster-master-1
cluster-node-1
cluster-node-2

[master]
cluster-master-1

[node]
cluster-node-1
cluster-node-2

[all:vars]
ansible_ssh_user=centos
ansible_ssh_private_key_file=id_rsa_formation_tp
```

Créer un dossier playbooks
```
mkdir playbooks
```

Créer un dossier playbooks/files
```
mkdir playbooks/files
```

Créer un dossier plabooks/roles
```
mkdir playbooks/roles
```

Créer un dossier plabooks/library
```
mkdir playbooks/library
```

Ajouter le module helm_shell:
```
git clone https://github.com/hypi-universe/hypi-ansible-modules.git
mv helm_args.json playbooks/library
mv helm_shell playbooks/library
```

### Playbook principal

Créer un playbook cluster-and-rbac-from-scratch.yaml
```
- name: Cluster and rbac from scratch
  hosts: localhost
  connection: local

  vars:
    kubeconfig: "/home/malo/Documents/Kubeconfig/kube-prov.yaml"
    clusterName: "cluster"
    serverAddress: "https://142.93.96.139:6443"

  roles:
    - role: prerequisites
      become: yes
    - role: create-user
      become: no
    - role: helm-configuration
      become: no
    - role: helm-deployment
      become: no
    - role: k8s-create-user-and-context
      become: no
```

### Crétation des roles

#### Role: prerequisites

Se placer dans le répertoire playbooks/roles puis lancer
```
ansible-galaxy init prerequisites
```

Mofifier le fichier plabooks/roles/prerequisites/task/main.yml
```
- name: Install pip, openssl
  package:
    state: present
    name: "{{ item }}"
  loop:
    - python3-pip
    - openssl

- name: Install grpcio, pyhelm, pyopenssl
  pip:
    state: present
    name: "{{ item }}"
  loop:
    - grpcio
    - pyhelm
    - pyopenssl
    - openshift
```

#### Role: create-user

Se placer dans le répertoire playbooks/roles puis lancer
```
ansible-galaxy init create-user
```

Mofifier le fichier plabooks/roles/create-user/task/main.yml
```
- name: Generate an user key
  openssl_privatekey:
    path: files/treeptik.student-key.pem
    size: 2048

- name: Generate the user csr
  openssl_csr:
    path: files/treeptik.student.csr
    privatekey_path: files/treeptik.student-key.pem
    common_name: treeptik.student

- name: Set csr value
  shell: cat files/treeptik.student.csr | base64 | tr -d '\n'
  register: request
```

#### Role: helm-configuration

Se placer dans le répertoire playbooks/roles puis lancer
```
ansible-galaxy init helm-configuration
```

Mofifier le fichier plabooks/roles/helm-configuration/task/main.yml
```
- name: Create service account tiller
  k8s:
    state: present
    kubeconfig: "{{ kubeconfig }}"
    definition:
      kind: ServiceAccount
      apiGroup: rbac.authorization.k8s.io
      metadata:
        name: tiller
        namespace: kube-system

- name: Associate tiller with cluster-admin
  k8s:
    state: present
    kubeconfig: "{{ kubeconfig }}"
    definition:
      kind: ClusterRoleBinding
      apiGroup: v1.ClusterRoleBinding
      metadata:
        name: tiller-cluster-rule
      subjects:
      - kind: ServiceAccount
        name: tiller
        namespace: kube-system
        apiGroup: ""
      roleRef:
        kind: ClusterRole
        name: cluster-admin
        apiGroup: ""

- name: Init helm
  command: helm --kubeconfig "{{ kubeconfig }}" init --upgrade --service-account tiller
```

#### Role: helm-deployment

Se placer dans le répertoire playbooks/roles puis lancer
```
ansible-galaxy init helm-deployment
```

Se placer dans le répertoire playbooks/files puis lancer
```
helm create rbac-chart
```

Se placer dans le répertoire playbooks/files/rbac-chart/charts puis lancer
```
helm create rbac-namespace-chart
```

Supprimer tout les fichiers que contient le dossier playbooks/files/rbac-chart/charts/rbac-namespace-chart/templates et y ajouter le fichier namespace.yaml
```
kind: Namespace
apiVersion: v1
metadata:
  name: treeptik-namespace
```

Supprimer tout les fichiers que contient le dossier playbooks/files/rbac-chart/templates

Ajouter le fichier playbooks/files/rbac-chart/templates/role.yaml
```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: treeptik-namespace
  name: pod-reader
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["deployments", "replicasets", "pods"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```

Ajouter le fichier playbooks/files/rbac-chart/templates/role-binding.yaml
```
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-pod
  namespace: treeptik-namespace # Nom du namespace
subjects:
- kind: User
  name: treeptik.student
  apiGroup: ""
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: ""
```

Ajouter le fichier playbooks/files/rbac-chart/templates/cert-sign-req.yaml
```
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: user-request-treeptik-student
spec:
  groups:
  - system:authenticated
  request: "{{ .Values.cert.request }}"
  usages:
  - digital signature
  - key encipherment
  - client auth
```

Mofifier le fichier plabooks/roles/helm-deployment/task/main.yml
```
- name: Save old configuration
  command: mv $HOME/.kube/config $HOME/.kube/config.old

- name: Set Kubeconfig for helm
  command: cp {{ kubeconfig }} $HOME/.kube/config

- name: Copy request value into rbac chart
  command: 'echo "cert:\n  request: {{request.stdout}}" >> files/rbac-chart/values.yaml'

- name: Install helm chart
  helm_shell:
    name: rbac-chart
    version: 0.1.0
    source:
      type: directory
      location: "files/rbac-chart"

- name: Reset configuration
  command: mv $HOME/.kube/config.old $HOME/.kube/config

- name: Validate k8s CertificateSigningRequest
  command: kubectl --kubeconfig "{{ kubeconfig }}" certificate approve user-request-treeptik-student

- name: Get the new certificate
  shell: kubectl --kubeconfig "{{ kubeconfig }}" get csr user-request-treeptik-student -o jsonpath='{.status.certificate}' | base64 -d > files/treeptik.student.pem
```

#### Role: k8s-create-user-and-context

Se placer dans le répertoire playbooks/roles puis lancer
```
ansible-galaxy init k8s-create-user-and-context
```

Mofifier le fichier plabooks/roles/k8s-create-user-and-context/task/main.yml
```
- name: Credential creation
  shell: >
    kubectl config set-credentials treeptik.student
    --kubeconfig treeptik-student-config
    --embed-certs=true
    --client-certificate=files/treeptik.student.pem
    --client-key=files/treeptik.student-key.pem

- name: Context creation
  shell: >
    kubectl config set-context
    --kubeconfig treeptik-student-config treeptik-context
    --cluster="{{ clusterName }}"
    --user=treeptik.student
    --namespace=treeptik-namespace

- name: Context selection
  shell: kubectl config use-context --kubeconfig treeptik-student-config treeptik-context

- name: Cluster selection
  shell: >
    kubectl config set-cluster
    --kubeconfig treeptik-student-config "{{ clusterName }}"
    --insecure-skip-tls-verify=true
    --server="{{ serverAddress }}"
```

### Lancer le playbooks et admirer le résultat
```
sudo ansible-playbook playbooks/cluster-and-rbac-from-scratch.yaml
```

Le nouveau kubeconfig créer est le fichier : playbooks/treeptik-student-config
