# Fonctionnement de Jenkins

Sections tools we need to setup :
https://isurunuwanthilaka.medium.com/docker-container-deployment-with-jenkins-and-ansible-cf8ef1eaae96

## Installation de Jenkins

Nous pouvons installer Jenkins via un playbook ansible comme ceci :

Nous devons installer les paquets suivants :

sudo apt update
sudo apt install fontconfig openjdk-17-jre
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins

En ansible cela fait :
    
```yaml
- name: Install Jenkins
  hosts: lxd_containers
  connection: lxd
  become: yes
  tasks:
    - name: Install packages
      apt:
        name: "{{ packages }}"
        state: present
      vars:
        packages:
          - fontconfig
          - openjdk-17-jre
    - name: Add Jenkins key
      apt_key:
        url: https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
        state: present
    - name: Add Jenkins repository
      apt_repository:
        repo: deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/
        state: present
    - name: Install Jenkins
      apt:
        name: jenkins
        state: present
```