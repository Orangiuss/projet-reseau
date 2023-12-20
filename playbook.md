# Installation

Nous commencons par reprendre le playbook ansible de l'installation de 3 machines lxd

Nous ajoutons les roles suivants :

- jenkins
- docker
- kubernetes
- kubernetes-master
- kubernetes-node

Pour cela voici les commandes à executer :

```bash 
ansible-galaxy init --offline roles/jenkins
ansible-galaxy init --offline roles/docker
ansible-galaxy init --offline roles/kubernetes
ansible-galaxy init --offline roles/kubernetes-master
ansible-galaxy init --offline roles/kubernetes-node
```

Nous ajoutons les roles dans le fichier `playbook.yml` :

```yaml
- hosts: all
  become: true
  roles:
    - lxd
    - jenkins
    - docker
    - kubernetes
    - kubernetes-master
    - kubernetes-node
```