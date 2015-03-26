---
title: RLM Endpoint
---

## Overview

Endpoint for the [RLM](http://www2.ronlynn.com) ERP that saves order lines locally to be exported in bulk into a CSV formatted for RLM and uploaded to S3. Shipments are sent to the Persist method, and stored locally until Flush is called. The RLM ERP processes the orders line by line from a CSV file, this endpoint will break up the individual line items from the shipment and stores them in a format ready for RLM. 

We also store the complete payload for reference, so any future adjustments can be made from this endpoint.

+++
The source code for the [RLM Endpoint](https://github.com/spree/rlm_endpoint/) is available on Github.
+++

## Requirements for the connecting Spree store

To properly connect your Spree store with this endpoint you need to add a few custom attributes.

you need to add `upc` and `rlm_sku` attributes to `Spree::Variant` class. Both attributes are Strings.

***
For details on how to customize your store read the [Custom Attributes Tutorial](/integration/custom_attributes_tutorial.html)
***


## Services

### Persist

Stores the individual line items from a `shipment:ready` message. Each line is combined with all of the order header information, so that it can be stored in MongoDB until it is ready to be flushed. Lines are stored within the Endpoint so that they can be sent in batches. 


#### Parameters

| name | data_type | desc |
| -------------|---------|-------------|
|`rlm.shipping_map` | list | the mapping to determine the routing_code
|`rlm.ground_shipping_map` | list | the mapping to determine the routing_code (FX3 or FXG)
|`rlm.options_mapping` | list | the mapping from the options to size and color
|`rlm.options_mapping_defaults` |list | The list with default values for the option value when not available in payload. This can be empty.
|`rlm.sku_colors_mapping` | list | the mapping for SKU value for the color value.

### List type parameters.

We use the list types parameters to map values from the line_items to RLM used values.

The `rlm.shipping_map` parameter can look like this:

```ruby
{ 
  "name" => "rlm.shipping_map", 
  "data_type"=>"list", 
  "value" => [
    {"Overnight"=>"FXN", "International"=>"FXI", "Alaska"=>"FX2"}
  ]
}
```

In the endpoint we have access to this hash through the `@config` variable. We can access it with `@config["rlm.shipping_map"]` and it will return an array of hashes. For this list we only have one hash in the array so it's safe to always pick the first one like this `@config["rlm.shipping_map"].first`. The hash that we get from this paramater is used to map the used `shipping_method` to the rlm routing code. So if the the `shipping_method` was "Overnight" this mapping configures that the routing code will be "FXN".


#### Request

---shipment_ready.json---
```json
{
  "message":"shipment:ready",
  "payload":{
    "shipment":{
      "number":"H73631225586",
      "order_number":"R827587314",
      "email":"spree@example.com",
      "cost":5,
      "status":"shipped",
      "stock_location":null,
      "shipping_method":"UPS Ground (USD)",
      "tracking":null,
      "updated_at":null,
      "shipped_at":"2013-07-30T20:08:38Z",
      "shipping_address":{
        "firstname":"Brian",
        "lastname":"Quinn",
        "address1":"7735 Old Georgetown Rd",
        "address2":"",
        "zipcode":"20814",
        "city":"Bethesda",
        "state":"Maryland",
        "country":"US",
        "phone":"555-123-456"
      },
      "items":[
        {
          "name":"Spree Baseball Jersey",
          "sku":"SPR-00001",
          "external_ref":"",
          "quantity":1,
          "price":19.99,
          "variant_id":8,
          "options":{
            "tshirt-color":"BLK",
            "tshirt-size":"Medium"
          }
        },
        {
          "name":"Ruby on Rails Baseball Jersey",
          "sku":"ROR-00004",
          "external_ref":"",
          "quantity":1,
          "price":19.99,
          "variant_id":20,
          "options":{
            "tshirt-color":"WHT",
            "tshirt-size":"Medium"
          }
        }
      ]
    },
    "order":{
      "number":"R827587314",
      "channel":"spree",
      "email":"spree@example.com",
      "currency":"USD",
      "placed_on":"2013-07-30T19:19:05Z",
      "updated_at":"2013-07-30T20:08:39Z",
      "status":"complete",
      "totals":{
        "item":99.95,
        "adjustment":15,
        "tax":5,
        "shipping":0,
        "payment":114.95,
        "order":114.95
      },
      "line_items":[
        {
          "name":"Spree Baseball Jersey",
          "sku":"SPR-00001",
          "external_ref":"",
          "quantity":2,
          "price":19.99,
          "variant_id":8,
          "options":{

          }
        },
        {
          "name":"Ruby on Rails Baseball Jersey",
          "sku":"ROR-00004",
          "external_ref":"",
          "quantity":3,
          "price":19.99,
          "variant_id":20,
          "options":{
            "tshirt-color":"Red",
            "tshirt-size":"Medium"
          }
        }
      ],
      "adjustments":[
        {
          "name":"Shipping",
          "value":5
        },
        {
          "name":"Shipping",
          "value":5
        },
        {
          "name":"North America 5.0%",
          "value":5
        }
      ],
      "shipping_address":{
        "firstname":"Brian",
        "lastname":"Quinn",
        "address1":"7735 Old Georgetown Rd",
        "address2":"",
        "zipcode":"20814",
        "city":"Bethesda",
        "state":"Maryland",
        "country":"US",
        "phone":"555-123-456"
      },
      "billing_address":{
        "firstname":"Brian",
        "lastname":"Quinn",
        "address1":"7735 Old Georgetown Rd",
        "address2":"",
        "zipcode":"20814",
        "city":"Bethesda",
        "state":"Maryland",
        "country":"US",
        "phone":"555-123-456"
      },
      "payments":[
        {
          "number":6,
          "status":"completed",
          "amount":5,
          "payment_method":"Check"
        },
        {
          "number":5,
          "status":"completed",
          "amount":109.95,
          "payment_method":"Credit Card"
        }
      ],
      "shipments":[
        {
          "number":"H73631225586",
          "cost":5,
          "status":"shipped",
          "stock_location":null,
          "shipping_method":"UPS Ground (USD)",
          "tracking":null,
          "updated_at":null,
          "shipped_at":"2013-07-30T20:08:38Z",
          "items":[
            {
              "name":"Spree Baseball Jersey",
              "sku":"SPR-00001",
              "external_ref":"",
              "quantity":1,
              "price":19.99,
              "variant_id":8,
              "options":{

              }
            },
            {
              "name":"Ruby on Rails Baseball Jersey",
              "sku":"ROR-00004",
              "external_ref":"",
              "quantity":1,
              "price":19.99,
              "variant_id":20,
              "options":{
                "tshirt-color":"Red",
                "tshirt-size":"Medium"
              }
            }
          ]
        },
        {
          "number":"H62140775641",
          "cost":5,
          "status":"ready",
          "stock_location":null,
          "shipping_method":"UPS Ground (USD)",
          "tracking":"4532535354353452",
          "updated_at":null,
          "shipped_at":null,
          "items":[
            {
              "name":"Ruby on Rails Baseball Jersey",
              "sku":"ROR-00004",
              "external_ref":"",
              "quantity":2,
              "price":19.99,
              "variant_id":20,
              "options":{
                "tshirt-color":"Red",
                "tshirt-size":"Medium"
              }
            },
            {
              "name":"Spree Baseball Jersey",
              "sku":"SPR-00001",
              "external_ref":"",
              "quantity":1,
              "price":19.99,
              "variant_id":8,
              "options":{

              }
            }
          ]
        }
      ]
    },
    "original":{
      "id":5,
      "number":"R827587314",
      "item_total":"99.95",
      "total":"114.95",
      "state":"complete",
      "adjustment_total":"15.0",
      "user_id":1,
      "created_at":"2013-07-29T17:42:02Z",
      "updated_at":"2013-07-30T20:08:39Z",
      "completed_at":"2013-07-30T19:19:05Z",
      "payment_total":"114.95",
      "shipment_state":"partial",
      "payment_state":"paid",
      "email":"spree@example.com",
      "special_instructions":null,
      "currency":"USD",
      "ship_total":"10.0",
      "tax_total":"5.0",
      "bill_address":{
        "id":5,
        "firstname":"Brian",
        "lastname":"Quinn",
        "address1":"7735 Old Georgetown Rd",
        "address2":"",
        "city":"Bethesda",
        "zipcode":"20814",
        "phone":"555-123-456",
        "company":null,
        "alternative_phone":null,
        "country_id":49,
        "state_id":26,
        "state_name":null,
        "country":{
          "id":49,
          "iso_name":"UNITED STATES",
          "iso":"US",
          "iso3":"USA",
          "name":"United States",
          "numcode":840
        },
        "state":{
          "id":26,
          "name":"Maryland",
          "abbr":"MD",
          "country_id":49
        }
      },
      "ship_address":{
        "id":6,
        "firstname":"Brian",
        "lastname":"Quinn",
        "address1":"7735 Old Georgetown Rd",
        "address2":"",
        "city":"Bethesda",
        "zipcode":"20814",
        "phone":"555-123-456",
        "company":null,
        "alternative_phone":null,
        "country_id":49,
        "state_id":26,
        "state_name":null,
        "country":{
          "id":49,
          "iso_name":"UNITED STATES",
          "iso":"US",
          "iso3":"USA",
          "name":"United States",
          "numcode":840
        },
        "state":{
          "id":26,
          "name":"Maryland",
          "abbr":"MD",
          "country_id":49
        }
      },
      "line_items":[
        {
          "id":5,
          "quantity":2,
          "price":"19.99",
          "variant_id":8,
          "variant":{
            "id":8,
            "name":"Spree Baseball Jersey",
            "product_id":8,
            "sku":"SPR-00001",
            "upc":"SPR-UPC-00001",
            "rlm_sku":"SPR-RLM-00001",
            "price":"19.99",
            "weight":null,
            "height":null,
            "width":null,
            "depth":null,
            "is_master":true,
            "cost_price":"17.0",
            "permalink":"spree-baseball-jersey",
            "options_text":"",
            "option_values":[

            ],
            "images":[
              {
                "id":41,
                "position":1,
                "attachment_content_type":"image/jpeg",
                "attachment_file_name":"spree_jersey.jpeg",
                "type":"Spree::Image",
                "attachment_updated_at":"2013-07-24T17:01:27Z",
                "attachment_width":480,
                "attachment_height":480,
                "alt":null,
                "viewable_type":"Spree::Variant",
                "viewable_id":8,
                "attachment_url":"/spree/products/41/product/spree_jersey.jpeg?1374685287"
              },
              {
                "id":42,
                "position":2,
                "attachment_content_type":"image/jpeg",
                "attachment_file_name":"spree_jersey_back.jpeg",
                "type":"Spree::Image",
                "attachment_updated_at":"2013-07-24T17:01:28Z",
                "attachment_width":480,
                "attachment_height":480,
                "alt":null,
                "viewable_type":"Spree::Variant",
                "viewable_id":8,
                "attachment_url":"/spree/products/42/product/spree_jersey_back.jpeg?1374685288"
              }
            ]
          }
        },
        {
          "id":7,
          "quantity":3,
          "price":"19.99",
          "variant_id":20,
          "variant":{
            "id":20,
            "name":"Ruby on Rails Baseball Jersey",
            "product_id":3,
            "sku":"ROR-00004",
            "upc":"ROR-UPC-00004",
            "rlm_sku":"ROR-RLM-00004",
            "price":"19.99",
            "weight":null,
            "height":null,
            "width":null,
            "depth":null,
            "is_master":false,
            "cost_price":"17.0",
            "permalink":"ruby-on-rails-baseball-jersey",
            "options_text":"Size: M, Color: Red",
            "option_values":[
              {
                "id":33,
                "name":"Red",
                "presentation":"Red",
                "option_type_name":"tshirt-color",
                "option_type_id":2
              },
              {
                "id":16,
                "name":"Medium",
                "presentation":"M",
                "option_type_name":"tshirt-size",
                "option_type_id":1
              }
            ],
            "images":[
              {
                "id":7,
                "position":1,
                "attachment_content_type":"image/png",
                "attachment_file_name":"ror_baseball_jersey_red.png",
                "type":"Spree::Image",
                "attachment_updated_at":"2013-07-24T17:00:58Z",
                "attachment_width":240,
                "attachment_height":240,
                "alt":null,
                "viewable_type":"Spree::Variant",
                "viewable_id":20,
                "attachment_url":"/spree/products/7/product/ror_baseball_jersey_red.png?1374685258"
              },
              {
                "id":8,
                "position":2,
                "attachment_content_type":"image/png",
                "attachment_file_name":"ror_baseball_jersey_back_red.png",
                "type":"Spree::Image",
                "attachment_updated_at":"2013-07-24T17:00:59Z",
                "attachment_width":240,
                "attachment_height":240,
                "alt":null,
                "viewable_type":"Spree::Variant",
                "viewable_id":20,
                "attachment_url":"/spree/products/8/product/ror_baseball_jersey_back_red.png?1374685259"
              }
            ]
          }
        }
      ],
      "payments":[
        {
          "id":6,
          "amount":"5.0",
          "state":"completed",
          "payment_method_id":5,
          "payment_method":{
            "id":5,
            "name":"Check",
            "environment":"development"
          }
        },
        {
          "id":5,
          "amount":"109.95",
          "state":"completed",
          "payment_method_id":1,
          "payment_method":{
            "id":1,
            "name":"Credit Card",
            "environment":"development"
          }
        }
      ],
      "shipments":[
        {
          "id":6,
          "tracking":null,
          "number":"H73631225586",
          "cost":"5.0",
          "shipped_at":"2013-07-30T20:08:38Z",
          "state":"shipped",
          "order_id":"R827587314",
          "stock_location_name":"default",
          "shipping_rates":[
            {
              "id":74,
              "cost":"10.0",
              "selected":false,
              "shipment_id":6,
              "shipping_method_id":2
            },
            {
              "id":75,
              "cost":"15.0",
              "selected":false,
              "shipment_id":6,
              "shipping_method_id":3
            },
            {
              "id":73,
              "cost":"5.0",
              "selected":true,
              "shipment_id":6,
              "shipping_method_id":1
            }
          ],
          "shipping_method":{
            "name":"UPS Ground (USD)"
          },
          "inventory_units":[
            {
              "id":16,
              "state":"shipped",
              "variant_id":8,
              "order_id":null,
              "shipment_id":6,
              "return_authorization_id":null
            },
            {
              "id":15,
              "state":"shipped",
              "variant_id":20,
              "order_id":null,
              "shipment_id":6,
              "return_authorization_id":null
            }
          ]
        },
        {
          "id":5,
          "tracking":"4532535354353452",
          "number":"H62140775641",
          "cost":"5.0",
          "shipped_at":null,
          "state":"ready",
          "order_id":"R827587314",
          "stock_location_name":"default",
          "shipping_rates":[
            {
              "id":80,
              "cost":"10.0",
              "selected":false,
              "shipment_id":5,
              "shipping_method_id":2
            },
            {
              "id":81,
              "cost":"15.0",
              "selected":false,
              "shipment_id":5,
              "shipping_method_id":3
            },
            {
              "id":79,
              "cost":"5.0",
              "selected":true,
              "shipment_id":5,
              "shipping_method_id":1
            }
          ],
          "shipping_method":{
            "name":"UPS Ground (USD)"
          },
          "inventory_units":[
            {
              "id":13,
              "state":"on_hand",
              "variant_id":20,
              "order_id":5,
              "shipment_id":5,
              "return_authorization_id":null
            },
            {
              "id":12,
              "state":"on_hand",
              "variant_id":20,
              "order_id":5,
              "shipment_id":5,
              "return_authorization_id":null
            },
            {
              "id":10,
              "state":"on_hand",
              "variant_id":8,
              "order_id":5,
              "shipment_id":5,
              "return_authorization_id":null
            }
          ]
        }
      ],
      "adjustments":[
        {
          "id":16,
          "amount":"5.0",
          "label":"Shipping",
          "mandatory":true,
          "eligible":true,
          "originator_type":"Spree::ShippingMethod",
          "adjustable_type":"Spree::Order"
        },
        {
          "id":20,
          "amount":"5.0",
          "label":"Shipping",
          "mandatory":true,
          "eligible":true,
          "originator_type":"Spree::ShippingMethod",
          "adjustable_type":"Spree::Order"
        },
        {
          "id":22,
          "amount":"5.0",
          "label":"North America 5.0%",
          "mandatory":false,
          "eligible":true,
          "originator_type":"Spree::TaxRate",
          "adjustable_type":"Spree::Order"
        }
      ],
      "credit_cards":[
        {
          "id":2,
          "month":"7",
          "year":"2013",
          "cc_type":"visa",
          "last_digits":"1111",
          "first_name":"Brian",
          "last_name":"Quinn",
          "gateway_customer_profile_id":"BGS-414100",
          "gateway_payment_profile_id":null
        }
      ]
    },
    "parameters":[
      {
        "name":"rlm.shipping_map",
        "data_type":"list",
        "value":[
          {
            "UPS Ground (USD)":"FXN",
            "International":"FXI",
            "Alaska & Hawaii":"FSP",
            "US-3-DAY":"FX3"
          }
        ]
      },
      {
        "name":"rlm.ground_shipping_map",
        "data_type":"list",
        "value":[
          {
            "Montana":"FXG",
            "Wyoming":"FXG"
          }
        ]
      },
      {
        "name":"rlm.options_mapping",
        "data_type":"list",
        "value":[
          {
            "size":"tshirt-size",
            "color":"tshirt-color"
          }
        ]
      },
      {
        "name":"rlm.sku_colors_mapping",
        "data_type":"list",
        "value":[
          {
            "WHT":"W",
            "BLK":"B",
            "TAN":"T",
            "NAVY":"N",
            "GREY":"G",
            "ROYAL":"RB"
          }
        ]
      }
    ]
  },
  "store_id":"51f8eaa2b4"
}
```

#### Response

The content of the payload storage is truncated in the example response below.

---persisted_lines.json---

```json
{
  "persisted_lines": [
    {
      "_id": {
        "$oid": "5226e72ddbf0383df8000001"
      },
      "billing_address_1": "7735 Old Georgetown Rd",
      "billing_address_2": "",
      "billing_city": "Bethesda",
      "billing_country": "US",
      "billing_fullname": "Brian Quinn",
      "billing_state": "MD",
      "billing_zip": "20814",
      "color": "BLK",
      "created_at": "2013-09-04T07:54:21.655Z",
      "email": "spree@example.com",
      "flushed_at": null,
      "is_spree_order": true,
      "line_discount": "0.00000",
      "order_discount": 0.0,
      "order_no": "R827587314",
      "payload": {
        "shipment": { "..." },
        "order": { "..." },
        "original": { "..." },
        "parameters": [ "..." ]
      },
      "price": 19.99,
      "quantity": 1,
      "rlm_sku": "SPR-RLM-00001",
      "routing_code": "FXN",
      "shipment_number": "H73631225586",
      "shipping_address_1": "7735 Old Georgetown Rd",
      "shipping_address_2": "",
      "shipping_city": "Bethesda",
      "shipping_country": "US",
      "shipping_fullname": "Brian Quinn",
      "shipping_state": "MD",
      "shipping_zip": "20814",
      "size": "Medium",
      "sku": "SPR-00001",
      "special001": "",
      "special002": "",
      "special003": "",
      "special004": "",
      "style_no": "SPR-00001",
      "total": 114.95,
      "upc": "SPR-UPC-00001",
      "updated_at": "2013-09-04T07:54:21.655Z"
    },
    {
      "_id": {
        "$oid": "5226e72ddbf0383df8000002"
      },
      "billing_address_1": "7735 Old Georgetown Rd",
      "billing_address_2": "",
      "billing_city": "Bethesda",
      "billing_country": "US",
      "billing_fullname": "Brian Quinn",
      "billing_state": "MD",
      "billing_zip": "20814",
      "color": "WHT",
      "created_at": "2013-09-04T07:54:21.662Z",
      "email": "spree@example.com",
      "flushed_at": null,
      "is_spree_order": true,
      "line_discount": "0.00000",
      "order_discount": 0.0,
      "order_no": "R827587314",
      "payload": {
        "shipment": { "..." },
        "order": { "..." },
        "original": { "..." },
        "parameters": [ "..." ]
      },
      "price": 19.99,
      "quantity": 1,
      "rlm_sku": "ROR-RLM-00004",
      "routing_code": "FXN",
      "shipment_number": "H73631225586",
      "shipping_address_1": "7735 Old Georgetown Rd",
      "shipping_address_2": "",
      "shipping_city": "Bethesda",
      "shipping_country": "US",
      "shipping_fullname": "Brian Quinn",
      "shipping_state": "MD",
      "shipping_zip": "20814",
      "size": "Medium",
      "sku": "ROR-00004",
      "special001": "",
      "special002": "",
      "special003": "",
      "special004": "",
      "style_no": "ROR-00004",
      "total": 114.95,
      "upc": "ROR-UPC-00004",
      "updated_at": "2013-09-04T07:54:21.662Z"
    }
  ]
}
```

### Flush

Exports the stored lines in 2 different csv files. One for Amazon imported orders, and one for the regular Spree orders. Flush is called from the Hub periodically, using a scheduler. When Flush is called, it loads every shipment in the database that hasn't yet been sent into a CSV file. 

#### File Structure

Every line item on every shipment waiting to be flushed will be included in the file when the flush action is run. All of the Order Header information, such as number, address and totals are included on every line, so that RLM knows how to process it. A line is generated for each item on the shipment, containing identifying information (SKU, style, color, size), transactional information (price and discount), and quantity to ship. 

#### File Names

File names take multiple datapoints to construct. 

| Name | Value | Example |
| :----| :-----| :------ |
| file_name_ts | Current Date, formatted as Year_Month_Day | 2013_09_25 |
| name_part | 10_40 or 14_40 depending on whether it is currently before noon or after noon | 10_40 |
| file_type | Either orders_integrator or orders_amazon based on destination | orders_amazon |

These all get put together into a final filename such as 2013_09_25_10_40_orders_amazon.csv

#### File Upload
Files are generated in a temp location on the server the endpoint is running on. They are then uploaded to S3 based on the config parameters below, and the temp file is removed.

#### Parameters

| key | desc |
| ------------- |-------------|
|`rlm.csv_target_bucket` | Bucket where we will store the RLM CSV
|`rlm.aws_access_key_id` | AWS access ID
|`rlm.aws_secret_access_key` | AWS access Key
|`rlm.timezone` | The timezone on the server to correctly name the files


#### Request

---flush_message.json---
```json
{
  "message":"rlm:line:flush",
  "payload":{
    "parameters":[
      { "name":"rlm.csv_target_bucket", "value":"dev-bucket"},
      { "name":"rlm.aws_access_key_id", "value":"abc" },
      { "name":"rlm.aws_secret_access_key", "value":"cded"},
      { "name":"rlm.timezone", "value":"EST"}
    ]
  },
  "store_id":"51f8eaa2b4"
}
```

#### Response
```json
{
  "message_id":"52263b13b43957220e004c1a",
  "spree":{
    "filename":"2013_09_03_14_40_orders_integrator.csv",
    "s3_file_path":"https://dev-bucket.s3.amazonaws.com/2013_09_03_14_40_orders_integrator.csv?AWSAccessKeyId=abc&Signature=9Jz1zm4bi9olss3I09%2ByfeL7G90%3D&Expires=1380051606",
    "shipments":[
      "H32727236376",
      "H40806867857",
      "H38466380585",
      "H82103347522",
      "H18836822206",
      "H58434365632",
      "H32152647223",
      "H52531641761",
      "H12081176427",
      "H57531134225",
      "H63073244070",
      "H81166387746",
      "H53264268082",
      "H31760327803",
      "H50827715061"
    ]
  },
  "amazon":{
    "filename":"2013_09_03_14_40_orders_amazon.csv",
    "s3_file_path":"https://dev-bucket.s3.amazonaws.com/2013_09_03_14_40_orders_amazon.csv?AWSAccessKeyId=abc&Signature=WPmBU0oGU%2ByAKv78%2BdTrhkPd5pE%3D&Expires=1380051606",
    "shipments":[
      "H48343226468"
    ]
  }
}
```
