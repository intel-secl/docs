# Intel Security Libraries Configuration Settings

???+ note 
    All the answer file options would remain common for containerized K8s deployments with the except of URLS where Kubernetes DNS would be used. The respective `configMap.yml` for each service and agent would carry the defaults for the same when built under `<working directory>/k8s/manifests/<service/agent/db names>`

##  SGX Host Verification Service

### Installation Answer File Options

| Key                            | Sample Value                                            | Description                                                  |
| ------------------------------ | ------------------------------------------------------- | ------------------------------------------------------------ |
| CMS_BASE_URL                   | https://< IP address or hostname for CMS >:8445/cms/v1/ | Base URL of the CMS                                          |
| AAS_API_URL                    | https://< IP address or hostname for AAS >:8444/aas/v1  | Base URL of the AAS                                          |
| SCS_BASE_URL                   | https://< IP or hostname of SCS >:9000/scs/sgx/         | Base URL of SCS                                              |
| SHVS_DB_PORT                   | 5432                                                    | Defines the port number for communication with the database server. By default, with a local database server installation, this port will be set to 5432. |
| SHVS_DB_NAME                   | pgshvsdb                                                | Defines the schema name of the database. If a remote database connection will be used, this schema must be created in the remote database before installing the SGX Host Verification Service |
| SHVS_DB_USERNAME               | aasdbuser                                               | Username for accessing the database. If a remote database connection will be used, this user/password must be created and granted all permissions for the database schema before installing the SGX Host Verification Service. |
| SHVS_DB_PASSWORD               | aasdbpassword                                           | Password for accessing the database. If a remote database connection will be used, this user/password must be created and granted all permissions for the database schema before installing the SGX Host Verification Service. |
| SHVS_DB_HOSTNAME               | localhost                                               | Defines the database server IP address or hostname. This should be the loopback address for local database server installations but should be the IP address or hostname of the database server if a remote database will be used. |
| SAN_LIST                       | 127.0.0.1,localhost                                     | Comma-separated list of IP addresses and hostnames that will be valid connection points for the service. Requests sent to the service using an IP or hostname not in this list will be denied, even if it resolves to this service |
| SHVS_ADMIN_USERNAME            | shvsuser@shvs                                           | Username for a new user to be created during installation.   |
| SHVS_ADMIN_PASSWORD            | shvspassword                                            | Password for the user to be created during installation.     |
| CMS_TLS_CERT_SHA384            | < Certificate Management Service TLS digest>            | SHA384 hash of the CMS TLS certificate                       |
| BEARER_TOKEN                   |                                                         | Installation token from AAS                                  |
| SHVS_PORT                      | 13000                                                   | SGX Host Verification Service HTTP Port                      |
| SHVS_SCHEDULER_TIMER           | 10                                                      | SHVS Scheduler timeout                                       |
| SHVS_HOST_PLATFORM_EXPIRY_TIME | 240                                                     | SHVS Host Info Expiry time                                   |
| SHVS_AUTO_REFRESH_TIMER        | 120                                                     | SHVS Auto-refresh timeout                                    |


## Configuration Options

The SGX Host Verification Service configuration is in path /etc/shvs/config.yml.

## Command-Line Options

The SGX Host Verification Service supports several command-line options that can be executed only as the Root user:

Syntax:
```
shvs \<command\>
```

### Available Commands

#### Help
```
shvs help
```
Displays the list of available CLI commands.

#### Start
```
shvs start
```
Starts the SGX Host Verification service

#### Stop
```
shvs stop
```
Stops the SGX Host Verification service

#### Status
```
shvs status
```
Reports whether the service is currently running.

#### Uninstall
```
shvs uninstall \[\--purge\]
```
Removes the service. Use \--purge option to remove configuration directory(/etc/shvs/)

#### Version
```
shvs version
```

Shows the version of the service.

#### Setup [task]

Runs a specific setup task.

Syntax:

shvs setup [task]

### Setup tasks and its Configuration Options for SGX Host Verification Service

```shell
Available Tasks for setup:
    all                       Runs all setup tasks
                              Required env variables:
                                  - get required env variables from all the setup tasks
                              Optional env variables:
                                  - get optional env variables from all the setup tasks

    shvs setup database
        - Available arguments are:
            SHVS_DB_HOSTNAME
            SHVS_DB_PORT
            SHVS_DB_USERNAME
            SHVS_DB_PASSWORD
            SHVS_DB_NAME
            SHVS_DB_SSLMODE <disable|allow|prefer|require|verify-ca|verify-full>
            SHVS_DB_SSLCERT path to where the certificate file of database. Only applicable
                         for db-sslmode=<verify-ca|verify-full. If left empty, the cert
                         will be copied to /etc/shvs/shvs-dbcert.pem
                         alternatively, set environment variable
            - SHVS_DB_SSLCERTSRC <path to where the database ssl/tls certificate file>
                         mandatory if db-sslcert does not already exist
                         alternatively, set environment variable

    update_service_config    Updates Service Configuration
                             Required env variables:
                                 - SHVS_PORT                                         : SGX Host Verification Service port
                                 - SHVS_SERVER_READ_TIMEOUT                          : SGX Host Verification Service Read Timeout
                                 - SHVS_SERVER_READ_HEADER_TIMEOUT                   : SGX Host Verification Service Read Header Timeout Duration
                                 - SHVS_SERVER_WRITE_TIMEOUT                         : SGX Host Verification Service Request Write Timeout Duration
                                 - SHVS_SERVER_IDLE_TIMEOUT                          : SGX Host Verification Service Request Idle Timeout
                                 - SHVS_SERVER_MAX_HEADER_BYTES                      : SGX Host Verification Service Max Length Of Request Header Bytes
                                 - SHVS_LOG_LEVEL                                    : SGX Host Verification Service Log Level
                                 - SHVS_LOG_MAX_LENGTH                               : SGX Host Verification Service Log maximum length
                                 - SHVS_ENABLE_CONSOLE_LOG                           : SGX Host Verification Service Enable standard output
                                 - SHVS_ADMIN_USERNAME                               : SHVS Service Username
                                 - SHVS_ADMIN_PASSWORD                               : SHVS Service Password
                                 - SHVS_SCHEDULER_TIMER                              : SHVS Scheduler Timeout Seconds
                                 - SHVS_AUTO_REFRESH_TIMER                           : SHVS autoRefresh Timeout Seconds
                                 - SHVS_HOST_PLATFORM_EXPIRY_TIME                    : SHVS Host Platform Expiry Time in seconds
                                 - SCS_BASE_URL                                      : SGX Caching Service URL
                                 - AAS_API_URL                                       : AAS API URL

    download_ca_cert         Download CMS root CA certificate
                             Required env variables specific to setup task are:
                                 - CMS_BASE_URL=<url>                                : for CMS API url
                                 - CMS_TLS_CERT_SHA384=<CMS TLS cert sha384 hash>    : to ensure that SHVS is talking to the right CMS instance

    download_cert TLS        Generates Key pair and CSR, gets it signed from CMS
                             Required env variable if SHVS_NOSETUP=true or variable not set in config.yml:
                                 - CMS_TLS_CERT_SHA384=<CMS TLS cert sha384 hash>      : to ensure that SHVS is talking to the right CMS instance
                             Required env variables specific to setup task are:
                                 - CMS_BASE_URL=<url>               : for CMS API url
                                 - BEARER_TOKEN=<token>             : for authenticating with CMS
                                 - SAN_LIST=<san>                   : list of hosts which needs access to service
                             Optional env variables specific to setup task are:
                                 - KEY_PATH=<key_path>              : Path of file where TLS key needs to be stored
                                 - CERT_PATH=<cert_path>            : Path of file/directory where TLS certificate needs to be stored
```

## Directory Layout

The SGX Host Verification Service installs by default to /opt/shvs with the following folders.

### Bin

This folder contains executable scripts.

### Configuration

This folder /etc/shvs contains certificates, keys, and configuration files.

### Logs

This folder contains log files: /var/log/shvs/

## SGX Agent

### Installation Answer File Options

| Key                 | Sample Value                                     | Description                                                  |
| ------------------- | ------------------------------------------------ | ------------------------------------------------------------ |
| SCS_BASE_URL        | https://< AAS IP or Hostname>:9000/scs/sgx/      | The url used during setup to request information from SCS.   |
| CMS_BASE_URL        | https://< CMS IP or hostname>:8445/cms/v1/       | API URL for Certificate Management Service (CMS).            |
| SHVS_BASE_URL       | https://< SHVS IP or hostname>:13000/sgx-hvs/v2/ | The url used during setup to request information from SHVS.  |
| BEARER_TOKEN        |                                                  | Long Lived JWT from AAS that contains "install" permissions needed to access ISecL services during provisioning and registration |
| CMS_TLS_CERT_SHA384 | < Certificate Management Service TLS digest>     | SHA384 Hash for verifying the CMS TLS certificate.           |
| SHVS_UPDATE_INTERVAL| 120                                              | Interval for SHVS updates in minutes. Values should be in the range of 1 minutes to 120 minutues.|
| SGX_AGENT_NOSETUP   | false                                            | Skips setup during installation if set to true               |


### Configuration Options

The SGX Agent configuration is in path /etc/sgx_agent/config.yml.

### Command-Line Options

The SGX Agent supports several command-line options that can be executed only as the Root user:

Syntax:

sgx_agent \<command\>

#### Available Commands

#### Help

Show the help message.

#### Start

sgx_agent start

Start the SGX Agent service.

#### Stop

sgx_agent stop

Stop the SGX Agent service.

#### Status

sgx_agent status

Get the status of the SGX Agent Service.

#### Uninstall

sgx_agent uninstall \--purge

Removes the service. Use \--purge option to remove configuration directory(/etc/sgx_agent/)

#### Version

sgx_agent version

Reports the version of the service.

#### Setup [task]

Runs a specific setup task.

Syntax:

sgx_agent setup [task]

### Setup Tasks and its Configuration Options for SGX Agent

```shell
Available Tasks for setup:
    all                       Runs all setup tasks
                              Required env variables:
                                  - get required env variables from all the setup tasks
                              Optional env variables:
                                  - get optional env variables from all the setup tasks

    update_service_config    Updates Service Configuration
                             Required env variables:
                                 - SCS_BASE_URL                                     : SCS Base URL
                                 - SGX_AGENT_LOGLEVEL                               : SGX_AGENT Log Level
                                 - SGX_AGENT_LOG_MAX_LENGTH                         : SGX Agent Log maximum length
                                 - SGX_AGENT_ENABLE_CONSOLE_LOG                     : SGX Agent Enable standard output
                                 - SHVS_UPDATE_INTERVAL                             : SHVS update interval in minutes
                                 - WAIT_TIME                                        : Time between each retries to PCS
                                 - RETRY_COUNT                                      : Push Data Retry Count to SCS
                                 - SHVS_BASE_URL                                    : HVS Base URL
                                 - BEARER_TOKEN                                     : BEARER TOKEN

    download_ca_cert         Download CMS root CA certificate
                             Required env variables specific to setup task are:
                                 - CMS_BASE_URL=<url>                                : for CMS API url
                                 - CMS_TLS_CERT_SHA384=<CMS TLS cert sha384 hash>    : to ensure that SGX-Agent is talking to the right CMS instance

```

### Directory Layout

#### Linux

The Linux SGX Agent installs by default to /opt/sgx_agent, with the following subfolders:

#### Bin

Contains executables and scripts.

#### Configuration

Contains the config.yml configuration file.

#### Logs

This folder contains log files: /var/log/sgx_agent


## Integration Hub

### Installation Answer File Options

| Key                     | sample Value                                                 | Description                                                  |
| ----------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| AAS_API_URL             | https://< Authentication and Authorization Service IP or  Hostname>:8444/aas/v1 | Base URL for the AAS                      |
| CMS_BASE_URL            | https://< Certificate Management Service IP or Hostname>:8445/cms/v1 | Base URL for the CMS                                 |
| SHVS_BASE_URL           | https://< SGX Host Verification Service IP or hostname>:13000/sgx-hvs/v2/ | Base URL  of SHVS                               |
| IHUB_SERVICE_USERNAME   | ihubuser@ihub                                                | Database username                                            |
| IHUB_SERVICE_PASSWORD   | ihubpassword                                                 | Database password                                            |
| CMS_TLS_CERT_SHA384     | < Certificate Management Service TLS digest>                 | SHA384 digest of the CMS TLS certificate                     |
| BEARER_TOKEN            |                                                              | Installation token                                           |
| TENANT                  | KUBERNETES                                                   | Tenant Orchaestrator                                         |
| KUBERNETES_URL          | https://< Kubernetes Master Node IP or  Hostname> :6443      | Kubernetes Master node URL                                   |
| KUBERNETES_CRD          | custom-isecl-sgx                                             | CRD Name to be used                                          |
| TLS_SAN_LIST            | 127.0.0.1, localhost                                         | Comma-separated list of IP addresses and hostnames that will be valid connection points for the service. Requests sent to the service using an IP or hostname not in this list will be denied, even if it resolves to this service. |
| KUBERNETES_TOKEN        |                                                              | Token from Kubernetes Master Node                            |
| KUBERNETES_CERT_FILE    | /root/apiserver.crt                                          | Kubernetes server certificate path                           |
| POLL_INTERVAL_MINUTES   | 2                                                            | IHUB Polling Interval in Minutes                             |
| INSTANCE_NAME           | ihub                                                         | IHUB default instance name                                   |

### Configuration Options

The Integration Hub configuration can be found in /etc/ihub/config.yml.

### Command-Line Options

The Integrtion HUB supports several command-line options that can be executed only as the Root user:

Syntax:

ihub <command>

#### Available Commands

#### Help

ihub help

Displays the list of available CLI commands

#### Start

ihub start

Start the service

#### Stop

ihub stop

stops the service

#### Status

ihub status

Reports whether the service is currently running.

#### Uninstall

ihub uninstall [--purge] [--exec]

Removes the service. Use --purge option to remove configuration directory(/etc/ihub/). Use \--exec option to remove ihub instance specific directories

#### Version

ihub version

Reports the version of the service.

#### Setup [task]

Runs a specific setup task.

Syntax:

ihub setup [task]

### Setup Tasks and its Configuration Options for Integration Hub

```shell
Available Tasks for setup:
        all                                 Runs all setup tasks
        download-ca-cert                    Download CMS root CA certificate
        download-cert-tls                   Download CA certificate from CMS for tls
        attestation-service-connection      Establish Attestation service connection
        tenant-service-connection           Establish Tenant service connection
        create-signing-key                  Create signing key for IHUB
        update-service-config               Sets or Updates the Service configuration

Following environment variables are required for "download-ca-cert"
    CMS_BASE_URL                CMS base URL in the format https://{{cms}}:{{cms_port}}/cms/v1/
    CMS_TLS_CERT_SHA384         SHA384 hash value of CMS TLS certificate

Following environment variables are required in "download-cert-tls"
    CMS_BASE_URL        CMS base URL in the format https://{{cms}}:{{cms_port}}/cms/v1/
    BEARER_TOKEN        Bearer token for accessing CMS api
Following environment variables are optionally used in download-cert-tls
    TLS_CERT_FILE       The file to which certificate is saved
    TLS_KEY_FILE        The file to which private key is saved
    TLS_COMMON_NAME     The common name of signed certificate

    TLS_SAN_LIST        Comma separated list of hostnames to add to Certificate, including IP addresses and DNS names

Following environment variables are required for 'attestation-service-connection' setup:
    SHVS_BASE_URL       Base URL for the SGX Host Verification Service

Following environment variables are required for 'tenant-service-connection' setup:
    TENANT      Type of Tenant Service (OpenStack or Kubernetes)
Following environment variables are required for Kubernetes tenant:
    KUBERNETES_TOKEN            Token for Kubernetes deployment
    KUBERNETES_CERT_FILE        Certificate path for Kubernetes deployment
    KUBERNETES_URL              URL for Kubernetes deployment
    KUBERNETES_CRD              CRD Name for Kubernetes deployment
Following environment variables are required for OpenStack tenant:
    OPENSTACK_PLACEMENT_URL     Placement API endpoint for OpenStack deployment
    OPENSTACK_USERNAME          UserName for OpenStack deployment
    OPENSTACK_PASSWORD          Password for OpenStack deployment
    OPENSTACK_AUTH_URL          Keystone API endpoint for OpenStack deployment

Following environment variables are required for update-service-config setup:
    LOG_LEVEL           Log level
    LOG_MAX_LENGTH      Max length of log statement
    LOG_ENABLE_STDOUT   Enable console log
    AAS_BASE_URL        AAS Base URL
    SERVICE_USERNAME    The service username as configured in AAS
    SERVICE_PASSWORD    The service password as configured in AAS
```

### Directory Layout

The Integration HUB installs by default to /opt/ihub with the following folders.

#### Bin

This folder contains executable scripts.

#### Configuration

This folder /etc/ihub/ contains certificates, keys, and configuration files.

#### Logs

This folder contains log files: /var/log/ihub/


## Certificate Management Service

### Installation Answer File Options

| Key         | Sample Value                                         | Description                                                  |
| ----------- | ---------------------------------------------------- | ------------------------------------------------------------ |
| CMS_PORT    | 8445                                                 | Default Port where Certificate Management Service Runs       |
| CMS_NOSETUP | false                                                | Determines whether “setup” will be executed after installation. Typically this is set to “false” to install and perform setup in one action. The “true” option is intended for building the service as a container, where the installation would be part of the image build, and setup would be performed when the container starts for the first time to generate any persistent data. |
| AAS_API_URL | https://< AAS Hostname or IP address>:8444/aas/v1    | URL to connect to the AAS, used during setup for authentication. |
| AAS_TLS_SAN | < Comma-separated list of IPs/hostnames for the AAS> | SAN list populated in special JWT token; this token is used by AAS to get TLS certificate signed from CMS. SAN list in this token and CSR generated by AAS must match. |


### Configuration Options

The CMS configuration can be found in /etc/cms/config.yml.

### Command-Line Options

The Certificate Management Service supports several command-line options that can be executed only as the Root user:

Syntax:

cms <command>

#### Available Commands

#### Help

cms help

Displays the list of available CLI commands.

#### Start

cms start

Starts the services.

#### Stop

cms stop

Stops the service.

#### Status

cms status

Reports whether the service is currently running.

#### Uninstall

cms uninstall [--purge]

Uninstalls the service, including the deletion of all files and folders.

#### Version

cms version

Reports the version of the service.

#### Tlscertsha384

cms tlscertsha384

Shows the SHA384 digest of the TLS certificate.

#### Setup [task]

Runs a specific setup task.

Syntax:

cms setup [task]

Available Tasks for setup:

##### cms setup server [--port=<port>]

-   Setup http server on <port>

-   Environment variable CMS_PORT=<port> can be set alternatively

#####  cms setup root_ca [--force]

-   Create its own self signed Root CA keypair in /etc/cms for quality of life

-   Option [--force] overwrites any existing files, and always generate new Root CA keypair

##### cms setup tls [--force] [--host_names=<host_names>]

-   Create its own root_ca signed TLS keypair in /etc/cms for quality of life

-   Option [--force] overwrites any existing files, and always generate root_ca signed TLS keypair

-   Argument <host_names> is a list of host names used by local machine, seperated by comma

-   Environment variable CMS_HOST_NAMES=<host_names> can be set alternatively

##### cms setup cms-auth-token [--force]

-   Create its own self signed JWT keypair in /etc/cms/jwt for quality of life

-   Option [--force] overwrites any existing files, and always generate new

JWT keypair and token

### Setup Tasks and its Configuration Options for Certificate Management Service

```shell
Available Tasks for setup:
    all                       Runs all setup tasks
    root-ca                   Creates a self signed Root CA key pair in /etc/cms/root-ca/ for quality of life
    intermediate-ca           Creates a Root CA signed intermediate CA key pair(signing, tls-server and tls-client) in /etc/cms/intermediate-ca/ for quality of life
    tls                       Creates an intermediate-ca signed TLS key pair in /etc/cms for quality of life
    cms-auth-token            Create its own self signed JWT key pair in /etc/cms/jwt for quality of life
    update-service-config     Sets or Updates the Service configuration

Following environment variables are required for 'tls' setup:
    SAN_LIST    TLS SAN list

Following environment variables are required for 'authToken' setup:
    AAS_JWT_CN          Common Name for JWT Signing Certificate used in Authentication and Authorization Service
    AAS_TLS_CN          Common Name for TLS Signing Certificate used in  Authentication and Authorization Service
    AAS_TLS_SAN         TLS SAN list for Authentication and Authorization Service

Following environment variables are required for 'update-service-config' setup:
    AAS_BASE_URL                AAS Base URL
    SERVER_PORT                 The Port on which Server Listens to
    SERVER_READ_TIMEOUT         Request Read Timeout Duration in Seconds
    SERVER_READ_HEADER_TIMEOUT  Request Read Header Timeout Duration in Seconds
    SERVER_IDLE_TIMEOUT         Request Idle Timeout in Seconds
    LOG_LEVEL                   Log level
    LOG_MAX_LENGTH              Max length of log statement
    SERVER_WRITE_TIMEOUT        Request Write Timeout Duration in Seconds
    SERVER_MAX_HEADER_BYTES     Max Length Of Request Header in Bytes
    LOG_ENABLE_STDOUT           Enable console log
    TOKEN_DURATION_MINS         Validity of token duration

Following environment variables are required for 'root-ca' setup:
    CMS_CA_PROVINCE             CA Certificate Province
    CMS_CA_COUNTRY              CA Certificate Country
    CMS_CA_CERT_VALIDITY        CA Certificate Validity
    CMS_CA_ORGANIZATION         CA Certificate Organization
    CMS_CA_LOCALITY             CA Certificate Locality

Following environment variables are required for 'intermediate-ca' setup:
    CMS_CA_PROVINCE             CA Certificate Province
    CMS_CA_COUNTRY              CA Certificate Country
    CMS_CA_CERT_VALIDITY        CA Certificate Validity
    CMS_CA_ORGANIZATION         CA Certificate Organization
    CMS_CA_LOCALITY             CA Certificate Locality
```

### Directory Layout

The Certificate Management Service installs by default to /opt/cms with the following folders.

#### Bin

This folder contains executable scripts.

#### Configuration

This folder /etc/cms contains certificates, keys, and configuration files.

#### Logs

This folder contains log files: /var/log/cms/

#### Cacerts

This folder contains the CMS root CA certificate.

## Authentication and Authorization Service

### Installation Answer File Options

| Key                    | Sample Value                          | Description                                                  |
| ---------------------- | ------------------------------------- | ------------------------------------------------------------ |
| CMS_BASE_URL           | https://< cms IP or hostname>/cms/v1/ | Provides the URL for the CMS.                                |
| AAS_NOSETUP            | false                                 | Determines whether “setup” will be executed after installation. Typically this is set to “false” to install and perform setup in one action. The “true” option is intended for building the service as a container, where the installation would be part of the image build, and setup would be performed when the container starts for the first time to generate any persistent data. |
| AAS_DB_HOSTNAME        | localhost                             | Hostname or IP address of the AAS database                   |
| AAS_DB_PORT            | 5432                                  | Database port number                                         |
| AAS_DB_NAME            | pgdb                                  | Database name                                                |
| AAS_DB_USERNAME        | aasdbuser                             | Database username                                            |
| AAS_DB_PASSWORD        | aasdbpassd                            | Database password                                            |
| AAS_DB_SSLMODE         | verify-full                           |                                                              |
| AAS_DB_SSLCERTSRC      | /usr/local/pgsql/data/server.crt      | Required if the “AAS_DB_SSLMODE” is set to “verify-ca.” Defines the location of the database SSL certificate. |
| AAS_DB_SSLCERT         | < path_to_cert_file_on_system >       | The AAS_DB_SSLCERTSRC variable defines the  source location of the database SSL certificate; this variable determines the  local location. If the former option  is used without specifying this option, the service will copy the SSL  certificate to the default configuration directory. |
| AAS_ADMIN_USERNAME     | admin@aas                             | Defines a new AAS administrative user. This user will be able to create new users, new roles, and new role-user mappings. This user will have the AAS:Administrator role. |
| AAS_ADMIN_PASSWORD     | aasAdminPass                          | Password for the new AAS admin user                          |
| AAS_JWT_CERT_SUBJECT   | "AAS JWT Signing Certificate"         | Defines the subject of the JWT signing certificate.          |
| AAS_JWT_TOKEN_DURATION | 5                                     | Defines the amount of time in minutes that an issued token will be valid. |
| SAN_LIST               | 127.0.0.1,localhost                   | Comma-separated list of IP addresses and hostnames that will be valid connection points for the service. Requests sent to the service using an IP or hostname not in this list will be denied, even if it resolves to this service. |
| BEARER_TOKEN           |                                       | Installation Token from AAS.                                 |



### Configuration Options

The AAS configuration can be found in /etc/authservice/config.yml.

### Command-Line Options

The AAS supports several command-line options that can be executed only as the Root user:

Syntax:

authservice \<command\>

#### Available Commands

#### Help

authservice help

Displays the list of available CLI commands.

#### Start

authservice start

Starts the service.

#### Stop

authservice stop

Stops the service.

#### Status

authservice status

Displays the current status of the service.

#### Uninstall

authservice uninstall [--purge]

Removes the service. Use the "--purge" flag to also delete all data.

#### Version

authservice version

Shows the version of the service.

#### Setup [task]

Executes a specific setup task. Can be used to change the current configuration.

Syntax:

authservice setup [task]

Available Tasks for setup:

##### authservice setup all

 Runs all setup tasks

##### authservice setup database [-force] [-arguments=<argument_value>]

-   Available arguments are:

-   db-host alternatively, set environment variable AAS_DB_HOSTNAME

-   db-port alternatively, set environment variable AAS_DB_PORT

-   db-user alternatively, set environment variable AAS_DB_USERNAME

-   db-pass alternatively, set environment variable AAS_DB_PASSWORD

-   db-name alternatively, set environment variable AAS_DB_NAME

-   db-sslmode <disable|allow|prefer|require|verify-ca|verify-full> alternatively, set environment variable AAS_DB_SSLMODE

-   db-sslcert path to where the certificate file of database. Only applicable for db-sslmode=\<verify-ca\|verify-full. If left empty, the cert will be copied to /etc/authservice/tdcertdb.pem alternatively, set environment variable AAS_DB_SSLCERT

-   db-sslcertsrc <path to where the database ssl/tls certificate file>

mandatory if db-sslcert does not already exist alternatively, set environment variable AAS_DB_SSLCERTSRC

-   Run this command with environment variable AAS_DB_REPORT_MAX_ROWS and AAS_DB_REPORT_NUM_ROTATIONS can update db rotation arguments

##### authservice setup server [--port=<port>]

-   Setup http server on <port>

-   Environment variable AAS_PORT=<port> can be set alternatively authservice setup tls [--force] [--host_names=<host_names>]

-   Use the key and certificate provided in /etc/threat-detection if files exist

-   Otherwise create its own self-signed TLS keypair in /etc/authservice for quality of life

-   Option [--force] overwrites any existing files, and always generate self-signed keypair

-   Argument <host_names> is a list of host names used by local machine, seperated by comma

-   Environment variable AAS_TLS_HOST_NAMES=<host_names> can be set alternatively

##### authservice setup admin [--user=<username>] [-pass=<password>]

-   Environment variable AAS_ADMIN_USERNAME=<username> can be set alternatively

-   Environment variable AAS_ADMIN_PASSWORD=<password> can be set alternatively

##### authservice setup jwt

-   Create jwt signing key and jwt certificate signed by CMS

-   Environment variable CMS_BASE_URL=<url> for CMS API url

-   Environment variable AAS_JWT_CERT_CN=<CERTIFICATE SUBJECT> AAS JWT

Certificate Subject

-   Environment variable AAS_JWT_INCLUDE_KEYID=<KEY ID> AAS include key id in JWT Token

-   Environment variable AAS_JWT_TOKEN_DURATION_MINS=<DURATION> JWT Token validation minutes

-   Environment variable BEARER_TOKEN=<token> for authenticating with CMS


### Setup Tasks and its Configuration Options for Authentication and Authorization Service

```shell
 Available Tasks for setup:
                all                      Runs all setup tasks
                download-ca-cert         Download CMS root CA certificate
                download-cert-tls        Download CA certificate from CMS for tls
                database                 Setup authservice database
                admin                    Add authservice admin username and password to database and assign respective
                                         roles to the user
                jwt                      Create jwt signing key and jwt certificate signed by CMS
                update-service-config    Sets or Updates the Service configuration


Following environment variables are required for 'download-ca-cert'
    CMS_BASE_URL                CMS base URL in the format https://{{cms}}:{{cms_port}}/cms/v1/
    CMS_TLS_CERT_SHA384         SHA384 hash value of CMS TLS certificate

Following environment variables are required in 'download-cert-tls'
    CMS_BASE_URL        CMS base URL in the format https://{{cms}}:{{cms_port}}/cms/v1/
    BEARER_TOKEN        Bearer token for accessing CMS api
Following environment variables are optionally used in download-cert-tls
    TLS_CERT_FILE       The file to which certificate is saved
    TLS_KEY_FILE        The file to which private key is saved
    TLS_COMMON_NAME     The common name of signed certificate

    TLS_SAN_LIST        Comma separated list of hostnames to add to Certificate, including IP addresses and DNS names

Following environment variables are required for 'Database' related setups:
    DB_VENDOR                   Vendor of database, or use AAS_DB_VENDOR alternatively
    DB_HOST                     Database host name, or use AAS_DB_HOSTNAME alternatively
    DB_USERNAME                 Database username, or use AAS_DB_USERNAME alternatively
    DB_PASSWORD                 Database password, or use AAS_DB_PASSWORD alternatively
    DB_SSL_MODE                 Database SSL mode, or use AAS_DB_SSL_MODE alternatively
    DB_SSL_CERT                 Database SSL certificate, or use AAS_DB_SSLCERT alternatively
    DB_PORT                     Database port, or use AAS_DB_PORT alternatively
    DB_NAME                     Database name, or use AAS_DB_NAME alternatively
    DB_SSL_CERT_SOURCE          Database SSL certificate to be copied from, or use AAS_DB_SSLCERTSRC alternatively
    DB_CONN_RETRY_ATTEMPTS      Database connection retry attempts
    DB_CONN_RETRY_TIME          Database connection retry time

Following environment variables are required for 'admin' setup:
    AAS_ADMIN_USERNAME  Authentication and Authorization Service Admin Username
    AAS_ADMIN_PASSWORD  Authentication and Authorization Service Admin Password

Following environment variables are required in 'jwt'
    CMS_BASE_URL        CMS base URL in the format https://{{cms}}:{{cms_port}}/cms/v1/
    BEARER_TOKEN        Bearer token for accessing CMS api
Following environment variables are optionally used in jwt
    CERT_FILE           The file to which certificate is saved
    KEY_FILE            The file to which private key is saved
    COMMON_NAME         The common name of signed certificate

Following environment variables are required for 'update-service-config' setup:
    AUTH_DEFENDER_LOCKOUT_DURATION_MINS         Auth defender lockout duration in minutes
    SERVER_MAX_HEADER_BYTES                     Max Length Of Request Header in Bytes
    JWT_INCLUDE_KID                             Includes JWT Key Id for token validation
    SERVER_READ_HEADER_TIMEOUT                  Request Read Header Timeout Duration in Seconds
    AUTH_DEFENDER_MAX_ATTEMPTS                  Auth defender maximum attempts
    SERVER_PORT                                 The Port on which Server Listens to
    SERVER_READ_TIMEOUT                         Request Read Timeout Duration in Seconds
    LOG_MAX_LENGTH                              Max length of log statement
    LOG_ENABLE_STDOUT                           Enable console log
    JWT_CERT_COMMON_NAME                        Common Name for JWT Certificate
    SERVER_WRITE_TIMEOUT                        Request Write Timeout Duration in Seconds
    SERVER_IDLE_TIMEOUT                         Request Idle Timeout in Seconds
    LOG_LEVEL                                   Log level
    JWT_TOKEN_DURATION_MINS                     Validity of token duration
    AUTH_DEFENDER_INTERVAL_MINS                 Auth defender interval in minutes

```

### Directory Layout

The Authentication and Authorization Service installs by default to /opt/authservice with the following folders.

#### Bin

Contains executable scripts and binaries.

#### Configuration

This folder /etc/authservice contains certificates, keys, and configuration files.

#### Logs

This folder contains log files: /var/log/authservice

#### Dbscripts

This folder /opt/authservice/dbscripts Contains database scripts


## Key Broker Service

### Installation Answer File Options

| Variable Name        | Default Value                                 | Notes                                              |
| -------------------- | --------------------------------------------- | -------------------------------------------------- |
| CMS_BASE_URL         | https://< CMS IP or hostname >:8445/cms/v1/   | Required for generating TLS certificate            |
| AAS_API_URL          | https://< AAS IP or hostname >:8444/aas/v1    | AAS service url                                    |
| SQVS_URL             | https://< SQVS IP or hostname >:12000/svs/v1/ | Required to get the SGX Quote verified             |
| CMS_TLS_CERT_SHA384  | < Certificate Management Service TLS digest > | SHA384 digest of CMS TLS certificate               |
| BEARER_TOKEN         |                                               | JWT token for installation user                    |
| KBS_SERVICE_USERNAME | admin@kms                                     | KBS Service Username                               |
| KBS_SERVICE_PASSWORD | kmsAdminPass                                  | KBS Service User Password                          |
| ENDPOINT_URL         | https://< KBS Hostname >:9443/kbs/v1              | KBS Endpoint URL                                   |
| TLS_COMMON_NAME      | KBS TLS Certificate                           | KBS TLS Certificate common-name                    |
| SERVER_PORT          | 9443                                          | KBS Secure Port                                    |
| SKC_CHALLENGE_TYPE   | SGX                                           | Challenge Type                                     |
| TLS_SAN_LIST         | < KBS IP/Hostname >                           | IP addresses/hostnames to be included in SAN list. |
| KEY_MANAGER          | KMIP                                          | Key Manager Backend to store keys                  |

### Configuration Options

The Key Broker Service configuration is in path /etc/kbs/config.yml.

### Command-Line Options

The Key Broker Service supports several command-line options that can be executed only as the Root user:

Syntax:

kbs \<command\>

#### Available Commands

#### Help

kbs help

Displays the list of available CLI commands.

#### Start

kbs start

Starts the service

#### Stop

kbs stop

Stops the service

#### Status

kbs status

Displays the current status of the service.

#### Uninstall

kbs uninstall [--purge]

Removes the service

#### Version

kbs version

Displays the version of the service

#### Setup [task]

Runs a specific setup task.

Syntax:

kbs setup [task]

### Setup Tasks and its Configuration Options for Key Broker Service

```shell
Available Tasks for setup:
        all                                 Runs all setup tasks
        download-ca-cert                    Download CMS root CA certificate
        download-cert-tls                   Download CA certificate from CMS for tls
        create-default-key-transfer-policy  Create default key transfer policy for KBS
        update-service-config               Sets or Updates the Service configuration

Following environment variables are required for 'update-service-config' setup:
    SERVICE_USERNAME            The service username as configured in AAS
    AAS_BASE_URL                AAS Base URL
    KMIP_ROOT_CERT_PATH         KMIP Root Certificate path
    KMIP_SERVER_IP              IP of KMIP server
    KMIP_CLIENT_KEY_PATH        KMIP Client key path
    SERVER_READ_TIMEOUT         Request Read Timeout Duration in Seconds
    SERVER_READ_HEADER_TIMEOUT  Request Read Header Timeout Duration in Seconds
    SERVER_IDLE_TIMEOUT         Request Idle Timeout in Seconds
    SQVS_URL                    SQVS URL
    SESSION_EXPIRY_TIME         Session Expiry Time
    SERVER_PORT                 The Port on which Server Listens to
    LOG_LEVEL                   Log level
    LOG_MAX_LENGTH              Max length of log statement
    LOG_ENABLE_STDOUT           Enable console log
    KMIP_SERVER_PORT            PORT of KMIP server
    KMIP_CLIENT_CERT_PATH       KMIP Client certificate path
    SERVER_WRITE_TIMEOUT        Request Write Timeout Duration in Seconds
    SERVER_MAX_HEADER_BYTES     Max Length Of Request Header in Bytes
    SERVICE_PASSWORD            The service password as configured in AAS
    SKC_CHALLENGE_TYPE          SKC challenge type

Following environment variables are required for 'download-ca-cert'
    CMS_BASE_URL                CMS base URL in the format https://{{cms}}:{{cms_port}}/cms/v1/
    CMS_TLS_CERT_SHA384         SHA384 hash value of CMS TLS certificate

Following environment variables are required in 'download-cert-tls'
    CMS_BASE_URL        CMS base URL in the format https://{{cms}}:{{cms_port}}/cms/v1/
    BEARER_TOKEN        Bearer token for accessing CMS api
Following environment variables are optionally used in download-cert-tls
    TLS_KEY_FILE        The file to which private key is saved
    TLS_COMMON_NAME     The common name of signed certificate
    TLS_CERT_FILE       The file to which certificate is saved

    TLS_SAN_LIST        Comma separated list of hostnames to add to Certificate, including IP addresses and DNS names

```

### Directory Layout

The Key Broker Service installs by default to /opt/kbs with the following folders.

#### Bin

Contains executable scripts and binaries.

#### Configuration

This folder /etc/kbs contains certificates, keys, and configuration files.

#### Logs

This folder contains log files: /var/log/kbs


## SGX Caching Service

### Installation Answer File Options

| Key                               | Sample Value                                                 | Description                                                  |
| --------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| CMS_BASE_URL                      | https://< CMS IP or hostname >:8445/cms/v1/                  | CMS URL for Certificate Management Service                   |
| AAS_API_URL                       | https://< AAS IP or hostname >:8444/aas/v1                   | API URL for Authentication Authorization Service             |
| SCS_ADMIN_USERNAME                | scsuser@scs                                                  | SCS Service username                                         |
| SCS_ADMIN_PASSWORD                | scspassword                                                  | SCS Service password                                         |
| BEARER_TOKEN                      |                                                              | Installation Token from AAS                                  |
| CMS_TLS_CERT_SHA384               | < Certificate Management Service TLS digest >                | SHA384  Hash sum for verifying the CMS TLS certificate.      |
| INTEL_PROVISIONING_SERVER         | https://sbx.api.trustedservices.intel.com/sgx/certification/v3 | Intel pcs server url                                       |
| INTEL_PROVISIONING_SERVER_API_KEY | < Add your API subscription key >                            | Intel PCS Server API subscription key                        |
| SCS_REFRESH_HOURS                 | 1 hour                                                       | Time after which the SGX collaterals in SCS db get refreshed from  Intel PCS server |
| RETRY_COUNT                       | 3                                                            | Number Of times to connect to PCS if PCS service is not accessible |
| WAIT_TIME                         | 1                                                            | Number Of Seconds between retries to connect to PCS          |
| SCS_DB_HOSTNAME                   | localhost                                                    | SCS Databse hostname                                         |
| SCS_DB_PORT                       | 5432                                                         | SCS Database port                                            |
| SCS_DB_NAME                       | pgscsdb                                                      | SCS Database name                                            |
| SCS_DB_USERNAME                   | aasdbuser                                                    | SCS Database username                                        |
| SCS_DB_PASSWORD                   | aasdbpassword                                                | SCS Database password                                        |
| SCS_DB_SSLCERTSRC                 | /usr/local/pgsql/data/server.crt                             |                                                              |
| SAN_LIST                          | 127.0.0.1,localhost                                          | Comma-separated list of IP addresses and hostnames that will be valid connection points for the service. Requests sent to the service using an IP or hostname not in this list will be denied, even if it resolves to this service. |


### Configuration Options

The SGX Caching Service configuration can be found in /etc/scs/config.yml.

### Command-Line Options

The SGX Caching Service supports several command-line options that can be executed only as the Root user:

Syntax:

scs \<command\>

#### Available Commands

#### Help

scs help

Displays the list of available CLI commands.

#### Start

scs start

Starts the SGX Caching Service

#### Stop

scs stop

Stops the SGX Caching Service

#### Status

scs status

Reports whether the SGX Caching Service is currently running

#### Uninstall

scs uninstall \[\--purge\]

uninstall the SGX Caching Service. \--purge option needs to be applied to remove configuration files

#### Version

scs version

Reports the version of the scs

#### Setup \[task\]

Runs a specific setup task.

Syntax:

scs setup [task]

### Setup Tasks and its Configuration Options for SGX Caching Service

```shell
 Avaliable Tasks for setup:
    all                       Runs all setup tasks
                              Required env variables:
                                  - get required env variables from all the setup tasks
                              Optional env variables:
                                  - get optional env variables from all the setup tasks

    scs setup database
        - Avaliable arguments are:
            - SCS_DB_HOSTNAME
            - SCS_DB_PORT
            - SCS_DB_USERNAME
            - SCS_DB_PASSWORD
            - SCS_DB_NAME
            - SCS_DB_SSLMODE <disable|allow|prefer|require|verify-ca|verify-full>
            - SCS_DB_SSLCERT path to where the certificate file of database. Only applicable
                         for db-sslmode=<verify-ca|verify-full. If left empty, the cert
                         will be copied to /etc/scs/tdcertdb.pem
            - SCS_DB_SSLCERTSRC <path to where the database ssl/tls certificate file>
                         mandatory if db-sslcert does not already exist

    update_service_config    Updates Service Configuration
                             Required env variables:
                                 - SCS_PORT                                         : SGX Caching Service port
                                 - SCS_SERVER_READ_TIMEOUT                          : SGX Caching Service Read Timeout
                                 - SCS_SERVER_READ_HEADER_TIMEOUT                   : SGX Caching Service Read Header Timeout Duration
                                 - SCS_SERVER_WRITE_TIMEOUT                         : SGX Caching Service Request Write Timeout Duration
                                 - SCS_SERVER_IDLE_TIMEOUT                          : SGX Caching Service Request Idle Timeout
                                 - SCS_SERVER_MAX_HEADER_BYTES                      : SGX Caching Service Max Length Of Request Header Bytes
                                 - INTEL_PROVISIONING_SERVER                        : Intel ECDSA Provisioning Server URL
                                 - INTEL_PROVISIONING_SERVER_API_KEY                : Intel ECDSA Provisioning Server API Subscription key
                                 - SCS_LOGLEVEL                                     : SGX Caching Service Log Level
                                 - SCS_LOG_MAX_LENGTH                               : SGX Caching Service Log maximum length
                                 - SCS_ENABLE_CONSOLE_LOG                           : SGX Caching Service Enable standard output
                                 - SCS_REFRESH_HOURS                                : SCS Automatic Refresh of SGX Data
                                 - RETRY_COUNT                                      : Number of retry to PCS server
                                 - WAIT_TIME                                        : Duration Time between each retries to PCS
                                 - AAS_API_URL                                      : AAS API URL

    download_ca_cert         Download CMS root CA certificate
                             Required env variables specific to setup task are:
                                 - CMS_BASE_URL=<url>                                : for CMS API url
                                 - CMS_TLS_CERT_SHA384=<CMS TLS cert sha384 hash>    : to ensure that AAS is talking to the right CMS instance

    download_cert TLS        Generates Key pair and CSR, gets it signed from CMS
                             Required env variable if SCS_NOSETUP=true or variable not set in config.yml:
                                 - CMS_TLS_CERT_SHA384=<CMS TLS cert sha384 hash>      : to ensure that AAS is talking to the right CMS instance
                             Required env variables specific to setup task are:
                                 - CMS_BASE_URL=<url>               : for CMS API url
                                 - BEARER_TOKEN=<token>             : for authenticating with CMS
                                 - SAN_LIST=<san>                   : list of hosts which needs access to service
                             Optional env variables specific to setup task are:
                                 - KEY_PATH=<key_path>              : Path of file where TLS key needs to be stored
                                 - CERT_PATH=<cert_path>            : Path of file/directory where TLS certificate needs to be stored

```

### Directory Layout

The SGX Caching Service installs by default to /opt/scs with the following folders.

#### Bin

Contains SGX Caching Service executable binary.

#### Configuration

This folder /etc/scs contains certificates, keys, and configuration files.

#### Logs

This folder contains log files: /var/log/scs


## SGX Quote Verification Service

### Installation Answer File Options

| Key                      | Sample Value                                                 | Description                                                  |
| ------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| CMS_BASE_URL             | https://< CMS IP address or hostname >:8445/cms/v1/          | Defines the base URL for the CMS owned by  the image owner. Note that this CMS  may be different from the CMS used for other components. |
| AAS_API_URL              | https://< AAS IP address or hostname >:8444/aas/v1           | Defines the baseurl for the AAS owned by  the image owner. Note that this AAS  may be different from the AAS used for other components. |
| SCS_BASE_URL             | https://< SCS IP address or hostname >:9000/scs/sgx/certification/v1/ | The SCS url is needed.                              |
| SGX_TRUSTED_ROOT_CA_PATH | /tmp/trusted_rootca.pem                                      | The path to SGX root ca used to verify quote                 |
| CMS_TLS_CERT_SHA384      | < Certificate Management Service TLS digest >                | SHA384 hash of the CMS  TLS certificate                      |
| BEARER_TOKEN             |                                                              | Token from CMS with  permissions used for installation.      |
| SQVS_LOG_LEVEL           | INFO (default), DEBUG                                        | Defines the log level  for the SQVS. Defaults to INFO.       |
| SQVS_PORT                | 12000                                                        | SQVS Secure Port                                             |
| SQVS_NOSETUP             | false                                                        | Skips setup during installation if set to true               |
| SIGN_QUOTE_RESPONSE      | false                                                        | If set to false, the SQVS response is not signed, signed if set to true              |
| RESPONSE_SIGNING_KEY_LENGTH    | 3072                                                   | if SIGN_QUOTE_RESPONSE is set to true, then create RSA signing key of length defined by RESPONSE_SIGNING_KEY_LENGTH     |

| SAN_LIST                 | 127.0.0.1,localhost                                          | Comma-separated list of IP addresses and hostnames that will be valid connection points for the service. Requests sent to the service using an IP or hostname not in this list will be denied, even if it resolves to this service. |
| SQVS_INCLUDE_TOKEN       | true                                                         | If true, SQVS will authenticate KBS before Quote Verifiation |


### Configuration Options

The SGX Quote Verification Service configuration can be found in /etc/sqvs/config.yml.

### Command-Line Options

The SGX Quote Verifiction Service supports several command-line options that can be executed only as the Root user:

Syntax:

sqvs \<command\>

#### Available Commands

#### Help

sqvs help

Displays the list of available CLI commands.

#### Start

sqvs start

Starts the SGX Quote Verification Service

#### Stop

sqvs stop

Stops the SGX Quote Verification Service

#### Status

sqvs status

Reports whether the SGX Quote Verification Service is currently running.

#### Uninstall

sqvs uninstall \[\--purge\]

uninstalls the SGX Quote Verification Service. \--purge option needs to be applied to remove configuration files

#### Version

sqvs version

Reports the version of the sqvs

#### Setup \[task\]

Runs a specific setup task.

Syntax:

sqvs setup [task]

### Setup Tasks and its Configuration Options for SGX Quote Verification Service

```shell
Available Tasks for setup:
                              Required env variables:
                                  - get required env variables from all the setup tasks
                              Optional env variables:
                                  - get optional env variables from all the setup tasks

    update_service_config    Updates Service Configuration
                             Required env variables:
                                 - SQVS_PORT                                         : SGX Verification Service port
                                 - SQVS_SERVER_READ_TIMEOUT                          : SGX Verification Service Read Timeout
                                 - SQVS_SERVER_READ_HEADER_TIMEOUT                   : SGX Verification Service Read Header Timeout Duration
                                 - SQVS_SERVER_WRITE_TIMEOUT                         : SGX Verification Service Request Write Timeout Duration
                                 - SQVS_SERVER_IDLE_TIMEOUT                          : SGX Verification Service Request Idle Timeout
                                 - SQVS_SERVER_MAX_HEADER_BYTES                      : SGX Verification Service Max Length Of Request Header Bytes
                                 - SQVS_LOGLEVEL                                    : SGX Verification Service Log Level
                                 - SQVS_LOG_MAX_LENGTH                               : SGX Verification Service Log maximum length
                                 - SQVS_ENABLE_CONSOLE_LOG                           : SGX Verification Service Enable standard output
                                 - SQVS_INCLUDE_TOKEN                                : Boolean value to decide whether to use token based auth or no auth for quote verifier API
                                 - SGX_TRUSTED_ROOT_CA_PATH                          : SQVS Trusted Root CA
                                 - SCS_BASE_URL                                      : SGX Caching Service URL
                                 - AAS_API_URL                                       : AAS API URL

    download_ca_cert         Download CMS root CA certificate
                             Required env variables specific to setup task are:
                                 - CMS_BASE_URL=<url>                                : for CMS API url
                                 - CMS_TLS_CERT_SHA384=<CMS TLS cert sha384 hash>    : to ensure that AAS is talking to the right CMS instance

    download_cert TLS        Generates Key pair and CSR, gets it signed from CMS
                                 - CMS_TLS_CERT_SHA384=<CMS TLS cert sha384 hash>      : to ensure that AAS is talking to the right CMS instance
                             Required env variables specific to setup task are:
                                 - CMS_BASE_URL=<url>               : for CMS API url
                                 - BEARER_TOKEN=<token>             : for authenticating with CMS
                                 - SAN_LIST=<san>                   : list of hosts which needs access to service
                             Optional env variables specific to setup task are:
                                - KEY_PATH=<key_path>              : Path of file where TLS key needs to be stored
                                - CERT_PATH=<cert_path>            : Path of file/directory where TLS certificate needs to be stored

    create_signing_key_pair  Generates Key pair and CSR and downloads Signing certificate from CMS
                             - Option [--force] overwrites any existing files and always downloads new Signing cert
                             Required env variable if SQVS_NOSETUP=true or variable not set in config.yml:
                                 - CMS_TLS_CERT_SHA384=<CMS TLS cert sha384 hash>      : to ensure that AAS is talking to the right CMS instance
                             Required env variables specific to setup task are:
                                 - CMS_BASE_URL=<url>               : for CMS API url
                                 - BEARER_TOKEN=<token>             : for authenticating with CMS


```

### Directory Layout

The SGX Quote Verification Service installs by default to /opt/sqvs with the following folders.

#### Bin

This folder contains executable scripts.

#### Configuration

This folder /etc/sqvs contains certificates, keys, and configuration files.

#### Logs

This folder contains log files: /var/log/sqvs

