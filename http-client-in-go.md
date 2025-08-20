# how does web communication work?
When two computers communicate with each other they use some rules.

This "language" that computers use to communicate is called a protocol.
The most popular protocol for the web is HTTP, which stands for "Hyper
Text Transfer Protocol".


# HTTP Request Response
At the heart of HTTP it is a simple request-response system. The 
"requesting" computer also known as the client asks another computer for
some information. That computer, "the server" sends back a response with
the information that was needed.

# HTTP Powers Websites
HTTP or Hyper Text Transfer Protocol is a protocol designed to transfer
information between computers.

There are other protocols for communicating over the internet, but HTTP
is the most popular and is *particularly great for websites and web applications*.
Each time you visit a website, your browser is making an HTTP request to that website's server. The server responds will all the text, images and
styling information that your browser needs to render its pretty website.

# HTTP URLs
A URL, or uniform Resource locator, is the address of another computer or *server* on the internet. Part of the URL specifies where to reach the 
server *what information we want*.

# Using URLs in HTTP
The `http://` at the beginning of the website URL specifies that the 
HTTP protocol will be used for communication.

Other communication protocol also use URLs as well, (hence "Uniform Resource Locator"). That's why we need to be specific when we're making HTTP
requests the URL with `http://`

# net/http
We will mainly be using Go's standard 'net/http' package and the `http.client` to make HTTP requests.

```go

package main

import (
	"fmt"
	"io"
	"net/http"
)

func getProjects() ([]byte, error) {
	res, err := http.Get("https://api.jello.com/projects")
	if err != nil {
		return nil, fmt.Errorf("error making request: %w", err)
	}
	defer res.Body.Close()
	
	data, err := io.ReadAll(res.Body)
	if err != nil {
		return nil, fmt.Errorf("error reading response: %w", err)
	}
	return data, nil
}

```

- `http.Get` uses the `http.DefaultClient` to make a request to the given URL
- res is the http response that comes back from the server
- defer `res.Body.Close()` ensures that the response body is properly 
closed after reading. Not doing so can cause memory issues 
- `io.ReadAll()` reads the response body into a slice of bytes []byte called data


# JSON Syntax
JSON supports the following primitive data types:
- Strings
- Numbers
- Booleans
- Null e.g `null`

And the following collection types:
- Arrays e.g `[1, 2, 3]`
- Object Literals e.g `{ "key": "value" }`


# Decoding JSON
When you receive JSON data in a body of an HTTP response, it comes as a 
stream of bytes.
We can convert stream of bytes to a string.
In Go there is a better way. It's typically best to decode the JSON data
into a struct.

```json
[
  {
    "id": "001-a",
    "title": "Unspaghettify code",
    "estimate": 9001
  }
]
```
To decode this JSON into a slice of of `Issues` structs, we need to know
the JSON fields and their types. The standard `encoding/json` package uses
tags to map JSON fields to structs fields.

*Struct fields must be exported (capitalized) to decode JSON.*

```go

type Issue struct {
	Id string `json:"id"`
	Title string `json:"title"`
	Estimate string `json:"estimate"`
}

```

After receiving the response, we can decode it into a slice of `Issues`
with the "address of" operator `&`

```go
// res is a successful `http.Response`

var issues []Issues
// you tell json.NewDecoder() from where to read the 
// stream of bytes
decoder := json.NewDecoder(res.Body)
// here is where all the actual parsing and mapping happens
// from stream of bytes to Issue Struct
if err := decoder.Decode(&issues); err != nil {
	fmt.Println("error decoding response body")
	return // you can return error in a function
}

```

If no error occurs, we can use the slice of items in our program.

```go

for _, issue := range issues {
	fmt.Printf("Issue - id: %v, title: %v, estimate: %v\n", issue.id, issue.Ttile, issue.Estimate)
}

```

# JSON Review
JSON is a *stringified representation* of JS Object, which makes it perfect
for saving to a file or sending in an HTTP request. Remember, an actual
JS object is something that exists only with you program's variables. If 
we want to send an object outside our program, for example across the 
internet in an HTTP request we need to convert it to JSON first.

# JSON isn't just for javascript
Just because JSON is called JavaScript Object Notation doesn't mean it's only used by JavaScript code! JSON is a common standard that is recognized and supported by every major programming language. For example, even though Boot.dev's backend is written in Go, we still use JSON as the communication format between the front-end and backend.

# Common Use-Cases
- In HTTP request and response bodies
- As formats for text files `.json` are often as configuration files
- NoSql databases like MongoDB, Elastic Store and Firestore


# Unmarshel JSON
We can decode JSON byte (strings) into a Go struct using `json.Unmarshal` or a 
`json.Decoder`.

The `Decode` method of `json.Decoder` streams data from a `io.Reader` into a Go
struct, while `json.Unmarshal` works with data that's already in `[]byte` format 
. Using a `json.Decoder` can be more memory-efficient because it doesn't load all
the data into memory at once. `json.Unmarshal` is ideal for small JSON data you already
have in memory. When dealing with HTTP requests and responses, you will likely use
`json.Decoder` since it works directly with an `io.Reader`.

```go
defer res.Body.Close()

data, err := io.ReadAll(res.Body)
if err != nil {
	return nil, err
}

var issues []Issue
if err = json.Unmarshal(data, &issues); err != nil {
	return nil, err
}
return issues, nil

```

# Marshal JSON
If there is a way to `unmarshal` JSON data, there must be a way to marshal
it as well. The `json.Marshal` function converts a Go struct into a slice bytes
representing JSON data.

```go
type Board struct {
	Id		int		`json:"id"`
	Name		string		`json:"name"`
	TeamId		int		`json:"teamId"`
	TeamName	string		`json:"teamName"`
}

board :=  Board{
	Id:		1,
	Name:		"API",
	TeamId:		9001,
	TeamName:	"Backend",
}

// converting Go struct into JSON
data, err := json.Marshal(board)
if err != nil {
	log.Fatal(err)
}
// stringifying the JSON Object to log it
fmt.Println(string(data))
```

Note: you are going to be confuse 'marshal' and 'unmarshal'
Marshaling ->  converting Go Struct to JSON.
Unmarshaling -> converting JSON to Go Struct.


# XML
We can't talk about JSON without mentioning XML, XML or Extensible
Markup Language is a text based format for representing structured 
information, like JSON but different.

# XML Syntax
XML is markup language like HTML, but it's more generalised is that 
doesn't have predefined tags. Just like how in JSON object keys can
be anything.

```xml
<root>
	<id>1</id>
	<title>Action</title>
	<title>Iron Man</title>
	<director>John Favreau</director>
</root>

```

```json

{
	"id": "1",
	"genre": "Action",
	"title": "Iron Man",
	"director": "Jon Favreau"
}

```

# map[string]interface{}
Sometimes you have to deal with JSON data of unknown or varying
structure in Go. In those instances `map[string]interface{}` offers
a flexible way to handle it without predefined structs.

Think about it: a JSON object is a key-value pair, where the key is a 
string and the value can be any JSON type `map[string]interface{}` is
a map where the key is string and the value can any go type.

Note: `any` is an alias for `interface{}`

```go

var data map[string]interface{}
jsonString :=  `{"name": "Alice", "age": 30, "address": {"city": "wonderland"}}`
jsonUnmarshal([]byte(jsonString), &data)
fmt.Println(data["name"]) // Alice
fmt.Println(data["address"]).(map[string]interface{})["city"] // Alice


```

# Web Addresses
In computing, web clients find other computers over the internet using
Internet Protocol (IP) addresses. Each device to the internet has a 
*unique IP address*.

# Domain Names and IP address
When we browse the internet, we type in a human readable domain name.
That domain name is converted into IP address. The IP address tells our
computer where the server is located on the internet.

Version of IP address
IP4 -> is kinda old now
IP6 -> very commonly used now.


# Deploying a Website 
Your server will host your app/website.
You Buy domain name.
Connect Domain name with IP address of the server.

# DNS
A "domain name" or "hostname" is just one portion of a URL. We'll get to
the other parts of the URL later.

For Example:
`https://homestarunner.com/toons` has a HOSTNAME of `homestarrunner.com`.
The `https://` and `/toons` aren't part of the `domain name -> IP address
mapping that we've been talking about.


## The `net/url` package
The `net/url` package is part of Go's standard library. You can instantiate
a URL struct using `url.Parse`

```go

parsedURL, err := url.Parse("https://boot.dev")
if err != nil {
	// you can also return an error
	fmt.Println("unable to parse raw URL %w", err)
	return
}

parsedURL.Hostname() // boot.dev


```

# What is a domain name system?
So we're talked about domain names, but we haven't talked about the 
*system* what makes them work.

DNS or the "Domain Name System", is the phonebook of internet. Humans
type easy-to-read domain names like 'boot.dev'. DNS resolves those domains
names to their associated "IP addresses" so that web clients can find 
the server they're looking for.

*Domain names are for humans, IP addresses are for computers*

## How does DNS Work?
Let's just introduce ICANN. ICANN is a not-for-profit organization
that manages DNS for entire internet.

Whenever you computer attempts to resolve a domain name it contracts
one of ICANN's "root nameservers" whose address is included in your
computer's networking configuration. From there, that nameserver can
gather the domain records for a specific domain name from their
distributed DNS database.

If you think of DNS as a phonebook, ICANN is the publisher that keeps
the phonebook in print and available.

# Subdomains
We learned about how a domain name resolves to an IP address, which is
just a computer on a network - often the internet

A *subdomain* prefixes a domain name, allowing a domain to route network
to many different servers and resources

For example,  the 'Boot.dev' website is hosted on a different machine
than our blog. Our blog,  found at 'blog.boot.dev' is hosted on our
"blog" subdomain(and the IP address/computer it points to)


# Uniform Resource Identifiers
We briefly touched on URLs earlier, let's drive a little deeper into
the subject.

A URI, or *Uniform Resource Identifier* is a unique character sequence
that identifies a resource that is (almost always) accessed via the
internet

Just like Golang has a syntax rules so do URIs. These rules help ensure
uniformity so that any program can interpret the meaning of the URI in
the same way.

URIs come two main types:
- URLs
- URNs

We will focus specifically on URLs in this course, but its' important to 
know that URLs are only one kind URI.

# Sections of a URL
URLs have quite few sections. Some are required, some are not.

After you parse a raw url it returns `type URL struct{}`
where you can use `parsedURL.Hostname()` like methods or `parsedURL.Scheme`
like fields to get parsed values.


# Further Dissecting  URL
There are 8 main parts to a URL, though not all the sections are always
present. Each piece plays a role in helping clients locate the resources
they're looking for.

```

https://waynelagner:pas55w0rd@jello.app:8080/boards?sort=createdAt#id

http -> protocol
waynelager -> username
pa55w0rd -> password
jello.app -> domain name
8080 -> port name
/boards -> path
?sort=createdAt -> query parameter (query paramenter start after ?)
#id -> fragment


```

| Part |  Required|
| -------------- | --------------- |
| Protocol (also refered to as the "scheme") | Yes | 
| Username | No |
| Password | No |
| Domain | Yes |
| Port | No (defaults to 80 or 443) |
| Path | No (defaults to '/' ) |
| Query | No |
| Fragment | No |


# The Protocol
The protocol also referred to as the "scheme" is the first component
of a URL. It defines the rules by which the data being communicated
is displayed encoded or formatted.

Some examples of different URL protocols:
- http
- ftp
- mailto
- https

For example:
- `http://example.com`
- `mailto:noreply@jello.app`

## Not All Schemes Require a "//"
The "http" in a URL is always followed by `://`. All URls have the colon
but the `//` part is only included for schemas that have the 'authority
component'. As you can see `mailto` schema doesn't use an authority 
component, so it doesn't need the slashes.

# URL Ports
The server is running multiple programs it does it using ports.
Like a Database, Server, Listening for a client request all these things
are done by using ports.

The port in a URL is a virtual point where network connections are made.
Ports are managed by a computer's operating system and are numbered from 0
to `65,535`(Though port 0 is reserved for the system API)
Port numbers are represented by unsigned 64-bit integer so that is the
reason for `65,535` number of ports.

Whenever you connect to another computer over a network, you're connecting
to a specific port on that computer, which is listened to by a program
on that computer. A port can only be used by one program at a time, which 
is why there are some many possible ports.

The port component of a URL is often not visible when browsing normal sites
on the internet, because 99% of the time you're using default ports for 
the *HTTP and HTTPS schemas 80 and 443 respectively*.

```

http://google.com:80/?q=bootdev
		  |
		equals	
		  |
http://google.com/?q=bootdev

```

Whenever you are not using default port, you need to specify it in the 
URL. For example, port 8080 is often used by web developers when
they're running their server in "test mode" on their own machines.


# URL Paths
On static sites (like blogs or documentation sites) a URL's path mirrors
the servers' filesystem hierarchy.

For example, if the websites `https://exampleblog.com` had a static web
server running in its `/home` directory, then a request to 
`https://exampleblog.com/site/index.html` probably return the file
located at.
`/home/site/index.html`

*But technically, this is just a convention. The server could be 
configured to return any file or data given that path*


## Its Not Always That Simple
Paths in URLs are essentially just another type of parameter that can
be passed to the server when making a request. For dynamic sites and 
web applications the path is often used to denote a specific resource
or endpoint.


# Query Parameters
Query Parameters in a URL are *not* always present. In the context of 
websites, query parameters are often used for *marketing analytics* or 
for changing a variable on the web page. With websites URLs, query
parameters rarely change which page you are viewing though they will
often change the page's contents.
That said, query parameters can be used for anything the server chooses 
to use them for, just like the URL's path.

# What are HTTP Headers?
An HTTP header allows client and servers to pass additional information
with each request or response. Headers are just case-insensitive key-value
pairs that pass additional metadata about the request to response.

HTTP request from a web browser automatically carry with them many headers,
including but not limited to:

- The type of client (e.g Google Chrome)
- The Operating System (e.g Linux)
- The preferred Language (e.g English)

As developers, we can also define custom headers in each request.

# Headers in Go's net/http Package
In Go, the net/http package provides us with the necessary tools to
work with HTTP headers. We can access headers through the Headers type,
which  is essentially a map of string slices  `map[string][]string[]`.
This allows us to perform various actions on our request and response
headers such as retrieving, setting and removing them.

```go

// creating a new request
req, err := http.NewRequest("GET", "https://api.example.com/users", nil)
if err != nil {
	fmt.Println("error creating request", err)
	return
}

// setting a header on the new request
req.Headers.set("x-api-key", "123456")

// making a request
client := http.Client{}
res, err := client.Do(req)

if err != nil {
	fmt.Println("error making request", err)
	return
}
defer res.Body.Close()

// reading a header from the response
header := res.Header.Get("last-modified")
fmt.Println("last modified", header)

// deleting a header from the response
res.Header.Del("last-modified")

```

# The Browser Network Tab
While all of the tabs within the dev tools are useful, we will focus
on the network tab in this chapter so we can play with HTTP headers.
The network tab monitors your browser's network activity and records 
all of the requests and responses the browser is making, including how
long each of those requests and responses takes to fully process.


# Why are Headers Useful?
Headers are useful for several reasons from *design* to *security*, but most
often headers are used for *metadata* about the request or response itself.
For example, let's say we wanted to ask for a player's level from the Jello
server. We need to send that player's ID to the server so it knows which player
to send back the information for. That ID is my request its not information about 
my request.

Authentication is a common use case for headers. If I ask jello to complete a 
project, I need to provide authentication information that I'm logged in, but that

# HTTP Methods - GET
HTTP defines a set of *methods*. We must choose one to use each time we
make and HTTP request. The most common ones include:
- Get
- Post
- Put
- Delete

The GET method is used to "get" a representation of specified resource.
It does not take (remove) the data from the sever but rather gets a 
representation or copy of the resource in its current state. A GET request
is considered a *safe* method to call multiple time because it shouldn't
alter the state of the server.

## Making a GET Request in Go
There are two basic ways to make a Get request in Go.
- The simple but less powerful way: `http.Get`
- The verbose but more powerful way: `http.Clinet`, `http.NewRequest`
and `http.Client.Do`

If all you need to do is make a simple GET request to a  URL, `http.Get`
will work:

```go

resp, err := http.Get("<url-here>")


```

If you need to customize things like headers, cookies or timeouts you'll
want to create a custom `http.Client` and `http.NewRequest` then use the
client' `Do` method to execute it.

```go

client := &http.Client{
	Timeout: time.Second * 10,
}

req, err := http.NewRequest("GET", "<url-here>", nil)
if err != nil {
	log.Fatal(err)
}

resp, err := client.Do(req)


```

# Why use HTTP methods?
The primary purpose of HTTP methods is to indicate to the server what we
want to do the resource we're trying to interact with. At the end of the
day, an HTTP method is just a string, like GET, POST, Or DELETE but by 
convention backend developers write their server code so that the methods
correspond with different CRUD actions.

The "CRUD" actions are:
- Create -> POST
- Read ->  GET
- Update -> PUT
- Delete -> DELETE

These are conventions some people may *not* choose to follow it.
Some use POST for Updating and Deleting entirely.

# POST Requests
An HTTP POST request sends data to a server, typically to *create* a new resource.

## Adding a Body
The body of a request is the payload send to the server. The special 
`Content-Type` header is used to tell the server the format of the body:
`application/json` for JSON data in our case. POST requests are generally
not safe methods to call multiple times because that would create duplicate
records. For example you wouldn't want to accidentally send a tweet twice.

Like `http.Get`, the standard library's `http.Post` function can be used
to send  simple `POST` requests. The trouble is, it's a bit limited. And 
because we need to add an `X-API-KEY` header, the simple `http.POST` function
won't work for us. Instead, we need to use `https.NewRequest`

```go

type Comment struct {
	Id string `json:"id"`
	UserId int `json:"user_id"`
	Comment string `json:"comment"`
}

func createCommnet(url, apiKey string, commentStruct Comment) (Comment, error) {
	jsonData, err := json.Marshal(commentStruct)
	if err != nil {
		return Comment{}, err
	}

	req, err := http.NewRequest("POST", url, byte.NewBuffer(jsonData))
	if err != nil {
		return Comment{}, err
	}

	// set request Headers
	req.Header.Set("Content-Type", "application/json")
	req.Header.Set("X-API-KEY", apiKey)

	client := &http.Client{}
	res, err := client.Do(req)
	if err != nil {
		return Commnet{}, err
	}
	defer res.Body.Close()

	// decode the json data from the response
	var comment Commnet
	decoder := json.NewDecoder(res.Body)
	if err := decoder.Decode(&comment); err != nil {
		return Comment{}, err
	}

	return comment, nil
}


```

# Status Code Property
The `http.Response` struct has a `.StructCode` property that contains
the status code of the response.

# HTTP PUT
The HTTP PUT method creates or more commonly updates the target resources
with the contents of the request's body. Unlike GET and POST, there is
no `http.Put` functions. You will have to create a raw `http.Request`
that an `http.Client` can 'Do'

## POST vs PUT
While POST and PUT are both used for creating resources, PUT is more 
common for updates and is idempotent, meaning it's safe to make the 
same request multiple times without changing the sever state more
than once. For example

```go

POST /users/bob (create bob user)
POST /users/bob (create duplicate bob user)
POST /users/bob (create duplicate bob user)

--------------------------------------------

PUT /users/bob (create bob user if it doesn't exist)
PUT /users/bob (update bob user with the same data)
PUT /users/bob (update bob user with the same data)

```

# Patch vs PUT
You may encounter the PATCH method from time to time, It's frankly
not used very often, it's meant to *partially* modify a resource,
whereas PUT is meant to completely replace a resource

Patch is not a popular as PUT,  and many servers, even if they allow
partial updates, just use PUT.

# Delete
The DELETE method does exactly what you expect it deletes a spcified
resources.

```go

// This deletes the location with ID: 52fdfc07-2182-454f-963f-5f0f9a621d72
url := "https://api.boot.dev/v1/courses_rest_api/learn-http/locations/52fdfc07-2182-454f-963f-5f0f9a621d72"
req, err := http.NewRequest("DELETE", url, nil)
if err != nil {
	fmt.Println(err)
    return
}

client := &http.Client{}
res, err := client.Do(req)
if err != nil {
	fmt.Println(err)
    return
}
defer res.Body.Close()

if res.StatusCode > 299 {
    fmt.Println("request to delete location unsuccessful")
    return
}
fmt.Println("request to delete location successful")

```

# Paths
The URLs Path comes right after the domain (or port if one is provided)
in the URL string.
`http://testdomain.com/root/next`

The Pat is:
`/root/next`

## Path Conversion
Most static website use paths to a direct mapping to the path on the 
server's file system.
For example:
`http://www.example.com/document/hello.txt` URL Path will server
`/document/hello.txt` file from the server.

Most dynamic web applications dont' use this simple mapping of 
`URL path -> file path`. Technically, a URL path is just a string
that a web server can decide to whatever to do with it and most 
modern web application take advantage of this flexibility.
Some common example of what paths are used for include:

- The hierarchy of pages on a website, whether or not that reflects
a server's file structure.
- Parameter passed into an HTTP request like the ID of a resource.
- The version of the API
- The type of resource being requested.

# RESTful APIs
REpresentational State Transfer  or REST, is popular convention that 
many dynamic HTTP servers follow. Not all HTTP APIs are "REST APIs",
or "RESTful" but it is very common

RESTful servers follow a loose set of rules that makes it easy to
build a reliable and predictable web APIs. REST is a set of conventions
about how HTTP APIs should be built.

## Separate and Agnostic
The big idea behind REST is that resource are transformed via well
recognized, language-agnostic client/server interactions. A RESTful 
style means the implementation of the client and server can be created
independently of one another, as long as some simple standards, like
the names of the available resources have been estb.

(This means you can make your client in JS and backend in python and
it would not matter and  either don't care about the implementation)

## Stateless
A RESTful architecture is *stateless*, which means the server does not
need to know what state the client is in, nor does the client need to
know  what state the server is in. Statelessness in REST enforced by
interacting with resource instead of commands. Keep in mind, this doesn't
mean the application are stateless - what would "updating a resource" even mean if the server wasn't keeping track of its state?

-- NOTE:
If the client is sending a GET request to the server should server that GET request.
If the client is sending a POST request to the server should server that POST request.


## Paths in Rest
In a RESTful API, the last section of the `path` of a URL specifies the resource.
For Jello, this means an issue, a `user` or a `project`. Depending on whether the 
request is a GET, POST, PUT or DELETE the resource is read, created, updated or deleted.

# URL query Parameter
A URL's query parameters appear next in the URL structure but are not always
present - they're optional

```text

https://www.google.com/search?q=boot.dev

````
`q=boot.dev` is a query parameter. Like headers query parameters are 
`key / value` pairs. In this case `q` is the key and `boot.dev` is the value.

# The documentation of an HTTP Server
The good news is that you don't need to. When you work with a backend server,
it's the responsibility of that server's developers to provide you with instructions,
or documentation that explains how to interact with it. For example, the
documentation should tell you:

- The domain of the server.
- The resource you can interact with (HTTP paths).
- The supported query parameters.
- The supported HTTP methods.
- Anything else you'll need to know to work with the server.

## The Server is the captain now
The server has complete control over how the path in a URL is interpreted 
and used in a request. The same goes for query parameters. While there are
a lot of strong conventions around how servers should interpret paths and query
parameters, the server can do whatever it wants. That's why you need docs.

# Multiple Query Parameter
Query parameter are `key/value` pairs - that means there can be multiple pairs!

```text

http://example.com?firstName=lane&lastName=wagner

- firstName = lane
- lastName = wagner

```

The `?` separates the query parameters from the rest of the URL. The `&`
is then used to separate each additional pair of query parameters after that


# HTTPS
HyperText Transfer Protocol Secure or HTTPS is an extension of the HTTP
protocol. HTTPS secures the data transfer between client and the server
by encrypting all of the communication.

HTTPS allows a client to safely share sensitive information with the
server through HTTP request, such as credit card informations, passwords,
or bank account numbers.

# Security and Encryption
HTTPS requires that the client uses SSL or TLS to protect requests
and traffic by encrypting the information in the request. You don't
need to manually handle encrypting or decrypting. Its all done for
you by the protocol *HTTPS is simple HTTP with built-in security*

## HTTPS != Privacy
- HTTPS will encrypt the data in a request (like passwords, headers,
body information, query parameters, etc). You and the recipient are
the only ones who can read that information.

- HTTPS will not hide who you are or that you're communicating with 
the given server. If you don't want your wife who knows how to sniff
network traffic to know you're using Boot.dev, HTTPS won't help.

## HTTPS verifies the Server
In addition to encrypting the information within a request. HTTPS uses
*digital signatures* to prove that you're communicating with the sever
that you think you are. If a hacker were to intercept an HTTPS request
by tapping into a network cable, they wouldn't be able to successfully 
pretend they are your bank's web sever.

When using HTTPS: 
- The client verifies the servers digital certificates to ensure it's
communicating with the correct server.
- The client and server use the public/private key pair to perform
a handshake and securely agree on a shared symmetric encryption key.

These provide the following benefits:
- Only the client and the server can read the request and response.
- The client is communicating with the server it *thinks* its 
communicating with.

Using HTTPS you can verify the server you are talking to actually owns
the domain you are requesting from it not some other server pretending.

HTTPS on protect from the data. It DOES NOT hide who you are talking
to.



# Handling Errors
When making HTTP requests in Go, it is essential to correctly handle
both network errors and non-OK responses from the sever. Proper error
handling ensures your application can gracefully manage issues and 
provide meaningful feedback.

For the sake of simplicity, all previous lessons have assumed that
all our requests have resulted in successful responses (status code `200` - `299`)


## Network Errors
Network errors happen when there are problems reaching the server, like DNS
failures, connectivity issues, or timeouts. You can detect these errors by
checking the error returned by the HTTP request function.

```go

res, err := http.Get("https://example.com/api/resources")
if err != nil {
	log.Printf("Network error: %w", err)
	return
}
defer res.Body.Close()

```

## Non-OK responses
Even if the request is successful, the server may return a non-OK status
code (e.g 404 Not Found, 500 Internal Server Error). These responses need
to be handled separately from network errors.

```go
res, err := http.Get("https://example.com/api/resources")
if err != nil {
	log.Printf("Network error: %w", err)
}
defer res.Body.Close()

if res.StatusCode != http.StatusOK {
	fmt.Println("status code != 200")
	return
}


```

*A successful response usually has a status code of http.StatusOK (200),
but it can be any code between 200 and 299.*

What you should do with a non-OK response for your request depends on... you. 
You may choose to return an error, try the request again, or just log the error.

# Bugs vs Errors
Error handling is *not* the same as debugging. Likewise, errors are *not* 
the same as bugs.

- Good code with no bugs can still produce errors that are gracefully
handled
- Bugs are, by definition, bits of code that aren't working as intended.

## Debugging
"Debugging" a program is the process of going through your code to find
where it is not behaving as expected. Debugging is manual process performed
by the developer. Sometimes developers use special software called "debuggers"
to help them find bugs, but often they just use print statements to figure
out what's going on.

Examples of Debugging:
- Adding a missing parameter to a function call
- Updating a broken URL that an HTTP call was trying to reach
- Fixing a date-picker component in an app that wasn't displaying properly

# Error Handling
"Error handling" is code that can handle *expected edge* cases your program.
Error handling is an automated process that we  design into our production
code to protect it from things like weak internet connections, bad user input,
or bugs in other people's code that we have to interface with.

Examples of error handling:
- Checking error values and returning early or logging them
- Checking if pointer are not nil


## Don't use Error Handling to Fix Bugs
If your code has a bug, error's won't help you. You need to just go
find the bug and fix it.

Errors are part of the program.
Bugs are not.

# cURL
cURL is a command-line tool for transferring data using various protocols like
HTTP and HTTPS. It's favorite among developers. Among other things, developers
use cURL for

- Quick Testing: make HTTP requests with a single command.
- Automation: can seamlessly be integrated into scripts automation workflows.
- Debugging: easily see requests and responses.

# cURL Basics
The simplest cURL command makes a GET request to the  provided URL,

```bash

curl https://jsonplaceholder.typeicode.com/users/1

```
The response body is written is *stdout*
Because the out written to stdout you can combine it with other commands e.g
redirecting the output a file

```bash

curl https://jsonplaceholder.com/users/1 > user.json

```

# cURL POST request
Making a POST request with cURL is almost as simple as a GET request. 
Use the `-X POST` option to specify the request method and the `-d` 
option to pass data.

Here's how to make a POST request. 

```bash

curl -X POST https://example.com/resource -d 'param1=value1&param2=value2'

```

The command makes an HTTP POST request to `http://example.com/resource`
with the data `param1=value1&param2=value2`. The servers response body
is written to `stdout`

If you need to send JSON data, use the `-H` option to set `Content-Type` header
and the `-d` option to pass the JSON data:

```bash

curl -X POST https://example.com/resource -H "Content-Type: application/json" \
-d '{"key1": "value1", "key2": "value2"}'

```

# jq

`jq` is a powerful command-line tool for processing JSON data. It can
- Parse JSON: easily read and extract data from json response.
- Manipulate JSON: modify JSON data on the fly.
- Filter JSON: find extactly what you're looking for within large JSON
structure.

Here are some of the command that might be useful:
(should probably look up docs for more)

```json

// filename: user.json

{
  "name": "John",
  "age": 30,
  "city": "New York"
}

```

```bash

jq '.name' user.json
# "John"


```

You can use `jq` with `cURL` to parse `JSON`

```bash

curl https://jsonplaceholder.typicode.com/users/1 | jq .username

```

To get field from each element in an array you can use the
`array/object vale iterator` `.[]`, which can in turn be combined 
with identifier

```bash

curl https://jsonplaceholder.typicode.com/users | jq '.[].username'
# "Bret"
# "Antonette"
# "Samantha"
# "Karianne"
# "Kamren"
# "Leopoldo_Corkery"
# "Elwyn.Skiles"
# "Maxime_Nienow"
# "Delphine"
# "Moriah.Stanton"

```
 
```bash

curl https:jsonplaceholder.typicode.com/users  | jq '.name, .email'

```
