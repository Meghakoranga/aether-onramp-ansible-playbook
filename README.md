# Aether-onrmap setup for ubuntu
 ```
   $sudo ufw status
   ```
This should show Status: inactive

 ```
   $ systemctl status systemd-networkd.service
 ```
Installing ansible
```
$ sudo apt install sshpass python3-venv pipx make git
$ pipx install --include-deps ansible
$ pipx ensurepath
$ source ~/.bashrc
```
Once installed,  verify the Ansible version by typing the following. 
```
$ ansible --version
```
cloning the Aether OnRamp repo on this target deployment server.

```
$ git clone --recursive https://github.com/opennetworkinglab/aether-onramp.git
$ cd aether-onramp
```
aether-onramp data structure
```
aether-onramp/
├── Makefile
├── hosts.ini
├── vars/
│   └── main.yml
├── deps/
│   ├── 5gc/
│   │   ├── roles/
│   │   │   └── core/
│   │   │       ├── tasks/
│   │   │       │   ├── install.yml
│   │   │       │   └── other_task_files.yml
│   │   │       └── templates/
│   │   └── playbook.yml
│   ├── gnbsim/
│   └── other_subsystems/
```
deps directory: This contains the Ansible deployment specifications (i.e., the playbooks) for all of Aether’s subsystems.
<ul>
 <li>Each subdirectory corresponds to a specific component of Aether, such as deps/5gc for the 5G Core.</li>
<li>In each directory, there are Ansible playbooks that can be executed to install and configure that specific component. For example, the 5GC (5G Core) installation playbook is found at deps/5gc/roles/core/tasks/install.yml.</li>
</ul>

### Modifying Parameters for  Target Deployment
<ul>
 <li>We’ll modify hosts.ini to reflect your server’s IP and authentication method.</li>
<li>We’ll adjust vars/main.yml to ensure the correct network interface and IP addresses are set. This ensures Ansible configures the networking aspects correctly when deploying Aether</li>
</ul>
a.&nbsp;</space><b>hosts.ini </b><br>
Ansible's inventory file (hosts.ini) specifies which servers or hosts the playbooks will target. In this file, you will need to configure:
IP address of your server.
Login credentials (username and password or SSH private key) for Ansible to connect to the server.

```
[all]
node1 ansible_host=192.168.4.140 ansible_user=ubuntu ansible_password=ubuntu ansible_sudo_pass=ubuntu

[master_nodes]
node1

[worker_nodes]

[gnbsim_nodes]
node1
```
b. &nbsp;<b>vars/main.yml</b> (Ansible Variables)<br>
vars/main.yml is the primary Ansible variables file. Here, we define the specific configuration details for our environment, such as:
<ul>
 <li>Network interfaces: These need to match the interface name on your server (e.g., ens18).</li>
<li>IP addresses: Such as the IP address of the AMF (Access and Mobility Function).</li>
</ul>

```
core:
  data_iface: eth0
 amf:
    ip: "192.168.4.140"
 router:
    data_iface: eth0
```
server's ip can be found using command  $ ip a

Deploying aether with ansible
```
$ make aether-install
```
Verify the pods in the omec namespace:
```
$ kubectl get pods -n omec
```
Run Emulated RAN Test: Deploy and run gNBsim to emulate traffic:
```
$ make aether-gnbsim-install
$ make aether-gnbsim-run
```
Check test results:
```
$ docker exec -it gnbsim-1 cat summary.log
```
Clean Up: To uninstall all components, use:
```
$ make aether-uninstall
```
