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

If you want to learn more about Ansible read:
[Understanding Ansible](documentation/ansible.md)

Or visit the official documentation
* https://docs.ansible.com/ansible/latest/user_guide/index.html#getting-started

<h2>Setup</h2>
When a Pi boots up for the first time, it receives a DHCP-assigned IP address, which is then changed to a static IP using Ansible Playbook.

First, we have the [inventory](inventory)
file defining the current IP addresses of the Pis:

Note that we’ve also got the definition of the default user pi in there.

Then there is the [vars.yml](role/vars/main.yml) file, which contains the mapping of hardware MAC addresses to specific IPs.
This is the "should" state of the Pi, which is to configure specific hostnames and IP addresses and use domain names identified by their MAC addresses. More precisely, we switch from dynamic IPs to static IPs.

