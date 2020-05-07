---
layout: post
title: freeclimb_quick_conference
---

Let's setup a conference call with multiple callers on [FreeClimb](www.freeclimb.com). We'll start by creating the most minimal conference we can, bells and whistles can be added from there. A prerequisite is [](https://jakeschmitz.com/blog/ruby/rails/freeclimb/2020/04/15/inbound-freeclimb-call.html), if you haven't completed the steps in that post you'll need to in order to complete a conference setup. The additional resources needed to complete the steps in this post will be:
1. A FreeClimb Conference
2. Two phone calls

## Create FreeClimb Conference
FreeClimb conferences are persistent so you can re-use them as needed. This means that conferences can be created before phone calls arrive to your conference arrive. We will use the FreeClimb API to create a conference using curl.

First, lets checkout the reference docs to [create a conference](https://docs.freeclimb.com/reference/conferences#create-a-conference).  The curl in the example provided is:

```bash
curl -X POST "https://{api_url}/Accounts/{account_id}/Conferences" -d "{\"alias\":\"test\", \"playBeep\":\"always\", \"record\":false, \"waitUrl\":\"http://myapp/waitaudio\", \"statusCallbackUrl\":\"http://myserver/confstatuscallback\"}" -u "{account_id}:{auth_token}"
```

When working with a command of this length I'll put the string into a text editor, make adjustments, and copy and paste to the terminal. I find that easier than trying to edit the command in the terminal itself.

There are three keys in the curl command you will need to replace:


| Key         | Replace with                                                             |
| ------------|--------------------------------------------------------------------------|
| api_url     | replace with www.freeclimb.com/apiserver                                 |
| account_id  | Go to the dashboard, replace with your account_id (**two replacements!**)|
| auth_token  | Go to the dashboard, replace with your auth_token                        |  

When making your replacements, remove the curly braces. The only curly braces in the command should be associated with the data parameter (-d). If you leave them in with auth portion, after -u, you will receive *Unauthorized Error* responses.  The waitUrl and statusCallbackUrl are also removed in the example below for simplicity.

Your curl command should looks something like this after completing your adjustments:

```bash
curl -X POST "https://www.freeclimb.com/apiserver/Accounts/AC5asdf345321421442412sfsdvsgd/Conferences" -d "{\"alias\":\"Jakes test Conference\", \"playBeep\":\"always\", \"record\":false }" -u "AC5asdf345321421442412sfsdvsgd:bsdf214sadsfasfdsfasfasfasffht"
```

You should get a *conferenceId* returned in the response. We're going to need that for our next step.

## Handle Inbound Calls
This portion builds off of the previous blog post Receiving a [Phone Call from FreeClimb](https://jakeschmitz.com/blog/ruby/rails/freeclimb/2020/04/15/inbound-freeclimb-call.html). If you haven't check that post out, you'll need to get this to your phone calls connecting. The next update will be to the InboundCallsController. We're going to use the *conferenceId* we just created and use that to connect phone calls using the PerCl [addToConference](https://docs.freeclimb.com/reference/conference-participants-1#addtoconference) command. Start up your server and confirm your "Hello World" app is working. Then, lets make a couple changes:

The most minimal code in your controller would be:
```ruby
class InboundCallsController < ApplicationController

  def create
    script = Percl::Script.new

    addToConference = Percl::AddToConference.new("conferenceId_from_your_curl", params[:callId])
    script.add(addToConference)

    render json: script.to_json
  end
end
```

Ensure your server is running and try out a couple of phone calls. The first call you're going to get connected and hear silence, its a bit strange. The second call will trigger a beep in the conference and you'll be connected and the two calls can chats.

## Extra! - getting rid of the creepy silence

First a bit of a greeting goes along way. You can add a greeting into your PerCl response
```ruby
    say = Percl::Say.new("Hello, you're about to be added to a conference.")
    script.add(say)
```
The string you provide will be read out, Text To Speech (TTS), to the caller upon arrival.

Next, you can add folks into what many conferencing folks would call the *lobby* by setting the startConfOnEnter attribute to false.
```ruby
    addToConference = Percl::AddToConference.new(conference_id, call_id)
    addToConference.startConfOnEnter = false
    script.add(addToConference)
```
If the conference is not started and *startConfOnEnter* is set to false, callers will hear hold music. When placing callers into the lobby you'll have to determine what you want to trigger a conference to start and add that logic. Perhaps a specific phone number calling in triggers the conference to start or a PIN being entered (checkout getDigits!)?  Up to you.


## Conclusion
You've created a conference call that is available to anyone that calls that number, pretty cool and also something you probably don't want to keep running on your account :) Once the calls you have calls connecting try getting conferences created dynamically using the API and using startConfOnEnter to provide folks with hold music. You can keep adding building blocks of FreeClimb API along with other API services to make your app more and more awesome.
