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
in the response body, depending on the URL, request headers, and request body. The meaning of the
request body is different depending on the "action" implied by the URL: for view and browse URLs,
the request body is ignored; for search URLs, the request body contains the search query.

Many people and Web frameworks appear to believe that GET requests may not have a request body, but
RFC 7231 clearly specifies that request bodies are permitted but do not have a defined meaning.
With the previous paragraph, their meaning becomes defined for Cantus servers and user agents.

.. _`head http method`:

HEAD
----

In theory, the `HEAD <https://tools.ietf.org/html/rfc7231#section-4.3.2>`_ method is a GET request
that returns only response headers without a response body. In effect, user agents can use a HEAD
request like a resource-specific or search-query-specific OPTIONS request, in that the response
headers contain information about the fields and resources that would be returned in the response
body (being the resources themselves).

The API author guesses there will be two primary purposes for HEAD requests. First, to determine
whether a resource has changed (by using the :http:header:`ETag` header, for instance). Second, to
determine the characteristics of a search result without downloading and inspecting all results.

.. _`search http method`:

SEARCH
------

The SEARCH method, defined in `RFC 5323 <http://tools.ietf.org/html/rfc5323>`_, may be supported in
future versions of the Cantus API. For now, please use the search URLs described in :ref:`searching`.
