---
title: Message Basics
---

## Message Generation

The Cenit hub is responsible for processing and delivering [messages](terminology#messages) based on a pre-configured set of business logic specific to a particular store. The messages processed by the Hub are generated in one of three possible ways.

### Polling

The most common way for messages to enter the system is via a scheduled poller that talks to the Cenit [storefront API](/api). These pollers will ask the storefront for new messages that have been created or updated since the last time it checked.

It's not important to understand exactly how these API differences result in a message. For the purposes of this guide, it is sufficient for you to understand that changes to orders, payments, shipments, etc. come in via a scheduled polling action at regular intervals. The result of this will be a series of appropriate messages relevant to the state change.

For example, if the API poller comes back with an order that the hub is not already aware of, then this results in an `order:new` message. If the order already existed but the API poller was showing some kind of change (ex. a change in quantity for a line item) then an `order:update` message would be generated instead.

Finally, in some instances, an update to an order can result in a more specific message being generated. For instance, if the order status was changed from `complete` to `canceled` an `order:cancel` message would be produced instead of the more general `order:update`.

***
See the specific [Message](messages_overview) guides for more details on each of these and other message Types.
***

### Response

The second way for a message to enter the system is in response to a [service request](terminology#service_requests) to an [endpoint](terminology#endpoints). When an end point is processing a message, it will typically respond with a new message. That new message can in turn be processed by the Cenit hub.

Let's look at a specific example where we are using an integration that takes new shipments and dispatches them to a third party logistics provider (3PL) for drop shipping. The `process_shipment` service of this endpoint will return a `notification:info` message in response to this service request.

```ruby
post '/process_shipment' do
  # DO STUFF HERE
  process_result 200,
    { 'message_id' => @message[:message_id],
      'messages' => [
        { 'message' => 'notification:info',
          'payload' => {
            'subject' => 'Shipment transmitted',
            'description' => '#H123456 transmitted successfully.'
          }
        }
      ]
    }
end
```

In the above example, we returned a single message in the response, but messages generated in this way are technically an array of messages and so it is possible to generate more than one message as part of a service request. For example, if you were designing a service that returned the list of shipments that have shipped since the last check, then you would likely need to return multiple `shipment:confirm` messages.

***
See the [Creating Endpoints](creating_endpoints_tutorial) tutorial for some more detailed examples on how to generate messages in response to processing a service request.
***

### Push

Finally, it is possible to push messages into the hub from an external source.

$$$
Need more docs on message push.
$$$

***
Endpoints should never push Messages, since they are intended to be a passive consumer of Messages. Instead, they should be polled via a Message sent from the Hub, so that they can return the necessary information.
***

## Service Requests

Integration Endpoints expose various [services](terminology#services) to the Hub. The Hub is configured with a series of [mappings](terminology#mappings), which tell it how it route a specific Message to a particular Service offered by an Endpoint.

***
See the [Mapping Guide](mapping_basics) for more information on how Messages are mapped to Endpoints.
***

Passing a Message to a particular Endpoint Service is also referred to as making a [service request](terminology#service_request). Service Requests are always made use the `HTTP POST` method and the Messages they pass are required to be in a JSON format. Now let's take a look at some aspects of message delivery.

***
Integrations can be written in any language (not just Ruby). All that is required is that your Endpoint be able to respond to `HTTP POST` requests, and that your Service methods be capable of reading a JSON-formatted form parameter.
***

## Message Delivery

Messages routed through the Hub are guaranteed to be delivered to their intended endpoints. When attempting to deliver a message to a service (i.e. making a service request), one of three things can happen.

### Successful Delivery

If the message is delivered to the endpoint and it is processed without incident, then the message can be considered successfully delivered. The minimum requirement for an endpoint to indicate that the service request was successful is to return a `200 OK` response, along with the `message_id`.

The following is a simple example of an endpoint built using sinatra to return the bare minimum required to indicate successful processing of the message.

```ruby
post '/do_something' do
  message = JSON.parse(request.body.read)
  json 'message_id' => message['message_id']
end
```

Here's what the server sends back when a message is sent to the `do_something` service via `HTTP POST`:

```bash
HTTP/1.1 200 OK
Content-Type: application/json;charset=utf-8
Content-Length: 35
X-Content-Type-Options: nosniff
Server: WEBrick/1.3.1 (Ruby/1.9.3/2012-04-20)
Date: Tue, 02 Jul 2013 20:12:23 GMT
Connection: Keep-Alive

{"message_id":"518726r84910000001"}
```

***
See the [Creating Endpoints](creating_endpoints_tutorial) tutorial for more detailed examples of how to build simple endpoints.
***

There is, however, a more informative way to convey the successful delivery of a message. In addition to returning `200 OK` a service can return an array of messages (or just a single message). The previous discussion of [responses](#response) contains an example of how this can be done.

Responding with a [notification message](notification_messages) has the advantage of allowing the service to pass additional human-readable information back to the hub. By default, notification messages will be turned into [log entries](terminology#log_entries), which can be displayed in a detailed tab in the store's admin interface.

Sometimes, however, it's important to convey more than just "success" plus a log entry. A major advantage to using Cenit hub is that it allows one integration to respond to events taking place in another. In order for this to happen, however, the Service Request of the first integration needs to return something more specific in addition to a log message.

Let's look at a specific example where we want a particular integration to capture a previously authorized credit card payment once a shipment is ready to ship. Suppose further that we want to send an email message to the customer once we capture the payment on their card. Once we've taken care of the necessary mappings, we could use an endpoint with a `capture_payment` method.

---payment_integration.rb---
```ruby
post '/capture_payment' do
  # DO STUFF HERE
  process_result 200,
    { 'message_id' => @message[:message_id],
      'messages' => [
        { 'message' => 'notification:info',
          'payload' => {
            'subject' => 'Payment captured',
            'description' => 'Captured: 00000111122223333'
          }
        },
        { 'message' => 'payment:capture',
          'payload' => {
            'payment' => {
              'payment_id' => '12345567890',
              'amount' => '19.99',
              'currency' => 'USD'
            },
            'auth_code' => '00000111122223333',
            'timestamp' => '1969-07-21 T 02:56 UTC'
          }
        }
      ]
    }
end
```

In this case we've actually returned multiple messages. We return a `notification:info` message so that can be displayed on the events tab in the Cenit storefront, but we also return a `payment:capture` message. The idea here is that another integration can then listen specifically for `payment:capture` messages and do something specific knowing that a payment has been captured (update Quickbooks, send an email to the customer, etc.)

***
Note that services are not required to return a pre-defined message Type. You are free to create your own endpoints that return custom message types.
***

### Error on Delivery

There are various error conditions that may be encountered during the processing of a service request. Endpoints can have logic that maps to either internal business rules or the rules/validation logic of a third party API that it's facilitating integration with.

Let's look at a specific example. Suppose you are working with a third party logistics (3PL) firm to drop ship your packages to your customers. Suppose further that this 3PL has an API that checks to verify that it can deliver to the requested address before it accepts your ship instructions. When this happens, you will most likely want to return a [notification error](notification_messages#error) message.

The `notification:error` message is used to signify that there was a problem with processing that service request. This approach should be used when the nature of the problem is that the request is not consistent with some type of business logic, or the rules of the third party API that you're communicating with are not met.

---validation_failure_example.rb---
```ruby
post '/ship_package' do
  # AFTER 3PL TELLS US CAN'T DELIVER
  process_result 200,
    { 'message_id' => @message[:message_id],
      'messages' => [
        { 'message' => 'notification:error',
          'payload' => {
            'subject' => 'Can't deliver to that address',
            'description' => 'Afghanistan is not somewhere we ship to!'
          }
        }
      ]
    }
end
```

***
Use a notification error when the problem is a "validation" type issue or some other problem with a third party API where it does not make sense to reattempt the service request.
***

!!!
Error conditions should always be returned with status code `200 OK` even though there was technically a problem. They are distinct from [failures](terminology#failures) which are returned with `HTTP 5XX` error codes.
!!!

### Failed Delivery

There are times where processing a service request results in an exceptional condition. These are situations where it makes sense to continue to retry the service request in the hopes that the sitaution has been resolved. In these situations it is appropriate to treat this service request as failed.

#### How Failures are Generated

Endpoints can sometimes encounter an exception when trying to communicate with a third-party service through their API (ex. Shipwire is down for maintenance.) Typically these situations will result in exceptions. In these instances there is typically no need to rescue the exception, you can just allow it to result in a `5XX Error` response and the Cenit hub will treat the request has failed.

If the third party API is recuing exceptions or reporting permanent failure type situations as `200 OK` you may need to examine the respone more carefully and then raise your own exception (or just return `500 Internal Server Error` and supply your own error message.)

#### How Failures are Handled

The Cenit hub will treat a failure as extraordinary condition. It will continue to attempt to redeliver the message until it is successful. This is extremely useful when third party services experience temporary service disruptions since no messages are ever lost during the outage.

***
Our hub uses an [exponential backoff](http://en.wikipedia.org/wiki/Exponential_backoff) algorithm to gradually increase the amount of time between retries.
***

Failure conditions in a live production environment are also monitored by the Cenit staff. As part of your paid Cenit plan they will take action to help rectify the situation.

## Message Parameters

Integration endpoints are typically stateless, meaning that they don't retain information from one service request to the next. This allows the same endpoint to serve more than one store and to function in more than one environment at a time (ex. staging and production.) This poses a challenge, however, since each storefront may have its own API keys or other configuration inforamtion needed to talk to a third party service.

[Parameters](terminology#parameters) serve as a mechanism for transmitting this type of information to the endpoint. They are configured on a per store basis using the mapping tool (see the [Mapping Configuration](mapping_configuration)) guide for more details.

Once configured the hub will send the parameters for a particular mapping upon each request.

---order_cancel.json---
```ruby
{
  "message": "order:cancel",
  "payload": {
    "order": {
    },
    "parameters": [
      {
        "name": "api_key",
        "value": "1234567890"
      }
    ]
  }
}
```

Message parameters can also contain lookup data. In addition to passing API type parameters there are usecases where parameters can provide lookup data needed to bridge two different systems. For instance, if you are importing orders into your Spree store you may need to translate the shipping method names Amazon uses into the ones used by your Spree store.  The following is an example of the type of message and parameters you could use to accomplish this.

---order_import.json---
```ruby
{
  "message": "order:import",
  "payload": {
    "order": {
    },
    "parameters": [
      {
        "name": "amazon_login",
        "value": "foo@example.com"
      },
      {
        "name": "amazon_secret",
        "value": "1234567890"
      },
      {
        "name": "shipping_method_lookup",
        "value": [
          {
            "standard": "UPS Next Day",
            "amazon": "Express Next Day",
          },
          {
            "standard": "USPS Priority",
            "amazon": "Saver",
          }
        ]
      }
    ]
  }
}
```
