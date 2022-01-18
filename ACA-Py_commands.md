# Learning ACA-Py

The [readthedocs](https://readthedocs.org/projects/aries-cloud-agent-python/downloads/pdf/latest/) are a bit "heavy" to browse if the goal is a quick POC. 

`aca-py` has two positional arguments:

1. `provision` used to provision and agent
2. `start` used to start a new agent process

## Provision

```bash
aca-py provision -h
> usage: aca-py provision [-h] [--auto-disclose-features] [--disclose-features-list DISCLOSE_FEATURES_LIST] [--arg-file ARG_FILE]
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
* `--log-file`


TODO 

1. Understand difference between `--storage-type` options.
2. 
*  
