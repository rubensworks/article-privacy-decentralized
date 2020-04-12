## A Personalized Address Book Use Case Scenario

{:#use-case}

Before presenting our privacy preserving federation framework we first provide a high level overview of our personalized address book use case scenario,
where the address book is merely a list of WebIDs,
and the actual details of each contact is stored in their own respective pod.
To keep this use case simple, we assume an address book of Alice that contains two contacts: Bob and Carol.
In practise, such an address book could contain many more contacts.
Alice has chosen to make this address book public,
so that everyone is able to see everyone she knows,
albeit without necessarily having access to everyone's private contact details as these are controlled via separate access control policies.
We also consider Dave as a fourth person that has no relationship with anyone else.

For the sake of simplicity, we consider three hierarchical authorization roles per pod,
where the members of each role can be configured for each pod:

- _R1_: Everyone (without authentication)
- _R2_: Acquaintances (inherits from _R1_)
- _R3_: Friends (inherits from _R2_)

{:.comment data-author="SS"}
↑ Later on (i.e., [](#access-policy-specification)) we use `Rs` to indicate `Rules`. Also this might be focusing too much on the **Role** part, rather than roles being just one of the aspects of a policy? i.e. policy `p` consisting of a set of rules `R` where each rule `r` is represented as a tuple `r = ⟨t, s, a, o⟩` with `t ∈ {perm,proh}` specifying the `type` of the rule, i.e. whether it is permitted (`perm`) or prohibited (`proh`) for requester `s` to perform action `a` on asset `o`. As such, we could then reformulate those to:

{:.comment data-author="SS"}

- Bob:

  - Bob is quite liberal, and allows everyone to read both his name and email.
    - `r1`<sub>`B`</sub> `= ⟨perm, Everyone, read, {name, email}⟩`
  - His telephone number is however only readable for friends.
    - `r2`<sub>`B`</sub> `= ⟨perm, Friends, read, {telephone number}⟩`
  - Bob considers Alice a friend
    - `Alice ∈ Friends, Friends ⊆ Acquaintances ⊆ Everyone`

- Carol:
  - Carol only allows her name to be read by the public
    - `r1`<sub>`C`</sub> `= ⟨perm, Everyone, read, {name}⟩`
  - Her email is only readable by acquaintances
    - `r2`<sub>`C`</sub> `= ⟨perm, Acquaintances, read, {email}⟩`
  - Her telephone number is readable by friends only
    - `r3`<sub>`C`</sub> `= ⟨perm, Friends, read, {telephone number}⟩`
  - Carol considers Alice a acquaintance
    - `Alice ∈ Acquaintances, Friends ⊆ Acquaintances ⊆ Everyone`

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
