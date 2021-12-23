# Uninstallation

This section describes steps used for uninstalling Intel SecL-DC services.


## Certificate Management Service

To uninstall the Certificate Management Service, run the following command:

```
cms uninstall \--purge
```

Removes following directories:

1.  /opt/cms

2.  /run/cms

3.  /var/log/cms

4.  /etc/cms


## Authentication and Authorization Service

To uninstall the Authentication and Authorization Service, run the following command:

```
authservice uninstall \--purge
```

Removes following directories:

1.  /opt/authservice

2.  /run/authservice

3.  /var/log/authservice

4.  /etc/authservice


## SGX Host Verification Service

To uninstall the SGX Host Verification Service, run the following command:

```
shvs uninstall \--purge
```

Removes following directories:

1.  /opt/shvs

2.  /run/shvs

3.  /var/log/shvs

4.  /etc/shvs


## SGX_Agent

To uninstall the SGX Agent, run the following command:

```
sgx_agent uninstall \--purge
```

Removes following directories:

1.  /opt/sgx_agent

2.  /run/sgx_agent

3.  /var/log/sgx_agent

4.  /etc/sgx_agent

## Integration Hub

To uninstall the Integration Hub, run the following command:
```
ihub uninstall \--purge
```
Removes the following directories:

1.  /opt/ihub

2.  /run/ihub

3.  /var/log/ihub

4.  /etc/ihub

## SGX Caching Service

To uninstall the SGX Caching Service , run the following command:
```
scs uninstall \--purge
```
Removes the following directories:

1.  /opt/scs

2.  /run/scs

3.  /var/log/scs

4.  /etc/scs

## SGX Quote Verification Service

To uninstall the SGX Quote Verification Service, run the following command:

```
sqvs uninstall \--purge
```
Removes the following directories:

1.  /opt/sqvs

2.  /run/sqvs

3.  /var/log/sqvs

4.  /etc/sqvs

## Key Broker Service

```
kbs uninstall \--purge
```

To uninstall the Key Broker Service , run the following command:

Removes the following directories:

1.  /opt/kbs

2.  /run/kbs

3.  /var/log/kbs

4.  /etc/kbs


## SKC Library

To uninstall the SKC Library, run the following command:
```
./opt/skc/devops/scripts/uninstall.sh
```
Removes the following directories:

   /opt/skc



## isecl-k8s-extensions

Cluster admin can uninstall the isecl-k8s-extensions by running following commands:

```
    kubectl delete svc isecl-scheduler-svc -n isecl
    kubectl delete deployment isecl-controller isecl-scheduler -n isecl
    kubectl delete crds hostattributes.crd.isecl.intel.com
    rm -rf /opt/isecl-k8s-extensions
    rm -rf /var/log/isecl-k8s-extensions
```

## TLS Certificates
----------------

TLS certificates for each service are issued by the Certificate
Management Service during installation. If the CMS root certificate is
changed, or to regenerate the TLS certificate for a given service, use
the following commands (note: environment variables will need to be set;
typically these are the same variables set in the service installation
.env file):

-   `<servicename> download_ca_cert`
-   Download CMS root CA certificate

-   Environment variable CMS\_BASE\_URL=\<url\> for CMS API url

-   `<servicename> download_cert TLS`
-   Generates Key pair and CSR, gets it signed from CMS

-   Environment variable CMS\_BASE\_URL=\<url\> for CMS API url

-   Environment variable BEARER\_TOKEN=\<token\> for authenticating
        with CMS

-   Environment variable KEY\_PATH=\<key\_path\> to override default
        specified in config

-   Environment variable CERT\_PATH=\<cert\_path\> to override
        default specified in config


