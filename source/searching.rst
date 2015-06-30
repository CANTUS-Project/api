.. _`searching`:

How to Search
=============

Resources are searchable with a specially-formatted :http:method:`SEARCH` request to their "browse"
URL. For example, you may submit a request to the root URL:

.. sourcecode:: http

    GET / HTTP/1.1

And receive a response like this:

.. sourcecode:: http

    HTTP/1.1 200 OK

    ...
    {
        "resources": {
            "browse": {
                "genre": "/genres/",
                },
            "view": {
                "genre": "/genre-view/id?"
                },
        }
    }

Thus in this case clients should use the ``/genres/`` URL to search for a genre. Most of the rest
of this section describes how to format the :http:method:`SEARCH` request to obtain desired results.

The Cantus API also allows searching for resources of unknown type. For this use the
:http:search:`/(browse.all)/` URL.

Search Request
--------------

The request body for a :http:method:`SEARCH` request must be a JSON string. One member is required,
``"query"``, which holds the search query in a string. The :ref:`cantus headers` section describes
which headers may be used to modify the formatting of search results. The following request headers
may also be incorporated directly in the SEARCH request body, as indicated below:

    - X-Cantus-Include-Resources as ``"include-resources"``
    - X-Cantus-Fields as ``"fields"``
    - X-Cantus-No-Xref as ``"no-xref"``
    - X-Cantus-Per-Page as ``"per-page"``
    - X-Cantus-Page as ``"page"``
    - X-Cantus-Sort as ``"sort"``
    - X-Cantus-Search-Help as ``"search-help"``

If a header and request body member hold conflicting values (e.g., ``X-Cantus-Search-Help: true`` in
the headers but ``"search-help": false`` in the request body) the server MUST use the value from the
request body, disregarding the value from the header, even if the request body's value is invalid
and the header's value is valid. The same error response codes SHOULD be provided as for an error
in the corresponding header.

The grammar of the ``"query"`` string is described below in :ref:`<cantus query grammar>`.

Example Search Query
^^^^^^^^^^^^^^^^^^^^

The following search request example yields all the chants on a folio of the same manuscript. You
may "flip the pages" of the manuscript by changing the "folio" value in the ``"query"`` string.

.. sourcecode:: http

    SEARCH /(browse.chants)/ HTTP/1.1
    X-Cantus-Per-Page: 50

    {
        "query": "+source_id:123614 +folio:042r",
        "sort": "sequence,asc"
    }

The following query is also possible, replacing the ``source_id`` field with ``source``. The server
will automatically search for the ``source_id`` on behalf of the user agent, but this is obviously
more error-prone than using the ``source_id`` directly, if it is known to the user agent. Refer to
the :ref:`lengthier discussion below <name-based filter>` for more information.

.. sourcecode:: http

    SEARCH /(browse.chants)/ HTTP/1.1
    X-Cantus-Per-Page: 50

    {
        "query": '+source:"Klosterneuburg, Augustiner-Chorherrenstift - Bibliothek, 1010" +folio:042r',
        "sort": "sequence,asc"
    }

Although it would be a nice touch, you cannot use the ``X-Cantus-Page`` header to "flip the pages"
in the manuscript. Furthermore, if the ``X-Cantus-Per-Page`` header is not set manually to an
arbitrarily high value, users may inadvertently miss some chants on some pages.

.. _`cantus query grammar`:

"cantus" Query Grammar
----------------------

The "cantus" query grammar is inspired by Solr's "standard query parser," but differs notably.
Unlike with Solr, query parameters are part of the request body rather than the URL---as
required by the HTTP :http:method:`SEARCH` method. Also, query
parameters are modified and added by the server implementation before being sent to Solr, in order
to fetch the expected results. Therefore, even if a server implementation does use Solr, which is
not required by this API, clients should not expect their query will be submitted to the Solr
server verbatim. This query grammar will be retained even if additional grammars become available
in future versions of the API.

All parameters belong in the "query" member of the request body, described in :ref:`query string syntax`.

The fields available depends on the resource type being queried (refer to the relevant
:ref:`resource types` subsection for more information). Some fields---those that refer to a resource
type---also have a variant suffixed with "_id"to allow more accurate :ref:`id-based filter`. For
those resources, ID-based filtering is preferred; otherwise a :ref:`name-based filter` will
happen.

For example, a query at the ``/(browse.source)/`` URL may use the following content-based fields:
id, title, siglum, provenance_detail, date, source_status_desc, summary, liturgical_occasions,
description, indexing_notes, and indexing_date. In addition, the following fields correspond to
another resource, so they may be used in ID-based filtering with an "_id" suffix, or in a name-based
sub-query: rism, provenance, century, notation_style, editors, indexers, proofreaders, segment,
and source_status.

In all cases, any unknown, invalid, or inapplicable data are ignored. If all data are ignored, an
empty result body will be provided. For example, a search to the ``/(browse.source)/`` URL for
``{'query': '+city:Waterloo'}`` will always return no results because Source resources do not have
a "city" field.

.. _`query string syntax`:

Syntax in the "query" String
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The syntax of this string is kept as close as possible to that of the Solr standard query parser.
The "query" string MUST NOT use URL encoding, but it SHOULD be escaped in the same way as any other
JavaScript string.

You may include search terms the following ways:

- Term searches by using that word (e.g., ``antiphon``). Beware this does not match similar terms,
  or partial terms---"antiphoner" will not be included in the results of this search.
- Phrase searches with ``"`` (e.g., ``'"of bingen"'`` will not match "bingen" unless preceded by
  "of"). Note that this requires double-quote marks; single-quote marks will not work.
- Wildcard with ``?`` and ``*``, matching a single character and zero or more characters,
  respectively. You may want to use the ``*`` wildcard more often than not, since not using it may
  lead to fewer results than expected.
- Fuzzy searches by appending ``~``, which returns results arbitrarily similar to a term. For
  example, ``antiphon~`` would also match "antiphoner."
- Proximity searching with ``~`` and an integer, as in ``"manuscript available"~5``, which matches
  "manuscript is available" and "manuscript is freely available."
- Range searches, as in ``date:[1300 TO 1400}`` matches the "date" field between 1300 and 1400,
  including 1300 itself but not 1400 itself. May also use alphabetically ordered ranges.
- Boosting term or phrases with ``^`` and a positive number. The default boost value is 1. The
  higher a term's score including boost, the higher it will appear in the default sort (that is,
  unless the sort field is changed).
- Field specification with ``:``, as in ``'incipit:*deus*'``, which will return every Chant where
  "deus" is part of the "incipit" field.
- Boolean operators ``&&``, ``!``, and ``||``, or their word equivalents ``AND``, ``NOT``, and
  ``OR``, which must be capitalized.
- Requirement operators ``+`` and ``-``. which require that a term is or is not present in the
  results, respectively. The default (not using these symbols) means that a term is optional, though
  documents matching more terms will have a higher relevance score.
- Grouping with ``()``, as in ``'(cat AND breading) OR silliness'``.

Refer to `this page <https://cwiki.apache.org/confluence/display/solr/The+Standard+Query+Parser>`_
for more complete descriptions. Clients may provide users the opportunity to use the
:ref:`X-Cantus-Search-Help` header, which allows the server to run a less strict query in
the hope it will return more results.

Fetching a Resource with Its "id"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It is possible to fetch a single resource with a known "id" value using a ``SEARCH`` query,
though we recommend you use the resource's URL. For example, ``/(browse_indexer)/14`` will fetch
the Indexer with an "id" of ``14``. This requires less server-side processing, and reduces the
chance of other query parameters interfering. However, the "id" field is still useful in a
``SEARCH`` query to obtain a range. For example, ``id:[14 TO 16]`` will return the resources
with "id" of ``14``, ``15``, and ``16``.

.. _`id-based filter`:

ID-based Query
^^^^^^^^^^^^^^

When you want to limit search results by a particular resource and you know its "id," use an ID-based
filter. This search strategy is more accurate than name-based sub-queries, so we prefer it whenever
possible.

For example, if the "id" of the "Antiphon" genre is ``122``, the "id" of the "Ljubljana,
Nadškofijski arhiv (Archiepiscopal Archives), 19 (olim 18)" source is ``123659``, and the "id" of
the "Jacobi" feast is ``2378``, we can find all the Jacobi antiphons in that manuscript with the
following query:

.. sourcecode:: http

    SEARCH /(browse.chants)/ HTTP/1.1

    {"query": "genre_id:122 AND source_id:123659 AND feast_id:2378"}

The equivalent name-based query follows:

.. sourcecode:: http

    SEARCH /(browse.chants)/ HTTP/1.1

    {"query": 'genre:antiphon AND source:"Ljubljana, Nadškofijski arhiv (Archiepiscopal Archives), 19 (olim 18)" AND feast:Jacobi'}

.. _`name-based filter`:

Name-based Query
^^^^^^^^^^^^^^^^

When you want to limit search results by a particular resource but you do not know the "id," you
can use a name-based sub-query to avoid submitting two queries. For example, to search for Easter
antiphons that mention "jesus" in the incipit, you might submit this query:

.. sourcecode:: http

    SEARCH /(browse.chant)/ HTTP/1.1

    {
        "incipit": "jesus",
        "feast": "pascha",
        "genre": "antiphon",
    }

On the server side, the "_name" fields are first replaced with the corresponding "_id" fields by
running a search on the appropriate resource type where the "_name" field is "name," and using *all*
the returned "id" values in a final search. For example, the preceding example is equivalent to
submitting the following three queries:

.. sourcecode:: http

    SEARCH /(browse.feast)/ HTTP/1.1

    {"name": "pascha"}
    <!-- returns one feast with an id of "08020100" -->

.. sourcecode:: http

    SEARCH /(browse.genre)/ HTTP/1.1

    {"name": "antiphon"}
    <!-- returns one genre with an id of "422" -->

.. sourcecode:: http

    SEARCH /(browse.chant)/ HTTP/1.1

    {
        "incipit": "jesus",
        "feast_id": "08020100",
        "genre_id": "422",
    }

The benefit of a name-based sub-query is that using fewer requests means transmitting less data
and getting results sooner. The disadvantage is that the results may be much less useful if the
"field_name" result provides many more results, or unexpected results. The preceding search, for
example, returns results associated with the "Pascha Annotinum" feast, which is not Easter. Because
it is virtually impossible for a client or server to predict whether users are running into this
problem, ID-based filtering is preferred whenever a resource "id" is available.

Unsearchable Resource Types
---------------------------

I decided it did not make sense to search for these---users will always want to search something
else too.

* Cantusid
* Portfolio
* Segment
* Source Status
