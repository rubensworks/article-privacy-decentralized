## Use Case Scenario
{: #use-case}

Before presenting our privacy-preserving federation framework we first provide a high level overview of the personalised address book use case scenario used to guide our work.
Following the Solid design principles, address books are merely lists of WebIDs, and the actual contact details are stored in the respective contacts' pod.
To keep this use case simple, we assume an address book of Alice that contains two contacts: Bob and Carol.
In practise, such an address book could contain many more contacts.
Alice has chosen to make this address book public,
so that everyone is able to see everyone she knows,
albeit without necessarily having access to everyone's private contact details as these are controlled via separate access control policies.
We also consider Dave as a fourth person that has no relationship with anyone else.

For the sake of simplicity, we consider three hierarchical subject groups per pod,
where the members of each group can be configured for each pod:

- $$S_E$$: Everyone (without authentication)
- $$S_A$$: Acquaintances ($$S_A \subseteq S_E$$)
- $$S_F$$: Friends ($$S_F \subseteq S_A$$)

The following statements indicate that Carol considers Alice to be an acquaintance, and Bob considers Alice as a friend:

_`<https://alice.pods.org/profile#me>` ∈ S<sub>A</sub> ⊆ S<sub>E</sub>_
{: style="text-align: center"}

_`<https://alice.pods.org/profile#me>` ∈ S<sub>F</sub> ⊆ S<sub>A</sub> ⊆ S<sub>E</sub>_
{: style="text-align: center"}

The data stored in Alices, Bobs and Carols pods, and the various access control rules are depicted in [](#figure-use-case).

<figure id="figure-use-case">
<img src="img/use-case.svg" alt="[Personal Address Book]" class="figure-width-twothird">
<figcaption markdown="block">
An overview of the proposed personalised address book use case scenario.
</figcaption>
</figure>

Alice uses the `/contacts` file in her pod to list everyone that she knows using WebIDs that point to the profiles of the respective people.
The profiles of Bob and Carol both contain their name, email and telephone number, which are readable for select people. Access control rules in the form of <em> <subject, access rights, resource></em> tuples represent the various access control policies:

Bob is quite liberal, and allows everyone (_S<sub>E</sub>_) to read both his name and email:
{: #r1}
<em>r<sub>1</sub> = ⟨{s | s ∈ S<sub>E</sub>}, read, {q | q ∈ Profile<sub>B</sub> ∧ q.predicate ∈ {:name,:email}}⟩</em>
{: style="text-align: center"}

Bobs telephone number can only be accessed by friends:
{: #r2}
<em>r<sub>2</sub> = ⟨{s | s ∈ S<sub>F</sub>}, read, {q | q ∈ Profile<sub>B</sub> ∧ q.predicate ∈ {:telephone}}⟩</em>
{: style="text-align: center"}


Carol only allows her name to be read by the public:
{: #r3}
<em>r<sub>3</sub> = ⟨{ s | s ∈ S<sub>E</sub>}, read, {q | q ∈ Profile<sub>C</sub> ∧ q.predicate ∈ {:name}}⟩</em>
{: style="text-align: center"}

Carols email is readable by acquaintances:
{: #r4}
<em>r<sub>4</sub> = ⟨{ s | s ∈ S<sub>A</sub>}, read, {q | q ∈ Profile<sub>C</sub> ∧ q.predicate ∈ {:email}}⟩</em>
{: style="text-align: center"}

Carols telephone number can only be accessed by friends:
{: #r5}
<em>r<sub>5</sub> = ⟨{ s | s ∈ S<sub>F</sub> }, read, { q | q ∈ Profile<sub>C</sub> ∧ q.predicate ∈ {:telephone}}⟩</em>
{: style="text-align: center"}


For this use case, we consider the following example queries:

1. **Alice wants to send an email to everyone she knows.**
   <br />
   Alice is able to read the email of both Bob and Carol.
   Because Bob's email is readable for everyone ([_r1_](#r1)),
   and Carol's email is readable for acquaintances ([_r4_](#r4)).
2. **Alice wants to call everyone she knows.**
   <br />
   Alice is able to read the telephone number of Bob, but not Carol.
   Because Bob's telephone number is readable for friends ([_r2_](#r2)),
   but Carol's telephone number is only readable for her friends ([_r5_](#r5)).
3. **Dave wants to send an email to everyone Alice knows.**
   <br />
   Dave is able to read the email of Bob, but not Carol.
   Because Bob's email is readable for everyone ([_r1_](#r1)),
   and Carol's email is only readable for acquaintances ([_r4_](#r4)).
4. **Dave wants to call everyone Alice knows.**
   <br />
   Dave is not able to read the telephone number of Bob or Carol.
   Because Bob's telephone number is readable for friends ([_r2_](#r2)),
   but Carol's telephone number is only readable for her friends ([_r5_](#r5)).
