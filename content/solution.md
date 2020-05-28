## Challenges and Opportunities
{:#solution}

When it comes to access policy specification, summarisation generation and maintenance, source selection, and access control enforcement there are several open challenges and opportunities, which we discuss hereafter.

<br/>
**Access Policy Specification.**
We assume that pod owners need to be able to specify access control policies that can be enforced both from an indexing and also a query processing perspective, taking into consideration the **no data leaking** requirement.
Considering our use case scenario, in order to support privacy-preserving summaries, there is a need to generate access keys for both the *acquaintances* and the *friends* files, such that the summary generation process does not work with plain text attributes but rather cipher text. In the case where data is public by default, for instance in the case of the *everybody* file, no key is needed.
In this paper, we assume that there is a many to many mapping between quads and policies and a one to one mapping between access policies (enforced at query time) and access keys (used to create privacy preserving data summaries). Our initial proposal makes use of simple symmetric keys, however for more complex scenarios both attribute-based encryption and/or key derivation algorithms could be use to provide support for more complex access policies. When to comes to policy management, there is a need to ensure that (i) access keys are tightly bound to access policies, and (ii) said keys are distributed to authorised individuals (i.e. acquaintances and friends). In order to revoke access to a particular individual one would need to regenerate the keys and redistribute them to authorised individuals.

<br/>
**Summary Generation and Maintenance.**
The technical requirements for enabling federated querying in an efficient manner through privacy-preserving aggregators are mainly driven by the summarisation technology. In this context symmetric keys are used to create privacy-preserving summaries that do not leak access restricted data. We consider AMFs, such as Bloom filters, as being one possible candidate for such summaries that meet the **privacy-preserving summary creation.** and **summary combinations** requirements.
The main advantage of using AMFs, such as Bloom filters, is that all of the performance-critical operations on summaries (adding, combining, membership checking) can happen very efficiently, as these are essentially just bitwise operations.
However, when using AMFs, it is important to consider that certain parameters need to be configured, and that all operations must be known before they can be operationalised. For example, for Bloom filters the parameters are the number of hashes and bits. These parameters and the number of entries all impact the false positive error rate. Concretely, the parameters used to setup individual summaries need to be identical such that they can be combined by an aggregator. In a decentralised environment, it is however difficult to reach a consensus between all parties with respect to fixed parameters.
Furthermore, since the number of values within an AMF has an impact on the error rate,
it is sometimes even required to use different parameters for different numbers of entries.
As such, no single set of AMF parameters should be used.
This means that a parameter determination mechanism is needed for aggregators
that want to combine multiple AMFs. Also, since the creation of an AMF for a file can become expensive,
sources may decide to adopt different maintenance strategies.

<br/>
**Source Selection.**
In the proposed framework, a client-side query engine can make use of the aggregator's summary to perform source selection,
in order to reduce the number of sources that are being consulted by this engine. From a source selection perspective, we address the **authorized membership checking.** requirement.
As these summaries allow source selection based on quad patterns instead of full SPARQL queries,
source selection can be pushed down into the query plan,
which allows quad patterns in the query to be executed over a different range of sources.
Furthermore, instead of applying source selection before query execution,
this allows source selection to optionally happen adaptively during query execution,
following the federation algorithm of [Triple Pattern Fragments](cite:cites ldf).
A hybrid approach where source selection happens both before and during query execution could be investigated.
One open challenge will be to investigate how this file-based source selection method could be combined and enhanced
by existing source selection methods for SPARQL endpoints. Another open challenge is the automatic discovery of applicable aggregators by clients.

<br/>
**Query Execution with Access control.**
Since file-based APIs are the basis for data retrieval on the Web as prescribed by the HTTP protocol,
we assume this as a starting point for federated querying in decentralised environments.
Furthermore, we consider quad pattern-based access to file sources instead of more complex SPARQL queries.
This is because triple and quad patterns are the fundamental elements of SPARQL queries,
and any SPARQL query can be decomposed into multiple smaller quad pattern queries.
For example, client-side query engines such as [Comunica](cite:cites comunica) decompose any SPARQL query
into a sequence of quad pattern queries for evaluation against heterogeneous sources,
where the results of these quad pattern queries are joined together locally.
More complex SPARQL features such as `FILTER` and aggregates are fully handled client-side.
Once the query engine has identified the data sources that could potentially contribute results to their query, the query engine needs to authenticate the user to the server(s) and execute the query or parts thereof. In turn the server is responsible for enforcing access control, and executing the query or parts thereof. Here, we address the **query execution with access control** requirements. One of the key challenges with respect to access and usage Control relates to the enforcement of authorisations, i.e., policies (cf., [](#policy_spec)) which govern _who_ can do _what_ with _which_ resources under _what_ conditions. Here, we envision a mechanism that translates access policies (i.e. sets of authorisations) into constraints (e.g., data shapes like [SHACL](cite:cites spec:shacl)) which requests and respective query results can then be validated against. However, the trade-off between granularity of policies (e.g., file-based, pattern-based, quad-based, ...) and associated computational overhead needs to be thoroughly investigated.
