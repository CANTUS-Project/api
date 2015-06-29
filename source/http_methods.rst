HTTP Methods
============

- :ref:`OPTIONS http method` returns the methods and header fields and values a resource supports.
- :ref:`GET http method` retrieves data about a resource, formatted according to options indicated
  in the headers.
- :ref:`HEAD http method` returns only the header part of a GET request.
- :ref:`SEARCH http method` search for specific resources.

.. _`options http method`:

OPTIONS
-------

The `OPTIONS <https://tools.ietf.org/html/rfc7231#section-4.3.7>`_ method returns information about
which HTTP methods and Cantus extension headers are available for a specified request-target (URL).
Although it is not a requirement of the HTTP standard, the reality of Cantus servers is that all
resources of a given type and "action" (refer to :ref:`url explanation`) will produce the same
response to an OPTIONS request. Differences between different resource types with the same "action"
will be minimal.

The HTTP methods allowed for a resource are indicated in the :ref:`cantus header allow` header, as
required in RFC 7231.

The Cantus request headers (defined in :ref:`cantus headers`) applicable to a resource MUST be
returned in the response to an OPTIONS request, either used for their intended purpose (like
:http:header:`X-Cantus-Version`) or with the value ``allow`` (case insensitive).

.. _`get http method`:

GET
---

The `GET <https://tools.ietf.org/html/rfc7231#section-4.3.1>`_ method returns one or more resources
in the response body, depending on the URL, request headers, and request body. For the Cantus API,
request bodies are ignored.

.. _`head http method`:

HEAD
----

The `HEAD <https://tools.ietf.org/html/rfc7231#section-4.3.2>`_ method is effectively a GET request
without a response body. In effect, user agents can use a HEAD request like a resource-specific
OPTIONS request, in that the response headers contain information about the fields and resources
that would be returned in the response body of the corresponding GET request.

.. _`search http method`:

SEARCH
------

The `SEARCH <http://tools.ietf.org/html/rfc5323>`_ method is like a GET request with search query
parameters held in the request body. For now, the Cantus API's use of the SEARCH method is not
compliant with the RFC 5323 definition, and using the method according to the standard specified
in RFC 5323 results in unknown operation. In the future, the Cantus API may use a standards-
compliant implementation of the SEARCH method.

Refer to :ref:`searching` for information on how to prepare a search query.
