+++
author = ""
comments = true
date = "2017-07-18T22:19:34+01:00"
draft = false
image = ""
menu = ""
share = true
tags = ["Oracle VirtualBox", "Kali 2017","wifi security"]
title = "Connecting USB devices to VirtualBox using USB 3.0 ports"

+++

This took me quite a bit of time to figure out, hopefully someone finds the steps useful.

Many of the wireless cards that support monitor mode are all USB 2.0 devices - e.g. Alfa AWUS036NHA, TP-LINK TL-WN722N and run into problems while connecting to newer laptops that come with only USB 3.0 ports. If you try to connect the wireless card to the USB 3.0 port and then try to attach it to VirtualBox VMs like Kali, it will not work straightaway. To fix this issue, follow the below steps.

- Power-off the Kali VM and then connect the wireless card to the USB 3.0 port.

- Under the Kali VM Settings on VirtualBox, find the Vendor ID and Product ID by hovering the mouse pointer over the wireless card name as shown below (click the USB icon with '+' sign to list the connected devices). In this case the Vendor ID is 0CF3 and the Product ID is 9271.

  ![vbox-1](/images/vbox-1.png)

- Add a new USB filter using the Vendor ID and Product ID of the device that needs to be connected. Make sure USB 3.0 is selected as the controller.

  ![vbox-2](/images/vbox-2.png)

- Unplug the wireless card from the USB 3.0 port and power on the Kali VM. Once the Kali VM is powered on, re-attach the wireless card to the USB 3.0 port and it should automatically connect to the Kali VM.

