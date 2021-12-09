# K3s-on-Raspberry-x64

<h1>Ansible Playbooks for initial Raspberry Pi Setup</h1>
Simple Ansible playbooks, roles and tasks to lock down and perform initial setup for a new Raspberry Pi.

<h2>Assumptions and Dependencies</h2>
These playbooks assume a freshly installed Raspberry Pi running the current version of either Raspbian or Raspbian Lite.

Here you can find the Raspberry Pi 64-bit operating system - Bullseye
- https://qengineering.eu/install-raspberry-64-os.html

Use the Raspberry Pi Imager to load the Image onto the USB Card
- https://www.raspberrypi.com/software/

These playbooks also assume that you have Ansible installed and ready on your control machine.

With my four Raspberry Pi 4B I use PoE HATs (Power Over Ethernet), which is why I also make a few adjustments for temperature management. I will go into this again in the relevant section. 

<h2>Inventory</h2>
When a Pi boots up for the first time, it receives a DHCP-assigned IP address, which is then changed to a static IP using Ansible Playbook.

---

<h1>Understanding Ansible</h1>

<h2>Ansible ad hoc command</h2>

Ad hoc commands are great for tasks you repeat rarely.
It uses the command-line tool to automate a single task on one or more managed nodes.
A quick one-liner in Ansible without writing a playbook. An ad hoc command looks like this:

````text
$ ansible [pattern] -m [module] -a "[module options]"
````

* "-m" tells Ansible what module to use
* "-a" gives arguments for the module

If you do not specify a module, "-m command" is used as default

````shell
 ansible pi -a "free -h" -u pi
````

<h2>Inventory</h2>
_The inventory file describes which nodes can be accessed via Ansible.
The inventory contains the IP addresses or names of the available hosts.
The entries in the inventory can be grouped if necessary to facilitate management.
Both individual entries and groups can be linked to variables, for example, to assign a specific NTP server to all hosts in group X.
In addition, it is possible to obtain the inventory dynamically from other sources as well as to use multiple inventory files in parallel._

**Ping Client using Ansible**

Create an inventory file and define an inventory group of servers - in our example we name the group 'pi'.
List all the servers beneath the group. You can do this with an ip-address or hostname.

```
[pi]
192.168.154.130
```
Ping "all" server from the inventory group 'pi' and use the user 'pi'
```
ansible -i inventory pi -m ping -u pi
```
_"-i inventory" can be omitted in the command if this was defined in the ansible.cfg file_

If you want to sort your server into several groups, but also work with all of them at the same time. Then you can define a multigroup that contains groups.
````shell
[master]
Tinky-Winky ansible_host=192.168.154.130

[worker]
Dipsy ansible_host=192.168.154.129
Laa-Laa ansible_host=192.168.154.131
Po ansible_host=192.168.154.132

# Group that has groups
[multi:children]
master
worker
````

As in the example above, you can name your server to get better debugging response in our Ansible logs

If you want to use an ssh key instead of an insecure password to connect to the server, 
you can store the user and the path to the private key file as a variable.

````shell
[multi:vars]
ansible_ssh_user=pi
ansible_ssh_private_key_file=~/.ssh/id_ansible
````

###### _Hint_

If the ssh access is done with an encrypted ssh key

````text
In ansible There is no option to store passphrase protected private key.

For this we need to add the passphrase protected private key in the ssh-agent

Start the ssh-agent in the background.

# eval "$(ssh-agent -s)"
Add the SSH private key to the ssh-agent

# ssh-add ~/.ssh/id_rsa
Now try to run ansible-playbook and ssh on the hosts.
````

Alternative you can use "--ask-pass" or "-k"
You will be asked for the passphrase only once. Timeouts no longer occur
You also need "sshpass" installed. However, I could not get it to run under Windows Subsystem (Ubuntu 20.04)
The same error appeared as described in https://mikethecanuck.wordpress.com/tag/ansible/
````shell
ansible pi -m ping --user pi --ask-pass
````


<h2>Playbooks</h2>

````shell
 ansible-playbook playbooks/ip_setup.yml --syntax-check
````

````shell
ansible-playbook -u pi playbooks/ip_setup.yml -v
````