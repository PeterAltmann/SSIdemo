# Learning ACA-Py

The [readthedocs](https://readthedocs.org/projects/aries-cloud-agent-python/downloads/pdf/latest/) are a bit "heavy" to browse if the goal is a quick POC. 

`aca-py` has two positional arguments:

1. `provision` used to provision and agent
2. `start` used to start a new agent process

## Provision

```bash
aca-py provision -h
usage: aca-py provision [-h] [--auto-disclose-features] [--disclose-features-list DISCLOSE_FEATURES_LIST] [--arg-file ARG_FILE]
                        [--plugin <module>] [--plugin-config PLUGIN_CONFIG] [-o <KEY=VALUE> [<KEY=VALUE> ...]] [--storage-type <storage-type>]
                        [-e <endpoint> [<endpoint> ...]] [--profile-endpoint <profile_endpoint>] [--read-only-ledger]
                        [--tails-server-base-url <tails-server-base-url>] [--tails-server-upload-url <tails-server-upload-url>]
                        [--notify-revocation] [--monitor-revocation-notification] [--ledger-pool-name <ledger-pool-name>]
                        [--genesis-transactions <genesis-transactions>] [--genesis-file <genesis-file>] [--genesis-url <genesis-url>]
                        [--no-ledger] [--ledger-keepalive LEDGER_KEEPALIVE] [--ledger-socks-proxy <host:port>]
                        [--genesis-transactions-list <genesis-transactions-list>] [--log-config <path-to-config>] [--log-file <log-file>]
                        [--log-level <log-level>] [--mediator-invitation <invite URL to mediator>] [--mediator-connections-invite]
                        [--seed <wallet-seed>] [--wallet-local-did] [--wallet-key <wallet-key>] [--wallet-rekey <wallet-rekey>]
                        [--wallet-name <wallet-name>] [--wallet-type <wallet-type>] [--wallet-storage-type <storage-type>]
                        [--wallet-storage-config <storage-config>] [--wallet-key-derivation-method <key-derivation-method>]
                        [--wallet-storage-creds <storage-creds>] [--replace-public-did] [--recreate-wallet]
```

Below, each option is explained.

**General**
* `--auto-disclose features` and `--disclosure-features-list` comes from [aries rfc 0557](https://github.com/hyperledger/aries-rfcs/blob/main/features/0557-discover-features-v2/README.md). It is a service discovery protocol that allows one agent to query another for its features.
* `--arg-file` Args that start with `--` (eg. `--admin`) can be set in a config file. See an [example here](https://github.com/hyperledger/aries-cloudagent-python/issues/1027).
*  `--plugin` and `--plugin-config` are used to load an external plugin or a configuration file for plugins. See [here](https://githubhelp.com/swcurran/aries-acapy-plugin-toolbox) and [here](https://start-here.hyperledger.org/pull-requests/hyperledger/aries-acapy-plugin-toolbox) for examples.
*  `-o` sets the plugin configuration options
*  `--storage-type` specifies the storage interface to store the internal state (ie wallet storage). Supported are `basic` or `indy`. 
*  `--e` and `--profile-endopoint` specifies the endpoints to be put into the DIDDocs, and the profile endpoint for the public DID. Enpoints can be directly to the agent, or to a mediator of the agent.
* `--read-only-ledger` sets the VDR to read only to prevent accidental updates (useful when running agents on production networks and networks where VDR writes incur costs).

**Revocation**
* `--tails-server-base-url` and `--tails-server-upload-url` sets the url for the tails server for use and for upload. The tails server is used for revocation as explained [here](https://github.com/hyperledger/aries-cloudagent-python/blob/main/docs/GettingStartedAriesDev/CredentialRevocation.md). Read more about the tails file [here](https://hyperledger-indy.readthedocs.io/projects/sdk/en/latest/docs/concepts/revocation/cred-revocation.html).
* `--notify-revocation` and `--monitor-revocation-notification` speicifies that aca-py will notify credential recipient of revocation, and emit a webhook on recieved revocation.

**Ledger**
* `--ledger-pool-name` is the name of the indy pool to open. Useful when multiple indy pools are running.
* `--genesis-transactions` specifies the genesis transaction to use to connect to a Hyperledger Indy agent.
* `--genesis-file` and `--genesis-url` specifies the file / url for where the genesis transaction can be found
* `--no-ledger` runs aca-py without a ledger and allows running in no ledger mode.
* `--ledger-keepalive` specifies the amount of time to keep the connection to the ledger open
* `--ledger-socks-proxy` specifies a socks proxy for cases where aca-py is running in a corporate/private network behind a corporate proxy
* `--genesis-transactions-list` loads a YALM configuration foc connecting to multiple HyperLedger Indy ledgers.

**Logging**
* `--log-config` specifies a custom logging profile
* `--log-file` overrides the output destination of the root logger as defined by the log config file
* `--log-level` specifies the logging level to `debug`, `info`, `warning`, `error`, or `critical`.


**Mediation**
* `--mediator-invitation` connects to a mediator and sets it up as a default mediator
* `--mediator-connections-invite` connect to a mediator through an invitation.

Wallet:
* `--seed` Specifies the seed to use for the creation of a DID.
* `--wallet-local-did` provisions the wallet with a local DID from the `-seed` parameter instead of the public DID from a VDR.
* `--wallet-key` sets the master key used to open the wallet
* `--wallet-rekey` sets the master key used to rotate and to open the wallet the next time
* `--wallet-name` specifies the wallet name to be used by the agent.
* `--wallet-type` specifies the type of Indy wallet provider to use. Supported storage types are `basic` and `indy`.
* `--wallet-storage-type` specifies the type of Indy wallet backend to use. Supported internal storage types are `basic` (memory), `default` (sqlite), and `postgres_storage`.
* `--wallet-storage-config` required if you are for using `postgres_storage` wallet as configuration.
* `--wallet-key-derivation-method` specifies the key derivation method used for wallet encryption. If RAW key derivation method is used, also `--wallet-key` parameter is expected.
* `--wallet-storage-creds` specifies the storage credentials to use for the wallet. Required if you are for using `postgres_storage` wallet.
* `--replace-public-did` replaces a public DID with a new did from the `--seed` parameter.
* `--recreate-wallet` if an existing wallet exists with the same name, remove and recreate it during provisioning.

TODO 
1. Understand difference between `--storage-type` options
2. Understand the difference between `wallet-type` options
3. Revocation
4. Mediators

## Start

```bash
aca-py start -h
usage: aca-py start [-h] [--admin <host> <port>] [--admin-api-key <api-key>] [--admin-insecure-mode] [--no-receive-invites]
                    [--help-link <help-url>] [--webhook-url <url#api_key>] [--admin-client-max-request-size ADMIN_CLIENT_MAX_REQUEST_SIZE]
                    [--debug] [--debug-seed <debug-did-seed>] [--debug-connections] [--debug-credentials] [--debug-presentations] [--invite]
                    [--connections-invite] [--invite-label <label>] [--invite-multi-use] [--invite-public]
                    [--invite-metadata-json <metadata-json>] [--test-suite-endpoint <endpoint>] [--auto-accept-invites] [--auto-accept-requests]
                    [--auto-respond-messages] [--auto-respond-credential-proposal] [--auto-respond-credential-offer]
                    [--auto-respond-credential-request] [--auto-respond-presentation-proposal] [--auto-respond-presentation-request]
                    [--auto-store-credential] [--auto-verify-presentation] [--auto-disclose-features]
                    [--disclose-features-list DISCLOSE_FEATURES_LIST] [--arg-file ARG_FILE] [--plugin <module>] [--plugin-config PLUGIN_CONFIG]
                    [-o <KEY=VALUE> [<KEY=VALUE> ...]] [--storage-type <storage-type>] [-e <endpoint> [<endpoint> ...]]
                    [--profile-endpoint <profile_endpoint>] [--read-only-ledger] [--tails-server-base-url <tails-server-base-url>]
                    [--tails-server-upload-url <tails-server-upload-url>] [--notify-revocation] [--monitor-revocation-notification]
                    [--ledger-pool-name <ledger-pool-name>] [--genesis-transactions <genesis-transactions>] [--genesis-file <genesis-file>]
                    [--genesis-url <genesis-url>] [--no-ledger] [--ledger-keepalive LEDGER_KEEPALIVE] [--ledger-socks-proxy <host:port>]
                    [--genesis-transactions-list <genesis-transactions-list>] [--log-config <path-to-config>] [--log-file <log-file>]
                    [--log-level <log-level>] [--auto-ping-connection] [--auto-accept-intro-invitation-requests] [--invite-base-url <base-url>]
                    [--monitor-ping] [--monitor-forward] [--public-invites] [--timing] [--timing-log <log-path>] [--trace]
                    [--trace-target <trace-target>] [--trace-tag <trace-tag>] [--trace-label <trace-label>] [--preserve-exchange-records]
                    [--emit-new-didcomm-prefix] [--emit-new-didcomm-mime-type] [--exch-use-unencrypted-tags] [--auto-provision]
                    [-it <module> <host> <port>] [-ot <module>] [-oq OUTBOUND_QUEUE] [-l <label>] [--image-url IMAGE_URL]
                    [--max-message-size <message-size>] [--enable-undelivered-queue] [--max-outbound-retry MAX_OUTBOUND_RETRY]
                    [--ws-heartbeat-interval <interval>] [--ws-timeout-interval <interval>] [--mediator-invitation <invite URL to mediator>]
                    [--mediator-connections-invite] [--open-mediation] [--default-mediator-id <mediation id>] [--clear-default-mediator]
                    [--seed <wallet-seed>] [--wallet-local-did] [--wallet-key <wallet-key>] [--wallet-rekey <wallet-rekey>]
                    [--wallet-name <wallet-name>] [--wallet-type <wallet-type>] [--wallet-storage-type <storage-type>]
                    [--wallet-storage-config <storage-config>] [--wallet-key-derivation-method <key-derivation-method>]
                    [--wallet-storage-creds <storage-creds>] [--replace-public-did] [--recreate-wallet] [--multitenant]
                    [--jwt-secret <jwt-secret>] [--multitenant-admin] [--multitenancy-config <multitenancy-config>]
                    [--endorser-protocol-role <endorser-role>] [--endorser-invitation <endorser-invitation>]
                    [--endorser-public-did <endorser-public-did>] [--endorser-alias <endorser-alias>] [--auto-request-endorsement]
                    [--auto-endorse-transactions] [--auto-write-transactions] [--auto-create-revocation-transactions]
```

Let us focus on the argumnents that differ from the provision mode.

**General**
* `--admin`, `--admin-api-key`, or `--admin-insecure-mode` are used to specify the place where the admin server will run. The server either requires an API key, or to be run in insecure mode.
* `--no-receive-invites` prevents an agent from receiving invites.
* `--help-link` provides an URL to an admin interface help page that a controller can request from an admin agent.
* `--webhook-url` sends webhooks containing internal state changes
* `--admin-client-max-request-size` sets the maximum client request size to an admin server
* `--auto-provision` if the requested profile does not exist, then it will be initialized when the given parameters

**Debug**
* `--debug` a remote debugging service
* `--debug-seed` the debug seed
* `--debug-connections` additional logging for connections
* `--debug-credentials` additional logging for credential exchanges

**Invitation**
* `--invite` after startup, the agent generates and prints a new out-of-band protocol connection invitation URL
* `--connections-invite` after startup, the agent generates a new connections protocol connection invitation URL
* `--invite-label` labels the generated invitation
* `--invite-multi-use` flags the generated invite for multi-use
* `--invite-public` flags the invite as public
* `--invite-metadata-json` adds metadata json to the invititation if created with the `--invite` argument
* `--test-suite-endpoint` URL endpoint for msgs sent to the test suite agent
* `--auto-accept-invites` automatically accept invites
* `--auto-accept-requests` automatically accepts connection requests
* `--auto-respond-messages` set up an autoresponse to received messages
* `--auto-respond-credential-proposal` automatically responds to credential requests with a corresponding credential offer
* `--auto-respond-credential-offer` automatically responds to an Indy credential offer with a credential request
* `--auto-respond-credential-request` automatically responds with a corresponding credential
* `--auto-respond-presentation-proposal` automatically respond with a presentation request
* `--auto-respond-presentation-request` automatically respond with a presentation
* `--auto-store-credential` automatically stores a credential upon receipt
* `--auto-verify-presentation` automatically verify a presentation upon receipt

**Protocol**
* `--auto-ping-connection` pings the agent to confirm connection is active after a connection response is accepted.
* `--auto-accept-intro-invitation-requests` automatically accept introduction invitations
* `--invite-base-url` base URL to use when formatting connection invitations in URL format
* `--monitor-ping` send a webhook when a ping is received
* `--monitor-forward` send a webhook when a forward is received
* `--public-invites` send invitation out, and receive connection requests using the piblic DID for the agent
* `--timing` and `timing-log` used for timing information which can be sent in the response messages
* `--trace`, `--trace-target`, `--trace-tag`, and `--trace-label` are used to generate specific tracing events and tag and label these event traces.
* `--preserve-exchange-records` allows an agent to keep the exchange records even after an exchange has completed
* `--emit-new-didcomm-prefix` emitt protocol message with new DIDComm prefix (eg., https://didcomm.org instead of `did:sov:<id>;spec`)
* `--emit-new-didcomm-mime-type` send packed agent msg with DIDComm MIME type. See [aries rfc 0044](https://github.com/hyperledger/aries-rfcs/blob/main/features/0044-didcomm-file-and-mime-types/README.md).
* `--exch-use-unencrypted-tags` store tags for exchange protocols using unencrypted rather than encrypted tags

**Transport**
* `--inbound-transport` and `--outband-transport` sets the inbound and outbound transports on which an agent listens to msgs and sends msgs. Multiple interfaces possible by setting the argument multiple times.
* `--outbound-queue` sets the location of the outbound queue engine
* `--label` specifies the label for the agent
* `--image-url` specifies the image url for the agent
* `--max-message-size` sets max inbound message size for agent
* `--enable-undelivered-queue` enables an agent to hold outbound msgs to an agent that does not have an endpoint
* `--max-outbound-retry` sets the max retry attempts for undelivered outbound messages
* `--ws-heartbeat-interval` when using Websocket Inbound Transport, sends a ws ping as specified in seconds
* `--ws-timeout-interval` when using Websocket Inbound Transport, timeout the connection after specified seconds

**Mediation**
* `--open-mediation` enables automatic granting of mediation. Agents may request msg mediation.
* `--default-mediator-id` sets the default mediator by ID
* `--clear-default-mediator` clear the stored default mediator

**Wallet functions are the same as provision**

**Multitenant**
* `--multitenant` enables multitenant mode
* `--jwt-secret` specifies the secret to be used for JSON Web Token creation and verification
* `--multitenant-admin` enables the multi-tenant admin API
* `--multitenancy-config` specifies multitenance configuration (eg., sets `"wallet_type": ""` and `"wallet_name": ""`).

**Endorsement**
* `--endorser-protocol-role` specify the role of `author` or `endorser` which this agent has in the network. Authors request tx endorsement from endorsers.
* `--endorser-invitation` for tx authors, specifies the endorser agent to connect to for tx endorsement 
* `--endorser-public-did` for tx authors, specifies the endorser agent DID
* `--endorser-alias` for tx authors, sets an alias the the endorser connection used to endorse a tx
* `--auto-request-endorsement` for tx authors, automatically requests endorsement for all tx
* `--auto-endorse-transaction` for endorsers, automatically endorses any endorsement request
* `--auto-write-transactions` for authors, specifies whether to automatically write any endorsed tx
* `--auto-create-revocation-transactions` for authors, specify whether to automatically create transactions for a cred def's revocation registry.

TODO
1. Difference between connection invite and a connection request
2. What is an introduction invitation?
3. What is a forward?
4. How does mediation work?
5. What is multitentant?
6. How does the `--auto-create-revocation-transactions` work?
