---
layout: post
title: "The Agony Of Moving To Docker"
subtitle: "The One Where We Pulled Our Hair Out!"
description: "How we migrated our servers to Docker, and the issues we had along the way"
date: 2021-01-31
author: Ben Cousins - Managing Director
hero-image: 
published: true
---
In Computer Science, there are two types of people. Those who use bare metal and those who virtualise. Until now, we've been virtualising using VMWare ESXi and running things on virtual servers, which has served us well. As we reach a point where we need to expand, we look for options that allow us to develop fast, and deploy fast. 

As a SysAdmin, I had always considered myself capable of doing everything that I needed to. Right down to spending a majority of my time in a terminal. 

A couple of years ago, I looked into automated deployments, and in that time we've gone from Gitlab to Github. Our deployment needs changed and we moved to Jenkins and Github. 

Jenkins is a versatile piece of software - you can do just about anything you can think of on it. We are now building Docker Containers, deploying them to our Harbor instance, and then are using Jenkins to pull the containers. 

Getting to this point was a bit rough. We ran into a few problems. 

## Certificate Valid For pfSense
`Docker Login` requires a valid certificate to log into a repository. Good, secure software. No issues there. But sometimes weird things happen when both your Jenkins instance and your Container Repository are on the same network. Building in one instance gave us this: 

<a href="/static/img/blog/agony-of-docker/img1-pfsense.png" target="_new">
	<img src="/static/img/blog/agony-of-docker/img1-pfsense.png">
</a>

Initially, we believed that the way that SSH works was to do a DNS lookup and try to connect to the *external IP* of the server. We didn't account for the fact the external IP was the same as the firewall IP. What we think happened was that the firewall went "Oh, that's my WAN IP, we'll quietly not send that out", and tried to redirect the traffic. 

The way that firewalls work is like your home router, you forward ports on your WAN interface. If pfSense doesn't push the traffic to the WAN interface, stupid things happen. 

### How We Fixed It
So, the way the network is set up, all HTTP and HTTPS traffic hits a reverse proxy configured on its own virtual server. This includes our Docker traffic. So instead of going through the WAN on pfSense, we set an /etc/hosts entry for the IP of the reverse proxy, and that fixed the issue. Basically, bypassing the middleman allowed us to push containers without pushing to the external internet, using the LetsEncrypt certificate that we would be using if we pushed to external. Security, ahoy. 

## No Such Container
Docker is... interesting. And so is Bash. 

<a href="/static/img/blog/agony-of-docker/img2-docker.png" target="_new">
	<img src="/static/img/blog/agony-of-docker/img2-docker.png">
</a>

We had been using a naming convention that included dashes. It turns out, Bash doesn't like those. Or Jenkins doesn't. Or something. I don't know, I don't care to find out. I spent a good few hours running builds and logged into the server because I thought I was going insane. That container definitely does exist and there aren't any spelling errors. 

### How We Fixed It
So what happened? Not sure. We dropped the dash and it suddenly started working for us. Your mileage may vary, but that's what worked for us. 

## Conclusion
Docker is a fantastic software, automation is hard. But oh so worth it. Everything is now configured to build as soon as a Github commit gets pushed, it'll deploy automagically, and everyone will be happy. Including our clients. 