---
title: "Requests"
teaching: 0
exercises: 0
questions:
- "How can I send HTTP requests to a web server?"
- "How to interact with web services that require authentication?"
- "What are the data formats that are used in HTTP messages?"
objectives:
- "Use the Python requests library for `get` and `post` requests"
- "Understand how to deal with common authentication mechanisms."
- "Understand what else the `requests` library can do for you."
keypoints:
- "GET requests are used to read data from a particular resource."
- "POST requests are used to write data to a particular resource."
- "GET and POST methods may require some form of authentication (POST usually does)"
- "The Python requests library offers various ways to deal with authentication."
- "curl can be used instead for shell-based workflows and debugging purposes."
---

In this episode we are going to use 
the Python `requests` library
to interact with various web services 
via the HTTP protocol.

## Some terminology

Communication through the HTTP protocol
happens through _messages_, 
which are of two kinds:
_requests_ and _responses_.

A requests is composed of 
a start line, a number of headers and an optional body.
Practically,
a request needs to specify one of the HTTP _verbs_ 
and a URI (Uniform Resource Identifier, the "address" of the resource)
in the start line
and an optional payload (the body).

A response is composed of 
a status line, a number of headers and an optional body. 


## The body: JSON vs XML vs HTML and others

The data to be transferred 
with the body of a request
needs to be represented in some way.
"Unstructured" text representations are used,
e.g., to transmit CSV data.  
A popular text-based (ASCII) format to transmit data 
is the JavaScript Object Notation (JSON) format.
The Python standard library
includes a module to deal with JSON,
for serialisation
(i.e. representing python objects 
as JSON strings):
~~~
import json
data = dict(a = 1, b = dict(c = (2,3,4)))
representation = json.dumps(data)
representation
~~~
{: .language-python}

~~~
'{"a": 1, "b": {"c": [2, 3, 4]}}'
~~~
{: .output}
And for parsing
(i.e. recovering python objects 
from their JSON string representation):
~~~
data_reparsed = json.loads(representation)
data_reparsed
~~~
{: .language-python}
~~~
{'a': 1, 'b': {'c': [2, 3, 4]}}
~~~
{: .output}
The Python `requests` library 
can parse JSON and serialise the objects,
so that you don't have to deal with this aspect on your own.

Another ASCII format that is used with APIs
is the eXtensible Markup Language (XML),
which is much more complex to deal with than JSON.
Facilities to deal with the XML format are 
in the `xml.etree.ElementTree` library.

Another markup language widely used 
in HTTP message bodies
is the HyperText Markup Language, HTML.

## HTTP verbs
The HTTP protocol has a number of _verbs_, 
each associated with an operation falling in 
one of the 4 CRUD categories 
(Create, Read, Update, Delete),
the most common of which are:
- GET: to read resources (no body);
- POST: to create new resources;
- PUT: to update/replace existing resources; 
- PATCH: to update/modify existing resources;
- DELETE: to delete resources  
We will now focus on GET and POST requests only.

## A Get request example

A previous example, 
now with the Python `requests` library:
~~~
import requests
response = requests.get("http://www.carpentries.org")
~~~
{: .language-python}
As this is the url of a website, 
we expect the reponse to contain a web page:
~~~
response.headers["Content-Type"]
~~~
{: .language-python}
~~~
text/html
~~~
{: .output}
We can also check the `Content-Length` header:
~~~
response.headers["Content-Length"]
~~~
{: .language-python}
~~~
55036
~~~
{: .output}
And, as expected,
the length of the body of the response
is the same:
~~~
len(response.text)
~~~
{: .language-python}
~~~
55036
~~~
{: .output}
We can look at the content of the body:
~~~
response.text
~~~
{: .language-python}
Which is going to show a HTML document.



## Get with parameters 
It is now time to use the Metoffice api key 
you have obtained in the Setup phase.
Load the api key into memory:
~~~
api_key = open("metoffice-api-key.txt",'r').read().strip()
~~~
{: .language-python}
Looking at the [Metoffice API reference](
https://www.metoffice.gov.uk/services/data/datapoint/api-reference
),
we build the url to access the forecasts
for Swansea
~~~
base_metoffice_url = "http://datapoint.metoffice.gov.uk/public/data/"
resource = "val/wxfcs/all/json/310149"
url = base_metoffice_url + resource
url
~~~
{: .language-python}

As shown in the API reference,
this time we need to pass 2 parameters in the request:
a resource description and the api key,
in order for the Met Office server to identify us.
With `curl` from the command line,
we would have to use the following command
~~~
curl "http://datapoint.metoffice.gov.uk/public/data/val/wxfcx/all/json/310149?res=3hourly&key=$(cat metoffice-api-key.txt)" | less
~~~
{: .language-bash}
building the parameter string explicitly.
This is also the syntax that is used 
in a browser address bar:
~~~
"protocol://host/resource/path?parname1=value1&parname2=value2..."
~~~
by using the requests library,
we can use a nicer syntax:
~~~
response = requests.get(url, params={"res":"3hourly", "key":api_key})
response
~~~
{: .language-python}
~~~
<Response [200]>
~~~
{: .output}
The code 200 means "success".
To make sure the response contains what we expect,
let's quickly print its headers 
(which has the structure of a dictionary):
~~~
{print(kv) for kv in response.header.items()}
~~~
{: .language-python}
~~~
('Server', 'WaveServer 1.0')
('ETag', '1615686466177')
('Content-Type', 'application/json')
('WebServer', '-PROD-01')
('Access-Control-Allow-Origin', '*')
('Content-Encoding', 'gzip')
('Content-Length', '1051')
('Cache-Control', 'public, no-transform, must-revalidate, max-age=611')
('Expires', 'Sun, 14 Mar 2021 12:38:16 GMT')
('Date', 'Sun, 14 Mar 2021 12:28:05 GMT')
('Connection', 'keep-alive')
('Vary', 'Accept-Encoding')

{None}
~~~
{: .output}
As expected the `Content-Type` is `application-json`.
We can now look at the body of the response:
~~~
response.text[:100]
~~~
{: .language-python}
~~~
'{"SiteRep":{"Wx":{"Param":[{"name":"F","units":"C","$":"Feels Like Temperature"},{"name":"G","units"
~~~
{: .output}
As mentioned, the `requests` library 
can parse this JSON representation 
and return a much wieldier python object,
of which we can access the inner data:
~~~
data = response.json()
data["SiteRep"]["Wx"]
~~~
{: .language-python}

> ## Another location
> As described in the API reference 
> (https://www.metoffice.gov.uk/services/data/datapoint/api-reference)
> the Met Office has a list of locations available 
> at "/public/data/val/wxfcs/all/json/sitelist".
> Choose a site near you.
> What is the expected temperature 
> tomorrow at 11 AM?
>
> Hint: Once you have the right data,
> you can use
> ~~~
> data["SiteRep"]["DV"]["Location"]["Period"][1]["Rep"][3]["T"]
> ~~~
> {: .language-python}
> to get to the quantity of intest.
>
> > ## Solution
> > We query the MetOffice API using
> > ~~~
> > sitelist_url = base_metoffice_url + 'val/wxfcs/all/json/sitelist'
> > site_response = requests.get(sitelist_url, 
> >                              params = dict(key = api_key)) 
> > sitelist = site_response.json()["Locations"]["Location"] # sic, unfortunately
> > ~~~
> > {: .language-python}
> > We can look for a location,
> > e.g. Cardiff:
> > ~~~
> > [ site for site in sitelist if site["name"] == "Cardiff"] 
> > ~~~
> > {: .language-python}
> > One of the location ID is `350758`,
> > so we can use it: 
> > ~~~
> > resource = "val/wxfcs/all/json/350758"
> > url = base_metoffice_url + resource
> > response = requests.get(url, params={"res":"3hourly", "key":api_key})
> > data = response.json()
> > ~~~
> > {: .language-python}
> > Now we must explore the data 
> > to find the information we need.
> > It turns out that it is in
> > ~~~
> > data["SiteRep"]["DV"]["Location"]["Period"][1]["Rep"][3]["T"]
> > ~~~
> > {: .language-python}
> > The meanings of the keys in each dictionary
> > can be found in 
> > ~~~
> > data["SiteRep"]["Wx"]
> > ~~~
> > {: .language-python}
> {: .solution}
{: .challenge}

## Authentication and POST 

For POST requests, 
we will use the function `requests.post`.
We will need the GitHub personal access token 
you have set up previously.

For this example, we are going 
to post a comment on an [issue on GitHub](https://github.com/mmesiti/aimlac-cdt-2021-03/issues/1).
Issues on GitHub are a simple way 
to keep track of bugs, 
and a great way to manage focused discussions
on the code. 

In order to do so, we need to authenticate.
We will now create an object 
of the `HTTPBasicAuth` class
provided by `requests`,
and pass it to `requests.post`.

First of all, let's load the github access token:
~~~
ghtoken = open("github-access-token.txt",'r').read().strip()
~~~
{: .language-python}
Let's then create the HTTPBasicAuth object:
~~~
from requests.auth import HTTPBasicAuth
auth = HTTPBasicAuth("your-github-username",ghtoken)
~~~
{: .language-python}
We will now create the body of the comment,
as a JSON string:
~~~
body = json.dumps({u"body":u"Another test comment"})
~~~
{: .language-python}
Finally, we will post the comment on github
and make sure we get a success code:
~~~
response = requests.post(url="https://api.github.com/repos/mmesiti/aimlac-cdt-2021-03/issues/1/comments",
              data=body,
              auth=auth)
response
~~~
{: .language-python}
~~~
<Response [201]>
~~~
{: .output}
The code 201 is the typical success response 
for a POST request, 
signaling that the creation of a resource 
has been successful.
We can go to the [issue page](https://github.com/mmesiti/aimlac-cdt-2021-03/issues/1)
and check that our new comment is there.

> ## Curl and POST
> Curl can be used as well for POST requests,
> which can be useful for shell-based workflows.
> One needs to use the `--data` option.
{: .callout}


> ## What have I asked you?
> 
> The request that generated a given `response` object
> can be retrieved as `response.request`.
> Can you see the headers of that request?
> And what about the body of the message?
> What is the type of the request object?
> > ## Solution
> > To print the headers:
> > ~~~
> > { print(kv) for kv in response.requests.headers.items}
> > ~~~
> > {: .language-python}
> > ~~~
> > ('User-Agent', 'python-requests/2.25.1')
> > ('Accept-Encoding', 'gzip, deflate')
> > ('Accept', '*/*')
> > ('Connection', 'keep-alive')
> > ('Content-Length', '26')
> > ('Authorization', 'Basic XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX')
> > ~~~
> > {: .output}
> > The body of the request is accessible just with
> > ~~~
> > response.requests.body
> > ~~~
> > {: .language-python}
> > ~~~
> > '{"body": "A test comment"}'
> > ~~~
> > {: .output}
> > ~~~
> > And the type is PreparedRequest:
>> > ~~~
> > type(response.requests)
> > ~~~
> > {: .language-python}
> > ~~~
> > request.models.PreparedRequest
> > ~~~
> > {: .output}
> > For better control,
> > one could in principle create 
> > a `Request` object beforehand,
> > call the `prepare` method on it
> > to obtain a `PreparedRequest`,
> > and then send it through a `Session` object.
> {: .solution} 
{: .challenge}

> ## Forgot the key
> What error code do we get 
> if we just forget to add the auth?
> How do the headers of the request change?
> > ## Solution
> > ~~~
> > r = requests.post(url=url,data=body)
> > r
> > ~~~
> > {: .language-python}
> > ~~~
> > <Response [401]>
> > ~~~
> > {: .output}
> > The request headers are:
> > ~~~
> > ('User-Agent', 'python-requests/2.25.1')
> > ('Accept-Encoding', 'gzip, deflate')
> > ('Accept', '*/*')
> > ('Connection', 'keep-alive')
> > ('Content-Length', '26')
> > ~~~
> > {: .output}
> > Most notably, the "Authorization" header is missing.
> {: .solution}
{: .challenge}

Authentication is a vast topic.
The requests library implements a number 
of authentication mechanisms that you can use.
To handle authentication for multiple requests,
one could also use a `Session` object 
from the `requests` library
(see [Advanced Usage](https://requests.readthedocs.io/en/master/user/advanced/)).
