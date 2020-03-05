## A Privacy Preserving Federation Framework 
{:#framework}

<!-- In this section, we introduce a framework to enable querying
over decentralized environments in a privacy-preserving manner,
which provides support for use cases such as the one introduced in [](#use-case).
We put a particular emphasis on three core aspects of the framework:
(i) we introduce a high-level architecture for enabling federated querying in a privacy-preserving manner;
(ii) we list the requirements for enabling fine-grained access control in decentralized environments; and
(iii) we discuss required extensions to identity management within decentralized environments. -->

<!-- ### Privacy-Preserving Federated Querying -->

The proposed privacy-preserving federation architecture is depicted in [](#figure-privacy-federation-architecture).
We first provide a high-level overview of the architecture,
after which we introduce our abstract aggregation algorithm, which is used to create privacy-preserving summaries,
and a client-side algorithm, which is used to execute queries over such summaries.

<figure id="figure-privacy-federation-architecture">
<img src="img/privacy-federation-architecture.svg" alt="[Privacy-Preserving Federated Querying Architecture]">
<figcaption markdown="block">
Overview of a privacy-preserving federation architecture
with six access restricted sources and privacy-preserving summaries,
and a third-party aggregator that combines these summaries in a privacy-preserving manner,
together with a list of all sources it summarizes.
Client-side query engines can use this combined summary to derive which sources are relevant for any given query.
</figcaption>
</figure>

### Architecture Overview

Based on our use case from [](#use-case), we assume that multiple data pods, each potentially containing multiple privacy-constrained files, exist.
If clients want to read the contents of these files,
they have to authenticate themselves to the data pod server. Depending on the pods access control policy the client may be authorized to read the full contents or parts of it.

Since realistic decentralized environments could easily contain hundreds or thousands of files,
it is inefficient for the client to query each of them.
For this reason, we make make use of the _[data summaries](cite:cites summaries)_ concept
in order to reduce the number of sources that need to be queried by the client.
We assume that each data pod exposes a data summary for each separate file,
which is subsequently aggregated by third-party aggregators.
Since files may contain private data, these data summaries must be *privacy-preserving*,
i.e., they must not allow access restricted data to be leaked to unauthorised individuals.
Pods can generate these summaries lazily on demand, either periodically or upon file changes.
An overview of this architecture can be seen in [](#figure-privacy-federation-architecture).
Following the approach from [Vander Sande et al.](cite:cites tpf_amf),
each summary consists of 4 parts, corresponding to the 4 components in RDF quads,
as illustrated in [](#figure-summary-components).

<figure id="figure-summary-components">
<img src="img/summary-components.svg" alt="[Summarization of a file]" height="150px">
<figcaption markdown="block">
Summarization of all RDF quads within a file.
The summary contains 4 parts, corresponding to all subjects, predicates, objects and graphs in the file.
The `CreateSummary` implementation depends on the type of summary,
for which [Vander Sande et al.](cite:cites tpf_amf) provide different implementations.
*Note: The summary values are not necessarily an exact representation of the way the summary is stored,
these values are merely an indication of what information is used to construct the summary.*
</figcaption>
</figure>

Using the summaries of these files, third-party aggregators can create _combined summaries_.
Since the separate summaries are privacy-preserving, the combined summaries will also be privacy-preserving,
which means that third-party aggregators will not be able to know the actual contents of the data,
and they need not necessarily be trusted parties.
In addition to exposing the combined summary,
an aggregator also needs to maintain and expose the list of sources it aggregates over,
such that clients know which data sources could potentially contribute query results.
Although in our example we consider one aggregator, in practise multiple aggregators can exist with different ranges.

<!-- This aggregation process will be explained in more detail in [](#framework-aggregation). -->

A client-side query engine can make use of the combined summary provided by the aggregator to perform source selection before query execution, i.e., reduce the number of sources to be queried. Thus the combined summaries serve to determine the data sources that contain data, which is both relevant and accessible.

<!-- For each source, it should do this by testing the summary for its query using its authentication key.
If the test is true, then the client should consider this source as valid target it can query.
This client-side process will be explained in more detail in [](#framework-client). -->


### Architecture Requirements

The main technical requirements are derived from the fact that our architecture need to support efficient privacy preserving query execution over personal data that is distributed across many sources.
Thus we consider the following key requirements, where `Σ.c` and `Σ'.c` denote existing summaries, `q.c` denotes a quad component, `k` denotes a given access key, and `u` denotes the URI for the data source:

1. **No data leaking**:
    Data within summaries must not be interpretable by those who are not authorised to have access to this data.
2. **Value additions**:
    It must be possible to add values to summaries by access key and file URI.
    An implementation for `SummaryAdd(Σ.c, q.c, k, u)` is required,
    based on an initialized summary as implemented by `SummaryInitialize()`.
3. **Summary combinations**
    It must be possible to combine two summaries,
    where the combined summary is identical to a summary where all of the entries were added directly.
    An implementation for `SummaryCombine(Σ.c, Σ'.c)` is required.
4. **Authorized membership checking**
    Probabilistic membership checking must be possible for a given value, access key and file URI.
    False positives are allowed, but true negatives are required.
    We require that the file URI can be falsy, in case all file URIs must be tested.
    An implementation for `SummaryContains(Σ.c, q.c, k, u)` is required.


### Access Key Creation Algorithm

The main purpose of an aggregator is to enable clients to optimize federated querying
by reducing the range of source to query over through summaries.
Since we are considering private data, these summaries need to be privacy-preserving, 
which means that only people with the appropriate access rights can determine the presence of data.
The first step is to create a hashmap of keys to quads based on existing access policies, using the algorithm outlined in [](#key-generation-algorithm). Here we assume that pod owners already have a set of access control policies that governs quads stored in theirs pods. 


<figure id="key-generation-algorithm" class="listing">
````/code/key-generation-algorithm.txt````
<figcaption markdown="block">
Algorithm for generating keys for quads based on existing access policies, with `MapInitialize` for initializing a hashmap between quads and keys, `GenerateKey` which generates a key for a quad based on a policy, and `AddKey` which adds a key to a hashmap.
</figcaption>
</figure>


### Summary Creation Algorithm
{:#framework-summary-creation}

In practise, multiple aggregators can exist,
which are not necessarily trusted,
and each one should not necessarily aggregate over *all* sources.
For example, an aggregator could be setup within a family to aggregate over all family events that may be hosted by several people,
or a company-wide aggregator can be setup to keep track of the birthdays of all employees.

In the proposed framework, data pods expose a separate summary for each file,
and aggregators: (i) create combined summaries using these separate summaries;
and (ii) maintain a list of all source URIs that they aggregate over.
We assume that pods expose summaries that are created according to the algorithm presented in [](#summarization-algorithm).
In this algorithm, a file summary is created for each quad component,
where we iterate over all the file's quads,
and the access key that are applicable for each quad.
For each of these combinations, we add the quad component to the summary,
for the given key and file source URI.
The `SummaryInitialize` and  `SummaryAdd` functions that are used in the algorithm depend on the type of summary that is being used.
A high-level example of this summarization algorithm can be seen in [](#figure-summary-components-privacy).

<figure id="summarization-algorithm" class="listing">
````/code/summarization-algorithm.txt````
<figcaption markdown="block">
Algorithm for creating a summary over a file within a data pod,
with `SummaryAdd` a summary-type-dependent function for adding a quad component, key, and file URI to summary,
and `SummaryInitialize` a summary-type-dependent function for initializing a new summary.
</figcaption>
</figure>


<figure id="figure-summary-components-privacy">
<img src="img/summary-components-privacy.svg" alt="[Privacy-preserving summarization of a file]">
<figcaption markdown="block">
Privacy-preserving summarization of all RDF quads (`Q`) within a file (`u`),
based on a set of quad-dependent keys that are derived using a mapping function `qk`.
*Note: The summary values are not necessarily an exact representation of the way the summary is stored,
these values are merely an indication of what information is used to construct the summary.*
</figcaption>
</figure>


### Summary Aggregation Algorithm
{:#framework-summary-aggregation}

Based on the resulting file summaries,
the aggregator can create a combined summary using the algorithm from [](#aggregation-algorithm).
As before, the `SummaryInitialize` and `SummaryCombine` functions that are used in these algorithms
depend on the type of summary that is being used.
[](#figure-summary-components-privacy-aggregated) shows a high-level example of how this aggregation can happen in practise.

<figure id="aggregation-algorithm" class="listing">
````/code/aggregation-algorithm.txt````
<figcaption markdown="block">
Algorithm for creating a combined summary over a set of sources,
with `SummaryCombine` a summary-type-dependent function for combining two summaries,
and `SummaryInitialize` a summary-type-dependent function for initializing a new summary.
</figcaption>
</figure>

<figure id="figure-summary-components-privacy-aggregated">
<img src="img/summary-components-privacy-aggregated.svg" alt="[Privacy-preserving aggregated summarization]">
<figcaption markdown="block">
Privacy-preserving aggregation of multiple summaries.
*Note: The summary values are not necessarily an exact representation of the way the summary is stored,
these values are merely an indication of what information is used to construct the summary.*
</figcaption>
</figure>

In practise, summaries and combined summaries require some bookkeeping.
Each file summary must remain up-to-date with respect to the file's contents.
This could be done by either immediately invalidating the summary upon file changes,
or by periodically regenerating the summary.
The combined summary requires similar actions to avoid going stale.
This can be achieved through immediate notifications from the pod to the aggregator upon file changes,
or the aggregator can periodically scan the files or its summaries for changes.


### Client-side Querying Algorithm
{:#framework-client}

Since file-based APIs are the basis for data retrieval on the Web as prescribed by the HTTP protocol,
we assume this as a starting point for federated querying in decentralized environments.
Furthermore, we consider quad pattern-based access to file sources instead of more complex [SPARQL queries](cite:cites spec:sparqllang).
This is because triple and quad patterns are the fundamental elements of SPARQL queries,
and any SPARQL query can be decomposed into multiple smaller quad pattern queries.
For example, client-side query engines such as [Comunica](cite:cites comunica) decompose any SPARQL query
into a sequence of quad pattern queries for evaluation against heterogeneous sources,
where the results of these quad pattern queries are joined together locally.
More complex SPARQL features such as `FILTER` and aggregates are fully handled client-side.
Although intelligent clients could detect more expressive interfaces such as
[SPARQL endpoints](cite:cites spec:sparqlprot) and [Triple Pattern Fragments](cite:cites ldf) interfaces,
and make use of them during query execution, we consider this out-of-scope for this work.


Assuming we have an aggregator exposing a summary over a set of sources,
we introduce the algorithm in [](#client-algorithm) where a client-side query engine can make use of an aggregator's summary
to reduce the number of sources the client should query over.
As input, this algorithm assumes a quad pattern query,
the list of access keys provided by the user,
and the summary and list of sources it obtained from an aggregator.
Based on these inputs, the client will iterate over all non-variable quad components
and all available keys.
For each combination, it will first do a pre-filtering step before locally iterating over all sources.
It will check whether or not the component value
is present in the summary for the current key and component,
with source URI set to `ε` to match with all sources.
If it is not present, then we return an empty array, as none of the sources will contain the given component value.
If it is present, some of the sources *may* contain the component value,
because we consider summaries as being probabilistic.
After that, we iterate over each source URI, and check its presence in the summary of the current component,
combined with the component value and key.
When a true negative is found for a source, this source is removed from the list of sources.
Finally, the remaining list of sources is returned,
which can be used by the query engine to execute the quad pattern query over.
In this algorithm, the `SummaryContains` also depends on the type of summary that is being used.
[](#figure-query-execution) shows an example of how this source selection algorithm can be used in client-side query engines.

<figure id="client-algorithm" class="listing">
````/code/client-algorithm.txt````
<figcaption markdown="block">
Client-side algorithm for selecting query-relevant sources for a quad pattern query
based on a given privacy-preserving summary.
`SummaryContains` is a summary-type-dependent function for checking if a summary contains a given quad component
for a given key and source URI.
</figcaption>
</figure>

<figure id="figure-query-execution">
<img src="img/query-execution.svg" alt="[Query execution over privacy-preserving summaries]">
<figcaption markdown="block">
Federated query execution using a privacy-preserving summary.
*Note: The summary values are not necessarily an exact representation of the way the summary is stored,
these values are merely an indication of what information is used to construct the summary.*
</figcaption>
</figure>

The presented algorithm should be seen as a basis for federated querying over decentralized environments with private data,
where there is a single aggregator, and all sources we want to query over are considered by the aggregator.
In practise, multiple aggregators can exist,
they may apply to overlapping sources,
and some sources may not be aggregated at all.
For these cases, extensions to this algorithm will be needed,
which we consider out-of-scope for this work.


### Access Control Algorithm
{:#framework-access-control}

Once the client has obtained the list of sources that it needs to query, the next step is to execute the query against each source. 
Here there is a need for access control enforcement, such that it is possible to check that a client does in fact possess the credentials necessary to execute the request.

<figure id="access-control-algorithm" class="listing">
````/code/access-control-algorithm.txt````
<figcaption markdown="block">
An algorithm that takes the requesters credentials, the requested access right, a quad pattern, and a policy, and returns the query execution results. The algorithm is composed of an `AllowedAccess` function that checks if access is permitted, and an `ExecuteQuery` function that executes a query and adds the result to the result list.
</figcaption>
</figure>

