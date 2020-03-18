# Notes

Prepare hardware
* create sd card with raspbian
* create ssh file in boot directory
* if applicable create a wpa_supplicant file

First steps - provisioning with ansible.

ansible-play

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
ansible_ssh_pass=raspberry

run the provisioning playbook




ansible cluster -m shell -a 'printf $(sudo blkid -o export /dev/sda1|grep UUID)" /data/glusterfs/cluster_volume/brick1 xfs defaults,noatime 1 2\n" | sudo tee -a /etc/fstab'

ansible cluster -m shell -a 'sudo mount -a' 

ansible cluster -m shell -a 'sudo gluster volume create cluster_volume replica 4 tethys:/data/glusterfs/cluster_volume/brick1/brick titan:/data/glusterfs/cluster_volume/brick1/brick ganymede:/data/glusterfs/cluster_volume/brick1/brick'

sudo gluster volume create cluster_volume replica 4 tethys:/data/glusterfs/cluster_volume/brick1/brick titan:/data/glusterfs/cluster_volume/brick1/brick ganymede:/data/glusterfs/cluster_volume/brick1/brick

sudo gluster volume create cluster_volume replica 3 tethys:/data/glusterfs/cluster_volume/brick1/brick titan:/data/glusterfs/cluster_volume/brick1/brick ganymede:/data/glusterfs/cluster_volume/brick1/brick

mkdir /mnt/gluster_data
  
   
ansible cluster -m shell -a 'sudo mkdir /mnt/gluster_data && sudo mount -t glusterfs localhost:/cluster_volume /mnt/gluster_data/'
   
ansible cluster -m shell -a 'echo "localhost:/cluster_volume /mnt/gluster_data/ nfs defaults,_netdev,vers=3 0 0" | sudo tee -a /etc/fstab'