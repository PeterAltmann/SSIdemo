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

The prerequisites for the labs are a computer (with access to Ubuntu 18.04) and a smart phone. All the labs can be run locally on your machine, or using the browser using a service called "Play with Docker", which allows you to access a terminal command line without having to install anything locally. If you want to run the labs locally, you will need a terminal CLI running bash shell, [docker](https://docs.docker.com/get-docker/) and [docker-compose](https://docs.docker.com/compose/install/), and [git](https://www.linode.com/docs/guides/how-to-install-git-on-linux-mac-and-windows/). To run the labs in your browser, go to the docker playground http://play-with-von.vonx.io/ (already has all the prerequisites installed). Optionally, if you are not comfortable with CLI, there is this guide that is focused on [openAPI](https://medium.com/@khalifa.toumi/how-to-use-hyperledger-aries-cloud-agent-for-a-classical-workflow-issuer-holder-verifier-9dd595f2f847).

## Lab 1. Establishing a connection between two agents

To establish a connection, we need 

1. A working VDR
2. One agent issuing a connection invitation
3. Another agent acceping the connection invitation

There exist several ways to create a working VDR. The options to run a VDR on a local machine, on a remote server we control, and connecting to a pre-existing test oriented VDR are shown below. Once the VDR is up and running, we can focus on the agents. The demo inluded in ACA-Py includes two agents that seek to connect with each other. For now, we will only run the demo to see what a connection flow looks like. The first of two agents are Faber, who issues the connection request and is a made up university. The second agent is Alice, a recently graduated university student who wants her diploma as a VC.

### For a local VDR

Running a VDR locally will create the VDR on the same machine as the two agents as follows.

```bash
git clone https://github.com/bcgov/von-network
cd von-network
./manage build
./manage start --logs
```
Check http://localhost:9000) to make sure the VDR is up and running. Open a new terminal tab and do the following for the Faber instance.

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

Follow the CLI options to establish a connection.

### For a remote VDR

To install the VDR on a remote server, do as follows.

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

Do the below on the Alice instance.

```bash
git clone https://github.com/hyperledger/aries-cloudagent-python
cd aries-cloudagent-python/demo
LEDGER_URL=http://<server_ip>:9000 ./run_demo alice
```
Follow the CLI options to establish a connection.

### For a public VDR

There exist multiple public VDRs that run test and developer networks. One such network is the [BCovrin network](http://dev.bcovrin.vonx.io/) hosted by the government of British Colombia.

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

Follow the CLI options to establish a connection.

### The connection

Once Faber has started, a connection request is published. It can look like this:

```JSON
{
  "@type": "https://didcomm.org/out-of-band/1.0/invitation",
  "@id": "a6ff56d3-539e-4c09-bcfc-5cb582bae824",
  "handshake_protocols": [
    "https://didcomm.org/didexchange/1.0"
  ],
  "services": [
    {
      "id": "#inline",
      "type": "did-communication",
      "recipientKeys": [
        "did:key:z6MkiMKr3xYA8o8mHb2S92Va7eWdjTTtW21QL7VomxBXnKne"
      ],
      "serviceEndpoint": "http://172.17.0.1:8020"
    }
  ],
  "label": "faber.agent"
}
```

Note that the connection request is public. Anyone who has the invitation Alice can now accept the invitiation and generate the following response:

```JSON
{
  "connection_protocol": "didexchange/1.0",
  "their_role": "inviter",
  "invitation_key": "4u4oTiHioFeJB6BjTTXjGYxdutC368m3e6aswgDWs71G",
  "invitation_mode": "once",
  "state": "request",
  "rfc23_state": "request-sent",
  "accept": "auto",
  "routing_state": "none",
  "my_did": "VHNZAZvmrDLoJc5p8NhXz3",
  "their_label": "faber.agent",
  "connection_id": "75d75abc-4032-4747-8de6-9990d199b886",
  "updated_at": "2022-01-14T08:25:01.527751Z",
  "created_at": "2022-01-14T08:25:01.453026Z",
  "request_id": "12d17855-6990-4fee-b27e-4cb0ab3dbbb4",
  "invitation_msg_id": "a6ff56d3-539e-4c09-bcfc-5cb582bae824"
}
```

So, what happened? When Faber started, he used the VDR's genesis file (`<server_ip>:9000/genesis`) to get a list of stewards (nodes with VDR write permission) and service endpoints. Faber then generated a DID and asked a steward to register his DID on the VDR. It is worth taking a quick look at parts (more specificaly node 1) of the genesis file to understand what it contains.

```JSON
{
  "reqSignature": {},
  "txn": {
    "data": {
      "data": {
        "alias": "Node1",
        "blskey": "4N8aUNHSgjQVgkpm8nhNEfDf6txHznoYREg9kirmJrkivgL4oSEimFF6nsQ6M41QvhM2Z33nves5vfSn9n1UwNFJBYtWVnHYMATn76vLuL3zU88KyeAYcHfsih3He6UHcXDxcaecHVz6jhCYz1P2UZn2bDVruL5wXpehgBfBaLKm3Ba",
        "blskey_pop": "RahHYiCvoNCtPTrVtP7nMC5eTYrsUA8WjXbdhNc8debh1agE9bGiJxWBXYNFbnJXoXhWFMvyqhqhRoq737YQemH5ik9oL7R4NTTCz2LEZhkgLJzB3QRQqJyBNyv7acbdHrAT8nQ9UkLbaVL9NBpnWXBTw4LEMePaSHEw66RzPNdAX1",
        "client_ip": "172.17.0.1",
        "client_port": 9702,
        "node_ip": "172.17.0.1",
        "node_port": 9701,
        "services": [
          "VALIDATOR"
        ]
      },
      "dest": "Gw6pDLhcBcoQesN72qfotTgFa7cbuqZpkX3Xo6pLhPhv"
    },
    "metadata": {
      "from": "Th7MpTaRZVRYnPiabds81Y"
    },
    "type": "0"
  },
  "txnMetadata": {
    "seqNo": 1,
    "txnId": "fea82e10e894419fe2bea7d96296a6d46f50f93f9eeda954ec461b2ed2950b62"
  },
  "ver": "1"
}
```

If we use the ledger browser and go to `<server_ip>:9000/browse/domain` we can look for the tx that registered faber. The raw tx data is quite extensive, but the part that matters looks like follows:

```JSON
{
  "...": "...",
  "txn": {
    "data": {
      "dest": "DWSDA3P6MaPgsgTmDbQ2TK",
      "raw": "{\"endpoint\":{\"endpoint\":\"http://172.17.0.1:8020\"}}"
    },
    "...": "..."
  }
}
```

Using the ledger, Alice can verify that the connection request indeed came from Faber. Alice can also browse the VDR and look for faber's endpoints. The VDR provides the source of technical trust Alice needs to know that Faber's connection request indeed came from an actor registered as Faber. As long as Alice can trust the identity proofing process Faber did with the VDR steward, she can trust that the registered actor is indeed Faber.

To stop the VDR (try and see what happens while Faber and Alice are still connected):

```bash
./manage stop
```

To completely destroy the VDR instance:

```bash
./manage down
```

This concludes Lab 1.

In Lab 2. we look at the demos in ACA-Py and von network to get a basic idea of the entities necessary and the flows required for an identity ecosystem based on Indy, Aries, Ursa. Follow [this link](https://github.com/PeterAltmann/SSIdemo/blob/main/LAB2.md) to get to the lab. 

## Lab 3

In Lab 3. we take a closer look at ACA-Py and see how we can use it to start our own agents and issue a VC. See this [link](https://github.com/PeterAltmann/SSIdemo/blob/main/LAB3.md). 
