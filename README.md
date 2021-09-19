# Devops-web-1

A simple web app, but with all the devops bells and whistles (Vagrant, Ansible, Docker, Docker Swarm)

## Setup

We will provision using Vagrant 1 control vm and 3 nodes for deployment from VirtualBox.
(the `control vm` will connect to the `nodes` using Ansible to apply the correct configuration)

**Note:** The ip addresses as described in the Vagrantfile should belong to one of the subnets in the VirtualBox Host Network Manager ( `File` -> `Host Network Manager` )
