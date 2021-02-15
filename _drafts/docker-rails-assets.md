---
layout: post
title: Docker, Rails, NGINX and Assets
---

Assets, [NGINX](https://nginx.org/en/), and the [Rails Asset Pipeline](https://guides.rubyonrails.org/asset_pipeline.html) have the potential to cause a bit of frustration and adding Docker to the mix things can add an extra element of trickiness. This post will walk you through the configurations that will run production Rails application in a production Docker environment, precompile your assets, and serve the precompiled assets using NGINX.

# Build for Production Environment

A simple but easy step to omit to get started is to ensure your app will be configured to run in a production environment.

In your docker-compose.yml, ensure the application environment is set to production.
```bash
environment:
  - RAILS_ENV=production
```

# Precompile assets
The Rails Asset Pipeline is doing a good deal of work for you to help deliver the assets on your site more efficiently. Your assets can be precompiled to production and to do so add the following to your Dockerfile in your root directory.

```bash
# Dockerfile
RUN RAILS_ENV=production rails assets:precompile
```

On occasion, it seems that the precompile step does not work as expected and to fix that situation removing the existing assets and recompiling seems to fix any issue.

```bash
rm public/assets/*
rm -r public/packs/*
```

# Server the assets from NGINX

Reverse proxies, such as NGINX, are built to serve and cache your assets much more efficiently than your application. In production, this is the default and you should only need to configure NGINX in config/environments/production.rb and has a nice messaging letting you know why.

```bash
# config/environments/production.rb

# Disable serving static files from the `/public` folder by default since
# Apache or NGINX already handles this.
config.public_file_server.enabled = ENV['RAILS_SERVE_STATIC_FILES'].present?
```

The following snippet shows handling of routing for your application assets.

```bash
upstream your-app {
  server web:3000;
}

location ~ ^/(assets|packs|images|javascripts|stylesheets|swfs|system)/ {
    try_files $uri @your-app;
    access_log off;
    gzip_static on;

     # to serve pre-gzipped version
    expires max;
    add_header Cache-Control public;

    add_header Last-Modified "";
    add_header ETag "";
    break;
}
```

# Check It Out!
The application and NGINX are ready to server your assets. Use docker compose build and then start the processes for your application.

```bash
docker-compose build
docker-compose up
```

Using the domain you have configured check out your app with your assets served from NGINX!

# Extra - Troubleshooting

## Checking production in Docker
When you build your app and encounter the generic Rails error message page it may be because an asset was not found. The procedure you can use to check what the error is, asset related or otherwise, is to access the running container and check the log.

```bash
docker exec -it image_name /bin/bash
```

```bash
cat log/production.log
```

## Testing with a non docker NGINX
If you are testing locally before getting Docker into the mix a useful command to reload NGINX:

```bash
sudo nginx -t && sudo service nginx reload
```

The nginx -t command verifies you have valid configs and if you do the second command reload nginx with those valid configs.
