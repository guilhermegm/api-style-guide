# API Style Guide

### HTTP Methods

| Method | Description |
| ------ | ------ |
| GET | Retrieve a resource or a list of resources. |
| POST | Create a resource, or execute a complex operation on a resource. |
| PUT | Update a resource. |
| PATCH | Perform a partial update to a resource. |
| DELETE | Delete a resource. |

### HTTP Headers

| HTTP Header Name	| Description |
| --- | --- |
| [Accept](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept) | This header advertises which content types, expressed as MIME types, the client is able to understand. It is assumed that APIs support application/json. |
| [Accept-Charset](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Charset) | This header advertises which character set the client is able to understand. This value SHOULD include utf-8. |
| [Authorization](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization) | This header contains the credentials to authenticate a user. For example: Bearer {JWT Token} |
| [Content-Language](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Language) |  This entity header is used to describe the language(s) intended for the audience. APIs must provide this header in the response. Example: Content-Language: pt-BR |
| [Content-Type](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type) | This entity header is used to indicate the media type of the resource. This value should include application/json. |

### API Versioning

```
GET api.myservice.com/v1/customers
```

### Endpoints

The endpoint URLs should be in the following format:

```
GET /v1/resource
GET /v1/resource/:id
GET /v1/resource/:id/subresource
GET /v1/resource/:id/subresource/:id
```

for composite names use [kebab-case](https://en.wikipedia.org/wiki/Naming_convention_(programming)#Multiple-word_identifiers).

Use plural nouns for resources and subresources.

```
GET /v1/customers
GET /v1/customers/1e92EA/addresses
GET /v1/customers/1e92EA/addresses/321e5EA
```

The IDs in the endpoint URLs should not be sequential.

Trailing slash is optional.

### Use Envelopes

Use envelope to avoid some [potentials hacks](http://haacked.com/archive/2009/06/25/json-hijacking.aspx/). Click here to [see more](https://medium.com/studioarmix/learn-restful-api-design-ideals-c5ec915a430f).

### Letter Case

For the JSON use snake_case or camelCase as you prefer. Snake case is 20% easier to read but it is up to you.

### Errors

#### Status Codes

| Codes | Description |
| ------ | ------ |
| **Standards** |  |
| 200 OK | When everything is okay. |
| 201 Created | Response to POST method execution to indicate successful creation of a resource. |
| 202 Accepted | Asynchronous method execution to specify the server has accepted the request and will execute it at a later time. |
| 204 No Content | When everything is okay, but there’s no content to return. |
| 405 Method Not Allowed | The server has not implemented the requested HTTP method. |
| 406 Not Acceptable | When it cannot return the payload of the response using the media type requested by the client. For example, if the client sends an Accept: application/xml header, and the API can only generate application/json. |
| 415 Unsupported Media Type | When the media type of the request's payload cannot be processed. For example, if the client sends a Content-Type: application/xml header, but the API can only accept application/json. |
| 500 Internal Server Error | When the server throws a completely unexpected error. |
| **Data Errors** | 
| 400 Bad Request | When the requested information is incomplete or malformed. |
| 404 Not Found | When everything is okay, but the resource doesn’t exist. |
| 409 Conflict | When a conflict of data exists, even with valid information. |
| 422 Unprocessable Entity | When the requested information is okay but invalid. |
| **Auth Errors** |  |
| 401 Unauthorized | When an access token isn't provided or is invalid. |
| 403 Forbidden | When an access token is valid, but requires more privileges. |
Sources:
[Paypal - API Design Guidelines](https://github.com/paypal/api-standards/blob/master/api-style-guide.md#date-common-types)
[RESTful API Design Tips from Experience](https://medium.com/studioarmix/learn-restful-api-design-ideals-c5ec915a430f)

*Data errors examples:*

```
If the email field is missing, return a 400.
If the password field is too short, return a 422.
If the email field isn’t a valid email, return a 422.
If the email is already taken, return a 409.
```

#### Error Format

```
422 Unprocessable Entity
{
  "data": {
    "error": "FIELDS_VALIDATION_ERROR",
    "description": "One or more fields raised validation errors."
    "fields": {
      "email": "Invalid email address.",
      "password": "Password too short."
    }
  }
}
```

```
409 Conflict
{
  "data": {
    "error": "EMAIL_ALREADY_EXISTS",
    "description": "An account already exists with this email."
  }
}
```

### Pagination

Use query string parameter **page** starting at **1** to return a limited number of results. The **page_size** query string has a maximum of **1000** and default page_size of **20** for the pagination.

```
GET /v1/customers/?page=5&page_size=300
```

### Sort

Use **sort_by** query string parameters to order the resources as specificed by the client.

```
GET /v1/customers/?sort_by=name,weight,-age
```

```
// an SQL example
SELECT name, weight, age FROM customers ORDER BY name ASC, weight ASC, age DESC;
```

The fields are separeted by commas and their order should be respected. The minus symbol denotes DESC order and no minus symbol denotes ASC order.

### Date, Time and Timezone

The date and time string MUST conform to the date-time universal format defined in section 5.6 of [RFC3339](https://www.ietf.org/rfc/rfc3339.txt).

All APIs MUST only emit UTC time (aka Zulu time or GMT) in the responses.

When processing requests, an API SHOULD accept full-date, date-time or full-time fields that contain an offset from UTC. For example, 2016-09-28T18:30:41.000+05:00 SHOULD be accepted as equivalent to 2016-09-28T13:30:41.000Z.

(PayPal - Date, Time and Timezone)[https://github.com/paypal/api-standards/blob/master/api-style-guide.md#date-time-and-timezone]

### Fetching Data

#### POST

Create a new resource and return the resource object.

```
POST /v1/customers
{
  "name": "Vando",
  "email": "vando@bee"
}

201 Created
{
  "id": "1e92EA",
  "name": "Vando",
  "email": "vando@bee"
}
```

#### GET (single resource)

Get a single resource using the ID in the URL query string.

```
GET /v1/customers/1e92EA

200 OK
{
  "data": {
    "id": "1e92EA",
    "name": "Vando",
    "email": "vando@bee"
  }
}
```

#### GET (resource list)

Get a list of resources plus the meta information about this list such as total items and pages.

```
GET /v1/customers

200 OK
{
  "data": {
    "meta": {
      "total_items": 100,
      "total_pages": 10
    },
    "items": [
        { ... },
        ...
    ]
  }
}
```

#### PATCH

Update partially a single resource.

```
PATCH /v1/customers/1e92EA
{
  "name": "Vando Almeida"
}

204 No Content
```

#### DELETE

Delete a single resource.

```
DELETE /v1/customers/1e92EA

204 No Content
```

### Avoid Unnecessary Query Strings

[RESTful API Design Tips from Experience
 - Avoid Unnecessary Query Strings](https://medium.com/studioarmix/learn-restful-api-design-ideals-c5ec915a430f)

### Implement a “Health-Check” Endpoint

A simple endpoint that can indicate if your API instance is alive and does not need to be restarted. It’s also useful for easily checking what version of the API is on any machine at any time, without authentication.

```
GET /

200 OK
{
  "version": "fdb1d5e"
}
```

[See more](https://medium.com/studioarmix/learn-restful-api-design-ideals-c5ec915a430f)

### Request ID

You should beware. X-Request-ID support means that you can easily trace an HTTP request all the way from a client to your backend web processes (via our proxies).

[See more](https://atech.blog/viaduct/x-request-id)

### Bulk Operations

Used for a single request with a list of resources.

This kind of requisition should be [atomic](https://en.wikipedia.org/wiki/Atomicity_(database_systems)).

#### Request Format

Recommended to use bulk at the end of the URL.

```
POST /v1/customers/bulk

{
    items: [
        { ... },
        { ... },
        { ... },
        ...
    ]
}

200 OK
{
    batch_result: [
        { < Success Response > },
        { < Success Response > },
        { < Error Response > },
        { < Error Response > }
    ]
}
```

### Internationalization

| Type | Description |
| --- | --- |
| Country Code | APIs must use the [ISO 3166-1 alpha-2](http://www.iso.org/iso/country_codes.htm) two letter country code standard. |
| Currency Code | Currency type must use the three letter currency code as defined in [ISO 4217](http://www.currency-iso.org/). For quick reference on, see [ISO_4217](http://en.wikipedia.org/wiki/ISO_4217). |
| Language Code | IETF specified by RFC 5646 and RFC 4647. For quick reference on, see [Microsoft - Language Codes](https://msdn.microsoft.com/pt-br/library/ms533052(v=vs.85).aspx). |

[PayPal - Internationalization](https://github.com/paypal/api-standards/blob/master/api-style-guide.md#internationalization)

### GZIP

Compress your response using GZIP, it will save time and bytes.

See more:
[How much time should you save?](https://stackoverflow.com/a/27319309)
[Best Practices for Designing a Pragmatic RESTful API](http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api#pretty-print-gzip)

How to GZIP:
[nginx - Enable GZIP](https://easyengine.io/tutorials/nginx/enable-gzip/)


### Reefers

[RESTful API Design Tips from Experience
](https://medium.com/studioarmix/learn-restful-api-design-ideals-c5ec915a430f)

[PayPal - API Design Patterns And Use Cases](https://github.com/paypal/api-standards/blob/master/patterns.md)

[PayPal - API Design Guidelines](https://github.com/paypal/api-standards/blob/master/api-style-guide.md)

[Best Practices for Designing a Pragmatic RESTful API](http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api)
