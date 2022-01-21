# Lab 4. DIDComm and Connections in ACA-Py

In this lab the ways to connect agents is explored. Note that the ways to connect agents is evolving in both Aries and at the Decentralized Identity Foundation (cf. [DIDComm Messaging](https://identity.foundation/didcomm-messaging/spec/)).

## DIDComm Messaging

To understand the way agents connect, a woring understanding of DIDComm Messaging (DIDComm from now on) is helpful. DIDComm is a communication methodology built to avoid centralized assumptions and to work with DIDs. DIDComm aims to enable higher-order protocols for message transport. It does so by exchanging a series of messages with the following goals in mind:

1. Resolve a DID to get endpoints where messages can be delivered
2. Key exchange required to establish a private communications channel

The paradigm for DIDComm is message-based, asynchronous, and simplex. Simply put, DIDComm assumes a partly conencted world that is closer to e-mail than it is to the traditional web. A few more differences to note:

* Mutual authentication happens using the same process using public key cryptography (and not using certificate for server and something else for client that creates a session object)
* Security is provided at the message object level as opposed to the transport level (TLS)
* Sessionless (sessions can be built atop it)

<img src = 'https://identity.foundation/didcomm-messaging/collateral/didcomm-envelopes.png'>

Figure 1. DIDComm Messaging with two layers of packaging the plaintext.

A plaintext message structure looks like this:

```JSON
{
  "typ": "application/didcomm-plain+json",
  "id": "1234567890",
  "type": "<message-type-uri>",
  "from": "did:example:alice",
  "to": ["did:example:bob"],
  "created_time": 1516269022,
  "expires_time": 1516385931,
  "body": {
    "messagespecificattribute": "and its value"
  }
}
```

The above is sufficient understanding for the Lab. For more details, please see the [DIDComm Messaging specs](https://identity.foundation/didcomm-messaging/spec/). The specifications also detail how the recipient can know how to interpret the message object (using IANA media type strings that detail choices that fit into various profiles, e.g., `didcomm/aip2;env=rfc587`). 

## Connections in ACA-Py

ACA-Py implements the DIDComm v2 specs. In the previous Labs, we have relied on the Out-Of-Band (OOB) protocol to connect the agents. Commonly, OOB messages are presented in the form of a URL (or a QR code thereof).

Start two agents:

```bash
aca-py start --arg-file config_bv.yaml --public-invites --auto-ping-connections
aca-py start --arg-file config_ab.yaml --public-invites --auto-ping-connections
``` 

With `--public-invites`, we prepare the agents to accept connection invites based off a public DID on the VDR. Setting `--auto-ping-connections` forces the respective agent to send a ping to trigger a state change from response-sent to completed (cf [RFC23](https://github.com/hyperledger/aries-rfcs/tree/main/features/0023-did-exchange)). If auto ping is not set, a ping has to be sent manually to mark the connection invitation as complete.

**Note**: *In my testing, I could not disable the multi-use feature. Even when set to false, the Aktiebolaget agent could reuse the same invite multiple times. No idea why.*

### Single(?) use public invite 

Note that in AIP 1.0, ACA-Py uses [Aries RFC0160](https://github.com/hyperledger/aries-rfcs/blob/master/features/0160-connection-protocol/README.md). For AIP 2.0, ACA-Py will use [Aries RFC0023](https://github.com/hyperledger/aries-rfcs/tree/main/features/0023-did-exchange) (superseded RFC0160) and the OOB protocol detailed in [Aries RFC0023](https://github.com/hyperledger/aries-rfcs/blob/main/features/0434-outofband/README.md).

#### Bash
Bolagsverket creates a single use public invite:

```bash
curl -X POST "http://localhost:11000/connections/create-invitation?public=true" \
  -H  "accept: application/json" \
  -H  "Content-Type: application/json" -d '{
    "my_label": "BV public"
  }'
```

Note the `connection_id` in the reply: `eeab9754-dc17-435c-bd2f-2830502ac230`
The `rfc32_state` is now `invitation-sent`

Aktiebolaget can initiate a connection using the public invite.

```bash
curl -X POST "http://localhost:11001/connections/receive-invitation" -H  "accept: application/json" -H  "Content-Type: application/json" -d '{
  "@type": "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/connections/1.0/invitation",
    "@id": "c56d331c-3ca2-4f78-ab4d-bc7d07a92e0e",
    "did": "did:sov:h2aCQwapPeZtdoHH1VAh6",
    "label": "BV public"
}'
```

Once sent, the connection changes its `rfc23_state` to `request-sent`, and when auto ping is complete it changes state to `completed`. At this point, the agents are connected.

Show active connections using `curl -X GET "http://localhost:<admin port>/connections?state=completed" -H  "accept: application/json"`.

#### Swagger

Create invitation
1. Navigate to http://localhost:11000/api/doc#/connection/post_connections_create_invitation
2. Paste into body
```JSON
{
  "my_label": "BV public"
}
```
3. Name the invitation whatever you want in alias (or leave blank)
4. Leave multi use off for now
5. Set public to true

Receive invitation
1. Navigate to http://localhost:11001/api/doc#/connection/post_connections_receive_invitation
2. Paste into body
  ```JSON
    {
      "@type": "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/connections/1.0/invitation",
      "@id": "fed67f45-f680-49d1-9b27-9bca564440f1",
      "did": "did:sov:h2aCQwapPeZtdoHH1VAh6",
      "label": "BV public"
    }
  ```
3. Set alias to whatever

The agents are now connected.
