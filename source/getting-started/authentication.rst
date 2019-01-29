Authentication
==============

Before you can interact with the Fulfil API, your app must provide the 
necessary authentication credentials in each HTTP request that you
make to Fulfil. The way to provide these credentials depends on the 
type of app that you're developing. 


Fulfil API offers two forms of authentication:

API Key / Personal Access Token
-------------------------------

Also know as the V1 authentication, this allows users to create an
API key associated to their user account to build private applications.

All activity performed using this API key will reflect as though the
user performed the action.

Use this for personal bots that need to do something with Fulfil on
behalf of you.

On the other hand, if you are building an app that is likely to be used
by other users in your company or publicly by other Fulfil merchants, you
should use the OAuth2 authentication system instead.

.. attention::

   Treat the API tokens like you would any other password,
   since whoever has access to these credentials has full API access
   to the platform.

   * Delete any unused tokens.
   * Delete a token at once if you suspect it's been compromised and
     create another one if necessary.

Access tokens are managed in the User Preferences. Click on your username
on the top right and then preferences. Click on Personal Access tokens.
The page lets you view, add, or delete tokens. More than one token can be
active at the same time. Deleting a token deactivates it permanently.

To use personal access tokens as authorization on requests, you should send
the token in the `X-API-KEY` header.

.. code-block:: shell

    curl -X GET \
    https://{merchant}.fulfil.io/api/v1/model/company.company \
    -H 'Content-Type: application/json' \
    -H 'X-API-KEY: {your-api-key}'

OAuth access token
------------------

OAuth2 an open protocol to allow secure authorization in a simple and
standard method from web, mobile and desktop applications.

An app built with OAuth2 can interact with the Fulfil API on behalf of
multiple users (within an organization) or on behalf of multiple Fulfil
merchants.

:doc:`OAuth2 Implementation Guide </getting-started/oauth2>`

In your requests, specify the access token in an Authorization header as
follows:

`Authorization: Bearer token-here`

.. code-block:: shell

    curl -X GET \
    https://{merchant}.fulfil.io/api/v2/model/company.company \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer {token}'

.. tip::

    Note that the version on the endpoint is v2

V1 vs V2 API
------------

API version V2 is completely backwards compatioble with V1 (except for authentication).
Only V2 supports Oauth (and hence access tokens and offline access), while V1 only supports
personal access tokens.

Adding personal access token support for V2 API is on the roadmap.