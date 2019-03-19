Creating orders
===============

Like with everything else in Fulfil, orders can also be created over the API.
However, given how common this use case is, Fulfil provides a different API
endpoint that provides an idempotent interface to create orders, process them,
handle payments and do a lot with UI based configuration.

This guide explains the implementation using this API endpoint. However, you
are free to roll out your multi-call solution to creating orders if you need
more flexibility than this endpoint offers.

.. contents:: In this guide
   :local:
   :depth: 2


Lifecycle of an Order
---------------------

When an order is created in Fulfil, it always starts as a draft. At this point,
any detail in the order can be edited by anyone with create access to the channel
it was created in.

draft 
    Any detail on the order can be edited.

quote
    Usually used by wholesale b2b transactions where the customer should be sent a
    quote prior to order confirmation.

confirmed
    When the order has been confirmed by the customer. For web/e-commerce orders, this
    would be the checkout step. For wholesale customers with pre-existing relationships
    this could be an email confirming the order.

processing
    When an order is processed, Fulfil creates additional resources required to Fulfil
    the order. This could be customer shipments, purchase orders for drop shipments or
    raising invoices for non-inventory items (like a gift card).

done
    When all items in an order has been fulfilled, the order is moved to the done state.

Channel Setup
-------------

Before you can create orders over this endpoint, you must have a channel and
the API user must have create access to the channel.

Create Order API
----------------

To create an order using this endpoint, you can call Channel.create_order with the channel ID
and the JSON representing an order.

.. code-block:: python

    CHANNEL_ID = 1
    order_details = {
        # Unique identifier that fulfil uses as 
        # idempotency key to avoid duplicates.
        'channel_identifier': 'ORDER-ID-1',
        # Order number from external channel
        'reference': 'ORDER-ID-1',
        # The time at which the order was confirmed.
        "confirmed_at": "2018-12-13T08:20:23.251-05:00",

        'customer': {
            'name': 'John Doe',                 # Full name of the customer
            'contacts': [
                ['email', 'john@thedoe.com']    # Email address of the customer. Also used to deduplicate.
            ],
        },

        'billing_address': {
            'name': 'John Doe',
            'address1': '444 Castro St',
            'address2': 'Suite 1200',
            'city': 'Mountain View',
            'zip': '94041',
            'subdivision_code': 'CA',           # Two character country code
            'country_code': 'US',               # Two character subdivison code (state, province, district)

            # This details is used for shipping notifications.
            # Shipping carriers are given the phone number.
            'email': 'john@thedoe.com',
            'phone': '123-456-7890',

            # ID of the address in external system
            # Used to identify changes to addresses. 
            'channel_identifier': '6745',
        },

        'shipping_address': {
            'name': 'Joe Doe',
            'address1': '67 Yonge St',
            'address2': 'Suite 1600',
            'city': 'Toronto',
            'zip': 'M5E 1J8',
            'subdivision_code': 'ON',           # Two character country code
            'country_code': 'CA',               # Two character subdivison code (state, province, district)

            # This details is used for shipping notifications.
            # Shipping carriers are given the phone number.
            'email': 'john@thedoe.com',
            'phone': '123-456-7890',

            # ID of the address in external system
            # Used to identify changes to addresses. 
            'channel_identifier': '6745',
        },
        # Order items excluding shipping lines
        'sale_lines': [
            {
                # Unique ID for the order line item from external channel
                # Often required when partial fulfillment is done on items
                'channel_identifier': '15425',

                'sku': 'LIGHTSABER-9',
                'quantity': 1,
                'unit_price': Decimal('200.00'),
                'amount': Decimal('200.00'),

                # a note specific to the line.
                'comment': 'With extended range for a tall person'
            },
        ],
        'shipping_lines': [
            {
                # Unique ID for the shipping line item from external channel
                'channel_identifier': '7848',
                'description': 'Canada Post',
                'amount': Decimal('0'),
                'carrier_service_code': 'ups_ground',
            }
        ],

        'amount': Decimal('200.00'),
        'currency_code': 'USD',
        'requested_shipping_service': 'ups_ground',

        'status': 'pending',
        'financial_status': 'paid',
        'fulfillment_status': 'unshipped',

    }
    Channel.create(CHANNEL_ID, order_details)

.. code-block:: shell

    curl 'https://{merchant_id}.fulfil.io/api/{version}/model/sale.channel/create_order' \
    -X 'PUT' \
    -H 'X-API-KEY: {YOUR_API_KEY}' \
    -H 'Content-Type: application/json' \
    -d '[1, {"status": "pending", "billing_address": {"city": "Mountain View", "name": "John Doe", "zip": "94041", "subdivision_code": "CA", "address1": "444 Castro St", "address2": "Suite 1200", "channel_identifier": "6745", "email": "john@thedoe.com", "phone": "123-456-7890", "country_code": "US"}, "reference": "ORDER-ID-1", "shipping_lines": [{"carrier_service_code": "ups_ground", "amount": {"decimal": "0", "__class__": "Decimal"}, "description": "Canada Post", "channel_identifier": "7848"}], "confirmed_at": "2018-12-13T08:20:23.251-05:00", "sale_lines": [{"sku": "LIGHTSABER-9", "amount": {"decimal": "200.00", "__class__": "Decimal"}, "quantity": 1, "unit_price": {"decimal": "200.00", "__class__": "Decimal"}, "channel_identifier": "15425"}], "customer": {"name": "John Doe", "contacts": [["email", "john@thedoe.com"]]}, "shipping_address": {"city": "Toronto", "name": "Joe Doe", "zip": "M5E 1J8", "subdivision_code": "ON", "address1": "67 Yonge St", "address2": "Suite 1600", "channel_identifier": "6745", "email": "john@thedoe.com", "phone": "123-456-7890", "country_code": "CA"}, "financial_status": "paid", "fulfillment_status": "unshipped", "amount": {"decimal": "200.00", "__class__": "Decimal"}, "channel_identifier": "ORDER-ID-1", "currency_code": "USD"}]'

Idempotency
```````````

Networks are unreliable. Even between servers hosted on data centers, intermittent
failures are unusual but still bound to happen. To overcome this, it's important to
design critical APIs to handle failure. In this case, Fulfil does that for orders using
the `channel_identifier` as an idempotency key.

This is inspired by Stripe's idempotent API design.

The `create_order` endpoint can be called any number of times and Fulfil
guarantees that the side effects occur only once. So even if you call the endpoint
multiple times for the same order, the order only gets created once - and the second
request returns the same response (the record of the order created).


Order Properties
----------------

This list of properties are not properties of the order object itself, but the properties
that can be used when creating orders using this endpoint.

Example JSON
````````````

.. code-block:: javascript

    {
        "channel_identifier": "ORDER-ID-1",
        "reference": "ORDER-ID-1",
        "confirmed_at": "2018-12-13T08:20:23.251-05:00",

        
        "customer": {
            "name": "John Doe",
            "contacts": [
                [
                    "email",
                    "john@thedoe.com"
                ]
            ]
        },
        

        "billing_address": {
            "city": "Mountain View",
            "name": "John Doe",
            "zip": "94041",
            "subdivision_code": "CA",
            "address1": "444 Castro St",
            "address2": "Suite 1200",
            "channel_identifier": "6745",
            "email": "john@thedoe.com",
            "phone": "123-456-7890",
            "country_code": "US"
        },
        "shipping_address": {
            "city": "Toronto",
            "name": "Joe Doe",
            "zip": "M5E 1J8",
            "subdivision_code": "ON",
            "address1": "67 Yonge St",
            "address2": "Suite 1600",
            "channel_identifier": "6745",
            "email": "john@thedoe.com",
            "phone": "123-456-7890",
            "country_code": "CA"
        },


        "sale_lines": [
            {
                "sku": "LIGHTSABER-9",
                "amount": "200.00",
                "quantity": 1,
                "unit_price": "200.00",
                "channel_identifier": "15425"
            }
        ],

        "shipping_lines": [
            {
                "carrier_service_code": "ups_ground",
                "amount": "0",
                "description": "Free Ground Shipping",
                "channel_identifier": "7848"
            }
        ],


        "amount": "200.00",
        "currency_code": "USD",


        "status": "pending",
        "financial_status": "paid",
        "fulfillment_status": "unshipped"
    }

`channel_identifier`
`````````````````````

The unique identifier of the order. This can be an internal database ID if
that's the unique identifier for your order or an order number if it is unique
enough.

For example, the fulfil Amazon integration uses the customer facing order number
`102-102909028-09090` as the channel_identifier while the shopify integration uses
the integer id of the order as the channel_identifier.

.. important::

    This is the only identifier used to ensure deduplication. If the API is called
    multiple times with the same channel_identifier, then Fulfil ensures order is
    only created once.

`reference`
```````````

The customer facing order number. 

`confirmed_at`
``````````````

The time at which the order was confirmed. The date and time (ISO 8601 format) when the order 
was confirmed.

`customer`
``````````

The name of the customer could be the name of the contact (First and Last) that placed
the order or the name of the business.

.. code-block:: javascript

    {
        "name": "First Last",
        "contacts": [
            ("phone", "640-6404-676"),
            ("phone", "640-6404-676 Ex 789"),
            ("email": "joe@thedoe.com"),
        ]
    }

Supported contact types:

* phone
* mobile
* fax
* email
* website
* skype
* twitter
* facebook
* instagram
* pigeon-post

`billing_address` and `shipping_address`
`````````````````````````````````````````

The billing and shipping address follows the same structure.

    .. code-block:: javascript

        {
            "name": "John Doe",
            "address1": "444 Castro St",
            "address2": "Suite 1200",
            "city": "Mountain View",
            "zip": "94041",
            "subdivision_code": "CA",           # Two character country code
            "country_code": "US",               # Two character subdivison code (state, province, district)

            # This details is used for shipping notifications.
            # Shipping carriers are given the phone number.
            "email": "john@thedoe.com",
            "phone": "123-456-7890",

            # ID of the address in external system
            # Used to identify changes to addresses. 
            "channel_identifier": "6745",
        },

`sale_lines`
````````````

An array of order items to be shipped.

Each order item should have at-least the following properties:

.. code-block:: python

    {
        # Unique ID for the order line item from external channel
        # Often required when partial fulfillment is done on items
        "channel_identifier": '15425',

        'sku': 'LIGHTSABER-9',
        'quantity': 1,
        'unit_price': Decimal('200.00'),
        'amount': Decimal('200.00'),
    },


`channel_identifier`
    The id of the order line on the external system. This is usually used to
    identify the order line being fulfilled when shipment information is sent
    back. This comes in handy, especially when customer service could substitute
    another SKU.
`sku`
    The SKU of the product sold. Either the SKU or `variant_identifier` must be
    specified.
`unit_price`
    The price at which the product was sold. This should be the price before taxes,
    but after any and all discounts.
`amount`
    The amount of the order line.
`quantity`
    The quantity being sold in the sales uom of the product.

**Other attributes**

`variant_identifier`
    If there a listing variant_identifier, you could use that in addition to the
    SKU to indicate to fulfil which listing was sold. The variant_identifier allows
    you to have an unique identifier like the external system database ID of the
    product as an identifier (in addition to the SKU).
`description`
    Text description to be printed on the order line.
`list_price`
    The suggested retail price to show discounts against. If no value if given,
    the list_price on the product is automatically picked up.
`comment`
    Line item comments. Usually printed on picking slips for customization like
    engraving.
`metadata`
    A JSON object for additional metadata about the line item
`shipping_date`
    The date on which the items should be shipped. This becomes the planned_date
    of the shipments from this order.
`fulfil_strategy`
    The strategy for fulfilling this line item. Allowed strategies are

    * `ship` (default), 
    * `pick_up` (for in-store pick-ups), 
    * `dropship`,
    * `backorder`
    * `make_on_order`. 

    Learn more about these 
    `strategies here <https://www.fulfil.help/hc/en-us/articles/360011390331>`_.
`gift_message`
    Gift message specific to the line item.
`warehouse`
    ID of the warehouse from which the line item should be fulfilled. It is recommended
    **not** to set this and let Fulfil channels manage this setting.
`delivery_address`
    If the line item is being shipped to different address (from the main shipping address).
    This is only used for multi-destination orders.

**Point of Sale**

If the orders you are creating are POS orders, we recommend changing the channel
source as `POS` on the channel settings and then using the `fulfil_strategy` property
to set what will be shipped from another warehouse and what has been picked up from
the store.

**Gift cards**

These attributes are only required when you are using Fulfil's gift card
features :doc:`outlined here </guides/gift-cards>`.

`recipient_name`
    The full name of the recepient of a gift card.

`recipient_email`
    The email to which digital gift cards should be sent.

.. _tax-lines:

**`tax_lines`**

For each order line, you can also specify the taxes that needs to be applied on
the order line as an array.

Each tax line (in the array) should have the following structure:

.. code-block:: javascript

    {
        "name": "UK Standard VAT",
        "rate": 0.20,
        "amount": 34.45
    }

`name`
    The name of the tax and should match tax names in settings.
`rate`
    The rate of tax. 20% is 0.20, 5% is 0.05.
`amount`
    The amount of tax. Should usually match `unit_price * quantity * rate`

When there are multiple taxes (state, county and city), you can send a tax line for
each of those taxes.


`shipping_lines`
`````````````````

An array of shipping lines. Most orders will have just one line.

.. code-block:: javascript

    [{
        "carrier_service_code": "canada_post_expedited_parcel",
        "description": "Canada Post",
        "amount": 0,
        "channel_identifier": "7848",
        "tax_lines": [],
    }]


`carrier_service_code`
    The code that identifies the carrier and service. This should be setup ahead of
    time on the channel's shipping carrier settings. This avoids having to update
    your code everytime shipping services change.
    This will be deprecated soon, use requested shipping_service attribute on order.
`description`
    Line item description. Usually what the customer would have been displayed
    when the order was placed. For example, "Expedited Shipping" could be what was
    displayed to the customer, while the service could have been *UPS - Next Day Air*.
`amount`
    Amount of shipping. Could be 0 for free shipping.
`channel_identifier`
    If the line item has an ID on the external channel. This is optional.
`tax_lines`
    If shipping is taxable, you can send the list of taxes that apply. This follows
    the same format as :ref:`tax_lines <tax-lines>` on `sale_lines`.

`payments`
```````````

Fulfil can also track payments associated with orders. This could be to track
receivables, to reconcile payments with your payment gateway or for facilitating
refunds by customer service teams within Fulfil.

As of this writing, the payment integration supports shopify payments, stripe,
authorize.net and Braintree.

The payments are sent as a list of payment objects. Most orders should only have
1 payment object.

.. code-block:: javascript

    [
        {
            "amount": 499.45,
            "provider_reference": "3669",
            "method": "stripe",
            "state": "success"
        }
    ]

`provider_reference`
    The transaction ID which will then be used by Fulfil to manage refunds
    or credits.
`amount`
    Amount of payment in the order currency.
`method`
    The code of the method used. This should match the `code` of the payment gateway
    in Fulfil.
`state`
    The state should be `success`. At this point payments in all other states are
    ignored.

**Gift Cards**

If you're using Fulfil powered gift cards in your store, and the method macthes a
gift card gateway, Fulfil also validates that the `channel_identifier` is a valid
gc_number that is active.

`discounts`
```````````

A list of discounts.

.. todo:: To be documented

`currency_code`
````````````````

Three character ISO code of the currency.

`amount`
````````

Total amount of the order. Fulfil uses this to ensure that the computed total of
the order matches the total amount provided. 

`gift_message`
```````````````

The optional `gift_message` to print on the order.

`comment`
```````````

Internal order comments.

This field will be deprecated in future and any comments sent will be added as
a private note on the order.

`requested_shipping_service`
````````````````````````````

The code that identifies the carrier and service. This should be setup ahead of
time on the channel's shipping carrier settings. This avoids having to update
your code everytime shipping services change.

Status Fields
````````````````

There are three status fields Fulfil expects to be sent with each order.

`status`
    The status of the order itself. Valid values are `pending`, `completed` and
    `cancelled`

`financial_status`
    Payment status. Valid values are `paid`, `unpaid` and `refunded`.

`fulfillment_status`
    Status of the fulfillment of the items. Valid values are `unshipped`, 
    `partial` and `shipped`.
