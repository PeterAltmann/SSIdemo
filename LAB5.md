# Revocation

For revocation to work, you need to:

1. Start up the VDR
2. Start up the indy-tails server
3. Start agents with config that enables the use of tails

Issuer config file:
```bash
$ cat config_bv.yaml 
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
auto-respond-credential-offer: true
auto-respond-credential-request: true
auto-store-credential: true
public-invites: true
auto-ping-connection: true
tails-server-base-url: http://localhost:6543
preserve-exchange-records: true
```

Holder config file:
```bash
$ cat config_ab.yaml 
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
auto-respond-credential-proposal: true
auto-respond-credential-offer: true
auto-respond-credential-request: true
auto-store-credential: true
public-invites: true
auto-ping-connection: true
preserve-exchange-records: true
```

## Setting up the VDR and the issuer

I have the config files in `~/Bolagsverket/` 

The following script initiates the issuer (no, that is not my normal password):

```bash
#!/bin/sh

### sets the sudo cache and then issues a command in such a way that the pw is not logged in history
# echo 123456 | sudo -S -v
# sudo <command>
###

export HISTIGNORE='*sudo -S*' # does not save your password in bash history

### Option with clear start
Clear() {
  echo 123456 | sudo -S -v;
  [ "$(ls -A ~/.indy_client/wallet/)"  ] && sudo rm -r ~/.indy_client/wallet/* && echo "Wallet folder emptied" || echo "Wallet folder already empty";
}

"$@"

### Start the ledger and register agents on the ledger
echo 123456 | sudo -S -v
echo ""
echo "$(tput setaf 1)Starting VDR. Will take about 15 seconds. $(tput sgr 0)"
sudo ~/von-network/manage start
echo ""
sleep 15
echo "Note the Public DID of the agent"
curl -X POST "http://localhost:9000/register" -d '{"seed": "Bolagsverket00000000000000000001", "role": "TRUST_ANCHOR", "alias": "Bolagsverket"}'

### Start tails server for revocation
echo ""
echo ""
echo "Starting the revocation server ..."
echo ""
echo 123456 | sudo -S -v
sudo ~/indy-tails-server/docker/manage start
echo ""

### Prep and start issuer
aca-py start --arg-file ~/Bolagsverket/config_bv.yaml
```

Running `./wallet_start.sh Clear` will ensure you have a fresh start.

Register the schema:

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
}' | jq
```

And the cred def with revocation supported

```bash
curl -X POST "http://localhost:11000/credential-definitions" \
  -H  "accept: application/json" \
  -H  "Content-Type: application/json" \
  -d '{
  "schema_id": "h2aCQwapPeZtdoHH1VAh6:2:e-VC-reg:1.0",
  "tag": "default",
  "revocation_registry_size": 15,
  "support_revocation": true
  }' | jq
```

## Request a VC

Start the holder:

```bash
$ aca-py start --arg-file config_ab.yaml 
```

Connect with the issuer:

```bash
curl -X POST \
  -H  "accept: application/json" \
  "http://localhost:11001/didexchange/create-request?their_public_did=h2aCQwapPeZtdoHH1VAh6&my_endpoint=http%3A%2F%2F0.0.0.0%3A8001" | jq
```

First we get the connection ID:
```bash
curl -X GET "http://localhost:11001/connections" -H  "accept: application/json" | jq | grep connection_id\":
```

Then the holder requests the VC:

```bash
curl -X POST http://localhost:11001/issue-credential-2.0/send-proposal  -H "Content-Type: application/json"  -d '{
  "comment": "I want a company registration VC",
  "connection_id": "ebbacd0c-ce40-4213-82cb-840af97b50dc",
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
}' | jq
```

Check if the holder has the credential:

```bash
curl -X GET "http://localhost:11001/credentials" -H "accept: application/json"
```

## Revocation status and state updates

### Holder
From the holder's perspective:

```bash
$ curl -X GET "http://localhost:11001/credential/revoked/8a12ea3b-a1a2-4b82-849f-411cf4245fcd" -H  "accept: application/json" | jq
{
  "revoked": false
}
```

### Issuer

Get the `revoc_reg_id`

```bash
curl -X GET "http://localhost:11000/revocation/active-registry/h2aCQwapPeZtdoHH1VAh6%3A3%3ACL%3A8%3Adefault" -H  "accept: application/json" | jq | grep revoc_reg_id
```

With the revocation registry id we can check how many VCs have been issued against that registry:
```bash
$ curl -X GET "http://localhost:11000/revocation/registry/h2aCQwapPeZtdoHH1VAh6%3A4%3Ah2aCQwapPeZtdoHH1VAh6%3A3%3ACL%3A8%3Adefault%3ACL_ACCUM%3A5b4b751c-7b3b-41da-8f39-f7770ba80c42/issued" -H  "accept: application/json"
{
  "result": 1
}
```

We can check the status:

```bash
$ curl -X GET "http://localhost:11000/revocation/credential-record?cred_rev_id=1&rev_reg_id=h2aCQwapPeZtdoHH1VAh6%3A4%3Ah2aCQwapPeZtdoHH1VAh6%3A3%3ACL%3A8%3Adefault%3ACL_ACCUM%3A5b4b751c-7b3b-41da-8f39-f7770ba80c42" -H  "accept: application/json" | jq | grep state
"state": "issued"
```

Let us revoke the VC:

```bash
curl -X POST "http://localhost:11000/revocation/revoke" -H  "accept: application/json" -H  "Content-Type: application/json" -d "{  \"connection_id\": \"6ee9922b-ec12-4e9c-9c49-5781d5de2064\",  \"cred_rev_id\": \"1\",  \"notify\": false,  \"publish\": true,  \"rev_reg_id\": \"h2aCQwapPeZtdoHH1VAh6:4:h2aCQwapPeZtdoHH1VAh6:3:CL:8:default:CL_ACCUM:5b4b751c-7b3b-41da-8f39-f7770ba80c42\"}"
```

```bash
curl -X GET "http://localhost:11000/revocation/credential-record?cred_rev_id=1&rev_reg_id=h2aCQwapPeZtdoHH1VAh6%3A4%3Ah2aCQwapPeZtdoHH1VAh6%3A3%3ACL%3A8%3Adefault%3ACL_ACCUM%3A5b4b751c-7b3b-41da-8f39-f7770ba80c42" -H  "accept: application/json" | jq | grep state
"state" : "revoked"
```

## Holder is notifed and checks

```bash
$ curl -X GET "http://localhost:11001/credential/revoked/8a12ea3b-a1a2-4b82-849f-411cf4245fcd" -H  "accept: application/json" | jq
{
  "revoked": true
}
```

End of lab.
