+++
date = "2018-03-19T20:47:25+02:00"
draft = false
title = "Restoring real IP address from Cloudflare in logs when using nginx-proxy with Docker"
tags = ["docker","cloudflare","nginx","nginx proxy"]
image = ""
comments = true
share = true
menu= ""
author = "Sid"
+++

While using [nginx-proxy from jwilder](https://github.com/jwilder/nginx-proxy) with Cloudflare, one of the common issues you run into is that the logs contain the Docker internal IP rather than the real external IP passed by Cloudflare. 

![Internal IP](/images/cf-nginx-proxy-1.png)

Note - During my tests, this only applies when nginx-proxy is run as two separate containers -  [jwilder/docker-gen](jwilder/docker-gen) and the official nginx image. Also, I have whitelisted my server to accept connections only from [Cloudflare IP range](https://www.cloudflare.com/ips) to port 443 (Full - Strict config).

<!--more-->

Basically nginx-proxy serves as a high performance reverse proxy that lets you run multiple websites behind it. For more info, please read [this](http://jasonwilder.com/blog/2014/03/25/automated-nginx-reverse-proxy-for-docker/).

Coming back to the issue, all you need to do to fix the IP is to edit the [nginx.tmpl](https://github.com/jwilder/nginx-proxy/blob/master/nginx.tmpl) file that comes with the nginx-proxy.

Original snippet (line 63)

```
log_format vhost '$host $remote_addr - $remote_user [$time_local] '
                 '"$request" $status $body_bytes_sent '
                 '"$http_referer" "$http_user_agent"';
```

Change  **remote_addr** to either **http_cf_connecting_ip** or **http_x_forwarded_for** as [explained](https://support.cloudflare.com/hc/en-us/articles/200170706-How-do-I-restore-original-visitor-IP-with-Nginx-) by Cloudflare.

    log_format vhost '$host $http_cf_connecting_ip - $remote_user [$time_local] '
                     '"$request" $status $body_bytes_sent '
                     '"$http_referer" "$http_user_agent"';
**OR**

    log_format vhost '$host $http_x_forwarded_for - $remote_user [$time_local] '
                     '"$request" $status $body_bytes_sent '
                     '"$http_referer" "$http_user_agent"';
Now start the containers and you will see that logs report the real IP.

![External IP](/images/cf-nginx-proxy-2.png)