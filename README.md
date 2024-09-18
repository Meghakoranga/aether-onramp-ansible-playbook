# Aether-ONRAMP Quick Start Guide

##  Introduction

### What is Aether?
Aether is an open-source platform designed by the Open Networking Foundation (ONF) to support cloud-native 5G and edge computing services. It provides a complete 5G Core and Radio Access Network (RAN) solution for private cellular networks. Aether enables enterprises to deploy and manage private 5G networks with ease.

### What is ONRAMP?
ONRAMP (ONF RAN Management Platform) is a tool integrated with Aether for managing Radio Access Networks (RAN). It provides features to manage, configure, and monitor radio resources in real-time, and it helps to optimize network performance. ONRAMP plays a crucial role in automating the deployment and management of 5G RAN in the Aether ecosystem.

### Prerequisites
---
Before we begin the installation, we make sure we have the following:

<b>Hardware:</b>
<ul style="circle">
<li>Linux-based host machine (Ubuntu/CentOS recommended).</li>
<li>At least 16 GB of RAM, 4+ CPU cores, and 50 GB of disk space.</li>
<li>Internet connection.</li>
</ul>
<b>Software:</b>
<ul style="circle">
<li>Docker : Aether uses containerized services, so Docker needs to be installed.</li> 
<li>Kubernetes: Aether's microservices are deployed on Kubernetes. You can use a local Kubernetes cluster (like Minikube or Kind).</li>
<li>  Git: To clone repositories and manage version control.</li>
<li>  Helm: Kubernetes package manager to deploy Aether components.</li>
Here is the formatted content for your Google document:

---

## Prep Environment

To install Aether OnRamp, you must ensure that you can run `sudo` without a password, and there should be no firewall running on the server. You can verify this by executing the following command, which should report `Status: inactive`:

```bash
$ sudo ufw status
Status: inactive
```

Your server should also use **systemd-networkd** to configure the network. You can verify this by the following command:

```bash
$ systemctl status systemd-networkd.service
```

Note that Aether assumes **Ubuntu Server** (as opposed to Ubuntu Desktop), with the key difference being that Ubuntu Server uses **systemd-networkd** rather than **Network Manager** to manage network settings. Although it is possible to work around this requirement, doing so may impact the Ansible playbook used for installing SD-Core.

### Installing Ansible

OnRamp depends on **Ansible**, which you can install on your server using the following commands:

```bash
$ sudo apt install sshpass python3-venv pipx make git
$ pipx install --include-deps ansible
$ pipx ensurepath
$ source ~/.bashrc
```

Once installed, you can verify the Ansible version by running command 
``` ansible --version```.<br> On Ubuntu 20.04, you should see output similar to the example below. On Ubuntu 22.04, it will show `ansible [core 2.16.4]`.

```bash
ansible [core 2.13.13]
  config file = None
  configured module search path = ['/home/ubuntu/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /home/ubuntu/.local/pipx/venvs/ansible/lib/python3.8/site-packages/ansible
  ansible collection location = /home/ubuntu/.ansible/collections:/usr/share/ansible/collections
  executable location = /home/ubuntu/.local/bin/ansible
  python version = 3.8.10 (default, Nov 22 2023, 10:22:35) [GCC 9.4.0]
  jinja version = 3.1.3
  libyaml = True
```

---
## Download Aether OnRamp
Once ready, clone the Aether OnRamp repo on this target deployment server.
```
$ git clone --recursive https://github.com/opennetworkinglab/aether-onramp.git
$ cd aether-onramp
```
Important aether-onramp Directory Structure :
<ul>
<li>deps/: Contains Ansible deployment specs for Aether subsystems (e.g., deps/5gc). You can run Make targets for each subsystem to execute playbooks.</li>
<li>Main Makefile: Controls all subsystems and allows full Aether management from the root directory.</li>
<li>  vars/main.yml: Contains Ansible variables for deployment. The default is set for Quick Start; modify it for custom configurations.</li>
<li>hosts.ini: Ansible’s inventory file specifying the target servers. It defaults to a single-node setup but can be modified for multi-node clusters.</li>
</ul>
<br>

Modify Target Parameters:</br>
1. Edit hosts.ini with your server’s IP and credentials:

```
 node1 ansible_host=YOUR_SERVER_IP ansible_user=USER ansible_password=PASSWORD
```

If using SSH keys, modify it to:
```
node1 ansible_host=YOUR_SERVER_IP ansible_user=USER ansible_ssh_private_key_file=~/.ssh/id_rsa
```
2.Edit vars/main.yml to match your network interface and AMF IP:
```
data_iface: YOUR_INTERFACE
amf:
  ip: "YOUR_SERVER_IP"
```
Get your server’s IP and interface:
```
$ ip a
```

Verify Connectivity: Run the following to check if Ansible can connect to your nodes:
```
$ make aether-pingall
```
### Install Kubernetes: Set up an RKE2 Kubernetes cluster:
```
$ make aether-k8s-install
```
Use the following commands to verify installation:
View running namespaces:```
kubectl get pods --all-namespaces```

Check Helm repos: 
```
helm repo list
```
View deployed charts:
```
helm list --namespace kube-system
```
Install SD-Core: Deploy the 5G SD-Core:
```
$ make aether-5gc-install
```
This command installs Aether's 5G SD-Core. It runs the corresponding playbook, which deploys all necessary components (such as AMF, UPF, and other core services) required for 5G functionality.
Verify the pods in the omec namespace:
```
$ kubectl get pods -n omec
```
Run Emulated RAN Test: Deploy and run gNBsim to emulate traffic:
```
$ make aether-gnbsim-install
```
This command installs the gNBsim emulator, which is a tool that mimics the behavior of a 5G Radio Access Network (RAN). It is used to generate simulated user traffic for testing the SD-Core.
```
$ make aether-gnbsim-run
```
This command runs the gNBsim emulator, sending simulated traffic to the 5G Core. It helps verify that the Core can handle user registrations, sessions, and data transmission.
Check test results:
```
$ docker exec -it gnbsim-1 cat summary.log
```
Now  to tear down the entire Aether setup when you're done testing or want to start fresh use command
```
$ make aether-uninstall
```
