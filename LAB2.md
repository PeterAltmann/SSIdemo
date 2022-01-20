# Lab 2. An introduction to agents and wallets with ACA-Py

In Lab 1, we used a runner to create two agents, Faber and Alice, and set up a connection between them. The runner consists of an agent (an instance of ACA-Py) and the controller (the code that interacts with the HTTP Admin Endpoints of ACA-Py). Moving forward, we do not want to use the demo for experimenting with our agents and our wallets. We instead start ACA-Py instances ourselves, and will use `curl` commands to act as the controller. We could of course use any programming language to act as our controller as long as we are able to interact with Web Services.


## Installing ACA-Py

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

## Starting the credential issuer Alice

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

## Starting a credential holder Bob

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

## The issuer creates a schema and credential definition

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

## Non-public connection invitation

With a non-public connection, Alice sends over an object to Bob that contains all the information Bob needs without having to browse a VDR.

### Creating a non-public connection invite

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

## Issuing a Credential

Alice and Bob now share a connection. The credential issuance flow depends on which party initiates the process and with what. 

1. Holder sends a proposal to the issuer (issuer receives proposal)
2. Issuer sends an offer to the holder based on the proposal (holder receives offer)
3. Holder sends a request to the issuer (issuer receives request)
4. Issuer sends credential to holder (holder receives credentials)
5. Holder stores credential (holder sends acknowledge to issuer)
6. Issuer receives acknowledge

Step 1 is optional and the flow can start at step 2. Optionally, the entire offer part can be skipped and then the flow starts at step 3. Below, we run the entire process to demonstrate the entire possible flow.

The steps can be very confusing the first time around (especially when multiple VC are invovled). To keep track of the particular VC flow, each agent assigns a `cred_ex_id` value to the object. Note that both agents can find the existing state machines they are handling using some shared value, e.g., their `connection_id`.

### Step 1: Holder sends proposal

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

### Step 2: Issuer sends an offer

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

### Step 3: Holder sends a request

Desired state transitions:

* Alice: offer sent > request-received
* Bob: offer-received > request-sent

```bash
curl -X POST http://localhost:11001/issue-credential-2.0/records/c56a5b63-add1-4b58-a307-fde6ff0e8e80/send-request
```

### Step 4: Issuer sends credential

Desired state transitions:

* Alice: request-received > credential-issued
* Bob: request-sent > credential-received

```bash
curl -X POST http://localhost:11000/issue-credential-2.0/records/18953612-0d25-41c9-ad08-8a440b9b8774/issue \
  -H "Content-Type: application/json" -d '{"comment": "Please have this"}'
```

### Step 5 & 6: Holder stores credential and confirms receipt

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

## Automating and debugging connections and credential flows with ACA-Py

To debug, you can set the flag `--debug-credentials` in ACA-Py and it will log information to the console. 

The endpoint `/issue-credential-2.0/send` sets the flags `auto_offer` and `auto_issue` to `true`. The holder will automatically accept offers and turn them into requests, making it easier to automate the process from the issuers's dev standpoint. 

The entire credential flow can be automated further with:

* `--auto-respond-credential-proposal`
* `--auto-respond-credential-offer`
* `--auto-respond-credential-request`
* `--auto-store-credential`

The credential exchange record will be automatically removed after issuing the corresponding credential. Thi can be disabled usign the flag `--preserve-exchange-records` in ACA-py.

This concludes Lab 2.
