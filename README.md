# Provision a Docker Swarm Cluster

## Introduction

Provision a Docker Swarm cluster composed of two masters and two workers using Vagrant and Ansible.

## Quick Installation

To create all the virtual machines and provision with the default swarm stacks execute:

```bash
vagrant plugin install vagrant-hostmanager

vagrant up
```

If the provision of the swarm stack fails you can execute only the provision part:

```bash
vagrant provision
```

If you want to add more hosts (masters and/or workers) just edit both files:

* Vagrantfile
* ansible/inventory

After that just run

```bash
vagrant up --provision
```

After this you can connect to one of the master nodes and check the cluster status

```bash
vagrant ssh swarm-master-1

docker node ls
```

## Description

This project has docker-compose files to create swarm stacks. The following stacks are available:

* Traefik/Consul Stack
* Web User Interface Stack
* Automation Stack

### Web User Interface Stack

Web User Interface Stack provides the following:

| DevOps Tool | URL | Description |
| ----------- | --- | ----------- |
| [Swarmpit](https://swarmpit.io/) | http://swarm-master-1:888 | Operate your docker infrastructure like a champ |
| [Portainer](https://www.portainer.io/) | http://swarm-master-1:9000 | Lightweight Docker Swarm management UI |

There are 2 docker-compose files:

- docker-compose.swarmpit.yml (this is automatically installed)
- docker-compose.portainer.yml

## Installing the stacks with traefik

You can also install all the stack and use traefik. 
This stack uses traefik and consul and handles HTTPS certificates automatically (including renewals) with Let's Encrypt.

To install the traefik stack run the following commands on the swarm manager:

```bash
vagrant ssh swarm-master-1

docker stack rm web_ui
docker stack rm automation
```

Edit the environment variables under ```ansible/traefik_consul-playbook.yml``` and execute the playbook

```bash
ansible-playbook -i ansible/inventory ansible/deploy-traefik-stack.yml --tags traefik
```

If you want to use docker-compose outside ansible you need to define some environment variables

```bash
vagrant@swarm-master-1:~$ export DOMAIN=intra.local.com
vagrant@swarm-master-1:~$ export USERNAME=admin
vagrant@swarm-master-1:~$ export PASSWORD=admin
vagrant@swarm-master-1:~$ export HASHED_PASSWORD=$(openssl passwd -apr1 $PASSWORD)
vagrant@swarm-master-1:~$ docker stack deploy -c docker-compose.swarmpit.traefik.yml web_ui
```

Make sure you have the following DNS entries:

| DevOps Tool | URL |
| ----------- | --- |
| [Traefik](https://traefik.io/) | http://traefik.intra.local.com |
| [Consul](https://www.consul.io/) | http://consul.intra.local.com |
| [Swarmpit](https://swarmpit.io/) | http://swarmpit.intra.local.com |
| [Portainer](https://www.portainer.io/) | http://portainer.intra.local.com |

## @TODO

* Vagrantfile reads nodes from ansible/inventory
* Use traefik.toml file to configure traefik
* Add Portworx support for persistent storage (https://portworx.com/use-case/docker-persistent-storage/)

## External Links for reference

* https://dockerswarm.rocks/
* https://github.com/swarmstack/swarmstack
* https://github.com/acuto/swarm-vagrant-ansible
* https://github.com/stefanprodan/swarmprom
* https://github.com/geerlingguy/ansible-vagrant-examples
