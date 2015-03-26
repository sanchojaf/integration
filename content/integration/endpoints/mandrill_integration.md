---
title: Mandrill Endpoint
---

## Overview

[Mandrill](http://mandrill.com/) is a transactional email platform that you can use to automatically process your store's emails through the Spree Integrator. Mandrill could be called on any time you needed to, for example:

* confirm to a user that you have received their order,
* notify a user that their order was canceled, or
* send shipment confirmation to a customer.

## Requirements

In order to configure and use the [mandrill_endpoint](https://github.com/spree/mandrill_endpoint), you will need to first have:

* an account on Mandrill,
* an API token from that account, and
* templates set up in your Mandrill account

## Services

***
To see thorough detail on how a particular JSON Message should be formatted, check out the [Notification Messages guide](notification_messages).
***

### Templates

The Mandrill Integration sends a collection of Merge Variables which can be used in Mandrill templates to fill in relevant information. Example templates are included with each of the actions below, and full information about the variables being sent can be found in [the code](https://github.com/spree/mandrill_endpoint/blob/master/lib/mandrill_sender.rb).

### Order Confirmation

This Service should be triggered when an order is completed, or when an existing order is updated. When the Endpoint receives a validly-formatted Message to the `/order_confirmation` URL, it passes the order's information on to Mandrill's API. Mandrill then sends an email to the user using its matching stored [template](#template), confirming that their order was received.

<pre class="headers"><code>A new order is created</code></pre>
```json
{
  "message": "order:new",
  "payload": {
    "order": {
      ...
    }
  }
}
```

<pre class="headers"><code>An order is updated</code></pre>
```json
{
  "message": "order:updated",
  "payload": {
    "order": {
      ...
    }
  }
}
```

#### Parameters

| Name | Value | Example |
| :----| :-----| :------ |
| mandrill.api_key | Your Mandrill API Key | Aqws3958dhdjwb39 |
| mandrill.order_confirmation.from | Email Address to Send From | orders@spreecommerce.com |
| mandrill.order_confirmation.subject | Subject Line of Email | Your Order is Complete |
| mandrill.order_confirmation.template | Mandrill Template to Fill In | order_confirmation |


#### Example Template
```html
<h1>*|ORDER_NUMBER|*</h1>

Thank you for your order!

<br>

<table> 
<tr> 
<th>Name</th> 
<th>Quantity</th> 
<th>Price</th> 
</tr> 
*|LINE_ITEM_ROWS|*
<tr>
    <td colspan="3">&nbsp;</td>
</tr>
<tr>
    <td align="right" colspan="2">Item Total:</td>
    <td>*|ITEM_TOTAL|*</td>
</tr>
<tr>
    <td align="right" colspan="2">Total:</td>
    <td>*|TOTAL|*</td>
</tr>
</table>

<p>
    Ship to:

*|SHIPPING_ADDRESS_FIRST_NAME|*
*|SHIPPING_ADDRESS_LAST_NAME|*,<br>
*|SHIPPING_ADDRESS_ADDRESS1|*,<br>
*|SHIPPING_ADDRESS_CITY|*,<br>
*|SHIPPING_ADDRESS_STATE|*,<br>
*|SHIPPING_ADDRESS_COUNTRY|*,<br>
*|SHIPPING_ADDRESS_ZIPCODE|*<br>
</p>
```



### Order Cancellation

If an admin cancels an existing order, the hub will pick up on it and send a JSON message with the relevant data to the `/order_cancellation` URL. The Endpoint will transmit a Message to Mandrill, which then sends an email to the user, confirming that the order was canceled.

```json
{
  "message": "order:canceled",
  "payload": {
    "order": {
      ...
    }
  }
}
```

#### Parameters

| Name | Value | Example |
| :----| :-----| :------ |
| mandrill.api_key | Your Mandrill API Key | Aqws3958dhdjwb39 |
| mandrill.order_cancellation.from | Email Address to Send From | orders@spreecommerce.com |
| mandrill.order_cancellation.subject | Subject Line of Email | Your Order has been Cancelled |
| mandrill.order_cancellation.template | Mandrill Template to Fill In | order_cancellation |


#### Example Template

```html
<h1>*|ORDER_NUMBER|*</h1>

Your Order has been cancelled


<table> 
<tr> 
<th>Name</th> 
<th>Quantity</th> 
<th>Price</th> 
</tr> 
*|LINE_ITEM_ROWS|*
</table>

Item Total: *|ITEM_TOTAL|*
Total: *|TOTAL|*

Thanks

*|FIRST_NAME|*
*|LAST_NAME|*
*|COMPANY|*
*|ADDRESS1|*
*|ADDRESS2|*
*|CITY|*
*|STATE|*
*|COUNTRY|*
*|ZIPCODE|*

```


### Shipment Confirmation

After an order moves to the `shipped` order state, the store should send notice via the Hub to the Endpoint's `/shipment_confirmation` URL, with the relevant order and shipment data. The Endpoint will then instruct Mandrill to email the customer, notifying them that the order is en route.

```json
{
  "message": "shipment:confirmation",
  "message_id": "518726r84910000004",
  "payload": {
    "shipment_number": 1,
    "tracking_number": "71N4i304",
    "tracking_url": "http://www.ups.com/WebTracking/track",
    "carrier": "UPS",
    "shipped_date": "2013-06-27T13:29:46Z",
    ...
  }
}
```

### Parameters

| Name | Value | Example |
| :----| :-----| :------ |
| mandrill.api_key | Your Mandrill API Key | Aqws3958dhdjwb39 |
| mandrill.shipment_confirmation.from | Email Address to Send From | orders@spreecommerce.com |
| mandrill.shipment_confirmation.subject | Subject Line of Email | Your Shipment has Shipped |
| mandrill.shipment_confirmation.template | Mandrill Template to Fill In | shipment_confirmation |

#### Example Template

```html

<h1>*|ORDER_NUMBER|*</h1>

Your Order has Shipped!

*|CARRIER|*

*|TRACKING_NUMBER|*

*|TRACKING_URL|*

<table> 
<tr> 
<th>Name</th> 
<th>Quantity</th> 
<th>Price</th> 
</tr> 
*|LINE_ITEM_ROWS|*
</table>

Item Total: *|ITEM_TOTAL|*
Total: *|TOTAL|*

Thanks

*|FIRST_NAME|*
*|LAST_NAME|*
*|COMPANY|*
*|ADDRESS1|*
*|ADDRESS2|*
*|CITY|*
*|STATE|*
*|COUNTRY|*
*|ZIPCODE|*

```
