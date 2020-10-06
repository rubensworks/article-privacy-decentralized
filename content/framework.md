## Efficient Privacy-Preserving Federated Querying
{:#framework}

The goal of the proposed efficient privacy-preserving federated query execution framework, depicted in [](#figure-privacy-federation-architecture), is to provide a high level overview of the components needed to support privacy-preserving querying within decentralised environments.

<figure id="figure-privacy-federation-architecture">
<img src="img/privacy-federation-architecture.svg" alt="[Privacy-Preserving Federation]" class="figure-width-twothird">
<figcaption markdown="block">
Our efficient privacy-preserving federated query execution framework.
</figcaption>
</figure>

### Using Data Summaries for Efficient Querying

Based on the use case scenario presented in [](#use-case), we assume that many data pods exist, each potentially containing multiple privacy-constrained files and, clients need to authenticate themselves to the respective data pod servers. Depending on each file's access control policy, the client may be authorised to read the full file contents, parts of it, or not at all.
Since realistic decentralised environments could easily contain hundreds or thousands of files,
it would be inefficient for the client to query each of them.
For this reason, we make make use of the [_data summaries_](cite:cites summaries) concept
in order to reduce the number of sources that need to be queried by the client.
We assume that each data pod exposes a data summary for each separate file, which is subsequently aggregated by third-party aggregators, as depicted in [](#figure-privacy-federation-architecture).
The figure provides an overview of a privacy-preserving federation with six access restricted sources and privacy-preserving summaries, and a third-party aggregator that combines these summaries in a privacy-preserving manner, together with a list of all sources it summarises. Client-side query engines can use this combined summary to derive which sources are relevant for any given query. Since files may contain private data, these data summaries must be _privacy-preserving_,
i.e., they must not allow access restricted data to be leaked to unauthorised individuals.
In the proposed framework, access policies are represented as access keys that are taken into account by the summary generation algorithm. Pods could generate these summaries lazily on demand, periodically or upon file changes.
Following the approach from [Vander Sande et al.](cite:cites tpf_amf), each summary consists of 4 parts, corresponding to the 4 components in RDF quads (subjects, predicates, objects and graphs).
Using the summaries of these files, third-party aggregators can create _combined summaries_.
Since the separate summaries are expected to be privacy-preserving, the combined summaries will also be privacy-preserving,
which means that third-party aggregators need not necessarily be trusted parties.
In addition to exposing the combined summary,
an aggregator also needs to maintain and expose the list of sources it aggregates over,
such that clients know which pods could potentially contribute query results.
Although in our example we consider one aggregator, in practice multiple aggregators can exist with different source ranges.
A client-side query engine can make use of the combined summary provided by the aggregator to perform source selection before query execution, i.e., reduce the number of sources to be queried. Thus the combined summaries serve to determine the pods that contain _relevant_ and _accessible_ data.
While, the pods take care of the access control enforcement at query time, by taking into account permissions specified in terms of authorisation rules.

### Technical Requirements

The main technical requirements are derived from the fact that our architecture needs to support efficient privacy-preserving query execution over personal data that is distributed across many sources.

- **No data leaking.**
   Access restricted data must not be available to those who are not authorised to access it.
- **Privacy-preserving summary creation.**
   It must be possible to add values to summaries by access key and file URI.
- **Summary combinations.**
   It must be possible to combine two summaries,
   where the combined summary is identical to a summary where all of the entries were added directly.
- **Authorised membership checking.**
   Probabilistic membership checking must be possible for a given value, access key and file URI.
   False positives are allowed, but true negatives are required.
- **Query Execution with Access control.**
   It must be possible for the pod to limit query results based on a set of access policies.

### Core Functions of the Framework

We also propose a set of abstract algorithms that are needed in order to realise the proposed privacy-preserving federation framework. The proposed abstraction is benefical as each algorithm could be implemented in a variety of ways.


<br/>

#### Access Key Creation Algorithm.
{:#framework-key-creation}

As a prerequisite for encoding access into summaries,
the first step is to create a map of access keys to quads based on existing access policies, using the algorithm outlined in [](#key-generation-algorithm). Here we assume that pod owners already have a set of access control policies that govern access to quads stored in theirs pods. Although there is a many to many mapping between quads and policies, there is a one to one mapping between access policies that are used for policy enforcement at query time, and access keys that are used to create privacy-preserving summaries that are needed to optimise federated querying.


<font size="2">
<figure id="key-generation-algorithm" class="listing">
````/code/key-generation-algorithm.txt````
<figcaption markdown="block">
Algorithm for generating keys for quads based on existing access policies.
</figcaption>
</figure>
</font>

#### Summary Creation Algorithm.
{:#framework-summary-creation}

In the proposed framework, data pods expose a separate summary for each file,
and aggregators create combined summaries using these separate summaries;
and maintain a list of all source URIs that they aggregate over.
We assume that pods expose summaries that are created according to the algorithm presented in [](#summarisation-algorithm).
In this algorithm, a file summary is created for each quad component,
where we iterate over all the file's quads,
and the access key that are applicable for each quad.
For each of these combinations, we add the quad component to the summary,
for the given key and file source URI.
The `SummaryInitialize` and `SummaryAdd` functions that are used in the algorithm depend on the type of summary that is being used.
A high-level example of this summarisation algorithm can be seen in [](#figure-summary-components-privacy).


<font size="2">
<figure id="summarisation-algorithm" class="listing">
````/code/summarization-algorithm.txt````
<figcaption markdown="block">
Algorithm for creating a summary over a file within a data pod.
</figcaption>
</figure>
</font>


<figure id="figure-summary-components-privacy">
<img src="img/summary-components-privacy.svg" alt="[Privacy-preserving summarisation of a file]" class="figure-width-twothird">
<figcaption markdown="block">
Privacy-preserving summarisation of all RDF quads within a file.
</figcaption>
</figure>

#### Summary Combination Algorithm.
{:#framework-summary-aggregation}

Based on the resulting file summaries,
the aggregator can create a combined summary using the algorithm from [](#aggregation-algorithm).
As before, the `SummaryInitialize` and `SummaryCombine` functions that are used in these algorithms
depend on the type of summary that is being used.
[](#figure-summary-components-privacy-aggregated) shows a high-level example of how this aggregation could be performed.
It is worth noting that both summaries and combined summaries require some bookkeeping.
Each file summary must remain up-to-date with respect to the file's contents.
This could be done by either immediately invalidating the summary upon file changes,
or by periodically regenerating the summary.
The combined summary requires similar actions to avoid going stale.
This can be achieved through immediate notifications from the pod to the aggregator upon file changes,
or the aggregator can periodically scan the files or its summaries for changes.

<font size="2">
<figure id="aggregation-algorithm" class="listing">
````/code/aggregation-algorithm.txt````
<figcaption markdown="block">
Algorithm for creating a combined summary over a set of sources.
</figcaption>
</figure>
</font>

#### Client-side Source Selection Algorithm.
{:#framework-client}

Assuming we have an aggregator exposing a summary over a set of sources,
we introduce the algorithm in [](#client-algorithm) where a client-side query engine can make use of an aggregator's summary
to reduce the number of sources the client should query over, i.e., to perform _source selection_.
In this case, we only consider quad pattern queries,
because they form the foundation of more expressive SPARQL queries,
and [query engines typically decompose SPARQL query into several smaller quad pattern queries through a query planner](cite:cites comunica).
As input, our algorithm assumes a quad pattern query,
the list of access keys provided by the user,
and the summary and list of sources it obtained from an aggregator.
Based on these inputs, the client will iterate over all non-variable quad components
and all available keys.
For each combination, it will first do a pre-filtering step before locally iterating over all sources.
It will check whether or not the quad component value
is present in the summary for the current key and quad component,
with source URI set to `ε` to match with all sources.
If it is not present, then we return an empty array, as none of the sources will contain the given component value.
If it is present, some of the sources _may_ contain the component value,
because we consider summaries as being probabilistic.
After that, we iterate over each source URI, and check its presence in the summary of the current quad component,
combined with the component value and key.
When a true negative is found for a source, this source is removed from the list of sources.
Finally, the remaining list of sources is returned,
which can be used by the query engine to execute the quad pattern query over.
In this algorithm, the `SummaryContains` also depends on the type of summary that is being used.

<font size="2">
<figure id="client-algorithm" class="listing">
````/code/client-algorithm.txt````
<figcaption markdown="block">
Client-side algorithm for selecting query-relevant sources.
</figcaption>
</figure>
</font>

#### Client-side Query Execution Algorithm.
{:#framework-access-control-client}

Once the client has obtained the list of sources that it needs to query for a given quad pattern, the next step is to execute the query against each source.
The client uses the sources returned by the aggregator to execute queries against the various pods using the algorithm outlined in [](#client-query-algorithm).
The algorithm takes as input a client identification (e.g., WebID), a quad pattern query, and the set of sources returned by the source selection algorithm. Individual queries are executed against each of the sources and the aggregated results are returned to the client.

<font size="2">
<figure id="client-query-algorithm" class="listing">
````/code/request-algorithm.txt````
<figcaption markdown="block">
Client-side algorithm for querying query-relevant sources.
</figcaption>
</figure>
</font>

#### Server-side Query Execution Algorithm.
{:#framework-access-control-server}

On receipt of a query the server uses the algorithm outlined in [](#server-query-algorithm) to ensure that only authorised query results are returned to the client.
In the proposed algorithm a map relating quads to policies and keys is used to identify access policies that govern a particular query. We assume that there may be multiple policies that govern a particular quad and thus envisage a simple conflict resolution strategy whereby either prohibitions override permissions or visa versa. The algorithm stops as soon as it finds a policy that permits the given query to be executed and returns the results of the query execution.

<font size="2">
<figure id="server-query-algorithm" class="listing">
````/code/access-control-algorithm.txt````
<figcaption markdown="block">
Querying with access control algorithm.
</figcaption>
</figure>
</font>

### Query Execution Over Privacy-Preserving Summaries

[](#figure-query-execution) shows an example of how our privacy preserving summaries can be used in client-side query engines.
The presented high level architecture should be seen as a basis for federated querying over decentralised environments with private data, where there is a single aggregator, and all sources we want to query over are considered by the aggregator.
In practice, multiple aggregators can exist,
they may apply to overlapping sources,
and some sources may not be aggregated at all.
For these cases, extensions to this algorithm will be needed,
which we consider out-of-scope for this work.

<figure id="figure-query-execution">
<img src="img/query-execution.svg" alt="[Query execution over privacy-preserving summaries]" class="figure-width-twothird">
<figcaption markdown="block">
Federated query execution using a privacy-preserving summary.
</figcaption>
</figure>

<br/><br/>
