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

Response bodies are JSON formatted, which reduces complexity for client applications written in
JavaScript. For single resources, they will follow this form:

.. sourcecode:: http

    {"resource type":
         {"field": "value",
          "field": "value"
         }
    }

The slight modification required for response bodies to search queries are described in
:ref:`search response bodies`.

Only fields with data will be included in a response, regardless of which fields are requested. In
addition, hyperlinks to corresponding resources are automatically included whenever possible, in the
"resources" member of the response body. Consider the following example:

.. sourcecode:: http

    {"chant":
        {"incipit": "Dixit dominus paralytico",
         "cantus_id": "002288",
         "feast": "Dom. 21 p. Pent.",
         "feast_desc": "21st Sunday after Pentecost"},
         "resources": {"cantus_id": "/chants/cantus_id/002288/",
                       "feast": "/feasts/1616/"
                      }
    }

The response body tells us that this chant occurs during the feast called "Dom. 21 p. Pent." which
is the "21st Sunday after Pentecost," and that more information about this feast is available at the
``/feasts/1616/`` URL. However, some fields do not have corresponding "resources" members, and some
"resources" fields do not have corresponding data fields.

Use the :ref:`OPTIONS <options http method>` method to determine the fields available for a
particular resource.

.. _`search response bodies`:

Response Bodies to Searches
---------------------------

The only difference between the response body for a specific resource, and the response body for a
search query is the the latter is "wrapped in" a "results" element. For example:

.. sourcecode:: http

    {"results": [
        {"chant": {
             "id": "149243",
             "inicipit": "Estote parati similes",
             "cantus_id": "002685"
             }},
        {"chant": {
            "id": "149244",
            "incipit": "Salvator mundi domine qui nos",
            "cantus_id": "830303"
            }},
        {"chant": {
            "id": "149245",
            "incipit": "Estote parati similes",
            "cantus_id": "002685",
            }}
        ]
    }

While this nesting may seem unnecessary, it helps to clarify the type of resources found by a search.
In many cases this clarification is unnecessary, like with a search request to the
``/(browse_feasts)`` URL. However, some search requests may return mixed results---especially
searches on the root URL.

For more information about searching, refer to :ref:`searching`.




































