## Related Work
{:#related-work}

shapes (SHACL/ShEx) and [using shapes for Web APIs](cite:cites hypermedia_shapes), Solid (WebID/WAC) extensions, AMF
{:.todo}

### Solid

[Solid](cite:cites solid) is a _decentralized Web-based ecosystem_ that decouples _data_ from _application_.
With Solid, everyone has their own personal _data pod_, in which any kind of data can be stored.
Applications are independent, and require permission from users to access their personal data.
People can decide which actors and applications can read from or write to specific parts of their data.
With this, Solid enables true data ownership for users.

Solid is not an application or tool, but rather a collection of open standards and conventions.
Concretely, Solid builds upon existing Web standards, including the [Linked Data stack](cite:cites linkeddata),
[Linked Data Platform (LDP)](cite:cites spec:ldp), [WebID](cite:cites spec:webid), [Web Access Control (WAC)](cite:cites spec:wac),
and [Linked Data Notifications (LDN)](cite:cites spec:ldn).
Linked Data (and [RDF](cite:cites spec:rdf)) is used to give data a universal meaning through URIs,
and to allow data to be linked across multiple data pods.
Solid data pods are assumed to implement the LDP specification to allow read-write RDF through a RESTful Web API.
Solid also allows non-RDF data, such as plain text or images, to be stored in data pods,
but these can only be managed through the usual HTTP methods such as GET and PUT.
Using WebID, everyone has a personal online identifier using which they can _authenticate_ at any data pod.
With WAC, people can be _authorized_ using their WebID to read, write, append, or control data in files.
Finally, LDN is used to enable pods to communicate with each other,
as the LDN specification defines how messages can be sent between two agents as Linked Data.

Make a simple overview figure of the specs in Solid and how they work together?
{:.todo}

### Querying a Decentralized Web

The standard way for querying RDF on the Web is through [SPARQL endpoints](cite:cites spec:sparqlprot).
Due to the unbounded complexity of [SPARQL queries](cite:cites spec:sparqllang),
and the unlimited number of clients that can send queries,
[SPARQL endpoints are costly to host publicly on the Web](cite:cites sparql_costly).
For this reason, there is increasing interest in
[investigating alternative Web APIs for accessing interfaces, and measuring trade-offs between server and client effort for query evaluation](cite:cites ldf,comunica).

In a truly decentralized Web, data is spread over multiple sources,
which means that there is no single SPARQL endpoint through which all data can be retrieved.
For this, _[federated query processing](cite:cites sparqlfederation)_ is an active area of research
in which techniques are investigated to intelligently delegate the execution of parts of SPARQL queries to specific sources.
Current federation engines support around the order of 10 sources,
which is insufficient for environments that will be produced by decentralization efforts such as Solid.
As Solid gives everyone their own data pod, federated queries over 100 to 1000 sources will become possible,
for example when decentralized social networking applications are being built.
To make such federations scale better, aggregation techniques could be used,
where one or more independent _aggregators_ would continously _crawl_ sources,
and maintain _[data summaries](cite:cites summaries)_.
Query engines could then make use of such summaries as an index structure to identify
which sources are relevant for specific queries,
which reduces the range of sources that the engines need to request.
