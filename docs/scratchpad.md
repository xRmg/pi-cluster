# Notes

Prepare hardware
* create sd card with raspbian
* create ssh file in boot directory
* if applicable create a wpa_supplicant file

First steps - provisioning with ansible.

Can you make a playbook that has 'commandline parameters'?

* change hostname
* Install keys
* disable password auth SSH
* Reboot


Connect with new hostname & keys

* update & upgrade 
* partition usb drive
* format usb drive
* mount & add to fstab
* install docker, kubernetes, glusterfs
* configure glusterfs


Ready to put in cluster. 
 * config kubernetes
 * 


Ansible hosts file for unprovisioned

[unprovisioned]
raspberrypi

[unprovisioned:vars]
ansible_ssh_user=pi

