## Background and Related Work
{:#background}

Before describing our motivating scenario and introducing our privacy-preserving federation framework, we first present the necessary background information in relation to the Solid platform, federated query processing, approximate membership functions, authentication and access control.

{::options parse_block_html="true" /}

<div class="bs-callout bs-callout-info">
By browsing through solid github issues (e.g., [solid/authorization-and-access-control-panel#53](https://github.com/solid/authorization-and-access-control-panel/issues/53) [solid/authorization-and-access-control-panel#67](https://github.com/solid/authorization-and-access-control-panel/issues/67?)) and respective  solid ac/auth panel's meeting minutes ([20.04.20](https://github.com/solid/authentication-panel/blob/master/meetings/2019-04-20.md#user-focused-client-constraining-access-control) and [27.04.20](https://github.com/solid/authentication-panel/blob/master/meetings/2019-04-20.md#user-focused-client-constraining-access-control)) I stumbled over a discussion about **User-Focused Client Constraining Access Control** which then led me to
some stuff that sounds kinda related to what we propose, especially the CfA Request/Accept Policies look quite interesting! what do you think?

  <strong>1. Peer Relationship Management</strong>\\
  [https://hackmd.io/Yb8qBdIoQROeF32TCa0zTg?view](https://hackmd.io/Yb8qBdIoQROeF32TCa0zTg?view)
<details>

PRM reconciles well known [Customer Relationship Management (CRM)](https://en.wikipedia.org/wiki/Customer_relationship_management) with less known [Vendor Relationship Management (VRM)](https://en.wikipedia.org/wiki/Vendor_relationship_management). In supply chains scenarios, many coops take on customer role in relationship with some coops and vendor role in relationship with others. Peer relationship management attempts to provide all the coops with a way to manage their relationship with other coops no matter which roles they play in any given relationship. In some cases each of two coops can play both roles in their relationship.

In scenario below we will use [Conversation for Action (CfA)](http://conversationsforaction.com/) flow for interactions between coops. In CfA coop playing role of _vendor_ is commonly refered to as **performer**.

![](https://i.imgur.com/Urq3oBB.png)

On the diagram we can focus on 3 Coops with 2 distinct relationship between them:

- Yoydyne (vendor) -> ACME (customer)
- ACME (vendor & customer) <-> Sirius (customer & vendor)

Each of those 3 Coops at the same time have number of other relationships with various other coops.

In each of those relationships we can have any number of Confersations for Action happening at any given moment. One simplest CfA would consist of those 4 steps:

- ACME -> Yoyodyne request: 1L glass jars
- Yoyodyne -> ACME accept: 1L glass jars
- Yoyodyne -> ACME done: 1L glass jars
- ACME -> Yoyodyne approve: 1L glass jars

More complex conversations may involve any number of additional steps like: _revise_, _counter_, _revoke_, _request re-do_ etc. On the other hand some conversations may end on second step with _decline_ from vendor/performer or _cancel_ from customer.

Many people can work working in each of those coops. At the same time each person person can work for any number of coops.

## Data requirements

- Each coop has one or more instances of Solid Storage.
- Each coop publishes all CfA artifacts they create to their storage.
  - They give read access rights to the other coop which artifact is addressed to. They give those access rights directly to the coop identity.
  - They notify that coop about new artifact that they now has access to.
- Each coop keeps in their storage a copy of all the CfA artifacts created by other coops, which they have read access to in all those other coop's storages. Whenever they receive notification an automated service create those copies on behalf of the coop.
- Each coop manages independently data access for each of the workers. No worker directly accesses data in storage of other coops. They only can access copies in the storage of coop which they work for.
- Each worker can use PRM applications of their choice running on their personal devices. One person can work for more than one coop and they shouldn't need to switch applications whenever they do PRM related work for a different coop.
- When when worker relationship betwen person and coop ends. That coop can independently remove that person's access to any data (reminder: agents never authorize workers to access any data in storages of other coops)

### Example

Continuing on example where ACME requests _1L glass jars_ from Yoyodyne.

> _NOTE:_ Code snippets use [TriG](https://www.w3.org/TR/trig/) syntax only to ephasize where LDP RDF Source is stored. Snippets also omit details not relevant to where data gets stored.

#### ACME -> Yoyodyne request: 1L glass jars

ACME creates request and publishes it to their storage:

~~~ turtle
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix cfa: <https://cfa.example/ns#> .

GRAPH <https://acme.example/6800yrn/tv00vyj>
{
<https://acme.example/6800yrn/tv00vyj#it>
  a cfa:Request ;
  cfa:inConversation <urn:uuid:e1dc2594-b219-46e5-a850-8dc2db1885f4> ;
  cfa:customer <https://acme.example/#coop> ;
  cfa:performer <https://yoyodyne.example/#coop> ;
  rdf:label "request 1L glass jars" .
}

~~~

ACME sends Linked Data Notification to Yoyodyne

~~~ turtle
# TODO
~~~

Yoyodyne's automated system creates copy of that request in yoyodyne's storage.

> _NOTE:_ That copy will be created in predetermined location for CfA artifacts between Yoyodyne and ACME `https://yoyodyne.example/5f00ken/`, this way workers managing this relationship will have access to that copy.

~~~ turtle
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix cfa: <https://cfa.example/ns#> .

GRAPH <https://yoyodyne.example/5f00ken/nb00ah3>
{
<https://acme.example/6800yrn/tv00vyj#it>
  a cfa:Request ;
  cfa:inConversation <urn:uuid:e1dc2594-b219-46e5-a850-8dc2db1885f4> ;
  cfa:customer <https://acme.example/#coop> ;
  cfa:performer <https://yoyodyne.example/#coop> ;
  rdf:label "request 1L glass jars" .
}

~~~

#### Yoyodyne -> ACME accept: 1L glass jars

~~~ turtle
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix cfa: <https://cfa.example/ns#> .

GRAPH <https://yoyodyne.example/5f00ken/4j00v5c>
{
  <https://yoyodyne.example/5f00ken/4j00v5c#it>
    a cfa:Accept ;
    cfa:inConversation <urn:uuid:e1dc2594-b219-46e5-a850-8dc2db1885f4> ;
    cfa:customer <https://acme.example/#coop> ;
    cfa:performer <https://yoyodyne.example/#coop> ;
    rdf:label "request 1L glass jars" .
}

~~~

Yoyodyne sends Linked Data Notification to ACME

~~~ turtle
# TODO
~~~

ACME's automated system creates copy of that acceptance in ACME's storage.

> _NOTE:_ Similar to previous step, ACME stores CfA artifacts between them and Yoyodyne in `https://acme.example/6800yrn/`. This way ACME workers manging its relationship with Yoyodne can access all relevant CfA artifacts.

~~~ turtle
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix cfa: <https://cfa.example/ns#> .

GRAPH <https://acme.example/6800yrn/83009ft>
{
  <https://yoyodyne.example/5f00ken/4j00v5c#it>
    a cfa:Accept ;
    cfa:inConversation <urn:uuid:e1dc2594-b219-46e5-a850-8dc2db1885f4> ;
    cfa:customer <https://acme.example/#coop> ;
    cfa:performer <https://yoyodyne.example/#coop> ;
    rdf:label "request 1L glass jars" .
}

~~~

</details>

----

<strong>2. Conversation for Action (CfA)</strong>\\
[http://conversationsforaction.com/](http://conversationsforaction.com/)

<details>

![](http://conversationsforaction.com/sites/default/files/loop.jpg){:height="100%" width="100%"}

Each individual workflow is a structure of commitments that constitutes a transaction between two constituencies-- a customer and a performer -- to accomplish a purpose. This purpose will often change and be re-articulated as the two parties work together, develop new insights/learning, and become clearer about what is possible and what is needed.

The structure of commitments of any process within a company can be observed and designed as a collection of interconnected workflows. I called these structures commitment processes to distinguish them from more traditional interpretations of business processes as flows of information, materials and tasks. A commitment process is a network of individual workflows configured to fulfill one of the recurrent commitments or ongoing objectives of an enterprise, such as developing and marketing new offers, delivering offers to customers, or conducting other general operations of the business.
</details>

----


<strong>3. Web of Needs</strong>\\
[https://github.com/researchstudio-sat/webofneeds](https://github.com/researchstudio-sat/webofneeds)

<details>

![interaction-diagram](http://researchstudio-sat.github.io/webofneeds/images/interaction-diagram-book.png){:height="100%" width="100%"}

The Web of Needs is a decentralized infrastructure that allows people to publish documents on the Web which make it possible to contact each other. The document may contain a description of a product or service required or offered, a description of a problem to be solved with the help of others, an invitation to social activities, or anything else users may think of. Some concrete use cases are described here. On the abstract level of description, the document can be said to represent an interest in or a need for some kind of interaction with others.

As this document or entity is the central and indivisible building block of the system, we refer to it as an atom. Each atom has a globally unique identifier and an owner, i.e., a person or other entity that creates and controls it. When atom owners want to communicate with each other, a connection object is created for each atom involved.

Web of Needs is built out of three main components. Owner applications enable users to create and manage their atom objects. They can be any type of UI application like web applications or mobile apps for example. Owner applications publish atoms as RDF documents to won nodes on the Web. When atoms are published on the Web, independent matching services can crawl them (or subscribe for atom updates at won nodes) and look for suitable matches. A protocol is in place to inform the won nodes and atom owners of possible matches using hint messages. Based on this process atom owners can initiate connections to other atoms and start communication and other transactions.

Anyone can run any of these components. They can all talk to each other.
</details>

----

</div>

{::options parse_block_html="false" /}

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

[Web Identity and Discovery (WebID)](cite:cites spec:webid) is a HTTP URI used to uniquely identify and [authenticate a person, company, organisation, or other entity](cite:cites Sambra2014). A description of the agent is provided in an RDF document, known as a dereferencable WebID profile. The agent places the URI for their WebID profile document in the Subject Alternative Names field of their certificate and the public key details to their [WebID profile document](cite:cites Inkster2014). The WebID Transport Layer Security (TLS) protocol specifies how the WebID profile and public key certificates can together be used to [authenticate users](cite:cites Inkster2014). A service wishing to authenticate the user, needs to verify that the public key of the certificate it receives matches the public key specified in the [WebID profile](cite:cites Hollenbach2009).

[OpenID Connect](cite:cites OpenIDConnect) is an industry standard authentication protocol, which enables applications to delegate responsibility for authentication to third party identity providers. One of the primary benefits being the ability to connect to multiple sites using the same login credentials. A comprehensive security analysis of the protocol is provided by [Fett et al.](cite:cites fett2017web).

[Self-sovereign identity (SSI)](cite:cites SSI) is a paradigm shift in terms of identity management, whereby individuals manage their own identity credential as opposed to relying on centralised identity providers, such as private or public sector organisations. [Mühle et al.](cite:cites muhle2018survey) provides a high level overview of the various components that are necessary to support SSI. Key supporting technology includes, verifiable claims which has claims that can be verified via a digital signature, and blockchain technology which plays the role of the third party identity provider.

Given that WebID is the default authentication mechanism for Solid, in this paper we have elected to use it for the instantiation of the proposed federated querying with policies framework. However it is worth noting that both OpenIDConnect and SSI are both viable alternatives.

### Authorisation and Access Control

The term authorisation is used to refer to the access control rules that specify that a given subject has access to a given resource.

In this section we provide a brief overview of three different approaches that can be used to specify authorisation rules.

[Web Access Control (WAC)](cite:cites spec:wac) is an RDF vocabulary and an access control framework, which demonstrates how together WebID and access control policies specified using the WAC vocabulary, can be used to enforce distributed access control. Essentially WAC authorisations grant agents, access to resources. Both [Villata et al.](cite:cites Villata2011) and [Sacco and Passant](cite:cites Sacco2011b) extend the WAC vocabulary to cater for context based access control policies and privacy preferences respectively.

Pattern-based Access Control is a flexible means to specify the triples that can be access. A triple pattern is composed of an RDF triple with optionally a variable (denoted by a ?) in the subject, predicate and/or object position. [Kirrane et al.](cite:cites Kirrane2013) demonstrate how authorisations based on quad patterns (where the fourth element denotes the named graph) can be used to enforce Discretionary Access Control (DAC), whereby users can pass their access rights on to other users. Typical enforcement strategies involve filtering unauthorised data based on access control policies and executing queries against the filtered dataset, or using query rewriting techniques to inject access control filters into queries.

Encryption-Based Access Control involves encrypting RDF fragments (i.e. subjects, predicates, objects, graphs or some combination thereof) with an encryption key, such that only those that have the key are permitted to access the data, thus serving as both an authentication and an authorisation mechanism. Existing proposals involve using [symmetric encryption](cite:cites kasten2013towards), [public-key encryption](cite:cites giereth2005partial), or [functional encryption](cite:cites fernandez2017self) to generate RDF ciphers.

_if we remove the SHACL stuff, we don't really add anything to the body of work we've outlined 2 paragraphs earlier (e.g. [Kirrane et al.](cite:cites Kirrane2013)) right?_{:.sidenote}

In this paper we discuss how pattern-based access control policies expressed in terms of sets of authorisations can provide support for expressive access control policies beyond the simple file-based access control currently used in Solid, and demonstrate how existing encryption mechanisms can be used to create privacy-preserving aggregation.
{:.sidebar-comment}
