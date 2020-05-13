## Challenges and Opportunities
{:#solution}

When it comes to access policy specification, summarisation generation and maintenance, source selection, and access control enforcement there are several open challenges and opportunities.

### Access Policy Specification

We assume that pod owners need to be able to specify access control policies that can be enforced both from an indexing and also a query processing perspective, taking into consideration the following core requirement:

- **No data leaking**: Access restricted data must not be available to those who are not authorised to access it.

In order to support privacy-preserving summaries, there is a need to generate access keys for both the acquaintances and the friends files, such that the summary generation process does not work with plain text attributes but rather cipher text. Given that data in the everyone file is public by default, no key is needed.

In this paper, we propose that there is a many to many mapping between quads and policies and a one to one mapping between access policies (enforced at query times) and access keys (used to create privacy preserving data summaries). Our initial proposal makes use of simple symmetric keys, however for more complex scenarios both attribute-based encryption and/or key derivation algorithms could be use to provide support for more complex access policies.

When to comes to policy management, there is a need to ensure that (i) access keys are tightly bound to access policies, and (ii) said keys are distributed to authorised individuals (i.e. acquaintances and friends). In order to revoke access to a particular individual one would need to regenerate the keys and redistribute them to authorised individuals.

### Summary Generation and Maintenance

The technical requirements for enabling federated querying in an efficient manner through privacy-preserving aggregators are mainly driven by the summarisation technology. In this context symmetric keys are used to create privacy-preserving summaries that do not leak access restricted data.

We consider Approximate Membership Functions (AMFs), such as Bloom filters, being
one possible candidate for such summaries that meet the following requirements:

- **No data leaking.**:
  Values in Bloom filters are hashed, which can not be reversed.

- **Value additions.**:
  Additions to Bloom filters are possible by inserting a bit string.
  `SummaryInitialize() = 0` and
  `SummaryAdd(Σ.c, q.c, k, u) = Σ.c | (q.c & k) | u`

- **Summary combinations.**
  Bloom filters can be combined by `OR`-ing them.
  `SummaryCombine(Σ.c, Σ'.c) = Σ.c | Σ'.c`.

The main advantage of using AMFs such as Bloom filters is that all of the performance-critical operations on summaries (adding, combining, membership checking) can happen very efficiently, as these are essentially just bitwise operations.

#### Parameter Handling

When using AMFs, it is important to consider that certain parameters need to be configured,
and that all operations must be known before they can be operationalised.
For example, for Bloom filters the parameters are the number of hashes and bits.
These parameters and the number of entries all impact the false positive error rate.
Concretely, the parameters used to setup individual summaries need to be identical such that they can be combined by an aggregator.

In a decentralised environment, it is however difficult to reach a consensus between all parties with respect to fixed parameters.
Furthermore, since the number of values within an AMF has an impact on the error rate,
it is sometimes even required to use different parameters for different numbers of entries.
As such, no single set of AMF parameters should be used.
This does however mean that a parameter determination mechanism is needed for aggregators
that want to combine multiple AMFs.
This mechanism is driven by the file sizes within each data pod,
and by the number of sources the aggregator works over.
Once the aggregator has determined these numbers,
it can determine an appropriate set of AMF parameters,
which must then be sent to each data pod to trigger an AMF creation for the requested file under the given set of parameters.
In cases where many aggregators request such AMFs,
the data pods may decide to only support a fixed set of parameters to reduce the load on them, and to enable caching of AMFs.
For this, some kind of parameter negotiation may be required between the aggregator and all data pods.

#### Maintenance of Summaries within Sources

Since the creation of an AMF for a file can become expensive,
sources may decide to adopt different maintenance strategies.
Within this work, we discuss four approaches:

1. **Eager AMF creation.**:
   <br />
   For each file, an AMF is created as soon as a change occurs in the file.
   This ensures that the AMFs are always synchronised with the state of the files,
   and requests for AMFs will always return instantly.
   The downside of this approach is that frequently changing files may cause more AMF updates than needed,
   even when certain AMFs may not be used that often.
2. **Transient AMF creation.**:
   <br />
   When the AMF of a file is requested, it is created on-the-fly.
   This means that the server does not store any AMFs physically,
   but only constructs them when needed.
   The advantages of this is that AMFs are always synchronised with the state of the files.
   The main disadvantage is that requests for AMFs are slowed down by AMF creation time,
   which may be significant for larger files.
3. **Lazy AMF creation.**:
   <br />
   This approach combines the two previous approaches.
   Each AMF will only be created and stored from the moment that it is requested.
   From the moment that a file change is detected, the stored AMF is invalidated,
   which will cause a recalculation when it will be requested again.
   The advantages of this approach are that AMFs are always synchronised with the state of the files,
   and that even when files change frequently, no unneeded AMF creations will occur.
   The downside is that requests for AMFs are sometimes slowed down by AMF creation time.
4. **Periodic AMF creation.**:
   <br />
   This approach involves creating AMFs for files at a following a certain frequency,
   for example each hour, or each day.
   The advantage of this approach is that requests for AMFs will always return instantly.
   The downsides are that AMFs may not always be synchronised with the state of the files,
   and that more AMF updates occur than needed.

In practise, we expect that different approaches may be valuable for different types of use cases.
For example, for sources that contain static or slowly changing files,
the eager and periodic approaches may be beneficial.
But for sources that contain very volatile files,
the transient and lazy approaches may work better.
Furthermore, some of these approaches may be combined with each other.
For example, the lazy approach could be combined with the periodic approach
so that for small files the regular lazy approach is followed,
but for larger files, AMFs are periodically created to avoid slowing down requests for these AMFs.
One open challenge for volatile files is to investigate incremental AMF creation approaches
where only small parts of the data is added or removed.
For instance, Bloom filters allow data to be appended, but not removed.

#### Source-Aggregator Communication

Different _push_ and _pull_ based techniques can be used to trigger aggregated summary creation.
Push based techniques require the aggregators to _subscribe_ to the sources, and the sources could _notify_ the aggregators upon any change, at which point the aggregators could restart the aggregated summary creation process.
Pull based techniques require the aggregator to periodically poll the applicable sources,
which could be done efficiently by sending `HEAD` requests and checking the last modification date of the files or summaries in the response headers.
Once the aggregator detects a change in one of the sources, the aggregated summary creation could restart.
[Linked Data Notifications](cite:cites spec:ldn) is one possible technique for sending such notifications. In practise, there is a need to support both push and pull based techniques, for instance when only some sources support change subscriptions, which means that other sources will require polling.

### Source Selection

In the proposed framework, a client-side query engine can make use of the aggregator's summary to perform source selection,
in order to reduce the number of sources that are being consulted by this engine.
As these summaries allow source selection based on quad patterns instead of full SPARQL queries,
source selection can be pushed down into the query plan,
which allows quad patterns in the query to be executed over a different range of sources.
Furthermore, instead of applying source selection before query execution,
this allows source selection to optionally happen adaptively during query execution,
following the federation algorithm of [Triple Pattern Fragments](cite:cites ldf).
A hybrid approach where source selection happens both before and during query execution could be investigated.

From a source selection perspective, we address the following core requirement:

- **Authorized membership checking.**
  Membership in Bloom filters can be tested by hashing the value,
  and checking its membership inside the filter.
  `SummaryContains(Σ.c, q.c, k, u) = Σ.c[(q.c & k) | u]`.


One open challenge will be to investigate how this file-based source selection method could be combined and enhanced
by existing source selection methods for SPARQL endpoints, such as [Hibiscus](cite:cites hibiscus) and [Splendid](cite:cites splendid).
Another open challenge is the automatic discovery of applicable aggregators by clients.
For now, we assume that clients will have zero or more preconfigured links to certain aggregators,
such as an aggregator for a person's family, or an employee's company. More elaborate aggregator discovery mechanisms should be investigated, where for example an aggregator is discovered based on the topic of a certain query,
or based on the current location of the user.

### Query Execution

Since file-based APIs are the basis for data retrieval on the Web as prescribed by the HTTP protocol,
we assume this as a starting point for federated querying in decentralised environments.
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

Once the query engine has identified the data sources that could potentially contribute results to their query, the query engine needs to authenticate to the server(s) and execute the query or parts thereof. In turn the server is responsible for authenticating the request, enforcing access control, and executing the query or parts thereof. Concretely, we address the following core requirement:

- **Query Execution with Access control.** It must be possible for the data source to verify the integrity of the requesting party and for the data source to limit query results based on a set of access policies.

#### Authentication

Web Identity and Discovery, is a mechanism used to uniquely identify and authenticate a person, company, organisation or other entity, by means of a URI. Essentially a WebID is a HTTP URI that should: (i) be under the control of the entity it describes; (ii) be linkable on the web; (iii) describe the entity is represents; (iv) enable authentication and access control; (v) respect the privacy of the entity it describes; (v) rely solely on HTTP and semantic Web technologies. A description of the agent is provided in an RDF document, known as a WebID profile, which can be dereferenced using 303 or Hash URIs. The WebID-TLS protocol (where TLS stands for Transport Layer Security) specifies how together the WebID profile and public key certificates, can be used to authenticate users. The user places their WebID profile document URI in the _Subject Alternative Names_ field of their certificate. Once the certificate has been generated the user adds the public key details to their WebID profile document. A service wishing to authenticate the user, needs to verify that the public key of the certificate it receives matches the public key specified in the WebID profile.

#### Authorisation

For access control purposes, we envision a mechanism that translates access policies (i.e. sets of authorisations) into constraints (e.g., data shapes like [SHACL](cite:cites spec:shacl)) against which requests and respective query results can then be validated against. The following policy could be used to allow read access to all _public_ URIs of `<http://alice.pod/share/photoAnnotations/>`

~~~ turtle
<http://alice.pod/policy:88>
    a odrl:Policy ;
    odrl:permission [
        odrl:target [
            a odrl:AssetCollection ;
            odrl:source <http://alice.pod/share/photoAnnotations/> ;
            odrl:refinement [
                a sh:NodeShape ;
                sh:pattern "public" ;
                sh:flags   "i" .
            ]
        ]
        odrl:action acl:Read;
    ] .
~~~

One of the key challenges relates to the management of access control policies and the associated access keys. Additionally, there is a need to investigate the overhead associated with increasingly granular access control policies (e.g., file based, pattern based, quad based).   
