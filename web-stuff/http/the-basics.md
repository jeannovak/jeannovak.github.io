# The Basics

## HTTP Basics

Structure of HTTP requests and responses:

```text
HEADERS\r\n
\r\r

MESSAGE\r\n
```

```text
\r: Carriage Return, moves the cursos to the beginning of the next line
\n: Line Feed, moves the cursor to next line
\r\n: Same thing as pressing enter.
```

### REQUEST

Example of GET HTTP request to google.com

```bash
GET / HTTP/1.1
Host: www.google.com
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Cookie: 1P_JAR=2020-04-14-14; NID=202=Y072CM6yIPqsRVr3BViIOnTa1y67QxqUh0P1IqWMNdl9i6ohTQzu8xtxwJk2JN9IcPCRjrqNtyokjUpcewksaEM_hnMi7hN1YPfW-62d8j5o2EtP-jYodVbS_JIuGg2mIPG4U2M2o5V8skaHeXAWMMwJZdidgIppupU4fh0Qfo8; OGPC=19016257-10:; ANID=AHWqTUmC2qWkx-S3bCqztqibWcLFYjxar3bfXT9d4R_Z1-qUnI3x1BHmULMwhcwh; OTZ=5384900_68_64_73560_68_416340
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0
```

HTTP headers follow the structure of Name: Value

Line 1 specifies three important things:

* **GET** is the Request method used
* **/** is the path you are requesting, in this case, the root directory \(URI\)
* **HTTP/1.1** os the HTTP Protocol version

**Host** is the website we're trying to access, this is useful for example in cases where the webserver hosts multiple websites, so we can specificy which one we want to access via host header.  
The Host value + the path creates the full URL you're trying to access.

**User-Agent** reveals your browser, OS and language so the webserver knows which kind of page it needs to display, for example, a mobile version.

**Accept** is used to specificy which kind of document is expected in the webservers response. **Accept-Language** has the same idea.

**Accept-Encoding** has the same idea, primarily used to allow a document to be compressed.

**Connection** is used to specific if you need to keep this session open and to sent further requests through it without initiating new connections every time. In this case, we need to pass the Connection value as 'keep-alive'

Since HTTP is a stateless / connection-less protocol, no information can be retained between different requests and responses, so we **Cookies** to store useful information, like that you are already logged in to a page. Do not mix that with the fact that TCP is connection-oriented, we're talking about the application, not the Layer 4 protocol.

**Upgrade-Insecure-Requests** is set to tell browsers to request HTTPS instead of HTTP.

**Cache-Control** controls how the information of the request and response is stored, its nature and for how long.

### RESPONSE

```bash
HTTP/1.1 200 OK
Date: Tue, 14 Apr 2020 14:41:35 GMT
Expires: -1
Cache-Control: private, max-age=0
Content-Type: text/html; charset=UTF-8
Strict-Transport-Security: max-age=31536000
Server: gws
X-XSS-Protection: 0
X-Frame-Options: SAMEORIGIN
Set-Cookie: 1P_JAR=2020-04-14-14; expires=Thu, 14-May-2020 14:41:35 GMT; path=/; domain=.google.com; Secure; SameSite=none
Connection: close
Content-Length: 191933

<content ommited>
```

Line 1 specifies:

* **HTTP/1.1** is the protocl
* **200 OK** is the way Webservers use to say the result of the client's request. More information [here](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status): 

![API calls and HTTP Status codes - ITNEXT](https://miro.medium.com/max/920/1*w_iicbG7L3xEQTArjHUS6g.jpeg)

Date represents date and time when the message was originated.

Server is the banner, in this case, GWS means Google Web Server.

X-XSS-Protection

X-Frame-Options: SAMEORIGIN

Set-Cookie: 1P\_JAR=2020-04-14-14; expires=Thu, 14-May-2020 14:41:35 GMT; path=/; domain=.google.com; Secure; SameSite=none

Connection: close

Content Lenght

