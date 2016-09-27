Setting up Plex Pass on a minimal CENTOS 7 system inside ESXi 6 and using FreeNAS storage.

1. Install ESXi
2. Configure a Storage network for the media access on Plex.
  *This is going to allow us to keep all data transfers internally to the box. We also wont have any external access to this network from the outside.
   *I call this second network my "Storage Network"
   *Staticlly assign IP addresses on our Storage Newtork. I put it on a different subnet but with the same IP address as the regular network addresses to make it easier to know what ip goes to what system.
2. Install CentOS, I reccomend setting up a Minimal install.
3. Add a new virtual interface to the VM and assign it to our Storage network.
4. Configure our network interfaces
```
To verify the status of Network Manager service:
$ systemctl status NetworkManager.service

To check which network interface is managed by Network Manager, run:
$ nmcli dev status

Lets manually configure an ipv4 address
Go to the /etc/sysconfig/network-scripts directory, and locate its configuration file (ifcfg-enp0s3). Create it if not found.
Open the configuration file and edit the following variables:
BOOTPROTO='static'
IPADDR=192.168.100.12
NETMASK=255.255.255.0
ONBOOT="yes"

"NM_CONTROLLED=no" indicates that this interface will be set up using this configuration file, instead of being managed by Network Manager service. "ONBOOT=yes" tells the system to bring up the interface during boot.
Save changes and restart the network service using the following command:

# systemctl restart network.service
```

4. update centos
$ yum update

5. install esxi vmware tools
$ yum install open-vm-tools

6. enable automatic security updates for our system
  * https://serversforhackers.com/video/automatic-security-updates-centos

7. I'm using NFS to access the media files on the NAS. 
Install the nfs utilities
``
$ yum install nfs-utils nfs-utils-lib
lets create a folder to mount the NFS share to the local plex server
$ mkdir -p /mnt/NAS/Videos #The -p flag: This tells mkdir to create any leading directories that do not already exist. Effectively, it makes sure that myProject gets created before creating myProject/src.
Then add this line to /etc/fstab file: 192.168.100.11:/mnt/Pirate/Videos /mnt/NAS nfs defaults 0 0. 
To test it run mount -a then ls /mnt/NAS/Videos
``
Configure our system to connect to the NFS share on FreeNAS

10. Download, install and configure PLEX
Install WGET:
``
yum -y install wget
wget https://downloads.plex.tv/plex-media-server/1.1.4.2757-24ffd60/plexmediaserver-1.1.4.2757-24ffd60.x86_64.rpm
``
download plex RPM for CentOS
``
$ yum localinstall plexmediaserver-1.1.4.2757-24ffd60.x86_64.rpm
``
Enable PLEX service to start at boot
``
systemctl is-enabled plexmediaserver.service
``
If it returns “disabled” then turn it on with 
``
$ systemctl enable plexmediaserver.service
``
Start the PLEX service manually.
``
$ systemctl start plexmediaserver.service
``

Now go to your Plex media server via a web browser on another machine and start adding libraries – http://192.168.1.146:32400/web/index.html
