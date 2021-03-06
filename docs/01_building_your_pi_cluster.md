# Building your Raspberry Pi Cluster

I'm going to explore and document the setup of my raspberry pi cluster. This guide is not a beginner RPi guide and does expect some linux knowledge.

The topics in this howto will be how to provision and manage your cluster with Ansible and to set up docker swarm.

But first a list of ingredients.

## Ingredients
* 4x Raspberry pi 4 (4gb)
* 4x Verbatim SDCard (16GB)
* 1x Anker Powerport (12A)
* 1x TL-SG105 - 5 port gigabit switch.

A linux box to configure this all from.

## Preparing your Raspberry Pi's
![Pi banner](images/Raspberry_Pi_Banner.jpg)
First thing to do is flash the SDCards with the [lastest](https://www.raspberrypi.org/downloads/raspbian/) Raspbian distribution. Follow the instructions on the Raspberry Pi website and flash with BalenaEtcher, do not forget to create a ssh file in your boot partition to [enable SSH](https://www.raspberrypi.org/documentation/remote-access/ssh/).

After flashing the SDCards and creating the ssh file in the boot partition, hook the pi's up to the ethernet switch. (Do not power them on yet.)

## Introducing Ansible.

![Ansible banner](images/ansible.png)

[Ansible](https://www.ansible.com/) is an open-source IT automation engine, with Ansible you can automatically execute tasks on nodes over SSH. We currently have 4 raspberry pi's that are in their default state, the hostname for all of them is raspberrypi, the login is raspberry, ssh password authentication is enabled. All stuff we would like to change. We could do this manually, login over ssh, do the changes and login in the next, but this is tedious and fault sensitive. 

Ansible can help you with this, by creating a playbook that does the changes for you and running this playbook against your raspberry pi's.

First step is to [install Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).

After installing create your Ansible host file, edit `/etc/ansible/hosts` with your favourite editor and add the folowing text. In this example your host group will be called 'cluster' and your hosts will be called raspberrypi_1 to raspberrypi_4, if you want to change your cluster name and hostnames do that now.

#### Ansible Host File

```
[pi_cluster]
raspberrypi_1
raspberrypi_2
raspberrypi_3
raspberrypi_4

[pi_cluster:vars]
ansible_ssh_user=pi

[unprovisioned]
raspberrypi

[unprovisioned:vars]
ansible_ssh_user=pi
ansible_ssh_pass=raspberry
ansible_ssh_args='-o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no'
```

Unprovisioned is the host group for the fresh raspberry pi's, we will have multiple 
In the next step we are going to create the playbook.

#### Ansible Playbook ( Provision the Pi )

Take a look at the file `playbooks/01_setup_fresh_pi.yaml` this file takes one argument (newhostname), it will run on unprovisioned hosts (`- hosts: unprovisioned`) and will do the following steps:

* Deploy the SSH key ansible-pi.pub from `../keys/ansible-pi.pub` (relative to the playbook folder)
* Change the hostname.
* Also change the hostname in the `/etc/hosts` file on the target.
* Disable Password Authenication for the SSH daemon.
* Reboot the raspberry pi.

After this play book you will have a raspberry pi that has a new hostname, a public key for logging in, and disabled password authentication.

So First step, create the ansible-pi private and public keys.

Run `ssh-keygen -b 4096` and store the public key in the keys folder and add the private key to your `~/.ssh/` folder and `~/.ssh/config`. You can use a passphrase protected key but do not forget to load your key in the ssh agent or Ansible can't connect to your hosts

Put power on your first raspberry pi and wait untill you can until it is up, you can check with `ping raspberrypi` 

When the keys are created and the Rapsberry pi is reachable you are ready to run your first playbook.

run `ansible-playbook playbooks/01_setup_fresh_pi.yml -e newhostname=raspberrypi_1` and check for errors. If everything went ok the raspberry pi is now rebooting and will soon be reachable with the new hostname.

After a minute or so try a ping pong command with Ansible, if all went well you should get the following reply

```
$ ansible pi_cluster -m ping --limit raspberrypi_1
raspberrypi_1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

You are now ready to provision the rest of the Raspberry Pi's, hook up the power to the next Raspberry Pi and run the command with the next hostname, repeat these steps for all 4 pi's

`ansible-playbook playbooks/01_setup_fresh_pi.yml -e newhostname=raspberrypi_2`

`ansible-playbook playbooks/01_setup_fresh_pi.yml -e newhostname=raspberrypi_3`

`ansible-playbook playbooks/01_setup_fresh_pi.yml -e newhostname=raspberrypi_4`


<sub> If you run into issues with SSH host fingerprints, remove the offending ones with `ssh-keygen -f ~/.ssh/known_hosts -R raspberrypi` and add them manually with `ssh-keyscan -H raspberrypi >> ~/.ssh/known_hosts`</sub>

if you are done you can ping your whole cluster, the reply should look like

```
$ ansible pi_cluster -m ping
raspberrypi_1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
raspberrypi_2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
raspberrypi_3 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
raspberrypi_4 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```
![Pi cluster](images/pi-cluster.jpg)


After your whole cluster is reachable run the second playbook like:
 
 run `ansible-playbook playbooks/02_setup_roles.yaml` this will install docker on the cluster.
 
 
 When this is finished without errors it is time to create the Docker swarm and join it.
 
 On the host, initialize the docker swarm by running `docker swarm init` and you will get a docker swarm token. 
 
 execute the docker join command on all the host with ansible  `ansible cluster -m shell -a 'docker swarm join --token SWMTKN-1-2rok9rdutcakeenetyrx9d0eqm8itbz8ajfhb8zi48rngsmfcs-4w7tnuga3my15hpjb4pree2bp 192.168.178.54:2377 --advertise-addr eth0'`
 
 And check the status of your swarm with 
 
 `docker node ls` to see all the nodes you have in your docker swarm 
 
 ```rmg@gen8:~$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
icz3g1o48fs2kp3e0t5ol40za     atlas               Ready               Active                                  19.03.6
x7uqct8j780jqnk3chip55sb4     ganymede            Ready               Active                                  19.03.6
z91sgwtlxtmh0ezk5v6emjxm2 *   gen8                Ready               Active              Leader              19.03.13
q98iuqbk4w4bfncvxyy2z27op     tethys              Ready               Active                                  19.03.6
zf4mckfqgep4l820mwvcgqzdr     titan               Ready               Active                                  19.03.6
```

Now you are ready to deploy stuff on your swarm! go read about it here:https://docs.docker.com/get-started/swarm-deploy/
