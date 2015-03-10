.. _`response bodies`:

Request and Response Bodies
===========================

For as long as we're read-only, the request bodies will probably be empty, except possibly for
searching. I'm debating with myself over whether it's more proper to hold a query in the URI or in
the request body. The "pro-URI" points include that everyone else seems to be doing it. The
"pro-body" points include that (I think?) a URI should always return "the same" document, and I'm
not sure that's true if we use query things in the URI; if we include query parameters in the URI,
it means very long URIs; and also that query parameters in the URI cannot be encrypted even if we
start to use SSL.

.. _`fancy json example`:

In terms of HATEOAS, all related resources will be accessible in the JSON response body from the
``"resources"`` member. For example, part of a "chant" resource may be:

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
    }

This would mean, for example, that this chant occurs during the feast called "Dom. 21 p. Pent."
(being the 21st Sunday after Pentecost), and that you may view all the chants associated with this
feast at /feasts/1616/.

If a member with the same name appears as a child of both the ``"this_record"`` and the
``"resources"`` members, they will hold corresponding values as in the example above. The hyperlink
in ``"resources"`` will usually produce a list of all the other records of the same type that have
the same value in that field (e.g., in the previous example, the hyperlink would be to a list of all
the chants that happen during the 21st Sunday after Pentecost). However, the link may point to any
resource that provides more information about that value for the field.

Some members of ``"this_record"`` may not have corresponding ``"resources"`` members, and some
``"resources"`` members may not have corresponding ``"this_record"`` members.

Notes:
    - write a thing about how to tell a client when they request a field but it can't be delivered
        - also: that OPTIONS will indicate whether a field is available for a particular resource

.. _`json and xml`:

A Note about JSON and XML
-------------------------

For reasons of standards-compliance, the request and response bodies will be in "application/xml"
or "text/xml" formats by default. However, most examples in this document, are provided in the
"application/json" data type. Both formats are always acceptable for the server, and it will always
respond with "application/json" whenver requested.

The XML and JSON encodings of a resource are easy to guess, and the differences are few. For
example, this is the XML encoding of the :ref:`JSON example <fancy json example>` above:

.. sourcecode:: http

        <chant>
            <incipit>Dixit dominus paralytico</incipit>
            <cantus_id>002288</cantus_id>
            <feast>Dom. 21 p. Pent.</feast>
            <feast_desc>21st Sunday after Pentecost</feast_desc>
            <resources>
                <cantus_id>/chants/cantus_id/002288/</cantus_id>
                <feast>/feasts/1616/</feast>
            </resources>
        </chant>

For this reason, clients that only wish to receive JSON data must always submit an "Accept" header,
though we recommend it all clients submit "Accept" headers for completeness.
