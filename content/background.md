## Background and Related Work
{:#background}

Before describing our motivating scenario and introducing our privacy-preserving federation framework, we first present the necessary background information in relation to the Solid platform, federated query processing, approximate membership functions, authentication and access control.

<!-- the W3C Shapes Constraint Language (SHACL). -->

### Solid

[Solid](cite:cites solid) is a _decentralized Web-based ecosystem_ that decouples _data_ from _applications_.
With Solid, everyone has their own personal _data pod_, in which any kind of data can be stored.
Applications are decoupled from the data, and require permission from users to access their personal data store.
Individuals can decide which actors and applications can read from or write to specific parts of their data store.

Solid is not an application or tool, but rather a collection of open standards and conventions.
Concretely, Solid makes use of a collection of Web standards, including the [Resource Description Framework](cite:cites spec:rdf), the [Linked Data stack](cite:cites linkeddata), the [Linked Data Platform (LDP)](cite:cites spec:ldp), [Linked Data Notifications (LDN)](cite:cites spec:ldn), [WebID](cite:cites spec:webid), and [Web Access Control (WAC)](cite:cites spec:wac).
The RDF data model together with the Linked Data principles are used to give data a universal meaning through URIs, and to allow data to be linked across multiple data pods.
Solid data pods are assumed to implement the LDP specification to allow read-write RDF through a RESTful Web API.
Solid also allows non-RDF data, such as plain text or images, to be stored in data pods,
but these can only be managed through the usual HTTP methods such as GET and PUT.
The LDN specification, which defines how messages can be sent between two agents as Linked Data, is used to enable pods to communicate with each other.
Finally using WebID, everyone has a personal online identifier which they can use to _authenticate_ against a data pod, while WAC, is used to specify if they are _authorized_ to read, write, append, or control RDF files.

<!-- Make a simple overview figure of the specs in Solid and how they work together?
{:.todo} -->


### Federated Query Processing

In a truly decentralized Web, data is spread over multiple sources,
which means that there is no single [endpoint](cite:cites spec:sparqlprot) through which all data can be retrieved.
For this, _[federated query processing](cite:cites sparqlfederation)_ is an active area of research
in which techniques are investigated to intelligently delegate the execution of parts of [SPARQL queries](cite:cites spec:sparqllang) to specific sources.
To make such federations over many sources scale better, aggregation techniques,
whereby one or more independent _aggregators_ continously _crawl_ sources,
and maintain _[data summaries](cite:cites summaries,summaries_heritage)_, could be used to reduce the number of sources that need to be consulted.
Query engines could then make use of such summaries as an index structure to identify
which sources are relevant for specific queries,
which reduces the range of sources that the query engines need to interact with.

In the context of this work, we will tackle the federation over many Solid data pods.
We extend the data summary approach by introducing a privacy-preserving aggregation approach.

### Approximate Membership Functions

Approximate Membership Functions (AMFs) are probabilistic data structures used to efficiently determine whether or not elements are part of a collection.
Given that AMFs are probabilistic, they may produce false positives, but they always produce true negatives.
Since AMFs are typically much smaller than a full dataset,
they are a valuable method for pre-filtering when querying.

[_Bloom filters_](cite:cites bloomfilter) are one example of an AMF technique.
A Bloom filter is a bitmap, and a predetermined set of hash functions.
To construct a Bloom filter, each element is run through these hash functions to produce a bit vector,
and all of these vectors are `OR`-ed into the bitmap.
To test the membership of an element, the same hash functions are applied, and their binary membership is tested.
Since Bloom filters are bitmaps, multiple Bloom filters can be merged together efficiently by `OR`-ing them.

AMFs have been used in various of RDF querying scenarios,
such as [reducing the number of expensive I/O operations](cite:cites bloomIO) during triple pattern query evaluation,
[improving the performance of join operations](cite:cites bloomjoinslarge),
and [reducing the number of HTTP requests for Triple Pattern Fragments](cite:cites tpf_amf).
In the context of federated querying, the [SPARQL ASK response has been enhanced with Bloom filters to share a summary of the matching results](cite:cites bloomsparqlask), which allows overlaps between different sources to be identified.

In the context of our work, Bloom filters are used to encode encrypted triple components that are available within each source, and we let aggregators combine them.


### Authentication

Authentication is the process of verifying someone is who they say they are and/or verifying someones attributes or credentials are valid. From an authentication perspective, we provide a brief overview of the predominant identity management approaches:

[Web Identity and Discovery (WebID)](cite:cites spec:webid) is a HTTP URI used to uniquely identify and [authenticate a person, company, organisation, or other entity](cite:cites Sambra2014). A description of the agent is provided in an RDF document, known as a dereferencable WebID profile. The agent places the URI for their WebID profile document in the Subject Alternative Names field of their certificate and the public key details to their [WebID profile document](cite:cites Inkster2014). The WebID Transport Layer Security (TLS) protocol specifies how the WebID profile and public key certificates can together be used to [authenticate users](cite:cites Inkster2014). A service wishing to  authenticate the user, needs to verify that the public key of the certificate it receives matches the public key specified in the [WebID profile](cite:cites Hollenbach2009).

[OpenID Connect](cite:cites OpenIDConnect) is an industry standard authentication protocol, which enables applications to delegate responsibility for authentication to third party identity providers. One of the primary benefits being the ability to connect to multiple sites using the same login credentials. A comprehensive security analysis of the protocol is provided by [Fett et al.](cite:cites fett2017web).

[Self-sovereign identity (SSI)](cite:cites SSI) is a paradigm shift in terms of identity management, whereby individuals manage their own identity credential as opposed to relying on centralised identity providers, such as private or public sector organisations. [Mühle et al.](cite:cites muhle2018survey) provides a high level overview of the various components that are necessary to support SSI. Key supporting technology includes, verifiable claims which has claims that can be verified via a digital signature, and blockchain technology which plays the role of the third party identity provider.

Given that WebID is the default authentication mechanism for Solid, in this paper we have elected to use it for the instantiation of the proposed federated querying with policies framework. However it is worth noting that both OpenIDConnect and SSI are both viable alternatives.


### Access Control

The term authorisation is used to refer to the access control rules that specify that a given subject has access to a given resource. In this section we provide a brief overview of three different approaches that can be used to specify authorizations.

{:.comment data-author="SS"}
↑ above paragraph should be aligned with the new subsection title "Access Control" (IIRC it was named "Authorisation" before?)

[Web Access Control (WAC)](cite:cites spec:wac) is an RDF vocabulary and an access control framework, which demonstrates how together WebID and access control policies specified using the WAC vocabulary, can be used to enforce distributed access control. Essentially WAC authorisations grant agents, access to resources. Both [Villata et al.](cite:cites Villata2011) and [Sacco and Passant](cite:cites Sacco2011b) extend the WAC vocabulary to cater for context based access control policies and privacy preferences respectively.

Pattern Based Access Control is a flexible means to specify the triples that can be access. A triple pattern is composed of an RDF triple with optionally a variable (denoted by a ?) in the subject, predicate and/or object position. [Kirrane et al.](cite:cites Kirrane2013) demonstrate how authorisations based on quad patterns (where the fourth element denotes the named graph) can be used to enforce Discretionary Access Control (DAC), whereby users can pass their access rights on to other users. Typical enforcement strategies involve filtering unauthorised data based on access control policies and executing queries against the filtered dataset, or using query rewriting techniques to inject access control filters into queries.

Encryption Based Access Control involves encrypting RDF fragments (i.e. subjects, predicates, objects, graphs or some combination thereof) with an encryption key, such that only those that have the key are permitted to access the data, thus serving as both an authentication and an authorisation mechanism. Existing proposals involve using [symmetric encryption](cite:cites kasten2013towards), [public-key encryption](cite:cites giereth2005partial), or [functional encryption](cite:cites fernandez2017self) to generate RDF ciphers.

In this paper we discuss how pattern based access control policies specified using shapes languages such as [SHACL](cite:cites spec:shacl) can provide support for expressive access control policies beyond the simple file based access control currently used in Solid, and demonstrate how existing encryption mechanisms can be used to create privacy-preserving aggregation.
