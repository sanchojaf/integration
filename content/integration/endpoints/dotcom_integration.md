---
title: Dotcom Distribution Endpoint
---

## Overview

[Dotcom Distribution](http://www.dotcomdist.com/) is the premier provider of logistics and fulfillment services to growing retailers & manufacturers.

+++
The source code for the [Dotcom Distribution Endpoint](https://github.com/spree/dotcom_endpoint/) is available on Github.
+++

## Services

### Send Shipment

Send a shipment to Dotcom Distribution.

#### Request

---shipment_ready.json---
```json
{
    "message_id": "51af1dc5fe53543f1200f519",
    "message": "shipment:ready",
    "payload": {
      "shipment": {
        "number": "H67606710123",
        "order_number": "R805661123",
        "email": "brian@spreecommerce.com",
        "cost": 10.0,
        "status": "ready",
        "stock_location": null,
        "shipping_method": "Standard",
        "tracking": null,
        "updated_at": null,
        "shipped_at": null,
        "shipping_address": {
          "firstname": "Brian",
          "lastname": "Quinn",
          "address1": "2 Wisconsin Cir.",
          "address2": "",
          "zipcode": "20815",
          "city": "Chevy Chase",
          "state": "Maryland",
          "country": "US",
          "phone": "555-123-123"
        },
        "items": [
          {
            "name": "Socks",
            "sku": "9011CC",
            "external_ref": "",
            "quantity": 1,
            "price": 29.0,
            "variant_id": 4123,
            "options": {
              "size": "M",
              "color": "BLK"
            }
          },
          {
            "name": "Shoes",
            "sku": "2001SSBM",
            "external_ref": "",
            "quantity": 1,
            "price": 27.0,
            "variant_id": 4321,
            "options": {
              "option-name": "value"
            }
          }
        ]
      },
      "order": {},
      "original": {}
      ]
    }
}
```

#### Parameters

| Name | Value | Data Type |Example |
| :----| :-----| :------ |:------ |
| api_key | Your Dotcom Distribution API key | string | dj20492dhjkdjeh2838w7 |
| password | Your Dotcom Distribution account password | string | dj20492dhjkdjeh2838w7 |
| shipping_lookup | Spree to Dotcom Distribution shipping methods' mapping | list | [{'UPS Ground (USD)' => '03'}, {'UPS Two Day (USD)' => '02'}] |

#### Response

---notification_info.json---

```json
{
  "message_id": "51af1dc5fe53543f1200f519",
  "notifications": [
    {
      "level": "info",
      "subject": "Successfully Sent Shipment to Dotcom Distribution",
      "description": "Successfully Sent Shipment to Dotcom Distribution"
    }
  ]
}
```

### Tracking

Track shipment dispatches.

#### Request

---dotcom_shipment_results_poll.json---
```json

{
  "message_id": "51af1dc5fe53543f1200f519",
  "message": "dotcom:shipment_results:poll",
  "payload": {}
}
```

#### Parameters

| Name | Value | Data Type |Example |
| :----| :-----| :------ |:------ |
| api_key | Your Dotcom Distribution API key | string |dj20492dhjkdjeh2838w7 |
| password | Your Dotcom Distribution account password | string| dj20492dhjkdjeh2838w7 |
| last_polling_datetime | Initial date shipment polling will start from | string | 2013-10-28 3:15:21 -0400 |

#### Response

---shipment_confirm.json---
```json
{
  "message_id": "51af1dc5fe53543f1200f519",
  "messages": [
    {
      "message": "shipment:confirm",
      "payload": {
        "shipment": {
          "number": "H662132822",
          "order_number": "R894254432",
          "tracking": "1ZV2X5140100003086"
        }
      },
      "parameters": [
        {
          "name": "dotcom.last_polling_datetime",
          "value": "2013-10-29 3:15:21 -0400"
        }
      ]
    }
  ]
}
```