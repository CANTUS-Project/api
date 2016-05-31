Request and Response Bodies
===========================

Request Bodies
--------------

In the first version of the Cantus API, the only type of request that has a request body is a
search. Refer to :ref:`searching` for more information. Request bodies will be more widely
used in subsequent versions of the Cantus API, especially for updating and uploading new records.

.. _`response bodies`:

Response Bodies
---------------

.. note::
    The response body examples below are not intended to represent actual data---always check the
    "resources" URLs provided by the server, rather than generating them on the client side.

.. note::
    Every resource has will have at least two fields: ``id`` and ``type``.

Response bodies are JSON formatted, which reduces complexity for client applications written in
JavaScript. They always have the same form:

.. sourcecode:: http

    {
        "id_value": {
            "id": "id_value",
            "type": "example_resource",
            "fieldA": "value",
            "fieldB": "value"
        },
        "resources": {
            "id_value": {
                "fieldA": "value URL",
                "fieldA_id": "someID",
                "fieldB": "value URL",
                "fieldB_id": "otherID"
            },
        },
        "sort_order": [
            "id_value"
        ]
    }

Only fields with data will be included in a response, regardless of which fields are requested. In
addition, hyperlinks to corresponding resources are automatically included whenever possible, in the
"resources" member of the response body. Consider the following example:

.. sourcecode:: http

    {
        "361434": {
            "id": "361434",
            "type": "chant",
            "incipit": "Dixit dominus paralytico",
            "cantus_id": "002288",
            "feast": "Dom. 21 p. Pent.",
            "feast_desc": "21st Sunday after Pentecost"
        },
        "resources": {
            "361434": {
                "self": "/chants/361434cantus/",
                "feast": "/feasts/1616/",
                "feast_id": "1616"
            }
        },
        "sort_order": [
            "361434"
        ]
    }

The response body tells us that this chant occurs during the feast called "Dom. 21 p. Pent." which
is the "21st Sunday after Pentecost," and that more information about this feast is available at the
``/feasts/1616/`` URL. In general, some fields may not have corresponding "resources" members, and
some "resources" fields may not have corresponding data fields.

Use the :ref:`HEAD <head http method>` method to fetch the :http:header:`X-Cantus-Fields` header to
determine which fields are available for a particular resource.

Sort Order
^^^^^^^^^^

The ``'sort_order'`` member MUST be included in a response body, containing a list of the resource
IDs in that response body.  This list is the only means to provide a response to the
:http:header:`X-Cantus-Sort` header, or to provide relevance-based sorting of search results. User
agents MAY ignore the ``'sort_order'`` if they wish.

Note in the following section, for example, that the ``'sort-order'`` indicates the two resources
should be displayed in the opposite order they appear in the JSON response itself.

.. _`search response bodies`:

Response Bodies to Searches
---------------------------

The only difference between the response body for a specific resource, for a collection of resources
at the "browse" URL, and the result of a search, is that search results may include resources of
more than one type. Thus the "type" member is important in this case to know how results should be
displayed. For example:

.. sourcecode:: http

    {
        "361434": {
            "id": "361434",
            "type": "chant",
            "incipit": "Dixit dominus paralytico",
            "cantus_id": "002288",
            "feast": "Dom. 21 p. Pent.",
            "feast_desc": "21st Sunday after Pentecost"
        },
        "123673": {
            "id": "123673",
            "type": "source",
            "title": "München, Franziskanerkloster St. Anna - Bibliothek, 12o Cmm 1",
            "provenance": "Italy"
        },
        "resources": {
            "361434": {
                "self": "/chants/361434cantus/",
                "feast": "/feasts/1616/",
                "feast_id": "1616"
            },
            "123673": {
                "self": "/books/123673/",
                "provenance": "/provenances/3608/",
                "provenance_id": "3608"
            }
        },
        "sort_order": [
            "123673",
            "361434"
        ]
    }

For more information about searching, refer to :ref:`searching`.
