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

The Cantus headers (defined in :ref:`cantus headers`) applicable to a resource MUST be returned in
the response to an OPTIONS request, either used for their intended purpose (like
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
whether a resource has changed (by using the :http:header:`Etag` header, for instance). Second, to
determine the characteristics of a search result without downloading and inspecting all results.

.. _`search http method`:

SEARCH
------

The SEARCH method, defined in `RFC 5323 <http://tools.ietf.org/html/rfc5323>`_, may be supported in
future versions of the Cantus API. For now, please use the search URLs described in :ref:`searching`.

..
    ************** NOTE ************* all this bit is commented and won't appear in output
    Unlike most of the other HTTP methods, ``SEARCH`` can be quite complex. Searching, of course, is a
    complex task! The ``SEARCH`` method is described in `RFC 5323 <http://tools.ietf.org/html/rfc5323>`_.

    As described in `Section 1.6 <http://tools.ietf.org/html/rfc5323#section-1.6>`_, the four steps a
    client follows to conduct a query are as follows:

    #. The client constructs a query in the chosen grammar.
    #. The client submits a ``SEARCH`` request on the relevant resource, submitting the query as
    ``text/xml`` or ``application/xml`` data.
    #. The server performs the query.
    #. The server responds with the query results in the WebDAV multistatus format defined in
    `RFC 4918 S. 13 <http://tools.ietf.org/html/rfc4918#section-13>`_.

    Knowing Whether to Use SEARCH
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

    If a client invokes the OPTIONS method on a resource, the server will include "SEARCH" in the
    response's "Allow" header.

    .. sourcecode:: http

        OPTIONS /(browse_chants) HTTP/1.1

    .. sourcecode:: http

        HTTP/1.1 200 OK

        Allow: GET, HEAD, OPTIONS, SEARCH
        DASL: <dav:basicsearch> <dav:cantus>

    .. _`dasl header full description`:

    The DASL Header
    ^^^^^^^^^^^^^^^

    As defined in `S. 3.2 <http://tools.ietf.org/html/rfc5323#section-3.2>`_, the DASL header in
    response to an OPTIONS request specifies which query grammars are supported in the search scope.
    Not all resources may respond with a DASL header, and importantly the DASL header may be omitted if,
    for example, the server's search server is temporarily unavailable.

    Note that multiple search grammars may be supported at once, and they may be included either as
    multiple ``DASL`` headers, a comma-separated list in a single header, or a mix of both.

    URIs corresponding to a search grammar will be enclosed in < > brackets.

    Query Schema Discovery
    ^^^^^^^^^^^^^^^^^^^^^^

    At this point, Cantus servers will not implemented QSD as defined in `S. 3.2 <http://tools.ietf.org/html/rfc5323#section-3.2>`_.

    Query Grammar: DAV:basicsearch
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

    As defined in `S. 5.2 <http://tools.ietf.org/html/rfc5323#section-5.2>`_. The DAV:basicsearch
    grammar is relatively simple, and reminiscent of SQL. Because this schema is intended for generic
    DAV situations, it is more complicated than the Cantus-specific query grammar.

    .. note:: Since this query grammar is more complicated to implement, it will not be available in
        early testing versions of the Cantus reference server (Abbott). It is required for WebDAV
        compliance, so it will become available before the release of Abbott v1.0.

    This example searches for all Chant resources.

    ??????????????

    Response Body Format
    ^^^^^^^^^^^^^^^^^^^^

    .. _`search result truncation`:

    If the value of "rows" is too high or omitted, and the number of results is higher than an arbitrary,
    server-determined value, the server may choose to truncate results using the "507 Insufficient
    Storage" response code. For example:

    .. sourcecode:: http

        SEARCH /(browse_chants)/ HTTP/1.1
        Host: abbott.cantusdatabase.org
        Content-Type: text/xml; charset="utf-8"
        Content-Length: xxx

        <?xml version="1.0" encoding="utf-8"?>
        <DAV:searchrequest>
            <cantus:query>incipit:Deus</cantus:query>
        </DAV:searchrequest>

    .. sourcecode:: http

        HTTP/1.1 207 Multistatus
        Content-Type: text/xml; charset="utf-8"
        Content-Length: xxx

        <?xml version="1.0" encoding="utf-8"?>
        <DAV:multistatus>
            <DAV:response>
                <DAV:href>/(browse_chants)/945/</DAV:href>
                <DAV:status>HTTP/1.1 200 OK</DAV:status>
                <cantus:chant>
                    <cantus:id>945</cantus:id>
                    <cantus:incipit>Deus a Libano veniet</cantus:incipit>
                </cantus:chant>
            </DAV:response>
            <DAV:response>
                <DAV:href>/(browse_chants)/372579/</DAV:href>
                <DAV:status>HTTP/1.1 200 OK</DAV:status>
                <cantus:chant>
                    <cantus:id>372579</cantus:id>
                    <cantus:incipit>Deus auribus nostris</cantus:incipit>
                </cantus:chant>
            </DAV:response>
            <DAV:response>
                <DAV:href>/(browse_chants)/</DAV:href>
                <DAV:status>HTTP/1.1 507 Insufficient Storage</DAV:status>
                <DAV:responsedescription xml:lang="en">
                Only first two matching records were returned
                </DAV:responsedescription>
            </DAV:response>
        </DAV:multistatus>
