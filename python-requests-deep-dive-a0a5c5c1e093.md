
# Python requests deep dive

I just finished replacing httplib in a very large project, Apache Libcloud.

When httplib was selected, requests wasn’t around ([it only hit v1 in 2012](http://docs.python-requests.org/en/latest/community/updates/#id49)). We needed to provide a set of base classes that would handle HTTP and HTTPS REST/JSON, REST/XML and various other bizarre HTTP APIs. Libcloud has over 80 client libraries for every major cloud service out there. Each of those libraries share a single Connection class that handles encoding and decoding of JSON, XML or Raw data. Whipping the tablecloth out from under our diners was going to be tricky. Especially having experienced first hand how awful some APIs are ([see related post](https://medium.com/@anthonypjshaw/8-reasons-your-api-sucks-7f6ff60ddc04#.7d98c25m5)) I was tentative that this could be done without knock on issues for our users. Libcloud has excellent test coverage, with each driver typically having around 90% and the base utilities are tested with all expected and supported behaviours across all major version of Python 2 and 3.

### Making requests in requests

There are a few ways to make HTTP/HTTPS requests in requests. Firstly, it is worth noting that in httplib, you have to spend a lot of time deciding how you would handle TLS or SSL decryption and trust for HTTPS endpoints.

This is no longer the case in requests. The abstraction of HTTPS handling, for us was the big appeal and also for our users.

<iframe src="https://medium.com/media/774d84af425c57ca0b124d6388f3ebe2" frameborder=0></iframe>

This is actually just a friendly wrapper around the main request method with the HTTP verb as the first argument. I have seen some bad examples of people writing long if/elif/elif around the HTTP Verb but request is part of the main API.

<iframe src="https://medium.com/media/5d81dd13b0807c0ec3b37e66a9b7985b" frameborder=0></iframe>

Requests seems to have modelled a lot of it’s API around a web browser. You can set things like cookies, redirect, HTTPS is handled cleanly.

Calling [requests.request in turn creates a Session](https://github.com/kennethreitz/requests/blob/master/requests/api.py#L55-L56). If you are making multiple requests to the same endpoint you are better to use a session since it will hold open the TCP session between connections, keep a cookie jar and also remember any preferences for each request.

All of the APIs that Libcloud talks to have some form of authentication, perhaps Basic HTTP (base 64 username/password), OAuth, API keys. All of these would require configuration of custom headers for the lifetime of the session, so we decided early to leverage the requests Session object for all requests. So now each Libcloud driver connection will have it’s own Session in the background. I realised early on that whilst the high-level APIs were great for a single request, I needed to get more familiar with the way requests operates to get the outcome I needed. If you use requests often, read on.

### Understanding the Session object

I often see code snippets like this, ignoring sessions and the JSON-native features.

<iframe src="https://medium.com/media/54d33db6960fcd158e4d0f482d1c47a7" frameborder=0></iframe>

Providing a dictionary to **json** argument will encode the data for you, using a Session context-manager you can set default **headers** by the headers property.

<iframe src="https://medium.com/media/7787f2d77c0af9b0280965255eb5038f" frameborder=0></iframe>

Within the Session initialiser it will assign a set of default headers including the flags to allow gzip responses and use keep-alive. *Avoid setting sess.headers to a new dictionary*, especially since the default property is a **case-insensitive** dictionary which comes in mighty-handy doing lookups.

![](https://cdn-images-1.medium.com/max/2000/1*bweD04YrlNc7ZGbbW9kQZw.png)

### Understanding PreparedRequest

When you keep peeling back the onion, Session.request creates an instance of the PreparedRequest object. You can construct these objects by hand, typically when you have a specific binary payload to send and set of headers.

<iframe src="https://medium.com/media/063737151a2103d7cdbdb41cc36e8f1a" frameborder=0></iframe>

## Handling large requests, streams and iterators

If you’re uploading or downloading large requests or responses, requests provides some utilities for working with streams or generic iterators.

### Uploading with streams

Requests will [detect](https://github.com/kennethreitz/requests/blob/master/requests/models.py#L450) when the data argument is an iterator like a file stream object or a BytesIO or StringIO stream.

<iframe src="https://medium.com/media/1b0c0c72d7591732978e8bfced6bdf76" frameborder=0></iframe>

### Downloading with streams

Making a request with the stream flag will allow **iter_content** on the Response object to iterate the data from an open connection. **iter_content** will still work without stream, but it will download the [response into memory firs](https://github.com/kennethreitz/requests/blob/master/requests/models.py#L711-L717)t (probably doubling the execution time to achieve the same result).

<iframe src="https://medium.com/media/b270cd2ed3337986d087158a74b6ae99" frameborder=0></iframe>

## Testing with requests

You can easily mock out responses to requests within your tests using the [requests_mock](https://github.com/openstack/requests-mock) package maintained by the OpenStack folks

<iframe src="https://medium.com/media/4424f18deba15207efe08ced40a45c98" frameborder=0></iframe>

There are 2 primary ways of replacing/extending the default behaviours, using hooks or using

* Using a library like requests_mock to replace the [default transport adapters w](http://docs.python-requests.org/en/master/user/advanced/#transport-adapters)ith ones that return static test data

* [Setting hooks on a session object](http://docs.python-requests.org/en/master/user/advanced/#event-hooks)

### Footnote on Old(er) versions of Python

“Requests officially supports Python 2.6–2.7 & 3.3–3.5, and runs great on PyPy.”, as part of this change we ran tests on 3.6 (it works fine) and dropped support for Python 3.2, which was long overdue.
