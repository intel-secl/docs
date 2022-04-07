# Integration Hub

## Installation Answer File

```
# Authentication URL and service account credentials
AAS_API_URL=https://isecl-aas:8444/aas/v1
IHUB_SERVICE_USERNAME=<Integration Hub Service User username>
IHUB_SERVICE_PASSWORD=<Integration Hub Service User password>

# CMS URL and CMS webserivce TLS hash for server verification
CMS_BASE_URL=https://isecl-cms:8445/cms/v1
CMS_TLS_CERT_SHA384=<TLS hash>

# TLS Configuration
TLS_SAN_LIST=127.0.0.1,192.168.1.1,hub.server.com #comma-separated list of IP addresses and hostnames for the Hub to be used in the Subject Alternative Names list in the TLS Certificate

# Verification Service URL
HVS_BASE_URL=https://isecl-hvs:8443/hvs/v2
ATTESTATION_TYPE=HVS

#Integration tenant type.  Currently supported values are "KUBENETES" or "OPENSTACK"
TENANT=<KUBERNETES or OPENSTACK>

# OpenStack Integration Credentials - required for OpenStack integration only
OPENSTACK_AUTH_URL=<OpenStack Keystone URL; typically http://openstack-ip:5000/>
OPENSTACK_PLACEMENT_URL=<OpenStack Nova API URL; typically http://openstack-ip:8778/>
OPENSTACK_USERNAME=<OpenStack username>
OPENSTACK_PASSWORD=<OpenStack password>

# Kubernetes Integration Credentials - required for Kubernetes integration only
KUBERNETES_URL=https://kubernetes:6443/
KUBERNETES_CRD=custom-isecl
KUBERNETES_CERT_FILE=/etc/ihub/apiserver.crt
KUBERNETES_TOKEN=eyJhbGciOiJSUzI1NiIsImtpZCI6Ik......

# Installation admin bearer token for CSR approval request to CMS - mandatory
BEARER_TOKEN=eyJhbGciOiJSUzM4NCIsImtpZCI6ImE…

#Optional, configures the polling interval at which the Hub retrieves attestations from the HVS
POLL_INTERVAL_MINUTES=2

#Optional, runs the installer skipping setup
IHUB_NO_SETUP=false

#Optional, configures the TLS certificate common name
TLS_COMMON_NAME=Integration Hub TLS Certificate

#Optional, log configuration
LOG_MAX_LENGTH=1500
LOG_LEVEL=Info
LOG_ENABLE_STDOUT=true

```



## Configuration Options

```
config-file: /etc/ihub/config
log:
  max-length: 1500
  enable-stdout: true
  level: trace
ihub:
  service-username: admin@hub
  service-password: hubAdminPass
  poll-interval-minutes: 1
aas:
  url: https://<aas_ip>:8444/aas/v1
cms:
  url: https://<cms_ip>:8445/cms/v1/
  tls-cert-digest: 8a035e3cdd...
attestation-service:
  attestation-url: https://<hvs_ip>:8443/hvs/v2
  attestation-type: HVS
end-point:
  type: KUBERNETES or OPENSTACK
  url: https://<kubernetes_ip>:6443/ or OpenStack Nova URL
  crd-name: custom-isecl
  token: eyJhbGciOiJSUzI...
  username: OpenStack Username
  password: OpenStack Password
  auth-url: OpenStack Authentication URL
  cert-file: /etc/ihub/apiserver.crt
tls:
  cert-file: /etc/ihub/tls-cert.pem
  key-file: /etc/ihub/tls-key.pem
  common-name: Integration Hub TLS Certificate
  san-list: 127.0.0.1,localhost
```

## Command-Line Options

### Available Commands

#### Help

`ihub -h | --help`

Displays the list of available CLI commands.

#### Start

`ihub start`

Starts the services.

#### Stop

`ihub stop`

Stops the services.

#### Status

`ihub status`

Reports whether the service is currently running.

#### Uninstall

`ihub uninstall [--purge]`

Uninstalls the service, including the deletion of all files and folders.
Database content is not removed. If the `--purge` option is used, database
content will be removed during the uninstallation.

#### Version

`ihub -v | --version`

Reports the version of the service.

#### Setup

```
ihub setup <task> [--help] [--force] [-f <answer-file>]
```

Usage of ihub setup:
        ihub setup <task> [--help] [--force] [-f <answer-file>]
                --help                      show help message for setup task
                --force                     existing configuration will e overwritten if this flag is set
                -f|--file <answer-file>     the answer file with required arguments

Available Tasks for setup:
        all                                 Runs all setup tasks
        download-ca-cert                    Download CMS root CA certificate
        download-cert-tls                   Download CA certificate from CMS for tls
        attestation-service-connection      Establish Attestation service connection
        tenant-service-connection           Establish Tenant service connection
        create-signing-key                  Create signing key for IHUB
        download-saml-cert                  Download SAML certificate from Attestation service
        update-service-config               Sets or Updates the Service configuration

### Directory Layout

The ihub installs by default to /etc/ihub.  This directory contains the config.yaml configuration file,  saml certificate, trusted ca, and the webservice TLS certificate.

/etc/ihub/

├── apiserver.crt

├── certs

│   ├── saml

│   │   └── saml-cert.pem

│   └── trustedca

│       └── 58f6bcfcd.pem

├── config.yml

├── ihub_private_key.pem

├── ihub_public_key.pem

├── tls-cert.pem

└── tls-key.pem



#### Logs

/var/logs/ihub
