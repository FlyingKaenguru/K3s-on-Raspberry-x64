# Ansible Playbooks for initial Raspberry Pi Setup
Simple Ansible playbooks, roles and tasks perform the initial setup, as well as a k3s setup for a new Raspberry Pi.


## Assumptions and Dependencies
These playbooks assume a freshly installed Raspberry Pi running the current version of either Raspbian or Raspbian Lite.

Here you can find the Raspberry Pi 64-bit operating system - Bullseye
- https://qengineering.eu/install-raspberry-64-os.html

Use the Raspberry Pi Imager to load the Image onto the USB Card
- https://www.raspberrypi.com/software/
>**ATTENTION**: Also upload your private SSH key to the Pi. In the further course we will connect to the Pis via ssh. You can already set this up in the Raspberry Pi imager so you don't have to do this manually for each Pi. To do this, open the additional settings using Ctrl-Shift-X.

These playbooks also assume that you have Ansible installed and ready on your control machine.

For k3s (https://k3s.io/) you should also install kubectl. This will allow you to access their later cluster.

## Additional customizations

With my four Raspberry Pi 4B I use PoE HATs (Power Over Ethernet), which is why I also make a few adjustments for temperature management. I will go into this again in the relevant section. 

Since I like to connect a screen directly in debug cases (e.g. ssh connection not possible), there is also an Ansible role that makes hdmi changes in the config file. If this is not relevant, you can simply comment out the role. I'll go into this again in the related topic.

## Info

If you want to learn more about Ansible read my tutorial:
[Understanding Ansible](documentation/ansible.md)

Or visit the official documentation
* https://docs.ansible.com/ansible/latest/user_guide/index.html#getting-started


## Setup
When a Pi boots up for the first time, it receives a DHCP-assigned IP address, which is then changed to a static IP using Ansible Playbook.
You can use tools like wireshark to find out the ip address of your pis.

### Adjust the values to your environment

#### Inventory

First, we have the file [inventory](inventory). 
The group "pi" contains the automatically assigned IPs from DHCP. They are needed to connect to the pis initially.  

The group "multi", which includes the groups "master" and "node", contains the static IP addresses. We need these to set up k3s later on. If you want the future static IP addresses to be the same as assigned by DHCP, you can delete the groups "pi". 

>**ATTENTION**: Change the existing IPs (group pi) to the IP addresses assigned to your pis. Also, add the static IP addresses (group multi).

>**ATTENTION**: Give the same name to the same Pi, no matter in which group. This name does not have to be the same as the Pi's host name. It is only used to identify the Pi. This is very important, otherwise the tasks can't check if the Pis are reachable after an IP change.

Example: If no host can be found (in group 'pi') with the IP of 'TinkyWinky', Ansible accesses the IP in another group. However, this can only be done if the names match.  

``` ini
[pi]
TinkyWinky ansible_host=192.168.154.130
[master]
TinkyWinky ansible_host=192.168.154.80
```

#### Vars
Then there is the [vars.yml](role/ip-setup/vars/main.yml) file, which contains the mapping of hardware MAC addresses to specific IPs.
This is the "should" state of the Pi, which is to configure specific hostnames and IP addresses and use domain names identified by their MAC addresses.

>**ATTENTION**: Insert the MAC addresses of your Pis and define the static IP addresses, dns nameserver and the ip gateway. Also, define the final names of your Pis.

#### ks3 version
Since you will probably run this tutorial at a later time, please check the k3s version you want to use.
>**ATTENTION**: If a newer version is available, adjust the variable in [all.yml](group_vars/all.yml).


## Playbook explanation
You will find the files [01.init_setup.yml](playbooks/01.init_setup.yml) and [02.k3s_setup.yml](playbooks/02.k3s_setup.yml) in the folder 'Playbooks'. They contain lists of tasks that are automatically executed against hosts. 

### 01.init_setup.yml
Let's first take a look at the file [01.init_setup.yml](playbooks/01.init_setup.yml). The variable 'hosts' has the value 'pi'. This tells us that subsequent tasks and roles will be executed for the hosts in the group 'pi' ([inventory](inventory)). At the beginning hosts are tried to be pinged. If they are reachable, an update will take place.

Under the variable 'roles' you will find the role calls "poe-setup" and 'display'. As already mentioned in the section [Additional customizations](#additional-customizations) **you can comment them out or delete them if you don't need them.** 

### role ip-setup
Ansible roles are a specific type of playbook. They are standalone and can be added to any playbook. You can find the tasks of the ip-setup role in [main.yml](role/ip-setup/tasks/main.yml). 

### handler
If you look more closely at the tasks of [main.yml of the tasks directory](role/ip-setup/tasks/main.yml), you will see the call to all handlers in the [main.yml of the handlers directory](role/ip-setup/handlers/main.yml) within the "notify" section. Among other things, they reboot the Pis and check whether the switch to static IPs has worked. If this "wait for host to return" step fails, please check if the Pi-names in the [inventory](inventory) file match in all groups. See my explanation in the ["Matching the values to your inventory"](#inventory) section.

After this step you can reach your Pis under the new static IP. 

### 02.k3s_setup.yml

The playbook [02.k3s_setup.yml](playbooks/02.k3s_setup.yml) sets up a k3s cluster. In preparation, the following three roles are executed on all hosts in the "multi" group.
    
- [prereq](role/k3s-setup/prereq/tasks/main.yml)
- [download](role/k3s-setup/download/tasks/main.yml)
- [raspberrypi](role/k3s-setup/raspberrypi/tasks/main.yml)

Then, the playbook divides into master and worker nodes as they require different configurations. 

*This part of the Ansible script is from https://github.com/k3s-io/k3s-ansible. However, the structure and unnecessary parts (e.g. Ubuntu configurations) have been removed.*



## Usage

Pi setup
``` 
ansible-playbook playbooks/01.init_setup.yml -v
```

Afterwards, k3s can be rolled out. 
```
ansible-playbook playbooks/02.k3s_setup.yml -v
```

Get your kubeconfig from the master node
```
scp pi@master_ip:~/.kube/config ~/.kube/config
```

## Definition of Terms: What is...?

### ansible
Red Hat Ansible Automation Platform is an IT automation technology
- https://www.ansible.com/

### kubectl
Command Line Tool to control your Kubernetes cluster.
- https://kubernetes.io/docs/reference/kubectl/overview/

### kubernetes
Production-Grade Container Orchestration
- https://kubernetes.io/

### k3s
Lightweight Kubernetes:
- https://k3s.io/
