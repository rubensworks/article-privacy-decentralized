## Background and Related Work
{:#background}

We start by presenting background and related work on the Solid platform, federated query processing, approximate membership functions, and access control.

<br/>
**Social Linked Data.**
[Solid](cite:cites solid) is a _decentralised Web-based ecosystem_ that decouples _data_ from _applications_.
With Solid, everyone has their own personal _data pod_, in which any kind of data can be stored.
Concretely, Solid makes use of a collection of Web standards and technical specifications, including the [Resource Description Framework (RDF)](cite:cites spec:rdf), the [Linked Data stack](cite:cites linkeddata), the [Linked Data Platform (LDP)](cite:cites spec:ldp), [Linked Data Notifications (LDN)](cite:cites spec:ldn), [WebID](cite:cites spec:webid), and [Web Access Control (WAC)](cite:cites spec:wac).
The RDF data model together with the Linked Data principles are used to give data a universal meaning, and to allow data to be linked across multiple data pods.
Solid data pods implement the LDP specification which caters for RDF read-write operations via RESTful Web Application Programming Interfaces (APIs). The LDN specification is used to enable pods to communicate with each other.
Using WebID, everyone has a personal online identifier that they can use to _authenticate_ against a data pod. While, in turn WAC is used to specify rules that determine if agents and applications are _authorised_ to read, write, append, or control RDF files. The framework described in this paper discusses how Solid could be extended to enable efficient privacy-preserving federated query evaluation over many Solid data pods.

<br/>
**Federated Query Processing.**
In a truly decentralised Web, data is spread over multiple sources,
which means that there is no single endpoint through which all data can be retrieved.
For this, federated query processing is an active area of research
in which techniques are investigated to intelligently delegate the execution of parts of a SPARQL query to specific sources.
In order to enable federations over many sources to scale more efficiently aggregation techniques whereby one or more independent _aggregators_ continously _crawl_ sources (i.e., dereference Uniform Resource Identifiers)  and _maintain_ [_data summaries_](cite:cites summaries,summaries_heritage), could be used to reduce the number of sources that need to be consulted.
Query engines could use these summaries as an index structure that enables them to identify the sources that are needed to answer specific queries,
which reduces the range of sources that need to be queried.
In the context of this work, we extend existing query optimisation approaches by introducing an framework for efficient privacy-preserving federated query execution.

<br/>
**Approximate Membership Functions.**
Approximate Membership Functions (AMFs) are probabilistic data structures used to efficiently determine whether or not elements are part of a collection. Given that AMFs are probabilistic, they may produce false positives, but they always produce true negatives. Since AMFs are typically much smaller than a full dataset,
they are a valuable method for pre-filtering when querying.
[_Bloom filters_](cite:cites bloomfilter) are one example of an AMF technique.
A Bloom filter consists of a bitmap, and a predetermined set of hash functions.
AMFs have been used in various of RDF querying scenarios,
such as [reducing the number of expensive I/O operations](cite:cites bloomIO) during triple pattern query evaluation,
[improving the performance of join operations](cite:cites bloomjoinslarge),
and [reducing the number of HTTP requests for Triple Pattern Fragments](cite:cites tpf_amf).
In the context of federated querying, the [SPARQL ASK response has been enhanced with Bloom filters to share a summary of the matching results](cite:cites bloomsparqlask), which allows overlaps between different sources to be identified.
Herein, we use Bloom filters to encode encrypted triple components that are available within each source, and aggregators are responsible for aggregating privacy-preserving summaries for several Solid data pods.

<br/>
**Secure Indexes.**
[Secure Indexes](cite:cites goh2003secure) are data structures whereby membership can be tested in constant time without revealing any information about the contents of the index. The proposed secure indexes, which are build using pseudo-random functions and bloom filters, can be used to facilitate searches on encrypted data. The proposed scheme is secure against adaptive chosen keyword attacks, i.e., given two documents containing an unequal number of words and an index it is not possible for an adversary to determine which document is encoded in the index with a probability greater than random guessing. [Tekin and Sahin](cite:cites tekin2017implementation) further demonstrate how counting bloom filters can be used to cater for dynamic updates. In this paper we demonstrate how secure indexes can be used to generate privacy-preserving index for data contained in data pods.

<br/>
**Access Control.**
[Web Access Control (WAC)](cite:cites spec:wac) is an RDF vocabulary and an access control framework, which demonstrates how together WebID and access control policies specified using the WAC vocabulary, can be used to enforce distributed access control. [Villata et al.](cite:cites Villata2011) and [Sacco and Passant](cite:cites Sacco2011b) extend the WAC vocabulary to cater for context based access control policies and privacy preferences respectively. Alternatively, [Encryption-Based Access Control](cite:cites fernandez2017self) involves encrypting RDF fragments (i.e. subjects, predicates, objects, graphs or some combination thereof) with an encryption key, such that only those that have the key are permitted to access the data, thus serving as both an authentication and an authorisation mechanism. Existing proposals involve using [symmetric encryption](cite:cites kasten2013towards), [public-key encryption](cite:cites giereth2005partial), or [functional encryption](cite:cites fernandez2017self) to generate RDF ciphers.
In this paper, encryption mechanisms are used to create privacy-preserving aggregations, whereas access control policies are used to restrict access to data at query time.
