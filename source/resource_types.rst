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
string, ``id?``, must be removed from the URL. Cantus API "id" values may consist of any alphanumeric
character valid in a URL, plus hyphens and underscores.

The following points also apply:

- A resource's "id" MUST NOT change through the resource's lifetime.
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

    :>json string id: The "id" of this resource.
    :>json string name: Human-readable name for the resource.
    :>json string description: Brief explanation of the resource.

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

    :>json string id: The "id" of this resource.
    :>json string display_name: The indexer's name, as displayed.
    :>json string given_name: The indexer's given name.
    :>json string family_name: The indexer's family name.
    :>json string institution: The indexer's associated university or research institution.
    :>json string city: The city where the indexer lives.
    :>json string country: The country where the indexer lives.

.. _`feast resource type`:

Feast
^^^^^

.. http:get:: /(view.feast)/(string:id)/

    Feast resources add two fields over the standard set for "simple" resources.

    :>json string id: The "id" of this resource.
    :>json string name: Human-readable name for the feast.
    :>json string description: Brief explanation of the feast.
    :>json string date: Date on which the feast occurs.
    :>json string feast_code: Standardized feast code for the feast.

.. _`genre resource type`:

Genre
^^^^^

.. http:get:: /(view.genre)/(string:id)/

    Genre resources add one field over the standard set for "simple" resources.

    :>json string id: The "id" of this resource.
    :>json string name: Human-readable name for the genre.
    :>json string description: Brief explanation of the genre.
    :>json string mass_or_office: A case-insensitive string, either ``"mass"`` or ``"office"``.
    :>json string drupal_path: Optional URL to this resource on the Cantus Drupal instance.

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

    :>json string id: the Cantus ID of this resourse
    :>json string genre: ``"name"`` field of the corresponding "Genre" resource
    :>json string incipit: the chant's incipit with standardized spelling
    :>json string full_text: full text with standardized spelling

.. _`chant resource type`:

Chant
^^^^^

.. http:get:: /(view.chant)/(string:id)/

    A "Chant" resource is a chant written in a Source. These are available at the URLs indicated by
    ``["view"]["chant"]`` and ``["browse"]["chant"]``.

    :>json string id:
    :>json string incipit:
    :>json string source: the "title" field of the corresponding "Source" resource
    :>json string marginalia:
    :>json string folio: E.g., ``"05v"``
    :>json string sequence:
    :>json string office: the "name" field of the corresponding "Office" resource
    :>json string genre: the "name" field of the corresponding "Genre" resource, provided through the "CantusID" resource
    :>json string position:
    :>json string cantus_id: ``"id"`` field of the corresponding "CantusID" resource
    :>json string feast: ``"name"`` field of the corresponding "Feast" resource (e.g., "Dom. 21 p. Pent.")
    :>json string feast_desc: ``"description"`` of the corresponding "Feast" resource (e.g., "21st Sunday after Pentecost")
    :>json string mode: (will appear in ``"resources"`` after the first version)
    :>json string differentia:
    :>json string finalis: (will appear in ``"resources"`` after the first version)
    :>json string full_text: ``"full_text"`` of the corresponding "CantusID" resource
    :>json string full_text_manuscript: full text as written in the manuscript
    :>json string full_text_simssa: full text for SIMSSA use
    :>json string volpiano: neume information to be rendered with the "Volpiano" font
    :>json string notes:
    :>json string cao_concordances:
    :>json string siglum: the "siglum" field of the corresponding "Source" resource
    :>json string proofreader: ``"display_name"`` of an "Indexer" resource
    :>json string melody_id: (will appear in ``"resources"`` after the first version)
    :>json string resources>source: URL to the containing "Source" resource
    :>json string resources>office: URL to the corresponding "Office"
    :>json string resources>genre: *not provided* (ask the "CantusID" resource)
    :>json string resources>cantus_id: URL to the corresponding "CantusID" resource
    :>json string resources>feast: URL to the corresponding "Feast" resource
    :>json string resources>image_link: URL to an image, or a Web page with an image, of this Chant
    :>json string resources>proofreader: URL to an "Indexer" resource
    :>json string resources>drupal_path: URL to the Chant resource on the Drupal website
    :>json string resources>cantus_id: URL to the corresponding "CantusID" resource

.. _`source resource type`:

Source
^^^^^^

.. http:get:: /(view.source)/(string:id)/

    A "Source" resource is for a collection of folia containing Chants (usually a book). These are
    be avialable at the URLs indicated by ``["view"]["source"]`` and ``["browse"]["source"]``.

    :>json string id: The "id" of this resource.
    :>json string title: Full Manuscript Identification (City, Archive, Shelf-mark)
    :>json string rism: RISM number
    :>json string siglum: Siglum
    :>json string provenance: Provenance
    :>json string provenance_detail: More detail about the provenance
    :>json string date: Date
    :>json string century: Century
    :>json string notation_style: Notation used for the source
    :>json string editors: List of ``"display_name"`` of indexers who edited this manuscript
    :>json string indexers: List of ``"display_name"`` of indexers who entered this manuscript
    :>json string proofreaders: List of ``"display_name"`` of indexers who proofread this manuscript
    :>json string segment: Segment (i.e., source database)
    :>json string source_status: Status of this source
    :>json string source_status_desc: Elaboration of ``"source_status"``---probably never used.
    :>json string summary: Summary
    :>json string liturgical_occasions: Liturgical occasions
    :>json string description: Description
    :>json string indexing_notes: Indexing notes
    :>json string indexing_date: Indexing date
    :>json object resources: Links to other indexer who share the same characteristics.
    :>json string resources>provenance:
    :>json string resources>century:
    :>json string resources>notation_style:
    :>json string resources>editors: List of URLs to Indexer resources.
    :>json string resources>indexer: List of URLs to Indexer resources.
    :>json string resources>proofreaders: List of URLs to Indexer resources.
    :>json string resources>source_status:
    :>json string resources>image_link: Root URL linking to images for the entire source.
    :>json string resources>drupal_path: URL to this Source on the "old" Drupal site.
