## Possible Instantiations
{:#solution}

{:.todo}
...go over the 3 aspects

1. AMFs meet all of these requirements (but can require multiple param-versions, AMF-negotiation?, LDN)
2. ...
3. SHACL/ShEx, encryption stuff

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
    `Summary_Add(S_C, v, k, u) = S_C | (v & k) | u`
3. **Summary combinations**
    Bloom filters can be combined by `OR`-ing them.
    `Summary_Combine(S_C, S_C') = S_C | S_C'`.
4. **Authorized membership checking**
    Membership in Bloom filters can be tested by hashing the value,
    and checking its membership inside the filter.
    `Summary_Contains(S_C, v, k, u) = S_C[(V & K) | u]`.

The main advantage of using AMFs such as Bloom filters
is that all of the performance-critical operations on summaries
(adding, combining, membership checking)
can happen very efficiently, as these are essentially just bitwise operations.

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

### Extension of Access Control

{:.todo}
Simon

### Extension of Identity Management

{:.todo}



Sabrina