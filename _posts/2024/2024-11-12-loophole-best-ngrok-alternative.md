---
title: Loophole – Best ngrok alternative for port forwarding
date: 2024-11-12 8:05:00 +0800
categories: [Tools]
tags: [loophole, port forwarding]
image: /assets/img/posts/loophole_port_forwarding/undraw_server_status_5pbv.png
---
<div style="text-align: justify">
Whether you’re testing webhooks, developing an app, or sharing a local web server with someone remotely, you may be familiar with Ngrok. Ngrok has long been the go-to solution for developers who want to expose local ports to the internet. For port forwarding purposes – puplishing a local webserver for post/request data, I initialy used ngrok and was very satisfied. Unfortunately, ngrok has a limitation about number of requests in a month for an account. So that I couldn’t continue using it after a few days.
</div>

After exploring alternative solutions like [pinggy.io](https://pinggy.io), localtunnel, [localhost.run](https://localhost.run), [serveo.net](https://serveo.net), … I found [loophole.cloud](https://loophole.cloud) is the best ngrok alternative.

## Key feature
Putting aside the technical details, here are the advantages I personally see when using loophole:
* Open source [https://github.com/loophole/cli](https://github.com/loophole/cli)
* Free to use, not requests limitation.
* No API tokens needed.
* Automatically generates public and fixed HTTPS URLs.

## How to use
### 1. Install
Download the zip file according to your OS in [https://loophole.cloud/download](https://loophole.cloud/download). In my case I use windows 64bit.

Extract the zip file and add the extracted file path to your environment account.
![](/assets/img/posts/loophole_port_forwarding/loophole_environment.png)

### 2. Port forwarding
Open cmd, connect your account and create a port forwarding.
```
$ loophole account login
```
```
$ loophole http 5000 --hostname duclee
```
This create a fixed HTTPS URL **https://duclee.loophole.site** linked to your **http://localhost:5000**
![](/assets/img/posts/loophole_port_forwarding/loophole_cmd.png)

## Final thoughts
<div style="text-align: justify">
Loophole is an excellent tool for developers who need simple and secure port forwarding without the hassle of accounts or pricing tiers. While Ngrok still holds a lot of value for its advanced features, Loophole shines when it comes to ease of use and openness.
</div>
<div style="text-align: justify">
Give it a try — and enjoy instant port forwarding with zero friction.
</div>
