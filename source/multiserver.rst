.. _`multiserver`:

Multi-Server Interoperabilty
============================

This section describes how CANTUS servers may and may not interoperate with other servers.

Because CANTUS servers may be required to interoperate with a wide range of other servers, some of
which may provide CANTUS-like APIs or (especially) results in JSON, use the "X-CANTUS-Version" HTTP
header to know whether a server will fill a client's CANTUS expectations.

With CANTUS Servers
-------------------

When interoperating with other servers that implement the CANTUS API, "id" numbers need not be the
same for resources that appear identical to human users. Needless to say, different servers may
therefore hold otherwise-identical resources with different "id" values and sources with identical
"id" values but otherwise completely different. Moreover, it is not a requirement that every CANTUS
server give access to the same collection of resources: some servers may choose to cache metadata
of the entire CANTUS network, while others may not, or may cache a different subset.

With Non-CANTUS Servers
-----------------------

Something else?
