## Background and Related Work
{:#background}

The objective of this section is to present the necessary background and related work on the Solid platform, federated query processing, approximate membership functions, and access control.

<!-- the W3C Shapes Constraint Language (SHACL). -->

### Social Linked Data

[Solid](cite:cites solid) is a _decentralised Web-based ecosystem_ that decouples _data_ from _applications_.
With Solid, everyone has their own personal _data pod_, in which any kind of data can be stored.
Applications are decoupled from the data, and require permission from individuals in order to access their personal data store.
Pod owners can decide which agents and applications can read from or write to specific parts of their data store.

Solid is not an application or tool, but rather a collection of open standards and conventions.
Concretely, Solid makes use of a collection of Web standards and technical specifications, including the [Resource Description Framework (RDF)](cite:cites spec:rdf), the [Linked Data stack](cite:cites linkeddata), the [Linked Data Platform (LDP)](cite:cites spec:ldp), [Linked Data Notifications (LDN)](cite:cites spec:ldn), [WebID](cite:cites spec:webid), and [Web Access Control (WAC)](cite:cites spec:wac).
The RDF data model together with the Linked Data principles are used to give data a universal meaning through Uniform Resource Identifiers (URIs), and to allow data to be linked across multiple data pods.
Solid data pods implement the LDP specification which caters for RDF read-write operations via RESTful Web Application Programming Interfaces (APIs).
Solid also allows non-RDF data, such as plain text or images, to be stored in data pods,
however these can only be managed through the usual HyperText Transfer Protocol (HTTP) methods such as GET and PUT.
The LDN specification, which defines how messages can be sent between two agents as Linked Data, is used to enable pods to communicate with each other.
Finally using WebID, everyone has a personal online identifier that they can use to _authenticate_ against a data pod. While, in turn WAC is used to specify rules that determine if agents and applications are _authorised_ to read, write, append, or control RDF files.

The framework described in this paper discusses how Solid could be extended to enable efficient privacy-preserving federated query evaluation over many Solid data pods.

### Federated Query Processing

In a truly decentralised Web, data is spread over multiple sources,
which means that there is no single [endpoint](cite:cites spec:sparqlprot) through which all data can be retrieved.
For this, [_federated query processing_](cite:cites sparqlfederation) is an active area of research
in which techniques are investigated to intelligently delegate the execution of parts of a [SPARQL query](cite:cites spec:sparqllang) to specific sources.
In order to enable federations over many sources to scale more efficiently aggregation techniques whereby one or more independent _aggregators_ continously _crawl_ sources,
and maintain [_data summaries_](cite:cites summaries,summaries_heritage), could be used to reduce the number of sources that need to be consulted.
Query engines could use these summaries as an index structure that enables them to identify the sources that are needed to answer specific queries,
which reduces the range of sources that need to be queried.

In the context of this work, we extend existing query optimisation approaches by introducing an framework for efficient privacy-preserving federated query execution.

### Approximate Membership Functions

Approximate Membership Functions (AMFs) are probabilistic data structures used to efficiently determine whether or not elements are part of a collection.
Given that AMFs are probabilistic, they may produce false positives, but they always produce true negatives.
Since AMFs are typically much smaller than a full dataset,
they are a valuable method for pre-filtering when querying.
[_Bloom filters_](cite:cites bloomfilter) are one example of an AMF technique.
A Bloom filter consists of a bitmap, and a predetermined set of hash functions.
To construct a Bloom filter, each element is run through these hash functions to produce a bit vector,
and all of these vectors are `OR`-ed into the bitmap.
To test the membership of an element, the same hash functions are applied, and their binary membership is tested.
Since Bloom filters are bitmaps, multiple Bloom filters can be merged together efficiently by `OR`-ing them.
AMFs have been used in various of RDF querying scenarios,
such as [reducing the number of expensive I/O operations](cite:cites bloomIO) during triple pattern query evaluation,
[improving the performance of join operations](cite:cites bloomjoinslarge),
and [reducing the number of HTTP requests for Triple Pattern Fragments](cite:cites tpf_amf).
In the context of federated querying, the [SPARQL ASK response has been enhanced with Bloom filters to share a summary of the matching results](cite:cites bloomsparqlask), which allows overlaps between different sources to be identified.

In the proposed efficient privacy-preserving federated query execution framework, Bloom filters are used to encode encrypted triple components that are available within each source, and aggregators are responsible for aggregating privacy-preserving summaries for several Solid data pods.

### Access Control

The decision to grant or deny access is based on two distinct processes, authentication and authorisation. Authentication involves the verification of identity (you are
who you say you are). Whereas, authorisation is the process of granting or denying access to system resources based on identity.
From an authentication perspective, [Web Identity and Discovery (WebID)](cite:cites spec:webid) is a HTTP URI used to uniquely identify and [authenticate an agent (i.e. person, company, organization, or other entity)](cite:cites Sambra2014). A description of the agent is provided in an RDF document, known as a dereferencable WebID profile. The agent places the URI for their WebID profile document in the Subject Alternative Names field of their certificate. Once the certificate has been generated the user adds the public key details to their [WebID profile document](cite:cites Inkster2014). A service wishing to authenticate the user, needs to verify that the public key of the certificate it receives matches the public key specified in the [WebID profile](cite:cites Hollenbach2009). [OpenID Connect](cite:cites OpenIDConnect) is an industry standard authentication protocol, which enables applications to delegate responsibility for authentication to third party identity providers. One of the primary benefits being the ability to connect to multiple sites using the same login credentials. A comprehensive security analysis of the protocol is provided by [Fett et al.](cite:cites fett2017web).
[Web Access Control (WAC)](cite:cites spec:wac) is an RDF vocabulary and an access control framework, which demonstrates how together WebID and access control policies specified using the WAC vocabulary, can be used to enforce distributed access control. Essentially WAC authorisations grant agents, access to resources. [Villata et al.](cite:cites Villata2011) and [Sacco and Passant](cite:cites Sacco2011b) extend the WAC vocabulary to cater for context based access control policies and privacy preferences respectively. Alternatively, [Encryption-Based Access Control](cite:cites fernandez2017self) involves encrypting RDF fragments (i.e. subjects, predicates, objects, graphs or some combination thereof) with an encryption key, such that only those that have the key are permitted to access the data, thus serving as both an authentication and an authorisation mechanism. Existing proposals involve using [symmetric encryption](cite:cites kasten2013towards), [public-key encryption](cite:cites giereth2005partial), or [functional encryption](cite:cites fernandez2017self) to generate RDF ciphers.

{:.todo}
WebID-TLS is still mentioned above, even though it is deprecated.
It should be removed in favor of WebID-OIDC.

In this paper, encryption mechanisms are used to create privacy-preserving aggregations, whereas access control policies are used to restrict access to data at query time. Here, we assume that there is a one to one mapping between encryption keys and authorisations.
