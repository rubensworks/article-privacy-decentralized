## Related Work
{:#related-work}

Solid currently uses [Web Access Control (WAC)](cite:cites spec:wac),
which provides access control over _documents_.
For some use cases, where documents are large, and access is only granted to specific parts of the document,
WAC does not offer a sufficient granularity.
For this, [vocabulary extensions of WAC were introduced](cite:cites wac_ppo,wac_ext),
where more fine-grained access can be granted to specific RDF resources or named graphs,
but are not being used in practise. 

 {:.todo}
Emphasize that Solid *only* does WAC, not the extensions. So only access to full documents.
Also say that even the extensions are not fine-grained enough. 

For this work, we assume a decentralized environment as provided by Solid and its used standards
to illustrate our use case.
Furthermore, we will introduce extensions of the used standards to improve query execution
over multiple Solid data pods in a privacy-preserving manner. 



