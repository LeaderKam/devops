# UR1/ESIR DevOps Course
This repository contains the material and content of the DevOps course at the engineering school ESIR of the University of Rennes 1. 

## Year 2020-2021

### Scheduling

Introduction to the course and DevOps: Nov. 23rd, 2020 (8h-10h)

Office hours: 
- 23/11 16-18 (GJ)
- 26/11 10-12 (BC)
- 27/11 10-12 (GJ)
- 27/11 14-16 (BC)
- 30/11 10-12 (GJ)

Final presentations: Dec. 10th, 2020 (16h-18h)

### Material

The introduction to the course and DevOps can be found [here](https://people.irisa.fr/Benoit.Combemale/course/esir/esir3/). 
We meet on the dedicated team on MS Teams. 



## Configuration Management :
Configuration management occurs when a configuration platform is used to automate, monitor, design and manage otherwise manual configuration processes. System-wide changes take place across servers and networks, storage, applications, and other managed systems.
Configuration management tools like Chef, Puppet, and the others help configure the software and systems on this infrastructure that has already been provisioned.



## Infrastructure as Code:
Infrastructure as Code (IaC) is the management of infrastructure (networks, virtual machines, load balancers, and connection topology) in a descriptive model, using the same versioning as DevOps team uses for source code. Like the principle that the same source code generates the same binary, an IaC model generates the same environment every time it is applied. IaC is a key DevOps practice and is used in conjunction with continuous delivery.

## Before Ansible
Before launching on Ansible, we first wanted to discover Terraform one of the tools of Infrastructure as Code.
Terraform is an infrastructure provisioning tool created by Hashicorp. It allows you to describe your infrastructure as code, creates “execution plans” that outline exactly what will happen when you run your code, builds a graph of your resources, and automates changes with minimal human interaction.
We wanted to use this tool to be able to launch our application on multiple AWS (Amazon Web Service) nodes. We have created an academic account on AWS but unfortunately it only gives access to certain online training courses not to the allocation of nodes on the Amazon cloud.


## Why Ansible 
Setting up a web server is a tedious and complicated task to perform. and setting up tens is even more painful. In this tutorial, we are going to offer you a solution to automate the deployment of servers which consists concretely in automating the installation of tools and their configurations.<br>
There are quite a few tools that do this job. We can cite Chef, Puppet, Ansible and SALT. We chose Ansible for this solution because when we compared the different tools, we found that it is the easiest to do. It took a little while to figure out how it works, but in real life you just have to learn how to write the config file and then it will work.<br>



## Instaling Ansible on ubuntu : 
Ubuntu builds are available in PPA<br>
To configure the PPA on your machine and install Ansible run these commands:
```Shell
$ sudo apt update
$ sudo apt install software-properties-common
$ sudo apt-add-repository --yes --update ppa:ansible/ansible
$ sudo apt install ansible
```
## Réalisation du fichier de configuration
In this section we will show you how to make the configuration file. the following steps will create the contents of the configuration file which will allow you to automate the installation of the tools necessary for the launch of the application. 

At the beginning We need to create the header of our file which contains the important information for launching Ansible (name, server, connection ...) and also a variable which will be used for the installation of NodeJs.

```Ansible
---
- name: installation des serveur
  hosts: localhost
  become: yes
  connection: local
  vars:
    NODEJS_VERSION: "12"
    ansible_distribution_release: "xenial" #trusty
  tasks:
```
Once the summer is created, we can then move on to coding the tasks that we would like ansible to perform. <br>
The first task consists in uninstalling the old versionsdocker if it exists on the system and reinstalling it to have the latest updates. then configure it thanks to the following series of commands:
```Ansible
    #Install using the repository  https://docs.docker.com/engine/install/ubuntu/
    #SET UP THE REPOSITORY
    
    - name: Uninstall old versions
      command: sudo apt-get remove docker docker-engine docker.io containerd runc
      ignore_errors: yes
    - name: Installation of docker packages
      apt: name={{item}} update_cache=no state=latest
      with_items:
        - apt-transport-https
        - ca-certificates
        - curl
        - gnupg-agent
        - software-properties-common
    - name: Add Docker official GPG key
      shell: curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
      # become: yes
      # become_user: root
    - name: Verify that you now have the key with the fingerprint
      command: sudo apt-key fingerprint 0EBFCD88
    # - name: set up the stable repository
      # command: sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu xenial stable"
    #INSTALL DOCKER ENGINE
    - name:  install the latest version of Docker Engine and containerd
      apt: name=docker.io update_cache=no state=latest
    - name: update
      command: sudo apt update
    - name: install docker-compose
      shell: sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    - name: Access
      shell: sudo chmod +x /usr/local/bin/docker-compose
    #can be avoid
    - name: Fingerprint
      command: apt-cache madison docker-ce

    - name: Verify that Docker Engine is installed correctly
      command: sudo docker run hello-world
```
Still in the same file, this time we are going to install NodeJs and configure it to be able to launch our application (Front and Back)
```Ansible    
    #For the front management
    #- name: Nodejs installation for front launch
    #  shell: sudo apt install nodejs
    - name: Install the gpg key for nodejs LTS
      apt_key:
        url: "https://deb.nodesource.com/gpgkey/nodesource.gpg.key"
        state: present

    - name: Install the nodejs LTS repos
      apt_repository:
        repo: "deb https://deb.nodesource.com/node_{{ NODEJS_VERSION }}.x {{ ansible_distribution_release }} main"
        state: present
        update_cache: yes

    - name: Install the nodejs
      apt:
        name: nodejs
        state: latest

    #- name: Npm installation for node management
     # command: sudo apt install npm
```
Once all the previous installations are done, we will move on to the step of launching the application, starting by launching the Backend with the following commands:
```Ansible

    #close docker images
    - name: close docker
      shell: chdir=api/ sudo docker-compose down
      ignore_errors: yes

    #launching docker and after the api
    - name: detach docker
      shell: chdir=api/ sudo docker-compose up --detach

    #launching the api
    - name: Launching the api
      shell: chdir=api/ sudo bash ./mvnw compile quarkus:dev &
```
Launch the front
```Ansible
    #launch front
    - name: node module installation
      command: chdir=front/ npm i
    - name: Launching the front
      shell: chdir=front/ npm start
...
```
We now come to the end of creating our configuration file, which you will find in full [here](https://github.com/LeaderKam/TP_DLC/blob/master/playbook.yml). Manitenant it only remains to execute it and let's go. all the environment our application needs will be installed and configured.<br>
So that the created file can be executed correctly we have chosen to place it at the root of the project where there are the Front and API folders
```Ansible
#sudo ansible-playbook -i hosts, playbook.yml --connection=local

```
## If we had time

If we had a little more time, We would have liked to improve our configuration to be able to run our configuration file on remote machines (server, workstation)


## Resources 

You will find [here ](https://drive.google.com/file/d/1yBFzYDzCPUUMISpf_p_w-t_0JgERWp1J/view?usp=sharing)the presentation of some steps of implementation.<br>

You will find the Github link [here ](https://github.com/LeaderKam/TP_DLC/tree/master?fbclid=IwAR3ICdgru67ELRJBjV-jiyqpdfI3l3U8eI9oqH99MX48fe1JjRE6rvYXjfA).<br>
Click [here ](https://drive.google.com/file/d/1vjk2EngyNy5hNG7E2s485NWFXEs_EMaC/view?usp=sharing)to see the [demo](https://drive.google.com/file/d/1vjk2EngyNy5hNG7E2s485NWFXEs_EMaC/view?usp=sharing).




