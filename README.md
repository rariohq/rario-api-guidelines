# REST API Guidelines

#### References:
1. https://tools.ietf.org/html/rfc7231
2. https://opensource.zalando.com/restful-api-guidelines
3. https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md
4. https://pepa.holla.cz/wp-content/uploads/2016/01/REST-API-Design-Rulebook.pdf

## 1 Consistency fundamentals

### 1.1 REST Resource Naming

##### Use snake_case for Query Parameters
Examples:
```
order_nr, invoice_nr, awb_nr

GET /sales-orders?order_nr=SA9103D3921

```

##### Prefer Hyphenated-Pascal-Case for HTTP header Fields
Standard Headers Examples:
```
Content-Type
Accept-Encoding
Accept-Language
```

**Custom Headers Examples:**
```
M-Platform: ios or android or web
M-Build: ??
M-Client-Version: ??
M-Locale: en_ae or ar_ae or en_sa or ar_sa   [Pattern : {language}_{country}]
M-Theme: full_width
```


##### Pluralize Resource Names
Examples:
```
/customers
/products
/carts
/sales-orders
/sales-order-shipments
```

##### Use Domain-Specific Resource Names
Use domain-specific nomenclature for resource names helps developers to understand the functionality and basic 
semantics of your resources.
Examples:
```
/sales-orders
/sales-order-items
/sales-order-shipments
```
Here "sales-order-items" is superior to "order-items" in that it clearly indicates which business object it represents.

##### Use sub-resources for relations
If a resource is related to another resource use sub resources
```
/{resources}/[resource-id]/{sub-resources}/[sub-resource-id]
```

Examples:
```
GET /carts/1/items - Returns a list of items for cart 1
GET /carts/1/items/1 - Returns item #1 for cart 1
GET /customers/1/configuration - There was only one configuration per customer
```

##### Stick to Conventional Query Parameters
* **q** — default query parameter (e.g. used by browser tab completion); should have an entity specific alias, like sku
* **sort** — comma-separated list of fields to define the sort order. To indicate sorting direction, fields may be 
prefixed with + (ascending) or - (descending), ```e.g. /sales-orders?sort=-created_at,+grand_total```
* **fields** - to retrieve only a subset of fields of a resource 
```
e.g. GET /customers/1?fields=(name,phone_numbers(country_code,number,type))

{
  "name": "John Doe",
  "phone_numbers": [{
    "country_code": "+971",
    "number": "5391829",
    "type": "Home"
  }]
}
``` 
* **embed** - to expand or embed sub-entities
```
e.g. GET /customers/1?embed=[addresses(address1,area,city,country_code)]
{
  "customer_id": 1,
  "addresses": [
    {
      "address1": "Tower 1",
      "area": "City Centre Deira",
      "city": "dubai",
      "country_code": "UAE"
    }
  ]
}
```
* **offset** numeric offset of the first element on a page
* **limit** number of entries on a page

### 1.2 URL structure

##### Basic URL structure:

```
/{resources}/[resource-id]/{sub-resources}/[sub-resource-id]
/{resources}/[partial-id-1][separator][partial-id-2]

```

Examples:

```
/carts/1/items
/carts/1/items/1
/customers/1/addresses/1
/content/images/78123eg3c090e83494a2w512
/customers/1/orders/1/items
/customers/1/orders/1/items/1
```

##### URL length
The HTTP 1.1 message format, defined in RFC 7230, in section 3.1.1, defines no length limit on the Request Line, which 
includes the target URL
```
HTTP does not place a predefined limit on the length of a request-line. A server that receives a request-target longer 
than any URI it wishes to parse MUST respond with a 414 (URI Too Long) status code.
```

##### Avoid Trailing Slashes
As the last character within a URI’s path, a forward slash (/) adds no semantic value and may cause confusion. REST APIs 
should not expect a trailing slash and should not include them in the links that they provide to clients.
```
/customers/1/orders/     [Wrong]
/customers/1/orders      [Correct]
```
APIs may redirect(**301 Moved Permanently**) clients to URIs without a trailing forward slash.

##### File extensions should not be included in URIs
On the Web, the period (.) character is commonly used to separate the file name and
extension portions of a URI. A REST API should not include artificial file extensions
in URIs to indicate the format of a message’s entity body. Instead, they should rely on
the media type, as communicated through the **Content-Type** header, to determine how
to process the body’s content.
```
GET /customers/1/addresses.json - [Wrong]
GET, Request Headers(Accept: application/json) => /customers/1/addresses - [Correct]
```
REST API clients should be encouraged to utilize HTTP’s provided format selection
mechanism, the **Accept** request header

### 1.3 HTTP Methods

#### GET
**`GET`** requests are used to read either a single or a collection resource.
* **`GET`** requests for individual resources will usually generate a **404** if the resource does not exist.
Example:
```
GET /customers/1212121212 - 404 not found
```
* **`GET`** requests for collection resources may return either 200 (if the collection is empty) or 404 (if the collection is missing)
```
GET /customers?country_of_operation=usa - 200 ok
GET /customers/1/e-books - 404 not found
```
* **`GET`** requests must NOT have a request body payload.

**Note:** **`GET`** requests on collection resources should provide sufficient filter and Pagination mechanisms.

#### PUT
* **`PUT`** requests are used to **`update`** entire resources – single or collection resources.
* PUT must be used to both insert and update a stored resource
* on successful PUT requests, the server will replace the entire resource addressed by the URL with the 
representation passed in the payload (subsequent reads will deliver the same payload)
* successful PUT requests will usually generate **200** 
or **204** (if the resource was updated – with or without actual content returned), 
and **201** (if the resource was created)
  

#### POST
**`POST`** requests are used to create single resources on a collection resource endpoint.
* on a successful **`POST`** request, the server will create one or multiple new resources and provide their URI/URLs 
in the response
* successful POST requests will usually generate **200** (if resources have been updated), **201** (if resources have 
been created), **202** (if the request was accepted but has not been finished yet), and exceptionally **204** with 
Location header (if the actual resource is not returned).
  
**Note**: Resource IDs with respect to **`POST`** requests are created and maintained by server and returned with 
response payload.

#### PATCH
**`PATCH`** requests are used to update parts of single resources, i.e. where only a specific subset of resource fields 
should be replaced.

* **`PATCH`** requests are usually applied to single resources as patching entire collection is challenging
* **`PATCH`** requests are usually not robust against non-existence of resource instances
* on successful **`PATCH`** requests, the server will update parts of the resource addressed by the URL as defined by the 
change request in the payload
* successful **`PATCH`** requests will usually generate **200** or **204** (if resources have been updated with or without
 updated content returned)
 
 
#### DELETE
 **`DELETE`** requests are used to delete resources.
 * **`DELETE`** requests are usually applied to single resources, not on collection resources, as this would imply 
 deleting the entire collection.
 * successful  **`DELETE`** requests will usually generate **200** (if the deleted resource is returned) 
 or **204** (if no content is returned)
 * failed **`DELETE`** requests will usually generate **404** (if the resource cannot be found) 
 or **410** (if the resource was already deleted before)
 
 **Important**: After deleting a resource with **`DELETE`**, a **`GET`** request on the resource is expected to either 
 return **404** (not found) or **410** (gone) depending on how the resource is represented after deletion. 
 Under no circumstances the resource must be accessible after this operation on its endpoint.
 
 
#### HEAD
**`HEAD`** requests are used to retrieve the header information of single resources and resource collections.
* **`HEAD`** has exactly the same semantics as `GET`, but returns headers only, no body.

**Hint:** **`HEAD`** is particular useful to efficiently lookup whether large resources or collection resources have 
been updated in conjunction with the **`ETag`**-header.

#### OPTIONS
**`OPTIONS`** requests are used to inspect the available operations (HTTP methods) of a given endpoint.

* **`OPTIONS`** responses usually either return a comma separated list of methods in the Allow header or as a structured
 list of link templates.
  

### 1.4 Standard HTTP Status Codes

You must only use standardized HTTP status codes consistently with their intended semantics. You must not invent new 
HTTP status codes.

Category             | Description           
-------------------- | ---------------------
1xx                  | Informational Communicates transfer protocol-level information                     
2xx                  | Success Indicates that the client’s request was accepted successfully               
3xx                  | Redirection Indicates that the client must take some additional action in order to complete their request           
4xx                  | Client Error This category of error status codes points the finger at clients        
5xx                  | Server Error The server takes responsibility for these error status codes
  
#### Success Codes
Code  | Meaning                                                                             | Methods          
------| ------------------------------------------------------------------------------------|-------- 
200   | OK - this is the standard success response                                          | **`<all>`**            
201   | Created - Returned on successful entity creation. You are free to return either an empty response or the created resource in conjunction with the Location header. |  **POST**, **PUT**      
202   | Accepted - The request was successful and will be processed asynchronously.         | **POST**, **PUT**, **PATCH**, **DELETE**         
204   | No content - There is no response body.                                             | **PUT**, **PATCH**, **DELETE**
207   | Multi-Status - The response body contains multiple status informations for different parts of a batch/bulk request           | **POST**


#### Redirection Codes
Code  | Meaning                                                                             | Methods          
------| ------------------------------------------------------------------------------------|-------- 
301   | Moved Permanently - This and all future requests should be directed to the          | **`<all>`**            
301   | See Other - The response to the request can be found under another URI using a GET method. |  POST, PUT, PATCH, DELETE   
304   | Not Modified - indicates that a conditional GET or HEAD request would have resulted in 200 response if it were not for the fact that the condition evaluated to false, i.e. resource has not been modified since the date or version passed via request headers If-Modified-Since or If-None-Match. | GET, HEAD       

#### Client Side Error Codes
Code  | Meaning                                                                             | Methods          
------| ------------------------------------------------------------------------------------|-------- 
400   | Bad request - generic / unknown error. Should also be delivered in case of input payload fails business logic validation. | **`<all>`**
401   | Unauthorized - the users must log in (this often means "Unauthenticated"). |     **`<all>`**
403   | Forbidden - the user is not authorized to use this resource. |  **`<all>`**       
404   | Not found - the resource is not found. | <all>
405   | Method Not Allowed - the method is not supported, see OPTIONS. | **`<all>`**
406   | Not Acceptable - The client presented a content type in the Accept header which is not supported by the server API. | **`<all>`**
408   | Request timeout - the server times out waiting for the resource. | **`<all>`**
409   | Conflict - request cannot be completed due to conflict, e.g. when two clients try to create the same resource or if there are concurrent, conflicting updates. | POST, PUT, PATCH, DELETE
410   | Gone - resource does not exist any longer, e.g. when accessing a resource that has intentionally been deleted. | **`<all>`**
412   | Precondition Failed - returned for conditional requests, e.g. If-Match if the condition failed. Used for optimistic locking. | PUT, PATCH, DELETE
413   | Payload too large - Use it to signal that the request size exceeded the given limit e.g. regarding file uploads. | POST, PUT, PATCH
415   | Unsupported Media Type - e.g. clients sends request body without content type. | POST, PUT, PATCH, DELETE
423   | Locked - Pessimistic locking, e.g. processing states. | PUT, PATCH, DELETE
428   | Precondition Required - server requires the request to be conditional, e.g. to make sure that the "lost update problem" is avoided | **`<all>`**
429   | Too many requests - the client does not consider rate limiting and sent too many requests | **`<all>`**


#### Server Side Error Codes
Code  | Meaning                                                                             | Methods          
------| ------------------------------------------------------------------------------------|-------- 
500   | Internal Server Error - a generic error indication for an unexpected server execution problem (here, client retry may be sensible) | **`<all>`**
501   | Not Implemented - server cannot fulfill the request (usually implies future availability, e.g. new feature). |     **`<all>`**
503   | Service Unavailable - service is (temporarily) not available (e.g. if a required component or downstream service is not available) — client retry may be sensible. If possible, the service should indicate how long the client should wait by setting the Retry-After header. |  **`<all>`**       


### 1.5 Rate Limit
APIs that wish to manage the request rate of clients must use the **`429`** (Too Many Requests) response code, if the client exceeded the request rate.
Such responses must also contain header information providing further details to the client. There are two approaches a service can take for header information:

There are two approaches a service can take for header information:

* Return a **`Retry-After`** header indicating how long the client ought to wait before making a follow-up request. The Retry-After header can contain a HTTP date value to retry after or the number of seconds to delay. Either is acceptable but APIs should prefer to use a delay in seconds.
* Return a trio of **`X-RateLimit`** headers. These headers (described below) allow a server to express a service level in the form of a number of allowing requests within a given window of time and when the window is reset.
  
The **`X-RateLimit`** headers are:
 
* `X-RateLimit-Limit`: The maximum number of requests that the client is allowed to make in this window.
* `X-RateLimit-Remaining`: The number of requests allowed in the current window.
* `X-RateLimit-Reset`: The relative time in seconds when the rate limit window will be reset. Beware that this is different to Github and Twitter’s usage of a header with the same name which is using UTC epoch seconds instead.

### 1.8 Error condition responses
```
{
  "error": {
    "code": "INVALID_ARGUMENT",
    "target": "concat_info",
    "message": "Multiple errors in ContactInfo data",
    "details": [
      {
        "code": "NULL_VALUE",
        "target": "phone_number",
        "message": "Phone number must not be null"
      },
      {
        "code": "NULL_VALUE",
        "target": "last_name",
        "message": "Last name must not be null"
      },
      {
        "code": "MALFORMED_VALUE",
        "target": "address",
        "message": "Address is not valid"
      }
    ]
  }
}

```

## 2 Response structure


#### 2.1 Root level response structure
```
{
  "success": true/false,
  "message": "",
  "notifications": [
    {
      "type": "success",
      "message": ""
    },
    {
      "type": "info",
      "message": ""
    },
    {
      "type": "warning",
      "message": ""
    }
  ],
  "result": {
    "user": {},
    "users": [],
    "order": {},
    "orders": []
  },
  "error": {
    "code": "",
    "target": "",
    "message": "",
    "details": [
      {
        "code": "",
        "target": "",
        "message": ""
      }
    ]
  }
}
```

#### 2.2 Response structure with success response
**Example 1: Single resource**
```
HTTP/1.1 POST /login
Content-Type: application/json

Response: {
  "success": true,
  "messages": "Login successful",
  "result": {
    "user": {
      "name": "",
      "first_name": "",
      "middle_name": "",
      "last_name": "",
      "birthday": "",
      "address": [
        {
          "address1": "",
          "address2": "",
          "area": "",
          "city": "",
          "state": "",
          "zip": "",
          "country": "",
          "type": "",
          "longitude": "",
          "latitude": ""
        }
      ]
    }
  }
}
```

**Example 2: Get collection of resources**
```
HTTP/1.1 GET /customers/1/addresses
Content-Type: application/json

Response: {
  "success": true,
  "messages": "Fetched addresses resource successfully",
  "result": {
    "user_addresses": [
      {
        "address1": "",
        "address2": "",
        "area": "",
        "city": "",
        "state": "",
        "zip": "",
        "country": "",
        "type": "",
        "longitude": "",
        "latitude": ""
      }
    ]
  }
}
```

## 3 Security

### 3.1 Cross-Site Request Forgery (CSRF)
This attack method works by including malicious code or a link in a page that accesses a web application that the user is believed to have authenticated. If the session for that web application has not timed out, an attacker may execute unauthorized commands.

![CSRF attack diagram](./images/csrf.png)

* Bob browses a message board and views a post from a hacker where there is a crafted HTML image element. The element references a command in Bob's project management application, rather than an image file: <img src="http://www.webapp.com/project/1/destroy">
* Bob's session at www.webapp.com is still alive, because he didn't log out a few minutes ago.
* By viewing the post, the browser finds an image tag. It tries to load the suspected image from www.webapp.com. As explained before, it will also send along the cookie with the valid session ID.
* The web application at www.webapp.com verifies the user information in the corresponding session hash and destroys the project with the ID 1. It then returns a result page which is an unexpected result for the browser, so it will not display the image.
Bob doesn't notice the attack - but a few days later he finds out that project number one is gone.

It is important to notice that the actual crafted image or link doesn't necessarily have to be situated in the web application's domain, it can be anywhere - in a forum, blog post or email.

##### Prevention using Cookie-to-header token
* On an initial visit without an associated server session, the web application sets a cookie containing a random token that remains the same for the whole web session
```
Set-Cookie: csrf-token=1d9d07065c1686427845f5c23af6bfcc-1563535431850:aa00894cb39c179534e31162f540cde6a9d389887c529eeab34757fccf709b63; expires=Thu, 18-Jul-2019 10:00:00 GMT; Max-Age=600; Path=/; Secure
```

* JavaScript operating on the client side reads its value and copies it into a custom HTTP header sent with each transactional request

```
X-Csrf-Token: 1d9d07065c1686427845f5c23af6bfcc-1563535431850:aa00894cb39c179534e31162f540cde6a9d389887c529eeab34757fccf709b63
```

* The server validates presence and integrity of the token

Security of this technique is based on the assumption that only JavaScript running within the same `origin` will be able to read the cookie's value. JavaScript running from a rogue file or email will not be able to read it and copy into the custom header. Even though the `csrf-token` cookie will be automatically sent with the rogue request, the server will be still expecting a valid `X-Csrf-Token header`.

The CSRF token itself should be unique and unpredictable. It may be generated randomly, or it may be derived using HMAC:

```
//Node js sample code

const crypto = require('crypto');

const generateToken = function(timeout, length = 32) {
  const hex = crypto
      .randomBytes(Math.ceil(length / 2))
      .toString('hex'); // convert to hexadecimal format

  const msTime = new Date().getTime(); //timestamp in ms in UTC
  const expires = msTime + timeout; //1 sec = 1000 ms
  const raw = hex + "-" + expires;
  const hmac = crypto.createHmac('sha256', process.env.SECRET_KEY).update(raw).digest('hex');
  return raw + ":" + signature;
}‍

> generateToken(600000, 32)
// 1d9d07065c1686427845f5c23af6bfcc-1563535431850:aa00894cb39c179534e31162f540cde6a9d389887c529eeab34757fccf709b63

const validateToken = function(token) {
  const parts = token.split(":");
  const retrievedSignature = parts[1];
  const computedSignature = crypto.createHmac('sha256', process.env.SECRET_KEY).update(parts[0]).digest('hex');
  if(computedSignature !== retrievedSignature)
    return false;

  const expiryTs = parts[0].split("-")[1];
  const msTime = new Date().getTime(); //timestamp in ms in UTC
  if (msTime > expiryTs)
    return false;
    
  return true;
}

> validateToken(request.headers['X-Csrf-Token']')
// true

```

**Note:** The CSRF token cookie must not have httpOnly flag, as it is intended to be read by the JavaScript by design.

**Note:** For all incoming requests that are not using `HTTP GET, HEAD, OPTIONS or TRACE` a CSRF cookie must be present and correct. If it isn’t, the user will get a **`403`** error.
          
### 3.2 Secure Headers

##### 3.2.1 HTTP Strict Transport Security (HSTS)
HTTP Strict Transport Security (HSTS) is a web security policy mechanism which helps to protect websites against protocol `downgrade attacks` and `cookie hijacking`. It allows web servers to declare that web browsers (or other complying user agents) should only interact with it using secure HTTPS connections, and never via the insecure HTTP protocol. HSTS is an IETF standards track protocol and is specified in RFC 6797. A server implements an HSTS policy by supplying a header (Strict-Transport-Security) over an HTTPS connection (HSTS headers over HTTP are ignored).

##### Values
Value | Description
----- | -----------
max-age=SECONDS	| The time, in seconds, that the browser should remember that this site is only to be accessed using HTTPS.
includeSubDomains | If this optional parameter is specified, this rule applies to all of the site's subdomains as well.

##### Example
```
Strict-Transport-Security: max-age=31536000 ; includeSubDomains
```

##### 3.2.2 X-Frame-Options
X-Frame-Options response header improve the protection of web applications against `Clickjacking`. It declares a policy communicated from a host to the client browser on whether the browser must not display the transmitted content in frames of other web pages.

##### Values
Value              | Description
-----              | -----------
deny	           | No rendering within a frame.
sameorigin         | No rendering if origin mismatch.
allow-from: DOMAIN | Allows rendering if framed by frame loaded from DOMAIN.

##### Example
```
X-Frame-Options: sameorigin
```

##### 3.2.3 X-XSS-Protection
This header enables the `Cross-site scripting (XSS)` filter in your browser.

##### Values
Value               | Description
-----               | -----------
0	                | Filter disabled.
1                   | Filter enabled. If a cross-site scripting attack is detected, in order to stop the attack, the browser will sanitize the page.
1; mode=block       | Filter enabled. Rather than sanitize the page, when a XSS attack is detected, the browser will prevent rendering of the page.
1; report=http://[YOURDOMAIN]/your_report_URI | Filter enabled. The browser will sanitize the page and report the violation. This is a Chromium function utilizing CSP violation reports to send details to a URI of your choice.

##### Example
```
X-XSS-Protection: 1; mode=block
```

##### 3.2.4 X-Content-Type-Options
Setting this header will prevent the browser from `interpreting files as something else than declared by the content type` in the HTTP headers.

##### Values
Value               | Description
-----               | -----------
nosniff	            | Will prevent the browser from MIME-sniffing a response away from the declared content-type.

##### Example
```
X-Content-Type-Options: nosniff
```

##### 3.2.5 Content-Security-Policy
A Content Security Policy (CSP) requires careful tuning and precise definition of the policy. If enabled, CSP has significant impact on the way browsers render pages (e.g., inline JavaScript disabled by default and must be explicitly allowed in policy). CSP prevents a wide range of attacks, including `Cross-site scripting` and other `cross-site injections`.
##### Values
Directive           | Description
---------           | -----------
no-referrer | The Referer header will be omitted entirely. No referrer information is sent along with requests.
no-referrer-when-downgrade | This is the user agent's default behavior if no policy is specified. The origin is sent as referrer to a-priori as-much-secure destination (HTTPS->HTTPS), but isn't sent to a less secure destination (HTTPS->HTTP).
origin | Only send the origin of the document as the referrer in all cases. The document https://example.com/page.html will send the referrer https://example.com/.
origin-when-cross-origin | Send a full URL when performing a same-origin request, but only send the origin of the document for other cases.
same-origin | A referrer will be sent for same-site origins, but cross-origin requests will contain no referrer information.
strict-origin | Only send the origin of the document as the referrer to a-priori as-much-secure destination (HTTPS->HTTPS), but don't send it to a less secure destination (HTTPS->HTTP).
strict-origin-when-cross-origin | Send a full URL when performing a same-origin request, only send the origin of the document to a-priori as-much-secure destination (HTTPS->HTTPS), and send no header to a less secure destination (HTTPS->HTTP).
unsafe-url | Send a full URL (stripped from parameters) when performing a a same-origin or cross-origin request.

##### Example
```
Content-Security-Policy: script-src 'self'
```

##### 3.2.6 Referrer-Policy
The Referrer-Policy HTTP header governs which referrer information, sent in the Referer header, should be included with requests made.

##### Values
Value           | Description
-----               | -----------
base-uri | Define the base uri for relative uri.
default-src | Define loading policy for all resources type in case of a resource type dedicated directive is not defined (fallback).
script-src | Define which scripts the protected resource can execute.
object-src | Define from where the protected resource can load plugins.
style-src | Define which styles (CSS) the user applies to the protected resource.
img-src | Define from where the protected resource can load images.
media-src | Define from where the protected resource can load video and audio.
frame-src | Deprecated and replaced by child-src. Define from where the protected resource can embed frames.
child-src | Define from where the protected resource can embed frames.
frame-ancestors | Define from where the protected resource can be embedded in frames.
font-src | Define from where the protected resource can load fonts.
connect-src | Define which URIs the protected resource can load using script interfaces.
manifest-src | Define from where the protected resource can load manifest.
form-action | Define which URIs can be used as the action of HTML form elements.
sandbox | Specifies an HTML sandbox policy that the user agent applies to the protected resource.
script-nonce | Define script execution by requiring the presence of the specified nonce on script elements.
plugin-types | Define the set of plugins that can be invoked by the protected resource by limiting the types of resources that can be embedded.
reflected-xss | Instructs a user agent to activate or deactivate any heuristics used to filter or block reflected cross-site scripting attacks, equivalent to the effects of the non-standard X-XSS-Protection header.
block-all-mixed-content | Prevent user agent from loading mixed content.
upgrade-insecure-requests | Instructs user agent to download insecure resources using HTTPS.
referrer | Define information user agent must send in Referer header.
report-uri | Specifies a URI to which the user agent sends reports about policy violation.
report-to | Specifies a group (defined in Report-To header) to which the user agent sends reports about policy violation.

##### Example
```
Referrer-Policy: no-referrer
```

### 3.3 Input validation
* Do not trust input parameters/objects.
* Validate input: length / range / format and type.
* Achieve an implicit input validation by using strong types like numbers, booleans, dates, times or fixed data ranges in API parameters.
* Constrain string inputs with regexps.
* Reject unexpected/illegal content.
* Make use of validation/sanitation libraries or frameworks in your specific language.
* Define an appropriate request size limit and reject requests exceeding the limit with HTTP response status `413 Request Entity Too Large`.
* Consider logging input validation failures. Assume that someone who is performing hundreds of failed input validations per second is up to no good.
* Have a look at input validation cheat sheet for comprehensive explanation.
* Use a secure parser for parsing the incoming messages. If you are using XML, make sure to use a parser that is not vulnerable to XXE_Processing) and similar attacks.

### 3.4 Validate content types
* A REST request or response body should match the intended content type in the header. Otherwise this could cause misinterpretation at the consumer/producer side and lead to code injection/execution.
* Document all supported content types in your API.
* Reject requests containing unexpected or missing content type headers with HTTP response status `406 Unacceptable` or `415 Unsupported Media Type`.

## 4 HTTP Cache Headers
TODO