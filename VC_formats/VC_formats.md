# Verifiable Credential formats

All VC's contain attributes about an identity subject. There exits multiple VC formats (for a detailed description, see the following links: [Young infographic](https://github.com/PeterAltmann/SSIdemo/blob/main/VC_formats/Verifiable-Credentials-Flavors-Explained-Infographic.pdf), [Young paper](https://github.com/PeterAltmann/SSIdemo/blob/main/VC_formats/Verifiable-Credentials-Flavors-Explained.pdf), and [W3C proof formats](https://www.w3.org/TR/vc-imp-guide/#proof-formats))

Below, when used in a lab, the specific format will be detailed.

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

Today, the adoption of the AnonCred format is largest among the members of the Decentralized Identity Foundation ([DIF](https://identity.foundation/)). Of special interest here is the Hyperledger projects that are AnonCred compliant: Fabric (uses Idemix), Indy, and Ursa (uses CL signatures and is similar to Idemix). The Indy and Ursa support both AnonCred 1.0 (uses CL) and AnonCred 2.0 (uses BBS+).

 
