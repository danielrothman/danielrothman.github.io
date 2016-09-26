Setting up Plex Pass on a minimal CENTOS 7 system inside ESXi 6 and using FreeNAS storage.

Download minimal CENTOS from http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1511.iso

Install it on your ESXi system.

I setup a secondary internal network to pass the storage traffic between systems.
I have a managment LAN and a storage LAN.

I have DHCP on my managament LAN since its connected to my physical network.
Using Static IPs on my storage virtualized LAN.
Lets assign a static IP address to the plex server's interface.

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

update centos
$ yum update

install esxi vmware tools
$ yum install open-vm-tools

enable automatic security updates for our system
https://serversforhackers.com/video/automatic-security-updates-centos

I'm using NFS to access the media files on the NAS. 
In the FreeNAS server lets enable the NFS and point it to our media folder

Plex server lets install the nfs utilities
$ yum install nfs-utils nfs-utils-lib
lets create a folder to mount the NFS share to the local plex server
$ mkdir -p /mnt/NAS/Videos #The -p flag: This tells mkdir to create any leading directories that do not already exist. Effectively, it makes sure that myProject gets created before creating myProject/src.
Then add this line to /etc/fstab file: 192.168.100.11:/mnt/Pirate/Videos /mnt/NAS nfs defaults 0 0. 
To test it run mount -a then ls /mnt/NAS/Videos

We next need to download PLEX server using WGET.
Lets install WGET:
yum -y install wget

wget https://downloads.plex.tv/plex-media-server/1.1.4.2757-24ffd60/plexmediaserver-1.1.4.2757-24ffd60.x86_64.rpm

Install PLEX rpm
$ yum localinstall plexmediaserver-1.1.4.2757-24ffd60.x86_64.rpm

Lets manually start the plex service
$ systemctl start plexmediaserver.service

Make sure Plex starts on system boot. 
systemctl is-enabled plexmediaserver.service
If it returns “disabled” then turn it on with 
$ systemctl enable plexmediaserver.service

Now go to your Plex media server via a web browser on another machine and start adding libraries – http://192.168.1.146:32400/web/index.html
