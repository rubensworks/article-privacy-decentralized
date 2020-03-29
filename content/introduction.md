## Introduction
{:#introduction}

The Web was originally envisaged as a free, non-discriminatory decentralized information space built upon universal standards.  Although the Web brings major benefits as an economic, educational, and collaborative space, it also serves as a means for personal information harvesting, disinformation, and political manipulation. Additionally, in recent years, the Web has been dominated by a handful of centralized service providers commonly known as Google, Amazon, Facebook, Apple, or simply the GAFA. 

One potential solution to the existing problems is to enable individuals to take more control over their data in the form of personal data stores and to provide infrastructure that supports decentralised query processing. One of the leading efforts in this space is [Solid](cite:cites solid), which is a decentralized Web-based ecosystem that gives people more control over their data by enabling everyone to have a personal *data pod* to store their data, and by providing app developers with the infrastructure needed to develop applications that work over distributed data sources. By decoupling data from applications, individuals are afforded more control over how their personal data are processed. However, the number of data sources that need to be queried could potentially become very large, for instance in the case of a large social network. Thus giving rise to research into [data aggregators](cite:cites summaries) that can be used to improve query performance by reducing the number of sources that need to be requested for any given query.

One of the major limitations of current aggregation techniques, is the fact that they assume that all data is public, which is not realistic in many scenarios (e.g., in a social network context individuals may only wish to share a limited amount of data with acquaintances, while at the same time being able to share more data with friends). As such, query execution and aggregation techniques need to be extended to enable them to work with different access policies.
<div class="placeholder printonly">
<span style="display: block; height: 9em;"></span>
<!-- This is a dummy placeholder for the ACM first page footnote -->
</div>
Concretely, query engines need to be able to authenticate themselves to sources, such that only authorised data is returned. Furthermore, since aggregators may be untrusted third-parties, there is a need for privacy-preserving aggregation techniques, such that malicious aggregators are not able to access unauthorised data, but authorized query engines are still able to exploit them.

In this position paper, we propose a high-level framework that can be used to: (i) optimize federated querying through privacy-preserving aggregation; and (ii) enable federated querying with access control. <!--Our goal is *not* to introduce a complete solution to these problems, instead, our contribution is a framework that can be used to position future research.-->

The remainder of this article is structured as follows: In [](#background) we present relevant background in terms of querying and indexing.
In [](#use-case) we introduce the motivating use case that is used to guide our work.  Following on from this our framework is introduced in [](#framework) and a possible instantiation is presented in [](#solution).
Finally we conclude the article and discuss future work in [](#conclusions).
