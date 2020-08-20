---
layout: post
title: "FreeClimb: Conferences and Queues"
---

The _Conference_ and _Queue_ resources in FreeClimb are the backbone of making phone calls into FreeClimb. They do not behave quite the same and we can explore those differences in this post. First, a bit about of background.

## Background
When receiving and inbound call you will more than likely want to put that call into a queue to enable you to do something interesting with that call.  The _Queue_ resource will hold that call and continuously call your application back on a [webhook](https://docs.freeclimb.com/reference/queuewait) that enables you to take action with that call when your application becomes aware of an interesting event. For example, in a basic conferencing scenario a call comes in, is put in a queue, the moderator arrives (interesting event) and the original is dequeued and put into a conference.

## Conference Resource

The [_Conference_ resource](https://docs.freeclimb.com/reference/conferences) is:
* created before use with the [API](https://docs.freeclimb.com/reference/conferences#create-a-conference) or [PerCL](https://docs.freeclimb.com/reference/createconference)
* conference id is assigned by FreeClimb upon creation
* persisted - terminated conferences are NOT permanently deleted from the system

In your app, before you can put a call into a conference it must be created and you can

### Persist the Conference

In a basic conferencing system scenario the conference id may be assigned to a user (moderator) upon creation in a conferencing application.

```ruby
  #using the FC Ruby SDK
  conference_id = @freeclimb.create_a_conference({
    create_conference_request: Freeclimb::CreateConferenceRequest.new,
  })

  #  
  User.create!(conference_id: conference_id, , conference_phone_number: new_conf_phone_number)
```

When the call arrives the conference id can be retrieved by the DNIS used to put calls to that DNIS into the a conference call.

```ruby
  user = User.where(conference_phone_number: params[:to]).take
  addToConference = Freeclimb::AddToConference.new(conference_id: user.conference_id, call_id: params[:callId])
```

## Queue Resource
The [_Queue_ resource](https://docs.freeclimb.com/reference/call-queues) is:
* created using the [API](https://docs.freeclimb.com/reference/call-queues#create-a-queue) or [PerCL](https://docs.freeclimb.com/reference/enqueue-1)
* using the [PerCL Enqueue](https://docs.freeclimb.com/reference/enqueue-1), the queueId can be assigned by the developer
* ephemeral - when a queue moves from a populated to empty status the _Queue_ IS permanently deleted

An interesting and helpful bit is that dynamic creation of a queue by assigning the queueId in the PerCL command. The developer has the opportunity to assign a queue id upon enqueuing a call. This results in the ability to use a convention to assign calls to queues. For example, the originally dialed telephone number, DNIS, can be used assign incoming calls to a queue.

### Dynamic Queue ID Example
```ruby
def queue_id(phone_number)
  "QUaaaaaaaaaaaaaaaaaaaaaaaaaaaaa#{phone_number[1..-1]}"
end
```

```ruby
enqueue = Freeclimb::Enqueue.new(action_url: queue_action_url,
                                     queue_id: queue_id(dialed_phone_number),
                                     wait_url: queue_wait_url)
```
