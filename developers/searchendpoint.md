---
layout: writing_samples
comments: false
title:  "API Reference"
description: "<div width=50>This is my writing sample that I wrote for my writing bonk <b>bonk</b></div>"
---

# Searching Customers and Responses
You can use this API to search through your survey invitations and responses.

## Search Customer Endpoint
```curl
/app/customers/_search
```
Searches through your feedback and returns up to 50 search results.

## Search Responses Endpoint
```
/app/responses/_search
```
Searches through your responses and returns up to 50 records at a time.

## Optional Query Parameters
If you expect more than 50 records, you can optionally add the following parameters to paginate your search results.


<table class="table">
   <tr>
      <th>Name</th>
      <th>Required/Optional</th>
      <th>Description</th>
   </tr>
   <tr>
      <td>
         <strong>limit</strong>
         <br><em>number</em>
      </td>
      <td>
         Optional
      </td>
      <td>
         The maximum number of records returned per response. This is capped at 100. If you pass a value larger than 100, we will still return only up to 100 records at a time.  If not included, the limit will default to 50 records.
      </td>
    </tr>
    <tr>
       <td>
          <strong>offset</strong>
          <br><em>number</em>
       </td>
       <td>
          Optional
       </td>
       <td>
          Offset determines which set of records is returned in case of pagination. For example, if you make a request for the first 100 records, you should offset your next request by 100 so you can get responses 100-199 in your next response.  If not included, the offset will default to 0, which denotes the first set of records.
       </td>
     </tr>
</table>


#### Sample Search Customer Endpoint
```
/app/customers/_search?offset=0&limit=250
```
Returns the first 25 records.

#### Search Search Responses Endpoint
```
/app/responses/_search?offset=25&limit=50
```
Returns records 26 through 75.


## JSON Fields Array
By default, the response data in your search results will include every field that has a value. You can use the fields array to narrow down the fields you want to retrieve.

**Note:** All of the responses returned in your search results will have an ID number and a link to that person’s contact page in the app.

All of the responses returned in your search results will have an ID number and a link to that person’s contact page in the app. You can also specify which additional values you want to see for each response by including  a **fields array** in your JSON payload.

A fields array should include the contact fields you want to see for each response. For example, ```invitation_sent_date```, ```email```, and ```first_name```. If you include these in your fields array, each response will return only the values for these fields in addition to the ID number and the contact page link.

#### cURL Sample
This cURL command should return the first two responses in the search results, with the first_name, last_name, and email of each response.

```curl
curl -X POST -H "domain: npxacme" 
-H "Authorization: Basic Li9K1y8oaL6x7Li65f" 
-H "Content-Type: application/json" -d '{
    "fields": [
        "first_name",
        "last_name",
        "email"
        ]
}' "https://surveyland.com/app/responses/_search&limit=2"
```

#### Sample Response

```json
{
    "data": [
        {
            "id": "1459",
            "link": "https://surveyland.com/customers/1459369528080",
            "first_name": "John",
            "invitation_sent_date": "08/11/2016 01:00:08",
            "email": john.cruz@acme.com
            },
        {
            "id": "1358",
            "link": "https://surveyland.com/customers/1358869524216",
            "first_name": "Kim",
            "invitation_sent_date": "06/15/2016 08:30:68",
            "email": Kim.Li@acme.com
            },
    ],
}
```
