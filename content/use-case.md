## A Personalized Address Book Use Case Scenario
{: #use-case}

Before presenting our privacy-preserving federation framework we first provide a high level overview of our personalized address book use case scenario,
where the address book is merely a list of WebIDs,
and the actual details of each contact is stored in their own respective pod.
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



<!--
{::options parse_block_html="true" /}
<div class="bs-callout bs-callout-info">
  <strong>Roles, Rules and Confusion</strong>\\
  Later on (i.e., [](#access-policy-specification)) we use `R1C, R2B, etc.` to indicate `Rules` of Carol, Bob,... Also, the current description above may be focusing too much on the **Role** part, rather than roles being just one of the aspects of a policy.. What about something along the lines of:

- a policy `p` consisting of a set of rules `R` where each rule `r` is represented as \\
  a tuple `r = ⟨t, s, a, o⟩` with
  - `t ∈ {perm,proh,obl}` - specifying the `type` of the rule \*
  - `s` - the party/user/group... to which this rule applies to (WHO)
  - `a` - the action/mode/... to be executed (HOW)
  - `o` - the resource/asset/asset group/... in question (WHAT)

\* TBD → only required if we want to add the concept of _obligations_ at some point too.. otherwise, every `r` would represent an explicit permission, thus if there's no applicable rule for a request the request should be denied (prohibition)

</div>

{::options parse_block_html="false" /} -->



[](#figure-use-case) shows a detailed overview of this use case.
Alice uses the `/contacts` file in her pod to list everyone that she knows using their WebID,
which point to the profiles of the respective people.
The profiles of Bob and Carol both contain their name, email and telephone number,
which are readable for select people.
Bob is quite liberal, and allows everyone (_R1<sub>B</sub>_) to read both his name and email.
His telephone number is however only readable for friends (_R3<sub>B</sub>_).
Bob considers Alice a friend (_R1<sub>B</sub>_, _R2<sub>B</sub>_, _R3<sub>B</sub>_).
Carol only allows her name to be read by the public (_R1<sub>C</sub>_),
her email is only readable by acquaintances (_R2<sub>C</sub>_),
and her telephone number by friends (_R3<sub>C</sub>_).
Carol considers Alice an acquaintance (_R1<sub>C</sub>_, _R2<sub>C</sub>_).

{:.comment data-author="SS"}
should we move the rules from below the profile to above of it, surrounded by an `Access Policy` block like in [](#figure-request-processing)?



**Bob is quite liberal, and allows everyone to read both his name and email.**

- \$$r1_B = \langle\{s \vert s \in S_E\}, read, \{o \vert o \in File_C \wedge o \subseteq \{name, email\}\}\rangle$$

**His telephone number is however only readable for friends.**

- `r2`<sub>`B`</sub> `= ⟨{s | s ∈ Friends`<sub>`C`</sub>`}, read, {o | o ∈ File`<sub>`Friends`<sub>`B`</sub></sub>`∧ o ⊆ {telephone number}⟩`

**Bob considers Alice a friend**

- `Alice ∈ Friends`<sub>`B`</sub>`, Friends`<sub>`B`</sub>`⊆ Acquaintances`<sub>`B`</sub>`⊆ Everyone`
- `{s | s ∈ Everyone} ⊆ ε ∧ ε ⊆ {s | s ∈ Everyone}` ?

**Carol allows only her name to be read by the public**

- `r1`<sub>`C`</sub> `= ⟨{s | s ∈ Everyone}, read, {o | o ∈ File`<sub>`Everyone`</sub>`∧ o ⊆ name}⟩`

**Her email is only readable by acquaintances**

- `r2`<sub>`C`</sub> `= ⟨{s | s ∈ Acquaintances`<sub>`C`</sub>`}, read, {o | o ∈ File`<sub>`Acquaintances`<sub>`C`</sub></sub>`∧ o ⊆ email}⟩`

**Her telephone number is readable by friends only**

- `r3`<sub>`C`</sub> `= ⟨{s | s ∈ Friends`<sub>`C`</sub>`}, read, {o | o ∈ File`<sub>`Friends`<sub>`C`</sub></sub>`∧ o ⊆ telephone number}⟩`

**Carol considers Alice a acquaintance**

- `Alice ∈ Acquaintances`<sub>`C`</sub>`, Friends`<sub>`C`</sub>`⊆ Acquaintances`<sub>`C`</sub>`⊆ Everyone`
- `{s | s ∈ Everyone} ⊆ ε ∧ ε ⊆ {s | s ∈ Everyone}` ?



<figure id="figure-use-case">
<img src="img/use-case.svg" alt="[Personal Address Book]" class="figure-width-twothird">
<figcaption markdown="block">
Overview of the personalized address book use case where Alice, Bob and Carol each have separate data pods.
Alice has an address book that contains links to the profiles of Bob and Carol.
All triples in each profile are annotated with a role for users that can read that triple.
Full lines indicate data reading by people,
and dashed lines indicate data links.
</figcaption>
</figure>



For this use case, we consider the following example queries:

1. **Alice wants to send an email to everyone she knows.**
   <br />
   Alice is able to read the email of both Bob and Carol.
   Because Bob's email is readable for everyone (_Alice ∈ R1<sub>B</sub>_),
   and Carol's email is readable for acquaintances (_Alice ∈ R2<sub>C</sub>_).
2. **Alice wants to call everyone she knows.**
   <br />
   Alice is able to read the telephone number of Bob, but not Carol.
   Because Bob's telephone number is readable for friends (_Alice ∈ R3<sub>B</sub>_),
   but Carol's telephone number is only readable for her friends (_Alice ∉ R2<sub>C</sub>_).
3. **Dave wants to send an email to everyone Alice knows.**
   <br />
   Dave is able to read the email of Bob, but not Carol.
   Because Bob's email is readable for everyone (_Dave ∈ R1<sub>B</sub>_),
   and Carol's email is only readable for acquaintances (_Dave ∉ R2<sub>C</sub>_).
4. **Dave wants to call everyone Alice knows.**
   <br />
   Dave is not able to read the telephone number of Bob or Carol.
   Because Bob's telephone number is readable for friends (_Dave ∉ R3<sub>B</sub>_),
   but Carol's telephone number is only readable for her friends (_Dave ∉ R3<sub>C</sub>_).

SABRINA check use case and examples are aligned
{:.todo}
