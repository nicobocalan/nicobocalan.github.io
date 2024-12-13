---
layout: writing_samples
comments: false
title:  "API Reference"
description: "<div width=50>This is my writing sample that I wrote for my writing bonk <b>bonk</b></div>"
---

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
