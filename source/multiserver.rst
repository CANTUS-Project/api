.. _`multiserver`:

Multi-Server Interoperabilty
============================

This section describes how Cantus servers may and may not interoperate with other servers.

Because Cantus servers may be required to interoperate with a wide range of other servers, some of
which may provide Cantus-like APIs or (especially) results in JSON, use the "X-Cantus-Version" HTTP
header to know whether a server will fill a client's Cantus expectations.

With Cantus Servers
-------------------

When interoperating with other servers that implement the Cantus API, "id" numbers need not be the
same for resources that appear identical to human users. Needless to say, different servers may
therefore hold otherwise-identical resources with different "id" values and sources with identical
"id" values but otherwise completely different. Moreover, it is not a requirement that every Cantus
server give access to the same collection of resources: some servers may choose to cache metadata
of the entire Cantus network, while others may not, or may cache a different subset.

A link in the ``"resources"`` section of a response body must always point to a server that is
compatible with the Cantus API. Other links should be given in other fields.

With Non-Cantus Servers
-----------------------

Something else?
