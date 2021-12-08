# K3s-on-Raspberry-x64

<h1>Ansible Playbooks for Initial Raspberry Pi Setup</h1>
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
Test
---
Ping Client using Ansible

Create an inventory file and define an inventory group of servers - in our example its "pi".
List all the servers beneath the group. You can do this with an ip-address or hostname.
```
[pi]
192.168.154.130
```
Ping "all" server in the inventory list and use the user "pi"
```
ansible -i inventory pi -m ping -u pi
```
_"-i inventory" can be omitted in the command if this was defined in the inventory file_

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

---

"-m" tells Ansible what module to use
"-a" gives arguments for the module 
If you do not specify a module, "-m command" is used as default 

````shell
 ansible pi -a "free -h" -u pi
````
