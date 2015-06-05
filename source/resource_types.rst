..
    TODO: write full examples, with full headers and response bodies, so people get the idea

.. _`resource types`:

Resource Types
==============

You must discover the URL paths to all resource types with the URLs provided by the server. However,
resource types available in a context will always be named consistently. To help understand what
these points mean, consider the following example.

You submit a request to the root URL:

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

    GET https://abbott.uwaterloo.ca/chants/?="benedicamus_patrem_et_filium" HTTP/1.1

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
``id?`` string must be removed from the URL. Cantus API "id" values may consist of any alphanumeric
character valid in a URL, plus hyphens and underscores.

The following points also apply:

- A resource's "id" MUST remain the same through the resource's lifetime.
- Changing attributes, properties, or data in a resource MUST NOT attempt to change the "id" field.
- A resource's "id" field MAY be prefixed with an identifier indicating which database the holds the
  resources's authoritative copy.
- The same "id" MAY or may not refer to "the same" resource when served by a different deployment of
  a Cantus server application. That is, the Cantus API does not guarantee uniqueness of "id" values
  across deployments.

Simple Resource Types
---------------------

Unlike the types listed in the following section (:ref:`complex resource types`) the resources in
this category will not have fields that cross-reference another resource. Simple resources also tend
to have fewer fields, and are not expected to change often during the lifetime of the
database---perhaps never.

Each of the resources in this category will have two members in the JSON response body: ``"name"``,
which provides a human-readable name for that resource (e.g., "14th century" for a resource of the
"centuries" type); and ``"resources"``, which lists URLs to Chant, Source, or Indexer resources that
are classified as that type. They may also have a ``"description"``.

From the home URL (``/``), all of the following terms may be found in the ``"resources"`` member of
the response body. Resource "id" values are described in :ref:`resource ids` above. You may
discover the valid ``"id"`` values by visiting the generic URL (e.g., by visiting
/**(** *browse_centuries* **)**/ rather than /**(** *browse_centuries* **)**/**(** *id* **)**).

+----------------------+----------------------------+
| Description          | JSON Member                |
+======================+============================+
| Century              | ``"browse_centuries"``     |
+----------------------+----------------------------+
| Feasts               | ``"browse_feasts"``        |
+----------------------+----------------------------+
| Genres               | ``"browse_genres"``        |
+----------------------+----------------------------+
| Notation             | ``"browse_notations"``     |
+----------------------+----------------------------+
| Office               | ``"browse_offices"``       |
+----------------------+----------------------------+
| Portfolio categories | ``"browse_portfolia"``     |
+----------------------+----------------------------+
| Provenance           | ``"browse_provenances"``   |
+----------------------+----------------------------+
| RISM Sigla           | ``"browse_sigla"``         |
+----------------------+----------------------------+
| Segment              | ``"browse_segments"``      |
+----------------------+----------------------------+
| Source status        | ``"browse_source_statii"`` |
+----------------------+----------------------------+

Notes
^^^^^

- Every "genre" also has a "mass_or_office" field in the Solr database.
- Every "feast" MAY also have "date" and "feast_code" fields.

.. _`complex resource types`:

Complex Resource Types
----------------------

The following resource types (CantusID, Chant, Indexer, Source) hold many fields of information,
some of which correspond to a "taxonomy" field given in the previous section.

For the matrices in this section, "Field Name in MySQL" indicates the name of the field in the
Cantus Drupal MySQL database; "Field Name in Drupal" indicates the name of the field as displayed
in the Cantus Drupal user interface; "Field Name in JSON" is the member name of this data as
delivered in the Cantus API; ``"resources"`` indicates whether a hyperlink to more information
about that field's value *may* be included with a JSON response. Refer to the `Request and Response
Bodies <response bodies>`_ section for more information on how to make this bit work right.

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

..
    TODO: do we need a link in "resources" to all chants with the same "incipit" field? I would
    rather not do that, because there isn't an "incipit" resource, and there may be quite a lot of
    results, so it seems more like something you should SEARCH for... even though it would be a
    straight-forward SEARCH that the user interface may be able to offer with a single click.
    Anyway, point is that it's a lot of things, it's not a DB cross-reference, and it's to things
    that aren't sensibly *part of* the Chant itself.

..
    This table is for the developers' reference. It doesn't appear in the rendered documentation.

    TODO: why does "cantus_id" appear twice? Which are we actually using?

    +-----------------------------+-----------------------------------+----------------------+
    | Field Name in MySQL         | Field Name in Drupal              | Field Name in JSON   |
    +=============================+===================================+======================+
    | title                       | Incipit                           | incipit              |
    +-----------------------------+-----------------------------------+----------------------+
    | field_source                | Source                            | source               |
    +-----------------------------+-----------------------------------+----------------------+
    | field_marginalia            | Marginalia                        | marginalia           |
    +-----------------------------+-----------------------------------+----------------------+
    | field_folio                 | Folio                             | folio                |
    +-----------------------------+-----------------------------------+----------------------+
    | field_sequence              | Sequence                          | sequence             |
    +-----------------------------+-----------------------------------+----------------------+
    | field_office                | Office                            | office               |
    +-----------------------------+-----------------------------------+----------------------+
    | field_mc_genre              | Genre                             | genre                |
    +-----------------------------+-----------------------------------+----------------------+
    | field_position              | Position                          | position             |
    +-----------------------------+-----------------------------------+----------------------+
    | field_cantus_id             | Cantus ID                         | cantus_id            |
    +-----------------------------+-----------------------------------+----------------------+
    | field_mc_feast              | Feast                             | feast                |
    +-----------------------------+-----------------------------------+----------------------+
    |                             |                                   | feast_desc           |
    +-----------------------------+-----------------------------------+----------------------+
    | field_mode                  | Mode                              | mode                 |
    +-----------------------------+-----------------------------------+----------------------+
    | field_differentia           | Differentia                       | differentia          |
    +-----------------------------+-----------------------------------+----------------------+
    | field_finalis               | Finalis                           | finalis              |
    +-----------------------------+-----------------------------------+----------------------+
    | body                        | Full text (standardized spelling) | full_text            |
    +-----------------------------+-----------------------------------+----------------------+
    | field_full_text_ms          | Full text (MS spelling)           | full_text_manuscript |
    +-----------------------------+-----------------------------------+----------------------+
    | field_simssa_fulltext       | Full text (SIMSSA use)            | full_text_simssa     |
    +-----------------------------+-----------------------------------+----------------------+
    | field_volpiano              | Volpiano                          | volpiano             |
    +-----------------------------+-----------------------------------+----------------------+
    | field_image_link_chant      | Image link                        |                      |
    +-----------------------------+-----------------------------------+----------------------+
    | field_notes                 | Indexing notes                    | notes                |
    +-----------------------------+-----------------------------------+----------------------+
    | field_cao_concordances      | CAO Concordances                  | cao_concordances     |
    +-----------------------------+-----------------------------------+----------------------+
    | field_siglum_chant          | Siglum                            | siglum               |
    +-----------------------------+-----------------------------------+----------------------+
    | field_proofread_by          | Proofread by                      | proofreader          |
    +-----------------------------+-----------------------------------+----------------------+
    | path                        | URL path settings                 |                      |
    |                             |                                   |                      |
    +-----------------------------+-----------------------------------+----------------------+
    | ``field_nid_old_``          | NID (old)                         |                      |
    +-----------------------------+-----------------------------------+----------------------+
    | ``field_user_old_``         | User (old)                        |                      |
    +-----------------------------+-----------------------------------+----------------------+
    | field_fulltext_proofread    | Fulltext proofread                |                      |
    +-----------------------------+-----------------------------------+----------------------+
    | field_ms_fulltext_proofread | MS Fulltext proofread             |                      |
    +-----------------------------+-----------------------------------+----------------------+
    | field_volpiano_proofread    | Volpiano proofread                |                      |
    +-----------------------------+-----------------------------------+----------------------+
    | field_cantus_id_temp        | Cantus ID (temp)                  | cantus_id            |
    +-----------------------------+-----------------------------------+----------------------+
    | field_melody_id             | Melody ID                         | melody_id            |
    +-----------------------------+-----------------------------------+----------------------+

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

..
    This table is for the developers' reference. It doesn't appear in the rendered documentation.

    +----------------------------+--------------------------------+----------------------+------------------+-----------------------------------------------------------+
    | Field Name in MySQL        | Field Name in Drupal           | Field Name in JSON   | ``"resources"``? | Comments                                                  |
    +============================+================================+======================+==================+===========================================================+
    | title                      | Full Manuscript Identification | title                |                  |                                                           |
    |                            | (City, Archive, Shelf-mark)    |                      |                  |                                                           |
    +----------------------------+--------------------------------+----------------------+------------------+-----------------------------------------------------------+
    | field_rism                 | RISM                           | rism                 |                  |                                                           |
    +----------------------------+--------------------------------+----------------------+------------------+-----------------------------------------------------------+
    | field_siglum               | Siglum                         | siglum               |                  |                                                           |
    +----------------------------+--------------------------------+----------------------+------------------+-----------------------------------------------------------+
    | field_provenance_tax       | Provenance                     | provenance           | yes              |                                                           |
    +----------------------------+--------------------------------+----------------------+------------------+-----------------------------------------------------------+
    | field_provenance           | Provenance notes               | provenance_detail    |                  |                                                           |
    +----------------------------+--------------------------------+----------------------+------------------+-----------------------------------------------------------+
    | field_date                 | Date                           | date                 |                  | e.g., "1300s"                                             |
    +----------------------------+--------------------------------+----------------------+------------------+-----------------------------------------------------------+
    | field_century              | Century                        | century              | yes              |                                                           |
    +----------------------------+--------------------------------+----------------------+------------------+-----------------------------------------------------------+
    | field_notation             | Notation                       | notation_style       | yes              |                                                           |
    +----------------------------+--------------------------------+----------------------+------------------+-----------------------------------------------------------+
    | field_editors              | Editors                        | editors              | yes              | list of "title" of Indexers who edited the manuscript; in |
    |                            |                                |                      |                  | ``"resources"`` will be a list of URLs                    |
    +----------------------------+--------------------------------+----------------------+------------------+-----------------------------------------------------------+
    | field_indexer              | Indexer                        | indexers             | yes              | list of "title" of Indexers who entered the manuscript;   |
    |                            |                                |                      |                  | in ``"resources"`` will be a list of URLs                 |
    +----------------------------+--------------------------------+----------------------+------------------+-----------------------------------------------------------+
    | field_proofreader          | Proofreader                    | proofreaders         | yes              | in ``"resources"`` will be a list of URLs                 |
    +----------------------------+--------------------------------+----------------------+------------------+-----------------------------------------------------------+
    | field_segment              | Segment                        | segment              | yes              |                                                           |
    +----------------------------+--------------------------------+----------------------+------------------+-----------------------------------------------------------+
    | field_source_status        | Source status                  | source_status_desc   |                  | textual elaboration of "source_status"                    |
    +----------------------------+--------------------------------+----------------------+------------------+-----------------------------------------------------------+
    | field_source_status_tax    | Source status                  | source_status        | yes              |                                                           |
    +----------------------------+--------------------------------+----------------------+------------------+-----------------------------------------------------------+
    | field_summary              | Summary                        | summary              |                  |                                                           |
    +----------------------------+--------------------------------+----------------------+------------------+-----------------------------------------------------------+
    | field_liturgical_occasions | Liturgical occasions           | liturgical_occasions |                  |                                                           |
    +----------------------------+--------------------------------+----------------------+------------------+-----------------------------------------------------------+
    | body                       | Description                    | description          |                  |                                                           |
    +----------------------------+--------------------------------+----------------------+------------------+-----------------------------------------------------------+
    | field_bibliography         | Selected bibliography          |                      |                  | ignored (Drupal seems to ignore it)                       |
    +----------------------------+--------------------------------+----------------------+------------------+-----------------------------------------------------------+
    | field_image_link           | Image link                     |                      | image_link       | will **only** appear in ``"resources"`` as the root URL   |
    |                            |                                |                      |                  | for images for the entire Source                          |
    +----------------------------+--------------------------------+----------------------+------------------+-----------------------------------------------------------+
    | field_indexing_notes       | Indexing notes                 | indexing_notes       |                  |                                                           |
    +----------------------------+--------------------------------+----------------------+------------------+-----------------------------------------------------------+
    | field_indexing_date        | Indexing date                  | indexing_date        |                  |                                                           |
    +----------------------------+--------------------------------+----------------------+------------------+-----------------------------------------------------------+
    | field_indexed_by           | Indexing notes (old)           |                      |                  | ignored ("old")                                           |
    +----------------------------+--------------------------------+----------------------+------------------+-----------------------------------------------------------+
    | path                       | URL path settings              |                      | drupal_path      | will **only** appear in ``"resources"`` as the URI of the |
    |                            |                                |                      |                  | corresponding source in the Drupal website                |
    +----------------------------+--------------------------------+----------------------+------------------+-----------------------------------------------------------+

.. _`indexer resource type`:

Indexer
^^^^^^^

.. http:get:: /(view.indexer)/(string:id)/

    An "Indexer" corresponds to an agent who has entered or modified data in the Cantus Database
    (usually a human). These are avialable at the URLs ``["view"]["indexer"]`` and
    ``["browse"]["indexer"]``.

    :>json string id: The "id" of this resource.
    :>json string display_name: The indexer's name, as displayed.
    :>json string given_name: The indexer's given name.
    :>json string family_name: The indexer's family name.
    :>json string institution: The indexer's associated university or research institution.
    :>json string city: The city where the indexer lives.
    :>json string country: The country where the indexer lives.
    :>json object resources: Links to other indexer who share the same characteristics.
    :>json string resources>institution:
    :>json string resources>city:
    :>json string resources>country:

..
    This table is for the developers' reference. It doesn't appear in the rendered documentation.

    +---------------------------+----------------------+--------------------+------------------+-----------------------------------------------------------+
    | Field Name in MySQL       | Field Name in Drupal | Field Name in JSON | ``"resources"``? | Comments                                                  |
    +===========================+======================+====================+==================+===========================================================+
    | title                     | Name                 | display_name       |                  |                                                           |
    +---------------------------+----------------------+--------------------+------------------+-----------------------------------------------------------+
    | field_first_name          | First name           | given_name         |                  |                                                           |
    +---------------------------+----------------------+--------------------+------------------+-----------------------------------------------------------+
    | field_family_name         | Family name          | family_name        |                  |                                                           |
    +---------------------------+----------------------+--------------------+------------------+-----------------------------------------------------------+
    | field_indexer_institution | Institution          | institution        | yes              |                                                           |
    +---------------------------+----------------------+--------------------+------------------+-----------------------------------------------------------+
    | field_indexer_city        | City                 | city               | yes              |                                                           |
    +---------------------------+----------------------+--------------------+------------------+-----------------------------------------------------------+
    | field_indexer_country     | Country              | country            | yes              |                                                           |
    +---------------------------+----------------------+--------------------+------------------+-----------------------------------------------------------+
    | path                      | URL path settings    |                    | drupal_path      | will **only** appear in ``"resources"`` as the URI of the |
    |                           |                      |                    |                  | corresponding source in the Drupal website                |
    +---------------------------+----------------------+--------------------+------------------+-----------------------------------------------------------+
