---
layout: writing_samples
comments: false
title:  "API Reference"
description: "<div width=50>This is my writing sample that I wrote for my writing bonk <b>bonk</b></div>"
---

# Frontend and Backend Development

A typical application has a frontend (client component) and a backend (server component).

- The frontend refers to the user interface that receives user input. It is built using technologies such as HTML, CSS, and JavaScript for web applications and Objective-C, Swift, Java, or Kotlin for iOS and Android applications.

- The backend works behind the scenes. It receives user input, pulls the necessary data, and sends the data back to the frontend.

Square provides API libraries that developers embed when developing an application frontend and backend.


### **Frontend**

Developers design and construct the user experience on the web page or a mobile application using buttons, menus, links, graphics, and other elements.

A typical payment-processing application flow starts with a buyer providing payment information in the frontend UI (for example, providing card data in UI text boxes). Square provides client-side libraries to process payments. These libraries typically provide UI elements to embed in the application frontend for collecting payment information.

Square supports payment processing across multiple channels, including online, in-app, and in-person (see[ Payments).](https://developer.squareup.com/docs/payments) Square provides client-side libraries for these scenarios.

Some of these client-side libraries must be used in conjunction with the server-side[ Square API](https://developer.squareup.com/reference/square). These include:

- [Online payment](https://developer.squareup.com/docs/payments#online-payments) processing. Developers can use the Square Web Payments SDK to process payments in web applications. This SDK generates a payment token from the payment information that users provide. Developers must implement a backend that calls the Square Payments API to process payments using payment tokens as the payment source. For an example application, see[ Payment Processing example](https://github.com/square/connect-api-examples/tree/master/connect-examples/v2/node_payment) on GitHub.

- [In-app payment](https://developer.squareup.com/docs/payments#in-app-payments) processing. This refers to seller-created mobile applications that buyers install on their Android or iOS devices. These applications embed the Square In-App Payments SDK in the frontend development. This SDK generates a secure payment token. The application must provide a server (backend) that calls the Payments API to take the payment. For an example application, see[ In-App Payments SDK: Quickstart.](https://developer.squareup.com/docs/in-app-payments-sdk/quick-start/start)\
  Square also provides open source plugins,[ Flutter plugin](https://github.com/square/in-app-payments-flutter-plugin) and[ React Native plugin](https://github.com/square/in-app-payments-react-native-plugin), for Square In-App Payment SDK.

Square also provides client-side libraries to process[ in-person payments.](https://developer.squareup.com/docs/payments#in-person-payments) Because these libraries process payments end to end, your application backend does not need to explicitly call the Square Payments API to process payments. These include the:

- **Reader SDK.** A developer embeds the Reader SDK in their mobile application frontend to process payments. In this case, the application uses the Reader SDK connected to their mobile device to process payments. Note that the Reader SDK is available only in the United States. For an example, see[ Reader SDK: Quickstart.](https://developer.squareup.com/docs/reader-sdk/quick-start/start)\
  Square also provides the open source plugins,[ Flutter plugin](https://github.com/square/reader-sdk-flutter-plugin) and[ React Native plugin](https://github.com/square/react-native-square-reader-sdk), for Square Reader SDK.

- **Point of Sale API.** To process in-person payments using Square hardware, developers can embed the Point of Sale API in their mobile application. The API simply acts like an application switcher. When time comes to process a payment, the API switches seamlessly to the Square Point of Sale application and collects the payment with a Square Reader. Applications do not need to make any backend Square API calls. For examples, see[ Point of Sale API: Overview.](https://developer.squareup.com/docs/pos-api/what-it-does)


### **Backend**

[Square APIs](https://developer.squareup.com/reference/square) (and the corresponding wrapper[ SDKs](https://developer.squareup.com/docs/sdks)) integrate with backend development. Your application server sends these API requests to Square and processes responses from Square. For example:

- **Bookings API.** Applications can use the Bookings API to allow buyers to manage bookings with a Square seller. The application backend makes the necessary Bookings API calls to create, update, and delete bookings. For an example application, see[ Bookings API example](https://github.com/square/connect-api-examples/tree/master/connect-examples/v2/node_bookings) on GitHub.

- **Subscriptions API.** Applications can use the Subscriptions API to offer subscription services to buyers. These applications make calls to Square APIs (such as the Catalog API or Subscriptions API) in their backend code to set up a plan and to create and manage subscriptions. For an example application, see[ Subscriptions API example](https://github.com/square/connect-api-examples/tree/master/connect-examples/v2/node_subscription) on GitHub.

Square backend APIs can be used with any programming language. Square supports multiple languages with SDKs and any language can call Square APIs directly.

# Idempotency

Learn about idempotency and how idempotency keys prevent unintended results from accidental duplicate requests.

[What is idempotency?](https://developer.squareup.com/docs/build-basics/common-api-patterns/idempotency#what-is-idempotency)

[Additional Square API patterns](https://developer.squareup.com/docs/build-basics/common-api-patterns/idempotency#additional-square-api-patterns)

[See also](https://developer.squareup.com/docs/build-basics/common-api-patterns/idempotency#see-also)

[](https://developer.squareup.com/docs/build-basics/common-api-patterns/idempotency#what-is-idempotency)

[]()


## What is idempotency?

An API call or operation is idempotent if it has the same result no matter how many times it's applied. That is, an idempotent operation provides protection against accidental duplicate calls causing unintended consequences.

For example, suppose `CreatePayment` didn't require idempotency and you make a successful `CreatePayment` request (Square successfully processes the payment). However, your connection fails before you receive success confirmation from Square. You don't know what happened to your request and therefore you make the request again, charging the customer twice.

Square supports idempotency by allowing API operations to provide an idempotency key (a unique string) to protect against accidental duplicate calls that can have negative consequences.

In the case of the preceding `CreatePayment` example:

- If you make the same `CreatePayment` request with the same idempotency key again, the endpoint knows it's a duplicate request. It doesn't take the payment again. Instead of getting an error, the endpoint returns the response as the first successful `CreatePayment` response.

- If you use the same idempotency key but change the `CreatePayment` request (for example, specify a different payment amount), you get an error indicating that you used the idempotency key previously. Note that this behavior might vary depending on the API.

Idempotency keys can be anything, but they need to be unique. Virtually all popular programming languages provide a function for generating unique strings. You should use one of these language calls.

| Language | Recommended function                                                                                                |
| -------- | ------------------------------------------------------------------------------------------------------------------- |
| Ruby     | [SecureRandom.uuid](https://ruby-doc.org/stdlib-2.5.0/libdoc/securerandom/rdoc/Random/Formatter.html#method-i-uuid) |
| PHP      | [uniqid](http://php.net/manual/en/function.uniqid.php)                                                              |
| Java     | [UUID.randomUUID](http://docs.oracle.com/javase/7/docs/api/java/util/UUID.html)                                     |
| Python   | [uuid](https://docs.python.org/3.1/library/uuid.html)                                                               |
| C#       | [Guid.NewGuid](https://msdn.microsoft.com/en-us/library/system.guid.newguid\(v=vs.110\).aspx)                       |
| Node.js  | [crypto.randomUUID](https://nodejs.org/api/crypto.html#cryptorandomuuidoptions)                                     |

[](https://developer.squareup.com/docs/build-basics/common-api-patterns/idempotency#additional-square-api-patterns)

[]()


## Additional Square API patterns

There are other Square API patterns. For more information, see [Common Square API Patterns](https://developer.squareup.com/docs/build-basics/common-api-patterns).

[](https://developer.squareup.com/docs/build-basics/common-api-patterns/idempotency#see-also)

[]()


## See also

- [Video: Sandbox Short - Idempotency](https://www.youtube.com/watch?v=J9jkEdNo5F8)
- [Video: The Sandbox - Idempotency](https://www.youtube.com/watch?v=Hx8UdQx_4bE\&t=67s)

&#x20; &#x20;


### Was this page helpful?

&#x20;

If you need more assistance, contact [Developer and App Marketplace Support](https://squareup.com/help/contact?panel=BF53A9C8EF68) or ask for help in the [Developer Forums](https://developer.squareup.com/forums).

Development

&#x20;

- [Guides](https://developer.squareup.com/docs)&#x20;
- [API Reference](https://developer.squareup.com/reference/square)&#x20;
