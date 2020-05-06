---
layout: post
title: freeclimb_quick_conference
---

Let's setup a phone call with two people on [FreeClimb](www.freeclimb.com). We'll do this as quickly and easily as possible. In addition to the resources we used to receive the call we'll need:
1. A FreeClimb Conference
2. Two phone calls

## Create FreeClimb Conference
FreeClimb conferences are persistent so you can re-use them as needed. This also means that conferences can be created before your phone calls arrive. With that in mind, let's create a conference using curl.

First lets checkout the reference docs to [create a conference](https://docs.freeclimb.com/reference/conferences#create-a-conference).  The curl in the example provided is:

```bash
curl -X POST "https://{api_url}/Accounts/{account_id}/Conferences" -d "{\"alias\":\"test\", \"playBeep\":\"always\", \"record\":false, \"waitUrl\":\"http://myapp/waitaudio\", \"statusCallbackUrl\":\"http://myserver/confstatuscallback\"}" -u "{account_id}:{auth_token}"
```

When working with a command of this length I'll put the string into a text editor, make adjustments, and copy and paste to the terminal. I find that easier than trying to edit the command in the terminal itself.

There are three keys in the curl command you will need to replace:
1. api_url => replace with www.freeclimb.com/apiserver
1. account_id => Go to the dashboard, replace with your account_id (two spots!)
1. auth_token => Go to the dashboard, replace with your auth_token

| Key         | Replace with                                                             |
| ------------|--------------------------------------------------------------------------|
| api_url     | replace with www.freeclimb.com/apiserver                                 |
| account_id  | Go to the dashboard, replace with your account_id (**two replacements!**)|
| auth_token  | Go to the dashboard, replace with your auth_token                        |  

When making your replacements, remove the curly braces. The only curly braces in the command should be associated with the data parameter (-d). Also, didn't want include a wait time are status url for the quick example so removed those.

Your curl command should looks something like this after completing your adjustments.

```bash
curl -X POST "https://www.freeclimb.com/apiserver/Accounts/AC5asdf345321421442412sfsdvsgd/Conferences" -d "{\"alias\":\"Jakes test Conference\", \"playBeep\":\"always\", \"record\":false }" -u "AC5asdf345321421442412sfsdvsgd:bsdf214sadsfasfdsfasfasfasffht"
```

You should get a conferenceId returned in the response. We're going to need that for our next step.

## Handle Inbound Calls
This portion builds off of the previous blog post Receiving a [Phone Call from FreeClimb](https://jakeschmitz.com/blog/ruby/rails/freeclimb/2020/04/15/inbound-freeclimb-call.html). We left off there using PerCl to say hello to the caller and hanging up. We'll add to that by placing all two callers into a conference using the PerCl [addToConference](https://docs.freeclimb.com/reference/conference-participants-1#addtoconference) command.

The most minimal code in your controller would be:
```ruby
script = Percl::Script.new

addToConference = Percl::AddToConference.new("conferenceId_from_your_curl", params[:callId])
script.add(addToConference)

render json: script.to_json
```

Start up your server and try out a couple of phone calls. The calls will connect and folks will be able to start talking to each other.

## Conclusion
You've created a conference call that is available, pretty cool and also something you probably don't want to keep running on your account :) Now that you have two calls connected you can use this and other building blocks of FreeClimb and other services to make your app more awesome.
