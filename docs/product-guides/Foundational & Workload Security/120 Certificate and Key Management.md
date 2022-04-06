# Certificate and Key Management

## Host Verification Service Certificates and Keys

The Host Verification Service has several unique certificates not
present on other services.

### SAML

The SAML Certificate a is used to sign SAML attestation reports, and is
itself signed by the Root Certificate. This certificate is unique to the
Verification Service.

`/opt/hvs/configuration/saml.crt`

`/opt/hvs/configuration/saml.crt.pem`

`/opt/hvs/configuration/SAML.jks`

The SAML Certificate can be replaced with a user-specified keypair and
certificate chain using the following command:

hvs replace-saml-key-pair --private-key=new.key.pem
--cert-chain=new.cert-chain.pem

This will:

-   Replace key pair in `/opt/hvs/configuration/SAML.jks`, alias
    samlkey1

-   Update `/opt/hvs/configuration/saml.crt` with saml DER public key
    cert

-   Update `/opt/hvs/configuration/saml.crt.pem` with saml PEM public
    key cert

-   Update configuration properties:

```shell
saml.key.password to null
saml.certificate.dn
saml.issuer
```
When the SAML certificate is replaced, all hosts will immediately be
added to a queue to generate a new attestation report, since the old
signing certificate is no longer valid. No service restart is necessary.

If the Integration Hub is being used, the new SAML certificate will need
to be imported to the Hub.

### Asset Tag

The Asset tag Certificate is used to sign all Asset Tag Certificates.
This certificate is unique to the Verification Service.

`/opt/hvs/configuration/tag-cacerts.pem`

The Asset Tag Certificate can be replaced with a user-specified keypair
and certificate chain using the following command:

`hvs replace-tag-key-pair --private-key=new.key.pem
--cert-chain=new.cert-chain.pem`

This will:

-   Replace key pair in database table mw\_file (cakey is private and
    public key pem formatted, cacerts is cert chain)

-   Update `/opt/hvs/configuration/tag-cacerts.pem with cert chain`

-   Update configuration properties:

```shell
tag.issuer.dn
```
No service restart is needed. However, all existing Asset Tags will be
considered invalid, and will need to be recreated. It is recommended to
delete any existing Asset Tag certificates and Flavors, and then
recreate and deploy new Tags.

### Privacy CA

The Privacy CA certificate is used as part of the certificate chain for
creating the Attestation Identity Key (AIK) during Trust Agent
provisioning. The Privacy CA must be a self-signed certificate. This
certificate is unique to the Verification Service.

The Privacy CA certificate is used by Trust Agent nodes during Trust
Agent provisioning; if the Privacy CA certificate is changed, all Trust
Agent nodes will need to be re-provisioned.

`/opt/hvs/configuration/PrivacyCA.p12`

`/opt/hvs/configuration/PrivacyCA.pem`

The Privacy CA Certificate can be replaced with a user-specified keypair
and certificate chain using the following command:

`hvs replace-pca-key-pair --private-key=new.key.pem
--cert-chain=new.cert-chain.pem`

This will:

-   Replace key pair in `/opt/hvs/configuration/PrivacyCA.p12`, alias
    1

-   Update `/opt/hvs/configuration/PrivacyCA.pem` with cert

-   Update configuration properties:

```shell
hvs.privacyca.aik.issuer
hvs.privacyca.aik.validity.days
```
After the Privacy CA certificate is replaced, all Trust Agent hosts will
need to be re-provisioned with a new AIK:

`tagent setup download-mtwilson-privacy-ca-certificate --force`

`tagent setup request-aik-certificate --force`

`tagent restart`

### Endorsement CA

The Endorsement CA is a self-signed certificate used during Trust Agent
provisioning.

`/opt/hvs/configuration/EndorsementCA.p12`

`/opt/hvs/configuration/EndorsementCA.pem`

The Endorsement CA Certificate can be replaced with a user-specified
keypair and certificate chain using the following command:

`hvs replace-eca-key-pair --private-key=new.key.pem
--cert-chain=new.cert-chain.pem`

This will:

-   Replace key pair in `/opt/hvs/configuration/EndorsementCA.p12`,
    alias 1

-   Update `/opt/hvs/configuration/EndorsementCA.pem` with accepted
    ECs

-   Update configuration properties:

```{=html}
hvs.privacyca.ek.issuer
hvs.privacyca.ek.validity.days
```
After the Endorsement CA certificate is replaced, all Trust Agent hosts
will need to be re-provisioned with a new Endorsement Certificate:

`tagent setup request-endorsement-certificate --force`

`tagent restart`



## TLS Certificates
----------------

TLS certificates for each service are issued by the Certificate
Management Service during installation. If the CMS root certificate is
changed, or to regenerate the TLS certificate for a given service, use
the following commands (note: environment variables will need to be set;
typically these are the same variables set in the service installation
.env file):

-   `<servicename> download-ca-cert`
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

