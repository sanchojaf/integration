---
title: SMTP Endpoint
---

## Overview

Send plain text email via SMTP (GMail, Sendgrid, etc.) using notification messages as the input.

+++
The source code for the [SMTP Endpoint](https://github.com/spree/smtp_endpoint/) is available on Github.
+++

## Services

### send

Receives notification messages (info, warn and error) and sends the notification to the configured recipients.

The subject of the mail will always be the same as the notification subject, with the level (info, warn or error) as an suffix in brackets. for example: ```[WARN] Disk space is low```

The body of the email will be the description posted with the notification message and will be send in plain text format.

We use the [pony](https://github.com/benprew/pony) gem internally and always configure pony to use smtp as the transport method. Also see [pony docs](https://github.com/benprew/pony#transport) for additional information.

#### Parameters

|key | value |
|----|-------|
|```smtp.from```| The from address |
|```smtp.to```| The recipient address, add multiple addresses separated by comma |
|```smtp.cc```| The CC address, add multiple addresses separated by comma|
|```smtp.bcc```| The BCC address, add multiple addresses separated by comma|
|```smtp.options```| the configuration for the smtp ```via_options```|


####Request

---notification_info.json---
```json
{
  "store_id":"1234",
  "message_id":"abc",
  "message":"notification:info",
  "payload":{
    "subject":"This is an info notification",
    "description":"You just received an info notification, live long and prosper!",
    "parameters":[
      {
        "name":"smtp.from",
        "type":"string",
        "value":"noreply@spreecommerce.com"
      },
      {
        "name":"smtp.to",
        "type":"string",
        "value":"spree@example.com"
      },
      {
        "name":"smtp.cc",
        "type":"string",
        "value":"spree+cc@example.com"
      },
      {
        "name":"smtp.bcc",
        "type":"string",
        "value":"spree+bcc@example.com"
      },
      {
        "name":"smtp.options",
        "type":"list",
        "value":[
          {
            "address":"smtp.mandrillapp.com",
            "port":"587",
            "user_name":"peter@spreecommerce.com",
            "password":"p4ssw0rd!",
            "enable_starttls_auto":"true",
            "authentication":"login",
            "domain":"spreecommerce.com"
          }
        ]
      }
    ]
  }
}```

#### Response

---success_response.json---
```json
{
  "message_id":"abc",
  "messages":[
    {
      "message":"email:sent",
      "payload":{},
    }
  ]
}
```

When an exception occures when sending the email we will catch the exception and send an error_response. Any exception that comes up will end up in the error_response. That way it's easy to find out what happend. For convenience we also send back all the parameters used while the error occured.

---error_response.json---
```json
{
  "message_id":"abc",
  "payload":{
    "parameters":[
      {
        "name":"smtp.from",
        "type":"string",
        "value":"noreply@spreecommerce.com"
      },
      {
        "name":"smtp.to",
        "type":"string",
        "value":"spree@example.com"
      },
      {
        "name":"smtp.cc",
        "type":"string",
        "value":"spree+cc@example.com"
      },
      {
        "name":"smtp.bcc",
        "type":"string",
        "value":"spree+bcc@example.com"
      },
      {
        "name":"smtp.options",
        "type":"list",
        "value":[
          {
            "address":"smtp.mandrillapp.com",
            "port":"587",
            "user_name":"peter@spreecommerce.com",
            "password":"p4ssw0rd!",
            "enable_starttls_auto":"true",
            "authentication":"login",
            "domain":"spreecommerce.com"
          }
        ]
      }
    ],
    "message":"MESSAGE INSPECT DATA"
  },
  "error":"error message",
  "backtrace":"error backtrace"
}
```
