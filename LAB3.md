# Lab 3.

In this lab, we will focus on the more realistic case of Bolagsverket issuing some verifiable credentials in response to a company errand. The guide for the flow will be a process on verksamt.se

## Setting up the VDR and the agents

### Initial configuration

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

### Register Bolagsverket's public DID on the VDR 

We first register Bolagsverket's DID on the VDR.

`curl -X POST "http://localhost:9000/register" -d '{"seed": "Bolagsverket00000000000000000001", "role": "TRUST_ANCHOR", "alias": "Bolagsverket"}'`

We can confirm that it is registered by browsing http://localhost:9000/browse/domain?page=1&query=Bolagsverket

We note the `nym` generated: `h2aCQwapPeZtdoHH1VAh6`, which is the Public DID for Bolagsverket.

<img width="500" alt="The BV registration tx" src="https://user-images.githubusercontent.com/30799110/150135240-04575ae4-c4c8-453a-bf37-b461039df339.png">

### Starting the agents 

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

You can find the schema:

* browsing the VDR: http://localhost:9000/browse/domain?page=1&query=e-VC-reg
* using the schema id: `curl http://localhost:11000/schemas/<schema_id>`
* or through the `seqNo` value: `curl http://localhost:11000/schemas/<seqNo>`

Alice now creates a credential definition with the newly created schema (takes ~10 seconds).

```bash
curl -X POST "http://localhost:11000/credential-definitions" \
  -H  "accept: application/json" \
  -H  "Content-Type: application/json" \
  -d '{
  "schema_id": "h2aCQwapPeZtdoHH1VAh6:2:e-VC-reg:1.0",
  "tag": "default"
  }'
```

You can find the credential defintion:

* on the VDR: http://localhost:9000/browse/domain?page=1&query=default
* using the `/credential-definitions/{id}` endpoint
* or the corresponding curl command

Bolagsverket now has what it needs to issue a company registration VC.

#### Forming a connection between Bolagsverket and Aktiebolaget

For options on how to create agent connections, see [Lab 4](https://github.com/PeterAltmann/SSIdemo/blob/main/LAB4.md). In this lab, we create a non-public connection invite:

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

### The company registration verifiable credential flow

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
        "value": "The board of directors is entitled to sign on behalf of the company. Signatory power individually by the board members."
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

To see Bob's company registration VC:

`curl -X GET "http://localhost:11001/credentials" -H  "accept: application/json"` 

which returns 

```JSON
{
  "results": [
    {
      "referent": "eb53f090-ad9f-4e6f-89fe-c918b4e6c771",
      "attrs": {
        "address": "Norrskensvägen 12, 851 81 SUNDSVALL",
        "signatory_power": "The board of directors is entitled to sign on behalf of the company. Signatory power individually by the board members.",
        "share_capital": "KSEK 50",
        "board_members": "8207728-2397 Andersson, Filip, Atmosfärstigen 20, 851 81 SUNDSVALL",
        "registration_number": "556907-XXXX",
        "registered_office": "Sundsvall",
        "business_name": "Företaget i Sundsvall Aktiebolag",
        "board_chair": "820726-2398 Al Samed, Mohamed, Stjärnallén 40, 851 81 SUNDSVALL"
      },
      "schema_id": "h2aCQwapPeZtdoHH1VAh6:2:e-VC-reg:1.0",
      "cred_def_id": "h2aCQwapPeZtdoHH1VAh6:3:CL:23:default",
      "rev_reg_id": null,
      "cred_rev_id": null
    }
  ]
}

```

Aktiebolaget now has a VC of their company registration. In future labs, we take a closer look on how they can use this (and other VCs) when conducting business.
