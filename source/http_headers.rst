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

List of Headers We Should Have:
    - also whatever I showed in the examples in "index.rst"

Permanent Headers
-----------------

These are given in the IANA's "`Permanent Message Header Field Names <https://www.iana.org/assignments/message-headers/message-headers.xhtml>`_"
list.

Accept
^^^^^^

`RFC 7231 S. 5.3.2 <http://tools.ietf.org/html/rfc7231#section-5.3.2>`_. At this point, Cantus
servers are only required to serve resources in JSON format. Refer to the `Content-Type`_ section
for more information.

In accordance with the WebDAV specification, the ``application/xml`` and ``text/xml`` foramts
(specified in `RFC 7303 <http://tools.ietf.org/html/rfc7303>`_) will be supported as defaults in a
future version of the Cantus API. Therefore, clients MUST provide the :http:header:`Accept`
header for forward compatibility.

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

Allow
^^^^^

`RFC 7231 S. 7.4.1 <http://tools.ietf.org/html/rfc7231#section-7.4.1>`_. When a client invokes the
``OPTIONS`` method on a resource, the "Allow" header in the response will indicate which other
methods may be invoked on that resource.

Example:

.. sourcecode:: http

    OPTIONS /(browse_chants) HTTP/1.1

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

In accordance with the WebDAV specification, the ``application/xml`` and ``text/xml`` foramts
(specified in `RFC 7303 <http://tools.ietf.org/html/rfc7303>`_) will be supported as defaults in a
future version of the Cantus API. Therefore, clients MUST provide the :http:header:`Accept`
header for forward compatibility.

.. Implementation note: Tornado handles the "Content-Type" header automatically.

.. _`cantus headers`:

Cantus-Specific Extension Headers
---------------------------------

These headers extend the HTTP and WebDAV standards in ways specific to the Cantus API. We create
extension headers only when no sensible alternative is sensible.

X-Cantus-Version
^^^^^^^^^^^^^^^^

Indicates the Cantus API version implemented by a client or server. For example,
``X-Cantus-Version: Cantus/0.0.1`` is version 0.0.1, and ``X-Cantus-Version: Cantus/3.2.6-test`` is
a version called "3.2.6-test." Also refer to :ref:`version numbers`.

X-Cantus-Include-Resources
^^^^^^^^^^^^^^^^^^^^^^^^^^

Clients MAY include this header in a requested, telling a server whether to include a "resources"
member with hyperlinks to related resources. This can be "true" or "false" (but is case-insensitive).
Servers MUST use this header to indicate whether "resources" members are included in a response.

X-Cantus-Fields
^^^^^^^^^^^^^^^

Clients MAY use this header to request only certain fields. Servers MUST include this header, which
lists the fields that are present in *all* returned resources. Fields that are only present in some
of the returned resources belong in the :http:header:`X-Cantus-Extra-Fields` header. Refer also to
the :ref:`cantus header example`.

X-Cantus-Extra-Fields
^^^^^^^^^^^^^^^^^^^^^

A semicolon-separated list of resource fields. This has no meaning in a request, but a server MUST
use this to indicate which fields were available for some, but not all, resources. Refer also to the
:ref:`cantus header example`.

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

If :http:header:`X-Cantus-Per-Page` is non-zero, servers MUST and clients MAY include this header to
indicate that the results should correspond to a particular sub-set of the full query. If a client
provides a value for this header greater than :http:header:`X-Cantus-Total-Results` divided by
:http:header:`X-Cantus-Per-Page` (i.e., greater than the total number of pages) the server MUST
respond with `409 Conflict <https://tools.ietf.org/html/rfc7231#section-6.5.8>`_.

X-Cantus-Sort
^^^^^^^^^^^^^

If the :http:header:`X-Cantus-Sort` is present in a request, it will contain a field name and
direction indicator (``asc`` or ``desc``) separated by a semicolon. The field name may also be
"relevance," the default value, which ranks search results by a server-determined relevance score.
If the indicated field is not present in all sources, the server MAY choose another field by which
to sort. For every search query, the server MUST include an :http:header:`X-Cantus-Sort` response
header indicating the actual field and sort direction of the response.

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
    X-Cantus-Fields: id;incipit
    X-Cantus-Extra-Fields: cantus_id
    X-Cantus-Total-Results: 10
    X-Cantus-Per-Page: 3
    X-Cantus-Page: 2

    {"results": [
        {"chant": {
             "id": "149243",
             "inicipit": "Estote parati similes",
             "cantus_id": "002685"
             }},
        {"chant": {
            "id": "149244",
            "incipit": "Salvator mundi domine qui nos",
            }},
        {"chant": {
            "id": "149245",
            "incipit": "Estote parati similes",
            "cantus_id": "002685",
            }}
        ]
    }















































