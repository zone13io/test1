+++
author = ""
comments = true
date = "2017-10-08T11:15:12+01:00"
draft = false
image = ""
menu = ""
share = true
tags = ["wifi", "wifi monitoring","wifi sniffing","raspberry pi","wimonitor","Windows","kali","wifi pentest", "wifi cracking wpa2 handshake"]
title = "Wi-Fi packet sniffing / monitoring on Windows using Raspberry Pi - inspired by Wimonitor"

+++

[Wimonitor](https://www.hackerarsenal.com/products/wimonitor) is a wonderful product from Hacker Arsenal that saves pentesters the hassle of having to configure VMs, carry compatible wireless cards that support monitor mode etc. and comes with a web interface to do the configurations. It gives you the flexibility to plug in the device into the Ethernet port and start Wi-Fi monitoring on any OS. Basically it is a [tp-link TL-MR3020](http://www.tp-link.com/il/products/details/cat-14_TL-MR3020.html) router with a custom firmware that does all the monitoring part and sends the packets to the host laptop (or Mac !) where you can start Wireshark and concentrate on the packet analysis.

I haven't got one yet, but have been hearing good reviews about the product since launch. The shipping cost to EU is a bummer :(. Hopefully they will start shipping from EU soon.

*Meanwhile - why not try this on a Raspberry Pi ?*

<!--more-->

The Pi can run a variety of OS including Kali and I have had good success using Pi's during Wi-Fi pentests. The Raspberry Pi 3B is powerful enough to do stable monitoring and with a few simple steps can be converted to a Wimonitor (well, almost !). 

The awesome folks at Hacker Arsenal have done a brilliant job in building Wimonitor. It is a stable plug n play device with firmware support expected. So if you are a beginner to Wi-Fi security or need a trouble free monitoring tool, I would recommend going with Wimonitor.

#### **Required hardware**

- Host laptop running Windows(tests and screenshots done on Windows 8.1)
- Raspberry Pi 3B, micro SD card, power adapter (USB 3.0 power should be enough to power the Pi + wireless card)
- Ethernet cable to connect the Pi to host laptop
- Wi-Fi card supporting monitor mode (e.g. TL-WN722N v1)

#### **Setup**

- Burn [RASPBIAN STRETCH LITE](https://www.raspberrypi.org/downloads/raspbian/) on to the micro SD card. The OS is light weight and comes with out of the box monitor mode support for cards like TL-WN722N. If you are not sure on how to burn the OS, steps can be found [here](http://www.raspberry-projects.com/pi/pi-operating-systems/win32diskimager).


- Once Raspbian is written to the micro SD card, enable SSH on the Pi. This can be done by creating a blank file named *ssh* (note: no file extension) on the micro SD card.


- In order to make the host laptop communicate with the Pi, the easiest method is to share the host laptop Wi-Fi with the Pi over Ethernet. This will ensure that the Pi gets an IP address in the range 192.168.137.x. To do this, go to network connections (ncpa.cpl), right-click on the Wi-Fi adapter and select Properties. Under the Sharing tab, select the Ethernet adapter to which you will plug in the Pi. Click OK.

![pi-monitor-wifi-properties](/images/pi-monitor-wifi-properties.png)

- It is now time to connect the Pi to the host laptop. Insert the micro SD card into the Pi's slot, Wi-Fi card into the USB port, connect the Ethernet cable between the host laptop and Pi and power ON.


- Once the Pi boots up, it should get an IP using the shared connection. The easiest way to discover Pi's IP address is to use [nmap](https://nmap.org/download.html) on the host laptop and do a host discovery on 192.168.137.1 /24 subnet.

![pi-monitor-nmap-scan](/images/pi-monitor-nmap-scan.png)

- Open [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) and ssh into Pi. The default username / password is **pi** / **raspberry** for the Raspbian OS we are using.

![pi-monitor-putty-login](/images/pi-monitor-putty-login.png)

- To set a static IP for the Pi, open up */etc/dhcpcd.conf* and add the following lines at the end of the file:

`interface eth0`

`static ip_address=192.168.137.100/24`

`static routers=192.168.137.1`

`static domain_name_servers=192.168.137.1`

- I prefer to use key based authentication for the SSH login. To do that open PuTTYgen that comes with PuTTYgen and generate a key pair.

![pi-monitor-puttygen](/images/pi-monitor-puttygen.png)

- Create a **.ssh** folder inside the Pi's home folder. Then create a file named **authorized_keys** inside the **.ssh** folder. Paste the public key from PuTTYgen into the **authorized_keys** file. Save the private key from PuTTYgen. Restart SSH service on the Pi. Also change the user **pi'**s default password on the Pi.

![pi-monitor-ssh-setup](/images/pi-monitor-ssh-setup.png)

- The Raspbian lite OS by default does not ship with the necessary packages to do the monitoring. So to install them, follow the steps below:

`sudo apt update`

`sudo apt install aircrack-ng tcpdump -y`

- Test that the wireless card is capable of doing monitoring on the Pi.

![pi-monitor-test-monitor](/images/pi-monitor-test-monitor.png)

- Now that we verified that the monitoring is working on the Pi, we can SSH into the Pi, run tcpdump on Pi and feed it to Wireshark running on the host laptop. [plink.exe](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) is command line interface to PuTTY.exe that makes all of this possible on Windows. For ease, I have put plink.exe, the SSH private key from PuTTYgen in the same folder. If they are in different folders, change the paths in the below command appropriately.

The one-liner to do all of this is:

`plink.exe -ssh -i pi-monitor-ssh-key.ppk pi@192.168.137.234 "sudo tcpdump -ni wlan1mon -s 0 -w -" | "C:\Program Files\Wireshark\Wireshark.exe" -k -i -` 

![pi-monitor-start-command](/images/pi-monitor-start-command.png)

![pi-monitor-wireshark](/images/pi-monitor-wireshark.png)

![pi-monitor-handshake-capture](/images/pi-monitor-handshake-capture.png)

To monitor only channel 1,

`sudo iwconfig wlan1mon channel 1`

As you can see, a Raspberry Pi can be used to do Wi-Fi monitoring with minimal effort and pipe the output for viewing using Wireshark on Windows or any OS using the same method.

#### **The "minimal" setup to get up and running**

The above steps are a one time setup. For subsequent monitoring, plug in the Pi with wireless card, power ON the Pi and run the below command to create a monitor mode on the Pi.

`plink.exe -ssh -i pi-monitor-ssh-key.ppk pi@192.168.137.100 "sudo airmon-ng start wlan1"`

Fire up Wireshark to start the monitoring.

`plink.exe -ssh -i pi-monitor-ssh-key.ppk pi@192.168.137.100 "sudo tcpdump -ni wlan1mon -s 0 -w -" | "C:\Program Files\Wireshark\Wireshark.exe" -k -i -` 

![pi-monitor-setup-1](/images/pi-monitor-setup-1.png)

![pi-monitor-setup-2](/images/pi-monitor-setup-2.png)

![pi-monitor-setup-3](/images/pi-monitor-setup-3.png)

That's it. :)

#### Summary

**Pros**:

- No need to worry about chunky VMs. Wi-Fi monitoring can be performed on Windows with Wireshark and plink.exe. I haven't tried on Linux / Mac OS yet, but should be able to pipe the SSH output to Wireshark (?).
- Cheap Wi-Fi monitoring. If you are already into pentesting or similar, you should be having a RPi, Wireless card lying around. :)
- Wimonitor is limited to the onboard Wi-Fi chipset (?). Raspberry Pi in this setup has the ability to use any (multiple) wireless card with monitor mode support. Only needs to plug the Pi into a power socket or use a batter pack with more juice than a USB hub to power the cards. How about a Yagi–Uda antenna with RPi's wireless card ?
- Minimal setup required. Raspbian, Kali comes with out-of-the-box support for cards like Alfa AWUS036NHA, TL-WN722N etc. Once the above steps are done, it is just a matter of plugging in the Pi with wireless card, and running the two lines of commands to start packet analysis.
- RPi has enough firepower to do the monitoring comfortably. In a relatively busy Wi-Fi neighborhood, the Pi hardly consumes 35 MB of RAM in total with the above setup.

**Cons**:

- Not plug n play like Wimonitor. Some initial setup (2 lines at cmd :)) is required to get it up and running.
- No web interface for configuration changes.
- No channel subset hopping capabilities for now.



Hope you find the post useful.