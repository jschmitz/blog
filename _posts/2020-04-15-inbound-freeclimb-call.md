---
layout: post
title:  "Receiving a phone call from FreeClimb"
date:   2020-04-15 06:04:34 -0500
categories:  [ruby, rails, FreeClimb]
tags: [inbound, phone call]
---

# Receiving a phone call from FreeClimb

This post walks through the set up of an application that will answer a phone call and say 'Hello World' to the caller using [FreeClimb](https://www.freeclimb.com/).



### Communication Flow
There are four things involved in the communication flow:
1. A Phone
1. [FreeClimb](https://www.freeclimb.com/)
1. [Ngrok](https://ngrok.com/)
1. Your local web applications (a Rails app in this case)



### Next

The communication flow is:
1. User makes a phone call to a FreeClimb phone number
1. FreeClimb answers that phone call
1. FreeClimb sends a HTTP Request to a NGROK url
1. NGROK proxies the HTTP Request to your local web server
1. The local web server responds with PerCL commands to _Say Hello World_ and _Hangup_
1. FreeClimb receives the HTTP Response command _Say_ and _Hangup_
1. FreeClimb says Hello World to the caller and hangs up

Sequence diagrams help me out quite a bit. Here's what this would look like:

![Incoming Call Sequence Diagram]({{site.url}}/blog/assets/diagrams/incoming_call.svg)

### Build Plan
To build the application and all the communication plumbing we'll proceed in the following order:
1. Create a local Rails API app first that will receive an HTTP Request
1. Configure FreeClimb
1. Configure NGROK
1. Use the FreeClimb SDK to talk back to FreeClimb in our Rails App

## Rails API App
Spin up a Rails API App. This will receive the HTTP request FreeClimb when a phone call arrives.
```bash
rails new kms_conferencing --api
```
The plan is to be very targeted so I'll create what I need mostly by hand.

Setup a route to a controller that will receive the HTTP Request when a phone call arrives.  Go to the config/routes.rb and add the following:
```ruby
Rails.application.routes.draw do
  post 'inbound_calls', to: 'inbound_calls#create'
end
```

Add a controller _inbound_calls_controller.rb_
```ruby
class InboundCallsController < ApplicationController

  def create
    puts 'recieved a call'
  end
end
```

Fire up your app locally
```bash
rails s
```

Test the endpoint you just made using curl
```bash
curl -d '{"key1":"value1", "key2":"value2"}' -H "Content-Type: application/json" -X POST http://localhost:3000/inbound_calls
```

Your local log output should look something like this
```bash
Started POST "/inbound_calls" for ::1 at 2020-03-21 08:20:50 -0500
Processing by InboundCallsController#create as */*
  Parameters: {"key1"=>"value1", "key2"=>"value2", "inbound_call"=>{"key1"=>"value1", "key2"=>"value2"}}
recieved a call
Completed 204 No Content in 0ms (ActiveRecord: 0.0ms | Allocations: 31)
````

Alrighty, your local web server should be setup for now. Let's get FreeClimb setup to send you an HTTP Request and we'll circle back to the app after FreeClimb can talk to it.


## FreeClimb
This is all setup no code, unless you want to use the API... In this case, I'm using the dashboard.

The Getting Started and SDK examples are the best reference to get going with FreeClimb. I'd recommend using those for detailed instructions. At a high level do the following with FreeClimb using the dashboard (or you can do this with the API too) to complete the following:
1. Get a _FreeClimb phone number_
2. Create an _Application_
3. Set your _voice_url_ on the _Application_ to point at your NGROK url (read the next paragraph, you need ngrok for this)

Once you have a phone number and an application assigned to that phone number lets make a call.  You should hear an error message and if you go to your error logs in your dashboard you should see an error message that says something like you need to add valid HTTP Endpoint. If you add your localhost... not gonna work, its not publicly accessible for FreeClimb to talk to. This is where we're going to use ngrok to open up that local server.

## NGROK
This is for our local development and helping us maintain a fast feedback cycle as we build. NGROK will expose a public url that FreeClimb can send an HTTP Request and NGROK will proxy the request to my local application serve. In this case a rails api. You can download their zip file to install or I used brew:

```bash
brew cask install ngrok
```

Set up a password protected tunnel
```bash
ngrok http -auth "user:password" 3000
```

Once you have your tunnel setup you can see and replay requests:
```bash
http://localhost:4040/inspect/http
```

This is awesome because you can now use the replay repeat your HTTP requests. In this case, instead of making phone calls to test your app you can click replay and trigger HTTP requests. In the long run setting up tests is the way to go and the replay button can get you jump started.

With your tunnel setup, circle back to the FreeClimb application dashboard. Set the _FreeClimb Application Voice_ to be
```bash
http://[user]:[password]@[your_ngrok_url]/inbound_calls
```
Thats good bit of substitution to take care of. Take your time and get it right :)

If you make a call now you'll likely see NGROK telling you something like

```bash
403 /inbound_calls FORBIDDEN
```

To fix this, circle back to your rails app. You need to whitelist ngrok urls and can so in the config/environments/development.rb file:
```ruby
   #Allow ngrok to contact application
  config.hosts << /[a-z0-9]+\.ngrok\.io/
```

After all of that jumping around, you have a pretty good chance of having a phone call trigger the request on your server! Pretty awesome.

## Back to your Rails App; Tell FreeClimb what to do
FreeClimb needs JSON sent back on the HTTP Response. We could form our own or use the FreeClimb SDK to help us out. Let get that installed by adding the FreeClimb gem. Related instructions can be found here: [FreeClimb Ruby Install](https://docs.freeclimb.com/docs/getting-started-with-ruby)

### Add the FreeClimb Gem

 ```bash
 gem 'freeclimb', '~> 1.0'
 ```

### Tell FreeClimb to play a message and hangup

```ruby
class InboundCallsController < ApplicationController

  def create
    say = Percl::Say.new('Hello World. Good bye.')
    hangup = Percl::Hangup.new
    script = Percl::Script.new
    script.add(say)
    script.add(hangup)

    render json: script.to_json
  end
end
```

## Summary
Give the FreeClimb phone number a call. We should hear a _Hello World_ and see the logs in NGROK and our local Rails application! We're all setup to proceed adding more awesomeness to the app.
