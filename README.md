# Aether-Multi-node-cluster
MULTI-NODE-CLUSTER FOR 5G NFV USING AETHER

Haswell CPU (or newer), with at least 4 CPU cores and 16GB RAM.
    • Clean install of Ubuntu 20.04 or 22.04 Server ISO(or later) kernel.
For example, something like an Intel NUC is likely more than enough to get started.

**Pre-requisite:**

     sudo apt update
     sudo apt upgrade 
     sudo apt install net-tools 
     sudo apt install iptables-persistent 
     sudo systemctl status ufw.service 
     sudo systemctl stop ufw.service 
     sudo systemctl disable ufw.service 
     sudo systemctl status ufw.service 
     systemctl status systemd-networkd.service

**Removing python3 and Installing python 3.8 v-env:**

     sudo apt autoremove python3.10
     python3 --version
     sudo apt install pipx
     sudo apt update
     sudo apt install ubuntu-advantage-tools
     sudo add-apt-repository ppa:deadsnakes/ppa
     sudo apt install software-properties-common
     sudo apt update
     sudo add-apt-repository ppa:deadsnakes/ppa
     sudo apt install python3.8-venv

Note that Aether assumes Ubuntu Server (as opposed to Ubuntu Desktop), the main implication being that it uses systemd-networkd rather than Network Manager to manage network settings. It is possible to work around this requirement, but be aware that doing so may impact the Ansible playbook for installing SD-Core.
OnRamp depends on Ansible, which you can install on your server as follows:

     sudo apt install pipx
     sudo apt install python3.8-venv
     pipx install --include-deps ansible
     pipx ensurepath
     sudo apt-get install sshpass
     sudo apt install ansible

Once installed, displaying the Ansible version number should result in output similar to the following:

  ansible --version
  ansible-galaxy collection install kubernetes.core

aether@aether:~$ ansible --version

ansible 2.10.8
  config file = None
  configured module search path = ['/home/aether/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.10.12 (main, Nov 20 2023, 15:14:05) [GCC 11.4.0]

Note that a fresh install of Ubuntu may be missing other packages that you need (e.g., git, curl, make), but you will be prompted to install them as you step through the Quick Start sequence.

**Download Aether OnRamp**

Once ready, clone the Aether OnRamp repo on this target deployment server:
  git clone --recursive https://github.com/opennetworkinglab/aether-onramp.git
  cd aether-onramp

**Set Target Parameters**

The Quick Start deployment described in this section requires that you modify two sets of parameters to reflect the specifics of your target deployment.
The first set is in file hosts.ini, where you will need to give the IP address and login credentials for the server you are working on. At this stage, we 
assume the server you downloaded OnRamp onto is the same server you will be installing Aether on.

**node1  ansible_host=192.168.1.200 ansible_user=aether ansible_password=aether ansible_sudo_pass=aether**

In this example, address 10.76.28.113 and the three occurrences of the stringaetherneed to be replaced with the appropriate values. Note that if you set up your server to use SSH keys instead of passwords, then ansible_password=aether needs to be replaced with ansible_ssh_private_key_file=~/.ssh/id_rsa(or wherever your private key can be found).
The second set of parameters is in vars/main.yml, where the two lines currently reading

**data_iface: enp2s0**

need to be edited to replace ens18 with the device interface for your server, and the line specifying the IP address of the Core’s AMF needs to be edited to reflect your server’s IP address:
**amf:**
   **ip: "192.168.1.200"**
   
**Scale Cluster**
Two aspects of our deployment scale independently. One is Aether proper: a Kubernetes cluster running the set of microservices that implement SD-Core and AMP (and optionally, other edge apps). The second is gNBsim: the emulated RAN that generates traffic directed at the Aether cluster. Minimally, two servers are required—one for the Aether cluster and one for gNBsim—with each able to scale independently. For example, having four servers would support a 3-node Aether cluster and a 1-node workload generator. This example configuration corresponds to the following hosts.inifile:

[all]
node1 ansible_host=172.16.144.50 ansible_user=aether ansible_password=aether ansible_sudo_pass=aether
node2 ansible_host=172.16.144.71 ansible_user=aether ansible_password=aether ansible_sudo_pass=aether
node3 ansible_host=172.16.144.18 ansible_user=aether ansible_password=aether ansible_sudo_pass=aether
node4 ansible_host=172.16.144.93 ansible_user=aether ansible_password=aether ansible_sudo_pass=aether

[master_nodes]
node1

[worker_nodes]
node2
node3
node4

[gnbsim_nodes]
node4

**Add nodes as you want in the host.ini file** 

[all]
node1 ansible_host=192.168.1.200 ansible_user=aether ansible_password=aether ansible_sudo_pass=aether
node2 ansible_host=192.168.1.220 ansible_user=worker ansible_password=worker ansible_sudo_pass=worker
#node3 ansible_host=10.76.28.117 ansible_user=aether ansible_password=aether ansible_sudo_pass=aether
#node4 ansible_host=10.76.28.119 ansible_user=aether ansible_password=aether ansible_sudo_pass=aether

[master_nodes]
node1

[worker_nodes]
node2
#node3
#node4

[gnbsim_nodes]
node1
#node4

The first block identifies all the nodes; the second block designates which node runs the Ansible client and the Kubernetes control plane (this is the node you SSH into and invoke Make targets and Kubectl commands); the third block designates the worker nodes being managed by the Ansible client; and the last block indicates which nodes run the gNBsim workload generator (gNBsim scales across multiple Docker containers, but these containers are not managed by Kubernetes). Note that having master_nodes and gnbsim_nodes contain exactly one common server (as we did previously) is what triggers Ansible to instantiate the Quick Start configuration.

You need to modify hosts.ini to match your target deployment. Once you’ve done that (and assuming you deleted your earlier Quick Start configuration), you can re-execute the same set of targets you ran before:

**To Install K8S Cluster:**
  make aether-k8s-install

**To Install Free5gc on K8S Cluster:**
   make aether-5gc-install

**To Install AMP Portal  (Dashboard) on K8S Cluster:**
   make aether-amp-install

