# An introduction to Authentic Company Data

This text aims to document the design process with the Authentic Company Data project at Bolagsverket. Digital identity is rapidly evolving. Open source projects like Hyperledger Aries and Ursa projects provide a set of protocols that, when paired with DLTs such as Hyperledger Indy or Besu, can be used for building distributed applications built on authentic and secure data.

A core element of this text is the [Trust over IP concept](https://trustoverip.org/toip-model/). The Linux Foundation added ToIP in to its projects in 2020. The mission of the ToIP foundation is to simplify and standardize how trust is established online, i.e., to provide mechanisms for how interacting entities can trust the credential exchanges they are engaged in. At its core, the ToIP introduces three actors: an issuer of credentials, a holder of credentials, and a verifier of credentials. The concept relies heavily on [the Verifiable Credentials data model](https://www.w3.org/TR/vc-data-model/). 

<img src="https://courses.edx.org/assets/courseware/v1/e6878fa7000fb538a7e8b9dec066d0b1/asset-v1:LinuxFoundationX+LFS173x+3T2021+type@asset+block/LFS172x_CourseGraphics_V1-04.png" alt="VC data model" width="500"/>

**Fig 1**. *The Verifiable Credentials Data Model.*

The concept introduces a technology and governance stack that affors a very high degree of privacy and user control to the credential holder. The ToIP concept can be viewed as an alternative to existing centralized and federated identity models where the user has very limited control over their online idenity and data. However, the ToIP concept does not assume any trust model for identity related data and can be configured to support many kinds of trust models. 

## Objectives

The specific objectives of this text is to build a good foundation for understanding ToIP compliant digital identities. The text will use a series of labs to demonstrate how Verifiable Credentials (or equivalent authenticated data) works and to motivate design choices along the way. The main focus of the text is  Layers 1-3 on the ToIP technology stack.

<img src="https://miro.medium.com/max/1400/1*DgPBpnT_RxDEnd_ptdynkQ.png" alt="drawing" width="800"/>

**Fig 2.** *The dual stack ToIP ([source](https://www.dizme.io/)).*

Also, the default settings for the governance stacks are often used since the focus is on the technology stack. More specifically:

* Layer 1: establishing cryptographic roots of trust labs:
  1. How to establish a secure and privacy connection between two actors.
  2. How to verify the identity of the party you are connecting to.
  3. Explain the governance choices necessary to enable technical trust on Layer 1.
* Layer 2: the agent labs:
  1. How to implement digital wallets and agents
* Layer 3: data exchange labs
  1. Offering, requesting, and creating verifiable credentials
  2. Verifying verifiable credentials
  3. Enabling advanced features like selective disclosure and predicate proofs

Note that the lab numbering will not correspond to the layer numbering.

## Terminology and key concepts

See this [link](https://docs.google.com/document/d/1gfIz5TT0cNp2kxGMLFXr19x1uoZsruUe_0glHst2fZ8/edit) for an exhaustive list of terms and definitions. Key concepts used in the labs are as follow:

* **Self-sovereign identity (SSI)**. An idea that digital identity should be privacy preserving and identity subject controlled by design. Identity related data flows should be solely in the control of the identity subject, i.e., there is no communication between the issuer and the verifier.
* **Trust over IP**. A Linux Foundation organization that has developed the dual stack model of how to establishing trust online.
* **Decentralized identifiers (DID)**. A DID is a universally unique identifier that can be cryptographically verified in such a way that does not rely on a central authority. See the [W3C proposed standards page](https://www.w3.org/TR/did-core/) for more information.
* **Zero Knowledge Proof (ZKP)**. A proof system that prooves only whether a fact or a statement is true or not without revealing any additional details about the fact or statement.
*  **Selective disclosure**. A capability of some verifiable credential formats that allows the holder to select which attributes from an issued verifiable credential to share with a verifier.
*  **Revocation**. The capability of an issuer to publish information used to verify the status of an issued credential. Revocation is either done using traditional techniques, or in a ZKP fashion.
*  **Verifiable Credential formats**. The specific way a verifiable credential is formatted. For an in-depth reading see [Young. Verifiable Credentials
Flavors Explained](https://www.lfph.io/wp-content/uploads/2021/02/Verifiable-Credentials-Flavors-Explained.pdf) and for an overview see this [infographic](https://www.lfph.io/wp-content/uploads/2021/04/Verifiable-Credentials-Flavors-Explained-Infographic.pdf). The credential format and the signature schemes used is one major design choice. For selective disclosure, you need a signature scheme capable of multi-message signing, e.g., CL or BBS+. Of the two, CL signatures are more widely deployed, but are not compliant with the W3C VC data model. In contrast, BBS+ is W3C VC compliant, but does not support ZKP revocation and redicate proofs. Each lab will detail the use of VC format.
     -. *AnonCred*. An early privacy focuse VC format focused on attributes. Not compatible with the W3C VC format but is easier to use and has more extensive privacy features. See details [here](https://github.com/PeterAltmann/SSIdemo/blob/main/VC_formats.md).
* **Secure storage**. A way to securely store secrets. This involves the management of cryptographic secrets handled within a key management service, and the storage of other sensitive data.
* **Agent**. An agent is a software that interacts with other entities in order to facilitate the handling of VCs.
* **Blockhains and Distributed Ledger Technology (DLT)**. An identity ecosystem must rely on some sort of verifiable data registry (a VDR) to maintain certain data that must be public. The exact nature of the VDR is another major design implication.  
* **Framework**. The framework is one out of two logical components that enables an agent to interact. The framework is what knows how to establish a connection, send a message, create a credential etc. There exist many frameworks, the most popular is arguably the frameworks that build on the Hyperledger Aries protocols. Most of the labs herein, build on the Aries framework called ACA-Py unless otherwise specified. The ACA-Py is a python based framework focused on enterprise agents.
* **Controller**. The controller is the second logical component of an agent. The framework does not know when to establish a connection or when to issue a credential. The controller encodes the organizations business rules and is responsible for telling the framework what to do and when to do it.

There are many protocols, technologies, implementations etc., mentioned in the text below. To facilitate reading, please se the following terminology/concepts/terms map in [this link](https://user-images.githubusercontent.com/30799110/150362394-5d0319ae-7bad-4674-863c-d6dd35346ea9.png).

# General setup guide for the labs

The prerequisites for the labs are a computer (with access to Ubuntu 18.04) and a smart phone. All the labs can be run locally on your machine, or using the browser using a service called "Play with Docker", which allows you to access a terminal command line without having to install anything locally. If you want to run the labs locally, you will need a terminal CLI running bash shell, [docker](https://docs.docker.com/get-docker/) and [docker-compose](https://docs.docker.com/compose/install/), and [git](https://www.linode.com/docs/guides/how-to-install-git-on-linux-mac-and-windows/). To run the labs in your browser, go to the docker playground http://play-with-von.vonx.io/ (already has all the prerequisites installed). Optionally, if you are not comfortable with CLI, there is this guide that is focused on [openAPI](https://medium.com/@khalifa.toumi/how-to-use-hyperledger-aries-cloud-agent-for-a-classical-workflow-issuer-holder-verifier-9dd595f2f847).

## Lab 1 

In Lab 1., we use the demos provided to get a basic feel for how agents work and interact. Follow [this link](https://github.com/PeterAltmann/SSIdemo/blob/main/LAB1.md) for the lab.

## Lab 2

In Lab 2. we move beyond the demos and take a closer look at how we can use ACA-Py to provision agents and do basic interactions. Follow [this link](https://github.com/PeterAltmann/SSIdemo/blob/main/LAB2.md) to get to the lab. 

## Lab 3

In Lab 3. we continue taking a closer look at ACA-Py and see how we can use it to start our own agents and issue a VC within the context of Authentic Company Data. See this [link](https://github.com/PeterAltmann/SSIdemo/blob/main/LAB3.md). 

## Lab 4

In [Lab 4](https://github.com/PeterAltmann/SSIdemo/blob/main/LAB4.md), we take a closer look at DIDComm and the ways agents can connect.
