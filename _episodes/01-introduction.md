---
title: "HTTP"
teaching: 0
exercises: 0
questions:
- "What are protocols and ports?"
- "What are HTTP and HTTPS?"
- "What are requests and responses? How can we look at them?"
objectives:
- "Understand the meaning of the terms protocol and port."
- "Understand what HTTP and HTTPS are, and how it relates to the Web and other aspects of the modern Internet."
- "Be able to use `curl` to make requests and view responses."
keypoints:
- "A protocol is a standard for communicating data across a network. A port is a number to identify which program should process a network connection."
- "HTTP is the protocol originally designed for requesting and receiving Web pages, but now also used as the basis for a variety of APIs. HTTPS is the encrypted version of HTTP."
- "Every page on the world wide web is identified with a URL or Uniform Resource Locator."
- "A request is how you tell a server what you want to see. A response will either give you what you asked for, or tell you why the server can't do that. Both requests and responses have a header, and optionally a body."
- "We can make requests and receive responses, as well as see their headers, using `curl`."
---

First introduced to the world in 1991, the World Wide Web brought together three key ideas:

1. The use of HTML (Hypertext Markup Language) documents which could container hyperlinks to other documents (or different parts of the same document). These could reference documents located on any web server in the world.
2. That every file on the world wide web would have a unique URL (Uniform Resource Locator). 
3. The Hypertext Transfer Protocol (HTTP) that is used to transfer data from the web server to the requesting client.

Since then it has gone from the toy of computer scientists and particle physicists to a dominant part of
everyday life for billions of people. It has gradually consumed many services
that were previously separate online services, or not available on the Internet
at all. Since the mid-2000s, the Web has increasingly been used to go beyond its
traditional usage of serving web pages to browsers. The same HTTP protocol which once 
served static HTML pages and images is now used to send dynamic content generating on 
the fly by other computer programs.

These Application Programming Interfaces (APIs) provide incredible amounts of
structured data, as well as the ability to control things that may previously
have required specialist proprietary software or even hardware. In particular,
the data available via web APIs is particularly useful for data scientists; many
data are now only made available via these APIs, and even in cases where data
are made available in other formats, using an API is frequently more convenient.

To make effective use of web APIs, we need to understand a little more about how
the Web works than a typical Web user might. This lesson will focus on
_clients_&mdash;computers and software applications that make requests to other
computers or applications, and receive information in response. Computers and
applications that respond to such requests are referred to as _servers_.


## Protocols and ports

You may (or may not) have wondered how it is that different web browsers,
written independently by different companies and running on different operating
systems, are able to talk to the same web servers using the same addresses, and
get the same web pages back. This is because all web browsers implement the
_HyperText Transfer Protocol_, or HTTP.

A protocol is nothing more than a system of rules that allow for communication
between computers (or other devices). Much like a (human) language, it defines
rules and syntax that when all parties follow, allow information to be
transmitted from one device to another. Other examples of protocols you may be
familiar with include the Secure Shell SSH, the File Transfer Protocol FTP, and
the Simple Mail Transfer Protocol SMTP. Wikipedia has a [long
list][protocol-list] of protocols that are (or once were) in common usage. HTTPS
is a protocol closely related to HTTP; it follows many of the same conventions
as HTTP, particularly in the way client and server code is written, but includes
additional encryption to ensure that untrusted third parties can't read or
modify data in transit.

Given the large number of protocols in existence, computers need a way to
identify which protocol a particular network connection is using, in particular
on devices that have many different servers running. This is done by another set
of protocols, which the above protocols build on top of: the Transmission
Control Protocol (TCP) and the User Datagram Protocol (UDP). The difference
between these isn't important today; the important fact is that both protocols
define _port numbers_ (or _ports_) that are used to identify which server should
handle a particular connection.

A server application must register a particular port to listen for connections
on, and then all connections with that port number will be directed to that
application. Ports are numbered 1&ndash;65,535, with ports up to 1,023 being
"system ports" that on Unix-like systems require root access to listen to. Many
protocols have standard ports that are used by convention&mdash;for example,
HTTP uses port 80 by default, and HTTPS port 443. However, there is nothing
stopping any protocol being used on any port.

You may have noticed that web addresses sometimes include a colon and a number
after the server name; this indicates to the browser which port to connect on,
in cases where you don't want to connect to the default port (80 or 443). For
example, Jupyter notebooks are frequently served at `http://localhost:8888`;
this indicates that your browser should make an HTTP connection to your own
local machine, on port 8888. Since only one application can listen to a port at
a time, sometimes Jupyter finds it can't listen on port 8888, and so will
reserve port 8889 or 8890 instead.

## URLs

A URL (also sometimes known as a URI or Uniform Resource Indicator) consists of two or three parts: the protocol followed by ://, the server name or IP address and optionally the path to the resource we wish to access. For example the URL http://carpentries.org means we want to access the default location on the server carpentries.org using the HTTP protocol. The URL https://carpentries.org/contact/ means we want to access the contact location on the carpentries.org server using the secure HTTPS protocol.

## Requests and responses

The two main objects in HTTP are the _request_ and the _response_. Each HTTP connection is initiated by sending a request, and is replied to with a response. Both the request and response have a _header_, that defines metadata about what is requested and what is included in the response, and both can also have a _body_, containing data. To look at these in more detail, we can use the `curl` command. Specifically, to see the request headers, we can use `curl -v` followed by the URL we wish to request.

~~~
$ curl -v http://carpentries.org
~~~
{: .language-bash}

~~~
*   Trying 13.32.168.28...
* TCP_NODELAY set
* Connected to carpentries.org (13.32.168.28) port 80 (#0)
> GET / HTTP/1.1
> Host: carpentries.org
> User-Agent: curl/7.64.1
> Accept: */*
>
< HTTP/1.1 301 Moved Permanently
< Server: CloudFront
< Date: Sat, 13 Mar 2021 01:10:22 GMT
< Content-Type: text/html
< Content-Length: 183
< Connection: keep-alive
< Location: https://carpentries.org/
< X-Cache: Redirect from cloudfront
< Via: 1.1 f25763791d7f1173b560742bb9507145.cloudfront.net (CloudFront)
< X-Amz-Cf-Pop: LHR62-C5
< X-Amz-Cf-Id: JJLCGx6qUOpaid_ArD0kph8QddidHgWnKoi72yNn0Jazmla8H5mUGg==
<
<html>
<head><title>301 Moved Permanently</title></head>
<body bgcolor="white">
<center><h1>301 Moved Permanently</h1></center>
<hr><center>CloudFront</center>
</body>
</html>
* Connection #0 to host carpentries.org left intact
* Closing connection 0
~~~
{: .output}

Lines starting `> ` here are request headers, and lines starting `< ` are response headers. Following this is the body (the section from `<html>` to `</html>`), which in this case is a short web page.

In this case, after identifying what type of request this is (a `GET` request), the location to look for (`/`), and the HTTP version, we include three headers: the first states the domain name we are looking to contact (in case one server is serving multiple domain names, as is quite common), the second identifies what software we're using to connect (as some servers will adjust the content depending on, for example, which browser you connect with), and the third tells the server what we're looking for&mdash;in this case we will accept whatever the server has to offer.

The server then responds with a status code, followed by a lot of metadata. In this case, the status code `301` indicates that the site is no longer at the location we tried, so the metadata includes where to look instead. This is followed by a short web page explaining the same thing. Most browsers will see the `301` and automatically redirect to the correct location so you never see this error message.

Let's see what happens when we follow the redirect. Web pages can be quite long, so for now let's ignore the body and look only at the headers.

~~~
$ curl -v https://carpentries.org > /dev/null
~~~
{: .language-bash}

In this case, because we're connecting via HTTPS, `curl` gives a lot more debugging information about the secure connection, but after this we see similar request headers (although this time we're using HTTP/2), and then the response headers start with `HTTP/2 200`, with the status code `200` indicating that this was a successful request, with the body providing what we asked for.

HTTP status codes are three digits long, and almost always begin with 2, 3, 4, or 5. Status codes beginning `2xx` indicate that the request was successfully received, understood, and accepted; `3xx` indicates a redirect of some kind; `4xx` indicates an error caused by the client (for example the famous `404 Not found` where the client has requested a resource that does not exist on the server), and `5xx` indicates an error on the server side.

It's rarely necessary to inspect the request, so if you're interested in the headers, it's more convenient to use `curl -I` to just show the response headers.

~~~
$ curl -I https://carpentries.org
~~~
{: .language-bash}

~~~
HTTP/2 200
content-type: text/html
content-length: 55036
date: Sat, 13 Mar 2021 01:32:50 GMT
last-modified: Sat, 13 Mar 2021 01:26:59 GMT
etag: "f16c8eaddc88e035134aa23e0f8a94ba"
server: AmazonS3
x-cache: Hit from cloudfront
via: 1.1 a25f829e86f504a329e71fa3f4d21485.cloudfront.net (CloudFront)
x-amz-cf-pop: LHR62-C5
x-amz-cf-id: WGyZEdVLxTFbdQ3eKX2rdnPWO0214DDcQi8TA5UpObYt2CgHjCUz7g==
age: 87
~~~
{: .output}

Noteworthy here is the first header `content-type: text/html`; this indicates that the response body is an HTML document (also known as a web page). HTML, the HyperText Markup Language, is the language that all web pages are written in; while we won't write any today, we will look a little more at how to read it (and get your code to read it) in a later episode.

> ## HyperText?
>
> Both HTTP and HTML refer to HyperText. This was a popular buzzword in the 1990s, and refers to the Web's ability to include not only text, but also cross-references in the form of links (_hypertext links_, or _hyperlinks_) to other documents stored elsewhere, which the user can immediately access.
>
> While this seems entirely obvious and second-nature today, it was revolutionary when it was first introduced, hence the name appearing prominently in technologies that supported it.
{: .callout}

> ## Another website
>
> Pick a web page you've visited recently and take a look at its response headers with `curl -I`. How do they differ from the `https://carpentries.org/` headers we looked at above? What parts are similar?
{: .challenge}

{% include links.md %}

[protocol-list]: https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers
