+++
author = ""
comments = true
date = "2016-09-23T19:55:06+01:00"
draft = false
share = true
tags = ["Responder", "NTLM","cracking"]
title = "Cracking NTLMv2 responses captured using responder"

+++

In the previous [post](https://zone13.io/post/Raspberry-Pi-Zero-for-credential-snagging/), a Raspberry Pi Zero was modified to capture hashes (or rather NTLMv2 responses from the client).

Let's see how [hashcat](https://hashcat.net/hashcat/) can be used to crack these responses to obtain the user password. I will be using dictionary based cracking for this exercise on a Windows system. 

<!--more-->

### Setup

Download the latest version of hashcat binaries from [here](https://hashcat.net/hashcat/) - v3.10 at the time of writing.

Unzip the 7z file and open a command prompt at the unzipped location.

For convenience, I have created two directories in the hashcat folder:

**hashes** - to store the responses that need to be cracked

**cracked** - to store the cracked passwords

### Captured responses

The client response captured by Responder was:

```
[HTTP] NTLMv2 Client   : 192.168.154.131
[HTTP] NTLMv2 Username : MANGO\neo
[HTTP] NTLMv2 Hash     : neo::MANGO:1122334455667788:2F6CC22CFDC387CFEEB2D325E8997564:0101000000000000C79708D95F14D2011D717D511A761654000000000200060053004D0042000100160053004D0042002D0054004F004F004C004B00490054000400120073006D0062002E006C006F00630061006C000300280073006500720076006500720032003000300033002E0073006D0062002E006C006F00630061006C000500120073006D0062002E006C006F00630061006C0008003000300000000000000000000000001000001047325384B3A000DD5138E15668CB4F44DA5471DAF7A327B5B4E3C19DC485120A001000000000000000000000000000000000000900120048005400540050002F0077007000610064000000000000000000
```

where,

MANGO is my domain name and neo is the user who was logged into the system.

I saved the response into a text file named **hash.txt** in the **hashes** folder created earlier.

![hash](/images/hash.png)

### Cracking using hashcat

The hashcat developers have done a wonderful job in simplifying the cracking process. All you need is a fast  cracking machine and patience. :)

Since this is a dictionary based cracking, there are two scenarios here:

1. Your password list does not contain the user password
2. Your password list has the user password

I wanted to show both scenarios here, starting with the worst case - not having the password in the list.

For the sake of the the demo, I extracted a subset of the passwords from example.dict that comes with hashcat and saved it as **password_list.txt** in the hashcat folder. For the first run, the **password_list.txt** does not have the user password. 

Open a command prompt at the extracted hashcat folder. For NTLMv2 cracking, the hashcat can be run as,

```
hashcat64.exe -m 5600 hashes\hash.txt password_list.txt -o cracked\cracked.txt
```

> If you don't specify -o switch, the password (if cracked) will be stored in **hashcat.pot** file in the hashcat folder.

If the password is not found, this is what you see once hashcat completes the cracking.

Status: **Exhausted**

![password_not_found](/images/password_not_found.png)



For the next run, the case if I have the user password in my password list. I edited the password_list.txt and appended the user password (I created the user, remember :)).

Running the hashcat tool again,

```
hashcat64.exe -m 5600 hashes\hash.txt password_list.txt -o cracked\cracked.txt
```

Status: **Cracked**

![password_found](/images/password_found.png)

To view the cracked password, see the **cracked.txt** file in the folder named **cracked**.



![cracked_out](/images/cracked_out.png)



The success of cracking the password using this method solely depends on whether or not your password list is good enough. 