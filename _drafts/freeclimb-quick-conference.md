---
layout: post
title: freeclimb_quick_conference
---

Let's setup a conference that will enable multiple callers to start chatting. We'll start by creating the most minimal conference we can on [FreeClimb](www.freeclimb.com). A prerequisite is a previous post [Receiving a phone call from FreeClimb](https://jakeschmitz.com/blog/ruby/rails/freeclimb/2020/04/15/inbound-freeclimb-call.html), if you haven't completed the steps in that post you'll need to in order to complete setting up a conference. The resources needed to complete the steps in this post will be:
1. A FreeClimb Conference
1. Two phone calls

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

## Connect Your Callers
This portion builds off of the previous blog post Receiving a [Phone Call from FreeClimb](https://jakeschmitz.com/blog/ruby/rails/freeclimb/2020/04/15/inbound-freeclimb-call.html). If you haven't check that post out, you'll need to get this to your phone calls connecting. You have a phone number available and that phone number can receive many phone calls. The code we'll add below will put all callers that call your number into a conference and enable those callers to talk to each other.

The next update will be to the InboundCallsController that you've already created. We're going to use the *conferenceId* we previously to host a conference. To do so, we'll use PerCL, its JSON commands under the hood, to tell FreeClimb to add all calls received to the conference resource. The PerCL command to use is [addToConference](https://docs.freeclimb.com/reference/conference-participants-1#addtoconference) and that will be added to your HTTP Response back to FreeClimb. Start up your server and confirm your "Hello World" app is working. Then, update the response your providing:

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

Ensure your server is running and try out a couple of phone calls. The first call you're going to get connected and hear silence, its a bit strange. The second call will trigger a beep in the conference and you'll be connected and the two callers can chats. Success!

## Extra! - Remove the strange initial silence

First a bit of a greeting goes along way. You can add a greeting into your PerCl response
```ruby
    say = Percl::Say.new("Hello, you're about to be added to a conference.")
    script.add(say)
```
The string you provide will be read out, Text To Speech (TTS), to the caller upon arrival. After the greeting the caller will experience silence until another caller arrives. We can keep improving.

Next, we can add hold music for the intial caller(s). We can add folks into what many conferencing folks would call the *lobby* by setting the startConfOnEnter attribute to false.
```ruby
    addToConference = Percl::AddToConference.new(conference_id, call_id)
    addToConference.startConfOnEnter = false
    script.add(addToConference)
```
If the conference is not started and *startConfOnEnter* is set to false, callers will hear hold music. When placing callers into the lobby you'll have to determine what you want to trigger a conference to start and add that logic. Perhaps a specific phone number calling in triggers the conference to start or a PIN being entered (checkout getDigits!)?  Up to you.


## Conclusion
You've created a conference call that is available to anyone that calls that number, pretty cool and also something you probably don't want to keep running for the world :) Next try getting conferences created dynamically using the API and use your own logic to get folks into the lobby and then start the conference. You can keep adding building blocks of FreeClimb API along with other API services to make your app more and more awesome.
