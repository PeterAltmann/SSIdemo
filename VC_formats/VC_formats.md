# Verifiable Credential formats

All VC's contain attributes about an identity subject. There exits multiple VC formats (for a detailed description, see the following links: [Young infographic](https://github.com/PeterAltmann/SSIdemo/blob/main/VC_formats/Verifiable-Credentials-Flavors-Explained-Infographic.pdf), [Young paper](https://github.com/PeterAltmann/SSIdemo/blob/main/VC_formats/Verifiable-Credentials-Flavors-Explained.pdf), and [W3C proof formats](https://www.w3.org/TR/vc-imp-guide/#proof-formats))

Below, when used in a lab, the specific format will be detailed. Lab 3-4 uses AnonCred.

## AnonCred

The AnonCred format has the following properties:

1. **Selective Disclosure**. The credential holder can disclose any subset of their total attribute set, even when the subset of attributes are part of different attribute attestations.
2. **Verifiable Authorship**. Credential verifiers can validate who the credential issuer and the credential holder are.
3. **Tamper-evident**. Any alteration to the credential is detectable.
4. **Privacy-preserving**
   - *Anonymity*. The credential holder has control over the amount of information the verifier learns in the specific interaction (note that this does not protect the holder from what the verifier can learn outside the specific interaction).
   - *Unlinkability*. The credential issuer cannot know where the credential, or parts thereof, is used.
   - *Anti correlation*. The credential cannot be correlated by any other value than the identity attributes contained within.

### History

The AnonCred format can be traced back to the mid 1980s and has since evolved to focus primarily on privacy (cf. [Chaum 1985](https://doi.org/10.1145/4372.4373); [Damgård 1990](https://doi.org/10.1007/0-387-34799-2_26); Brands [1995a](https://dl.acm.org/doi/abs/10.5555/1755009.1755034); [1995b](https://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.30.2008)). Early theoretical work was followed up by the development of efficient signature schemes (cf. Camenisch and Lysyanskaya [2001](https://eprint.iacr.org/2001/019.pdf); [2002](https://cs.brown.edu/people/alysyans/papers/camlys02.pdf); [2003](http://cs.brown.edu/people/alysyans/papers/camlys02b.pdf); Camenisch and Van Herreweghen [2002](https://dl.acm.org/doi/10.1145/586110.586114)) based on the [Strong RSA Assumption](https://en.wikipedia.org/wiki/Strong_RSA_assumption) (given a modulus *N* of unknown factorization, and a ciphertext *C*, it is infeasible to find any pair (*M,e*) s.t. *C* ≡ *M^e* mod *N* , even if the solver is allowed to pick the public exponent *e*). The CL signature scheme supports signing blocks of messages with a signle signature.

The CL signature scheme requires very large keys. In [2004](https://crypto.stanford.edu/~dabo/pubs/papers/groupsigs.pdf), Boneh, Boyen, and Shacham developed the BBS signature scheme. Based on the one of the Diffie-Hellman assumptions, the BBS signature builds on pairing based elliptic curve cryptography and thus requires a lot shorter keys. Later improvements (Au, Susilo, Mu [2006](https://link.springer.com/chapter/10.1007%2F11832072_8); Camenisch, Drijvers, and Lehmann [2016](https://eprint.iacr.org/2016/663)) lead to the BBS+ signature scheme. In [2020](https://github.com/decentralized-identity/snark-credentials/blob/master/whitepaper.pdf), Microsoft research published a white paper detailing an AnonCred implementation using [zk-SNARK](https://eprint.iacr.org/2019/550.pdf).

### Adoption

Some of the earliest adoptions of AnonCred was [U-Prove](https://www.microsoft.com/en-us/research/project/u-prove/?from=https%3A%2F%2Fresearch.microsoft.com%2Fen-us%2Fprojects%2Fu-prove) and Idemix (cf. [1](https://idemix.wordpress.com/) and [2](https://github.com/IBM/idemix)). Idemix has been integrated into projects like [IRMA](https://irma.app/) and Hyperledger Fabric Today, and both Hyperledger Indy and Hyperledger Ursa were developed with Idemix as a base. Between 2010-2015, the EU funded [ABC4Trust](https://abc4trust.eu/index.php) project defined a common, unified architecture for AnonCred systems. Both U-Prove and Idemix were integrated into the architecture (results published [here](https://abc4trust.eu/index.php/pub) and repo is available [here](https://github.com/p2abcengine/p2abcengine)).

Today, the adoption of the AnonCred format is largest among the members of the Decentralized Identity Foundation ([DIF](https://identity.foundation/)). In particular, [Trinsic](https://trinsic.id/), [Evernym](https://www.evernym.com/), [MATTR](https://mattr.global/), [Transmute](https://www.transmute.industries/), and [Finema](https://finema.co/) are all experimenting with the AnonCred format. Of special interest here is the Hyperledger projects that are AnonCred compliant: Fabric (uses Idemix), Indy, and Ursa (uses CL signatures and is similar to Idemix). The Indy and Ursa support both AnonCred 1.0 (uses CL) and AnonCred 2.0 (uses BBS+).

### Mathematical foundations of CL

Next follows three levels of explanation intended to help a reader understand the mathematical foundations of the CL signature scheme.

**In-depth explanation**:
See [this Jupyter notebook](https://github.com/PeterAltmann/SSIdemo/blob/main/VC_formats/math_CL.ipynb) for an extended explanation of the SRSA based CL signature scheme.

**Somewhat technical explanation**:
The CL signature scheme relies on two components: 1) a hardness assumption (e.g., SRSA or ECDLP), and 2) Pedersen commitments. A hardness assumption makes it possible to create so called trapdoor functions that maps elements from a domain to a co-domain, where it is difficult to find the inverse of a function without knowing some secret (as shown in the figure below). Prime factorization and the discrete log problem are two well known problems that form the basis of many trapdoor functions.

![A trapdoor function](https://github.com/PeterAltmann/SSIdemo/blob/main/VC_formats/functionGF.svg)

**Figure 1**. A function, **F**, maps an input *x* to an output *y*. The inverse function **G** takes as input (*s, y*) and outputs *x*. If *s* is not provided, it must be guessed.

Using such a trapdoor function, it is possible to create a Pedersen commitment. Such a commitment takes a message and inputs it to a trapdoor function. When the output of the function is shared it becomes a commitment to the input message. Without the secret, it is not practically possible to find the input message.

**An analogy based explanation**:
An analogy may help understand the above:

1. Alice writes a number on a note.
2. Alice puts the note into a box equipped with a PIN lock and locks the box.
3. Alice hands over the box to Bob
4. At a later date, Alice can reveal the PIN to Bob, who can use the code to open the box and access the note.

Step 1 is the message acting as the input. Step 2 is using the message as an input to a trapdoor function. Step 3 is equivalent to the public commitment of the message. And step 4 is sharing the secret that allows to allows the public revealing of the message.

### Mathematical foundations of BBS+

The BBS+ signature scheme relies on the q-Strong Diffie Hellman assumption with pairing based elliptic curve cryptography. The mathematics behind the scheme is beyond my current understanding. Instead, the text below introduces bilinear pairings and the q-SDH assumption.

**Prerequisites for understanding bilinear pairings**:
To understand what we are pairing, we must first understand cyclic groups, specifically those of prime order *p*. A cyclic group is a set of invertible elements where a single element generates all the other elements with a single associative and commutative binary operation. Every element non identity element in a cyclic group of prime order is a generator of the group. When we are dealing with elliptic curves, we are interested in multiplicative cyclic groups (to understand the math used below, see this [link](https://www.khanacademy.org/computing/computer-science/cryptography#modarithmetic)).

Example with Z_5 ie., the group {0,1,2,3,4}. We are interested in its multiplicative subgroup (in essence, we remove 0 since that is a non meaningful element with multiplication).

* `<1>` generates `{1,2,3,4}`
* `<2>` generates `{2,4,1,3}`
* `<3>` generates `{3,1,4,2}`
* `<4>` generates `{4,3,2,1}`

As we can see, the multiplicative group for Z_5 modulo 5 is a cyclic group with 4 generators.

An elliptic curve is always a cyclic group or the product of two cyclic groups ([source to an introduction to EC](https://www.math.brown.edu/johsilve/Presentations/WyomingEllipticCurve.pdf)).

**Bilinear mappings**:
Bilinear maps satisfy the following (+ and * are arbitrary operators):

1. `e(P, Q + R) = e(P, Q) * e(P, R)`
2. `e(P + S, Q) = e(P, Q) * e(S, Q)`

Example with simple integers:
* `e(3,4+5) = 2^(3*9) = 2^{27}`
* `e(3,4) * e(3,5) = 2^(12) * 2^{15} = 2^{12+15}`

Bilinear mappings with integers are easy to invert. In cryptography, we need mappings that are non efficiently invertible.

**Bilinear pairings**:
Given two cyclic groups `(G_1, G_2)` of the same prime order `p`, and their generators `(g_1, g_2)`, it is possible to generate a third group `G_T` using a bilinear map `e` as follows: `e: G_1 x G_2 --> G_T`. The bilinear map `e` generates an element in `G_T` as follows: `e(g_1, g_2) --> g_T`, where `g_T` is the generator of `G_T`. In addition, the bilinear map `e` must satisfy the following properties:

1. `e(g_1^x, g_2^y) = e(g_1^y, g_2^x)` for all x,y in Z.
2. `e(g_1^x, g_2^y) = e(g_1, g_2)^{xy}` for all x,y in Z.
3. `e(g_1^x, g_2^y) != 1` for all x,y in Z.

In BBS+, both `G_1` and `G_2` are groups of points on elliptic curves over finite fields. The groups are thus additive. The result `G_T` is a finite field that we can write as a multiplicative group. Given points `(P_1,P_2)` from `(G_1,G_2)` respectively, we can write `e` as:

* `P_1 = a \cdot g_1`
* `P_2 = b \cdot g_2`
* `e(P_1, P_2) = e(a \cdot g_1, b \cdot g_2) = e(g_1, g_2)^{ab} = g_T^{ab}`

This is where my understanding if EC pairings stops. Vitalik Buterin wrote a more extensive [guide](https://medium.com/@VitalikButerin/exploring-elliptic-curve-pairings-c73c1864e627) on medium and Fitzgerald introduces some [examples used in zkSNARK](https://blog.statebox.org/elliptic-curve-pairings-213131769fac) in his medium post, but I cannot understand the entire explanation and will therefore continue instead with discussing the q-SDH Assumption.

**q-SDH Assumption**:
The q-SDH Assumption extends the traditional DH Assumptions over two cyclic groups `(G_1,G_2)` where the might be an efficiently computable homomorphism from `G_2` to `G_1`.

**BBS+ Signatures**:
Given `{m_1, ..., m_L}` where all messages are elements of a cyclic group of prime order p, the BBS+ signature produces a public key that consists of `L+2` variables `(w,h_0, ..., h_L)` that are all elements in `e(G_1,G_2)`. The signature is generated using a secret `x` and random numbers `e,s` from `Z_p` s.t. `A = (g_1 * h_0^s * h_1^m_1 * ... * h_L^m_L)^{(e+x)^{-1}}`. The signature `(A,e,s)` exists in the bilinear mapping `e(G_1,Z_p^2)`. The verification is done by comparing two pairings but is a bit too complex to show here (cf. the [original medium article](https://medium.com/finema/anonymous-credential-part-3-bbs-signature-26797721ca74)).

### Selective Disclosure

Using a multi-message signature scheme, like CL or BBS+, selective disclosure can be enabled as follows:

1. Consider each identity attribute as the message
2. Create a Pedersen commitment to each message
3. Sign the Pedersen commitments using a multi-message signature scheme
4. Issue an object with the commitments and the signature on these commitments (this is the verifiable credential)
5. The holder, who knows the message, has two options of sharing a verifiable credential:
    * share the message and all information required to create the Pedersen commitment on the message
    * share only the Pedersen commitment on the message
6. The verifier:
    * can see any message shared and create the corresponding Pedersen commitment used in the verifiable credential
    * cannot learn any message that the holder did not share
    * can verify the signature on the Pedersen commitments in the verifiable credential
