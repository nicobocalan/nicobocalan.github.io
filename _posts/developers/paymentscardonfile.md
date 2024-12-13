---
layout: default
comments: false
title:  "Processing Recurring Payments via Card on File"
---

# Processing Recurring Payments via Card on File

This is similar to charging a card once, except that you take the extra step of creating a customer:

1. Create a customer

2. Add a card to the customer

3. Charge the customer

4. Repeat!

Follow these steps to set up recurring payments on your website with Square. 

In order to process recurring payments via the Connect API, you will need to use the [CreateCustomer](https://docs.connect.squareup.com/api/connect/v2/#endpoint-createcustomer) and [CreateCustomerCard](https://docs.connect.squareup.com/api/connect/v2/#endpoint-createcustomercard) endpoints.

This document includes code samples inline with the text. At the end of the article, all of the code is placed together in one block so that it's more convenient to copy and paste. 


### Make a Square Account

To start, you will need a Square account. Square developer accounts are the same as Square merchant accounts. If you already have a Square account, you're all set.

If you _don't_ already have a Square account, [create one](https://squareup.com/signup?v=developers) to start using the API.

**Important:** We’re going to start by using the sandbox API. However, if you plan to use Square APIs to process card payments, visit [squareup.com/activate](https://www.squareup.com/activate) after you create your account to ensure your account is enabled for payment processing. Otherwise, if you use a non-sandbox API that requires payments activation, you may get error responses that you must activate.


## Step 1: Register an application with Square. 

Now that you have a Square account, visit <https://connect.squareup.com/apps> and sign in.

This is your application dashboard. It's where you register new applications, change your application settings, and find all of your application credentials.

To register your first application:

1. Click New Application.

2. Enter a name for your application and click Create App. (Note that your application name can't include the word Square.)

As soon as your application is created, the control panel for it appears. The control panel consists of four tabs (Credentials, OAuth, Webhooks, and Point of Sale API), each with multiple sections.


## Step 2: Learn about application credentials

The Credentials and OAuth tabs of your application's control panel include a lot of different credentials that can be tricky to keep straight. Here's a quick rundown:


### Personal Access Token

This is found in the Credentials tab of the control panel.

This special token gives you full API access to your _own_ Square account. When you send HTTPS requests to Square API endpoints, you include this value in a header to identify yourself.

If you're building an application just for yourself, this is the only access token you'll ever need to use.


### Application ID

This is found in the Credentials tab of the control panel.

This is your application's unique ID. It's used in a couple places:

- If you're building an application that _other_ merchants will use, you'll specify this value during the OAuth flow (discussed later).

- If you're using the e-commerce API, you'll provide this value when you embed the secure payment form on your website.


### Sandbox Application ID and Access Token

This is found in the Credentials tab of the control panel.

The e-commerce API provides a sandbox that lets you try out processing credit card payments without actually moving any money.

To process e-commerce payments in sandbox mode, you provide these credentials anywhere you would typically provide your standard Application ID and Personal Access Token.


## Step 3. Install the square\_connect gem

You can install the square\_connect by typing in your Terminal

gem install square\_connect


## ********

## Step 4. Retrieve your location IDs

Every Square merchant's business consists of one or more locations, as described [here](https://docs.connect.squareup.com/articles/square-business-structure). In order to process a payment with Connect v2, you need to specify which location is taking the payment. 

In order to process credit cards, you want a location with CREDIT\_CARD\_PROCESSING enabled. **Here’s how to make sure**.

The square\_connect library has an easy method for obtaining a business' location IDs: the list\_locations method.

|                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| require 'square\_connect' access\_token = 'REPLACE\_WITH\_YOUR\_ACCESS\_TOKEN' SquareConnect.configure do \|config\|   config.access\_token = access\_token end locations\_api = SquareConnect::LocationsApi.new begin   locations\_response = locations\_api.list\_locations   puts locations\_response rescue SquareConnect::ApiError => e   raise "Error encountered while listing locations: #{e.message}" end # Get a location able to process transaction location = locations\_response.locations.detect do \|l\|   l.capabilities.include?("CREDIT\_CARD\_PROCESSING") end if location.nil?   raise "Activation required.   Visit https\://squareup.com/activate to activate and begin taking payments." end puts location |


## Step 5. Create a Customer

**Important:** If your application uses customer contact information, be sure to use that information judiciously. **Abuse of customer contact information may result in your application being disabled without notice.**

You can save a new customer for a business with the [CreateCustomer](https://docs.connect.squareup.com/api/connect/v2/#endpoint-createcustomer) endpoint. A valid [Customer ](https://docs.connect.squareup.com/api/connect/v2/#type-customer)object must have at least one of the following:

- Name (first and/or last)

- Email address

- Phone number

- Company name

You can also save the following optional information:

- Physical address

- An informational note about the customer

- A reference ID to associate the customer with an entity in your own system

|                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| require 'square\_connect' require 'securerandom' access\_token = 'REPLACE\_WITH\_YOUR\_ACCESS\_TOKEN' customer\_api = SquareConnect::CustomersApi.new reference\_id = SecureRandom.uuid customer\_request = {   given\_name: 'Amelia',   family\_name: 'Earhart',   email\_address: 'Amelia.Earhart\@example.com',   address: {     address\_line\_1: '500 Electric Ave',     address\_line\_2: 'Suite 600',     locality: 'New York',     administrative\_district\_level\_1: 'NY',     postal\_code: '10003',     country: 'US'   },   phone\_number: '1-555-555-0122',   reference\_id: reference\_id,   note: 'a customer' } begin   customer\_response = customer\_api.create\_customer(access\_token, customer\_request)   puts 'Customer ID to use with CreateCustomerCard:'   puts customer\_response rescue SquareConnect::ApiError => e   raise "Error encountered while creating customer: #{e.message}" end customer = customer\_response.customer |

Step 6. Create a CustomerCard

Once you have created a customer, you need to create a card nonce using the customer’s card information, then add it as a card on file. 

Use the SqPaymentForm to generate a card nonce and provide that card nonce to the[CreateCustomerCard](https://docs.connect.squareup.com/api/connect/v2/#endpoint-createcustomercard) endpoint. Copy and paste the SqPaymentForm in your site. When your visitor submits their card information, the SqPaymentForm will generate the card nonce and submit it to your server.

Copy and paste the following Javascript code into your site files. If you haven’t built your site yet, you can also download a barebones index.html file **here**. 

**Important:** You need the customer's express permission to link their card to their information. For example, you can include a checkbox in your purchase flow (unchecked by default) that the customer can check to specify that they wish to save their card information for future purchases. **Linking cards on file without obtaining customer permission may result in your application being disabled without notice.**

|                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| require 'square\_connect' access\_token = 'REPLACE\_WITH\_YOUR\_ACCESS\_TOKEN' customer\_card\_api = SquareConnect::CustomerCardApi.new customer\_card\_request = {   card\_nonce: CARD\_NONCE,   billing\_address: {     address\_line\_1: '1455 Market St',     address\_line\_2: 'Suite 600',     locality: 'San Francisco',     administrative\_district\_level\_1: 'CA',     postal\_code: '94103',     country: 'US'   },   cardholder\_name: 'Amelia Earhart' } begin   customer\_card\_response = customer\_card\_api.create\_customer\_card(access\_token, customer.id, customer\_card\_request)   puts 'CustomerCard ID to use with Charge:'   puts customer\_card\_response.customer\_card.id rescue SquareConnect::ApiError => e   raise "Error encountered while creating customer card: #{e.message}" end customer\_card = customer\_card\_response.customer\_card |


## Step 7. Charge the CustomerCard

Now that you've generated a card nonce with the SqPaymentForm and you have a way to retrieve a business' location IDs, you can charge a customer's card on file. 

**Important:** Card nonces expire after 24 hours. The CreateCustomerCard endpoint will return an error if you attempt to charge an expired nonce or a nonce that has already been used to charge a card.

You charge a card on file like so:

|                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| require 'square\_connect' require 'securerandom' # Assume you have correct values assigned to the following variables: #   location #   customer #   customer\_card # See the above code samples for how to obtain them. access\_token = 'REPLACE\_WITH\_YOUR\_ACCESS\_TOKEN' transaction\_api = SquareConnect::TransactionApi.new # Every payment you process for a given business hae a unique idempotency key. # If you're unsure whether a particular payment succeeded, you can reattempt # it with the same idempotency key without worrying about double charging # the buyer. idempotency\_key = SecureRandom.uuid  # Monetary amounts are specified in the smallest unit of the applicable currency. # This amount is in cents. It's also hard-coded for $1, which is not very useful. amount\_money = { :amount => 100, :currency => 'USD' } transaction\_request = {   :customer\_id => customer.id,   :customer\_card\_id => customer\_card.id,   :amount\_money => amount\_money,   :idempotency\_key => idempotency\_key } # The SDK throws an exception if a Connect endpoint responds with anything besides 200 (success). # This block catches any exceptions that occur from the request. begin   transaction\_response = transaction\_api.charge(access\_token, location.id, transaction\_request) rescue SquareConnect::ApiError => e   raise "Error encountered while charging card: #{e.message}" Endputs transaction\_response |

The value of transaction\_response is a [hash](https://docs.connect.squareup.com/api/connect/v2/#type-transaction) that contains all of the details of the processed transaction. In the United States, Square also takes care of automatically updating any cards on file that might have expired since you first attached them to a customer.


## Step 8. Repeat!

Once you have a card attached to a customer, you can use the customer information to make the charge. Next time you need to make the charge, use the customer\_id and customer\_card\_id instead of the card nonce. 

    api = squareconnect.apis.transactions_api.TransactionsApi()
    idempotency_key = str(uuid.uuid1())
    api.charge(LOCATION_ID, ChargeRequest(
      idempotency_key=idempotency_key,
      amount_money=Money(150, 'USD'),
      customer_id=customer_id,

      customer _card_id=customer_card_id,
      shipping_address=Address(
        address_line_1='123 Main St',
        locality='San Francisco',
        administrative_district_level_1='CA',
        postal_code='94114',
        country='US'
      ),
      billing_address=Address(
        address_line_1='500 Electric Ave',
        address_line_2='Suite 600',
        administrative_district_level_1='NY',
        locality='New York',
        postal_code='20003',
        country='US'
      ),
      reference_id='optional reference #112358',
      note='optional note'
    ))


## Try it out

A full sample that uses the SqPaymentForm and Rails is available on [Github](https://github.com/square/connect-api-examples/tree/master/connect-examples/v2/rails_payment). See the sample's README for information on running the sample. 

If you want to test out charging cards without actually moving any money, you can configure the sample to communicate with Connect v2 sandbox endpoints. For more information, see [Using the API sandbox](https://docs.connect.squareup.com/articles/using-sandbox).


## Full code

Below you can find all the code samples above in one copy and paste snippet, so that the only variables you need to replace are the access\_token and the card\_nonce.

|                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| require 'square\_connect' require 'securerandom' access\_token = 'REPLACE\_WITH\_YOUR\_ACCESS\_TOKEN' card\_nonce = 'CARD\_NONCE\_FROM\_PAYMENT\_FORM' SquareConnect.configure do \|config\|   config.access\_token = access\_token end locations\_api = SquareConnect::LocationsApi.new begin   locations\_response = locations\_api.list\_locations   puts locations\_response rescue SquareConnect::ApiError => e   raise "Error encountered while listing locations: #{e.message}" end # Get a location able to process transaction location = locations\_response.locations.find do \|l\|   l.capabilities.include?("CREDIT\_CARD\_PROCESSING") end if location.nil?   raise "Activation required.   Visit https\://squareup.com/activate to activate and begin taking payments." end puts location customer\_api = SquareConnect::CustomersApi.new reference\_id = SecureRandom.uuid customer\_request = {   given\_name: 'Amelia',   family\_name: 'Earhart',   email\_address: 'Amelia.Earhart\@example.com',   address: {     address\_line\_1: '500 Electric Ave',     address\_line\_2: 'Suite 600',     locality: 'New York',     administrative\_district\_level\_1: 'NY',     postal\_code: '10003',     country: 'US'   },   phone\_number: '1-555-555-0122',   reference\_id: reference\_id,   note: 'a customer' } begin   customer\_response = customer\_api.create\_customer(customer\_request)   puts 'Customer ID to use with CreateCustomerCard:'   puts customer\_response.customer.id rescue SquareConnect::ApiError => e   raise "Error encountered while creating customer: #{e.message}" end customer = customer\_response.customer customer\_card\_request = {   card\_nonce: card\_nonce,   billing\_address: customer.address.to\_hash,   cardholder\_name: 'Amelia Earhart' } begin   customer\_card\_response = customer\_api.create\_customer\_card(customer.id, customer\_card\_request)   puts 'CustomerCard ID to use with Charge:'   puts customer\_card\_response.card.id rescue SquareConnect::ApiError => e   raise "Error encountered while creating customer card: #{e.message}" end customer\_card = customer\_card\_response.card transactions\_api = SquareConnect::TransactionsApi.new idempotency\_key = SecureRandom.uuid amount\_money = { :amount => 100, :currency => 'USD' } transaction\_request = {   :customer\_id => customer.id,   :customer\_card\_id => customer\_card.id,   :amount\_money => amount\_money,   :idempotency\_key => idempotency\_key } begin   transaction\_response = transactions\_api.charge(location.id, transaction\_request) rescue SquareConnect::ApiError => e   raise "Error encountered while charging card: #{e.message}" end puts transaction\_response |


## Next steps

- **Learn how to set up subscription payments (coming soon!)**

- **Learn how to set up delayed capture transactions.** 

- **Learn about using the sandbox to test your APIs.**

- **Check out the full API reference**

\


Was this page helpful?

 


### **Still haven't found what you are looking for?**

Ask for help on [Stack Overflow ](https://stackoverflow.com/questions/tagged/square-connect)or join our [Slack channel](https://docs.google.com/forms/d/e/1FAIpQLSfAZGIEZoNs-XryKqUoW3atFQHdQw5UqXLMOVPq3V4DEq-AJw/viewform?usp=sf_link).


