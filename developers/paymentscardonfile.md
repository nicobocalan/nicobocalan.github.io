---
layout: writing_samples
comments: false
title:  "API Reference"
---

# Processing Recurring Payments via Card on File

This is similar to charging a card once, except that you take the extra step of creating a customer:

1. Create a customer
2. Add a card to the customer
3. Charge the customer
4. Repeat!

Follow these steps to set up recurring payments on your website with Square. 

In order to process recurring payments via the Connect API, you will need to use the [CreateCustomer](#) and [CreateCustomerCard](#) endpoints.

This document includes code samples inline with the text. At the end of the article, all of the code is placed together in one block so that it's more convenient to copy and paste. 


### Make a Square Account

To start, you will need a Square account. Square developer accounts are the same as Square merchant accounts. If you already have a Square account, you're all set.

If you _don't_ already have a Square account, [create one](#) to start using the API.

**Important:** We’re going to start by using the sandbox API. However, if you plan to use Square APIs to process card payments, visit [squareup.com/activate](#) after you create your account to ensure your account is enabled for payment processing. Otherwise, if you use a non-sandbox API that requires payments activation, you may get error responses that you must activate.


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

Every Square merchant's business consists of one or more locations, as described [here](#). In order to process a payment with Connect v2, you need to specify which location is taking the payment. 

In order to process credit cards, you want a location with CREDIT\_CARD\_PROCESSING enabled. **Here’s how to make sure**.

The square\_connect library has an easy method for obtaining a business' location IDs: the list\_locations method.

```
    require 'square_connect'

    access_token = 'REPLACE_WITH_YOUR_ACCESS_TOKEN'

    SquareConnect.configure do |config|
    config.access_token = access_token
    end

    locations_api = SquareConnect::LocationsApi.new

    begin
    locations_response = locations_api.list_locations
    puts locations_response
    rescue SquareConnect::ApiError => e
    raise "Error encountered while listing locations: #{e.message}"
    end

    # Get a location able to process transaction
    location = locations_response.locations.detect do |l|
    l.capabilities.include?("CREDIT_CARD_PROCESSING")
    end

    if location.nil?
    raise "Activation required.
    Visit https://squareup.com/activate to activate and begin taking payments."
    end

    puts location
```

## Step 5. Create a Customer

**Important:** If your application uses customer contact information, be sure to use that information judiciously. **Abuse of customer contact information may result in your application being disabled without notice.**

You can save a new customer for a business with the [CreateCustomer](#) endpoint. A valid [Customer ](#)object must have at least one of the following:

- Name (first and/or last)
- Email address
- Phone number
- Company name

You can also save the following optional information:

- Physical address
- An informational note about the customer
- A reference ID to associate the customer with an entity in your own system

```
require 'square_connect'
require 'securerandom'

access_token = 'REPLACE_WITH_YOUR_ACCESS_TOKEN'
customer_api = SquareConnect::CustomersApi.new
reference_id = SecureRandom.uuid

customer_request = {
  given_name: 'Amelia',
  family_name: 'Earhart',
  email_address: 'Amelia.Earhart@example.com',
  address: {
    address_line_1: '500 Electric Ave',
    address_line_2: 'Suite 600',
    locality: 'New York',
    administrative_district_level_1: 'NY',
    postal_code: '10003',
    country: 'US'
  },
  phone_number: '1-555-555-0122',
  reference_id: reference_id,
  note: 'a customer'
}

begin
  customer_response = customer_api.create_customer(access_token, customer_request)
  puts 'Customer ID to use with CreateCustomerCard:'
  puts customer_response
rescue SquareConnect::ApiError => e
  raise "Error encountered while creating customer: #{e.message}"
end

customer = customer_response.customer
```

## Step 6. Create a CustomerCard

Once you have created a customer, you need to create a card nonce using the customer’s card information, then add it as a card on file. 

Use the SqPaymentForm to generate a card nonce and provide that card nonce to the[CreateCustomerCard](#) endpoint. Copy and paste the SqPaymentForm in your site. When your visitor submits their card information, the SqPaymentForm will generate the card nonce and submit it to your server.

Copy and paste the following Javascript code into your site files. If you haven’t built your site yet, you can also download a barebones index.html file **here**. 

**Important:** You need the customer's express permission to link their card to their information. For example, you can include a checkbox in your purchase flow (unchecked by default) that the customer can check to specify that they wish to save their card information for future purchases. **Linking cards on file without obtaining customer permission may result in your application being disabled without notice.**

```
require 'square_connect'

access_token = 'REPLACE_WITH_YOUR_ACCESS_TOKEN'

customer_card_api = SquareConnect::CustomerCardApi.new
customer_card_request = {
  card_nonce: CARD_NONCE,
  billing_address: {
    address_line_1: '1455 Market St',
    address_line_2: 'Suite 600',
    locality: 'San Francisco',
    administrative_district_level_1: 'CA',
    postal_code: '94103',
    country: 'US'
  },
  cardholder_name: 'Amelia Earhart'
}

begin
  customer_card_response = customer_card_api.create_customer_card(access_token, customer.id, customer_card_request)
  puts 'CustomerCard ID to use with Charge:'
  puts customer_card_response.customer_card.id
rescue SquareConnect::ApiError => e
  raise "Error encountered while creating customer card: #{e.message}"
end

customer_card = customer_card_response.customer_card


```

## Step 7. Charge the CustomerCard

Now that you've generated a card nonce with the SqPaymentForm and you have a way to retrieve a business' location IDs, you can charge a customer's card on file. 

**Important:** Card nonces expire after 24 hours. The CreateCustomerCard endpoint will return an error if you attempt to charge an expired nonce or a nonce that has already been used to charge a card.

You charge a card on file like so:

```
require 'square_connect'
require 'securerandom'

# Assume you have correct values assigned to the following variables:
#   location
#   customer
#   customer_card
# See the above code samples for how to obtain them.

access_token = 'REPLACE_WITH_YOUR_ACCESS_TOKEN'

transaction_api = SquareConnect::TransactionApi.new

# Every payment you process for a given business hae a unique idempotency key.
# If you're unsure whether a particular payment succeeded, you can reattempt
# it with the same idempotency key without worrying about double charging
# the buyer.

idempotency_key = SecureRandom.uuid


# Monetary amounts are specified in the smallest unit of the applicable currency.
# This amount is in cents. It's also hard-coded for $1, which is not very useful.

amount_money = { :amount => 100, :currency => 'USD' }

transaction_request = {
  :customer_id => customer.id,
  :customer_card_id => customer_card.id,
  :amount_money => amount_money,
  :idempotency_key => idempotency_key
}

# The SDK throws an exception if a Connect endpoint responds with anything besides 200 (success).
# This block catches any exceptions that occur from the request.
begin
  transaction_response = transaction_api.charge(access_token, location.id, transaction_request)
rescue SquareConnect::ApiError => e
  raise "Error encountered while charging card: #{e.message}"
End

puts transaction_response

```

The value of transaction_response is a [hash](#) that contains all of the details of the processed transaction. In the United States, Square also takes care of automatically updating any cards on file that might have expired since you first attached them to a customer.


## Step 8. Repeat!

Once you have a card attached to a customer, you can use the customer information to make the charge. Next time you need to make the charge, use the customer\_id and `customer_card_id` instead of the card nonce. 

```
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
```



## Try it out

A full sample that uses the SqPaymentForm and Rails is available on [Github](#). See the sample's README for information on running the sample. 

If you want to test out charging cards without actually moving any money, you can configure the sample to communicate with Connect v2 sandbox endpoints. For more information, see [Using the API sandbox](#).


## Full code

Below you can find all the code samples above in one copy and paste snippet, so that the only variables you need to replace are the access\_token and the card\_nonce.


```
require 'square_connect'
require 'securerandom'

access_token = 'REPLACE_WITH_YOUR_ACCESS_TOKEN'
card_nonce = 'CARD_NONCE_FROM_PAYMENT_FORM'

SquareConnect.configure do |config|
  config.access_token = access_token
end

locations_api = SquareConnect::LocationsApi.new

begin
  locations_response = locations_api.list_locations
  puts locations_response
rescue SquareConnect::ApiError => e
  raise "Error encountered while listing locations: #{e.message}"
end

# Get a location able to process transaction
location = locations_response.locations.find do |l|
  l.capabilities.include?("CREDIT_CARD_PROCESSING")
end

if location.nil?
  raise "Activation required.
  Visit https://squareup.com/activate to activate and begin taking payments."
end

puts location

customer_api = SquareConnect::CustomersApi.new
reference_id = SecureRandom.uuid

customer_request = {
  given_name: 'Amelia',
  family_name: 'Earhart',
  email_address: 'Amelia.Earhart@example.com',
  address: {
    address_line_1: '500 Electric Ave',
    address_line_2: 'Suite 600',
    locality: 'New York',
    administrative_district_level_1: 'NY',
    postal_code: '10003',
    country: 'US'
  },
  phone_number: '1-555-555-0122',
  reference_id: reference_id,
  note: 'a customer'
}

begin
  customer_response = customer_api.create_customer(customer_request)
  puts 'Customer ID to use with CreateCustomerCard:'
  puts customer_response.customer.id
rescue SquareConnect::ApiError => e
  raise "Error encountered while creating customer: #{e.message}"
end

customer = customer_response.customer

customer_card_request = {
  card_nonce: card_nonce,
  billing_address: customer.address.to_hash,
  cardholder_name: 'Amelia Earhart'
}

begin
  customer_card_response = customer_api.create_customer_card(customer.id, customer_card_request)
  puts 'CustomerCard ID to use with Charge:'
  puts customer_card_response.card.id
rescue SquareConnect::ApiError => e
  raise "Error encountered while creating customer card: #{e.message}"
end

customer_card = customer_card_response.card

transactions_api = SquareConnect::TransactionsApi.new
idempotency_key = SecureRandom.uuid
amount_money = { :amount => 100, :currency => 'USD' }

transaction_request = {
  :customer_id => customer.id,
  :customer_card_id => customer_card.id,
  :amount_money => amount_money,
  :idempotency_key => idempotency_key
}

begin
  transaction_response = transactions_api.charge(location.id, transaction_request)
rescue SquareConnect::ApiError => e
  raise "Error encountered while charging card: #{e.message}"
end

puts transaction_response

```



## Next steps

- **Learn how to set up subscription payments**
- **Learn how to set up delayed capture transactions.** 
- **Learn about using the sandbox to test your APIs.**
- **Check out the full API reference**


Was this page helpful?


### **Still haven't found what you are looking for?**

Ask for help on [Stack Overflow ](#)or join our [Slack channel](#).


