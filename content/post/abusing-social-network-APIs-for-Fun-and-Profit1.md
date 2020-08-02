+++
author = ""
comments = true
date = "2017-01-31T23:58:30Z"
draft = false
image = ""
menu = ""
share = true
tags = ["Data exfiltration", "Red team","Python","Facebook","Penetration testing"]
title = "Abusing social network APIs for Fun & Profit - Facebook API"

+++

***Question - In a controlled corporate environment with DLP solutions monitoring the HTTP and Email traffic, how would you perform data exfiltration during a Red Team Pentest ?*** 

***'One' of the answers would be to use social networking sites, let's look at Facebook in this post.***

![fb_post](/images/fb_post.png)

Data exfiltration using Facebook (FB) and the like is nothing new. There has been various instances where these networks have been used as C&C, data receivers etc..

Why are social networking sites a good option - one major reason is that they are loosely controlled (not blocked in many organizations - is LinkedIn blocked at your workplace?)

FB allows [63,206 characters](http://mashable.com/2012/01/04/facebook-character-limit/#s7MHOr9Q6Gqo) in a single post, which is quite cool. So can we use that as a receiver, i.e, post data to a FB post from a corporate environment and then retrieve it from outside??

Ok, as some of you are already thinking - that would mean giving up the data to Facebook. Solution - Use Encryption  - encrypt the data using AES or similar and then post to FB. (Maybe?)

Well, 63,206 is not a big number. What if I want to send out a bigger file? - Split the file into chunks, say 50,000 characters into each post. Finally combine all of them.

Surely you don't want to login to your personal Facebook to paste data to your wall (!). Let's use Facebook APIs.

Now, I would assume that it is not always ASCII data that we would want to send out. E.g. Zip archives, image files or anything for that reason. So what do you do?

The approach I took is as below:

Read the file in binary mode --> Do a base-64 encoding of the binary data which gives the ascii version --> split them up as necessary and post to FB using API from inside the corporate network --> Retrieve the data from the (multiple) FB posts from outside and combine them --> do the base-64 decoding --> write the output in binary mode to file --> BINGO !!

Now, one thing you can also do is to Unpublish the Page / control the viewing to which you are making the post. That way, it will ensure that only the Page owner can view the posted data. 

![unpublished_page](/images/unpublish_page.png)

**<u>Red Team scenario</u>**

I would like to present the usage through a Red team exercise scenario where the task is to perform exfiltration of a sample file (of the order a few KBs magnitude). There is nothing to prevent the same method from being used to send out a large file; for the sake of the PoC, I will keep it simple. :)



<u>Explanation</u>



1. During a Red team exercise, you get access to a machine from where the data (file) needs to be pulled. Imagine that the DLP solutions would detect you from sending out data via webmail, file upload solutions and the only open ports for outside access are 80 and 443. Definitely you don't want to risk detection on port 80.
2. That leaves us with port 443. Say for instance there is a whitelist policy in place that only allows access to specific websites and incidentally Facebook is one of them (!). [As a side note, the same method can be allowed to send out data by abusing APIs provided by other social media sites as well. Time to block LinkedIn API from corporate network? :) ]
3. The method can be used as follows:
   1. Suppose the data to be sent out is in the form of a zip file (**secret.zip**) on the target system that we have access to.
   2. Read the zip file (I'm using Python for the PoC) in binary mode. 
   3. Do a base-64 encode of the binary data
   4. Post the base-64 encode of the data to FB
   5. Retrieve the encoded data from outside the corporate network, again FB API can be used.
   6. Decode the base-64 encoded data to binary and write to file (zip)

The base for the Python scripts have been adopted from [link 1](http://nodotcom.org/python-facebook-tutorial.html) and [link 2](https://github.com/minimaxir/facebook-page-post-scraper/blob/master/get_fb_posts_fb_page.py). All credits to the original work for making this simple. 

Python script for data exfiltration - [here](https://github.com/zone13/python_facebook/blob/master/put.py)

Python script for retrieving the FB post and writing to file - [here](https://github.com/zone13/python_facebook/blob/master/get.py)

This also goes to demonstrate how Facebook APIs can be exploited for malicious purposes, e.g. a malware with API tokens that can post to a Facebook page / wall maintained by the bad guy. 