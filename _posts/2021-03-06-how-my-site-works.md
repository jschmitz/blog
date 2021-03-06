---
layout: post
title: How this Site Works
date: 2021-03-06 16:53 -0600
---
One of my friend's dad was told me, when you're young you have energy and time and no money, middle aged you have money and energy but no time and when you're older money and time but no energy.

With that in mind, diving into the system administration efforts for the site gave me pause, using higher level provider, like [Heroku](www.heroku.com), would save a significant amount of time for my middle aged, time lacking self. I oscillated on this a bit and decided that that added control and the learning experience would be a good time investment.

Initially, I had pictured a single command deploying containers to a Virtual Private Server (VPS) in a few keystrokes of awesomeness. I plan to get there and I scaled back on the initial iteration. This post will share how that initial iteration works and we'll get that that awesome single command in the future. I have  a simple and safe workflow to deploy an application, an application I plan to simplify, and a couple simple commands to deploy the blog portion.

At a high level, the request flows in the site look like:

![HTTP Request Flow]({{site.url}}/blog/assets/diagrams/website_request_flow.png)

The key tooling used:
* [NGINX](https://nginx.org/en/)
* [Docker](https://www.docker.com/)
* [Docker Compose](https://github.com/docker/compose)
* [Let's Encrypt](https://letsencrypt.org/)
* [Jekyll](https://jekyllrb.com/)

## NGINX
NGINX is installed locally on the VPS and is used to accept incoming HTTP requests for my domain and NGINX routes requests   to the blog and the application. The VPS firewall needed to be configured to permit HTTP communication and Digital Ocean provides great documentation to set this up correctly. Here are two that are helpful:
* https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-20-04
* https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-18-04

The setup for encryption with LetsEncrypt and NGINX running locally on the VPS was very simple using [Certbot](https://certbot.eff.org/lets-encrypt/ubuntufocal-nginx). Initially, I went down the path of using docker for each and use docker for my local dev with self signed certs but I opted for the simpler configuration on the VPS.

With the firewall setup and the certs in place HTTPS Requests can arrive to NGINX. The requests with a route of  /blog go to the filesystem and get blog files. Other requests are proxied to the application that that is running in docker.

The application has two sets of routes:
1. to handle assets
1. to handle app all other app requests

This post provides more information on providing assets using NGINX.

## Deploying the App
Docker and docker-compose are used to the deploy the application. Application changes are built using docker-compose, tagged, sent to docker hub and than pulled from docker hub via a separate docker-compose script.

The following steps are taken to deploy the blog and the application.
1. Dev Machine: docker-compose build
1. Dev Machine: docker tag app_name _docker_hub_user/app_name:incrementing_number_
1. Dev Machine: docker push docker_hub_user/app_name:1
1. VPS: Update app_name image tag to pull the latest
1. VPS: docker pull _docker_hub_user/app_name:incrementing_number_
1. VPS: docker-compose down
1. VPS: docker-compose up -d

##  Deploying the Blog
After a blog post is written and reviewed the following done to deploy, Jekyll can generate the blog site locally for testing. The output from Jekyll are static html files
* rebuild the blog site:
  1. jekyll -b
* scp the directory to the server:
  1. scp -r _site/ user@<vps_ip_address>:/var/www/blog

## Summary
The opportunity to work with NGINX, LetsEncrypt, docker and docker-compose was fun, most of the time, and it is rewarding to see the site load nearly instantly. Despite the time investment, a higher level container or app solution is still appealing. As of now, the plan is to iterate the current setup and arrive at that single command awesomeness.
