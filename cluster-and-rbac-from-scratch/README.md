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

### Playbook principal

Créer un playbook cluster-and-rbac-from-scratch.yaml
```
- name: Cluster and rbac from scratch
  hosts: localhost
  connection: local
  become: no

  vars:
    kubeconfig: "<chemin du fichier kubeconfig>"
    clusterName: "<nom du cluster>"
    serverAddress: "<adresse du serveur>"

  roles:
    - prerequisites
    - create-user
    - k8s-namespace-creation
    - k8s-role-and-rolebinding-creation
    - k8s-cert-sign-req
    - k8s-create-user-and-context
```

### Crétation des roles

#### Role: prerequisites

Se placer dans le répertoire playbooks/roles puis lancer
```
ansible-galaxy init prerequisites
```

Mofifier le fichier plabooks/roles/prerequisites/task/main.yml
```
- name: Install pip, cfssl, cfssljson
  apt:
    name: "{{ item }}"
  with_items:
    - python-pip
    - golang-cfssl

- name: Install openshift
  pip:
    name: openshift
    state: present
```

#### Role: create-user

Se placer dans le répertoire playbooks/roles puis lancer
```
ansible-galaxy init create-user
```

Mofifier le fichier plabooks/roles/create-user/task/main.yml
```
- name: Create user with cfssl
  shell: echo '{"CN":"treeptik.student","hosts":[""],"key":{"algo":"rsa","size":2048}}' | cfssl genkey  - | cfssljson -bare files/treeptik.student

- name: Set csr value
  shell: cat files/treeptik.student.csr | base64 | tr -d '\n'
  register: request
```

#### Role: k8s-namespace-creation

Se placer dans le répertoire playbooks/roles puis lancer
```
ansible-galaxy init k8s-namespace-creation
```

Mofifier le fichier plabooks/roles/k8s-namespace-creation/task/main.yml
```
- name: Create a k8s namespace "treeptik-namespace"
  k8s:
    kubeconfig: "{{ kubeconfig }}"
    name: treeptik-namespace
    api_version: v1
    kind: Namespace
    state: present
```

#### Role: k8s-role-and-rolebinding-creation

Se placer dans le répertoire playbooks/roles puis lancer
```
ansible-galaxy init k8s-role-and-rolebinding-creation
```

Mofifier le fichier plabooks/roles/k8s-role-and-rolebinding-creation/task/main.yml
```
- name: Create a k8s Role "pod-reader"
  k8s:
    state: present
    kubeconfig: "{{ kubeconfig }}"
    definition:
      apiVersion: rbac.authorization.k8s.io/v1
      kind: Role
      metadata:
        namespace: treeptik-namespace
        name: pod-reader
      rules:
      - apiGroups: ["", "extensions", "apps"]
        resources: ["deployments", "replicasets", "pods"]
        verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]

- name: Create a k8s RoleBinding "read-pod"
  k8s:
    state: present
    kubeconfig: "{{ kubeconfig }}"
    definition:
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

#### Role: k8s-cert-sign-req

Se placer dans le répertoire playbooks/roles puis lancer
```
ansible-galaxy init k8s-cert-sign-req
```

Mofifier le fichier plabooks/roles/k8s-cert-sign-req/task/main.yml
```
- name: Remove a k8s CertificateSigningRequest
  k8s:
    state: absent
    kubeconfig: "{{ kubeconfig }}"
    api_version: certificates.k8s.io/v1beta1
    kind: CertificateSigningRequest
    namespace: default
    name: user-request-treeptik-student

- name: Create a k8s CertificateSigningRequest
  k8s:
    state: present
    kubeconfig: "{{ kubeconfig }}"
    definition:
      apiVersion: certificates.k8s.io/v1beta1
      kind: CertificateSigningRequest
      metadata:
        name: user-request-treeptik-student
      spec:
        groups:
        - system:authenticated
        request: "{{ request.stdout }}"
        usages:
        - digital signature
        - key encipherment
        - client auth

- name: Validate k8s CertificateSigningRequest
  shell: kubectl --kubeconfig "{{ kubeconfig }}" certificate approve user-request-treeptik-student

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
ansible-playbook playbooks/cluster-and-rbac-from-scratch.yaml
```

Le nouveau kubeconfig créer est le fichier : playbooks/treeptik-student-config






