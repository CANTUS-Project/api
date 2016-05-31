API for HTTP Access to the Cantus Database
==========================================

This is the Application Programming Interface for Cantus database-access projects. The 1.x versions
will only allow searching and reading the database, though (pending funding) 2.x versions will add
the ability to create and edit resources.

To navigate the specification, use the brief Table of Contents to the right, or the `Detailed Table
of Contents`_ below.

Description
-----------

The API's basic design works thusly:

- a resource's URL "pathname" identifies a resource uniquely
- the HTTP method indicates the action to perform; you may wish to :ref:`GET http method` a
  resource, receive a list of the available :ref:`OPTIONS http method` for a resource, or
  :ref:`SEARCH http method` for resources that meet certain criteria.
- the HTTP headers indicate a desired data format (in requests) and actual format plus metadata (in
  responses).
- when present, request and response bodies are normally in JSON, although the default may later
  change to XML.

Why an API for Cantus?
^^^^^^^^^^^^^^^^^^^^^^

This section answers several related questions about why we would bother to specify and write out an
API for Cantus. The quick answer is that we wanted to allow long-term interoperability and stability
across Cantus-related software.

Intentionally writing an API encourages wise design, rather than a haphazard compilation of
functionality. Having a known specification allows developers, both within Cantus and those working
for third parties, to easily know how to use the Cantus database server. Preparing an implementation-
independent API allows us to change any part of the server's implementation without affecting user
agents and third-party software. Offering an HTTP API allows writing new user interfaces, whether
for Web browsers or mobile devices, by only writing new user interface code (whereas replacing a
Drupal-generated user interface requires replacing Drupal's database management code).

The API was developed as part of an effort to replace an existing, Drupal-based database. The
developers wanted to allow easier interoperability with the "Cantus Ultimus" project produced by a
third party, and to develop two user interfaces (one for Web browsers, one for tablet computers),
all using the same server software.

There are undoubtedly additional benefits, and some disadvantages.

.. _`url explanation`:

A Note about URLs
^^^^^^^^^^^^^^^^^

Throughout this document, URLs are usually indicated in a generic format, to emphasize the
unimportance of the specific URL. For this reason, domain names are usually omitted so that URLs
begin with ``/``. In addition, most URLs contain parenthesized parts, as in ``/(browse.chant)/``,
to indicate that server implmentations may change actual URLs arbitrarily, so clients must
determine all URLs dynamically at runtime by consulting the ``"resources"`` member of response
bodies.

For this reason, servers MUST provide clients with relevant URLs in known locations as indicated
throughout the API. In general, there are three *actions* to expect for every resource *type*:

    - browse: to access a sortable, paginated list of all resources of a given type.
    - view: to access a specific
    - search: to browse filter resources by certain criteria

Requests to the root URL (i.e., ``/`` as in ``https://abbott.cantusproject.org/``) will enumerate
the URLs in the following way:

.. sourcecode:: http

    HTTP/1.1 200 OK
    Location (server_address)/
    ...

    {
        "resources":
            {
                "browse": {
                    "chant": "/chants",
                    "feast": "/browse/others/feasts/"
                    },
                "view": {
                    "chant": "/view/chant/id?",
                    "feast": "/view/id?/feast/"
                    }
            }
    }

As you can see, the URLs provided in this example response body are highly irregular, and represent
poor design desicions on the part of the server implementor. Yet because the URLs are provided in a
consistent manner, Cantus user agents may still navigate the database with relative ease. The URL
to use when viewing a specific chant, for instance, will always be available from the
``['resources']['view']['chant']`` member in the response body for the server's root URL.

There are a few other points to note:

    - URLs to related resources will always be held in the ``"resources"`` member of a response
      body, for every resources, throughout the Cantus API.
    - In the ``"resources"`` member, resource types are always in the singular form.
    - "View" URLs will contain the string ``id?``, which should be replaced with the "id" of the
      resource the user agent wishes to access.
    - Search queries use the "browse" URL with the :ref:`SEARCH http method` method; browse queries
      use the :ref:`GET http method` method.

.. _`root url json specification`:

Root URL JSON Specification
***************************

.. http:get:: /
    :synopsis: Get URLs for database resources.

    Fetch JSON object with URLs and URL patterns to access database records. All URLs are
    transmitted as strings. The URL patterns have ``'id?'`` in the string, which must be replaced
    with the :ref:`id <resource ids>` of a specific resource. All other URLs should be used wihtout
    modification.

    :>json object resources: URLs and URL patterns to database resources.
    :>json object resources.browse: URLs to retrieve all resources of a type (with a GET request) or
                                    only resources that match some criteria (with a SEARCH request).
    :>json URL resources.browse.all: URL to browse/search resources of every type.
    :>json URL resources.browse.chant: URL for Chants.
    :>json URL resources.browse.source: URL for Sources.
    :>json URL resources.browse.cantusid: URL for Cantusids.
    :>json URL resources.browse.indexer: URL for Indexers.
    :>json URL resources.browse.feast: URL for Feasts.
    :>json URL resources.browse.genre: URL for Genres.
    :>json URL resources.browse.century: URL for Centuries.
    :>json URL resources.browse.notation: URL for Notations.
    :>json URL resources.browse.office: URL for an Offices.
    :>json URL resources.browse.portfolio: URL for Portfolio Categories (portfolia).
    :>json URL resources.browse.provenance: URL for Provenances.
    :>json URL resources.browse.siglum: URL for RISM Siglums (sigla).
    :>json URL resources.browse.segment: URL for Database Segments.
    :>json URL resources.browse.status: URL for Source Statuses.
    :>json object resources.view: URL patterns to retrieve single resources with a known "id."
    :>json URL resources.view.chant: Pattern for a :ref:`chant resource type`.
    :>json URL resources.view.source: Pattern for a :ref:`source resource type`.
    :>json URL resources.view.cantusid: Pattern for a :ref:`cantusid resource type`.
    :>json URL resources.view.indexer: Pattern for a :ref:`indexer resource type`.
    :>json URL resources.view.feast: Pattern for a :ref:`feast resource type`.
    :>json URL resources.view.genre: Pattern for a :ref:`genre resource type`.
    :>json URL resources.view.century: Pattern for a :ref:`Century <simple resource types>`.
    :>json URL resources.view.notation: Pattern for a :ref:`Notation <simple resource types>`.
    :>json URL resources.view.office: Pattern for an :ref:`Office <simple resource types>`.
    :>json URL resources.view.portfolio: Pattern for a :ref:`Portfolio Category <simple resource types>`.
    :>json URL resources.view.provenance: Pattern for a :ref:`Provenance <simple resource types>`.
    :>json URL resources.view.siglum: Pattern for a :ref:`RISM Siglum <simple resource types>`.
    :>json URL resources.view.segment: Pattern for a :ref:`Database Segment <simple resource types>`.
    :>json URL resources.view.status: Pattern for a :ref:`Source Status <simple resource types>`.

Find Server Version
^^^^^^^^^^^^^^^^^^^

The fastest and safest way to ensure a user agent and server use compliant versions of the Cantus
API is to use a :ref:`HEAD http method` or :ref:`OPTIONS http method` request on the root URL.

.. http:head:: /

    Find the API version supported on the server.

    :resheader X-Cantus-Version: Indicates the Cantus API version implemented by a client or server.
        Values are described in :ref:`version numbers`.
    :resheader Server: Indicates the implementation and its version. This SHOULD NOT be used by the
        user agent to modify behaviour, but it MAY be of interest for debugging or other purposes.
        The reference implementation is called "Abbot."


Relationship to Existing Standards
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Cantus API implements parts of the HTTP/1.1bis standard (IETF RFCs
`7230 <https://tools.ietf.org/html/rfc7230>`_, `7231 <https://tools.ietf.org/html/rfc7231>`_,
`7232 <https://tools.ietf.org/html/rfc7232>`_, `7234 <https://tools.ietf.org/html/rfc7234>`_, and
`7237 <https://tools.ietf.org/html/rfc7237>`_).

.. possible future standards:
    webdav: `RFC 4918 <https://tools.ietf.org/html/rfc>`_
            `5323 <http://tools.ietf.org/html/rfc5323>`_
    authentication: `RFC 7235 <http://tools.ietf.org/html/rfc7235>`_
                     `7236 <http://tools.ietf.org/html/rfc7236>`_

This document supplements the IETF memoranda to describe what functionality is to be implemented in
Cantus software, and what behaviour to expect. In particular, the Cantus API defines
application-specific :ref:`headers <cantus headers>`, :ref:`content formatting <response bodies>`,
and a :ref:`search specification <searching>`.

Keywords indicating technical requirements are used as per `RFC 2119 <https://tools.ietf.org/html/rfc2119>`_.

High-Level Overview and Examples
--------------------------------

At a high level, knowing how to format a request is primarily an issue of choosing the right URL
path, the right HTTP method, and the right HTTP headers. Parsing the response is about knowing the
expected members in the JSON-formatted response body.

Beacuse this API is designed with the "REST" principles in mind, every session of interaction begins
with a ``GET`` request to the root URL, written in this documentation as ``/``. The response body
indicates the URL path to use in constructing URLs of various resource types. An abbreviated sample
interaction goes like this.

Request:

.. sourcecode:: http

    GET / HTTP/1.1

Response:

.. sourcecode:: http

    HTTP/1.1 200 OK
    ...

    {
        "resources": {
            "browse": {
                "source": "/sources/",
                "feast": "/feasts/",
                "chant": "/chants/",
            ...
        }
    }

From which you have learned the means by which to make the URLs to sources, feasts, and chants.
Whenever possible, metadata about the results is held in the HTTP headers, since this allows clients
to use easier formatting-for-display algorithms, and also to make use of the ``HEAD`` HTTP method
in many situations.

In this example request, the client requests the HTTP headers corresponding to a search query for
the "Chant" resources that contain "dixit dominus":

.. sourcecode:: http

    HEAD /(browse_chants)/357685/ HTTP/1.1
    If-None-Match "7827ff38cb8ef147d9f5edb749a0f300dac2ebe1"

Consider the following response:

.. sourcecode:: http

    HTTP/1.1 304 Not Modified
    ETag "7827ff38cb8ef147d9f5edb749a0f300dac2ebe1"
    Server "Abbott/1.0"
    X-Cantus-Fields: feast differentia position genre source sequence office fest_desc cantus_id id mode full_text incipit folio
    ...

With this (abbreviated) header information the client can make an informed decision about how to
proceed. The :http:header:`If-None-Match` and :http:header:`ETag` headers tell us that this chant's
content has not changed since the last time the client checked, so transmitting the response body
would have been redundant anyway. With the :http:header:`X-Cantus-Fields` header we also know the
fields the database has for this chant---and more importantly that some of the fields are mising
(like ``marginalia`` and ``volpiano``).

.. _`version numbers`:

API Versions and Version Numbers
--------------------------------

To help ensure easier interoperability and compatibility of clients and servers, the Cantus API will
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
   searching
   multiserver

Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
