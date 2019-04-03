Webhooks
========

Webhooks are a useful tool for apps that want to execute code after a
specific event occurs, for example, after an inventory level change for
a product.

Instead of having your app periodically poll Fulfil for a specific event,
you can register a webhook for the event and create an HTTPS endpoint as
your webhook receiver. Whenever the event occurs, a webhook is sent to
the endpoint. This approach uses less API requests, making it easier to
stay within API call limits while building more robust apps.

Currently supported webhook events are:

* Inventory Level Changes

.. attention::

  Webhooks are currently a Beta feature of Fulfil and is not covered
  by the standard SLAs.

Configuring Webhooks
--------------------

You can configure a webhook using the API or in the Fulfil admin.

Configure a webhook using the API
`````````````````````````````````

You can configure a webhook by making a HTTP POST request to the
Webhook resource in the REST API.

.. code-block:: shell

    curl -X POST \
      https://{merchant}.fulfil.io/api/v1/model/ir.webhook \
      -H 'Content-Type: application/json' \
      -H 'X-API-KEY: {your-api-key}' \
      -d '[
      {
        "event": "product.product.inventory_level_changed",
        "url": "https://hookb.in/03baYwB8DkUGmGkNPzlx"
      }
    ]'


Configure a webhook using the Fulfil admin
``````````````````````````````````````````

If you are developing an app for a specific merchant, then you can
configure your webhooks using the Fulfil admin:

1. From your Fulfil admin, go to **Settings** > **Webhooks**
2. In the **Webhooks** section, click **Create a webhook**
3. Select the event type you want to listen for and the URL where you
   want to receive notifications.

Creating an endpoint for webhooks
---------------------------------

Your endpoint must be an HTTPS webhook address with a valid SSL
certificate that can correctly process event notifications as
described below.

Payloads
````````

Payloads contain a JSON object with the data for the webhook event.
The contents and structure of each payload varies depending on the
subscribed event.


Receiving a webhook
```````````````````

After you register a webhook URL, Fulfil issues a HTTPS POST request
to the URL specified every time that event occurs. The request's POST
parameters contain JSON data relevant to the event that triggered
the request and following HTTP headers -

.. code-block:: shell

    X-Fulfil-Event: contains event name
    X-Fulfil-Organization: organization name

Fulfil verifies SSL certificates when delivering payloads to HTTPS webhook
addresses. Make sure your server is correctly configured to support HTTPS
with a valid SSL certificate.

Responding to a webhook
```````````````````````

Your webhook acknowledges that it received data by sending a 200 OK response.
Any response outside of the 200 range, including 3XX HTTP redirection codes,
indicates that you did not receive the webhook.

Frequency
`````````

Fulfil has implemented a five second timeout period and a retry period for
subscriptions. Fulfil waits five seconds for a response to each request to
a webhook. If there is no response, or an error is returned, then Fulfil
retries the connection 19 times over the next 48 hours.

A webhook is archived if there are 19 consecutive failures.

At the moment, there is no alert sent when a webhook is disabled, but this
will be implemented before the webhooks feature is out of beta.

Verifying webhooks
```````````````````

The verification will be implemented using a digital signature. This will
be available before the webhook is out of Beta. Stay tuned on this page. The
implementation would be HMAC-SHA256 signature as a header on the request.


Best practices
--------------

In the event that your app goes offline for an extended period of time, you
can recover your webhooks by re-registering your webhooks and importing the
missing data.

Stale Data
``````````

Every event body includes a timestamp when the webhook event was triggered.
You can use this to ignore events received out of order for time sensitive
data like inventory levels.

.. note::

  Your app should not rely solely on receiving data from Fulfil webhooks.
  Since webhook delivery is not always guaranteed, you should implement
  reconciliation jobs to periodically fetch data from Fulfil.

  For example, in addition to relying on webhooks for inventory updates,
  you can pull inventory of all products once a day to reconcile any
  missed webhooks.


Payload Examples
----------------

Below are examples for payloads for different webhook events.

Inventory Updates
`````````````````

.. code-block:: javascript

    [
      {
        "timestamp": "2019-03-19T21:34:49.296368",
        "product_code": "SKU",
        "warehouse_quantities": [
          {
            "warehouse_id": 4,
            "quantity_on_hand": 0,
            "quantity_available": 0,
            "warehouse_code": "WAREHOUSE-EAST"
          },
          {
            "warehouse_id": 65,
            "quantity_on_hand": 0,
            "quantity_available": 0,
            "warehouse_code": "WAREHOUSE-WEST"
          },
        ],
        "product_id": 3697,
        "listing_quantities": []
      }
    ]
