---
layout: post
title: "Productivity: Ngrok subdomain replace"
date: 2020-06-04 06:28 -0500
---

The mornings I’m working with FreeClimb at some point in my testing an ngrok connection is needed to be set up for my local test environment. This results in creating new ngrok tunnel, updating my FreeClimb webhook urls on my dashboard and then updating my local app environment variables used for callbacks from FreeClimb. This procedure has been executed more than twice so being a good lazy developer I wrote a little script to expedite the procedure and thought other folks may find it useful.

The script is posted as a [public gist](https://gist.github.com/jschmitz/df4e1f33fd19f05726ed5c909e013d5c/edit) and, if its helpful, please modify and use as needed. The script, as written, requires a specific setup that I'll describe.

## Ngrok
The script expects you to be using Basic Auth with your ngrok connection.

```bash
ngrok http --auth user:pass 3000
```

You are creating a tunnel into your machine so you might as well not be the least secure person out there. Also, I explicitly severe the connection when I am completed with my testing.

## Setup Environment Variables
The script needs the following environment variables created before execution:

Environment Variable  | Description
------------- | -------------
FREECLIMB_ACCOUNT_ID  | Your FreeClimb account id. Obtain from your FreeClimb Dashboard. This is used in the curl request to FreeClimb.
FREECLIMB_ACCOUNT_TOKEN  | Your FreeClimb account id. Obtain from your FreeClimb Dashboard. This is used in the curl request to FreeClimb.
YOUR_APP_PUBLIC_URL | This is the public url ngrok provides. The format must be: 'http://username:password@subdomain.ngrok.io"
YOUR_APP_APPLICATION_ID | Your FreeClimb application id. Obtain from your FreeClimb Dashboard. This is used in the curl request to FreeClimb.

You can name these anything, update the script to match your names.

## Update FreeClimb Webhooks
The script currently [updates your FreeClimb Application](https://docs.freeclimb.com/reference/applications#update-an-application) voiceURL ([inbound call webhook](https://docs.freeclimb.com/reference/calls-2#inbound)) and smsUrl ([messageDelivery webhook](https://docs.freeclimb.com/reference/sms-2#messagedelivery-1)). You may need to add or remove FreeClimb Application attributes being updated to fit your applications needs.

```bash
curl -s -X POST -d "{\"voiceUrl\":\"$new_public_url/inbound_calls\", \"smsUrl\":\"$new_public_url/inbound_messages\"}" -u $KMS_CONF_USERNAME:$KMS_CONF_PASSWORD https://freeclimb.com/apiserver/Accounts/$KMS_CONF_USERNAME/Applications/$KMS_CONF_APPLICATION_ID
```

## Execution
* Set your permissions to execute (see the first reference link)
* Pass in the subdomain

Time to run the script. I copy the subdomain from the ngrok terminal and paste to pass the new subdomain to the script:

```bash
./ngork_update newNgrokSubdomainPasted
```

And done! You’re all set to go to start making calls and sending messages to your local test environment.

## References
I don’t make Bash Scripts on a daily basis so I found the need to google a bunch of stuff. The list of resources I used is below and these could help you further modify and extend the example script.
* https://www.taniarascia.com/how-to-create-and-use-bash-scripts/
* https://www.lifewire.com/pass-arguments-to-bash-script-2200571
* https://stackoverflow.com/questions/18547881/shell-script-to-set-environment-variables
* https://linuxhint.com/curl_bash_examples/
* https://stackoverflow.com/questions/13210880/replace-one-substring-for-another-string-in-shell-script
* https://stackoverflow.com/questions/9741433/appending-a-line-break-to-an-output-file-in-a-shell-script
