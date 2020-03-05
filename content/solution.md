## A Possible Instantiation
{:#solution}

Summary of the section
{:.todo}

...go over the 3 aspects
1. AMFs meet all of these requirements (but can require multiple param-versions, AMF-negotiation?, LDN)
2. ...
3. SHACL/ShEx, encryption stuff
{:.todo}


### Privacy-Preserving Federated Querying

In this section, we discuss techniques for summarization,
suggestions on how to handle their different parameters,
and how communication between sources and aggregators could be handled.

**Summarization techniques**

The technical requirements for enabling federated querying in an efficient manner through privacy-preserving aggregators
are mainly driven by the summarization technology.
We consider Approximate Membership Functions (AMFs), such as Bloom filters or GCS
one possible candidate for such summaries that meet all of these requirements.
Concretely, they meet the requirements in the following manner:

1. **No data leaking**:
    Values in Bloom filters are hashed, which can not be reversed.
2. **Value additions**:
    Additions to Bloom filters are possible by inserting a bit string.
    `SummaryInitialize() = 0` and
    `SummaryAdd(Σ.c, q.c, k, u) = Σ.c | (q.c & k) | u`
3. **Summary combinations**
    Bloom filters can be combined by `OR`-ing them.
    `SummaryCombine(Σ.c, Σ'.c) = Σ.c | Σ'.c`.
4. **Authorized membership checking**
    Membership in Bloom filters can be tested by hashing the value,
    and checking its membership inside the filter.
    `SummaryContains(Σ.c, q.c, k, u) = Σ.c[(q.c & k) | u]`.

The main advantage of using AMFs such as Bloom filters
is that all of the performance-critical operations on summaries
(adding, combining, membership checking)
can happen very efficiently, as these are essentially just bitwise operations.

**AMF Parameter Handling**

When using AMFs, it is important to take account that these have certain parameters,
and that all operations must be known before any operation can be done with them.
For example, for Bloom filters the parameters are the number of hashes and bits.
These parameters and the number of entries all impact the false positive error rate.
Concretely, these parameters have to be identical before AMFs can be combined within an aggregator,
and they have to be known before a client can make use of them.

In a decentralized environment, it is however difficult to reach a consensus between all parties to use fixed parameters.
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

Concretely, AMFs can be used to efficiently implement privacy-preserving,
but it will require mechanisms to cope with their different parameters.

**Source-Aggregator Communication**

Different techniques are possible for triggering aggregated summary creation.
These techniques can be categorized in *pull-based* and *push-based* techniques.
Pull-based techniques require the aggregator to periodically poll the applicable sources.
Once the aggregator detects a change in one of the sources, the aggregated summary creation could restart.
Push-based techniques require the aggregators to *subscribe* to the sources,
after which the sources could *notify* the aggregators upon any change,
after which the aggregators could restart aggregated summary creation.
[Linked Data Notifications](cite:cites spec:ldn) is one possible technique for sending such notifications.
In practise, combinations of pull-based and push-based techniques may be valuable,
for instance when only some sources support change subscriptions,
which means that other sources will require polling.

### Shape Based Access Control
{:#solution-sac}

* Each user has a list of keys, keys are shared among users for same parts of data
* Explain key management (creation and where it's stored) and revocation
* Mention self-sovereign identity?
{:.todo}

Following the ... as outlined in [](cite:cites desiss:shapes), shapes languages such as [SHACL](cite:cites spec:shacl), specifically address the need to constrain the data in a graph to a certain shape

<!--
{:.todo}

two options:

* (shacl-spec): validation-based (~filter)
* (shacl note): filter/rule-based -->

<!-- (related https://github.com/solid/data-interoperability-panel/issues/34) -->


SIMON shapes (SHACL/ShEx) and [using shapes for Web APIs](cite:cites hypermedia_shapes)
{:.todo}

</figure>
<figure id="figure-request-processing">
<img src="img/request-processing_manualppt.svg" alt="[Shape-based access control]" style=" max-width: 60%; display: block; margin-left: auto; margin-right: auto;">
<figcaption markdown="block">
A server matches requests consisting of a client identifiation `i`, the requested access mode `a`, and a quad pattern query `q`, against a set of access control policies `P`.
A policy `p ∈ P` is applicable for a request `[i, a, q]` if the request conforms to the shape; policy `p` was specified against.
</figcaption>
</figure>

Instead of specifying ACLs explicitly for particular users and resources, shapes allow for constraining, i.e., shaping (i) the **resources** that can be accessed/should be returned, (ii) the **action/mode** that is permitted to be performed on the resource, and (iii) the **agents/party** whose requests the policy applies to. This allows for expressing more fine-grained access control policies, such as:

* All foaf:member of ex:Company1 which have at least 1 vcard:hasEmail, are permitted to perform acl:Read over all quads of File 1.
* Everyone is permitted to read rdf:type quads of File 1 and File 2

Eventually, `ProcessRequest` will return result set `R`.



{:.todo}
SIMON NOT FINISHED; have to fix it

<figure id="figure-acl-graph" markdown="block" style="background: #FFFFFF">

~~~ turtle

@prefix acl: <http://www.w3.org/ns/auth/acl#> .
@prefix odrl: <http://www.w3.org/ns/odrl/2/> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix sh: <http://www.w3.org/ns/shacl#> .

<http://alice.pod/share/policy>
    a sh:NodeShape ;
    sh:rule [
        a sh:SPARQLRule ;
        sh:construct """
            CONSTRUCT {
                ?s ?p ?o
            } WHERE {
                GRAPH <http://alice.pod/share/file1> {
                    ?s ?p ?o
                }
            }
        """ ;
        sh:condition [
            sh:property [
                sh:path odrl:assignee ; # the requesting agent
                sh:node [
                    a sh:PropertyShape ;
                    sh:path [ sh:inversePath foaf:member ] ;
                    sh:hasValue <http://company1/> ;
                ] ;
                sh:node [
                    a sh:PropertyShape ;
                    sh:path vcard:hasEmail ;
                    sh:minCount 1 .
                ]
            ] ;
            sh:property [
                sh:path odrl:action ; # the requested mode of access
                sh:hasValue acl:Read ;
            ] ;
        ] ;
    ] ;
~~~

<figcaption markdown="block">
ACL Policy
</figcaption>
</figure>

<!--
* Data owners are responsible for enforcing access control (as opposed to other approaches where federation engine takes care of that). We assume that access control is already taken care of at the (client-side) federation engine.
* Build on Solid's WebID-OIDC for auth, and WAC for access control.
* Allow shape-based/quadpattern-based (SHACL/SHEX) access modes in .acl files. (advantage of shapes is that fewer keys may be needed, which reduces the complexity of key mgmt) (motivation for keys is that Solid is going to use it for data validation: https://github.com/solid/data-interoperability-panel/blob/b2591bf2f8808972e5459db2aa4ac8d9854f5b5e/data-validation/use-cases.md)
* Right now, we do it role-based, but it could be attribute-based as well.
{:.todo} -->



