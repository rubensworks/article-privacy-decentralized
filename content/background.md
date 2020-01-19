## Background
{:#background}

Before introducing our privacy-preserving aggregation framework, we first present the necessary background information in relation to the Solid platform, decentralized query execution,  approximate membership functions, and the W3C Shapes Constraint Language (SHACL).

### Solid

[Solid](cite:cites solid) is a _decentralized Web-based ecosystem_ that decouples _data_ from _applications_.
With Solid, everyone has their own personal _data pod_, in which any kind of data can be stored.
Applications are decoupled from the data, and require permission from users to access their personal data store.
Individuals can decide which actors and applications can read from or write to specific parts of their data store.
With this, Solid enables true data ownership for users.

Solid is not an application or tool, but rather a collection of open standards and conventions.
Concretely, Solid makes use of a collection of Web standards, including the [Linked Data stack](cite:cites linkeddata),
the [Linked Data Platform (LDP)](cite:cites spec:ldp), [Linked Data Notifications (LDN)](cite:cites spec:ldn), [WebID](cite:cites spec:webid), [Web Access Control (WAC)](cite:cites spec:wac). The [Resource Description Framework (RDF) data model](cite:cites spec:rdf) together with the Linked Data principles are used to give data a universal meaning through URIs, and to allow data to be linked across multiple data pods.
Solid data pods are assumed to implement the LDP specification to allow read-write RDF through a RESTful Web API.
Solid also allows non-RDF data, such as plain text or images, to be stored in data pods,
but these can only be managed through the usual HTTP methods such as GET and PUT. 
The LDN specification, which defines how messages can be sent between two agents as Linked Data, is used to enable pods to communicate with each other.
Finally using WebID, everyone has a personal online identifier which they can use to _authenticate_ against a data pod, while WAC, is used to specify if they are _authorized_ to read, write, append, or control RDF files.

<!-- Make a simple overview figure of the specs in Solid and how they work together?
{:.todo} -->


### Federated Query Processing

The standard way to query RDF on the Web is through [SPARQL endpoints](cite:cites spec:sparqlprot).
Due to the unbounded complexity of [SPARQL queries](cite:cites spec:sparqllang),
and the unlimited number of clients that can send queries,
[SPARQL endpoints are costly to host publicly on the Web](cite:cites sparql_costly).
For this reason, there is increasing interest in
[investigating alternative Web APIs for accessing interfaces, and measuring trade-offs between server and client effort for query evaluation](cite:cites ldf,comunica).

In a truly decentralized Web, data is spread over multiple sources,
which means that there is no single SPARQL endpoint through which all data can be retrieved.
For this, _[federated query processing](cite:cites sparqlfederation)_ is an active area of research
in which techniques are investigated to intelligently delegate the execution of parts of SPARQL queries to specific sources.
Current federation engines usually provide support for around 10 different sources,
which is insufficient for environments that need to query many data sources.
For instance, Solid gives everyone their own data pod, thus in order to support decentralised social networking applications there is a need for federated queries over 100s of individual data sources.
To make such federations scale better, aggregation techniques,
whereby one or more independent _aggregators_ continously _crawl_ sources,
and maintain _[data summaries](cite:cites summaries,summaries_heritage)_, could be used to reduce the number of sources that need to be consulted.
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

<!-- Is a figure needed to illustrate how Bloom filters work?
{:.todo} -->

AMFs have been used in various parts related to RDF querying,
such as [reducing the number of expensive I/O operations](cite:cites bloomIO) during triple pattern query evaluation,
[improving the performance of join operations](cite:cites bloomjoinslarge),
and [reducing the number of HTTP requests for Triple Pattern Fragments](cite:cites tpf_amf).
In the context of federated querying, the [SPARQL ASK response has been enhanced with Bloom filters to share a summary of the matching results](cite:cites bloomsparqlask), which allows overlaps between different sources to be identified.

In this work, we use Bloom filters to encode encrypted triple components that are available within each source,
and we let aggregators combine them, similar to the [SPARQL ASK approach](cite:cites bloomsparqlask).
Just like with the [Triple Pattern Fragments approach](cite:cites tpf_amf),
clients can then use these Bloom filters to reduce the number of sources they need to fetch.

### Shapes

SIMON shapes (SHACL/ShEx) and [using shapes for Web APIs](cite:cites hypermedia_shapes)
{:.todo}