# K3s-on-Raspberry-x64

<h1>Ansible Playbooks for initial Raspberry Pi Setup</h1>
Simple Ansible playbooks, roles and tasks perform the initial setup, as well as a k3s setup for a new Raspberry Pi.


<h2>Assumptions and Dependencies</h2>
These playbooks assume a freshly installed Raspberry Pi running the current version of either Raspbian or Raspbian Lite.

Here you can find the Raspberry Pi 64-bit operating system - Bullseye
- https://qengineering.eu/install-raspberry-64-os.html

Use the Raspberry Pi Imager to load the Image onto the USB Card
- https://www.raspberrypi.com/software/
Also upload your private SSH key to the Pi. In the further course we will connect to the Pis via ssh. You can already set this up in the Raspberry Pi imager so you don't have to do this manually for each Pi. To do this, open the additional settings using Ctrl-Shift-X.

These playbooks also assume that you have Ansible installed and ready on your control machine.

For k3s you should also install kubectl. This will allow you to access their later cluster.
---

<h2>Additional customizations:</h2>

With my four Raspberry Pi 4B I use PoE HATs (Power Over Ethernet), which is why I also make a few adjustments for temperature management. I will go into this again in the relevant section. 

Since I like to connect a screen directly in debug cases (e.g. ssh connection not possible), there is also an Ansible role that makes hdmi changes in the config file. If this is not relevant, you can simply comment out the role. I'll go into this again in the related topic.

---

<h2>Info</h2>

If you want to learn more about Ansible read my tutorial:
[Understanding Ansible](documentation/ansible.md)

Or visit the official documentation
* https://docs.ansible.com/ansible/latest/user_guide/index.html#getting-started


<h2>Setup</h2>
When a Pi boots up for the first time, it receives a DHCP-assigned IP address, which is then changed to a static IP using Ansible Playbook.
You can use tools like wireshark to find out the ip address of your pis.

---
**<h3>**Adjust the values to your environment**</h3>**

* **inventory**

First, we have the file [inventory](inventory). 
The group "pi" contains the automatically assigned IPs from DHCP. They are needed to connect to the pis initially.  

The group "multi", which includes the groups "master" and "node", contains the static IP addresses. We need these to set up k3s later on. If you want the future static IP addresses to be the same as assigned by DHCP, you can delete the groups "pi". 

**Change the existing IPs (group pi) to the IP addresses assigned to your pis. Also, add the static IP addresses (group multi).**

!!!
**Please give the same name to the same Pi, no matter in which group. This name does not have to be the same as the host name. It is only used to identify the Pi. This is very important, otherwise the tasks can't check if the pis are reachable after the IP change.** Example: If, in the group "pi", no host can be found under the IP of "TinkyWinky", Ansible accesses the IP in another group. However, this can only be done if the names match.  

````shell
[pi]
TinkyWinky ansible_host=192.168.154.130
[master]
TinkyWinky ansible_host=192.168.154.80
````
---
* **vars**

Then there is the [vars.yml](role/ip-setup/vars/main.yml) file, which contains the mapping of hardware MAC addresses to specific IPs.
This is the "should" state of the Pi, which is to configure specific hostnames and IP addresses and use domain names identified by their MAC addresses. **Please enter here the MAC addresses of your Pis and define the desired static IP addresses.**

---
* **ks3 version**

Since you will probably run this tutorial at a later time, please check the k3s version you want to use. If a newer version is available, **adjust the variable in [all.yml](group_vars/all.yml)**.

---

**<h2>Playbook explanation</h2>**
In the folder "Playbooks" you will find the files [01.init_setup.yml](playbooks/01.init_setup.yml) and [02.k3s_setup.yml](playbooks/02.k3s_setup.yml). They contain lists of tasks that are automatically executed against hosts. 

**<h3>01.init_setup.yml</h3>**

First let's look at the file [01.init_setup.yml](playbooks/01.init_setup.yml). The variable "hosts" has the value "pi". This tells us that the subsequent tasts and roles will be executed for the hosts in the group "pi" ([inventory](inventory)). At the beginning the hosts are tried to be pinged. If they are reachable, an update of the pis is performed.

Under the variable "roles" you will find the role calls "poe-setup" and "display". As already mentioned in the section "Additional customizations" **you can comment them out or delete them if you don't need them.** 

**<h3>role ip-setup</h3>**

Ansible roles are a specific type of playbook. They are standalone and can be added to any playbook. You can find the tasks of the ip-setup role in [tasks](role/ip-setup/tasks/main.yml). 

**<h3>handler</h3>**

If you look more closely at the task of this file, you will see the call to handler in the "notify" section. Among other things, they reboot the Pis and check whether the switch to static IPs has worked. [handler](role/ip-setup/handlers/main.yml). If this "wait for host to return" step fails, please check if the hostnames in the [inventory](inventory) file match in all group. See my explanation in the "Matching the values to your environment/inventory" section.

After this step you can reach your Pis under the new static IP. 

**<h3>02.k3s_setup.yml</h3>**

The playbook [02.k3s_setup.yml](playbooks/02.k3s_setup.yml) sets up a k3s cluster. In preparation, the following three roles are executed on all hosts in the "multi" group.
    
- [prereq](role/k3s-setup/prereq/tasks/main.yml)
- [download](role/k3s-setup/download/tasks/main.yml)
- [raspberrypi](role/k3s-setup/raspberrypi/tasks/main.yml)

Then, the playbook divides into master and worker nodes as they require different configurations. 

*This part of the Ansible script is from https://github.com/k3s-io/k3s-ansible. However, the structure and unnecessary parts (e.g. Ubuntu configurations) have been removed.*



**<h2>Usage</h2>**

Pi setup
````shell
ansible-playbook playbooks/01.init_setup.yml -v
````
Afterwards, k3s can be rolled out. 
````shell
ansible-playbook playbooks/02.k3s_setup.yml -v
````
Get your kubeconfig from the master node
````shell
scp pi@master_ip:~/.kube/config ~/.kube/config
````
