+++
author = "Sid"
comments = true
date = "2017-03-27T21:18:17+01:00"
draft = false
image = ""
menu = ""
share = true
tags = ["motioneyeOS", "Raspberry Pi", "video surveillance", "physical security", "Twitter", "Python", "Pi Zero", "Pi 3"]
title = "Twitter notifications for MotionEyeOS video surveillance system on Raspberry Pi"

+++

[MotionEyeOS](https://github.com/ccrisan/motioneyeos) is a wonderful project by Calin Crisan that converts your single board computer into a video surveillance system in a matter of minutes. It is [supported](https://github.com/ccrisan/motioneyeos/wiki/Supported-Devices) on a number of devices and is well maintained with good community support. 

Now that the latest [Raspberry Pi Zero version (W)](https://www.raspberrypi.org/products/pi-zero-w/) comes with onboard Wi-Fi, it makes an ideal candidate to deploy MotionEyeOS across your perimeter and run a cheap video surveillance system with lesser clutter of having to attach a Wi-Fi dongle as in the previous versions of Pi Zero. 

<!--more-->

### Motion notifications

MotionEyeOS offers push notifications in the form of email alerts and custom scripts upon detection of motion. The community has already chipped in with options like [Pushover and IFTTT](https://www.pi-supply.com/make/adding-push-notifications-motioneyeos-formerly-motionpie/). I was playing around with the software on Raspberry Pi over the last few days and thought that Twitter notifications on mobile would be nice to have. I did some basic search to see if someone already implemented this, however couldn't find any and decided I will code my own.

Now the [SD card images](https://github.com/ccrisan/motioneyeos/releases) supplied by MotionEyeOS are stripped down versions of the Raspbian OS with only the necessary features for performance reasons. To enable Twitter notifications using my method below, you will have to install MotionEyeOS on top of Raspbian ([lite version](https://www.raspberrypi.org/downloads/raspbian/) preferred). This is to ensure that the [Python Twitter](https://github.com/bear/python-twitter) module can be installed.

### Installation steps

I assume that you have the latest version of Raspbian Jessie Lite [written](https://www.raspberrypi.org/documentation/installation/installing-images/README.md) to the Micro SD card. I have tested the following method on Pi Zero and Pi 3B and works fine. Use **sudo** where necessary.

1> Power ON the Pi and open a SSH session.

2> Use **raspi-config** to enable the camera module. 

![raspi-config](/images/raspi-config1.png)

![raspi-config](/images/raspi-config2.png)

![raspi-config](/images/raspi-config3.png)

3> Open the **/etc/modules** file and add **bcm2835-v4l2** on a new line.

![](/images/etc_modules_cam.png)

4> Follow the steps mentioned [here](https://github.com/ccrisan/motioneye/wiki/Install-On-Raspbian) under 'Instructions' to complete the installation of MotionEyeOS on Raspbian.

5> Create a dummy Twitter account and setup the API keys. [Here](https://www.slickremix.com/docs/how-to-get-api-keys-and-tokens-for-twitter/) is a good resource on generating the Twitter API keys to use in the script.

6> Install [Python Twitter](https://github.com/bear/python-twitter)

`pip install python-twitter`

7> There are a couple of ways of receiving the push notifications on your Twitter account:

​	a) Setup the dummy Twitter account to post a Tweet every time motion is detected - [script 1](https://gist.github.com/zone13/947e9eea96294a6e070b630378349e70). Configure your Twitter account to follow the dummy account and receive push notifications on your mobile device using the steps [here](https://support.twitter.com/articles/20169887).

​	b) Script variant that tweets the latest captured image along with the tweet - [script 2](https://gist.github.com/zone13/48a4a918331eda4be50afed6eb5f4f2c).

**IMPORTANT**: If you are planning to use this, make sure that your tweets are [protected](https://support.twitter.com/articles/20169886) so that only you or those accounts that you permit can view your tweets. ![Protected tweets](/images/protected_tweets.png)

​	c) Setup the dummy Twitter account to send Direct Messages (DM) to your Twitter account - [script 3](https://gist.github.com/zone13/308d5b39f9f968e21041a53cc9644d22). Enable notifications for DMs on your mobile device.

Using b) above has the added advantage of adding the latest image captured by MotionEyeOS to the Tweet. DMs do not support attaching an image though.

8> Once you finalize the script to use, create folder at **/var/lib/motioneye** and store the python script there. Ensure that the script has execute permissions.

![Motion notifications](/images/motion_notifications.png)

9> Login to the web GUI for MotionEyeOS and configure to run the script under 'Motion Notifications'

### Caveats

1. The Twitter API has limits on the number of tweets and direct messages. However during my tests, none of these limits were hit and I have a fully functional surveillance system at home using this method.
2. In one of the scripts (2), I have added a method to upload the latest image capture to the motion alert tweet. This method has its own privacy implications which is the same as uploading your data to any other cloud service. Make sure that you know what you are doing. :)
3. Do not expose your live feed to internet using port forwarding. In my case, I OpenVPN into my Raspberry instance to view the live feed.
4. Needless to say, use strong passwords for SSH and other service that you enable with MotionEyeOS. Do not re-use the passwords. Also remember that MotionEyeOS stores the password in plaintext in config files. 



Hope this post is useful for those who wish to setup Twitter notifications for MotionEyeOS. If you run into problems, please post a comment below and I will try my best to help.