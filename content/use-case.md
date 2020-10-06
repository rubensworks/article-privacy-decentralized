## Motivating Use Case Scenario
{: #use-case}


Following the Solid design principles, in the personalised address book use case scenario used to guide our work, address books are merely lists of WebIDs, and the actual contact details are stored in the respective contacts' pod.
To keep this use case simple, we assume an address book of Alice that contains two contacts: Bob and Carol.
In practise, such an address book could contain many more contacts.
Alice has chosen to make this address book public,
so that everyone is able to see everyone she knows,
albeit without necessarily having access to everyone's private contact details as these are controlled via separate access control policies.
We also consider Dave as a fourth person that has no relationship with anyone else.


For the sake of simplicity, we consider three hierarchical subject access groups (i.e., Everyone, Acquaintances, Friends) per pod (identified  by a unique id), where the members of each group can be configured for each pod. An assertion, of the following form, could be used to indicate that Carol considers Alice to be an acquaintance:

_`<https://alice.pods.org/profile#me>` $$\in$$ S<span class="supsub"><sup>A</sup><sub><i>Carol</i></sub></span> $$\subseteq$$ S<span class="supsub"><sup>E</sup><sub><i>Carol</i></sub></span>_
{: style="text-align: left"}

An assertion, of the following form, could be used to indicate that Bob considers Alice as a friend:

_`<https://alice.pods.org/profile#me>` $$\in$$ S<span class="supsub"><sup>F</sup><sub><i>Bob</i></sub></span> $$\subseteq$$ S<span class="supsub"><sup>A</sup><sub><i>Bob</i></sub></span> $$\subseteq$$ S<span class="supsub"><sup>E</sup><sub><i>Bob</i></sub></span>_
{: style="text-align: left"}

The data stored in pods maintained by Alice, Bob and Carol, and the various access control rules are depicted in [](#figure-use-case).

<figure id="figure-use-case">
<img src="img/use-case.svg" alt="[Personal Address Book]" class="figure-width-twothird">
<figcaption markdown="block">
An overview of the proposed personalised address book use case scenario.
</figcaption>
</figure>

Alice uses the `/contacts` file in her pod to list everyone that she knows using WebIDs that point to the profiles of the respective people.
The profiles of Bob and Carol both contain their name, email and telephone number, which are readable for select people.

Generally speaking, an access control policy is composed of one or more authorisations that are used to state that a given subject (e.g. a user, role, group) is permitted to perform an action (e.g. read, write, share) on a certain object (e.g. dataset, file, graph). Policies are often composed of sets of positive authorisations (permissions) of the form <em>&lt;subject, action, object&gt;</em>. Authorisations of this form could be used by Bob and Carol to restrict access to data stored in their respective pods:

Bob is quite liberal, and allows everyone (_S<span class="supsub"><sup>E</sup><sub><i>Bob</i></sub></span>_) to read both his name and email, however his telephone number can only be accessed by friends (_S<span class="supsub"><sup>F</sup><sub><i>Bob</i></sub></span>_):
{: #r1}
<em>r<sub>1</sub> = ⟨{s | s ∈ S<span class="supsub"><sup>E</sup><sub><i>Bob</i></sub></span>}, read, {q | q ∈ Profile<sub>Bob</sub> ∧ q.predicate ∈ {:name,:email}}⟩</em>
{: style="text-align: left"}
{: #r2}
<em>r<sub>2</sub> = ⟨{s | s ∈ S<span class="supsub"><sup>F</sup><sub><i>Bob</i></sub></span>}, read, {q | q ∈ Profile<sub>Bob</sub> ∧ q.predicate ∈ {:telephone}}⟩</em>
{: style="text-align: left"}


Carol only allows her name to be read by the public (_S<span class="supsub"><sup>E</sup><sub><i>Carol</i></sub></span>_), her email can be read by acquaintance (_S<span class="supsub"><sup>A</sup><sub><i>Carol</i></sub></span>_), however her telephone number can only be accessed by friends (_S<span class="supsub"><sup>F</sup><sub><i>Carol</i></sub></span>_):
{: #r3}
<em>r<sub>3</sub> = ⟨{ s | s ∈ S<span class="supsub"><sup>E</sup><sub><i>Carol</i></sub></span>}, read, {q | q ∈ Profile<sub>Carol</sub> ∧ q.predicate ∈ {:name}}⟩</em>
{: style="text-align: left"}
{: #r4}
<em>r<sub>4</sub> = ⟨{ s | s ∈ S<span class="supsub"><sup>A</sup><sub><i>Carol</i></sub></span>}, read, {q | q ∈ Profile<sub>Carol</sub> ∧ q.predicate ∈ {:email}}⟩</em>
{: style="text-align: left"}
{: #r5}
<em>r<sub>5</sub> = ⟨{ s | s ∈ S<span class="supsub"><sup>F</sup><sub><i>Carol</i></sub></span>}, read, { q | q ∈ Profile<sub>Carol</sub> ∧ q.predicate ∈ {:telephone}}⟩</em>
{: style="text-align: left"}
