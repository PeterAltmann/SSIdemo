# Lab 1. Establishing a connection between two agents

To establish a connection, we need 

1. A working VDR
2. One agent issuing a connection invitation
3. Another agent acceping the connection invitation

There exist several ways to create a working VDR. The options to run a VDR on a local machine, on a remote server we control, and connecting to a pre-existing test oriented VDR are shown below. Once the VDR is up and running, we can focus on the agents. The demo inluded in ACA-Py includes two agents that seek to connect with each other. For now, we will only run the demo to see what a connection flow looks like. The first of two agents are Faber, who issues the connection request and is a made up university. The second agent is Alice, a recently graduated university student who wants her diploma as a VC.

## For a local VDR

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

## For a remote VDR

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

## For a public VDR

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

## The connection

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
