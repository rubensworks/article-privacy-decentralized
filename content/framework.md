## Framework
{:#framework}

In order to execute queries on top of our use case,
a query engine needs to be able to perform several additional tasks compared to typical query execution scenarios.
First, since our data is spread over multiple sources, and we do not know all of these sources beforehand,
the query engine needs to be able to *traverse links* within each source, based on the given query.
Second, as some sources contain private data, the query engine needs to *authenticate* itself to the sources.
Third, the query engine needs to *federate* over the traversed sources to evaluate a given query.



* Explain high-level
* Emphasize that engine is client-side, including federation, but aggregators can be separate.
* For protocol: LDN can be used
{:.todo}

### Access Control Policies

* Data owners are responsible for enforcing access control (as opposed to other approaches where federation engine takes care of that). We assume that access control is already taken care of at the (client-side) federation engine.
* Build on Solid's WebID-OIDC for auth, and WAC for access control.
* Allow shape-based/quadpattern-based (SHACL/SHEX) access modes in .acl files. (advantage of shapes is that fewer keys may be needed, which reduces the complexity of key mgmt) (motivation for keys is that Solid is going to use it for data validation: https://github.com/solid/data-interoperability-panel/blob/b2591bf2f8808972e5459db2aa4ac8d9854f5b5e/data-validation/use-cases.md)
* Right now, we do it role-based, but it could be attribute-based as well.
{:.todo}

```
    <#authorization1>
        a             acl:Authorization;
        acl:agent     <https://alice.databox.me/profile/card#me>;  # Alice's WebID
        acl:accessTo  <https://alice.databox.me/docs/file1>;
        acl:mode      acl:Read, 
                      acl:Write, 
                      acl:Control,
                      [ a acl:FilteredRead;
                        sh:select """CONSTRUCT WHERE { ?subject a ?object } """ ].
```

### Policy Enforcement Policies

* Each user has a list of keys, keys are shared among users for same parts of data
* Explain key management (creation and where it's stored) and revocation
{:.todo}

### Privacy-Preserving Federated Querying

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

* Explain aggregation use case (aggregator for family, colleagues, personal, ...)
* Important to mention: client-side query evaluation, i.e., quad patterns are joined client-side.
* Important to mention: multiple aggregators can exist; aggregators are not necessarily trusted; each client could also have a personal aggregators.
* Aggregation is used to optimize federated querying, but not possible anymore, since aggregators may not see private data.
    A privacy-preserving aggregator may only see obfuscated data that 1) private data can not be read, 2) queries can only result in certain answers with appropriate keys
* Each source exposes AMFs for each (typically small) file (explain encoding of AMF)
* Aggregators combine AMFs, and keep list of source URIs
* Clients retrieve from aggregator combined AMF and list of sources
    Algo for each quad pattern:
        Foreach non-variable quad component c
            Check if AMF_c[c & key]
            Foreach source in sources
                If AMF_c[(c & key) | source]
                    Add source to valid_sources
        Do a federated query over valid_sources
* Mention that multiple AMF sizes are needed to limit error rates when aggregator aggregates over different numbers of sources. We could for example standardize some AMF params for _small_, _medium_ and _large_ aggregations.
{:.todo}
