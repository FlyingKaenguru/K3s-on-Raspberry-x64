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

There is another module, you can use to see all the information ansible can see about the server.

````shell
ansible -i inventory multi -m setup
````

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
* Contains multiple Ad-hoc commands
* simple yaml files
* run with "ansible-playbook"
* documented setup -> infrastructure as code

````shell
 ansible-playbook playbooks/ip_setup.yml --syntax-check
````

````shell
ansible-playbook -u pi playbooks/ip_setup.yml -v
````

<h2>Role</h2>
* Encapsulate configurations in reusable chunks
* Ansible Galaxy provides over 4000 roles#
* To create a new role you can use:

````shell
 ansible-galaxy init [role-name]
````
It creates a [rome-name]-directory. In this folder, you will find some default configurations. 
* In the default directory you want to put most of your variables. Variable that you might override in your playbooks.
* The VARs directory is good for variables that your role needs. But there might not want to be overridden by other playbooks.

To make sure the role gets executed correctly, check the meta file and make sure you have all the needed dependencies, that are used in your role, added.
Also call your role in your playbook.
* All you have to do is add a new section to the playbook: roles 
* Give it a list of roles

````shell
    roles:
      - [role-name]
````