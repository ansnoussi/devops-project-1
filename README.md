# devops-project-1

A simple web app, but with all the devops bells and whistles (Vagrant, Ansible, Docker Swarm, Portainer)

## Overview

![Architecture Description](./overview.drawio.svg)

## Setup

We will provision using Vagrant 1 control vm and 3 nodes for deployment from VirtualBox.
(the `control vm` will connect to the `nodes` using Ansible to apply the correct configuration)

**Note:** The ip addresses as described in the Vagrantfile should belong to one of the subnets in the VirtualBox Host Network Manager ( `File` -> `Host Network Manager` )

**Note:** Everything in the folder with the `Vagrantfile` is located will be shared to the vagrant host (in `/vagrant`) .

## Step by step guide:

1. `vagrant up` to provision the vms from virtualbox.
2. `vagrant ssh <machine_name>` to connect to the vm via ssh (example `vagrant ssh node1`).
3. copy the hosts file in the control machine so we can have name resolution available :

- `vagrant ssh control`
- `sudo cp /vagrant/hosts /etc/hosts`
- test with : `ping node1`

4. ssh connectivity should be available by default :

- from control : `ssh vagrant@node1` with password `vagrant`

5. if you want to enable ssh with key from control to different nodes:

- generate ssh key : `ssh-keygen`
- copy key to different nodes : `ssh-copy-id node1 && ssh-copy-id node2 && ssh-copy-id node3`
- ssh into any node without password : `ssh node1`

6. install ansible in the control vm:

```
sudo apt install software-properties-common
sudo apt-add-repository ppa:ansible/ansible
sudo apt update
sudo apt install ansible
```

7. check for connectivity with ansible : `ansible nodes -i myhosts -m command -a hostname` (`-i` : inventory, `-m` : module, `-a` : argument for the chosen module)
   => expected output:

```bash
node1 | SUCCESS | rc=0 >>
node1

node2 | SUCCESS | rc=0 >>
node2

node3 | SUCCESS | rc=0 >>
node3

```

8. install python-simplejson in all of the nodes:

```
ansible nodes -i myhosts -m command -a 'sudo apt-get update'
ansible nodes -i myhosts -m command -a 'sudo apt-get -y install python-simplejson python3-pip mc'
```

9. run the playbook to install docker: `ansible-playbook -i myhosts -K playbook1.yml`
   => expected output:

```bash
PLAY [nodes] **********************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************************
ok: [node2]
ok: [node1]
ok: [node3]

TASK [ensure docker is installed] *************************************************************************************************************************************************
changed: [node2]
changed: [node1]
changed: [node3]

TASK [ensure docker compose is installed] *****************************************************************************************************************************************
changed: [node2]
changed: [node1]
changed: [node3]

TASK [add the user vagrant to the docker group] ***********************************************************************************************************************************
changed: [node1]
changed: [node2]
changed: [node3]

PLAY RECAP ************************************************************************************************************************************************************************
node1                      : ok=4    changed=3    unreachable=0    failed=0
node2                      : ok=4    changed=3    unreachable=0    failed=0
node3                      : ok=4    changed=3    unreachable=0    failed=0
```

10. install and configure docker swarm on the nodes: `ansible-playbook -i myhosts -K swarm-playbook/swarm.yml`

11. deploy the stack in one of the nodes:

- `cd /vagrant/docker`
- `docker stack deploy --compose-file docker-compose-remote.yml my-flask-app`

**Note:** when using docker swarm, the image needs to be hosted on a container registry.
=> in the docker folder run:

- `docker build -t anissnoussi/simpleflask:1.0 .`
- `docker push anissnoussi/simpleflask:1.0`

12. when finished, just stop and destroy all the vms: `vagrant destroy` .

## Installing portainer :

- create an external overlay network

```
docker network create --driver overlay net-public
```

- deploy portainer stack from yaml file

```
docker stack deploy -c portainer.yml portainer
```

- portainer can be accessed now from the host machine at `http://192.168.57.11:9000`

### debug

- `docker node ls` : to see all the nodes in docker swarm.
- `docker stack ls` : to see the deployed stacks.
- `docker stack services my-flask-app` to see all the services in my-flask-app stack.
- `vagrant ssh node1 -- -L 8081:localhost:9000` port forward using ssh
