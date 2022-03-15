### Available setup task for Authservice/AAS
```shell script
    all                      Runs all setup tasks
    download-ca-cert         Download CMS root CA certificate
    download-cert-tls        Download CA certificate from CMS for tls
    database                 Setup authservice database
    admin                    Add authservice admin username and password to database and assign respective
                             roles to the user
    jwt                      Create jwt signing key and jwt certificate signed by CMS
    create-credentials       Generates credentials to support third party authentication and authorization
    update-service-config    Sets or Updates the Service configuration
```

```shell script
Following environment variables are required for download-ca-cert
    CMS_BASE_URL                CMS base URL in the format https://{{cms}}:{{cms_port}}/cms/v1/
    CMS_TLS_CERT_SHA384         SHA384 hash value of CMS TLS certificate

Following environment variables are required in download-cert-tls
    CMS_BASE_URL        CMS base URL in the format https://{{cms}}:{{cms_port}}/cms/v1/
    BEARER_TOKEN        Bearer token for accessing CMS api

Following environment variables are optionally used in download-cert-tls
    TLS_CERT_FILE       The file to which certificate is saved
    TLS_KEY_FILE        The file to which private key is saved
    TLS_COMMON_NAME     The common name of signed certificate
    TLS_SAN_LIST        Comma separated list of hostnames to add to Certificate, including IP addresses and DNS names

Following environment variables are required for Database related setups:
    DB_VENDOR                   Vendor of database, or use AAS_DB_VENDOR alternatively
    DB_PORT                     Database port, or use AAS_DB_PORT alternatively
    DB_NAME                     Database name, or use AAS_DB_NAME alternatively
    AAS_DB_PASSWORD             Database password, or use DB_PASSWORD alternatively
    DB_SSL_MODE                 Database SSL mode, or use AAS_DB_SSL_MODE alternatively
    DB_SSL_CERT_SOURCE          Database SSL certificate to be copied from, or use AAS_DB_SSLCERTSRC alternatively
    DB_CONN_RETRY_TIME          Database connection retry time
    DB_HOST                     Database host name, or use AAS_DB_HOSTNAME alternatively
    AAS_DB_USERNAME             Database username, or use DB_USERNAME alternatively
    DB_SSL_CERT                 Database SSL certificate, or use AAS_DB_SSLCERT alternatively
    DB_CONN_RETRY_ATTEMPTS      Database connection retry attempts

Following environment variables are required for admin setup:
    AAS_ADMIN_USERNAME  Authentication and Authorization Service Admin Username 
    AAS_ADMIN_PASSWORD  Authentication and Authorization Service Admin Password

Following environment variables are required in jwt
    CMS_BASE_URL        CMS base URL in the format https://{{cms}}:{{cms_port}}/cms/v1/
    BEARER_TOKEN        Bearer token for accessing CMS api
Following environment variables are optionally used in jwt
    KEY_FILE            The file to which private key is saved
    COMMON_NAME         The common name of signed certificate
    CERT_FILE           The file to which certificate is saved

Following environment variables are optional for create-credentials setup:
    CREATE_CREDENTIALS                  Trigger to run create-credentials setup task when set to True. Default is False
    NATS_OPERATOR_NAME                  Set the NATS operator name, default is "ISecL-operator"
    NATS_OPERATOR_CREDENTIAL_VALIDITY   Set the NATS operator credential validity in terms of duration (ex: "300ms","-1.5h" or "2h45m"), default is 5 years
    NATS_ACCOUNT_NAME                   Set the NATS account name, default is "ISecL-account"
    NATS_ACCOUNT_CREDENTIAL_VALIDITY    Set the NATS account credential validity in terms of duration (ex: "300ms","-1.5h" or "2h45m"), default is 5 years
    NATS_USER_CREDENTIAL_VALIDITY       Set the NATS user credential validity in terms of duration (ex: "300ms","-1.5h" or "2h45m"), default is 1 year

Following environment variables are required for update-service-config setup:
    NATS_USER_CREDENTIAL_VALIDITY               Set the NATS user credential validity, default is 1 year
    JWT_INCLUDE_KID                             Includes JWT Key Id for token validation
    JWT_TOKEN_DURATION_MINS                     Validity of token duration
    AUTH_DEFENDER_MAX_ATTEMPTS                  Auth defender maximum attempts
    SERVER_READ_TIMEOUT                         Request Read Timeout Duration in Seconds
    SERVER_WRITE_TIMEOUT                        Request Write Timeout Duration in Seconds
    NATS_OPERATOR_NAME                          Set the NATS operator name, default is "ISecL-operator"
    LOG_MAX_LENGTH                              Max length of log statement
    AUTH_DEFENDER_LOCKOUT_DURATION_MINS         Auth defender lockout duration in minutes
    NATS_ACCOUNT_CREDENTIAL_VALIDITY            Set the NATS account credential validity, default is 5 years
    LOG_LEVEL                                   Log level
    SERVER_PORT                                 The Port on which Server Listens to
    SERVER_READ_HEADER_TIMEOUT                  Request Read Header Timeout Duration in Seconds
    NATS_ACCOUNT_NAME                           Set the NATS account name, default is "ISecL-account"
    LOG_ENABLE_STDOUT                           Enable console log
    JWT_CERT_COMMON_NAME                        Common Name for JWT Certificate
    AUTH_DEFENDER_INTERVAL_MINS                 Auth defender interval in minutes
    SERVER_IDLE_TIMEOUT                         Request Idle Timeout in Seconds
    SERVER_MAX_HEADER_BYTES                     Max Length Of Request Header in Bytes
    NATS_OPERATOR_CREDENTIAL_VALIDITY           Set the NATS operator credential validity, default is 5 years
```

Variables that needs to be updated or added under aas-credentials secrets:
```shell script
AAS_ADMIN_USERNAME   
AAS_ADMIN_PASSWORD 
```

Variables that needs to be updated or added under aasdb-credentials secrets:
```shell script
AAS_DB_USERNAME   
AAS_DB_PASSWORD 
```

### Available setup task for CMS
```shell script
    all                       Runs all setup tasks
    root-ca                   Creates a self signed Root CA key pair in /etc/cms/root-ca/ for quality of life
    intermediate-ca           Creates a Root CA signed intermediate CA key pair(signing, tls-server and tls-client) in /etc/cms/intermediate-ca/ for quality of life
    tls                       Creates an intermediate-ca signed TLS key pair in /etc/cms for quality of life
    cms-auth-token            Create its own self signed JWT key pair in /etc/cms/jwt for quality of life
    update-service-config     Sets or Updates the Service configuration
```

```shell script
Following environment variables are required for tls setup:
    SAN_LIST    TLS SAN list

Following environment variables are required for authToken setup:
    AAS_JWT_CN          Common Name for JWT Signing Certificate used in Authentication and Authorization Service
    AAS_TLS_CN          Common Name for TLS Signing Certificate used in  Authentication and Authorization Service
    AAS_TLS_SAN         TLS SAN list for Authentication and Authorization Service

Following environment variables are required for update-service-config setup:
    AAS_BASE_URL                AAS Base URL
    TOKEN_DURATION_MINS         Validity of token duration
    SERVER_PORT                 The Port on which Server Listens to
    SERVER_READ_TIMEOUT         Request Read Timeout Duration in Seconds
    SERVER_READ_HEADER_TIMEOUT  Request Read Header Timeout Duration in Seconds
    SERVER_IDLE_TIMEOUT         Request Idle Timeout in Seconds
    LOG_LEVEL                   Log level
    LOG_MAX_LENGTH              Max length of log statement
    SERVER_MAX_HEADER_BYTES     Max Length Of Request Header in Bytes
    LOG_ENABLE_STDOUT           Enable console log
    SERVER_WRITE_TIMEOUT        Request Write Timeout Duration in Seconds

Following environment variables are required for root-ca setup:
    CMS_CA_CERT_VALIDITY        CA Certificate Validity
    CMS_CA_ORGANIZATION         CA Certificate Organization
    CMS_CA_LOCALITY             CA Certificate Locality
    CMS_CA_PROVINCE             CA Certificate Province
    CMS_CA_COUNTRY              CA Certificate Country

Following environment variables are required for intermediate-ca setup:
    CMS_CA_LOCALITY             CA Certificate Locality
    CMS_CA_PROVINCE             CA Certificate Province
    CMS_CA_COUNTRY              CA Certificate Country
    CMS_CA_CERT_VALIDITY        CA Certificate Validity
    CMS_CA_ORGANIZATION         CA Certificate Organization

```

### Available setup task for HVS
```shell script
        all                             Runs all setup tasks
        database                        Setup hvs database
        create-default-flavorgroup      Create default flavor groups in database
        create-default-flavor-template  Create default flavor templates in database
        create-dek                      Create data encryption key for HVS
        download-ca-cert                Download CMS root CA certificate
        download-cert-tls               Download CA certificate from CMS for tls
        download-cert-saml              Download CA certificate from CMS for saml
        download-cert-flavor-signing    Download CA certificate from CMS for flavor signing
        create-endorsement-ca           Generate self-signed endorsement certificate
        create-privacy-ca               Generate self-signed privacy certificate
        create-tag-ca                   Generate self-signed tag certificate
        update-service-config           Sets or Updates the Service configuration

```
```shell script
Following environment variables are required for Database related setups:
    DB_SSL_CERT                 Database SSL certificate, or use HVS_DB_SSLCERT alternatively
    DB_SSL_CERT_SOURCE          Database SSL certificate to be copied from, or use HVS_DB_SSLCERTSRC alternatively
    DB_CONN_RETRY_TIME          Database connection retry time
    DB_HOST                     Database host name, or use HVS_DB_HOSTNAME alternatively
    DB_PORT                     Database port, or use HVS_DB_PORT alternatively
    DB_USERNAME                 Database username, or use HVS_DB_USERNAME alternatively
    DB_SSL_MODE                 Database SSL mode, or use HVS_DB_SSL_MODE alternatively
    DB_VENDOR                   Vendor of database, or use HVS_DB_VENDOR alternatively
    DB_NAME                     Database name, or use HVS_DB_NAME alternatively
    DB_PASSWORD                 Database password, or use HVS_DB_PASSWORD alternatively
    DB_CONN_RETRY_ATTEMPTS      Database connection retry attempts

Following environment variables are required for update-service-config setup:
    HRRS_REFRESH_PERIOD                         Host report refresh service period
    HOST_TRUST_CACHE_THRESHOLD                  Maximum number of entries to be cached in the Trust/Flavor caches
    NAT_SERVERS                                 List of NATs servers to establish connection with outbound TAs
    SERVICE_PASSWORD                            The service password as configured in AAS
    AAS_BASE_URL                                AAS Base URL
    FVS_NUMBER_OF_DATA_FETCHERS                 Number of Flavor verification data fetcher threads
    FVS_SKIP_FLAVOR_SIGNATURE_VERIFICATION      Skips flavor signature verification when set to true
    SERVER_PORT                                 The Port on which Server listens to
    SERVER_READ_HEADER_TIMEOUT                  Request Read Header Timeout Duration in Seconds
    SERVER_WRITE_TIMEOUT                        Request Write Timeout Duration in Seconds
    LOG_ENABLE_STDOUT                           Enable console log
    FVS_NUMBER_OF_VERIFIERS                     Number of Flavor verification verifier threads
    SERVER_READ_TIMEOUT                         Request Read Timeout Duration in Seconds
    SERVER_IDLE_TIMEOUT                         Request Idle Timeout in Seconds
    SERVICE_USERNAME                            The service username as configured in AAS
    LOG_MAX_LENGTH                              Max length of log statement
    SERVER_MAX_HEADER_BYTES                     Max Length of Request Header in Bytes
    ENABLE_EKCERT_REVOKE_CHECK                  If enabled, revocation checks will be performed for EK certs at the time of AIK provisioning
    LOG_LEVEL                                   Log level
    VCSS_REFRESH_PERIOD                         VCenter refresh service period

Following environment variables are required in download-cert-flavor-signing
    CMS_BASE_URL        CMS base URL in the format https://{{cms}}:{{cms_port}}/cms/v1/
    BEARER_TOKEN        Bearer token for accessing CMS api
Following environment variables are optionally used in download-cert-flavor-signing
    FLAVOR_SIGNING_CERT_FILE    The file to which certificate is saved
    FLAVOR_SIGNING_KEY_FILE     The file to which private key is saved
    FLAVOR_SIGNING_COMMON_NAME  The common name of signed certificate

Following environment variables are optionally used in create-privacy-ca
    PRIVACY_CA_KEY_FILE         The file to which private key is saved
    PRIVACY_CA_COMMON_NAME      The common name of signed certificate
    PRIVACY_CA_ISSUER           The issuer of signed certificate
    PRIVACY_CA_VALIDITY_YEARS   The validity time in years of signed certificate
    PRIVACY_CA_CERT_FILE        The file to which certificate is saved

Following environment variables are optionally used in create-endorsement-ca
    ENDORSEMENT_CA_KEY_FILE             The file to which private key is saved
    ENDORSEMENT_CA_COMMON_NAME          The common name of signed certificate
    ENDORSEMENT_CA_ISSUER               The issuer of signed certificate
    ENDORSEMENT_CA_VALIDITY_YEARS       The validity time in years of signed certificate
    ENDORSEMENT_CA_CERT_FILE            The file to which certificate is saved

Following environment variables are optionally used in create-tag-ca
    TAG_CA_CERT_FILE            The file to which certificate is saved
    TAG_CA_KEY_FILE             The file to which private key is saved
    TAG_CA_COMMON_NAME          The common name of signed certificate
    TAG_CA_ISSUER               The issuer of signed certificate
    TAG_CA_VALIDITY_YEARS       The validity time in years of signed certificate

Following environment variables are required for Database related setups:
    DB_VENDOR                   Vendor of database, or use HVS_DB_VENDOR alternatively
    DB_NAME                     Database name, or use HVS_DB_NAME alternatively
    DB_PASSWORD                 Database password, or use HVS_DB_PASSWORD alternatively
    DB_CONN_RETRY_ATTEMPTS      Database connection retry attempts
    DB_CONN_RETRY_TIME          Database connection retry time
    DB_HOST                     Database host name, or use HVS_DB_HOSTNAME alternatively
    DB_PORT                     Database port, or use HVS_DB_PORT alternatively
    DB_USERNAME                 Database username, or use HVS_DB_USERNAME alternatively
    DB_SSL_MODE                 Database SSL mode, or use HVS_DB_SSL_MODE alternatively
    DB_SSL_CERT                 Database SSL certificate, or use HVS_DB_SSLCERT alternatively
    DB_SSL_CERT_SOURCE          Database SSL certificate to be copied from, or use HVS_DB_SSLCERTSRC alternatively

Following environment variables are required for download-ca-cert
    CMS_BASE_URL                CMS base URL in the format https://{{cms}}:{{cms_port}}/cms/v1/
    CMS_TLS_CERT_SHA384         SHA384 hash value of CMS TLS certificate

Following environment variables are required in download-cert-tls
    CMS_BASE_URL        CMS base URL in the format https://{{cms}}:{{cms_port}}/cms/v1/
    BEARER_TOKEN        Bearer token for accessing CMS api
Following environment variables are optionally used in download-cert-tls
    TLS_CERT_FILE       The file to which certificate is saved
    TLS_KEY_FILE        The file to which private key is saved
    TLS_COMMON_NAME     The common name of signed certificate

    TLS_SAN_LIST        Comma separated list of hostnames to add to Certificate, including IP addresses and DNS names

Following environment variables are required in download-cert-saml
    CMS_BASE_URL        CMS base URL in the format https://{{cms}}:{{cms_port}}/cms/v1/
    BEARER_TOKEN        Bearer token for accessing CMS api
Following environment variables are optionally used in download-cert-saml
    SAML_CERT_FILE      The file to which certificate is saved
    SAML_KEY_FILE       The file to which private key is saved
    SAML_COMMON_NAME    The common name of signed certificate

    SAML_ISSUER                 The issuer of signed certificate
    SAML_VALIDITY_SECONDS       The validity time in seconds of signed certificate

Following environment variables are required for Database related setups:
    DB_VENDOR                   Vendor of database, or use HVS_DB_VENDOR alternatively
    DB_NAME                     Database name, or use HVS_DB_NAME alternatively
    DB_PASSWORD                 Database password, or use HVS_DB_PASSWORD alternatively
    DB_CONN_RETRY_ATTEMPTS      Database connection retry attempts
    DB_CONN_RETRY_TIME          Database connection retry time
    DB_HOST                     Database host name, or use HVS_DB_HOSTNAME alternatively
    DB_PORT                     Database port, or use HVS_DB_PORT alternatively
    DB_USERNAME                 Database username, or use HVS_DB_USERNAME alternatively
    DB_SSL_MODE                 Database SSL mode, or use HVS_DB_SSL_MODE alternatively
    DB_SSL_CERT                 Database SSL certificate, or use HVS_DB_SSLCERT alternatively
    DB_SSL_CERT_SOURCE          Database SSL certificate to be copied from, or use HVS_DB_SSLCERTSRC alternatively

```

Variables that needs to be updated or added under hvs-credentials secrets:
```shell script
HVS_ADMIN_USERNAME   
HVS_ADMIN_PASSWORD 
```

Variables that needs to be updated or added under hvsdb-credentials secrets:
```shell script
HVS_DB_USERNAME   
HVS_DB_PASSWORD 
```

### Available setup task for IHub
```shell script
    all                                 Runs all setup tasks
    download-ca-cert                    Download CMS root CA certificate
    download-cert-tls                   Download CA certificate from CMS for tls
    attestation-service-connection      Establish Attestation service connection
    tenant-service-connection           Establish Tenant service connection
    create-signing-key                  Create signing key for IHUB
    download-saml-cert                  Download SAML certificate from Attestation service
    update-service-config               Sets or Updates the Service configuration
```

```shell script
Following environment variables are required for update-service-config setup:
    SERVICE_USERNAME    The service username as configured in AAS
    SERVICE_PASSWORD    The service password as configured in AAS
    LOG_LEVEL           Log level
    LOG_MAX_LENGTH      Max length of log statement
    LOG_ENABLE_STDOUT   Enable console log
    AAS_BASE_URL        AAS Base URL

Following environment variables are required for download-ca-cert
    CMS_BASE_URL                CMS base URL in the format https://{{cms}}:{{cms_port}}/cms/v1/
    CMS_TLS_CERT_SHA384         SHA384 hash value of CMS TLS certificate

Following environment variables are required in download-cert-tls
    CMS_BASE_URL        CMS base URL in the format https://{{cms}}:{{cms_port}}/cms/v1/
    BEARER_TOKEN        Bearer token for accessing CMS api
Following environment variables are optionally used in download-cert-tls
    TLS_CERT_FILE       The file to which certificate is saved
    TLS_KEY_FILE        The file to which private key is saved
    TLS_COMMON_NAME     The common name of signed certificate

    TLS_SAN_LIST        Comma separated list of hostnames to add to Certificate, including IP addresses and DNS names

Following environment variables are required for attestation-service-connection setup:
    HVS_BASE_URL        Base URL for the Host Verification Service
    SHVS_BASE_URL       Base URL for the SGX Host Verification Service

Following environment variables are required for tenant-service-connection setup:
    TENANT      Type of Tenant Service (Kubernetes)
Following environment variables are required for Kubernetes tenant:
    KUBERNETES_URL              URL for Kubernetes deployment
    KUBERNETES_CRD              CRD Name for Kubernetes deployment
    KUBERNETES_TOKEN            Token for Kubernetes deployment
    KUBERNETES_CERT_FILE        Certificate path for Kubernetes deployment

```

### Available setup task for TA
```shell script
all                         Runs all setup tasks to provision the trust agent. This command can be omitted with running only tagent setup
download-ca-cert            Fetches the latest CMS Root CA Certificates, overwriting existing files.
download-cert               Fetches the latest CMS Root CA Certificates, overwriting existing files.
download-credential         Fetches Credential from AAS
download-api-token          Fetches Custom Claims Token from AAS
update-certificates         Runs 'download-ca-cert' and 'download-cert'
provision-attestation       Runs setup tasks associated with HVS/TPM provisioning.
create-host                 Registers the trust agent with the verification service.
create-host-unique-flavor   Populates the verification service with the host unique flavor
get-configured-manifest     Uses environment variables to pull application-integrity
update-service-config       Updates service configuration
define-tag-index            Allocates nvram in the TPM for use by asset tags.
```

```shell script
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
  download-credential                       - Fetches Credential from AAS
                                                    Required environment variables:
                                                       - BEARER_TOKEN=<token>                              : for authenticating with AAS
                                                       - AAS_API_URL=<url>                                 : AAS API URL
                                                       - TA_HOST_ID=<ta-host-id>                           : FQDN of host
  download-api-token                        - Fetches Custom Claims Token from AAS
                                                    Required environment variables:
                                                       - BEARER_TOKEN=<token>                              : for authenticating with AAS
                                                       - AAS_API_URL=<url>                                 : AAS API URL
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
  update-service-config                     - Updates service configuration
                                                     Required environment variables:
                                                        - TRUSTAGENT_PORT=<port>                            : Trust Agent Listener Port
                                                        - TA_SERVER_READ_TIMEOUT                            : Trustagent Server Read Timeout
                                                        - TA_SERVER_READ_HEADER_TIMEOUT                     : Trustagent Read Header Timeout
                                                        - TA_SERVER_WRITE_TIMEOUT                           : Tustagent Write Timeout  
                                                        - TA_SERVER_IDLE_TIMEOUT                            : Trustagent Idle Timeout  
                                                        - TA_SERVER_MAX_HEADER_BYTES                        : Trustagent Max Header Bytes Timeout
                                                        - TRUSTAGENT_LOG_LEVEL                              : Logging Level            
                                                        - TA_ENABLE_CONSOLE_LOG                             : Trustagent Enable standard output
                                                        - LOG_ENTRY_MAXLENGTH                               : Maximum length of each entry in a log
  define-tag-index                          - Allocates nvram in the TPM for use by asset tags.

```