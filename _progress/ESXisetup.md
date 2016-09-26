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
