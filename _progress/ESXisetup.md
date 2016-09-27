Guide to setting up ESXi with FreeNAS as a guest VM and hosting datastore for all other VMs.

I have a small home setup that includes two servers that I wanted to condense down into a 1 small box. I have been having some issues with my old hardware causing issues with my baremetal FreeNAS install, and needed to replace it for something more stable. I have a tight budget and want to reuse some of my exsisting equipment to save some money.

Current Setup:
1. 1x WDC Red 5TB - non-raid storage
2. 4x Mix of 1TB drives in an ZPool1 

New Setup:

1. [Gigabyte Mini-ITX Motherboard (GA-B150N-GSM)](https://www.amazon.com/gp/product/B019GZC908/ref=oh_aui_detailpage_o01_s01?ie=UTF8&psc=1)
  * This motherboard supports vt-d for support with PCI passthrough to a VM. Does not support ECC memory.
2. [Intel BX80662I36100 Core i3-6100 3M Cache, 3.70 GHz Processor](https://www.amazon.com/gp/product/B015VPX2EO/ref=oh_aui_detailpage_o01_s01?ie=UTF8&psc=1)
  * Dual core processor with 2 with two hyperthreaded cores. Supports vt-d.
3. [Ballistix Sport LT 32GB Kit (16GBx2) DDR4 2400 MT/s](https://www.amazon.com/gp/product/B01B4F3IJY/ref=oh_aui_detailpage_o00_s00?ie=UTF8&psc=1)
  * Cheapest 16GB DIMM on the supported memory list. Maxed out supported memory so I wont have to worry about running out of RAM for additional VMs
4. [Silicon Power 32GB Blaze B30 USB 3.0](https://www.amazon.com/gp/product/B00H7PBWK8/ref=oh_aui_detailpage_o01_s00?ie=UTF8&psc=1)
5. [uxcell® USB 3.0 A Type Female to Female 20 Pin Box Header Slot Adapter](https://www.amazon.com/gp/product/B007PODI1W/ref=oh_aui_detailpage_o01_s00?ie=UTF8&psc=1)
  * these two pieces allows me to install ESXi and the 1 local guest VM for FreeNAS and to hide it away internally.
6. [LIAN LI PC-Q25B](http://www.newegg.com/Product/Product.aspx?Item=N82E16811112339)
  * Mini ITX case with internal 5 3.5 hotswap bay.
  
About ECC memory
ECC memory is importatnt when using FreeNAS to prevent issues with bad data getting written to your ZPool. I would perfer to use ECC memory, but in my case the extra cost for equipment that fits my needs that also support ECC was not going to work for me. If you plan to use non-ECC, I cant stress enough to keep backups of your data with incramental changes (crashplan). [There are some great articles about the difference between using ECC and non-ECC with ZFS.](http://jrs-s.net/2015/02/03/will-zfs-and-non-ecc-ram-kill-your-data/). I feel this article is a great read, I feel it puts a correct light on the comparison between ECC and Non-ECC for your server. 

Installing ESXi:
I installed ESXi onto a USB Flash drive. This method works great since ESXi does not write many much data to the storage drive. Any configuraiton changes will get written to the USB flash drive in 10 minute incraments. So if there are no changes there will be no data written to the drive.

Installing FreeNAS as a Guest VM:
So the tricky part is next, We need to setup vt-d on our system to passthrough the PCI ATCI controller to the FreeNAS VM. Since we are going to passthrough the controller we wont have access to it from our ESXi 6 Hypervysor so we need to rely on a USB drive to store our FreeNAS VM.

1. configure vt-d in the bios
2. disable usb passthrough to vms so we can use the USB drives for our datastore for FreeNAS (we need this locally since we have to load freenas before the storage comes up.
 *http://www.virten.net/2015/10/usb-devices-as-vmfs-datastore-in-vsphere-esxi-6-0/
```
Stop the USB arbitrator service. This service is used to passthrough USB device from an ESX/ESXi host to a virtual machine. (When disabling it, you can no longer passthrough USB devices to VMs)
 ~ # /etc/init.d/usbarbitrator stop
 Use this command to permanently disable the USB arbitrator service after reboot.
~ # chkconfig usbarbitrator off
Plug in the USB Device to your ESXi host
Get the device identifier (mpx.vmhbaXX). You should see the USB Device in /dev/disks/:
~ # ls /dev/disks/
Write a GPT label to the device (Assuming that the Device ID is mpx.vmhba36)
~ # partedUtil mklabel /dev/disks/mpx.vmhba36\:C0\:T0\:L0 gpt
To create a partition you need to know the start sector, end sector, which depends on the device size and the GUID.
The start sector is always 2048
The GUID for VMFS is AA31E02A400F11DB9590000C2911D1B8
The end sector can be calculated with the following formula (Use the numbers from getptbl):
~ # partedUtil getptbl /dev/disks/mpx.vmhba36\:C0\:T0\:L0
gpt
1947 255 63 31293440
1947 * 255 * 63 – 1 = 31278554

You can also calculate the endsector with the following command:

~ # eval expr $(partedUtil getptbl /dev/disks/mpx.vmhba36\:C0\:T0\:L0 | tail -1 | awk '{print $1 " \\* " $2 " \\* " $3}') - 1
31278554
Create the VMFS partition (Replace with your endsector)
~ # partedUtil setptbl /dev/disks/mpx.vmhba36\:C0\:T0\:L0 gpt "1 2048 31278554 AA31E02A400F11DB9590000C2911D1B8 0"
Format the partition with VMFS5
~ # vmkfstools -C vmfs5 -S USB-Stick /dev/disks/mpx.vmhba36\:C0\:T0\:L0:1
The USB-Stick should now appear in your datastores view.
```
3. manually enable the pci ahci controller to be selectable for passthrough to the FreeNAS VM.
 *https://forums.servethehome.com/index.php?threads/esxi-6-0-passthrough-onboard-sata.8902/
 *https://forums.servethehome.com/index.php?threads/usb-thumb-drive-datastore-built-in-intel-lynx-point-ahci-controller-vt-d-successful-aio.8716/
 *http://www.vmware.com/pdf/vsp_4_vmdirectpath_host.pdf
```
You'll need the vendor and device id. Run lspci at the ESXi command line. Look for the line pertaining to your mass storage controller. It will likely be "0000:00:XX.X Mass Storage Controller: Intel Corporation Wellsburg AHCI Controller".

Make a note of the hex prefix (0000:00:XX.X) and then run 'lspci -n'.

Find the line corresponding to the prefix, it'll look something like '0000:00:1f.2 Class 0106: 8086:8d62 [vmhba2]'. That last part, "8086:8d62" in this case, are the vendor and device id.

Then add them to your passthru.map file like so:

# INTEL Wellsburg AHCI
8086 8d62 d3d0 false

```
4. install freenas
5. enable the ahci pci inside the free
6. configure freenas the way you need it.


TODO:

1. offsite backup with differential versioning. CrashPlan 
1. securing systems
2. improving freenas, and other systems performance.
 *change storage network MTU to 9000
 *experiment with iSCSI storage vs NFS
