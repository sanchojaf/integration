---
title: Amazon Endpoint
---

## Overview

[Amazon](http://www.amazon.com/) is one of the Internet's largest retailers. This endpoint can be used to poll the Amazon API and import any new orders you may have there into your Spree store.

+++
The source code for the [Amazon Endpoint](https://github.com/spree/amazon_endpoint/) is available on Github.
+++

## Requirements

In order to configure and use the [amazon_endpoint](https://github.com/spree/amazon_endpoint), you will need to first have an [Amazon seller account](http://services.amazon.com/content/sell-on-amazon.htm), which provides you with your:

* Amazon AWS access key
* Amazon secret key
* Amazon seller ID
* Amazon marketplace_id

You'll also need to set a `amazon.last_updated_after` date to give the Endpoint a date range against which to compare.

## Services

### Return Orders

Returns all new orders created in the Amazon store since the `amazon.last_updated_after` date.

####Request

---amazon_order_poll.json---
```json
{
  "message": "amazon:order:poll",
  "message_id": "1234567",
  "payload": {}
}
```

#### Parameters

| Name | Value | Example |
| :----| :-----| :------ |
| amazon.aws_access_key | Your AWS Access Key | Aqws3958dhdjwb39 |
| amazon.secret_key | Your AWS Secret Key | dj20492dhjkdjeh2838w7 |
| amazon.marketplace_id | Your Amazon Marketplace ID | nama9vach3kis |
| amazon.seller_id | Your Amazon Seller ID | hugi0ty8su2zyh |
| amazon.last_updated_after | The last updated_at ([iso8601](http://en.wikipedia.org/wiki/ISO_8601) format) to poll orders | 2012-08-16T01:00:00Z |


#### Response

---order_import.json---
```json
{
  "message_id": "1234567",
  "parameters": [
    { "name": "amazon.last_updated_after", "value": "2013-08-16T01:30:00Z" }
  ],
  "messages": [
    {
      "message": "order:import",
      "payload": {
        "order": {
          "number": "100-EXAMPLE-1000000",
          "channel": "Amazon.com",
          "currency": "USD",
          "status": "Unshipped",
          "placed_on": "2013-08-16T01:25:00Z",
          "updated_at": "2013-08-16T01:30:00Z",
          "email": "ceth5ni9mo4whyc@marketplace.amazon.com",
          "totals": {
            "item": 64.0,
            "adjustment": 0.0,
            "tax": 0.0,
            "shipping": 0.0,
            "order": 64.0,
            "payment": 64.0
          },
          "adjustments": [
            { "name": "Shipping Discount", "value": 0.0 },
            { "name": "Promotion Discount", "value": 0.0 },
            { "name": "Amazon Tax", "value": 0.0 },
            { "name": "Gift Wrap Price", "value": 0.0 },
            { "name": "Gift Wrap Tax", "value": 0.0 }
          ],
          "line_items": [
            {
              "name": "T-Shirt, Black, XX-Large",
              "price": 29.0,
              "sku": "100TSBXXL",
              "quantity": 1,
              "variant_id": null,
              "external_ref": null,
              "options": {}
            },
            {
              "name": "T-Shirt, White, XX-Large",
              "price": 35.0,
              "sku": "100TSWXXL",
              "quantity": 1,
              "variant_id": null,
              "external_ref": null,
              "options": {}
            }
          ],
          "payments": [
            {
              "amount": 64.0,
              "payment_method": "Amazon",
              "status": "complete"
            }
          ],
          "shipments": [
            {
              "cost": 0.0,
              "status": "Unshipped",
              "shipping_method": "Standard",
              "items": [
                {
                  "name": "T-Shirt, Black, XX-Large",
                  "price": 29.0,
                  "sku": "100TSBXXL",
                  "quantity": 1,
                  "variant_id": null,
                  "external_ref": null,
                  "options": {}
                },
                {
                  "name": "T-Shirt, White, XX-Large",
                  "price": 35.0,
                  "sku": "100TSWXXL",
                  "quantity": 1,
                  "variant_id": null,
                  "external_ref": null,
                  "options": {}
                }
              ],
              "stock_location": "",
              "tracking": "",
              "number": ""
            }
          ],
          "shipping_address": {
            "firstname": "Patrick",
            "lastname": "Cook",
            "address1": "7159 edwards rd",
            "address2": "Apartment A-12",
            "city": "Seymour",
            "zipcode": "37284",
            "phone": "471-543-4073",
            "country": "US",
            "state": "Pennsylvania"
          },
          "billing_address": {
            "firstname": "Patrick",
            "lastname": "Cook",
            "address1": "7159 edwards rd",
            "address2": "Apartment A-12",
            "city": "Seymour",
            "zipcode": "37284",
            "phone": "471-543-4073",
            "country": "US",
            "state": "Pennsylvania"
          }
        }
      }
    }
  ]
}
```

### Return Specific Order

Returns a specific order from Amazon store filtered by `amazon_order_id`.

####Request

---amazon_import_by_number.json---
```json
{
  "message": "amazon:import:by_number",
  "message_id": "1234567",
  "payload": {
    "amazon_order_id": "100-EXAMPLE-1000000"
   }
}
```

#### Parameters

| Name | Value | Example |
| :----| :-----| :------ |
| amazon.aws_access_key | Your AWS Access Key | Aqws3958dhdjwb39 |
| amazon.secret_key | Your AWS Secret Key | dj20492dhjkdjeh2838w7 |
| amazon.marketplace_id | Your Amazon Marketplace ID | nama9vach3kis |
| amazon.seller_id | Your Amazon Seller ID | hugi0ty8su2zyh |


#### Response

---order_import.json---
```json
{
  "message_id": "1234567",
  "messages": [
    {
      "message": "order:import",
      "payload": {
        "order": {
          "number": "100-EXAMPLE-1000000",
          "channel": "Amazon.com",
          "currency": "USD",
          "status": "Unshipped",
          "placed_on": "2013-08-16T01:25:00Z",
          "updated_at": "2013-08-16T01:30:00Z",
          "email": "ceth5ni9mo4whyc@marketplace.amazon.com",
          "totals": {
            "item": 64.0,
            "adjustment": 0.0,
            "tax": 0.0,
            "shipping": 0.0,
            "order": 64.0,
            "payment": 64.0
          },
          "adjustments": [
            { "name": "Shipping Discount", "value": 0.0 },
            { "name": "Promotion Discount", "value": 0.0 },
            { "name": "Amazon Tax", "value": 0.0 },
            { "name": "Gift Wrap Price", "value": 0.0 },
            { "name": "Gift Wrap Tax", "value": 0.0 }
          ],
          "line_items": [
            {
              "name": "T-Shirt, Black, XX-Large",
              "price": 29.0,
              "sku": "100TSBXXL",
              "quantity": 1,
              "variant_id": null,
              "external_ref": null,
              "options": {}
            },
            {
              "name": "T-Shirt, White, XX-Large",
              "price": 35.0,
              "sku": "100TSWXXL",
              "quantity": 1,
              "variant_id": null,
              "external_ref": null,
              "options": {}
            }
          ],
          "payments": [
            {
              "amount": 64.0,
              "payment_method": "Amazon",
              "status": "complete"
            }
          ],
          "shipments": [
            {
              "cost": 0.0,
              "status": "Unshipped",
              "shipping_method": "Standard",
              "items": [
                {
                  "name": "T-Shirt, Black, XX-Large",
                  "price": 29.0,
                  "sku": "100TSBXXL",
                  "quantity": 1,
                  "variant_id": null,
                  "external_ref": null,
                  "options": {}
                },
                {
                  "name": "T-Shirt, White, XX-Large",
                  "price": 35.0,
                  "sku": "100TSWXXL",
                  "quantity": 1,
                  "variant_id": null,
                  "external_ref": null,
                  "options": {}
                }
              ],
              "stock_location": "",
              "tracking": "",
              "number": ""
            }
          ],
          "shipping_address": {
            "firstname": "Patrick",
            "lastname": "Cook",
            "address1": "7159 edwards rd",
            "address2": "Apartment A-12",
            "city": "Seymour",
            "zipcode": "37284",
            "phone": "471-543-4073",
            "country": "US",
            "state": "Pennsylvania"
          },
          "billing_address": {
            "firstname": "Patrick",
            "lastname": "Cook",
            "address1": "7159 edwards rd",
            "address2": "Apartment A-12",
            "city": "Seymour",
            "zipcode": "37284",
            "phone": "471-543-4073",
            "country": "US",
            "state": "Pennsylvania"
          }
        }
      }
    }
  ]
}
```

### Confirm Shipment

Confirm an order inside of Amazon as shipped using Amazon's MWS Feeds API. This action will submit an Order Fulfillment feed. Performing this action will add a tracking number, shipping method, and a shipped status to an order. Read more about the MWS Feeds API [here](https://images-na.ssl-images-amazon.com/images/G/01/mwsportal/doc/en_US/bde/MWSFeedsApiReference._V379394221_.pdf)

####Request

---shipment_confirm.json---
```json
{
  "message": "shipment:confirm",
  "message_id": "1234567",
  "payload": {
    "shipment": {
      "number": "H03606064322",
      "order_number": "103-6652650-4045858",
      "email": "spree@example.com",
      "cost": 0.0,
      "status": "ready",
      "stock_location": null,
      "shipping_method": "UPS Ground",
      "tracking": "915293072790129",
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
          "name": "test",
          "sku": "27368845791002",
          "external_ref": "",
          "quantity": 1,
          "price": 1.25,
          "variant_id": 2,
          "options": {
          }
        }
      ]
    }
  }
}
```

#### Parameters

| Name | Value | Example |
| :----| :-----| :------ |
| amazon.aws_access_key | Your AWS Access Key | Aqws3958dhdjwb39 |
| amazon.secret_key | Your AWS Secret Key | dj20492dhjkdjeh2838w7 |
| amazon.marketplace_id | Your Amazon Marketplace ID | nama9vach3kis |
| amazon.seller_id | Your Amazon Seller ID | hugi0ty8su2zyh |

#### Response

---amazon_feed_status.json---
```json
{
  "message_id": "123456",
  "messages": [
    {
      "message": "amazon:feed:status",
      "payload": {
         "feed_id": "123456"
      }
    }
  ]
}
```

### Get Feed Status

Check the status of a submitted feed to see if it was successfully processed.

####Request

---amazon_feed_status.json---
```json
{
  "message_id": "!23456",
  "message": "amazon:feed:status",
  "payload": {
     "feed_id": "7891011"
  }
}
```

#### Parameters

| Name | Value | Example |
| :----| :-----| :------ |
| amazon.aws_access_key | Your AWS Access Key | Aqws3958dhdjwb39 |
| amazon.secret_key | Your AWS Secret Key | dj20492dhjkdjeh2838w7 |
| amazon.marketplace_id | Your Amazon Marketplace ID | nama9vach3kis |
| amazon.seller_id | Your Amazon Seller ID | hugi0ty8su2zyh |

#### Response

---notification_info.json---
```json
{
  "message_id": "123456",
  "notifications": [
    {
      "level": "info",
      "subject": "Feed Complete",
      "description": "Succesfully processed feed #7891011"
    }
  ]
}
```


