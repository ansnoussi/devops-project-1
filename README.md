# Devops-web-1

A simple web app, but with all the devops bells and whistles (Vagrant, Ansible, Docker, Docker Swarm)

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
- ssh into any node without password : `ssh vagrant@node1`

6. install ansible in the control vm: `sudo apt install ansible`

7. check for connectivity with ansible : `ansible nodes -i myhosts -m command hostname` (`-i` : inventory, `-m` : module, `-a` : argument for the chosen module)
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
   `ansible nodes -i myhosts -m command -a 'sudo apt-get -y install python-simplejson'`

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
