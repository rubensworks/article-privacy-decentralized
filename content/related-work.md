## Related Work
{:#related-work}

In this section we survey the state of the art with respect to federated query processing and access control for the Resource Description Framework.

<br/>
**Access Control.**
[Web Access Control (WAC)](cite:cites spec:wac) is an RDF vocabulary and an access control framework, which demonstrates how together WebID and access control policies specified using the WAC vocabulary, can be used to enforce distributed access control. [Villata et al.](cite:cites Villata2011) and [Sacco and Passant](cite:cites Sacco2011b) extend the WAC vocabulary to cater for context based access control policies and privacy preferences respectively. Alternatively, [Encryption-Based Access Control](cite:cites fernandez2017self) involves encrypting RDF fragments (i.e. subjects, predicates, objects, graphs or some combination thereof) with an encryption key, such that only those that have the key are permitted to access the data, thus serving as both an authentication and an authorisation mechanism. Existing proposals involve using [symmetric encryption](cite:cites kasten2013towards), [public-key encryption](cite:cites giereth2005partial), or [functional encryption](cite:cites fernandez2017self) to generate RDF ciphers.
In this paper, encryption mechanisms are used to create privacy-preserving aggregations, whereas access control policies are used to restrict access to data at query time.
