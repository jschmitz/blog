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
* [Github Actions](https://github.com/features/actions)

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

## Deploying the Apps
The deployment process is a lot of fun. I push a commit to the main branch of the blog or web application repositories and the change is deployed automatically by Github Actions to a Virtual Private Server (VPS).

### Webapp Deploy
Git, Docker, Dockerhub and docker-compose are used to the deploy the application. Application changes are built, tested, a production image is created, sent to DockerHub, pulled from docker hub on the VPS and started on the VPS.

Check out [Webapp Continuous Delivery Script](https://github.com/jschmitz/rockworm/blob/main/.github/workflows/cd.yml)

### Blog Deploy
After a blog post is written and reviewed the following done to deploy, the change is committed and that triggers the deploy process. The process to deploy the blog is much simpler than the webapp. The project is checked out, Jekyll is used to build the blog site and the production directory is SCP'ed to the VPS

Check out the [Blog Deployment Script](https://github.com/jschmitz/rockworm/blob/main/.github/workflows/cd.yml)

## Summary
The opportunity to work with NGINX, LetsEncrypt, docker and docker-compose was fun, most of the time, and it is rewarding to see the site load nearly instantly. Despite the time investment, a higher level container or app solution is still appealing. As of now, the plan is to iterate the current setup and arrive at that single command awesomeness.
