# Workload Service

## Installation Answer File Options

| Key                 | Sample Value                                        | Description                                                  |
| ------------------- | --------------------------------------------------- | ------------------------------------------------------------ |
| WLS\_LOGLEVEL       | INFO                                                | (Optional) Alternatives include WARN and DEBUG. Sets the log level for the service. |
| WLS\_NOSETUP        | false                                               | (Optional) Determines whether “setup” will be executed after installation. Typically this is set to “false” to install and perform setup in one action. The “true” option is intended for building the service as a container, where the installation would be part of the image build, and setup would be performed when the container starts for the first time to generate any persistent data. Defaults to “false” if unset. |
| WLS\_PORT           | 5000                                                | (Optional) Defines the HTTPS port used by the service Defaults to 5000 if unset. |
| WLS\_DB\_HOSTNAME   | localhost                                           | (Required) Database hostname                                 |
| WLS\_DB             | wlsdb                                               | (Required) Database name                                     |
| WLS\_DB\_PORT       | 5432                                                | (Required) Database port number                              |
| WLS\_DB\_USERNAME   | wlsdbuser                                           | (Required) Database username                                 |
| WLS\_DB\_PASSWORD   | wlsdbuserpass                                       | (Required) Database password                                 |
| HVS\_URL            | https://\<HVS IP address or hostname\>:8443/hvs/v2/ | (Required) Base URL for the HVS                              |
| AAS\_API\_URL       | https://\<AAS IP address or hostname\>:8444/aas/v1  | Base URL for the AAS                                         |
| SAN\_LIST           | 127.0.0.1,localhost,10.x.x.x                        | Comma-separated list of IP addresses and hostnames that will be valid connection points for the service. Requests sent to the service using an IP or hostname not in this list will be denied, even if it resolves to this service. |
| CMS\_BASE\_URL      |                                                     | Base URL for the CMS                                         |
| BEARER\_TOKEN       | \<token\>                                           | (Required) Token from the CMS generated during CMS setup that allows the AAS to perform initial setup tasks. |
| WLS\_TLS\_CERT\_CN  | 'WLS TLS Certificate                                | (Optional) Set the Common name for TLS cert to be downloaded from CMS. Default is 'WLS TLS Certificate'. |
| WLS\_CERT\_ORG      | 'INTEL'                                             | (Optional) Set the Organization in Subject of CSR. Default is 'INTEL'. |
| WLS\_CERT\_COUNTRY  | 'US'                                                | (Optional) Set the Country in Subject of CSR. Default is 'US'. |
| WLS\_CERT\_PROVINCE | 'SF'                                                | (Optional) Set the Province in Subject of CSR. Default is 'SF'. |
| WLS\_CERT\_LOCALITY | 'SC'                                                | (Optional) Set the Locality in Subject of CSR. Default is 'SC'. |
| KEY\_CACHE\_SECONDS | 300                                                 | (Optional) Set the time till which the key will be cached. Default is '300 seconds'. |
| WLS\_LOGLEVEL       | Info, debug, error, warn                            | (Optional) Set the log level.                                |
| KEY\_PATH           |                                                     | (Optional) Redefines the path to the keystore folder         |
| CERT\_PATH          |                                                     | (Optional) Redefines the path to the certificates folder     |

## Configuration Options

The Workload Service configuration can be found in
`/etc/wls/config.yml`:

```
port: 5000
cmstlscertdigest: <sha384>
postgres:
  dbname: wlsdb
  user: <database username>
  password: <database password>
  hostname: <database IP or hostname>
  port: 5432
  sslmode: false
hvs_api_url: https://<HVS IP or hostname>:8443/hvs/v2/
cms_base_url: https://<CMS IP or hostname>:8445:/cms/v1/
aas_api_url: https://<AAS IP or hostname>:8444/aas/v1/
subject:
  tlscertcommonname: WLS TLS Certificate
  organization: INTEL
  country: US
  province: SF
  locality: SC
wls:
  user: <username of service account used by WLS to access other services>>
  password: <password>
loglevel: info
key_cache_seconds: 300
```

## Command-Line Options

The Workload Service supports several command-line commands that can be
executed only as the Root user:

Syntax:

`wls <command>`

###  Help

Available Commands:

`help` Show this help message

`start` Start wls

`stop` Stop wls

`status` Determine if wls is running

`uninstall \ [--purge\` Uninstall wls. --purge option needs to be applied to remove configuration and data files

`setup` Setup wls for use

Setup command usage: wls <command\> [task...]

Available tasks for setup:

download-ca-cert

\- Download CMS root CA certificate

\- Environment variable CMS_BASE_URL=<url> for CMS API url

download\_cert TLS

\- Generates Key pair and CSR, gets it signed from CMS

\- Environment variable CMS_BASE_URL=\<url> for CMS API url

\- Environment variable BEARER_TOKEN=<token> for authenticating with
CMS

\- Environment variable KEY_PATH=<key_path> to override default
specified in config

\- Environment variable CERT_PATH=<cert_path> to override default
specified in config

\- Environment variable WLS_TLS_CERT_CN=<COMMON NAME> to override
default specified in config

\- Environment variable WLS_CERT_ORG=<CERTIFICATE ORGANIZATION> to
override default specified in config

\- Environment variable WLS_CERT_COUNTRY=<CERTIFICATE COUNTRY> to
override default specified in config

\- Environment variable WLS_CERT_LOCALITY=<CERTIFICATE LOCALITY> to
override default specified in config

\- Environment variable WLS_CERT_PROVINCE=<CERTIFICATE PROVINCE> to
override default specified in config

server Setup http server on given port

-Environment variable WLS\_PORT=<port> should be set

database Setup wls database

Required env variables are:

\- WLS_DB_HOSTNAME : database host name

\- WLS_DB_PORT : database port number

\- WLS_DB_USERNAME : database user name

\- WLS_DB_PASSWORD : database password

\- WLS_DB : database schema name

hvsconnection Setup task for setting up the connection to the Host
Verification Service(HVS)

Required env variables are:

\- HVS_URL : HVS URL

aasconnection Setup to create workload service user roles in AAS

\- AAS_API_URL : AAS API URL

\- BEARER_TOKEN : Bearer Token

logs Setup wls log level

\- Environment variable WLS_LOG_LEVEL=<log level\> should be set

####  start

Start wls

####  stop

Stop wls

####  status

Determine if wls is running

####  uninstall

\[--purge\] Uninstall wls. --purge option needs to be
applied to remove configuration and data files

####  setup

Setup command usage:     wls setup [task] [--force]

Available tasks for setup:
   all                              Runs all setup tasks
   download-ca-cert                 Download CMS root CA certificate
   download-cert-tls                Generates Key pair and CSR, gets it signed from CMS
   database                         Setup wls database
   download-saml-ca-cert            Setup to download SAML CA certificates from HVS
   update-service-config            Sets or Updates the Service configuration  
### Directory Layout

The Workload Service installs by default to /opt/wls with the following
folders.
