---
layout: post
title: how_my_site_works
---

One of my friend's dad was told me, when you're young you have energy and time and no money, middle aged you have money and energy but no time and when you're older money and time but no energy.

With that in mind, diving into the doing the system administration for the site gave me pause, using higher level provider, like [Heroku](www.heroku.com), would save a significant amount of time from my middle aged, time lacking self. I oscillated on this a bit and decided that that added control and the learning experience would be a good time investment.

Initially, I had pictured a single command deploying containers to my VPS in a few keystrokes of awesomeness that only I might ever appreciate. I plan to get there but this post will share an intermediate step in the journey to arrive at that single command. I have achieved a simple and safe workflow to deploy an application, an application I plan to simplify, and a blog to a VPS relatively simply.

At a high level, the request flows in the site look like:

![HTTP Request Flow]({{site.url}}/blog/assets/diagrams/website_request_flow.png)

The key tooling used
* [NGINX](https://nginx.org/en/)
* [docker](https://www.docker.com/)
* [docker-compose](https://github.com/docker/compose)
* [letsencrypt](https://letsencrypt.org/)
* [jekyll](https://jekyllrb.com/)

## NGINX
The part of the setup that is not dockerized is NGINX. NGINX is installed on the VPS and is used to accept incoming HTTP requests and then route them as needed to the blog and the application. The VPS firewall needed to be configured and Digital Ocean provides great documentation to set this up correctly.

The setup for encryption with LetsEncrypt and NGINX running locally on the VPS was very simple using [Certbot](https://certbot.eff.org/lets-encrypt/ubuntufocal-nginx). Initially, I went down the path of using docker for each and use docker for my local dev with self signed certs but I opted for the simpler configuration on the VPS.

With the firewall setup and the certs in place HTTPS Requests can arrive to NGINX. The requests with a route of  /blog go to the filesystem and get blog files. Other requests are proxied to the application that that is running in docker.

The application has two sets of routes:
1. to handle assets
1. to handle app all other app requests

This post provides more information on providing assets using NGINX.

## Docker for app
Docker and docker-compose are used to the deploy the application. Application changes are built using docker-compose, tagged, sent to docker hub and than pulled from docker hub via a separate docker-compose script.

## Deployment Steps
The following steps are taken to deploy the blog and the application.

### Blog
After a blog post is written and reviewed the following done to deploy, Jekyll can generate the blog site locally for testing. The output from jekyll are static html files
1. jekyll -b
1. scp the directory to the server - scp _site/ user@<vps_ip_address>:/var/www/blog

### Application
1. Dev Machine: docker-compose build
1. Dev Machine: docker tag app_name docker_hub_user/app_name:1
1. Dev Machine: docker push docker_hub_user/app_name:1
1. VPS: Update app_name image tag to pull the latest
1. VPS: docker-compose down
1. VPS: docker-compose up

## Summary
The opportunity to work with NGINX, LetsEncrypt, docker and docker-compose was fun and it is rewarding to see the site load nearly instantly. Despite the time investment, I would still consider using a higher level container or app solution if I find my time slipping away.
