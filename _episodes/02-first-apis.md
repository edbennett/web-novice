---
title: "What do APIs look like?"
teaching: 0
exercises: 0
questions:
- "How can requests be made of web APIs?"
- "How can responses from web APIs arrive?"
- "How can requests to web APIs be authenticated?"
objectives:
- "Be able to make requests to web APIs using `curl` using endpoints, query parameters, and JSON data."
- "Be able to identify responses in plain text and JSON."
- "Be able to authenticate to web APIs with passwords and authentication tokens."
keypoints:
- "Interact with web APIs by sending requests to an _endpoint_ representing a function of interest. Parameters can be encoded into the request, or attached as e.g. JSON."
- "Responses are typically plain text or JSON, but could be anything."
- "Most APIs require some form of authentication. This can be by username and password, or via a token."
- "Which choices a given API makes for each of these will be described in the API's documentation."
---

We've done a lot of talking about the technologies that will let us interact
with APIs so far. Let's now start putting this into practice and query an API.

~~~
$ curl http://numbersapi.com/42
~~~
{: .language-bash}

~~~
42 is the number of laws of cricket.
~~~
{: .output}

[Numbers API][numbersapi] provides facts about numbers. By putting the number of
interest into the address, we tell Numbers API which number to give a fact
about. By adding other keywords to the address, we can refine the domain that
we're asking for information in; for example, for specifically mathematical
trivia, we can add `/math`.

~~~
$ curl http://numbersapi.com/42/math
~~~
{: .language-bash}

~~~
42 is a perfect score on the USA Math Olympiad (USAMO) and International Mathematical Olympiad (IMO).
~~~
{: .output}

Numbers API is not an especially sophisticated API. In particular, it only
offers a single _endpoint_ (specifically, `/`), and each response to a query is
a single string, provided as plain text.

We can think of an API as being similar to a package or library in a programming
language, but one that is usable from almost any programming language. In these
terms, an endpoint is equivalent to a function; Numbers API provides a single
function, `/`, which gives information about numbers. The response is the return
value of the function, and in this case is a single string. This maps well onto
HTTP, as the response body of a request is a string of either characters or of
bytes. (Byte strings don't translate well between languages, so are usually
avoided, except for specific portable formats such as images.)

However, many useful functions need to return something other than character
strings. For example, you might want to return a list, or an array, or a set of
related data. Let's look at another example of a web API and see how this can be
handled. [Newton][newton] is a web API for advanced mathematics. One thing it
can do is factorization:

~~~
$ curl https://newton.now.sh/api/v2/factor/x^2-1
~~~
{: .language-bash}

~~~
{"operation":"factor","expression":"x^2-1","result":"(x - 1) (x + 1)"}
~~~
{: .output}

Two things have changed. Firstly, now instead of `/`, we are specifying that we
want to use the `factor` endpoint provided by the `v2` version of the API. This
is a very common way of structuring APIs: firstly a version, and then one or
more levels of endpoints to specify what function you would like the API to
perform.

Secondly, rather than a plain text response, we get a data structure. This is
still encoded as plain text (because HTTP can't natively transmit much else),
but we can't use the text directly&mdash;instead, we need to parse it, first.
The syntax used here is the most common format for modern web APIs, and is
called JSON (pronounced like the name "Jason"; short for JavaScript Object
Notation). (You may also encounter older or more old-fashioned APIs that instead
use XML, the eXtensible Markup Language.) We can see that this response includes
three names, or _keys_ (`"operation"`, `"expression"`, and `"result"`), and
three associated _values_ (`"factor"`, `"x^2-1"`, and `"(x - 1) ( x + 1)"`,
respectively).

`factor` is not the only thing that Newton can do. Let's try a different
endpoint, for integration.

~~~
$ curl https://newton.now.sh/api/v2/integrate/x^2-1
~~~
{: .language-bash}

~~~
{"operation":"integrate","expression":"x^2-1","result":"1/3 x^3 - x"}
~~~
{: .output}

In this case Newton correctly tells us that the `"result"` of this integration
is `"1/3 x^3 - x"`.

The endpoints an API offers, and what format it will give its responses in, will
generally be listed in the API's documentation. Newton's documentation for
example can be found [on GitHub][newton-docs].

> ## More math
>
> Read through [Newton's documentation][newton-docs]. Try one or more of the
> other endpoints that we haven't tried. Check that the results match what you
> would expect.
>
> Try using a different input function than `x^2-1`. Again, check that the
> answers give what you expect.
{: .challenge}

> ## Errors (or not)
>
> Try using the `simplify` endopint for Newton to simplify the expression
> `0^(-1)` (i.e. 1 divided by 0).
>
> Use `curl -i` to see both the headers and the response. Do these match what
> you expect?
>
>> ## Solution
>>
>> The response code for this request is `200` (OK), but the `"result"`
>indicates that an error occurred.
>>
>> This is not uncommon; not all APIs will use the HTTP status code to indicate
>> an error condition. Some will even give you an HTML web page describing an
>> error condition when usually you would expect a non-HTML response. It's good
>> to check this behaviour for each API that you use, so that you can guard for
>> it in your software.
> {: .solution} 
{: .challenge}


## Authentication and identification

Many web APIs restrict access to registered users or applications. This may be because they are used to control things that are specific to a particular user account, because different people have different privilege levels and so different endpoints available, or simply because the API provider wants to collect statistics on how the API is being used.

Various ways exist for developers to authenticate to an API, including:

* A username and password, via HTTP [basic][basic-auth] or [digest][digest-auth] authentication;
* An authentication token (to identify you) or API key (to identify your application), generated on a developer dashboard page; and
* OAuth tokens, generated programmatically&mdash;these are particularly useful for applications used by others to log into their own accounts, so your application never sees the credentials used.

For everything other than HTTP authentication, there are also a variety of ways to present the credential to the server, such as:

* As a query parameter;
* In an extra header in the request;
* Encoded in, for example, JSON in the request body;
* As a [cookie][cookie]; and
* As an HTTP password, with or without a username.

One important fact about HTTP is that it is _stateless_: each request is treated entirely separately, with no memory from one request to the next. This means that you must present your authentication credentials with every request you make to the API. (This is in contrast to other protocols like SSH or FTP, where you authenticate once at the start of a session, and then subsequent messages can be sent back and forth without the need for re-authentication.)

For example, NASA offers an API that exposes much of the data that they make public. They require an API key to identify you, but don't require any authentication beyond this.

Let's try working with the NASA API now. To do this, first we need to generate our API key by providing our details at [the API home page][nasa-api]. Once that is done, NASA provide the API key instantly, and send a copy to the email address you provide. They helpfully also provide an example of an API query to try, querying the Astronomy Picture of the Day (APOD). This shows us that NASA expects the API key to be encoded as a query parameter.

~~~
$ curl -i https://api.nasa.gov/planetary/apod?api_key=ejgThfasPCRf4kTd39ar55Aqhxv8cwKBdVOyZ9Rr
~~~
{: .language-bash}

~~~
HTTP/1.1 200 OK
Date: Mon, 15 Mar 2021 00:08:34 GMT
Content-Type: application/json
Content-Length: 1135
Connection: keep-alive
Vary: Accept-Encoding
X-RateLimit-Limit: 2000
X-RateLimit-Remaining: 1998
Access-Control-Allow-Origin: *
Age: 0
Via: http/1.1 api-umbrella (ApacheTrafficServer [cMsSf ])
X-Cache: MISS
Strict-Transport-Security: max-age=31536000; preload

{"copyright":"Mia St\u00e5lnacke","date":"2021-03-14","explanation":"It appeared, momentarily, like a 50-km tall banded flag.  In mid-March of 2015, an energetic Coronal Mass Ejection directed toward a clear magnetic channel to Earth led to one of the more intense geomagnetic storms of recent years. A visual result was wide spread auroras being seen over many countries near Earth's magnetic poles.  Captured over Kiruna, Sweden, the image features an unusually straight auroral curtain with the green color emitted low in the Earth's atmosphere, and red many kilometers higher up. It is unclear where the rare purple aurora originates, but it might involve an unusual blue aurora at an even lower altitude than the green, seen superposed with a much higher red.  Now past Solar Minimum, colorful nights of auroras over Earth are likely to increase.   Follow APOD: Through the Free NASA App","hdurl":"https://apod.nasa.gov/apod/image/2103/AuroraFlag_Stalnacke_6677.jpg","media_type":"image","service_version":"v1","title":"A Flag Shaped Aurora over Sweden","url":"https://apod.nasa.gov/apod/image/2103/AuroraFlag_Stalnacke_960.jpg"}
~~~
{: .output}

We can see that this API gives us JSON output including a links to two versions of the picture of the day, and then metadata about the picture including its title, description, and copyright. The headers also give us some information about our API usage&mdash;our rate limit is 2000 requests per day, and we have 1998 of these remaining (probably because the malware scanner on my email server tested the link first to make sure it wasn't malicious).

With all of these ways to provide identification and authentication information, we don't have time to cover each possibility exhaustively. For the vast majority of APIs, there will exist good developer documentation that provides examples of how to use the token or other identifier that they provide to connect to their service, including examples.


## More complicated queries

Thus far we have queried APIs where any parameters are included as part of the effective "filename" on the server. For example, in `http://numbersapi.com/42`, the `42` is a parameter to the API, but at first glance it could equally well be an endpoint.

Many APIs make this distinction more clear, by accepting arguments in a _query string_. This is a sequence of `name=value` pairs, separated from each other by `&`s, and separated from the endpoint by a `?`.

We have already seen one example of this&mdash;we used it to provide our API key to NASA's APOD endpoint. The APOD endpoint also accepts other parameters, for example, to select the date or dates for which the picture is returned.

~~~
$ curl -i https://api.nasa.gov/planetary/apod?date=2005-04-01&api_key=ejgThfasPCRf4kTd39ar55Aqhxv8cwKBdVOyZ9Rr
~~~
{: .language-bash}

~~~
HTTP/1.1 200 OK
Date: Mon, 15 Mar 2021 00:31:45 GMT
Content-Type: application/json
Content-Length: 965
Connection: keep-alive
X-RateLimit-Limit: 2000
X-RateLimit-Remaining: 1996
Access-Control-Allow-Origin: *
Age: 0
Via: http/1.1 api-umbrella (ApacheTrafficServer [cMsSf ])
X-Cache: MISS
Strict-Transport-Security: max-age=31536000; preload

{"copyright":"Ellen Roper","date":"2005-04-01","explanation":"Can you help discover water on Mars?  Finding water on different regions on Mars has implications for understanding its complex geologic history, the possible existence of past life and the sustenance of potential future astronauts.  Many space missions have taken photographs of the surface of the red planet, and some of them might show a subtle clue pointing to water on Mars that has been missed.  By close inspection of images, following curiosity, applying scientific principles, applying knowledge about features on the Martian surface, and applying principles of planetary geology, such clues might be brought to light.  In the meantime, happy April Fool's Day from the folks at APOD!","hdurl":"https://apod.nasa.gov/apod/image/0504/WaterOnMars2_gcc_big.jpg","media_type":"image","service_version":"v1","title":"Water On Mars","url":"https://apod.nasa.gov/apod/image/0504/WaterOnMars2_gcc.jpg"}
~~~
{: .output}

One benefit of being able to construct queries in this way is that the query is more self-descriptive&mdash;for unfamiliar APIs, keyword arguments are significantly easier to read than positional ones.

One other way to provide parameters, in particular when they are more complex data structures than can be easily represented in a small string, is to use JSON in the body of the request. Since constructing JSON by hand is tedious, we will defer such APIs to the next section.


> ## NASA aerial imagery
>
> Look through NASA's API documentation. Use the Earth API to retrieve an aerial image of your current location.
>
> Try first using `curl` without any flags. What message do you get from `curl`? Why might this be?
>
> Now try inspecting the headers for the request using `curl -I`, and look at the `Content-Type`. Does this match your suspicion as to the reason for `curl`'s message?
>
> Finally, follow `curl`'s advice to save the output to a file. Open the resulting file and see if it matches what you expected.
{: .challenge}



[basic-auth]: https://en.wikipedia.org/wiki/Basic_access_authentication
[cookie]: https://en.wikipedia.org/wiki/HTTP_cookie
[digest-auth]: https://en.wikipedia.org/wiki/Digest_access_authentication
[nasa-api]: https://api.nasa.gov
[newton]: https://newton.now.sh
[newton-docs]: https://github.com/aunyks/newton-api
[numbersapi]: http://numbersapi.com
