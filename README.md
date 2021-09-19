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
