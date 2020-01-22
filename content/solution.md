## Possible Instantiations
{:#solution}

Summary of the section
{:.todo}

* Each user has a list of keys, keys are shared among users for same parts of data
* Explain key management (creation and where it's stored) and revocation
* Mention self-sovereign identity?
{:.todo}


...go over the 3 aspects
1. AMFs meet all of these requirements (but can require multiple param-versions, AMF-negotiation?, LDN)
2. ...
3. SHACL/ShEx, encryption stuff
{:.todo}


### Identity Management
[OpenID Connect](cite:cites OpenIDConnect) is an industry standard authentication protocol, which enables applications to delegate responsibility for authentication to third party identity providers. One of the primary benefits being the ability to connect to multiple sites using the same login credentials. A comprehensive security analysis of the protocol is provided by [Fett et al.](cite:cites fett2017web).

[Web Identity and Discovery (WebID)](cite:cites WebID) is a HTTP URI used to uniquely identify and [authenticate a person, company, organisation, or other entity](cite:cites Sambra2014). A description of the agent is provided in an RDF document, known as a WebID profile, which can be dereferenced using 303 or Hash URI's. The agent places URI for their WebID profile document in the Subject Alternative Names field of their certificate and the public key details to their [WebID profile document](cite:cites Inkster2014). The WebID Transport Layer Security (TLS) protocol specifies how together the WebID profile and public key certificates can be used to [authenticate users](cite:cites Inkster2014). A service wishing to  authenticate the user, needs to verify that the public key of the certificate it receives matches the public key specified in the [WebID profile](cite:cites Hollenbach2009).

[Self-sovereign identity (SSI)](cite:cites SSI) is a paradigm shift in terms of identity management, whereby individuals manage their own identity credential as opposed to relying on centralised identity providers, such as private or public sector organisations. [Mühle et al.](cite:cites muhle2018survey) provides a high level overview of the various components that are necessary to support SSI. Key supporting technology includes, verifiable claims which as claims that can be verified via a digital signature, and blockchain technology which plays the role of the third party identity provider.

### Shape Based Access Control


two options:
* (shacl-spec): validation-based (~filter)
* (shacl note): filter/rule-based
{:.todo}


(related https://github.com/solid/data-interoperability-panel/issues/34)
* Constrain the **resource** that can be accessed/is returned
* Constrain the **action/mode** that can be performed on the data
* Constrain the **agents/party** the policy applies to.
{:.todo}

<figure id="figure-acl-graph" markdown="block" style="background: #FFFFFF">

~~~ turtle

@prefix acl: <http://www.w3.org/ns/auth/acl#> .
@prefix odrl: <http://www.w3.org/ns/odrl/2/> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix sh: <http://www.w3.org/ns/shacl#> .

<http://alice.pod/share/policy>
    a odrl:Policy ;
    odrl:conflict odrl:perm ;
    odrl:prohibition [
        odrl:target <http://alice.pod/share/photoAnnotations/> ;
        odrl:action odrl:Read
    ] ;
    odrl:permission [
        odrl:target [
            a odrl:AssetCollection ;
            odrl:source <https://alice.databox.me/docs/file1> ;
            odrl:refinement [ # refine the asset/resource
                a sh:NodeShape ;
                sh:pattern "public" ;
                sh:flags   "i"
            ]
        ] ;
        odrl:action acl:Read ;
    ] ;
    odrl:permission [
        odrl:target <http://alice.pod/share/photoAnnotations/> ;
        odrl:action [
            rdf:value acl:Read ;
            odrl:refinement <#AccessRulesShape> # refine the action
        ]
    ] ;
    odrl:permission [
        odrl:target <http://alice.pod/share/photoAnnotations/> ;
        odrl:action acl:Read ;
        odrl:assignee [
            a foaf:Agent ;
            odrl:source <https://alice.example.com/work-groups#Accounting>;
            odrl:refinement [ # refine the agent
                a sh:NodeShape ;
                sh:property [
                    sh:path foaf:interest ;
                    sh:minCount 1 ;
                    sh:hasValue <#photography>
                ]
            ]
        ]
    ] .

~~~

<figcaption markdown="block">
ACL Policy
</figcaption>
</figure>


----------
<figure id="figure-shapes-graph" markdown="block" style="background: #FFFFFF">

~~~ turtle

# Shapes Graph

<#AccessRulesShape>
    a sh:NodeShape ;
    sh:targetObjectsOf ldp:contains  ;
    sh:rule [
        a sh:SPARQLRule ;
        sh:construct """
            CONSTRUCT WHERE {
                $this a ?o .
            }
        """ ;
        sh:condition <#PrivateAuthorizationShape> ;
    ] ;
    sh:rule [
        a sh:SPARQLRule ;
        sh:construct """
            CONSTRUCT WHERE {
                $this ?p ?o .
            }
        """ ;
        sh:condition <#PublicAuthorizationShape> ;
    ] .

<#MemberPropertyShape>
    a sh:PropertyShape ;
    sh:path [ sh:inversePath ldp:contains ] ;
    sh:hasValue  <http://alice.pod/share/photoAnnotations/> ;
    sh:minCount 1 .

<#PrivateAuthorizationShape>
    a sh:NodeShape ;
    sh:targetObjectsOf ldp:contains  ;
    sh:pattern "private" ;
    sh:flags   "i" ;
    sh:property <#MemberPropertyShape> .

<#PublicAuthorizationShape>
    a sh:NodeShape ;
    sh:targetObjectsOf ldp:contains  ;
    sh:pattern "public" ;
    sh:flags   "i" ;
    sh:property <#MemberPropertyShape> .

~~~

<figcaption markdown="block">
Shapes Graph
</figcaption>
</figure>

----------
<figure id="figure-data-graph" markdown="block" style="background: #FFFFFF">

~~~ turtle

# Data Graph
<http://alice.pod/share/photoAnnotations/> a ldp:BasicContainer ;
    ldp:contains <public/annot123>, <private/annot456>.

<public/annot123> a photo:Annotation ;
    dc:author "Alice" ;
    photo:photo <http://amy.pod/photos/photo123.jpg> ;
    photo:caption "Alice and Amy at work" .

<private/annot456> a photo:Annotation ;
    dc:author "Alice" ;
    photo:photo <http://bob.pod/photos/photo456.jpg>;
    photo:caption "Bob sleeping at work" .

~~~

<figcaption markdown="block">
Data Graph
</figcaption>
</figure>


* Data owners are responsible for enforcing access control (as opposed to other approaches where federation engine takes care of that). We assume that access control is already taken care of at the (client-side) federation engine.
* Build on Solid's WebID-OIDC for auth, and WAC for access control.
* Allow shape-based/quadpattern-based (SHACL/SHEX) access modes in .acl files. (advantage of shapes is that fewer keys may be needed, which reduces the complexity of key mgmt) (motivation for keys is that Solid is going to use it for data validation: https://github.com/solid/data-interoperability-panel/blob/b2591bf2f8808972e5459db2aa4ac8d9854f5b5e/data-validation/use-cases.md)
* Right now, we do it role-based, but it could be attribute-based as well.
{:.todo}



### Privacy-Preserving Federated Querying

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

<!-- #### AMF Parameter Handling -->

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




