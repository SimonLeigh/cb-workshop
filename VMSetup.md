# VM Setup

The following instructions can be used in order to set up the workshop VM-s. 

We are using the following versions of Software for these preperations:

* VirtualBox >4.3: https://www.virtualbox.org/wiki/Downloads
* CentOS 6.7: http://mirror2.hs-esslingen.de/centos/6.7/isos/x86_64/CentOS-6.7-x86_64-LiveCD.iso
* Couchbase: http://packages.couchbase.com/releases/4.0.0/couchbase-server-enterprise-4.0.0-centos6.x86_64.rpm

## Couchbase Server Instance $i

### VM Installation

The place holder ${i} is the id of the instance. We need 3 VM-s for the workshop, so ${i} is from [1,2,3].

Please perform the following steps in order to provide a CentOS6 VM:

* Download the CentOS image
* Create a new VirtualBox VM with 
  * the name 'CentOS6-DCJW-Node${i}' 
  * with the type Linux/Red Hat (64 bit)
  * with 2048MB RAM
  * an empty HDD
  * with a VDMI disk format
  * and a dynamically allocated size of 20GB
* Change the VM settings
  * Network: The first network adapter uses 'NAT'
  * Storage: Choose the CentOS iso image as a CDROM drive
* Start the VM
* Install CentOS 6
  * Select 'Install text mode' in the boot menu
  * Choose the installation language as 'English'
  * Choose the keyboard layout
  * Select 'Re-initialize all'
  * Pick a time zone
  * Enter the root password 'couchbase' twice
  * Use the entire drive and write the changes to disk
  * Wait until the installation completed
  * Power off the machine after the installation
* Change VM the settings again 
  * Storage: Disabling the CDROM drive by removing the image
* Start the VM and wait until started
* Quit the initial setup wizard
* Log-in to the CLI as root

### Network config

The network configuration is a bit more complicated with Virtualbox. What we need is a VM which can reach the outside world and which can be reached from the outside world. In order to achieve this we will need to define in sum 2 virtual networks for our VM. So far we already have defined the NAT (Network Address Translation) network. NAT allows to access the outside workd from a VM. Imagine that your VM is connected to a service which acts like a router whereby the VM can reach the outside world but can NOT be reached from the outside world or from other VM-s. First let's check if yor CentOS6 instance has an IP for the NAT network:

```
/sbin/ifconfig
```

The IP is usually something like '10.0.2.15'. Each VM ${i} has then usually the same NAT IP. There are no conflicts here because the NAT IP is anyway not accessible from the outside world.

In order to enable access from the outside world via NAT, port forwarding can be used. So to simplify further configuration steps it makes sense to allow the access from the outside world to the VM via NAT and port forwarding. Under the network settings of the VM's NAT network define the following port forwardings:

| Name          | Host port           | Gest port |
| ------------- |---------------------|---------- |
| SSH           |9${i}22              | 22        |
| CB            |9${i}91              | 8091      |
| VNC           |9${i}59              | 5901      |

Let's enable some service in order to allow to connect to them from the outside:

* Enable 'sshd' by executing 
```
chkconfig sshd on
```
* Reboot to make sure that your settings got applied
* Run a VNCServer by using the password 'couchbase' and by executing
```
vncserver
```
* Disable the firewall temp. by executing
```
/etc/init.d/iptables stop
```

You should now be able to establish a secure shell connection from the host to the VM via Putty or the following command:

```
ssh root@localhost -p 9${i}22
```

The next step is to allow the machine to be connected by other machines in the network because in the exercises we want build a cluster of VM-s here. First we need to add a second network adapter:

* Power off the VM
* Under the VM settings add 'Adapter 2' and enable it
* Select 'Host-only Adapter' as the type
* If there is no global host-only network then you need to create it via the general Virtualbox preferences
* Power on the maching

Your VM has not 2 network cards. Given that you installed from the LiveCD by default NetworkManager is used. Check if NetworkManager is running by executing:

```
service NetworkManager status
```

We need to make sure that our VM uses a static IP address in the host-only network. Before we do this let's find out the current network settings:
  
* Did you get an IP address assigned? Check via 'ifconfig' and note it as $previous_ip!
* Get the current name sever by using 'cat /etc/resolv.conf' and note it as $previous_dns!
* Use the command 'route' to find out what the default gateway is and note it as $previous_gw!
 
NetworkManager is best configured via the UI.

 * Connect with a VNCClient to the VM, e.g. by using:
```    
vncviewer localhost:9${i}59
```
* Go to the NetworkManager icon in the upper right corner and right click on it
* Choose 'Edit Connections'
* Select the host-only network
* Click on 'Edit'
* Go to the tab 'IPv4 Settings'
* Choose 'Manual' as the method
* Add the address:

| IP              | Network mask        | Gateway     |
| -------------   |---------------------|------------ |
|${previous_ip}+10|255.255.255.0        |$previous_gw |

The IP '${previus_ip}+10' means to use for instance 192.168.56.111 instead of the previous one which was 192.168.56.101. If the IP '${previous_ip}+10' is already taken by another VM then assign the next one and so on.

* Use the '$previous_dns' as DNS server
* Reboot
* Check if you can still ping the outside world from the VM
* Check if you can still connect via SSH to the forwarded port
* Check if you now can directly connect from other hosts in the same host-only network

### Install Dependencies

In the VM download additional dependencies:

  * Install OpenSSL

```
yum install openssl
```

  * The package 'wget' is already installed
  * Download the Couchbase Server RPM and place it under '/root/Downloads':

```
mkdir /root/Downloads
cd /root/Downloads
wget $cb_package_url
```

Whereb '$cb_package_url' is the url to the Couchbase package (see above).

  
## Development Instance

Create this VM with the same VM settings as the other ones, but:

   * Name the machine CentOS6-DCJW-Dev 

We need to install with a graphical user interface for this VM.

* In the boot menu of the LiveCD choose just 'Install'
* Use the same settings as before, but this time via the GUI
* If asked for the user creation then create an user 'couchbase' with password 'couchbase'

The network configuration part can be exactly performed as for the other machines before. It's especially important that the Development machine is in the same NAT and host-only network

Don't download the same dependencies, other software will be installed now.

TODO
