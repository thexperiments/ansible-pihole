# Ansible playbook for Pi-hole on debian

A minimalistic playbook to set up Pi-hole on a debian system (lxc containers in my case) together with a keepalived high availability configuration.

You can also use this with a single machine if you disable keepalived in the ```group_vars``` but iis strongly recommended to set up such an important service with high availability.

## Prerequisites

Two (or more) debian based machines with working ssh login. 

(for now only tested with root in ubuntu 20.04 lxc containers but should work with others too)

## Setup

1. Install Ansible
    * For debian ```sudo apt-get install -y python3-pip``` and ```pip3 install ansible``` should do the job
2. Get the files which are in this repository if you don't already have them
    * ```git clone --depth 1 https://github.com/thexperiments/ansible-pihole.git```

## Configuration

1. Update all IP addresses for the host in the inventory located in host_vars
2. Update all more general settings for all hosts in the group in the file ```group_vars/piholes.yml```
    * change IP addresses
      * make sure keepalived_virtual_ip is a IP not used by anything else and not in DHCP range. This will be the address you later use the Pi-hole with.
    * change passwords from the examples to secure passwords
    * check all settings
3. Check the template for the Pi-hole settings ```templates/setupVars.conf.j2``` if there are settings of interest configured which I have not set with variables

## Rollout

Just run 

```ansible-playbook main.yml```

It should connect to the two machines configured earlier, install Pi-hole on both, set up the one called ```pihole``` as master in keepalived. You now have two instances reachable on their own IPs and the high available one via the virtual IP configured with ```keepalived_virtual_ip```.

## Remarks

### Syncing

I did explicitly choose to not setup any syncing between the instances for several reasons

* complexity
* robustness
* I don't do manual changes to my Pi-hole often

Instead I configured the blocklists and the local hostnames and exported a backup via ```Teleporter``` in the web UI and imported it on the second instance.

### Updates

The task for installing can be repeated without problems (currently, not officially supported by the installer) so running the task again will automatically update Pi-hole. Settings should stay except the ones defined in this playbook.

### High availability setup

The current keepalived setup is very rudimentary it will switch over if the connection to the other host breaks. It does not check if Pi-hole is still answering DNS requests. (Would be a nice addition, feel free to contribute)
