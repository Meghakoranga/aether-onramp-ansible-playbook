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
Once ready, clone the Aether OnRamp repo on this target deployment server.

```
$ git clone --recursive https://github.com/opennetworkinglab/aether-onramp.git
$ cd aether-onramp
```
Making changes to hosts.ini file
```
[all]
node1 ansible_host=192.168.4.140 ansible_user=ubuntu ansible_password=ubuntu ansible_sudo_pass=ubuntu

[master_nodes]
node1

[worker_nodes]

[gnbsim_nodes]
node1
```
Now making changes to main.yml which is in vars directory 
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
