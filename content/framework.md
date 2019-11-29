## Framework
{:#framework}

In this section, we introduce a framework to enable querying
over decentralized environments in a privacy-preserving manner,
to enable use cases such as introduced in [](#use-case).
For this, we focus on three aspects.
First, we introduce a high-level architecture for enabling federated querying in a privacy-preserving manner.
Second, we list the requirements for enabling fine-grained access control in decentralized environments.
Third, we discuss required extensions to identity management within decentralized environments.
In the subsections hereafter, we discuss these three aspects in more detail.

### Privacy-Preserving Federated Querying

In this section, we introduce a general decentralized architecture federated querying over privacy-constrained data,
for which an overview can be seen in [](#figure-privacy-federation-architecture).
We first introduce a high-level overview of the architecture,
after which we introduce an aggregation algorithm for creating privacy-preserving summaries,
and a client-side algorithm to query over such summaries.
In this overview, we do not focus on specific technologies,
instead, we end the section with a list of requirements
and offer examples of technologies that can be used to instantiate this architecture.

#### Architecture Overview

Based on our use case from [](#use-case), we assume that multiple data pods exist,
which each can contain multiple privacy-constrained files.
If clients want to read the contents of these files,
they have to authenticate themselves to the data pod server,
after which they may be authorized to read the full contents or parts of it.

Since realistic decentralized environments could easily contain hundreds or thousands of files,
it is unfeasible for the client to query over them directly.
For this reason, we make make use of the concept of _[data summaries](cite:cites summaries)_
to allow clients to reduce the number of sources to query over.
For this, we assume that each data pod exposes a data summary for each separate file,
and third-party aggregators that can aggregate these summaries.
Since files may contain private data, these data summaries must be *privacy-preserving*,
i.e., they must not allow the presence of contents to be leaked without the proper authentication.
Pods can either generate these summaries lazily on demand,
or they can be generated periodically or upon file changes.
An overview of this architecture can be seen in [](#figure-privacy-federation-architecture).

Using the summaries of these files, third-party aggregators can create _combined summaries_.
Since the separate summaries are privacy-preserving, the combined summaries will also be privacy-preserving,
which means that third-party aggregators will not be able to know the actual contents of the data,
and they need not necessarily be trusted parties.
Next to exposing the combined summary,
an aggregator should also maintain and expose the list of sources it aggregates over,
so that clients can derive the range of sources a summary is applied over.
In our example we consider one aggregator, but in practise multiple aggregators can exist with different ranges.
This aggregation process will be explained in more detail in [](#framework-aggregation).

Finally, a client-side query engine that is aware of such an aggregator
can make use of the combined summary to perform source selection before query execution,
i.e., reduce the number of sources it has to request.
For each source, it should do this by testing the summary for its query using its authentication key.
If the test is true, then the client should consider this source as valid target it can query.
This client-side process will be explained in more detail in [](#framework-client).

<figure id="figure-privacy-federation-architecture">
<img src="img/privacy-federation-architecture.svg" alt="[Privacy-Preserving Federated Querying Architecture]">
<figcaption markdown="block">
Overview of a privacy-preserving federation architecture
with six sources with access control and privacy-preserving summaries,
and a third-party aggregator that combines these summaries in a privacy-preserving manner,
together with a list of all sources it summarizes.
Client-side query engines can use this combined summary to derive which sources are relevant for any given query.
</figcaption>
</figure>

#### Aggregation Algorithm
{:#framework-aggregation}

The main purpose of an aggregator is to enable clients to optimize federated querying
by reducing the range of source to query over through summaries.
Since we are considering private data, these summaries are privacy-preserving,
which means that only people with the proper credentials must be able to determine the presence of data.

In practise, multiple aggregators can exist,
which are not necessarily trusted,
and each one should not necessarily aggregate over *all* sources.
For example, an aggregator could be setup within a family to aggregate over all family events that may be hosted by several people,
or a company-wide aggregator can be setup to keep track of the birthdays of all employees.

In this framework, we assume that each data pod exposes a separate summary over each file,
an that aggregator creates a combined summary over these separate summaries
and maintains a list of all source URIs it is aggregating over.
We assume that pods expose summaries that are created according to the algorithm from [](#summarization-algorithm).
In this algorithm, a file summary is created for each quad component,
where we iterate over all the file's quads,
and the keys that are applicable for each quad.
For each of these combinations, we add the quad component to the summary, 
for the given key and file source URI.
Based on the resulting file summaries,
the aggregator can create a combined summary using the algorithm from [](#aggregation-algorithm).
The `Summary_Add` and `Summary_Combine` functions that are used in these algorithms
depend on the type of summary that is being used,
for which we will list the requirements and give examples at the end of this section.

<figure id="summarization-algorithm" class="listing">
````/code/summarization-algorithm.txt````
<figcaption markdown="block">
Algorithm for creating a summary over a file within a data pod,
with `Summary_Add` a summary-type-dependent function for adding a quad component, key, and file URI to summary.
</figcaption>
</figure>

<figure id="aggregation-algorithm" class="listing">
````/code/aggregation-algorithm.txt````
<figcaption markdown="block">
Algorithm for creating a combined summary over a set of sources,
with `Summary_Combine` a summary-type-dependent function for combining two summaries.
</figcaption>
</figure>

In practise, these summaries and combined summaries require some bookkeeping.
Each file summary must remain up-to-date with respect to the file's contents.
This could be done by either immediately invalidating the summary upon file changes,
or by periodically regenerating the summary.
The combined summary requires similar actions to avoid going stale.
This can be achieved through immediate notifications from the pod to the aggregator upon file changes,
or the aggregator can periodically scan the files or its summaries for changes.

#### Client-side Querying Algorithm
{:#framework-client}

Since file-based APIs are the basis for data retrieval on the Web as prescribed by the HTTP protocol,
we assume this as a starting point for federated querying in decentralized environments.
Intelligent clients could however detect more expressive interfaces such as
[SPARQL endpoints](cite:cites spec:sparqlprot) and [Triple Pattern Fragments](cite:cites ldf) interfaces,
and make use of them during query execution, but we consider this out-of-scope for this work.

Furthermore, we consider quad pattern-based access to file sources instead of more complex [SPARQL queries](cite:cites spec:sparqllang).
This is because triple patterns or quad patterns are the fundamental elements of SPARQL queries,
and any SPARQL query can be decomposed into multiple smaller quad pattern queries.
For examples, client-side query engines such as [Comunica](cite:cites comunica) decompose any SPARQL query
into a sequence of quad pattern queries for evaluation against heterogeneous sources,
where the results of these quad pattern queries are joined together locally.
More complex SPARQL features such as `FILTER` and aggregates are fully handled client-side.

Assuming we have an aggregator exposing a summary over a set of sources,
we introduce the algorithm in [](#client-algorithm) where a client-side query engine can make use of an aggregator's summary
to reduce the number of sources the client should query over.
As input, this algorithms assumes a quad pattern query,
the list of access control keys of the user,
and the summary and list of sources it obtained from an aggregator.
Based on these inputs, the client will iterate over all non-variable quad components
and all available keys.
For each combination, it will first do a pre-filtering step before locally iterating over all sources.
It will check whether or not the component value
is present in the summary for the current key and component,
with source URI set to `0` to match with all sources.
If it is not present, then we return an empty array, as none of the sources will contain the given component value.
If it is present, some of the sources *may* contain the component value,
because we consider summaries as being probabilistic.
After that, we iterate over each source URI, and check its presence in the summary of the current component,
combined with the component value and key.
When a true negative is found for a source, this source is removed from the list of sources.
Finally, the remaining list of sources is returned,
which can be used by the query engine to execute the quad pattern query over.
In this algorithm, the `Summary_Contains` also depends on the type of summary that is being used.

<figure id="client-algorithm" class="listing">
````/code/client-algorithm.txt````
<figcaption markdown="block">
Client-side algorithm for selecting query-relevant sources for a quad pattern query
based on a given privacy-preserving summary.
`Summary_Contains` is a summary-type-dependent function for checking if a summary contains a given quad component
for a given key and source URI.
</figcaption>
</figure>

The presented algorithm should be seen as a basis for federated querying over decentralized environments with private data,
where there is a single aggregator, and all sources we want to query over are considered by the aggregator.
In practise, multiple aggregators can exist,
they may apply to overlapping sources,
and some sources may not be aggregated at all.
For these cases, extensions to this algorithm will be needed,
which we consider out-of-scope for this work.

#### Architecture Requirements

The privacy-preserving summaries drive the main technical requirements within our architecture.
We consider the following requirements:

1. **No data leaking**:
    Data within summaries must not be readable without the proper authentication keys.
2. **Value additions**:
    It must be possible to add values to summaries by key and file URI.
    An implementation for `Summary_Add(S_C, v, k, u)` is required.
3. **Summary combinations**
    It must be possible to combine two summaries,
    where the combined summary is identical to a summary where all of the entries were added directly.
    An implementation for `Summary_Combine(S_C, S_C')` is required.
4. **Authorized membership checking**
    Probabilistic membership checking must be possible for a given value, key and file URI.
    False positives are allowed, but true negatives are required.
    We require that the file URI can be falsy, in case all file URIs must be tested.
    An implementation for `Summary_Contains(S_C, v, k, u)` is required.

### Extension of Access Control

{:.todo}
Simon

* Data owners are responsible for enforcing access control (as opposed to other approaches where federation engine takes care of that). We assume that access control is already taken care of at the (client-side) federation engine.
* Build on Solid's WebID-OIDC for auth, and WAC for access control.
* Allow shape-based/quadpattern-based (SHACL/SHEX) access modes in .acl files. (advantage of shapes is that fewer keys may be needed, which reduces the complexity of key mgmt) (motivation for keys is that Solid is going to use it for data validation: https://github.com/solid/data-interoperability-panel/blob/b2591bf2f8808972e5459db2aa4ac8d9854f5b5e/data-validation/use-cases.md)
* Right now, we do it role-based, but it could be attribute-based as well.
{:.todo}

~~~ turtle

 <#authorization1>
    a             acl:Authorization;
    acl:agent     <https://alice.databox.me/profile/card#me>;  # Alice's WebID
    acl:accessTo  <https://alice.databox.me/docs/file1>;
    acl:mode      acl:Read, 
                  acl:Write, 
                  acl:Control,
                  [ a acl:FilteredRead;
                    sh:select """CONSTRUCT WHERE { ?subject a ?object } """ ].

~~~


two options:

* (shacl-spec): validation-based (~filter)
* (shacl note): filter/rule-based

### Extension of Identity Management

{:.todo}
Sabrina

* Each user has a list of keys, keys are shared among users for same parts of data
* Explain key management (creation and where it's stored) and revocation
* Mention self-sovereign identity?
{:.todo}

#### Architecture Requirements

TODO
