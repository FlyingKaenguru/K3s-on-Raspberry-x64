# Understanding Ansible

## Ansible ad hoc command

Ad hoc commands are great for tasks you repeat rarely.
It uses the command-line tool to automate a single task on one or more managed nodes.
A quick one-liner in Ansible without writing a playbook. An ad hoc command looks like this:

```
ansible [pattern] -m [module] -a "[module options]"
```

* "-m" tells Ansible what module to use
* "-a" gives arguments for the module

If you do not specify a module, "-m command" is used as default

```
 ansible pi -a "free -h" -u pi
```

## Inventory
The inventory file describes which nodes can be accessed via Ansible.
It contains the IP addresses or names of the available hosts.
Entries in the inventory can be grouped if necessary to facilitate management.
Both individual entries and groups can be linked to variables, for example, to assign a specific NTP Server to all hosts in group X.
In addition, it is possible to obtain the inventory dynamically from other sources as well as to use multiple inventory files in parallel.

### Ping Client using Ansible

Create an inventory file and define an inventory group of servers - in our example we name the group 'pi'.
List all the servers beneath the group. You can do this with an IP-Address or hostname.

``` ini
[pi]
192.168.154.130
```
Ping 'all' server from the inventory group 'i' and use the user 'pi'
```
ansible -i inventory pi -m ping -u pi
```
_The flag `-i inventory` can be omitted in the command if this was defined in the ansible.cfg file_

### Shutdown command

Since we are using Ansible, we can also use it to shut down all the servers in our inventory. The following command can be used for this purpose.
* `-a 'shutdown now'` - send the comand to all the servers now
* `-b ` - use the root user and use sudo

```
ansible all -i inventory -a "shutdown now" -b
```

### Groups
If you want to organize your server into several groups, but also work with all of them at the same time. Then you can define a multigroup that contains groups.
``` ini
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
```

As in the example above, you can give your servers names to get a better debugging output in the Ansible logs.

There is another module, you can use to get all the information ansible can see about the server.

```
ansible -i inventory multi -m setup
```
## SSH
If you want to use an ssh key instead of an insecure password to connect to the server,
you can store the user and the path to the private key file as a variable.

``` ini
[multi:vars]
ansible_ssh_user=pi
ansible_ssh_private_key_file=~/.ssh/id_ansible
```

>**Hint** 
>
>If the ssh access is done with an encrypted ssh key
>
>In ansible There is no option to store passphrase protected private key. For this we need to add the passphrase protected private key in the ssh-agent.
>
>Start the ssh-agent in the background: `eval "$(ssh-agent -s)"`
>
> Add the SSH private key to the ssh-agent: `ssh-add ~/.ssh/id_rsa`
>
>Alternatively you can use `--ask-pass` or `-k`
>
>You will be asked for the passphrase only once. Timeouts no longer occur
You also need "sshpass" installed. However, I could not get it to run under Windows Subsystem (Ubuntu 20.04)
The same error appeared as described in https://mikethecanuck.wordpress.com/tag/ansible/
>
>`ansible pi -m ping --user pi --ask-pass`
>

## Playbooks
* Contains multiple Ad-hoc commands
* simple yaml files
* run with "ansible-playbook"
* documented setup -> infrastructure as code

```
ansible-playbook playbooks/01.init_setup.yml --syntax-check
```

```
ansible-playbook playbooks/01.init_setup.yml -v
```

## Role
* Encapsulate configurations in reusable chunks
* Ansible Galaxy provides over 4000 roles
* To create a new role you can use:

```
 ansible-galaxy init [role-name]
```
It creates a [role-name]-directory. In this folder, you will find some default configurations. 
* In the default directory you want to put most of your variables. Variable that you might override in your playbooks.
* The VARs directory is good for variables that your role needs. But there might not want to be overridden by other playbooks.

To make sure the role gets executed correctly, check the meta file and make sure you have all the needed dependencies, that are used in your role, added.
Also call your role in your playbook.
* All you have to do is add a new section to the playbook: roles 
* Give it a list of roles

``` yml
roles:
  - [role-name]
```

## Handler
Handlers are just like normal tasks in an Ansible playbook.
But sometimes you want a task to run only when a change is made on a machine. 
For example, you may want to restart a service if a task updates the configuration of that service, but not if the configuration is unchanged. 
Ansible uses handlers to address this use case. 
Handlers are tasks that only run when notified.