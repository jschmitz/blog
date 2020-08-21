---
layout: post
title: "FreeClimb: Conferences and Queues"
---

The [_Conference_](https://docs.freeclimb.com/reference/conferences) and [_Queue_](https://docs.freeclimb.com/reference/call-queues) resources in FreeClimb are the backbone of servicing phone calls with FreeClimb. The _Conference_ and _Queue_ resources have a several of unique behaviors that are of interest and will this post will describe each in resource in more detail. First, a bit about of background.

## Background
When receiving and inbound call you will more than likely want to put that call into a queue to enable you to do something interesting with that call.  The _Queue_ resource will hold that call and continuously call your application back on a [webhook](https://docs.freeclimb.com/reference/queuewait) that enables you to take action with that call when your application becomes aware of an interesting event. For example, in a basic conferencing scenario a call comes in, is put in a queue, the moderator arrives (interesting event) and the original is dequeued and put into a conference.

The _Conference_ resource is used to connect calls. This does not need to be a traditional conference call with numerous persons attending. The Conference resource is used to connect any two calls for two persons to speak with each other. Additionally, the Conference resource can also be used to host conferences with numerous people attending.

## Conference Resource

The [_Conference_ resource](https://docs.freeclimb.com/reference/conferences) is:
* created using the [API](https://docs.freeclimb.com/reference/conferences#create-a-conference) before a call arrives
* created using [PerCL](https://docs.freeclimb.com/reference/createconference) with a call in progress
* has a Conference Id assigned by FreeClimb upon resource creation (both API & PerCL methods)
* are persisted
  * Can be re-used (populated or in progress --> empty status does not delete)
  * Terminated conferences are NOT permanently deleted from the system

In your app, before you can put a call into a conference it must be created and you can

### Save the Conference Id

In a basic conferencing system scenario the conference id may be assigned to a user (moderator) upon user creation in a conferencing application. The conference id is created and assigned before a call arrives using the REST API

```ruby
  #using the FC Ruby SDK
  conference_id = @freeclimb.create_a_conference({
    create_conference_request: Freeclimb::CreateConferenceRequest.new,
  })

  #  
  User.create!(conference_id: conference_id, , conference_phone_number: new_conf_phone_number)
```
### Use the Conference

When subsequent calls arrive the conference id can be retrieved by the DNIS used to put calls to that DNIS into conference.

```ruby
  user = User.where(conference_phone_number: params[:to]).take
  addToConference = Freeclimb::AddToConference.new(conference_id: user.conference_id, call_id: params[:callId])
```

The conference id and the setting applied to that conference can be re-used many times.

## Queue Resource
The [_Queue_ resource](https://docs.freeclimb.com/reference/call-queues) is:

* created using the [API](https://docs.freeclimb.com/reference/call-queues#create-a-queue) before a call arrives
* created using [PerCL](https://docs.freeclimb.com/reference/enqueue-1)
* has a queue id assigned in two ways
  * the [PerCL Enqueue](https://docs.freeclimb.com/reference/enqueue-1), the queueId can be assigned by the developer
  * FreeClimb assigns and returns an id if not assigned by the developer
* is ephemeral. When a queue moves from a populated to empty status the _Queue_ IS permanently deleted

### Dynamic Queue ID Example

An interesting and helpful feature is dynamic queue id assignment. The developer has the opportunity to assign a queue id upon enqueuing a call. This results in the ability to use a convention to assign calls to queues and a. For example, the originally dialed telephone number, DNIS, can be used assign incoming calls to a queue.


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

### FreeClimb assigned Queue ID Example

The implementation for using the queue id that is assigned by FreeClimb will look very similar to a conference. Persist the queue id into your application using a DB or another storage mechanism and use that id to place calls into the specified queue.

There is a catch with this approach, a queue will be deleted from the system and you will need to detect the queue is not available as a developer. You can do this by retrieving the resource and when a 404 is returned you create a new queue and to place your calls into.



```ruby
def find_or_create_queue(queue_id)
if queue_id
  queue_id = create_queue.queue_id unless queue_available?(queue_id)
else
  queue_id = create_queue.queue_id
end

  queue_id
end

def create_queue
  opts = {
    queue_request: Freeclimb::QueueRequest.new, # QueueRequest | Queue details used to create a queue
  }

  @freeclimb.create_a_queue(opts)
rescue Freeclimb::ApiError => e
  puts "Exception when calling ->create_a_queue: #{e}"
end

def queue_available?(queue_id)
  @freeclimb.get_a_queue(queue_id)
  true
rescue Freeclimb::ApiError => e
  puts "Exception when calling ->get_a_queue: #{e}"
  false
end
```
