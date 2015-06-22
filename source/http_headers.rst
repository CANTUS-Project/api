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

`RFC 7231 S. 5.3.2 <http://tools.ietf.org/html/rfc7231#section-5.3.2>`_. At this point, Cantus
servers are only required to serve resources in JSON format. Refer to the `Content-Type`_ section
for more information.

While, in accordance with RFC 7231, user agents are not required to include a :http:header:`Accept`
header, the API may provide additional response formats in the future (in particular
``application/xml`` and ``text/xml``) and may even change the default format, so user agents are
strongly recommend to specify :http:header:`Accept` to ensure forward compatibility.

Accept-Charset
^^^^^^^^^^^^^^

`RFC 7231 S. 5.3.3 <http://tools.ietf.org/html/rfc7231#section-5.3.3>`_. Cantus clients should
always use Unicode and UTF-8 whenever possible, so the recommended value for this header is ``utf-8``.

Accept-Encoding
^^^^^^^^^^^^^^^

`RFC 7231 S. 5.3.4 <http://tools.ietf.org/html/rfc7231#section-5.3.4>`_. Cantus clients and servers
should always use data compression whenever possible, so the recommended value for this header is
``gzip``.

TODO: find out if that's possible without too much complication

.. _`cantus header allow`:

Allow
^^^^^

`RFC 7231 S. 7.4.1 <http://tools.ietf.org/html/rfc7231#section-7.4.1>`_. When a client invokes the
``OPTIONS`` method on a resource, the "Allow" header in the response will indicate which other
methods may be invoked on that resource.

Example:

.. sourcecode:: http

    OPTIONS /(browse.chant) HTTP/1.1

.. sourcecode:: http

    HTTP/1.1 200 OK

    Allow: GET, HEAD, OPTIONS

Content-Encoding
^^^^^^^^^^^^^^^^

`RFC 7231 S. 3.1.2.2 <http://tools.ietf.org/html/rfc7231#section-3.1.2.2>`_. Tells the client
the encoding of the response (i.e., whether its requested data compression algorithm was available
and used).

Content-Length
^^^^^^^^^^^^^^

`RFC 7230 S. 3.3.2 <http://tools.ietf.org/html/rfc7230#section-3.3.2>`_. A decimal number indicating
the number of octets expected in the response body. This is used by clients (probably automatically
by an HTTP library) to determine when all data has been received.

.. Implmementation note: Tornado handles this automatically.

Content-Type
^^^^^^^^^^^^

`RFC 7231 S. 3.1.1.5 <http://tools.ietf.org/html/rfc7231#section-3.1.1.5>`_. Cantus servers provide
the Content-Type header with every response, including the "charset" parameter. While "charset" will
almost invariably be ``utf-8``, it may be possible to provide other character sets in the future.

The media-type will be ``application/json``, indicating a message body encoded in JSON, as specified
in `RFC 7158 <http://tools.ietf.org/html/rfc7158>`_.

Future versions of the API may permit or require different response formats, in particular
``application/xml`` and ``text/xml``, and may also change the default response format. Therefore,
user agents are strongly recommended to provide the :http:header:`Accept` header for forward
compatibility.

.. Implementation note: Tornado handles the "Content-Type" header automatically.

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

The total number of results that match a search query. The server MUST include this header with the
results of every search query.

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
modify a search request to be more lenient if the original search request produced no results. In
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
