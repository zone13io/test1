+++
author = "Sid"
comments = true
date = "2017-02-21T22:37:30Z"
draft = false
image = ""
menu = ""
share = true
tags = ["Data exfiltration", "Red team","Python","Facebook","Twitter","Penetration testing","stealth","windows","backdoor"]
title = "Social network based backdoor for pentests"

+++

Once you gain access to a system during pentest, you might want to retain access by means of a backdoor. The most trivial method is to use [metsvc](https://www.offensive-security.com/metasploit-unleashed/meterpreter-backdoor/) which 'unfortunately' is very well fingerprinted by anti-virus software. 

In this post, let us look at how to use a backdoor that uses social network for communications. The method used by the backdoor is identical to what was mentioned in my previous [post](https://zone13.io/post/social-media-based-pentest-dropbox/).

<!--more-->

![design](/images/drop_box_design.png)

> I'm going to assume that the pentester already has a meterpreter shell on the system that he wants to backdoor.

##### Steps

- The Python code for the backdoor is compiled into **exe** using [PyInstaller](http://www.pyinstaller.org/)
- The backdoor is uploaded to the system using the existing meterpreter shell and executed
- The backdoor code polls the pentester's Twitter handle every 30 seconds for commands
- The command retrieved is executed on the system and the output is uploaded to the pentester's Facebook page.



Even if the target network containing the system we are backdooring keeps a check on all traffic (HTTPS), the proxy will only notice the C&C traffic from the system as legitimate API calls to Facebook and Twitter - hence providing the necessary levels of stealth.

##### Video demo (YouTube link)

[![Demo video](http://img.youtube.com/vi/RG5uQGGPDq4/0.jpg)](https://youtu.be/RG5uQGGPDq4)





