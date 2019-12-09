## Background and Related Work
{:#related-work}

Introduce this section
{:.todo}

### Solid

[Solid](cite:cites solid) is a _decentralized Web-based ecosystem_ that decouples _data_ from _applications_.
With Solid, everyone has their own personal _data pod_, in which any kind of data can be stored.
Applications are independent, and require permission from users to access their personal data.
People can decide which actors and applications can read from or write to specific parts of their data.
With this, Solid enables true data ownership for users.

Solid is not an application or tool, but rather a collection of open standards and conventions.
Concretely, _Solid builds upon existing Web standards,_{:.rephrase}, including the [Linked Data stack](cite:cites linkeddata),
[Linked Data Platform (LDP)](cite:cites spec:ldp),[WebID](cite:cites spec:webid), [Web Access Control (WAC)](cite:cites spec:wac)
and [Linked Data Notifications (LDN)](cite:cites spec:ldn).
Linked Data (and [RDF](cite:cites spec:rdf)) is used to give data a universal meaning through URIs,
and to allow data to be linked across multiple data pods.
Solid data pods are assumed to implement the LDP specification to allow read-write RDF through a RESTful Web API.
Solid also allows non-RDF data, such as plain text or images, to be stored in data pods,
but these can only be managed through the usual HTTP methods such as GET and PUT.
Using WebID, everyone has a personal online identifier _using which_{:.rephrase} they can _authenticate_ at any data pod.
With WAC, people can be _authorized_ using their WebID to read, write, append, or control data in files.
Finally, LDN is used to enable pods to communicate with each other,
as the LDN specification defines how messages can be sent between two agents as Linked Data.

Make a simple overview figure of the specs in Solid and how they work together?
{:.todo}

Solid currently uses [Web Access Control (WAC)](cite:cites spec:wac),
which provides access control over _documents_.
For some use cases, where documents are large, and access is only granted to specific parts of the document,
WAC does not offer a sufficient granularity.
For this, [vocabulary extensions of WAC were introduced](cite:cites wac_ppo,wac_ext),
where more fine-grained access can be granted to specific RDF resources or named graphs,
but are not being used in practise.

{:.todo}
Emphasize that Solid *only* does WAC, not the extensions. So only access to full documents.
Also say that even the extensions are not fine-grained enough.

For this work, we assume a decentralized environment as provided by Solid and its used standards
to illustrate our use case.
Furthermore, we will introduce extensions of the used standards to improve query execution
over multiple Solid data pods in a privacy-preserving manner.

### Shapes

shapes (SHACL/ShEx) and [using shapes for Web APIs](cite:cites hypermedia_shapes)
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
and maintain _[data summaries](cite:cites summaries,summaries_heritage)_.
Query engines could then make use of such summaries as an index structure to identify
which sources are relevant for specific queries,
which reduces the range of sources that the engines need to request.

While index-based query execution such as querying over SPARQL endpoints is an active research domain,
alternative querying methods exist, such as _[link traversal-based query execution (LTBQE)](cite:cites linktraversal)_.
LTBQE is based on the follow-your-nose principle, as it exploits the linked nature of Linked Data,
by incorporating crawling into the query execution process.
Due to the scale and the open nature of the Web, LTBQE can lead to infinitely long query execution
and does not guarantee result completeness.

In the context of this work, we will tackle the federation over many Solid data pods.
We extend the data summary approach by introducing a privacy-preserving aggregation approach.

### Approximate Membership Functions

Approximate Membership Functions (AMFs) are probabilistic data structures
that are used to efficiently determine whether or not elements are part of a collection.
Due to AMFs being probabilistic, they may produce false positives, but they always produce true negatives.
Since AMFs are typically much smaller than a full dataset,
they are a valuable method for pre-filtering when querying.

Different AMF techniques exist, such as [_Bloom filters_](cite:cites bloomfilter) and [_Golomb-coded sets_ (GCS)](cite:cites gcsfilter).
A Bloom filter is a bitmap, and a predetermined set of hash functions.
To construct a Bloom filter, each element is run through these hash functions to produce a bit vector,
and all of these vectors are `OR`-ed into the bitmap.
To test the membership of an element, the same hash functions are applied, and their binary membership is tested.
Since Bloom filters are bitmaps, multiple Bloom filters can be merged together efficiently by `OR`-ing them.
GCS are an improvement to Bloom filters where only a single hash function is used,
and the range is a uniformly distributed list instead of a bitmap to enable more efficient compression.
Compared to Bloom filters, GCS achieve better compression rates, but are slower to decompress.

Is a figure needed to illustrate how Bloom filters work?
{:.todo}

AMFs have been used in various parts related to RDF querying,
such as [reducing the number of expensive I/O operations](cite:cites bloomIO) during triple pattern query evaluation,
[improving the performance of join operations](cite:cites bloomjoinslarge),
and [reducing the number of HTTP requests for Triple Pattern Fragments](cite:cites tpf_amf).
In the context of federated querying, the [SPARQL ASK response has been enhanced with Bloom filters to share a summary of the matching results](cite:cites bloomsparqlask), which allows overlaps between different sources to be identified.

In this work, we use Bloom filters to encode encrypted triple components that are available within each source,
and we let aggregators combine them, similar to the [SPARQL ASK approach](cite:cites bloomsparqlask).
Just like with the [Triple Pattern Fragments approach](cite:cites tpf_amf),
clients can then use these Bloom filters to reduce the number of sources they need to fetch.


*[Solid builds upon existing Web standards,]: WebID and WAC aren't really standards though (if standard==W3C Rec)..
*[using which]: with which?