## Background & Related Work
{:#background}

We start by presenting the necessary background information with respect to the Solid platform, approximate membership functions, and secure indexes.

<br/>
**Social Linked Data.**
[Solid](cite:cites solid) is a _decentralised Web-based ecosystem_ that decouples _data_ from _applications_.
With Solid, everyone has their own personal _data pod_, in which any kind of data can be stored.
Concretely, Solid makes use of a collection of Web standards and technical specifications, including the [Resource Description Framework (RDF)](cite:cites spec:rdf), the [Linked Data stack](cite:cites linkeddata), the [Linked Data Platform (LDP)](cite:cites spec:ldp), [Linked Data Notifications (LDN)](cite:cites spec:ldn), [WebID](cite:cites spec:webid), and [Web Access Control (WAC)](cite:cites spec:wac).
The RDF data model together with the Linked Data principles are used to give data a universal meaning, and to allow data to be linked across multiple data pods.
Solid data pods implement the LDP specification which caters for RDF read-write operations via RESTful Web Application Programming Interfaces (APIs). The LDN specification is used to enable pods to communicate with each other.
Using WebID, everyone has a personal online identifier that they can use to _authenticate_ against a data pod. While, in turn WAC is used to specify rules that determine if agents and applications are _authorised_ to read, write, append, or control RDF files. The framework described in this paper discusses how Solid could be extended to enable efficient privacy-preserving federated query evaluation over many Solid data pods.

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
[Secure Indexes](cite:cites goh2003secure) are data structures whereby membership can be tested in constant time without revealing any information about the contents of the index. The proposed secure indexes, which are build using pseudo-random functions and bloom filters, can be used to facilitate searches on encrypted data. A secure index scheme is defined as a tuple of four distinct algorithms (_Keygen_, _Trapdoor_, _BuildIndex_, _SearchIndex_) such that:

- _Keygen(s)_ a randomized algorithm that takes as input a security parameter s and outputs a master private key kpriv.
- _Trapdoor(kpriv,w)_ takes as inputs a master private key kpriv and a word w and outputs a trapdoor tw for w.
- _BuildIndex(d,kpriv)_ takes as input a document d and the master key kpriv and outputs the index idxd.
- _SearchIndex(tw, idxd)_ takes as input the trapdoor tw and the index idxd and outputs 1 if the w is possibly in d and 0 otherwise.

The proposed scheme is secure against adaptive chosen keyword attacks, i.e., given two documents containing an unequal number of words and an index it is not possible for an adversary to determine which document is encoded in the index with a probability greater than random guessing.
In this paper we demonstrate how secure indexes can be used to generate privacy-preserving index for Solid data pods.
