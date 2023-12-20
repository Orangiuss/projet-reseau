# Installation

Nous commencons par reprendre le playbook ansible de l'installation de 3 machines lxd. Puis nous allons faire l'installation du cluster Kubernetes.

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
- name: Docker/Kubernetes roles
  hosts: lxd_containers
  connection: lxd
  roles:
    - docker
    - kubernetes

- name: Kubernetes master roles
  hosts: mgr
  connection: lxd
  roles:
    - kubernetes-master

- name: Kubernetes worker roles
  hosts: worker
  connection: lxd
  roles:
    - kubernetes-node
```

Nous recuperons le playbook pour l'installation de Docker. (TP2)

Pour Kubernetes nous allons nous appuyer sur plusieurs sources d'informations :
https://github.com/torgeirl/kubernetes-playbooks/blob/master/playbooks/kube-dependencies.yml
https://gist.github.com/allanger/84db2647578316f8e721f7219052788f
https://github.com/torgeirl/kubernetes-playbooks
