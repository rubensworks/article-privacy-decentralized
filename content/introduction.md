## Introduction
{:#introduction}

{:.todo}
Aggregation is privacy-preserving, not querying.

The Web, as it was envisioned originally, is a free, *decentralized* platform.
However, in recent years, this vision of a decentralized Web has become less and less emphasized,
and the Web has been growing increasingly more *centralized*.
This has brought various problems,
such as the misuse of personal information for advertisement purposes,
and the manipulation of information to influence political elections.

For these reasons, there is a push for *re-decentralizing the Web*,
to give people back the ownership of their data.
One of the leading efforts for decentralization is [Solid](cite:cites solid),
which is a decentralized Web-based ecosystem that is lead by the Web's inventor, Tim Berners-Lee.
Solid enables true data ownership by allowing everyone to have a personal *data pod* to store their data.
Applications are detached from this data, and need to ask for permission to access their data.
Ensuring data ownership is also one of the goals of the European General Data Protection Regulation (GDPR),
where Solid can be an answer to this new regulation,
as has been [exemplified in governmental processes](cite:cites solid_gov).

While decentralization may solve many of the *social* problems we are currently experiencing,
it introduces several *technological* problems,
since applications now need to be able to query data that is spread over the Web,
instead of having it available via a centralized endpoint.
Due to the massive size of the Web, the number of documents that need to be queried
can become very large.
For example, visualizing a social network feed may require hundreds or thousands of sources.
In order to improve the performance of query execution,
[data aggregators](cite:cites summaries) can be used
as a way to reduce the range of sources to query over.

Current aggregation techniques assume that all data is public.
However, in decentralized environments such as Solid,
not all data is necessarily public,
as data can be configured to only be visible to some people.
For example, the name of someone may be visible for everyone,
but the telephone number may only be visible to friends.
As such, query execution and aggregation techniques need to be extended to enable them to work with private data.
Concretely, query engines need to be able to *authenticate* themselves to sources,
so that they can be *authorized* to see specific parts of the data.
Furthermore, since aggregators may be untrusted third-parties,
*privacy-preserving* aggregation techniques need to be introduced,
so that malicious aggregators are not able to read this data.

In this article, we share our vision on querying in decentralized environments
in privacy-preserving manner.
We discuss the state of the art,
identify the open challenges to enable privacy-preserving querying,
and we propose a high-level framework for tackling these challenges,
including a theoretical evaluation about its feasibility.
Our goal is *not* to introduce a complete solution to these problems,
instead, our contribution is a framework into which future research can be positioned.
Concretely, our framework focuses on three aspects:

1. Optimizing federated querying through privacy-preserving aggregation
2. Fine-grained access control
3. Decentralized identity management (to allow encoding of AMFs)

This article is structured as follows.
In the next section, we present the related work.
In [](#use-case), we introduce a guiding use case,
followed by the introduction of our framework in [](#framework).
Next, we evaluate our framework using an attacker model in [](#evaluation),
and we conclude in [](#conclusions).
