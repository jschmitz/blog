---
layout: post
title: Set up Google Credentials for your Heroku Application
date: 2020-09-16 05:52 -0500
---
When using [Google Cloud](https://cloud.google.com/) with your application running [Heroku](https://www.heroku.com/) there is a small challenge regarding storing Google Credentials. Google Cloud provides you with a google-credentials.json file that contains your Google connection information. When developing locally, an environment variable points at the file and when using Heroku pointing an environment variable directly at a file is not an option. Not to worry, the work around steps are below.

The following steps can be taken to add your Google Credentials to Heroku:
1. Add the [Google Credentials Buildpack](https://elements.heroku.com/buildpacks/buyersight/heroku-google-application-credentials-buildpack) to your Heroku app
1. Set a *GOOGLE_CREDENTIALS* and *GOOGLE_APPLICATION_CREDENTIALS* environment variables for your Heroku app
1. Redeploy your application to Heroku

## Heroku Google Credentials Buildpacks
First, add a Heroku build pack to your project. There were a few buildpacks that were available that appeared to do the same thing and my first choice did not work. This one did:
* https://elements.heroku.com/buildpacks/buyersight/heroku-google-application-credentials-buildpack

To install the buildpack in your project you can do the following:
```bash
heroku buildpacks:set https://github.com/buyersight/heroku-google-application-credentials-buildpack.git -a your-app-name
```
## Set Heroku Environment Variables
After installing the buildpack is added to your project set the following environment variables.

```bash
heroku config:set GOOGLE_CREDENTIALS='<copy & paste the json contents of your google-credentials.json here'
heroku config:set GOOGLE_APPLICATION_CREDENTIALS='google-credentials.json'
```

## Push a New Release
Push another deploy to Heroku and the buildpack will be added enabling your Google Cloud service.
