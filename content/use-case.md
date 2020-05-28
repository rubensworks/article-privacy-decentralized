## Motivating Use Case Scenario
{: #use-case}

Following the Solid design principles, in the personalised address book use case scenario used to guide our work, address books are merely lists of WebIDs, and the actual contact details are stored in the respective contacts' pod.
To keep this use case simple, we assume an address book of Alice that contains two contacts: Bob and Carol.
In practise, such an address book could contain many more contacts.
Alice has chosen to make this address book public,
so that everyone is able to see everyone she knows,
albeit without necessarily having access to everyone's private contact details as these are controlled via separate access control policies.
We also consider Dave as a fourth person that has no relationship with anyone else.

For the sake of simplicity, we consider three hierarchical subject access groups per pod,
where the members of each group can be configured for each pod:
<em>S<sub>E</sub> : Everyone (without authentication)</em>,
<em>S<sub>A</sub> : Acquaintances (S<sub>A</sub> $$\subseteq$$ S<sub>E</sub>)</em>, and
<em>S<sub>F</sub> : Friends (S<sub>F</sub> $$\subseteq$$ S<sub>A</sub>)</em>.

Assertions, of the following form, could be used to indicate that Carol considers Alice to be an acquaintance, and Bob considers Alice as a friend:

_`<https://alice.pods.org/profile#me>` ∈ S<sub>A</sub> ⊆ S<sub>E</sub>_
{: style="text-align: left"}
_`<https://alice.pods.org/profile#me>` ∈ S<sub>F</sub> ⊆ S<sub>A</sub> ⊆ S<sub>E</sub>_
{: style="text-align: left"}

The data stored in Alices, Bobs and Carols pods, and the various access control rules are depicted in [](#figure-use-case).

<figure id="figure-use-case">
<img src="img/use-case.svg" alt="[Personal Address Book]" class="figure-width-twothird">
<figcaption markdown="block">
An overview of the proposed personalised address book use case scenario.
</figcaption>
</figure>

Alice uses the `/contacts` file in her pod to list everyone that she knows using WebIDs that point to the profiles of the respective people.
The profiles of Bob and Carol both contain their name, email and telephone number, which are readable for select people. Access control rules in the form of <em>&lt;subject, access rights, resource&gt;</em> tuples can be used by Bob and Carol to restrict access to data stored in their respective pods:

Bob is quite liberal, and allows everyone (_S<sub>E</sub>_) to read both his name and email, however his telephone number can only be accessed by friends:
{: #r1}
<em>r<sub>1</sub> = ⟨{s | s ∈ S<sub>E</sub>}, read, {q | q ∈ Profile<sub>B</sub> ∧ q.predicate ∈ {:name,:email}}⟩</em>
{: style="text-align: left"}
{: #r2}
<em>r<sub>2</sub> = ⟨{s | s ∈ S<sub>F</sub>}, read, {q | q ∈ Profile<sub>B</sub> ∧ q.predicate ∈ {:telephone}}⟩</em>
{: style="text-align: left"}


Carol only allows her name to be read by the public, her email can be read by acquaintance, however her telephone number can only be accessed by friends:
{: #r3}
<em>r<sub>3</sub> = ⟨{ s | s ∈ S<sub>E</sub>}, read, {q | q ∈ Profile<sub>C</sub> ∧ q.predicate ∈ {:name}}⟩</em>
{: style="text-align: left"}
{: #r4}
<em>r<sub>4</sub> = ⟨{ s | s ∈ S<sub>A</sub>}, read, {q | q ∈ Profile<sub>C</sub> ∧ q.predicate ∈ {:email}}⟩</em>
{: style="text-align: left"}
{: #r5}
<em>r<sub>5</sub> = ⟨{ s | s ∈ S<sub>F</sub> }, read, { q | q ∈ Profile<sub>C</sub> ∧ q.predicate ∈ {:telephone}}⟩</em>
{: style="text-align: left"}
