# Certificate and Key Management

## Host Verification Service Certificates and Keys

The Host Verification Service has several unique certificates not
present on other services.The HVS certificates can be found under 
`<nfs_path mentioned in values.yaml>/isecl/hvs` path.
---
**Note**
Please refer detail steps on running setup task and updating specific service on `Changing Configuration Settings` section.
To know about supported HVS setup task and configurations refer `HVS Setup Task and Configuration Settings` section.
To know about supported Trust Agent setup task and configurations refer `Trust Agent Setup Task and Configuration Settings` section.
---

### SAML

The SAML Certificate is used to sign SAML attestation reports, and is
itself signed by the Root Certificate. This certificate is unique to the
Verification Service.

`<nfs_path mentioned in values.yaml>/isecl/hvs/config/certs/trustedca/saml-cert.pem`

When the SAML certificate is regenerated, all hosts will immediately be
added to a queue to generate a new attestation report, since the old
signing certificate is no longer valid.

If the Integration Hub is being used, the new SAML certificate will need
to be imported to the IHub.

### Asset Tag

The Asset tag Certificate is used to sign all Asset Tag Certificates.
This certificate is unique to the Verification Service.

`<nfs_path mentioned in values.yaml>/isecl/hvs/config/certs/trustedca/tag-ca-cert.pem`


### Privacy CA

The Privacy CA certificate is used as part of the certificate chain for
creating the Attestation Identity Key (AIK) during Trust Agent
provisioning. The Privacy CA must be a self-signed certificate. This
certificate is unique to the Verification Service.

The Privacy CA certificate is used by Trust Agent nodes during Trust
Agent provisioning; if the Privacy CA certificate is changed, all Trust
Agent nodes will need to be re-provisioned.

`<nfs_path mentioned in values.yaml>/isecl/hvs/config/certs/trustedca/privacy-ca/privacy-ca-cert.pem`

### Endorsement CA

The Endorsement CA is a self-signed certificate used during Trust Agent
provisioning.

`<nfs_path mentioned in values.yaml>/isecl/hvs/config/certs/endorsement/EndorsementCA.pem`


## TLS Certificates
----------------

TLS certificates for each service are issued by the Certificate
Management Service during installation. If the CMS root certificate is
changed, or to regenerate the TLS certificate for a given service:

* To download CMS root CA certificate execute setup task `download-ca-cert`
* To Generate Key pair and CSR, gets it signed from CMS execute setup task `download-cert-tls`

---
**Note**
Refer `Changing Configuration Settings` section to run required setup tasks.
---
