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
Note that the WebDAV-specific headers are given in that list under the "http" protocol.

List of Headers We Should Have:
    - one to say whether to include "resources" links in the JSON response
    - one to say which fields to include, or which to exclude, from the JSON response
    - also whatever I showed in the examples in "index.rst"

Permanent Headers
-----------------

These are given in the IANA's "`Permanent Message Header Field Names <https://www.iana.org/assignments/message-headers/message-headers.xhtml>`_"
list.

Accept
^^^^^^

.. important:: CANTUS clients that wish to receive data in JSON format must always request it. The
    default response body format is XML, for standards compliance. Refer to :ref:`json and xml` for
    more information.

`RFC 7231 S. 5.3.2 <http://tools.ietf.org/html/rfc7231#section-5.3.2>`_. CANTUS servers will be
capable of serving all resources either in XML and JSON formats. Refer to the `Content-Type`_
section for more information.

Accept-Charset
^^^^^^^^^^^^^^

`RFC 7231 S. 5.3.3 <http://tools.ietf.org/html/rfc7231#section-5.3.3>`_. CANTUS clients should
always use Unicode and UTF-8 whenever possible, so the recommended value for this header is ``utf-8``.

Accept-Encoding
^^^^^^^^^^^^^^^

`RFC 7231 S. 5.3.4 <http://tools.ietf.org/html/rfc7231#section-5.3.4>`_. CANTUS clients and servers
should always use data compression whenever possible, so the recommended value for this header is
``gzip``.

TODO: find out if that's possible without too much complication

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

`RFC 7231 S. 3.1.1.5 <http://tools.ietf.org/html/rfc7231#section-3.1.1.5>`_. CANTUS servers provide
the Content-Type header with every response, including the "charset" parameter. While "charset" will
almost invariably be ``utf-8``, it may be possible to provide other character sets in the future.

The media-type will be one of:
    - ``application/json`` for a message body encoded in JSON, as specified in
        `RFC 7158 <http://tools.ietf.org/html/rfc7158>`_.
    - ``application/xml`` for a message body encoded in XML, as specified in
        `RFC 7303 <http://tools.ietf.org/html/rfc7303>`_.
    - ``text/xml`` as ``application/xml``

In accordance with the WebDAV specification, the XML formats shall be the default format provided
to and by the server (i.e., when the "Accept" header is not specified). Refer to
:ref:`json and xml` for more information.

.. Implementation note: Tornado handles this automatically.

DAV
^^^

`RFC 4918 S. 10.1 <http://tools.ietf.org/html/rfc4918#section-10.1>`_. The DAV header is only
returned to resources that support DAV extensions (i.e., chants and sources, plus the respective
parent paths, e.g., ``/chants/`` and ``/sources/``). By CANTUS API 2.0.0, once resources are
editable, the compliance class shall be "1," but until then it will be a URL to this API document.

Server
^^^^^^

`RFC 7231 S. 7.4.2 <http://tools.ietf.org/html/rfc7231#section-7.4.2>`_. Indicates the CANTUS API
version implemented by a server. For example, ``Server: CANTUS/0.0.1`` is version 0.0.1, and
``Server: CANTUS/3.2.6-test`` is a version called "3.2.6-test." Also refer to :ref:`version numbers`.

User-Agent
^^^^^^^^^^

`RFC 7231 S. 5.5.3 <http://tools.ietf.org/html/rfc7231#section-5.5.3>`_. Indicates the CANTUS API
version implemented by a client. For example, ``Server: CANTUS/0.0.1`` is version 0.0.1, and
``Server: CANTUS/3.2.6-test`` is a version called "3.2.6-test." Also refer to :ref:`version numbers`.

Vary
^^^^

`RFC 7231 S. 7.1.4 <http://tools.ietf.org/html/rfc7231#section-7.1.4>`_. The server sends this
header to indicate that a header field was used in determining the response format, and that a
subsequent request with a different value for that header may result in a different response format.

Because all response bodies may be either XML or JSON, the "Vary" header will be included with any
response that has a response body, with ``accept`` in the value::

    Vary: accept

Other headers may be included too, however only when a header changes the *representation* of data
returned, not the data itself. For example, a header specifying how many search results to return
does not change their representation, and will therefore not be included in the "Vary" header.

CANTUS-Specific Extension Headers
---------------------------------

These headers extend the HTTP and WebDAV standards in ways specific to the CANTUS API. We create
extension headers only when no sensible alternative is sensible.
