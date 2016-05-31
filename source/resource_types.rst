.. _`resource types`:

Resource Types
==============

You must discover the URL paths to all resource types with the URLs provided by the server. However,
resource types available in a context will always be named consistently. Those names are given at
the root URL as described in the :ref:`root url json specification`.

Consider the following example, of submitting a request to the root URL:

.. sourcecode:: http

    GET / HTTP/1.1

.. sourcecode:: http

    HTTP/1.1 200 OK

    ...
    {
        "resources": {
            "browse": {
                "genre": "/genres/",
                ...
                },
            "view": {
                "genre": "/view_genre/id?",
                ...
                },
            ...
        }
    }

To retrieve a paginated list of all genres, you would submit a ``GET`` request to ``/genres/``. To
access information about a particular genre, say genre 25, submit a request to ``/view_genre/25/``.
On a different deployment, or with a different implementation or version of the Cantus server, the
URL path may be entirely different (e.g., ``/genre-browser``). However, if ``["browse"]["genres"]``
appears in the ``"resources"`` member of a response, it will always point to the list of genres, and
``["view"]["genres"]`` will always give information related to a specific genre.

While the URLs are unlikely to change frequently (possibly never) in the above example, in the
following example it is much more important to pay careful attention to using the URLs provided by
the server in the "resources" member.

You submit a query about "Benedicamus patrem et filium" chants:

.. sourcecode:: http

    SEARCH https://abbott.uwaterloo.ca/chants/ HTTP/1.1

    {"query": "'benedicamus patrem et filium'"}

.. sourcecode:: http

    HTTP/1.1 200 OK

    ...
    {
        "556270": {
            "source": "Wolfenbüttel, Herzog August Bibliothek - Musikabteilung, 32",
            ...
        },
        "436440": {
            "source": "Salamanca, Catedral - Archivo Musical, 6",
            ...
        },
        "pem-8669": {
            "source": "P-Cug (Coimbra) Biblioteca Geral da Universidade MM 45",
            ...
        },
        "resources": {
            ...
            "556270": {"self": "https://abbott.uwaterloo.ca/chants/556270/"},
            "436440": {"self": "https://abbott.uwaterloo.ca/chants/436330/"},
            "pem-8669": {"self": "http://pemdatabase.eu/musical-item/8669"},
            ...
        }
    }

While the Cantus server has cached some metadata about all the chants, you can see from the provided
URLs that resource ``"pem-8669"`` is actually hosted on the Portuguese Early Music server, and we
refer the client to that URL for further information. Note also that the "id" field is not
consistent between the two servers: in this case, the ``uwaterloo.ca`` Cantus server has prefixed
the ``pemdatabase.eu`` server's "id" with ``"pem-"`` to avoid a resource "id" conflict.

As a reminder, the Cantus server will only "switch out" a resource provider like this when it is
API-compatible, so this "resources" URL represents a promise that the PEM server supports the
Cantus API. For more information, refer to :ref:`multiserver`.

.. _`resource ids`:

About the "id" Field
--------------------

For "view" URLs, where ``id?`` makes part of the server-provided URL, user agents MUST form a full
URL by substituting a resource's unique "id" value in that part of the URL. The full three-character
string, ``id?``, must be removed from the URL.

Cantus API "id" values may consist of the characters A through Z, a through z, and the digits 1
though 0, plus hyphens and underscores. The first and last characters of an "id" MUST NOT be a
hyphen or underscore (`-` or `_`). If a user agent requests a resource with an invalid "id" the
server MUST respond with the :http:statuscode:`422` HTTP status code.

The following points also apply:

- A resource's "id" MUST refer to the same resource through the existence of that "id." However, a
  resource may change its "id" at any time.
- Changing attributes, properties, or data in a resource MUST NOT change the "id" field.
- A resource's "id" field MAY be prefixed with an identifier indicating which database holds the
  resources's authoritative copy.
- The same "id" MAY or may not refer to "the same" resource when served by a different deployment of
  a Cantus server application. That is, the Cantus API does not guarantee uniqueness of "id" values
  across deployments.

.. _`simple resource types`:

Simple Resource Types
---------------------

Unlike the types listed in the following section (:ref:`complex resource types`) the resources in
this category will not have fields that cross-reference another resource. Simple resources also tend
to have fewer fields, and are not expected to change often during the lifetime of the
database---perhaps never.

.. http:get:: /(view.simple_resource)/(string:id)/

    Most simple resources follow this pattern.

    :resjson string id: The "id" of this resource.
    :resjson string name: Human-readable name for the resource.
    :resjson string description: Brief explanation of the resource.
    :resjson string drupal_path: Optional URL to this resource on the Cantus Drupal instance.

The following simple resource types use the default fields described above:

    - century
    - notation
    - office
    - portfolio categories
    - provenance
    - RISM siglum (*pl.* sigla)
    - segment
    - source status

Three simple resource types use additional fields, or different fields, than those described above.

.. _`indexer resource type`:

Indexer
^^^^^^^

.. http:get:: /(view.indexer)/(string:id)/

    An "Indexer" corresponds to a human who has entered or modified data in the Cantus Database.
    The set of fields is entirely different from other simple resource types.

    :resjson string id: The "id" of this resource.
    :resjson string display_name: The indexer's name, as displayed.
    :resjson string given_name: The indexer's given name.
    :resjson string family_name: The indexer's family name.
    :resjson string institution: The indexer's associated university or research institution.
    :resjson string city: The city where the indexer lives.
    :resjson string country: The country where the indexer lives.
    :resjson string drupal_path: Optional URL to this resource on the Cantus Drupal instance.

.. _`feast resource type`:

Feast
^^^^^

.. http:get:: /(view.feast)/(string:id)/

    Feast resources add two fields over the standard set for "simple" resources.

    :resjson string id: The "id" of this resource.
    :resjson string name: Human-readable name for the feast.
    :resjson string description: Brief explanation of the feast.
    :resjson string date: Date on which the feast occurs.
    :resjson string feast_code: Standardized feast code for the feast.
    :resjson string drupal_path: Optional URL to this resource on the Cantus Drupal instance.

.. _`genre resource type`:

Genre
^^^^^

.. http:get:: /(view.genre)/(string:id)/

    Genre resources add one field over the standard set for "simple" resources.

    :resjson string id: The "id" of this resource.
    :resjson string name: Human-readable name for the genre.
    :resjson string description: Brief explanation of the genre.
    :resjson string mass_or_office: A case-insensitive string, either ``"mass"`` or ``"office"``.
    :resjson string drupal_path: Optional URL to this resource on the Cantus Drupal instance.

.. _`complex resource types`:

Complex Resource Types
----------------------

The following resource types (CantusID, Chant, Indexer, Source) hold many data fields, some of which
cross-reference a simple resource described in the previous section, or another complex resource.

.. _`cantusid resource type`:

CantusID
^^^^^^^^

.. http:get:: /(view.cantusid)/(string:id)/

    A "Cantus ID" resource is an abstraction across multiple actual chants. These are available at
    the URLs indicated by ``["view"]["cantusid"]`` and ``["browse"]["cantusid"]``.

    Note that the "incipit" and "full_text" fields are not necessarily the same across all chants
    with the same Cantus ID and therefore may not be correct for a particular chant.

    :resjson string id: the Cantus ID of this resourse
    :resjson string genre: ``"name"`` field of the corresponding "Genre" resource
    :resjson string incipit: the chant's incipit with standardized spelling
    :resjson string full_text: full text with standardized spelling
    :resjson string drupal_path: Optional URL to this resource on the Cantus Drupal instance.

.. _`chant resource type`:

Chant
^^^^^

.. http:get:: /(view.chant)/(string:id)/

    A "Chant" resource is a chant written in a Source. These are available at the URLs indicated by
    ``["view"]["chant"]`` and ``["browse"]["chant"]``.

    :resjson string id:
    :resjson string incipit:
    :resjson string source: the "title" field of the corresponding "Source" resource
    :resjson string marginalia:
    :resjson string folio: E.g., ``"05v"``
    :resjson string sequence:
    :resjson string office: the "name" field of the corresponding "Office" resource
    :resjson string genre: the "name" field of the corresponding "Genre" resource, provided through the "CantusID" resource
    :resjson string position:
    :resjson string cantus_id: ``"id"`` field of the corresponding "CantusID" resource
    :resjson string feast: ``"name"`` field of the corresponding "Feast" resource (e.g., "Dom. 21 p. Pent.")
    :resjson string feast_desc: ``"description"`` of the corresponding "Feast" resource (e.g., "21st Sunday after Pentecost")
    :resjson string mode: (will appear in ``"resources"`` after the first version)
    :resjson string differentia:
    :resjson string finalis: (will appear in ``"resources"`` after the first version)
    :resjson string full_text: ``"full_text"`` of the corresponding "CantusID" resource
    :resjson string full_text_manuscript: full text as written in the manuscript
    :resjson string volpiano: neume information to be rendered with the "Volpiano" font
    :resjson string notes:
    :resjson string cao_concordances:
    :resjson string siglum: the "siglum" field of the corresponding "Source" resource
    :resjson string proofreader: ``"display_name"`` of an "Indexer" resource
    :resjson string melody_id: (will appear in ``"resources"`` after the first version)
    :resjson string drupal_path: Optional URL to this resource on the Cantus Drupal instance.
    :resjson string proofread_fulltext: Whether the "fulltext" field was proofread.
    :resjson string proofread_fulltext_manuscript: Whether the "fulltext_manuscript" field was proofread.
    :resjson string resources>source: URL to the containing "Source" resource
    :resjson string resources>source_id: resource ID for previous
    :resjson string resources>office: URL to the corresponding "Office"
    :resjson string resources>office_id: resource ID for previous
    :resjson string resources>genre: URL to the corresponding "Genre"
    :resjson string resources>genre_id: resource ID for previous
    :resjson string resources>feast: URL to the corresponding "Feast" resource
    :resjson string resources>feast_id: resource ID for previous
    :resjson string resources>image_link: URL to an image, or a Web page with an image, of this Chant
    :resjson string resources>proofreader: URL to an "Indexer" resource
    :resjson string resources>proofreader_id: resource ID for previous

.. _`source resource type`:

Source
^^^^^^

.. http:get:: /(view.source)/(string:id)/

    A "Source" resource is for a collection of folia containing Chants (usually a book). These are
    be avialable at the URLs indicated by ``["view"]["source"]`` and ``["browse"]["source"]``.

    :resjson string id: The "id" of this resource.
    :resjson string title: Full Manuscript Identification (City, Archive, Shelf-mark)
    :resjson string rism: RISM number
    :resjson string siglum: Siglum
    :resjson string provenance: Provenance
    :resjson string provenance_detail: More detail about the provenance
    :resjson string date: Date
    :resjson string century: Century
    :resjson string notation_style: Notation used for the source
    :resjson string editors: List of ``"display_name"`` of indexers who edited this manuscript
    :resjson string indexers: List of ``"display_name"`` of indexers who entered this manuscript
    :resjson string proofreaders: List of ``"display_name"`` of indexers who proofread this manuscript
    :resjson string segment: Segment (i.e., source database)
    :resjson string source_status: Status of this source
    :resjson string source_status_desc: Elaboration of ``"source_status"``---probably never used.
    :resjson string summary: Summary
    :resjson string liturgical_occasions: Liturgical occasions
    :resjson string description: Description
    :resjson string indexing_notes: Indexing notes
    :resjson string indexing_date: Indexing date
    :resjson string drupal_path: Optional URL to this resource on the Cantus Drupal instance.
    :resjson object resources: Links to other indexer who share the same characteristics.
    :resjson string resources>provenance:
    :resjson string resources>provenance_id: resource ID for previous
    :resjson string resources>century:
    :resjson string resources>century_id: resource ID for previous
    :resjson string resources>notation_style:
    :resjson string resources>notation_style_id: resource ID for previous
    :resjson string resources>editors: List of URLs to Indexer resources.
    :resjson string resources>editors_id: resource ID for previous
    :resjson string resources>indexer: List of URLs to Indexer resources.
    :resjson string resources>indexer_id: resource ID for previous
    :resjson string resources>proofreaders: List of URLs to Indexer resources.
    :resjson string resources>proofreaders_id: resource ID for previous
    :resjson string resources>source_status:
    :resjson string resources>source_status_id: resource ID for previous
    :resjson string resources>image_link: Root URL linking to images for the entire source.
