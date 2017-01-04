# Create A Sample Cluster
These simple instructions cover setting up a 3 node cluster.  The only pre-requisites are a copy of MarkLogic for CentOS 8 or RHEL 7, and ML 8 for RHEL 7.  You need a laptop or desktop (ideally) with 16 gigs of RAM.  You should also have an i7 or better processor that can support 8 vCPUs.  You might become processor starved on a low end i7 with only 2 physical cores or i3, or i5 processors.  

## Limitations
This is meant to be a simple cluster you can run on your desktop or laptop to support experimenting with cluster configurations.  It is not meant to be a production configuration.  A production configuration should span multiple physical hosts.  Nor is it a good "test" configuration, which should mirror your production configuration.  Nor is it meant to handle large amounts of data.  

## Install Virtual Box
Install [https://www.virtualbox.org/wiki/Downloads](Virtual Box) for your cluster.  Virtual Box is a virtual machine mananger along the lines of VMWare or Xen.  It is freely available and runs on most common platforms.  

## Create a Virtual Machine
Create a virtual machine with the following settings:

1. 3072 Megabytes of RAM
2. 2 CPUs.
3. 20 Gigs of Disk
4. A NAT network
5. A Host-Only Network (there's usually a default of vboxnet0)

### Install Cent OS 7 on the virtual machine.
Follow the instructions to install Cent OS 7 on the virtual machine.  Other Linux distributions are also perfectly fine, but either RHEL 7 or Cent OS 7 matches the instructions below.

### Set Firewall Rules
By default Cent OS/RHEL uses a firewall.  If you want to connect your servers (and have your servers connect to each other), then you will need to open ports on your filewall.  The format is:

```
sudo firewall-cmd --permanent --zone=public --add-port=7999/tcp
sudo firewall-cmd --permanent --zone=public --add-port=7998/tcp
sudo firewall-cmd --permanent --zone=public --add-port=8000/tcp
sudo firewall-cmd --permanent --zone=public --add-port=8001/tcp
sudo firewall-cmd --permanent --zone=public --add-port=8002/tcp
sudo firewall-cmd --permanent --zone=public --add-port=8100/tcp
sudo firewall-cmd --reload
```
The last firewall rule would be specific to your application.  If you need to add additional firewall rules (i.e. your application uses ports 8010, 8011 and 8012, add those lines but don't forget to reload.

### Update the Hosts File
Since you're probably not going to add the names to your DNS, you'll need to include them in the virtual machine's host file.  Add the following lines to the /etc/hosts file on your virtual machine.

```
192.168.56.201 node-1.localdomain node-1
192.168.56.202 node-2.localdomain node-2
192.168.56.203 node-3.localdomain node-3
```

### Clone the Virtual Machine
Then clone the virtual machine by right clicking on the virtual machine and selecting "Clone..." from the drop down menu.  Make sure you check the box to re-initialize the Mac address for the virtual machine.  You should name your nodes Node 1, Node 2, and Node 3.

### Configure the Virtual Machines
Start each virtual machine.  Your Host Only network should have an IP address like 192.168.56.0.  Sometimes this might be 192.168.57.0, so check the settings for your network in the VMWare preferrences dialog.  In the instructions below I'll use 192.168.56.0 so either change your Virtual Box to match or adjust as appropriate.  Vitual Box normally assigns IP addresses using DHCP to the 100-199 range.  You will give all three nodes fixed IP addresses.  Start by using ifconfig -a to see what he devices are for one of the machines.  (Because the machines are clones the information should be the same across all 3).

On my machines the Host only adapater is enp0s8.  in /etc/sysconfig/network-scripts look for a file called ifcfg-enp0s8 (or whatever you adapter happens to be named) or create the file.  This will be the adapter that's initially attached to the host-only network or is unattached on boot up.  The other device will be attached to the NAT network on 10.2.x.y.  Add the following settings to the file:

```
DEVICE=enp0s8
BOOTPROTO=none
ONBOOT=yes
PREFIX=24
IPADDR=192.168.56.201
```
The only line that differs for the 3 machines is the IPADDR line, and we'll be using 192.168.56.201, 192.168.56.202, and 192.168.56.203 for the 3 machines. 

Edit the /etc/hostname file to include the hostname for each host.  192.168.56.201 should contain the line 'node-1.localdomain', 192.168.56.202 should contain 'node-2.localdomain' and 192.168.56.203 should contain 'node-3.localdomain'.  


## Edit Your Hosts File
This is a setting on your laptop's or desktop's operating system.  (The host machine).  On Mac OS or Linux the file is called /etc/hosts and on Windows it's located under c:\Windows\System32\Drivers\etc\hosts.  (Don't forget to run Notepad as administrator when editing the file.  Add the lines above so that typing "http://node-1.localdomain" in a browser would take you to your first cluster node.

## Install MarkLogic
Download MarkLogic 
