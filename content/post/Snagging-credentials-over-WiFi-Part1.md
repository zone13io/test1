+++
author = ""
comments = true
date = "2016-10-03T23:26:44+01:00"
draft = false
image = ""
menu = ""
share = true
tags = ["NTLM", "Credentials","hashcat","Active Directory","WiFi"]
title = "Snagging Active Directory credentials over WiFi - Part 1"

+++

> This issue has been fixed on Windows 8+ as part of MS16-112 issued in September 2016. I haven't checked this on a Windows 7 machine which isn't listed on the Microsoft page. 

#### Basics

The basic theory behind the attack is same as [@mubix](https://twitter.com/mubix)'s [discovery](https://room362.com/post/2016/snagging-creds-from-locked-machines/) - if you connect a network interface to a Windows system and have [Responder tool](https://github.com/lgandx/Responder) poisoning the network, you can obtain the hashes (NTML responses) from the machine without any user intervention. This works even with the computer in **locked state**. 

In my scenario, I'm using a rogue Open WiFi network to grab the hashes. The result is the same and at times, I had better results than plugging in a [Pi Zero](https://zone13.io/post/Raspberry-Pi-Zero-for-credential-snagging/). The Responder tool captures the hashes which can then be cracked using tools like [hashcat](https://hashcat.net/hashcat/). 

<!--more-->

> Note: This only works with a domain joined computer. I haven't seen a local account sending out hashes without user intervention.



****

**Who is affected by this vulnerability and the related attack?**

*This vulnerability is applicable to Windows machines (7+) that are part of an Active Directory domain and connects to Open WiFi access points with 'Connect automatically' option enabled. The vulnerability can be exploited if [MS16-112](https://technet.microsoft.com/en-us/library/security/ms16-112.aspx) patch is not installed on the system.* 

------

 

#### Terminology:

**LLMNR / NBT-NS poisoning** - I assume this is familiar by now. If not, there are a number of good resources that explain how the Responder tool uses this to capture hashes. [Here](https://www.sternsecurity.com/blog/local-network-attacks-llmnr-and-nbt-ns-poisoning) is a good write-up from Stern Security.

**Open WiFi network** - A WiFi network that allows clients to connect to it without needing any form of authentication and there is no channel encryption employed when the clients communicate. This includes public networks like Starbucks, Airport WiFi etc. which allows users to access internet either free or after paying a fee.

**Rogue WiFi access point** - An access point that masquerades as a legitimate access point by advertising the same SSID. 





#### Attack scenario

The catch about Open WiFi networks is that they allow clients to connect, get an IP over DHCP and share a common network with other clients. This means that an attacker waiting on the network can do things like ARP spoofing, Man-in-the-middle etc. to compromise the client. An attacker can always connect to a legitimate Open WiFi network and do poisoning attacks. However, in that case, the chances of him getting detected and logged are more. So, what are the options for an attacker? 

*Create a rogue Open WiFi network and wait for the client to connect.*

**Why should the client connect to the rogue network anyway?**

This is the important bit. On Windows, when a user connects to a WiFi network for the first time, he is presented with an option to 'remember' the network so that the machine connects to the WiFi network every time it is in vicinity. So if you bring up a rogue open WiFi network, the client connects to it automatically without any user intervention. Also, it is to be noted that there are no notification pop-ups when the client connects to the rogue network, possibly a way to ensure seamless connectivity in an ESS network.

![open_wifi](/images/open_wifi.png)

**So, why only Open WiFi networks for this attack?**

A rogue Open access network can be created just by knowing the SSID of a legitimate network. This is unlike a WEP/WPA/WPA2 network which requires the attacker to know the WiFi passphrase in order to create a rogue network and make the client connect.

**So you are telling me that the client must have connected to an Open WiFi access point. Why do you think anyone will connect a corporate laptop that is part of a Active Directory domain to an unsecure Open network?**

Open WiFi networks are all around us. The whole point of enterprises distributing laptops is mobility. Users connect to various networks and establish a VPN connection back to the office network to access the resources. Imagine you are traveling for work. If you need internet access, one or the other time, you will most likely end up connecting to an Open WiFi network - like hotel WiFi, airport WiFi etc. Ask anyone who travels if you are still doubtful. :)

**Ok, so if I travel and indeed connect to Open WiFi networks, how does the attacker know the SSID of the network I have connected to?**

There are various ways for an attacker to find this:

1. Monitor the client probes from the client machines that queries for stored WiFi SSIDs
2. Resources like [Wigle.NET](https://wigle.net) that serve as a database of SSIDs with locations. If needed, these SSID's can be scripted to see which ones elicits a response from client machines
3. Google !! - basic recon stuff - search for the Hotel WiFi network name where the person has stayed (e.g. **Marriott_Guest**)





#### Setup

##### Rogue Open WiFi network 

Rogue APs can be of two types:

- Software based - E.g. Hostapd, Airbase-ng
- Hardware based - E.g. Actual WiFi routers, [WiFi Pineapple](https://www.wifipineapple.com/) 

> I have seen that WiFi Pineapple has a Responder [module](https://www.wifipineapple.com/modules), but I haven't tried it yet. I will leave it for another post.

I decided to go with an actual hardware WiFi router, the [TP-LINK MR3020](http://www.tp-link.com/lk/products/details/cat-14_TL-MR3020.html) for the following reasons:

- ~~WiFi clients are getting smarter and they are able to detect and prevent connecting to software based APs. This can be bypassed using an actual WiFi router to which the clients connect willingly.~~ <font color="red">Tried [hostapd-mana](https://github.com/sensepost/hostapd-mana) as pointed by [@singe](https://twitter.com/singe/status/784290909206646786) and it works well as an Evil twin.</font>

- MR3020 is a wonderful device that can be powered using a battery pack allowing you to even do a 'wardriving' exercise. 

- It has an [OpenWrt](https://wiki.openwrt.org/toh/tp-link/tl-mr3020) firmware with more options. However, I have used the factory firmware in this case.

  ​

##### A Linux machine to run Responder

I used a Raspberry Pi 3 running Raspbian (referred to as RPi here on).

##### An SSH client to remote into RPi and view the live capture 

I connected my laptop to the rogue WiFi and SSH'ed into RPi.



#### Execution

**Step1:** Clone the Responder tool from [GitHub repo](https://github.com/lgandx/Responder) on to the RPi.

**Step 2:** Configure the MR3020 using the SSID (**Z13_guest_wifi** in my case) of the Open WiFi network that the user has connected to in the past. Additionally, set the DNS server under DHCP options to the IP of the RPi so that we get all the DNS queries. The RPi can be given a static IP every time by doing an address reservation. Toggle the hardware switch to 3G/4G so that the RPi can be plugged into it's LAN port.

![MR3020_open_wifi_config](/images/MR3020_open_wifi_config.png)

![mr3020_dns_server](/images/mr3020_dns_server.png)



**Step 3:** Connect the RPi to MR3020 and power on both devices. The RPi boots up and gets an IP from the MR3020. 

**Step 4:** Connect to the rogue WiFi network and SSH into RPi. Run the Responder tool.

`./Responder.py -I eth0 -f -w -r -d -F`



![run_responder](/images/run_responder.png)



**Step 5:** Wait until the client connects to our rogue WiFi network. This can be done in a couple of ways:

1. Wait until the client connects by virtue of higher signal strength
2. De-auth the client from the legitimate network and connect to the rogue AP

Once the client connects, the Responder tool does its magic and captures the hashes. It was observed that *if the client was made to re-connect to the rogue network, the hash was captured much quickly*.

![hash_captured](/images/hash_captured.png)





The captured hashes can then be cracked using tools like hashcat. 



#### Advantages

1. No physical access to the system required, only WiFi proximity is needed
2. Plugging in USB devices like LAN Turtle can be disabled using AD policies. This attack vector has an edge in that case
3. No driver installation required - use of Pi Zero for mubix's attack required the driver to be pre-installed on the target system. 
4. No logs recorded on the enterprise logging systems. The only place where this gets logged is Windows Event Viewer
5. Less chances of discovering that the hashes were grabbed. This gives an attacker ample time to crack the hash on his cracking rig.
6. Ability to do a wardriving exercise to capture hashes from multiple machines which has connected to the same Open WiFi network.
7. The hash was observed to be captured quite quickly (often less than 30 seconds). 

#### Conclusion

This attack scenario can compromise the enterprise's AD domain without even touching the on premise network (wired or wireless). The only info that an attacker needs is the  SSID of '**any one**' Open WiFi network that the user has connected to in the past with 'Connect automatically' option enabled. From a network admin perspective, this can be a nightmare. Of course, the success of obtaining the plaintext credentials depend on whether the NTLM hash can be cracked. With ever increasing computational power and plaintext passwords getting dumped every now and then from various sources, the chances of cracking the passwords are quite good. 

