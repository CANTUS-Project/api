.. _`searching`:

How to Search
=============

Every resource type that may be searched will provide a "search" URL in its "resources" response.
As the "browsing" URL is the resource type prefixed with ``"browse_"``, the "searching" URL is the
resource type prefixed with ``"search_"``. For example, you may submit a request to the root URL:

.. sourcecode:: http

    GET / HTTP/1.1

And receive a response like this:

.. sourcecode:: http

    HTTP/1.1 200 OK

    ...
    {
        "resources": {
            "browse_genres": "/genres/id?/",
            "search_genres": "/genres/search/id?/",
            ...
        }
    }

Thus in this case clients should use the ``/genres/search/`` URL to search for a genre, and the
``/genres/search/id?/`` URL to search among Chants associated with a specific genre. Search
requests are submitted as a ``GET`` request to the indicated URL, with a specially-formatted request
body and optional headers. (Refer to :ref:`cantus headers` for more information about headers
relevant to searching).

Knowing which search URL to use can seem confusing; here are some simple guidelines:

    #. If you want one resource type, search with its URL. For example, use ``/(search_genres)/``
       to find a genre).
    #. To limit results to those associated with a particular resource, use that resource's search
       URL. For example, use ``/(search_genres)/(antiphon_genre_id)/`` to search among the Chants
       that are antiphons.
    #. To limit results with multiple resources, choose one of the above methods. For example, to
       find Easter antiphons, you could:

       - use the ``/(search_chants)/`` URL and provide the limiting genre and feast in the request body,
       - use ``/(search_genres)/`` with an "id" and provide the feast "id" in the request body, or
       - use ``/(search_feasts)/`` with an "id" and provide the genre "id" in the request body.

       The results are the same in all three situations, so your choice is merely a preference.
    #. If you do not know the limiting "id" then try to use a name-based sub-query as described
       :ref:`below <name-based filter>`.

.. note::

    An Idea for the Future

    Arbitrarily nestable search URLs. For example, ``/(search_sources)/123/(search_offices)/63/``
    would return all the Chants in a book that are in a particular office. You can already get the
    same functionality by using the search request body, but maybe this is more convenient.

Search Request
--------------

With the possible exception of a resource ID present in the URL, search parameters are carried in
the request body. Some additional parameters, used to change the format of results, are carried in
headers, as specified below in :ref:`search request headers`.

The format of a search request body depends on the search grammar. At present, the only supported
grammar is :ref:`described below <cantus query grammar>`.

TODO: write other things here

.. _`cantus query grammar`:

"cantus" Query Grammar
----------------------

The "cantus" query grammar is inspired by Solr's "standard query parser," but differs notably.
Unlike with Solr, query parameters are part of the request body rather than the URL---as
required by the HTTP ``SEARCH`` method. Also, query parameters are modified and added by the
server implementation before being sent to Solr, in order to fetch the expected results. Therefore,
even if a server implementation does use Solr, which is not required by this API, clients should not
expect their query will be submitted to the Solr server verbatim. This query grammar will be
retained when the Cantus API becomes WebDAV compatible.

All parameters belong in the "query" member of the request body, described in :ref:`query string syntax`.

The fields available depends on the resource type being queried (refer to the relevant
:ref:`resource types` subsection for more information). Some fields---those that refer to a resource
type---also have a variant suffixed with "_id"to allow more accurate :ref:`id-based filter`. For
those resources, ID-based filtering is preferred; otherwise a :ref:`name-based filter` will
happen.

For example, a query at the ``/(search_sources)/`` URL may use the following content-based fields:
id, title, siglum, provenance_detail, date, source_status_desc, summary, liturgical_occasions,
description, indexing_notes, and indexing_date. In addition, the following fields correspond to
another resource, so they may be used in ID-based filtering with an "_id" suffix, or in a name-based
sub-query: rism, provenance, century, notation_style, editors, indexers, proofreaders, segment,
and source_status.

In all cases, any unknown, invalid, or inapplicable data are ignored. If all data are ignored, an
empty result body will be provided. For example, a search to the ``/(search_sources)/`` URL for
``{'query': '+city:Waterloo'}`` will always return no results because Source resources do not have
a "city" field.

.. _`query string syntax`:

Syntax in the "query" String
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The syntax of this string is kept as close as possible to that of the Solr standard query parser.
The "query" string MUST NOT use URL encoding, but it SHOULD be escaped in the same way as any other
JavaScript string.

You may include search terms the following ways:

- Term searches by using that word (e.g., ``'antiphon'``). Beware this does not match similar terms,
  or partial terms---"antiphoner" will not be included in the results of this search.
- Phrase searches with ``"`` (e.g., ``'"of bingen"'`` will not match "bingen" unless preceded by "of").
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

ID-based Filtering
^^^^^^^^^^^^^^^^^^

When you want to limit search results by a particular resource and you know its "id," use a ID-based
filter. This search strategy is more accurate than name-based sub-queries, so we prefer it whenever
possible.

TODO: finish this

.. _`name-based filter`:

Name-based Sub-query
^^^^^^^^^^^^^^^^^^^^

When you want to limit search results by a particular resource but you do not know the "id," you
can use a name-based sub-query to avoid submitting two queries. For example, to search for Easter
antiphons that mention "jesus" in the incipit, you might submit this query:

.. sourcecode:: http

    GET /(search_chants)/ HTTP/1.1

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

    GET /(search_feasts)/ HTTP/1.1

    {"name": "pascha"}
    <!-- returns one feast with an id of "08020100" -->

.. sourcecode:: http

    GET /(search_genres)/ HTTP/1.1

    {"name": "antiphon"}
    <!-- returns one genre with an id of "422" -->

.. sourcecode:: http

    GET /(search_chants)/ HTTP/1.1

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

.. _`search request headers`:

Use Headers to Change the Result Format
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

asdf

Search Result
-------------

asdf

Resource-Specific Information
-----------------------------

Query without an "id" string to search amongst the resources of that type (e.g., to find a
particular feast use ``/(search_feasts)/``), or with an "id" string to find other resources
associated with a particular resource (e.g., to find all the chants that happen at Compline).

.. http:get:: /(search_indexers)/(string:id)/

    With an "id" string, finds all the Chant and Source resources created, edited, or indexed by
    an Indexer.

.. http:get:: /(search_chants)/(string:id)/

    With an "id" string, finds all the other resources associated with a Chant (i.e., the Source,
    the Office, the Siglum, and so on).

.. http:get:: /(search_sources)/(string:id)/

    With an "id" string, finds all the Chant resources in this Source.

.. http:get:: /(search_centuries)/(string:id)/

    With an "id" string, finds all the Chant and Source resources in a century.

.. http:get:: /(search_feasts)/(string:id)/

    With an "id" string, finds all the Chants associated with a feast.

.. http:get:: /(search_genres)/(string:id)/

    With an "id" string, finds all the Chants in a genre.

.. http:get:: /(search_notations)/(string:id)/

    With an "id" string, finds all the Sources written with a particular notation style.

.. http:get:: /(search_offices)/(string:id)/

    With an "id" string, finds all the Chants that happen in an office.

.. http:get:: /(search_provenances)/(string:id)/

    With an "id" string, finds all the Chants and Sources from a monastery.

.. http:get:: /(search_sigla)/(string:id)/

    With an "id" string, finds all the Chants and Sources with a siglum.

Unsearchable Resource Types
---------------------------

I decided it did not make sense to search for these---users will always want to search something
else too.

* Portfolio
* Segment
* Source Status