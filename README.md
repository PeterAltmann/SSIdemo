# An introduction to Authentic Company Data

This text aims to document the design process with the Authentic Company Data project at Bolagsverket. Digital identity is rapidly evolving. Open source projects like Hyperledger Aries and Ursa projects provide a set of protocols that, when paired with DLTs such as Hyperledger Indy or Besu, can be used for building distributed applications built on authentic and secure data.

The text assumes a high level familiarity with the [Trust over IP concept](https://trustoverip.org/toip-model/). The Linux Foundation added ToIP in to its projects in 2020. The mission of the ToIP foundation is to simplify and standardize how trust is established online, i.e., to provide mechanisms for how interacting entities can trust the credential exchanges they are engaged in. At its core, the ToIP introduces three actors: an issuer of credentials, a holder of credentials, and a verifier of credentials. The concept relies heavily on [the Verifiable Credentials data model](https://www.w3.org/TR/vc-data-model/). 

<img src="https://courses.edx.org/assets/courseware/v1/e6878fa7000fb538a7e8b9dec066d0b1/asset-v1:LinuxFoundationX+LFS173x+3T2021+type@asset+block/LFS172x_CourseGraphics_V1-04.png" alt="VC data model" width="500"/>

**Fig 1**. *The Verifiable Credentials Data Model.*

The concept introduces a technology and governance stack that affors a very high degree of privacy and user control to the credential holder. The ToIP concept can be viewed as an alternative to existing centralized and federated identity models where the user has very limited control over their online idenity and data. 

## Objectives

The specific objectives of this text is to use a series of labs to demonstrate how Verifiable Credentials (or equivalent authenticated data) works and to motivate design choices along the way. This text will focus primarily on Layers 1-3 on the ToIP technology stack.

<img src="https://miro.medium.com/max/1400/1*DgPBpnT_RxDEnd_ptdynkQ.png" alt="drawing" width="800"/>

**Fig 2.** *The dual stack ToIP ([source](https://www.dizme.io/)).*

Where necessary, we use default settings for the governance stacks. More specifically,

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

## Terminology and key concepts

See this [link](https://docs.google.com/document/d/1gfIz5TT0cNp2kxGMLFXr19x1uoZsruUe_0glHst2fZ8/edit) for an exhaustive list of terms and definitions. Key concepts used in the labs are as follow:

* Self-sovereign identity (SSI). An idea that digital identity should be privacy preserving and identity subject controlled by design. Identity related data flows should be solely in the control of the identity subject, i.e., there is no communication between the issuer and the verifier.
* Trust over IP. A Linux Foundation organization that has developed the dual stack model of how to establishing trust online.
* Decentralized identifiers (DID). A DID is a universally unique identifier that can be cryptographically verified in such a way that does not rely on a central authority. See the [W3C proposed standards page](https://www.w3.org/TR/did-core/) for more information.
* Zero Knowledge Proof (ZKP). A proof system that prooves only whether a fact or a statement is true or not without revealing any additional details about the fact or statement.
*  Selective disclosure. A capability of some verifiable credential formats that allows the holder to select which attributes from an issued verifiable credential to share with a verifier.
*  Revocation. The capability of an issuer to publish information used to verify the status of an issued credential. Revocation is either done using traditional techniques, or in a ZKP fashion.
*  Verifiable Credential formats. The specific way a verifiable credential is formatted. For an in-depth reading see [Young. Verifiable Credentials
Flavors Explained](https://www.lfph.io/wp-content/uploads/2021/02/Verifiable-Credentials-Flavors-Explained.pdf) and for an overview see this [infographic](https://www.lfph.io/wp-content/uploads/2021/04/Verifiable-Credentials-Flavors-Explained-Infographic.pdf). The credential format and the signature schemes used is one major design choice. For selective disclosure, you need a signature scheme capable of multi-message signing, e.g., CL or BBS+. Of the two, CL signatures are more widely deployed, but are not compliant with the W3C VC data model. In contrast, BBS+ is W3C VC compliant, but does not support ZKP revocation and redicate proofs. Each lab will detail the use of VC format.
* Secure storage. A way to securely store secrets. This involves the management of cryptographic secrets handled within a key management service, and the storage of other sensitive data.
* Agent. An agent is a software that interacts with other entities in order to facilitate the handling of VCs.
* Blockhains and Distributed Ledger Technology (DLT). An identity ecosystem must rely on some sort of verifiable data registry (a VDR) to maintain certain data that must be public. The exact nature of the VDR is another major design implication.  
* Framework. The framework is one out of two logical components that enables an agent to interact. The framework is what knows how to establish a connection, send a message, create a credential etc. There exist many frameworks, the most popular is arguably the frameworks that build on the Hyperledger Aries protocols. Most of the labs herein, build on the Aries framework called ACA-Py unless otherwise specified. The ACA-Py is a python based framework focused on enterprise agents.
* Controller. The controller is the second logical component of an agent. The framework does not know when to establish a connection or when to issue a credential. The controller encodes the organizations business rules and is responsible for telling the framework what to do and when to do it.


# General setup guide for the labs

The prerequisites for the labs are a computer and a smart phone. All the labs can be run locally on your machine, or using the browser using a service called "Play with Docker", which allows you to access a terminal command line without having to install anything locally. If you want to run the labs locally, you will need a terminal CLI running bash shell, [docker](https://docs.docker.com/get-docker/) and [docker-compose](https://docs.docker.com/compose/install/), and [git](https://www.linode.com/docs/guides/how-to-install-git-on-linux-mac-and-windows/). To run the labs in your browser, go to the docker playground http://play-with-von.vonx.io/ (already has all the prerequisites installed).

## Lab 1. Establishing a connection between two agents

In this description, we assume that you are running the lab using your browser. Navigate tot he docker playground and add three new instances if you want to run the VDR (or only two agent instances if you want to use a public VDR).

Instance 1 w ill be the VDR, in this case running a modified instance of the Hyperledger Indy DLT. To run the network locally, use the following commands:

### For a local VDR

Only do the below if you want to run the VDR instance locally on your home server!
```bash
git clone https://github.com/bcgov/von-network
cd von-network
./manage build
./manage start --logs
# Once the startup is ready, you can connect to the web server on http://localhost:9000
```
Open a new terminal tab. Do the following for the Faber instance.
```bash
git clone https://github.com/hyperledger/aries-cloudagent-python
cd aries-cloudagent-python/demo
./run_demo faber
```
Open a new terminal tab. Do the following for the Alice instance.
```bash
git clone https://github.com/hyperledger/aries-cloudagent-python
cd aries-cloudagent-python/demo
./run_demo alice
```

Continue to the "Establish connection" section below.

### For a remote VDR

Only do the below on your server if you want to setup a remote VDR instance on said server!
```bash
git clone https://github.com/bcgov/von-network
cd von-network
./manage build
./manage start <server_ip> &
```
Once the startup is ready, you can connect to the web server on http://<server_ip>:9000

Do the below on the Faber instance.
```bash
git clone https://github.com/hyperledger/aries-cloudagent-python
cd aries-cloudagent-python/demo
LEDGER_URL=<server_ip>:9000 ./run_demo faber
```

Continue to the "Establish connection" section below.

Do the below on the Faber instance.
```bash
git clone https://github.com/hyperledger/aries-cloudagent-python
cd aries-cloudagent-python/demo
LEDGER_URL=http://<server_ip>:9000 ./run_demo alice
```

### For a public VDR

Do the below on the Faber instance.
```bash
git clone https://github.com/hyperledger/aries-cloudagent-python
cd aries-cloudagent-python/demo
LEDGER_URL=http://dev.greenlight.bcovrin.vonx.io ./run_demo faber
```

Do the below on the Faber instance.
```bash
git clone https://github.com/hyperledger/aries-cloudagent-python
cd aries-cloudagent-python/demo
LEDGER_URL=http://dev.greenlight.bcovrin.vonx.io ./run_demo alice
```

### Establish connection

Once Faber has started, a connection request is published. In my instance, it looked like this following `Invitation data:`:

```JSON
{
  "@type": "https://didcomm.org/out-of-band/1.0/invitation",
  "@id": "e88affc6-5e67-4a64-a107-e5053f18fe38",
  "services": [
    {
      "id": "#inline",
      "type": "did-communication",
      "recipientKeys": [
        "did:key:z6MktM9tkzqT8u6PZon7KqVyXC9FDYZaFocEo3xGbTCWCZK4"
      ],
      "serviceEndpoint": "http://ip10-6-103-5-c7g3d43nbu9ga4a60skg-8020.direct.play-with-von.vonx.io"
    }
  ],
  "handshake_protocols": [
    "https://didcomm.org/didexchange/1.0"
  ],
  "label": "faber.agent"
}
```

Alice can now accept the invitiation and generate the following response:

```JSON
{
  "accept": "auto",
  "their_role": "inviter",
  "created_at": "2022-01-13T15:22:37.796205Z",
  "their_label": "faber.agent",
  "state": "request",
  "updated_at": "2022-01-13T15:22:37.885637Z",
  "invitation_msg_id": "e88affc6-5e67-4a64-a107-e5053f18fe38",
  "rfc23_state": "request-sent",
  "my_did": "NuJAe2R8JK644uBeqrXudT",
  "routing_state": "none",
  "request_id": "eff611a9-987f-4b2e-b2c8-b4b890258919",
  "connection_protocol": "didexchange/1.0",
  "invitation_mode": "once",
  "connection_id": "faac64c6-faf4-4720-9164-e3198e81218f",
  "invitation_key": "EttrAkb1oMbvTJwQeGY8g6bFPyHiqvMt733LmBEVHLXg"
}
```

So, what happened? When Faber started, he generated a DID and asked a steward (nodes with write permission to the VDR) to register his DID on the VDR. If we use the ledger browser and go to <server_ip>/browse/domain we can look for the tx that registered faber. The raw tx data is quite extensive, but the part that matters looks like follows:

```JSON
{
  ...
  "txn": {
    "data": {
      "alias": "faber.agent",
      "dest": "XLU6qEdvMpZ8kGkDGenmPk",
      "role": "101",
      "verkey": "HXyEFJ8YA5cfY7f3ALb2jddZ9FGfwSKywruD1QtfgD6o"
      },
    ...
}
```

Using the ledger, Alice can now verify that the connection request indeed came from Faber.
