Data Warehouse
==============

At Fulfil, we’re constantly searching for innovative ways to improve business 
efficiency and maximize profits for our merchants. Using a data warehouse and 
a business intelligence tool is a great way to improve efficiency with the data
you already have in Fulfil.

While our standard reports could answer many questions, we know that you might 
still need more data specific to your business than what's on those reports.

The data warehouse addon is one way to do this.

What is the data warehouse?
---------------------------

Fulfil's data warehouse is a fully managed, petabyte scale, low cost analytics 
data warehouse. There is no infrastructure to manage and you don't need a database 
administrator—so you can focus on analyzing data to find meaningful insights and use 
familiar SQL.

How do I access this data warehouse?
------------------------------------

When you sign up for the data warehouse addon, our platform team will share the credentials
to access the database.

The credentials include:

1. Hostname
2. Port 
3. Username
4. Password
5. Database Name

You can then connect to the database from any visualization tool.


Data in the data warehouse
--------------------------

The data warehouse contains denormalized tables with commonly used data in
a format that is heavily optimized for building reports. Because Data Warehouses
are mostly used for querying, we try to avoid JOINS at all cost. De-normalizing 
the data model is our attempt at simplifying querying.

Read more about `denormalization <https://rubygarage.org/blog/database-denormalization-with-examples>`_


================= ==========================
Data Source
================= ==========================
Sales             Included on All Plans
Return Shipments  Included on All Plans  
Revenue           Medium plans and above
================= ==========================

Sales
`````

The sales data is denormialized using order lines as the primary table. The table
contains 1 line for every order line in Fulfil. Only confirmed, processing and done
orders appear in the table.

Table Name: `dw_sale_line`

**Columns**

=============================== ==============================================
            Column                           Type                         
=============================== ==============================================
 id                             integer                                   
 quantity                       double precision                          
 line_type                      character varying                         
 amount                         double precision                          
 gross_profit_cpny_ccy_cache    numeric                                   
 cost_price_cpny_ccy_cache      numeric                                   
 untaxed_amount_cpny_ccy_cache  numeric                                   
 sale_id                        integer                                   
 sale_reference                 character varying                         
 sale_number                    character varying                         
 sale_date                      date                                      
 confirmation_time              timestamp(0) without time zone            
 state                          character varying                         
 price_list                     character varying                         
 shipment_address_name          character varying                         
 shipment_address_line1         character varying                         
 shipment_address_line2         character varying                         
 shipment_address_city          character varying                         
 shipment_address_zip           character varying                         
 currency                       character varying(3)                      
 product_code                   character varying                         
 product_name                   character varying                         
 product_category               character varying                         
 product_unit_weight            double precision                          
 brand                          character varying                         
 party_name                     character varying                         
 current_account_manager        character varying                         
 current_sales_rep              character varying                         
 invoice_country_code           character varying(2)                      
 invoice_country_name           character varying                         
 invoice_region_code            character varying                         
 invoice_region_name            character varying                         
 shipment_country_code          character varying(2)                      
 shipment_country_name          character varying                         
 shipment_region_code           character varying                         
 shipment_region_name           character varying                         
 channel_code                   character varying                         
 channel_name                   character varying                         
 sales_person_name              character varying                         
 warehouse                      character varying                         
 return_reason                  character varying                         
 note                           character varying                         
=============================== ==============================================

Return Shipments
````````````````

The returns table displays return shipments (not orders). 

Table Name: `dw_stock_shipment_out_return`

**Columns**

=============================== ==============================================
            Column                      Type           
=============================== ==============================================
 return_id                      integer                         
 return_ref                     character varying               
 return_state                   character varying               
 carrier                        character varying               
 return_shipment_cost           numeric                         
 return_shipment_cost_currency  character varying(3)            
 shelved_date                   date                            
 planned_date                   date                            
 quantity                       double precision                
 cost_price                     numeric                         
 sale_unit_price                numeric                         
 order_id                       integer                         
 order_reference                character varying               
 order_number                   character varying               
 order_date                     date                            
 order_state                    character varying               
 currency                       character varying(3)            
 product_code                   character varying               
 product_name                   character varying               
 product_template_name          character varying               
 product_category               character varying               
 party_name                     character varying               
 invoice_country_code           character varying(2)            
 invoice_country_name           character varying               
 invoice_region_code            character varying               
 invoice_region_name            character varying               
 shipment_country_code          character varying(2)            
 shipment_country_name          character varying               
 shipment_region_code           character varying               
 shipment_region_name           character varying               
=============================== ==============================================

Invoice Lines
``````````````

Invoices are usually generated in Fulfil after the items are shipped. If you are looking
forward to building reports based on revenue (not bookings), then this is the table you will
need.

.. note::

    This is only available on Data Warehouse Medium plans


Table Name: `dw_account_invoice_line`

**Columns**

=============================== ==============================================
        Column                  Type           
=============================== ==============================================
 id                             integer                         
 quantity                       double precision                
 amount                         double precision                
 invoice_id                     integer                         
 invoice_reference              character varying               
 invoice_number                 character varying               
 invoice_date                   date                            
 type                           character varying               
 account_code                   character varying               
 account_name                   character varying               
 invoice_address_name           character varying               
 invoice_address_line1          character varying               
 invoice_address_line2          character varying               
 invoice_address_city           character varying               
 invoice_address_zip            character varying               
 currency                       character varying(3)            
 product_code                   character varying               
 product_name                   character varying               
 product_category               character varying               
 party_name                     character varying               
 invoice_country_code           character varying(2)            
 invoice_country_name           character varying               
 invoice_region_code            character varying               
 invoice_region_name            character varying               
=============================== ==============================================

Limitations
-----------

Read-only
``````````

The Data Warehouse is a read-only data source. You won't be able to write back to the
tables.

An alternative approach would be to use your own custom data warehouse where the data
from Fulfil can be merged into your own custom data warehouse. Below are a few solutions
that can do this for you without having to write any code:

1. `Stitch <https://www.stitchdata.com/integrations/postgresql/>`_
2. `Zapier <https://zapier.com/apps/postgresql/integrations>`_

Custom Fields
`````````````

If you would like to have custom fields included in the data warehouse, you must request
this separately and will be added as a customization for you by the Fulfil professional
service team.