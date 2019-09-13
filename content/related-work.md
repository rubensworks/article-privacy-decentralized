## Related Work
{:#related-work}

Linked Data, (federated) querying, Solid (WebID/WAC) extensions
{:.todo}

### Solid

[Solid](cite:cites solid) is a _decentralized Web-based ecosystem_ that decouples _data_ from _application_.
With Solid, everyone has their own personal _data pod_, in which any kind of data can be stored.
Applications are independent, and require permission from users to access their personal data.
People can decide which actors and applications can read from or write to specific parts of their data.
With this, Solid enables true data ownership for users.

Solid is not an application or tool, but rather a collection of open standards and conventions.
Concretely, Solid builds upon existing Web standards, including the [Linked Data stack](cite:cites linkeddata),
[Linked Data Platform (LDP)](cite:cites spec:ldp), [WebID](cite:cites spec:webid), and [Web Access Control (WAC)](cite:cites spec:wac).
Linked Data (and [RDF](cite:cites spec:rdf)) is used to give data a universal meaning through URIs,
and to allow data to be linked across multiple data pods.
Solid data pods are assumed to implement the LDP specification to allow read-write RDF through a RESTful Web API.
Solid also allows non-RDF data, such as plain text or images, to be stored in data pods,
but these can only be managed through the usual HTTP methods such as GET and PUT.
Using WebID, everyone has a personal online identifier using which they can _authenticate_ at any data pod.
With WAC, people can be _authorized_ using their WebID to read, write, append, or control data in files.

Make a simple overview figure of the specs in Solid and how they work together?
{:.todo}
