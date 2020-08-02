+++
comments = true
date = "2016-09-17T15:25:24+01:00"
draft = false
menu = ""
share = true
tags = ["Python", "Twitter"]

categories = ["Coding"]
title = "Python code to retrieve the latest tweet from a user"

+++

This is a Python code snippet to retrieve the latest tweet from a user by making use of the Twitter developer API.

<!--more-->

The Twitter API details can be obtained from the [developer portal](https://apps.twitter.com/).

Github [link](https://github.com/zone13/python_dev/blob/master/twitter_latest_tweet.py).

```python
#!/usr/bin/env python

#Python script to retrive the latest tweet from a user
#Created by zone13.io (https://www.zone13.io)
#Version 1.0
#Usage - ./script.py <twitter_handle>
#e.g. ./script.py rihanna
#Adapted from https://github.com/bear/python-twitter
#Requirement - pip install python-twitter

import sys,twitter
api = twitter.Api()

#Populate your twitter API details below
consumer_key = ''
consumer_secret = ''
access_token_key = ''
access_token_secret = ''

api = twitter.Api(
    consumer_key=consumer_key,
    consumer_secret=consumer_secret,
    access_token_key=access_token_key,
    access_token_secret=access_token_secret
)

def user_tweet(thandle):
	statuses = api.GetUserTimeline(screen_name=thandle)
	return statuses[0].text
	
if __name__ == "__main__":
	latest_tweet = user_tweet(sys.argv[1])
	print latest_tweet
```

