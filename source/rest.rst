Representational State Transfer (REST)
======================================

About a couple of decades ago, an application or software product (online or
offline) was generally conceived of and delivered in isolation, as an entity in
its own right. We had text editors, internet search, Photoshop, Pro Tools, MS
Word, MS Excel, MS Powerpoint, all could be thought of by the end user as
existing in independent silos, with some bit of exchange facilitated by the
operating system via the file system or the "clipboard".

In the past decade or so, applications and products are no longer thought of in
isolation. The attention to interoperability led to the rise of "software as a
service" or SaaS platforms. Software such as payment systems may not
necessarily be user facing for the core business, but might exist to facilitate
payment for other businesses. Data storage systems such as Amazon's S3 exist
not to serve the storage needs of end users, but as an online storage service
that enterprises and other applications can be designed with. Even Google
spreadsheets, which looks like it might still fit the old school definition of
an "application", provides APIs that let other applications use a google sheet
as a database of sorts, with their own applications such as Google forms
building on top of it.

In a way, this "new thing" is not really new -- as happens in the computing
world all the time. The much older unix "command line programs" may look like
they were designed in isolation, but they were actually designed to co-exist
well in an ecosystem of tools via the common protocol of "piping" and the
"standard I/O" streams that all applications were given by the OS. Any application
that produced and consumed text via its standard I/O could delegate certain
kinds of processing to the existence of other tools in the ecosystem. ``ls``
does not need to implement what ``sort`` does as long as its output fit what
``sort`` could handle.

Another such mechanism was "Apple Events" on MacOS ... which actually is very
closely related to REST which we'll discuss in this section. Mac applications
can publish "vocabularies" that might send/receive and process events by other
applications. Although these messages are called "events", they almost always
take the form of some representation of an internal thing owned by an
application that is shuttled to and fro other applications, which is the
essence of REST.

Today, applications can no longer be created in ignorance of the internet and
we have a flexible communication mechanism and protocol in TCP/IP with HTTP
layered on top of it, enabling applications to talk to each other in the common
language of "resources" and their "representations". 

.. note:: You might also hear the term "API" used to refer to these interfaces.
   "API" stands for "Application Programming Interface" and can be used broadly
   for any part of a software system that wishes to expose some functionality
   to another part via some set of mechanisms and contracts. We therefore 
   can talk of "C APIs", "C++ class APIs", "Windows APIs", "MacOS APIs",
   in addition to "PayPal API", "Google sheet API" etc. Hence it is a broader
   term, though heavily used today in the context of internet applications
   to refer to "REST APIs".

Preliminaries
-------------

Before we dive into an example API, it is important to understand the
consequences of some of the design choices of the internet that affect these
systems.

What we call the "internet" today is a set of physical wired and wireless
connections between machines, and a set of algorithms that help route
information from any machine to any other machine that can be reached either
directly or indirectly. This "internet core" does not distinguish between
"server" and "client". Every machine that is connected to some other machine is
assigned an "IP address" that's different from that machine so the two can
address each other and send messages.

.. admonition:: **Rabbit hole here**

    There is far more to be talked about regarding the structure and function
    of the internet as a network than we can afford here. So I'll leave it
    at that and we'll learn the bits of importance to us as we go along.

Since it is hard to remember lots of numbers so we can type them into the
browser, the internet architecture also includes "Domain Name Service"
(abbreviated DNS). These DNS servers help map names to specific IP addresses.
So when you type ``https://dropbox.com``, your browser reaches out to a known
DNS server (say ``8.8.8.8``) and asks it for the IP address corresponding to
the name ``dropbox.com``. It then makes a connection to that IP address
to make any further requests. Of course, the browser also performs some
optimizations to speed things up, like caching the IP address for a while
before making another request. The DNS server provides additional information
such as "how long to wait before asking for a lookup again".

The key bit here for us to understand is that once a service is up and published,
basically **anyone** on the internet can make requests to the service. OTOH
machines that are behind "subnets" of service providers only have private
IP addresses and therefore cannot run services reachable by the rest of the
internet, though they can reach out to other known services. 

Simple Image Storage Service
----------------------------

Let's consider an example of a service modeled on Amazon's "Simple Storage Service"
a.k.a. "S3" for short. AWS's S3 has so many facets to it today that one may not
consider it "simple" on first look, but it certainly *simplifies* storage
by eliminating concerns of scale. Our version though is only going to be
concerned with the rudiments of storing and retrieving image files we'll call
"SISS".

1. The service will be available at ``https://localhost:8080/siss/``.

2. When we ask the service to store an image, we'll get back a URL to the
   stored image of the form ``https://localhost:8080/siss/2298651378563``,
   where the number is some unique identifier for the image file.

3. We want to be able to retrieve a stored image in original form.

4. We want to be able to retrieve a "thumbnail" version of the image stored.

5. We want to be able to modify an already stored image.

6. We want to be able to retrieve some "metadata" about the image such as
   dimensions, file size and file type.

7. We want to be able to delete a stored image.

How do we now design an "API" for this using the HTTP protocol?

In the language of REST, we first identify the "resource(s)" being
managed by our service. In this case, there is only one kind of 
"resource" -- images.

Create a new stored image
-------------------------

Since we've identified the service end point for storing images as
``https://localhost:8080/siss/`` (which we'll abbreviate to ``/siss/``
to reduce redundancy in this discussion), we're now looking for
a way to enable a user to create a new resource under that
path that will look like ``/siss/23952987654`` which stands for
our image. 

Remember that we can **only** make HTTP requests to our service.

Such a creation of a new resource is done using a HTTP ``POST``
request. So our ``POST`` request will take the following
form on the wire --

.. code::

    POST /siss/ HTTP/1.1
    Content-Type: image/png
    Content-Length: 237856

    ...237856 image-bytes....

... and our server is then expected to respond with

.. code::

    HTTP/1.1 200 OK
    Content-Type: application/json
    Content-Length: 27

    {"url":"/siss/23875627835"}

The "200 OK" response indicates successful creation of the
resource and the response body gives us some information
about the resource --- in this case, we get the URL of the
resource with its unique ID.

Reading back a stored image
---------------------------

Once we have a few such images uploaded, we'll want to get
the images back --- for what use is a storage service if
we can't get back what we stored?

To retrieve the ``/siss/23875627835`` image we stored earlier,
we make the following ``GET`` request to the server --

.. code::

    GET /siss/23875627835 HTTP/1.1
    Accept: image/jpeg

.. note:: ``GET`` requests have no request body. Even if you supply any bytes
   in the body, servers will usually ignore it, or may even flag an error and
   disconnect you.

... and in response the server may send us this upon success --

.. code::

    HTTP/1.1 200 OK
    Content-Type: image/jpeg
    Content-Length: 987135

    ...987135 bytes of the image in JPEG format...

Pay attention to what we did here. When we created the resource (our image),
we gave the service a PNG file, but when we retrieved the file, we said we
wanted it in the JPEG format (using the ``Accept`` header). The server obliged
us by sending the image in the JPEG format!

This is an illustration of what we mean by a "representation of a resource
being transferred". We think of our "image" resource not as being identified by
the specific sequence of bytes we sent the service, but as a **content
concept**. The way the service represents this content may vary but we expect
some consistency with what the image's actual contents are about, in that if we
upload a cat image png and ask for a jpeg, we don't get a dog image as a
response. Once the resource is created on the service, we can merely exchange
**representations** of the resource and perhaps its state. The resource
itself often cannot be transferred over wire in many cases.

We may want to request a thumbnail version of the image we uploaded for
some purpose. We might then make a request of the form --

.. code::

    GET /siss/23875627835?size=thumbnail HTTP/1.1
    Accept: image/webp

Here we're requesting a "thumbnail" (leaving the exact details to the service),
in the "WebP" format, to which the service may respond with --

.. code::

    HTTP/1.1 200 OK
    Content-Type: image/webp
    Content-Length: 776

    ... 776 byte thumbnail image ...

The assumption here is that we can parse this image to find out its details.

The service may also make other properties of the image available to us.
For example, it may support an end point to get the pixel dimensions
of the original image using a request like this ---

.. code::

    GET /siss/23875627835/dims HTTP/1.1

And it may respond with --

.. code::

    HTTP/1.1 200 OK
    Content-Type: application/json
    Content-Length: 27

    {"width":1024,"height":768}

Note that the dimensions being a property of the image "resource", the "dims"
part of the URL is placed **within** the resource path. It is not hard
to think of an alternative end point for this like ``/dims/23875627835``
and it is not that that wouldn't get the job done. Suffixing the ``dims``
to the resource path is a better indicator of it being the property
of the resource and helps a programmer discover the API. To permit
such discovery, HTTP has a few other verbs like ``HEAD`` and ``OPTIONS``.
In this case, we could've sent an ``OPTIONS`` request to our end point
to get a list of possible ways to get representations of our image.

.. code::

    OPTIONS /siss/23875627835 HTTP/1.1

... with a possible server response being --

.. code::

    HTTP/1.1 200 OK
    Content-Type: application/json
    Content-Length: 176

    [
       { "type": "thumbnail",
         "url": "/siss/23875627835?size=thumbnail" },
       { "type": "rotated",
         "url": "/siss/23875627835?rotation=90" }
    ]

Updating an image
-----------------

So now we've stored an image we're working on for a client (say I'm a designer)
and we have the next iteration of the image and we want to update the 
service with this new version. Unlike when we created the image resource,
we're now trying to change a resource that already exists. We do this
with a HTTP ``PUT`` request like this --

.. code::

    PUT /siss/23875627835 HTTP/1.1
    Content-Type: image/jpeg
    Content-Length: 2387627

    ... 2387627 bytes of the joeg image ...

and if the service accepts this request and successfully processes it, it may
respond with the following (still keeping it "simple") --

.. code::

   HTTP/1.1 200 OK
   Content-Type: application/json
   Content-Length: 27

   {"url":"/siss/23875627835"}

Deleting an image
-----------------

If we no longer require the service to hold this resource for us,
we may instruct it to "delete" the resource. This is done using the
HTTP ``DELETE`` request like this --

.. code::

    DELETE /siss/23875627835 HTTP/1.1

In this case, we may not have any more headers to add (we'll come
to some issues with this later), but DELETE requests do not have any
body as well. The server may then respond with --

.. code::

    HTTP/1.1 200 OK
    Content-Type: application/json
    Content-Length: 35

    {"message":"Successfully deleted"}

Resource CRUD
-------------

The earlier four operations are often collectively referred to using the
acronym "CRUD" - short for "Create, Read, Update, Delete". When you're
designing a service "end point" that manages a resource, these are four primary
operations you need to think about at the minimum. You may decide that the
service forbids updating a resource, and that's ok, but you still need to think
about, document it and communicate it to your users.

What about other resource types?
--------------------------------

The "image file as a resource" illustrates the idea of transferring
representations over wire. While in this case it looks like it might
well be possible to transfer the resource itself in its entirety in
its original from, that is not possible in many cases.

For example, a Facebook "post" may be thought of as a resource that Facebook
manages on your behalf. A naive perspective may cause us to think "so what, I
can get my post in its entirety, right?", but the nature of the Facebook
service is that once your post is submitted, it becomes entangled with other
posts if some friends reposted it, and others have commented on your post or
its reposts, your post may have gained "likes" or other reactions, and so on.
It is no longer clear where "your post" starts and the where it ends. Of course,
you can fetch what you originally wrote, but understand that that is merely
one representation of the state of your post.

The REST approach is to treat all services as providing the CRUD of a collection
of resources that it maintains on your behalf and all interactions with
other applications be constructed in terms of such HTTP protocol requests 
that work by transferring various representations of your resource's state.

Hence "Representational State Transfer".

Caveats
-------

While this is the general principle, as you work with services provided by
other parties, you may encounter the principle not being met to the letter. You
shouldn't generally expect that to be the case in every little detail. By and
large, this approach will be used with some small deviations in a few corners.

Most mature services, though, will conform to this approach as it has many
architectural benefits though. Therefore knowing the "REST language" is helpful
to navigate the sea of services available to us today.


.. _S3 user guide: https://docs.aws.amazon.com/AmazonS3/latest/userguide//Welcome.html
.. _s3: https://docs.aws.amazon.com/AmazonS3/latest/API/Type_API_Reference.html
.. _How Amazon S3 works: https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html#CoreConcepts





