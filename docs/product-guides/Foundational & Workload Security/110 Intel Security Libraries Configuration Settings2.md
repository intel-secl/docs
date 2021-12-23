# Trust Agent

## Installation Answer File Options

| Key                               | Description                                                  | Sample Value                                                 |
| --------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| AAS\_API\_URL                     | API URL for Authentication Authorization Service (AAS).      | AAS\_API\_URL=https://{host}:{port}/aas/v1                   |
| AUTOMATIC\_PULL\_MANIFEST         | Instructs the installer to automatically pull application-manifests from HVS similar to tagent setup get-configured-manifest | AUTOMATIC\_PULL\_MANIFEST=Y                                  |
| AUTOMATIC\_REGISTRATION           | Instructs the installer to automatically register the host with HVS similar to running tagent setup create-host and tagent setup create-host-unique-flavor. | AUTOMATIC\_REGISTRATION=Y                                    |
| BEARER\_TOKEN                     | JWT from AAS that contains "install" permissions needed to access ISecL services during provisioning and registration | BEARER\_TOKEN=eyJhbGciOiJSUzM4NCIsjdkMTdiNmUz...             |
| CMS\_BASE\_URL                    | API URL for Certificate Management Service (CMS).            | CMS\_BASE\_URL=https://{host}:{port}/cms/v1                  |
| CMS\_TLS\_CERT\_SHA384            | SHA384 Hash sum for verifying the CMS TLS certificate.       | CMS\_TLS\_CERT\_SHA384=bd8ebf5091289958b5765da4...           |
| HVS\_API\_URL                     | The url used during setup to request information from HVS.   | HVS\_API\_URL=https://{host}:{port}/hvs/v2                   |
| PROVISION\_ATTESTATION            | When present, enables/disables whether tagent setup is called during installation. If trustagent.env is not present, the value defaults to no ('N'). | PROVISION\_ATTESTATION=Y                                     |
| SAN\_LIST                         | CSV list that sets the value for SAN list in the TA TLS certificate. Defaults to 127.0.0.1. | SAN\_LIST=10.123.100.1,201.102.10.22,mya.example.com         |
| TA\_TLS\_CERT\_CN                 | Sets the value for Common Name in the TA TLS certificate. Defaults to CN=trustagent. | TA\_TLS\_CERT\_CN=Acme Trust Agent 007                       |
| TPM\_OWNER\_SECRET                | Default is null.  Can be any string of characters.  Use the "hex:" prefix to force hex characters rather than a string.<br />hex:0164837f83..." | TPM\_OWNER\_SECRET=625d6...<br />Starting in Intel SecL-DC 4.0, this value will now default to null unless a secret is specified.  Using a null TPM ownership secret is recommended.  The Trust Agent now only requires TPM ownership during Trust Agent provisioning. |
| TPM\_QUOTE\_IPV4                  | When enabled (=y), uses the local system's ip address as a salt when processing a quote nonce. This field must align with the configuration of HVS. | TPM\_QUOTE\_IPV4=no                                          |
| TA\_SERVER\_READ\_TIMEOUT         | Sets tagent server ReadTimeout. Defaults to 30 seconds.      | TA\_SERVER\_READ\_TIMEOUT=30                                 |
| TA\_SERVER\_READ\_HEADER\_TIMEOUT | Sets tagent server ReadHeaderTimeout. Defaults to 30 seconds. | TA\_SERVER\_READ\_HEADER\_TIMEOUT=10                         |
| TA\_SERVER\_WRITE\_TIMEOUT        | Sets tagent server WriteTimeout. Defaults to 10 seconds.     | TA\_SERVER\_WRITE\_TIMEOUT=10                                |
| TA\_SERVER\_IDLE\_TIMEOUT         | Sets tagent server IdleTimeout. Defaults to 10 seconds.      | TA\_SERVER\_IDLE\_TIMEOUT=10                                 |
| TA\_SERVER\_MAX\_HEADER\_BYTES    | Sets tagent server MaxHeaderBytes. Defaults to 1MB(1048576)  | TA\_SERVER\_MAX\_HEADER\_BYTES=1048576                       |
| TA\_ENABLE\_CONSOLE\_LOG          | When set true, tagent logs are redirected to stdout. Defaults to false | TA\_ENABLE\_CONSOLE\_LOG=true                                |
| TRUSTAGENT\_LOG\_LEVEL            | The logging level to be saved in config.yml during installation ("trace", "debug", "info"). | TRUSTAGENT\_LOG\_LEVEL=debug                                 |
| TRUSTAGENT\_PORT                  | The port on which the trust-agent service will listen.       | TRUSTAGENT\_PORT=10433                                       |

## Configuration Options

The Trust Agent configuration settings are managed in
`/opt/trustagent/configuration/config.yml`

| **Setting**                                   | **Description**                                              |
| --------------------------------------------- | ------------------------------------------------------------ |
| tpmquoteipv4: true                            | When enabled, the Trust Agent will perform an additional hash of the nonce using the bytes from the Trust Agent server IP when returning TPM quotes. This should always be set to True. |
| logging:                                      |                                                              |
| loglevel: info                                | Defines the Trust Agent logging level                        |
| logenablestdout: false                        | If set to True, the Trust Agent will log to stdout. By default this is False and the logs are sent to /var/log/trustagent/trustagent.log |
| logentrymaxlength: 300                        | Defines the maximum length of a single log entry             |
| webservice:                                   |                                                              |
| port: 1443                                    | Defines the port on which the Trust Agent API server will listen |
| readtimeout: 30s                              |                                                              |
| readheadertimeout: 10s                        |                                                              |
| writetimeout: 10s                             |                                                              |
| idletimeout: 10s                              |                                                              |
| maxheaderbytes: 1048576                       |                                                              |
| hvs:                                          |                                                              |
| url: https://0.0.0.0:8443/hvs/v2              | Defines the baseurl for the Verification Service             |
| tpm:                                          |                                                              |
| aas:                                          |                                                              |
| baseurl: https://0.0.0.0:8444/aas/v1/         | Defines the base URL for the AAS                             |
| cms:                                          |                                                              |
| baseurl: https://0.0.0.0:8445/cms/v1          | Defines the base URL for the CMS                             |
| tlscertdigest: 330086b3...ae477c8502          | Defines the SHA383 hash of the CMS TLS certificate           |
| tls:                                          |                                                              |
| certsan: 10.1.2.3,server.domain.com,localhost | Comma-separated list of hostnames and IP addresses for the Trust Agent. Used in the Agent TLS certificate. |
| certcn: Trust Agent TLS Certificate           | Common Name for the Trust Agent TLS certificate              |

## Command-Line Options

Usage:

  tagent <command> [arguments]

Available Commands:

  help|-h|-help                    Show this help message.
  setup [all] [task]               Run setup task.
  uninstall                        Uninstall trust agent.
  version                          Print build version info.
  start                            Start the trust agent service.
  stop                             Stop the trust agent service.
  status                           Get the status of the trust agent service.
  fetch-ekcert-with-issuer         Print Tpm Endorsement Certificate in Base64 encoded string along with issuer

Setup command usage:  tagent setup [cmd] [-f <env-file>]

Available Tasks for 'setup', all commands support env file flag

   all                                      - Runs all setup tasks to provision the trust agent. This command can be omitted with running only tagent setup
                                                Required environment variables [in env/trustagent.env]:
                                                  - AAS_API_URL=<url>                                 : AAS API URL
                                                  - CMS_BASE_URL=<url>                                : CMS API URL
                                                  - CMS_TLS_CERT_SHA384=<CMS TLS cert sha384 hash>    : to ensure that TA is communicating with the right CMS instance
                                                  - BEARER_TOKEN=<token>                              : for authenticating with CMS and VS
                                                  - HVS_URL=<url>                            : VS API URL
                                                Optional Environment variables:
                                                  - TA_ENABLE_CONSOLE_LOG=<true/false>                : When 'true', logs are redirected to stdout. Defaults to false.
                                                  - TA_SERVER_IDLE_TIMEOUT=<t seconds>                : Sets the trust agent service's idle timeout. Defaults to 10 seconds.
                                                  - TA_SERVER_MAX_HEADER_BYTES=<n bytes>              : Sets trust agent service's maximum header bytes.  Defaults to 1MB.
                                                  - TA_SERVER_READ_TIMEOUT=<t seconds>                : Sets trust agent service's read timeout.  Defaults to 30 seconds.
                                                  - TA_SERVER_READ_HEADER_TIMEOUT=<t seconds>         : Sets trust agent service's read header timeout.  Defaults to 30 seconds.
                                                  - TA_SERVER_WRITE_TIMEOUT=<t seconds>               : Sets trust agent service's write timeout.  Defaults to 10 seconds.
                                                  - SAN_LIST=<host1,host2.acme.com,...>               : CSV list that sets the value for SAN list in the TA TLS certificate.
                                                                                                        Defaults to "127.0.0.1,localhost".
                                                  - TA_TLS_CERT_CN=<Common Name>                      : Sets the value for Common Name in the TA TLS certificate.  Defaults to "Trust Agent TLS Certificate".
                                                  - TPM_OWNER_SECRET=<40 byte hex>                    : When provided, setup uses the 40 character hex string for the TPM
                                                                                                        owner password. Auto-generated when not provided.
                                                  - TRUSTAGENT_LOG_LEVEL=<trace|debug|info|error>     : Sets the verbosity level of logging. Defaults to 'info'.
                                                  - TRUSTAGENT_PORT=<portnum>                         : The port on which the trust agent service will listen.
                                                                                                        Defaults to 1443

  download-ca-cert                          - Fetches the latest CMS Root CA Certificates, overwriting existing files.
                                                    Required environment variables:
                                                       - CMS_BASE_URL=<url>                                : CMS API URL
                                                       - CMS_TLS_CERT_SHA384=<CMS TLS cert sha384 hash>    : to ensure that TA is communicating with the right CMS instance

  download-cert                             - Fetches a signed TLS Certificate from CMS, overwriting existing files.
                                                    Required environment variables:
                                                       - CMS_BASE_URL=<url>                                : CMS API URL
                                                       - BEARER_TOKEN=<token>                              : for authenticating with CMS and VS
                                                    Optional Environment variables:
                                                       - SAN_LIST=<host1,host2.acme.com,...>               : CSV list that sets the value for SAN list in the TA TLS certificate.
                                                                                                             Defaults to "127.0.0.1,localhost".
                                                       - TA_TLS_CERT_CN=<Common Name>                      : Sets the value for Common Name in the TA TLS certificate.
                                                                                                             Defaults to "Trust Agent TLS Certificate".

  update-certificates                       - Runs 'download-ca-cert' and 'download-cert'
                                                    Required environment variables:
                                                        - CMS_BASE_URL=<url>                                : CMS API URL
                                                        - CMS_TLS_CERT_SHA384=<CMS TLS cert sha384 hash>    : to ensure that TA is communicating with the right CMS instance
                                                        - BEARER_TOKEN=<token>                              : for authenticating with CMS
                                                    Optional Environment variables:
                                                        - SAN_LIST=<host1,host2.acme.com,...>               : CSV list that sets the value for SAN list in the TA TLS certificate.
                                                                                                              Defaults to "127.0.0.1,localhost".
                                                        - TA_TLS_CERT_CN=<Common Name>                      : Sets the value for Common Name in the TA TLS certificate.  Defaults to "Trust Agent TLS Certificate".

  provision-attestation                     - Runs setup tasks associated with HVS/TPM provisioning.
                                                    Required environment variables:
                                                        - HVS_URL=<url>                            : VS API URL
                                                        - BEARER_TOKEN=<token>                              : for authenticating with VS
                                                    Optional environment variables:
                                                        - TPM_OWNER_SECRET=<40 byte hex>                    : When provided, setup uses the 40 character hex string for the TPM
                                                                                                              owner password. Auto-generated when not provided.

  create-host                                 - Registers the trust agent with the verification service.
                                                    Required environment variables:
                                                        - HVS_URL=<url>                            : VS API URL
                                                        - BEARER_TOKEN=<token>                              : for authenticating with VS
                                                        - CURRENT_IP=<ip address of host>                   : IP or hostname of host with which the host will be registered with HVS
                                                    Optional environment variables:
                                                        - TPM_OWNER_SECRET=<40 byte hex>                    : When provided, setup uses the 40 character hex string for the TPM
                                                                                                              owner password. Auto-generated when not provided.

  create-host-unique-flavor                 - Populates the verification service with the host unique flavor
                                                    Required environment variables:
                                                        - HVS_URL=<url>                            : VS API URL
                                                        - BEARER_TOKEN=<token>                              : for authenticating with VS
                                                        - CURRENT_IP=<ip address of host>                   : Used to associate the flavor with the host

  get-configured-manifest                   - Uses environment variables to pull application-integrity
                                              manifests from the verification service.
                                                     Required environment variables:
                                                        - HVS_URL=<url>                            : VS API URL
                                                                                                                - BEARER_TOKEN=<token>                              : for authenticating with VS
                                                                                                                - FLAVOR_UUIDS=<uuid1,uuid2,[...]>                  : CSV list of flavor UUIDs
                                                                                                                                                                        - FLAVOR_LABELS=<flavorlabel1,flavorlabel2,[...]>   : CSV list of flavor labels

## Directory Layout

### Linux

The Linux Trust Agent installs by default to `/opt/trustagent`, with the
following subfolders:

#### Bin

Contains executables and scripts.

#### Configuration

Contains the `config.yml` configuration file, as well as certificates and
keystores. This includes the AIK public key blob after provitioning.

#### Var

Contains information gathered from the platform and SOFTWARE Flavor
manifests. All files with the name `manifest_*.xml` will be parsed to
define measurements during boot. Generally these should be automatically
provisioned from the Verification Service when creating/deploying
SOFTWARE Flavors.