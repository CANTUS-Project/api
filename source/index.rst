.. CANTUS API documentation master file, created by
   sphinx-quickstart on Mon Feb 16 23:04:23 2015.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

API for HTTP Access to the CANTUS Database
==========================================

This is the Application Programming Interface for CANTUS database-access projects. The 1.x versions
will only allow searching and reading the database, though (pending funding) 2.x versions will add
the ability to create and edit resources.

To navigate the specification, use the brief Table of Contents to the right, or the `Detailed Table
of Contents`_ below.

Description
-----------

The API's basic design works thusly:

- the resource's "pathname" identifies a resource uniquely (like feasts or chants)
- the HTTP method indicates the type action to perform; you may wish to ``GET`` a resource, to
    receive a list of the available ``OPTIONS`` for a resource, or to ``SEARCH`` for resources.
- the HTTP headers will be used to indicate desired data (when sent from client to server) and
    delivered data (when sent from server to client)
- request and response bodies will always be in JSON

.. note::
    Throughout this document, most example URIs begin with ``/``, rather than a fully-qualified
    host and domain, to indicate that they are not relevant in terms of how the API will work.

.. important::
    Always use URLs provided by the server in JSON response bodies, rather than hand-crafting URLs
    ahead of time. While the response body's formatting is set by the API, and therefore will not
    change, the URIs and URLs themselves may change from time to time as the server is improved.

    The partial URL paths are only included here as an aid for the person implementing the server.

Relationship to Existing Standards
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The CANTUS API implements parts of the HTTP/1.1bis and WebDAV standards (IETF RFCs
`7230 <https://tools.ietf.org/html/rfc7230>`_, `7231 <https://tools.ietf.org/html/rfc7231>`_,
`7232 <https://tools.ietf.org/html/rfc7232>`_, `7234 <https://tools.ietf.org/html/rfc7234>`_,
`7237 <https://tools.ietf.org/html/rfc7237>`_ for HTTP; `4918 <https://tools.ietf.org/html/rfc>`_
and `5323 <http://tools.ietf.org/html/rfc5323>`_ for WebDAV).

RFCs 7235 and 7236 will be required for authenticaion in version 2.0. This document supplements the
IETF memoranda to describe what functionality is to be implemented in CANTUS software, and what
behaviour to expect.

High-Level Overview and Examples
--------------------------------

At a high level, knowing how to format a request is primarily an issue of choosing the right URL
path, the right HTTP method, and the right HTTP headers. Parsing the response is about knowing the
expected members in the JSON-formatted response body.

Beacuse this API is designed with the "REST" principles in mind, every session of interaction begins
with a ``GET`` request to the root URL, written in this documentation as ``/``. The response body
indicates the URL path to use in constructing URLs of various resource types. An abbreviated sample
interaction go like this.

Request:

.. sourcecode:: http

    GET / HTTP/1.1

Response:

.. sourcecode:: http

    HTTP/1.1 200 OK

    {
        "resources": {
            "browse_sources": "/sources/{id}/",
            "browse_feasts": "/feasts/{id}/",
            "browse_chants": "/chants/{id}/"
        }
    }

From which you have learned the means by which to make the URIs to sources, feasts, and chants.
Whenever possible, metadata about the results is held in the HTTP headers, since this allows clients
to use easier formatting-for-display algorithms, and also to make use of the ``HEAD`` HTTP method
in some situations.

TODO: rewrite the following example so that it shouldn't use SEARCH
TODO: first consider whether it's a useful example at all, or you're just convincing yourself

In this example request, the client requests the HTTP headers corresponding to a search query for
the "Chant" resources that contain "dixit dominus":

.. sourcecode:: http

    HEAD /chants/?q="dixit_dominus" HTTP/1.1

Consider the following response:

.. sourcecode:: http

    HTTP/1.1 200 OK
    X-SearchResults-Total: 4206
    X-SearchResults-Paginated: true
    X-SearchResults-Page: 1
    X-SearchResults-ResultFrom: 1
    X-SearchResults-ResultTo: 10
    X-SearchResults-Fields: cantus_id incipit source folio genre feast mode siglum

With this header information the client can make an informed decision about how to proceed.
Considering the number of results, the client may decide to automatically help the user get a
high-level view of the results. The following ``GET`` request with the same query asks for 100
results per page, and to receive only the "cantus_id," "incipit," and "source" fields for each
result, to avoid being overloaded with unnecessary data:

.. sourcecode:: http

    GET /chants/?q="dixit_dominus"
    X-SearchResults-Paginated: true
    X-SearchResults-PerPage: 100
    X-SearchResults-Fields: cantus_id incipit source

.. _`version numbers`:

API Versions and Version Numbers
--------------------------------

To help ensure easier interoperability and compatibility of clients and servers, the CANTUS API will
use "semantic versioning." This results in three-part version numbers separated by a period, like
``1.0.0`` and ``4.6.3``, though note that the periods are not decimals, so numbers like ``4.14.4``
are possible.

- The leftmost number is the "major version," and it must be incremented whenever a
  backward-incompatible change is introduced, which should be rarely. Thus clients and servers
  that implemenet different major versions are simply incompatible.
- The middle number is the "minor version," and it must be incremented whenever a new feature is
  introduced. Thus clients and servers that implement different minor versions are compatible if
  only using the features of the lower minor version.
- The rightmost number is the "point release," and it must be incremented whenever an error is fixed
  (since this API is not software and cannot technically fail to implement itself, this will mostly
  consist of clarifications and spelling or grammatical corrections).

Note that this applies *after* the 1.0.0 version. 0.x.x versions are intended for API
development, so arbitrary changes may happen arbitrarily.

Detailed Table of Contents
==========================

.. toctree::
   :maxdepth: 2

   resource_types
   http_methods
   http_headers
   response_bodies
   multiserver

Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
