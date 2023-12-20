# Installation

Peut etre interessant : https://github.com/schu/kubedee

https://medium.com/@selvamraju007/setting-up-k8s-cluster-using-lxc-lxd-58f0c288af3e
https://medium.com/geekculture/a-step-by-step-demo-on-kubernetes-cluster-creation-f183823c0411

Nous allons utiliser un profile spécifique pour le cluster Kubernetes. 

Nous allons utiliser le profile suivant :

```yaml
config:
  limits.cpu: "2"
  limits.memory: 2GB
  limits.memory.swap: "false"
  linux.kernel_modules: ip_tables,ip6_tables,nf_nat,overlay,br_netfilter
  raw.lxc: "lxc.apparmor.profile=unconfined\nlxc.cap.drop= \nlxc.cgroup.devices.allow=a\nlxc.mount.auto=proc:rw
    sys:rw"
  security.nesting: "true"
  security.privileged: "true"
description: Kubernetes LXD profile
devices:
  eth0:
    name: eth0
    nictype: bridged
    parent: lxdbr0
    type: nic
  root:
    path: /
    pool: default
    type: disk
name: k8s
used_by: []
```


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

Nous allons donc créer les roles suivants :

- kubernetes
- kubernetes-master
- kubernetes-node

Après pas mal de soucis je me rends compte qu'il faut des profiles spéficique pour les noeuds du cluster Kubernetes. Je vais donc créer un profil spécifique pour le cluster Kubernetes. (Voir plus haut)

https://medium.com/geekculture/a-step-by-step-demo-on-kubernetes-cluster-creation-f183823c0411

Test :
```bash
echo 'KUBELET_EXTRA_ARGS="--fail-swap-on=false"' > /etc/default/kubeletsystemctl restart kubelet
apt install -qq -y net-tools
mknod /dev/kmsg c 1 11
echo 'mknod /dev/kmsg c 1 11' >> /etc/rc.local
chmod +x /etc/rc.local

```
Ca ne fonctionne pas, toujours erreur unable to load kernel module: "configs". 
En mettant le flag --ignore-preflight-errors=all nous avons une erreur.

cat <<EOF | sudo tee /etc/docker/daemon.json
{
"exec-opts": ["native.cgroupdriver=systemd"],
"log-driver": "json-file",
"log-opts": {
"max-size": "100m"
},
"storage-driver": "overlay2"
}
EOF