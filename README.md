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

## Lab 2. An introduction to agents and wallets with ACA-Py

In Lab 1, we used a runner to create two agents, Faber and Alice, and set up a connection between them. The runner consists of an agent (an instance of ACA-Py) and the controller (the code that interacts with the HTTP Admin Endpoints of ACA-Py). Moving forward, we do not want to use the demo for experimenting with our agents and our wallets. We instead start ACA-Py instances ourselves, and will use `curl` commands to act as the controller. We could of course use any programming language to act as our controller as long as we are able to interact with Web Services.


### Installing ACA-Py

To run this lab you have to install ACA-Py. It is available as a Python package through pip

```bash
pip3 install aries-cloudagent
```

ACA-Py can also be installed using the repository:

```bash
git clone https://github.com/hyperledger/aries-cloudagent-python
cd aries-cloudagent-python
pip3 install -r requirements.txt -r requirements.dev.txt -r requirements.indy.txt
pip3 install --no-cache-dir -e .
```

To create wallets, you will need `python3-indy` which allows you to communicate with `libindy` (the package that can create and manage a wallet). Both are part of https://pypi.org/project/python3-indy/

To install `libindy`
```bash
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 68DB5E88
sudo add-apt-repository "deb https://repo.sovrin.org/sdk/deb bionic master"
sudo apt update
sudo apt install libindy -y
```

Moving forward, we assume that you have access to a `aca-py` cli (if not, just substitue the `aca-py -args` command with `docker run --net=host bcgovimages/aries-cloudagent:py36-1.16-0_0.6.0 -args`). 

### Starting the credential issuer Alice

In Indy networks, an issuer of a VC must do the following:
1. Register their DID on the VDR
2. Register the credential schema (in essence a list of attributes contained in the VC)
3. Register a credential definition corresponding to the credential schema

We derive the DID from a public key, which in turn is generated using a seed (32 characters or base64). The command

```bash
curl -X POST "http://localhost:9000/register" -d '{"seed": "Alice000000000000000000000000001", "role": "TRUST_ANCHOR", "alias": "Alice"}'
```

returns 

```bash
{
  "did": "PLEVLDPJQMJvPLyX3LgB6S",
  "seed": "Alice000000000000000000000000001",
  "verkey": "DAwrZwgMwkTVHUQ8ZYAmuvzwprDmX8vFNXzFioxrWpCA"
}
```

Note that if an DID already exist, Indy will ignore the new registration. You can now see the tx on the VDR browser. 

Now that the DID is registered, we can continue the next step toward creating a schema and a credential schema for Alice. To do that, we need to initiate an ACA-Py instance for Alice.

```bash
aca-py start \
--label Alice \
-it http 0.0.0.0 8000 \
-ot http \
--admin 0.0.0.0 11000 \
--admin-insecure-mode \
--genesis-url http://localhost:9000/genesis \
--seed Alice000000000000000000000000001 \
--endpoint http://localhost:8000/ \
--debug-connections \
--auto-provision \
--wallet-type indy \
--wallet-name Alice1 \
--wallet-key secret \
--auto-accept-invites \
--auto-accept-requests

>

::::::::::::::::::::::::::::::::::::::::::::::
:: Alice                                    ::
::                                          ::
::                                          ::
:: Inbound Transports:                      ::
::                                          ::
::   - http://0.0.0.0:8000                  ::
::                                          ::
:: Outbound Transports:                     ::
::                                          ::
::   - http                                 ::
::   - https                                ::
::                                          ::
:: Public DID Information:                  ::
::                                          ::
::   - DID: PLEVLDPJQMJvPLyX3LgB6S          ::
::                                          ::
:: Administration API:                      ::
::                                          ::
::   - http://0.0.0.0:11000                 ::
::                                          ::
::                               ver: 0.7.3 ::
::::::::::::::::::::::::::::::::::::::::::::::

Listening...
```

The above commands explained (for a more detailed look at the various arguments, see the [ACA-Py commands list](https://github.com/PeterAltmann/SSIdemo/blob/main/ACA-Py_commands.md)):

* `--label` sets the name for the instance. This is the name that the WalletApp will see when you try to make a connection or when you receive a credential.
* `-it` and `ot` sets the inbound and outbound transport methods that ACA-Py will use when communicating with other ACA-Py instances.
* `--admin` and `--admin-insecure-mode` configure how the controller application can communicate, without authenticating, with ACA-Py using Admin Endpoints. If you open http://localhost.11000 you will see the Swagger docs and the provided label.
* `--genesis-url` is the URL to the genesis file that the instance needs to submit the tx details to the VDR.
* `--seed` sets the seed for the DID.
* `--endpoint` is the URL that ACA-Py sends to the VDR so that another agent can learn how to connect with the ACA-Py instance for a particular DID. Agents will find the endpoint for PLEVLDPJQMJvPLyX3LgB6S here http://localhost:9000/browse/domain?page=1&query=PLEVLDPJQMJvPLyX3LgB6S
* `--debug-connections` prints more information about the connections made between agents
* `--auto-provision` makes sure that ACA-Py creates a wallet even if such a wallet does not exist. Normally, the command `aca-py provision` is used to create a wallet.
* `--wallet-type`, `--wallet-name`, and `--wallet-key` are used to create the wallet. They key is used for read and write to the sqlite db. The wallet is found in `~./.indy_client/wallet`
* `--auto-accept-invites` and `--auto-accept-requests` set ACA-Py instances to automatically accepts connection invites and connection requests as they come in.

### Starting a credential holder Bob

Now we can start Bob.

```bash
aca-py start \
--label Bob \
-it http 0.0.0.0 8001 \
-ot http \
--admin 0.0.0.0 11001 \
--admin-insecure-mode \
--genesis-url http://localhost:9000/genesis \
--endpoint http://localhost:8001/ \
--debug-connections \
--auto-provision \
--wallet-local-did \
--wallet-type indy \
--wallet-name Bob1 \
--wallet-key secret \
--auto-accept-invites \
--auto-accept-requests

>

::::::::::::::::::::::::::::::::::::::::::::::
:: Bob                                      ::
::                                          ::
::                                          ::
:: Inbound Transports:                      ::
::                                          ::
::   - http://0.0.0.0:8001                  ::
::                                          ::
:: Outbound Transports:                     ::
::                                          ::
::   - http                                 ::
::   - https                                ::
::                                          ::
:: Administration API:                      ::
::                                          ::
::   - http://0.0.0.0:11001                 ::
::                                          ::
::                               ver: 0.7.3 ::
::::::::::::::::::::::::::::::::::::::::::::::

Listening...
```

The commands are similar with the exception that Bob does not register to the VDR as a holder. Also, `--wallet-local-did` is used to generate a private pairwise DID for Bob and Alice. For more on connection protocols, see [Aries RFC 0160](https://github.com/hyperledger/aries-rfcs/tree/9b0aaa39df7e8bd434126c4b33c097aae78d65bf/features/0160-connection-protocol).

### The issuer creates a schema and credential definition

Alice can now issue her credential schema and her credential definition. First the schema:

```bash
curl -X POST http://localhost:11000/schemas \
-H 'Content-Type: application/json' \
-d '{
  "attributes": [
    "name",
    "age"
  ],
  "schema_name": "basic-schema",
  "schema_version": "1.0"
}'
```

This returns:

```JSON
{
  "schema_id": "PLEVLDPJQMJvPLyX3LgB6S:2:basic-schema:1.0",
  "schema": {
    "ver": "1.0",
    "id": "PLEVLDPJQMJvPLyX3LgB6S:2:basic-schema:1.0",
    "name": "basic-schema",
    "version": "1.0",
    "attrNames": [
      "name",
      "age"
    ],
    "seqNo": 8
  }
}
```

You can find the schema browsing the VDR: http://localhost:9000/browse/domain?page=1&query=&txn_type=101. 

Alice now creates a credential definition with the newly created schema.

```bash
curl -X POST http://localhost:11000/credential-definitions \
  -H 'Content-Type: application/json' \
  -d '{
  "schema_id": "PLEVLDPJQMJvPLyX3LgB6S:2:basic-schema:1.0",
  "tag": "default"
}'
```

This returns the credential definition:

```JSON
{
  "credential_definition_id": "PLEVLDPJQMJvPLyX3LgB6S:3:CL:8:default"
}
```

You can find the credential schema browsing the VDR: http://localhost:9000/browse/domain?page=1&query=&txn_type=102

To issue a credential, Alice and Bob need to connect. A connection follows an invite. There exist both public and non-public invites. Public invitations allow anyone to connect with the issue based on information in the VDR only. A non-public invitation is self-contained. Below, we will rely on a non-public connection invitation simply because it was the one I got working first.

### Non-public connection invitation

With a non-public connection, Alice sends over an object to Bob that contains all the information Bob needs without having to browse a VDR.

#### Creating a non-public connection invite

```bash
curl -X POST "http://localhost:11000/out-of-band/create-invitation" \
  -H "Content-Type: application/json" \
  -d '{"handshake_protocols": ["did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/didexchange/1.0"], "use_public_did": false, "my_label": "Alice"}'
```

This returns the following private invitation:

```JSON
{
  "@type": "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/out-of-band/1.0/invitation",
  "@id": "49450337-07cc-41bc-a2fb-76d832353272",
  "services": [
    {
      "id": "#inline",
      "type": "did-communication",
      "recipientKeys": [
        "did:key:z6MkoeyEuKV7QhYE7YL2hpkxZ4qmxYT719r9qSJrCBhfwKuc"
      ],
      "serviceEndpoint": "http://localhost:8000/"
    }
  ],
  "handshake_protocols": [
    "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/didexchange/1.0"
  ],
  "label": "Alice"
}
```

Note how a non-public invite does not use a public DID, instead it contains a service endpoint url, so the invited agent can connect to inviter directly. 

Bob needs to receive the invitation object that Alice generated above. Bob can receive this either using the url in ALice's connection invite, or as follows:

```bash
curl -X POST "http://localhost:11001/out-of-band/receive-invitation" \
   -H 'Content-Type: application/json' \
   -d '{
  "@type": "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/out-of-band/1.0/invitation",
  "@id": "49450337-07cc-41bc-a2fb-76d832353272",
  "services": [
    {
      "id": "#inline",
      "type": "did-communication",
      "recipientKeys": [
        "did:key:z6MkoeyEuKV7QhYE7YL2hpkxZ4qmxYT719r9qSJrCBhfwKuc"
      ],
      "serviceEndpoint": "http://localhost:8000/"
    }
  ],
  "handshake_protocols": [
    "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/didexchange/1.0"
  ],
  "label": "Alice"
}'
```

The two agents are now connected. 

Alice's view using `curl -X GET "http://localhost:11000/connections" -H  "accept: application/json"`

```JSON
{
  "results": [
    {
      "accept": "auto",
      "connection_id": "50a1c481-b87b-4ea5-a48b-9fd9382322f9",
      "connection_protocol": "didexchange/1.0",
      "created_at": "2022-01-16T14:27:11.200483Z",
      "invitation_key": "ACiCK5Eg5A3m13VL2Fo7hyHn8yBFbGbo9RPvMujf278E",
      "invitation_mode": "once",
      "invitation_msg_id": "49450337-07cc-41bc-a2fb-76d832353272",
      "my_did": "EUHD9iSf1p1JwG951Z5XGQ",
      "request_id": "565c2eca-8323-473d-94bf-1425b971e4da",
      "rfc23_state": "completed",
      "routing_state": "none",
      "state": "active",
      "their_did": "UFw5hG1zZXRdPL1pn9CrVJ",
      "their_label": "Bob",
      "their_role": "invitee",
      "updated_at": "2022-01-16T14:28:54.322723Z"
    }
  ]
}

```

Bob's view using `curl -X GET "http://localhost:11001/connections" -H  "accept: application/json"` (sorted alphabetically for readability):

```JSON
{
  "results": [
    {
      "accept": "auto",
      "connection_id": "644cd762-bf83-4ed4-89f2-221fc220c3cf",
      "connection_protocol": "didexchange/1.0",
      "created_at": "2022-01-16T14:28:54.135817Z",
      "invitation_key": "ACiCK5Eg5A3m13VL2Fo7hyHn8yBFbGbo9RPvMujf278E",
      "invitation_mode": "once",
      "invitation_msg_id": "49450337-07cc-41bc-a2fb-76d832353272",
      "my_did": "UFw5hG1zZXRdPL1pn9CrVJ",
      "request_id": "565c2eca-8323-473d-94bf-1425b971e4da",
      "rfc23_state": "completed",
      "routing_state": "none",
      "state": "active",
      "their_did": "EUHD9iSf1p1JwG951Z5XGQ",
      "their_label": "Alice",
      "their_role": "inviter",
      "updated_at": "2022-01-16T14:28:54.305559Z"
    }
  ]
}

```

We can now continue the lab.

### Issuing a Credential

Alice and Bob now share a connection. The credential issuance flow depends on which party initiates the process and with what. 

1. Holder sends a proposal to the issuer (issuer receives proposal)
2. Issuer sends an offer to the holder based on the proposal (holder receives offer)
3. Holder sends a request to the issuer (issuer receives request)
4. Issuer sends credential to holder (holder receives credentials)
5. Holder stores credential (holder sends acknowledge to issuer)
6. Issuer receives acknowledge

Step 1 is optional and the flow can start at step 2. Optionally, the entire offer part can be skipped and then the flow starts at step 3. Below, we run the entire process to demonstrate the entire possible flow.

The steps can be very confusing the first time around (especially when multiple VC are invovled). To keep track of the particular VC flow, each agent assigns a `cred_ex_id` value to the object. Note that both agents can find the existing state machines they are handling using some shared value, e.g., their `connection_id`.

#### Step 1: Holder sends proposal

Desired state transitions:

* Alice: None > proposal-received
* Bob: None > proposal-sent

Bob sends a proposal:

```bash
curl -X POST http://localhost:11001/issue-credential-2.0/send-proposal \
 -H "Content-Type: application/json" \
 -d '{
  "comment": "I want this",
  "connection_id": "644cd762-bf83-4ed4-89f2-221fc220c3cf",
  "credential_preview": {
    "@type": "issue-credential/2.0/credential-preview",
    "attributes": [
      {
        "mime-type": "plain/text",
        "name": "name", 
        "value": "Bob"
      },
      {
        "mime-type": "plain/text",
        "name": "age", 
        "value": "30"
      }
    ]
  },
  "filter": {
    "indy": {}
  }
}'
```

returns the Credential Exchange Record (CER). We note the `cred_ex_id`:

```bash
curl -X GET "http://localhost:11001/issue-credential-2.0/records?connection_id=644cd762-bf83-4ed4-89f2-221fc220c3cf&state=proposal-sent" -H  "accept: application/json" | jq | grep cred_ex_id
>   "cred_ex_id": "c56a5b63-add1-4b58-a307-fde6ff0e8e80"
```

#### Step 2: Issuer sends an offer

Desired state transitions:

* Alice: proposal-received > offer-sent
* Bob: proposal-sent > offer-received

Alice receives the proposal and assigns it a `cred_dx_id`
```bash
curl -X GET "http://localhost:11000/issue-credential-2.0/records?connection_id=50a1c481-b87b-4ea5-a48b-9fd9382322f9&state=proposal-received" -H  "accept: application/json" | jq | grep cred_ex_id
>   "cred_ex_id": "18953612-0d25-41c9-ad08-8a440b9b8774"
```

Note how Alice's credential exchange id (`18953612-0d25-41c9-ad08-8a440b9b8774`) is not the same as Bob's (`c56a5b63-add1-4b58-a307-fde6ff0e8e80`).

Alice then sends an offer to Bob:
```bash
curl -X POST http://localhost:11000/issue-credential-2.0/records/18953612-0d25-41c9-ad08-8a440b9b8774/send-offer \
  -H "Content-Type: application/json"
```

#### Step 3: Holder sends a request

Desired state transitions:

* Alice: offer sent > request-received
* Bob: offer-received > request-sent

```bash
curl -X POST http://localhost:11001/issue-credential-2.0/records/c56a5b63-add1-4b58-a307-fde6ff0e8e80/send-request
```

#### Step 4: Issuer sends credential

Desired state transitions:

* Alice: request-received > credential-issued
* Bob: request-sent > credential-received

```bash
curl -X POST http://localhost:11000/issue-credential-2.0/records/18953612-0d25-41c9-ad08-8a440b9b8774/issue \
  -H "Content-Type: application/json" -d '{"comment": "Please have this"}'
```

#### Step 5 & 6: Holder stores credential and confirms receipt

Desired state transitions:

* Alice: credential-issued > None
* Bob: credential-received > None

```bash
curl -X POST http://localhost:11001/issue-credential-2.0/records/c56a5b63-add1-4b58-a307-fde6ff0e8e80/store \
  -H "Content-Type: application/json" -d '{}'
```

Bob now stores the credential and notifies Alice. 

To see Bob's VC:

```bash
curl -X GET "http://localhost:11001/credentials" -H  "accept: application/json"
```

which returns:

```JSON
{
  "results": [
    {
      "referent": "4a988ac0-501e-4d2e-9996-535db9633bbc",
      "attrs": {
        "name": "Bob",
        "age": "30"
      },
      "schema_id": "PLEVLDPJQMJvPLyX3LgB6S:2:basic-schema:1.0",
      "cred_def_id": "PLEVLDPJQMJvPLyX3LgB6S:3:CL:8:default",
      "rev_reg_id": null,
      "cred_rev_id": null
    }
  ]
}
```

### Automating and debugging connections and credential flows with ACA-Py

To debug, you can set the flag `--debug-credentials` in ACA-Py and it will log information to the console. 

The endpoint `/issue-credential-2.0/send` sets the flags `auto_offer` and `auto_issue` to `true`. The holder will automatically accept offers and turn them into requests, making it easier to automate the process from the issuers's dev standpoint. 

The entire credential flow can be automated further with:

* `--auto-respond-credential-proposal`
* `--auto-respond-credential-offer`
* `--auto-respond-credential-request`
* `--auto-store-credential`

The credential exchange record will be automatically removed after issuing the corresponding credential. Thi can be disabled usign the flag `--preserve-exchange-records` in ACA-py.

This concludes Lab 2.

## Lab 3.

In this lab, we will focus on the more realistic case of Bolagsverket issuing some verifiable credentials in response to a company errand. The guide for the flow will be a process on verksamt.se

### Setting up the VDR and the agents

#### Initial configuration

The following is the content of the respective `config_<>.yaml` files.

```YAML
label: Bolagsverket
admin: [0.0.0.0, 11000]
admin-insecure-mode: true
genesis-url: http://localhost:9000/genesis
inbound-transport:
   - [http, 0.0.0.0, 8000]
outbound-transport: http
endpoint: http://localhost:8000
wallet-type: indy
wallet-name: Bolagsverket1
debug-connections: true
auto-provision: true
seed: Bolagsverket00000000000000000001
wallet-key: secret
auto-accept-invites: true
auto-accept-requests: true
auto-respond-credential-proposal: true
auto-respond-credential-request: true
```
and 

```YAML
label: Aktiebolaget
admin: [0.0.0.0, 11001]
admin-insecure-mode: true
genesis-url: http://localhost:9000/genesis
inbound-transport:
   - [http, 0.0.0.0, 8001]
outbound-transport: http
endpoint: http://localhost:8001
wallet-type: indy
wallet-name: Aktiebolaget1
debug-connections: false
auto-provision: true
wallet-key: secret
auto-accept-invites: true
auto-accept-requests: true
auto-respond-credential-offer: true
auto-store-credential: true
```

#### Register Bolagsverket's public DID on the VDR 

We first register Bolagsverket's DID on the VDR.

`curl -X POST "http://localhost:9000/register" -d '{"seed": "Bolagsverket00000000000000000001", "role": "TRUST_ANCHOR", "alias": "Bolagsverket"}'`

We can confirm that it is registered by browsing http://localhost:9000/browse/domain?page=1&query=Bolagsverket

We note the `nym` generated: `h2aCQwapPeZtdoHH1VAh6`, which is the Public DID for Bolagsverket.

<img width="500" alt="The BV registration tx" src="https://user-images.githubusercontent.com/30799110/150135240-04575ae4-c4c8-453a-bf37-b461039df339.png">

#### Starting the agents 

We start the Bolagsverket agent from the configurations file:

`aca-py start --arg-file config_bv.yaml`

We start the Aktiebolaget agent from the configuration file:

`aca-py start --arg-file config_ab.yaml`

#### Registering schemas and credential definitions

Bolagsverket first need to register schemas and credential definitions for the various verifiable credentials it will issue. One important piece of information is the company registration certificate. In the example below, we will use [this e-certificate](https://bolagsverket.se/polopoly_fs/1.9371!/e-certificate-registration-eng.pdf) as a guide for our first schema.


```bash
curl -X POST http://localhost:11000/schemas \
-H 'Content-Type: application/json' \
-d '{
  "attributes": [
  "registration_number",
  "business_name",
  "address",
  "registered_office",
  "signatory_power",
  "board_members",
  "board_chair",
  "share_capital"
  ],
  "schema_name": "e-VC-reg",
  "schema_version": "1.0"
}'
```

You can find the schema browsing the VDR: http://localhost:9000/browse/domain?page=1&query=e-VC-reg

Alice now creates a credential definition with the newly created schema (takes ~10 seconds).

```bash
curl -X POST "http://localhost:11000/credential-definitions" \
  -H  "accept: application/json" \
  -H  "Content-Type: application/json" \
  -d '{"schema_id": "h2aCQwapPeZtdoHH1VAh6:2:e-VC-reg:1.0",  "tag": "default"}'
```

You can find the credential defintion on the VDR: http://localhost:9000/browse/domain?page=1&query=default

Bolagsverket now has what it needs to issue a company registration VC.

#### Forming a connection between Bolagsverket and Aktiebolaget

Create a non-public connection invite:

```bash
curl -X POST "http://localhost:11000/out-of-band/create-invitation" \
  -H "Content-Type: application/json" \
  -d '{"handshake_protocols": ["did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/didexchange/1.0"], "use_public_did": false, "my_label": "Bolagsverket"}'
```

This returns a private invitation with a service endpoint. Bob needs to receive the invitation object that Alice generated above to do:

```bash
curl -X POST "http://localhost:11001/out-of-band/receive-invitation" \
   -H 'Content-Type: application/json' \
   -d '{
  "@type": "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/out-of-band/1.0/invitation",
  "@id": "936eb9e9-d57b-44fc-8411-f65b5291b0d5",
  "services": [
    {
      "id": "#inline",
      "type": "did-communication",
      "recipientKeys": [
        "did:key:z6Mkezj9erf4qJ6tWPQZmreQXKDgYKw8hANoC2jU2RrtApqM"
      ],
      "serviceEndpoint": "http://localhost:8000"
    }
  ],
  "label": "Bolagsverket",
  "handshake_protocols": [
    "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/didexchange/1.0"
  ]
}'
```

The two are now connected. Aktiebolaget's connection specific DID is found using `curl -X GET "http://localhost:11001/connections" -H  "accept: application/json" | jq | grep "connection_id"`

#### The company registration certificate verifiable credential flow

It just dawned on me that the schema name should probably not include certificate. Will fix later.

Aktiebolaget will now submit a credential proposal.

```bash
curl -X POST http://localhost:11001/issue-credential-2.0/send-proposal  -H "Content-Type: application/json"  -d '{
  "comment": "I want a company registration VC",
  "connection_id": "dfd0e463-8f61-4297-b76f-85341567720a",
  "credential_preview": {
    "@type": "issue-credential/2.0/credential-preview",
    "attributes": [
       {
        "mime-type": "text/plain",
        "name": "share_capital", 
        "value": "KSEK 50"
      },
       {
        "mime-type": "text/plain",
        "name": "registration_number", 
        "value": "556907-XXXX"
      },
      {
        "mime-type": "text/plain",
        "name": "board_chair", 
        "value": "820726-2398 Al Samed, Mohamed, Stjärnallén 40, 851 81 SUNDSVALL"
      },
      {
        "mime-type": "text/plain",
        "name": "board_members", 
        "value": "8207728-2397 Andersson, Filip, Atmosfärstigen 20, 851 81 SUNDSVALL"
      },
      {
        "mime-type": "text/plain",
        "name": "signatory_power", 
        "value": "The board of directors is inteitled to sign on behalf of the company. Signatory power individually by the board members."
      },
      {
        "mime-type": "text/plain",
        "name": "business_name", 
        "value": "Företaget i Sundsvall Aktiebolag"
      },
      {
        "mime-type": "text/plain",
        "name": "address", 
        "value": "Norrskensvägen 12, 851 81 SUNDSVALL"
      },
      {
        "mime-type": "text/plain",
        "name": "registered_office", 
        "value": "Sundsvall"
      }
    ]
  },                              
  "filter": {                          
    "indy": {}
  }    
}'
```

To see Bob's VC:

curl -X GET "http://localhost:11001/credentials" -H  "accept: application/json"
