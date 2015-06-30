HTTP Headers
============

I'm not sure yet of the exact form this will take, but we'll use HTTP headers to tell clients
about the field names available for a resource type, and the fields in a particular record that
have valid values. Clients can also request only certain fields, in case they won't use some of the
data, since there's no point in compressing, shipping, and decompressing it if we know ahead of
time that it's going to be ignored.

We can probably include more information here too.

The HTTP/1.1bis headers are specified in `RFC 7230, S. 3.2 <https://tools.ietf.org/html/rfc7230#section-3.2>`_.
The IANA also maintains a
`list of standard headers <https://www.iana.org/assignments/message-headers/message-headers.xhtml>`_.

Permanent Headers
-----------------

These are given in the IANA's "`Permanent Message Header Field Names <https://www.iana.org/assignments/message-headers/message-headers.xhtml>`_"
list.

Accept
^^^^^^

Although Cantus servers are only required to supply response bodies in ``application/json`` format,
for completeness all Cantus user agents MUST provide a :http:header:`Accept` header with that value.

While, in accordance with RFC 7231, user agents are not required to include this header, the API
may provide additional response formats in the future (in particular ``application/xml`` and
``text/xml``) and may even change the default format, so user agents are strongly recommend to
specify :http:header:`Accept` to ensure forward compatibility.

Accept-Charset
^^^^^^^^^^^^^^

Cantus user agents MUST use UTF-8; for completeness this must be indicated in the
:http:header:`Accept-Charset` header as ``utf-8``.

Accept-Encoding
^^^^^^^^^^^^^^^

Cantus user agents and servers MAY use data compression whenever possible, so the recommended value
for the :http:header:`Accept-Encoding` header is ``gzip``.

.. _`cantus header allow`:

Allow
^^^^^

When a client invokes the :http:method:`OPTIONS` method on a resource, the :http:header:`Allow`
header in the response indicates which other methods may be invoked on that resource.

Example:

.. sourcecode:: http

    OPTIONS /(browse.chant) HTTP/1.1

.. sourcecode:: http

    HTTP/1.1 200 OK

    Allow: GET, HEAD, OPTIONS

Content-Encoding
^^^^^^^^^^^^^^^^

The :http:header:`Content-Encoding` header tells a user agent the encoding of the response (i.e.,
whether its requested data compression algorithm was available and used).

Content-Length
^^^^^^^^^^^^^^

The :http:header:`Content-Length` is a decimal number indicating the number of octets expected in
the response body. This is used by clients (probably automatically by an HTTP library) to determine
when all data has been received.

.. Implmementation note: Tornado handles this automatically.

Content-Type
^^^^^^^^^^^^

Cantus server MUST provide a :http:header:`Content-Type` header with every response, including the
"charset" parameter. Response bodies are formatted in `JSON <http://tools.ietf.org/html/rfc7158>`_,
with the UTF-8 character set, so the value of this header will almost invariably be
``application/json; charset=utf-8``.

Future versions of the API may permit or require different response formats, in particular
``application/xml`` and ``text/xml``, and may also change the default response format. Therefore,
user agents are strongly recommended to provide the :http:header:`Accept` header for forward
compatibility.

.. Implementation note: Tornado handles the "Content-Type" header automatically.

Cross-Origin Resource Sharing (CORS) Headers
--------------------------------------------

The following headers are used for `Cross-Origin Resource Sharing <http://www.w3.org/TR/cors/>`_
requests. In practice for Cantus, server-to-server requests will not be CORS requests, but requests
sent with a Web browser as the user agent are likely to be CORS requests. In everyday use, most
browser-sent requests are not CORS requests. Because we expect the Cantus server and user-facing
Web app to be served on different hosts (or at least on different ports) it is likely that a
user-facing Web app will be submitting CORS requests.

The user agent's implementation will be handled automatically by the user agent (being the browser).
Thus CORS is primarily a concern for the server implementation. Do note that, while Cantus server
implementations MAY support the following CORS headers, it is not strictly necessary to have a
useful server implementation.

When implementing CORS support, we strongly recommend server authors to follow the specification's
`Resource Processing Model <http://www.w3.org/TR/cors/#resource-processing-model>`_, which is both
quite precise and relatively easy to understand and follow.

In the context of this section, a **pre-flight request** is an OPTIONS request submitted before the
actual request, with the intent of determining whether the actual reqeust will succeed according to
the provided CORS header values.

Note there is an :ref:`example <CORS header example>` at the end of this section.

Access-Control-Allow-Origin
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The :http:header:`Access-Control-Allow-Origin` response header contains a single hostname, the value
of the :ref:`CORS origin header` request header, *if* that host is permitted to use the Cantus server.

This header is required in both the pre-flight and actual request.

Access-Control-Expose-Headers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The :http:header:`Access-Control-Expose-Headers` response header holds a list of the header names
that Cantus user agents will want to read. In practice, these are the Cantus-specific extension
headers. This will be used in both the

This header is required in both the pre-flight and actual request.

Access-Control-Max-Age
^^^^^^^^^^^^^^^^^^^^^^

To reduce server traffic, the Cantus server MAY include the :http:header:`Access-Control-Max-Age`
header in the response to a pre-flight request indicating the number of seconds for which the CORS
headers will be valid.

This header may only appear in the pre-flight request.

Access-Control-Allow-Methods
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The :http:header:`Access-Control-Allow-Methods` response header is a response to the
:ref:`CORS request method` request header. This response header will contain the value of the request
header if it is permitted for this resource and it is not a `simple method <http://www.w3.org/TR/cors/#simple-method>`_
(i.e., GET, HEAD, or POST).

This header may only appear in the pre-flight request.

Access-Control-Allow-Headers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The :http:header:`Access-Control-Allow-Headers` response header is a response to the
:ref:`CORS request headers` request header. The response header will hold all the values of the
request header that are allowed as header names for a CORS request.

This header may only appear in the pre-flight request.

.. _`CORS request method`:

Access-Control-Request-Method
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The user agent submits the :http:header:`Access-Control-Request-Method` header with the HTTP method
that will be used in the actual request.

This header may only appear in the pre-flight request.

.. _`CORS request headers`:

Access-Control-Request-Headers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The user agent submited the :http:header:`Access-Control-Request-Headers` header with the header
names that will be used in the actual request.

This header may only appear in the pre-flight request.

.. _`CORS origin header`:

Origin
^^^^^^

The user agent submited the :http:header:`Origin` header with the hostname (and access scheme and,
if it is not 80, the port) from which the user agent's Web page came. If this header is part of a
request, the server SHOULD add a ``Vary: Origin`` header, since a CORS request may receive a
different response than an otherwise-identical non-CORS request. Also note that, if the
:http:header:`Origin` header is not present in a request, any other CORS request headers MUST be
ignored.

This header is required in both the pre-flight request and the actual request.

.. note:: We recommend that server implementations check the value of the ``Origin`` header against
    a list of known Web app deployments, to prevent requests coming from unknown Web apps.

.. _`CORS headers example`:

Example
^^^^^^^

In this example, we assume that Abbott (the Cantus API server) is operating at ``https://abbott.cantusproject.org:8888/``
and that a user enters ``https://app.cantusproject.org/`` in their browser to access the GUI Web app.
From the perspective of the Cantus GUI Web app's author, one single request has been submitted---the
"actual" request---meaning the browser automatically creates the preflight request.

Preflight request:

.. sourcecode:: http

    OPTIONS https://abbott.cantusproject.org:8888/(browse.chants)/ HTTP/1.1
    Access-Control-Request-Method: SEARCH
    Access-Control-Request-Headers: X-Cantus-Search-Help, X-Cantus-Page, X-Cantus-Garbage-Header
    Origin: https://app.cantusproject.org/
    ...

Preflight response:

.. sourcecode:: http

    HTTP/1.1 200 OK
    Allow: GET, HEAD, OPTIONS, SEARCH
    Access-Control-Allow-Origin: https://app.cantusproject.org/
    Access-Control-Allow-Headers: X-Cantus-Search-Help, X-Cantus-Page
    Access-Control-Allow-Method: SEARCH
    Access-Control-Max-Age: 86400
    ...

Actual request:

.. sourcecode:: http

    SEARCH https://abbott.cantusproject.org:8888/(browse.chants)/ HTTP/1.1
    X-Cantus-Search-Help: true
    X-Cantus-Page: 4
    Origin: https://app.cantusproject.org/
    ...

    {"query": "incipit:deus"}

Actual response:

.. sourcecode:: http

    HTTP/1.1 200 OK
    Access-Control-Allow-Origin: https://app.cantusproject.org/
    Access-Control-Max-Age: 86400
    Access-Control-Expose-Headers: X-Cantus-Search-Help, X-Cantus-Page, X-Cantus-Per-Page, { others }
    X-Cantus-Search-Help: true
    X-Cantus-Page: 4
    X-Cantus-Per-Page: 10
    ...

    { /* chant data here */ }

Note that only a subset of the headers are shown, to emphasize the CORS behaviour.

.. _`cantus headers`:

Cantus-Specific Extension Headers
---------------------------------

These headers extend the HTTP standard in ways specific to the Cantus API. We create extension
headers only when no existing solution is sensible.

The server SHOULD return a `400 Bad Request <https://tools.ietf.org/html/rfc7231#section-6.5.1>`_
response code for any Cantus-specific request headers that contain invalid values. However, for a
request header that does not apply to the requested resource (like :http:header:`X-Cantus-Page` for
a "view" URL that will only return a single resource) the server MUST ignore the extraneous headers
regardless of whether their value is valid.

X-Cantus-Version
^^^^^^^^^^^^^^^^

Indicates the Cantus API version implemented by a client or server. For example,
``X-Cantus-Version: Cantus/0.0.1`` is version 0.0.1, and ``X-Cantus-Version: Cantus/3.2.6-test`` is
a version called "3.2.6-test." Also refer to :ref:`version numbers`.

X-Cantus-Include-Resources
^^^^^^^^^^^^^^^^^^^^^^^^^^

Clients MAY include this header in a request, telling a server whether to include a "resources"
member with hyperlinks to related resources. This can be "true" or "false" (but is case-insensitive).
Servers MUST use this header to indicate whether "resources" members are included in a response.

X-Cantus-Fields
^^^^^^^^^^^^^^^

Clients MAY use this header to request only certain fields in the response. Servers MUST include
this header, which lists the fields that are present in *all* returned resources. Fields that are
only present in some of the returned resources belong in the :http:header:`X-Cantus-Extra-Fields`
header. For both headers, if there are no fields, the server MAY omit the header or return an empty
header.

Both headers are a comma-separated list, like ``id, name, description``.

Note that the "id" and "type" fields MUST be returned for every resource, since this is the minimum
information required to find the resource again. If the :http:header:`X-Cantus-Extra-Fields` request
header does not contain the "id" and "type" fields, the server SHOULD include them anyway, and
SHOULD NOT consider this an invalid value.

Refer also to the :ref:`cantus header example`.

X-Cantus-Extra-Fields
^^^^^^^^^^^^^^^^^^^^^

If some, but not all, resources contain a field, the server MUST include that field name in this
header. This field has no meaning in a request. Refer also to the :ref:`cantus header example`.

X-Cantus-No-Xref
^^^^^^^^^^^^^^^^

Boolean header to instruct the server not to do cross-reference lookup in complex resources. Cross-
referenced fields are what define complex resources (:ref:`complex resource types`) and they may be
stored in the server's database in various ways. Filling all the cross-referenced fields may take
significant additional time, and may not be desirable in all use cases.

Setting this request header to ``'true'`` (case insensitive) means the server MUST NOT process
fields that would be provided by cross-references. The server MUST also include the "id" value of
any resources that would be used to create cross-references, formed by appending ``'_id'`` to the
resource type: a Feast associated with a Chant would therefore appear as, for example,
``{'feast_id': '62'}``.

Example response body with :http:header:`X-Cantus-No-Xref` set to ``false`` (or not set):

.. sourcecode:: http

    {"149243": {
        "id": "149243",
        "type": "chant",
        "inicipit": "Estote parati similes",
        "feast": "Nativitas Domini",
        "feast_desc": "Christmas Day"
        }
    }

Example response body with :http:header:`X-Cantus-No-Xref` set to ``true``.

.. sourcecode:: http

    {"149243": {
        "id": "149243",
        "type": "chant",
        "inicipit": "Estote parati similes",
        "feast_id": "2745"
        }
    }

X-Cantus-Total-Results
^^^^^^^^^^^^^^^^^^^^^^

The total number of results that match a search query. The server MUST include this header in the
response to every :ref:`search http method` request, and MAY also provide it in the
response to a "browse" or "view" request.

X-Cantus-Per-Page
^^^^^^^^^^^^^^^^^

Clients MAY use this header to negotiate "paginated" results with the server, where queries matching
a large number of resources will return information about only a portion of those resources. The
value should always be a positive integer or zero. A zero symbolizes a request for non-paginated
results---information for all matching resources. Servers MUST include this header if the
:http:header:`X-Cantus-Total-Results` is present and greater than ``0`` (i.e., for every search
query that yields results).

If the server determines that the number of requested resources is too high, it MUST return a status
code of `507 Insufficient Storage <https://tools.ietf.org/html/rfc4918#section-11.5>`_.
The limit is determined by the server, and may change arbitrarily. However, the 507 response MUST
include an :http:header:`X-Cantus-Per-Page` header with a suggested value that the server determines
it is likely to be capable of handling.

X-Cantus-Page
^^^^^^^^^^^^^

If the :http:header:`X-Cantus-Per-Page` request header is non-zero, clients MAY include this header
to indicate that the results should correspond to a particular sub-set of the full query. If a
client provides a value for this header greater than the :http:header:`X-Cantus-Total-Results`
response header divided by the :http:header:`X-Cantus-Per-Page` request header (i.e., greater than
the total number of pages) the server MUST respond with
`409 Conflict <https://tools.ietf.org/html/rfc7231#section-6.5.8>`_.

If a query is successful, servers MUST include this header in responses to indicate the effective
page of the results.

Note that the first page is numbered ``1``, not ``0``.

X-Cantus-Sort
^^^^^^^^^^^^^

If the :http:header:`X-Cantus-Sort` is present in a request, it SHOULD contain a list of 2-tuples of
field names and direction indicators (``asc`` or ``desc``) separated by a comma, each separated by a
semicolon. (For example: ``incipit,asc`` or ``incipit,asc;feast,desc``). "Ascending" results put the
numerical and textual results in canonical order (i.e., 1, 2, 3; and A, B, C). "Descending" is the
opposite. Only the following characters are permitted: upper- and lower-case letters, ``_``, ``,``,
``;``, and spaces.

If a request does not have a :http:header:`X-Cantus-Sort` header, the server MUST order results
according to an appropriate relevance score, with the most relevant results returned first.

If a field is not present in all (or any) search results, the server MAY choose a different field by
which to sort, return a `409 Conflict <https://tools.ietf.org/html/rfc7231#section-6.5.8>`_
response, or simply use the partially missing field regardless.

For every request including a :http:header:`X-Cantus-Sort` header, the server MUST include an
equivalent response header to indicate the actual sort field and direction used.

.. note:: For search queries, clients are recommended to trust the default relevance-based sort
    order. Cantus servers should be optimized to provide the most relevant results by default. This
    header makes the most sense when a user wants to browse all of a category.

.. _`X-Cantus-Search-Help`:

X-Cantus-Search-Help
^^^^^^^^^^^^^^^^^^^^

If the client indicates ``true`` in the :http:header:`X-Cantus-Search-Help` header, the server MAY
modify a search request to be more lenient if the original search request produces no results. In
this case, the server MUST return the actual query in the :http:header:`X-Cantus-Search-Help`
response header.

.. _`cantus header example`:

Example of Cantus Headers
^^^^^^^^^^^^^^^^^^^^^^^^^

A response.

.. sourcecode:: http

    HTTP/1.1 200 OK
    Content-Type: application/json; charset="utf-8"
    Content-Length: xxx
    X-Cantus-Version: 1.0.0
    X-Cantus-Include-Resources: false
    X-Cantus-Fields: id,incipit
    X-Cantus-Extra-Fields: cantus_id
    X-Cantus-Total-Results: 10
    X-Cantus-Per-Page: 3
    X-Cantus-Page: 2
    X-Cantus-Include-Resources: false

    {"149243": {
        "id": "149243",
        "type": "chant",
        "inicipit": "Estote parati similes",
        "cantus_id": "002685"
        },
     "149244": {
         "id": "149244",
         "type": "chant",
         "incipit": "Salvator mundi domine qui nos",
        },
     "149245": {
         "id": "149245",
         "type": "chant",
         "incipit": "Estote parati similes",
         "cantus_id": "002685",
        }
    }
