## Framework
{:#framework}

### Access Control Policies

* Data owners are responsible for enforcing access control (as opposed to other approaches where federation engine takes care of that). We assume that access control is already taken care of at the (client-side) federation engine.
* Build on Solid's WebID-OIDC for auth, and WAC for access control.
* Allow shape-based/quadpattern-based (SHACL/SHEX) access modes in .acl files.
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
{:.todo}
