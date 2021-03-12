---
layout: post
title: invalid_authenticity_token
---

# Problem
When sending the text message using an AJAX call the following error was occurring:

Processing by MessageController#create as */*
I, [2021-03-12T12:08:37.476025 #1]  INFO -- : [5ff6b38e-6f92-4226-a7e0-8d230454e385]   Parameters: {"message"=>"hello"}
W, [2021-03-12T12:08:37.476198 #1]  WARN -- : [5ff6b38e-6f92-4226-a7e0-8d230454e385] **HTTP Origin header (https://www.jakeschmitz.com) didn't match request.base_url (http://www.jakeschmitz.com)**
I, [2021-03-12T12:08:37.476519 #1]  INFO -- : [5ff6b38e-6f92-4226-a7e0-8d230454e385] Completed 422 Unprocessable Entity in 0ms (ActiveRecord: 0.0ms | Allocations: 113)
F, [2021-03-12T12:08:37.477477 #1] FATAL -- : [5ff6b38e-6f92-4226-a7e0-8d230454e385]   
[5ff6b38e-6f92-4226-a7e0-8d230454e385] ActionController::InvalidAuthenticityToken (ActionController::InvalidAuthenticityToken):
[5ff6b38e-6f92-4226-a7e0-8d230454e385]   
[5ff6b38e-6f92-4226-a7e0-8d230454e385] actionpack (6.0.3.2) lib/action_controller/metal/request_forgery_protection.rb:215:in `handle_unverified_request'
[5ff6b38e-6f92-4226-a7e0-8d230454e385] actionpack (6.0.3.2) lib/action_controller/metal/request_forgery_protection.rb:247:in `handle_unverified_request'
[5ff6b38e-6f92-4226-a7e0-8d230454e385] actionpack (6.0.3.2) lib/action_controller/metal/request_forgery_protection.rb:242:in `verify_authenticity_token'

# Solution
https://ramekintech.com/blog/the-invalid-authenticity-token-experiment

```bash
server_name jakescmitz.com;

proxy_set_header  X-Forwarded-Ssl on;
proxy_set_header  X-Forwarded-Host $host;
```
