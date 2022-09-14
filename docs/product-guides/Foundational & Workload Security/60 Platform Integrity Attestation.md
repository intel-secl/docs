# Platform Integrity Attestation

Platform attestation is the cornerstone use case for ISecL. Platform
attestation involves taking measurements of system components during
system boot, and then cryptographically verifying that the actual
measurements taken matched a set of expected or approved values,
ensuring that the measured components were in an acceptable or "**trusted**"
state at the time of the last system boot.

ISecL leverages the Trusted Compute Group specification for a trusted
boot process, extending measurements of platform components to registers
in a Trusted Platform Module, and securely generating quotes of those
measurements from the TPM for remote comparison to expected values
(attestation).

This section includes basic REST API examples for these workflows. See
the Javadoc for more detailed documentation on REST APIs supported by
ISecL.

Typical workflows in the datacenter might include:

-   Creating a set of acceptable flavors for attestation with automatic
    flavor matching that represent the known-good measurements for
    acceptable BIOS and OS versions in the datacenter

-   Registering hosts for attestation with automatic flavor matching

-   Upgrading hosts in the datacenter to a new BIOS or OS version

-   Removing hosts from the Verification Service

-   Removing flavors

-   Provisioning asset tags to hosts

-   Invalidating asset tags

-   Retrieving current attestation reports

-   Retrieving current host state information

-   Remediating an untrusted attestation

## Host Registration
-----------------

Registration creates a host record with connectivity details and other
host information in the Verification Service database. This host record
will be used by the Verification Service to retrieve TPM attestation
quotes from the Trust Agent to generate an attestation report.

### Trust Agent

### Registration via Verification Service API

Any Trust Agent or VMware ESXi host/cluster can be registered using a
Verification Service API request. Registration can be performed with or
without a set of existing Flavors. Rules for Flavor matching can be set
by using the Flavor Group in the request; if no Flavor Group is
specified, the `automatic` Flavor Group will be used. See the
Flavor Management section for additional details on Flavors, Flavor
Groups, and Flavor matching.

#### Sample Call

```
POST https://verification.service.com:30443/hvs/v2/hosts
Authorization: Bearer <token>

{
    "host_name": "<hostname of host to be registered>",
    "connection_string": "<connection string>",
    "flavorgroup_name" : "",
    "description" : "<description>"
}
```

Requires the permission `hosts:create`

#### Sample Call for ESXi Cluster Registration

```
POST https://verification.service.com:30443/hvs/v2/hosts
Authorization: Bearer <token>

{
    " esxi_clusters": [
       {
          "connection_string": "<password>",
          "cluster_name": "<cluster name>"
       }
    ]
 }
```

Requires the permission `esxi_clusters:create`



## Flavor Creation for Automatic Flavor Matching

Flavor creation is the process of adding one or more sets of acceptable
measurements to the Verification Service database. These measurements
correspond to specific system components, and are used as the basis of
comparison to generate trust attestations.

Flavors can be created manually, or can be imported from an example
host.

Flavors are automatically matched to hosts based on the Flavorgroup used
by the host and the Flavors, and the Flavor Match Policies of the
Flavorgroup. The ISecL Verification Service creates a default
Flavorgroups during installation called "**automatic**" This Flavorgroup is
configured to be used as a pool of all acceptable Flavors in a given
environment, and will automatically match the appropriate Flavor parts
to the correct host. This Flavorgroup is used by default and is expected
to be useful for the majority of deployments. If no Flavorgroup is
specified when creating a Flavor, it will be placed in the "**automatic**"
Flavorgroup.

Flavors are also divided into Flavor parts, which correspond to the
`PLATFORM`, `OS`, `HOST_UNIQUE`, `IMA`, and `ASSET_TAG` measurements. These
can be created and maintained separately (so that users can manage
acceptable OS and BIOS versions, rather than entire host
configurations). By default, if not specified, the Verification Service
will import Flavors as separate Flavor parts, as appropriate for the
host type.

By using individual Flavor parts, individual versions of OS or PLATFORM
measurements can be managed and automatically mapped. Whenever a host
changes states (Untrusted, Connected, etc.) the Verification Service
will attempt to match appropriate Flavors to that host. If a Flavor is
removed or added, all appropriate hosts will be updated to use the new
Flavor, or to no longer use the deleted Flavor. Hosts that are currently
using a BIOS where that BIOS versions’ PLATFORM Flavor was deleted will
now appear Untrusted, for example. This can be used to easily flag as
Untrusted hosts that are using software that has been End-Of-Lifed, or
perhaps an OS kernel with a known security vulnerability.

???+ note 
    See the Flavor Management section for additional details on how flavors can be managed, and how the Flavor matching engine works. The sample workflow provided here is intended to be an introduction only.

### Importing a Flavor from a Sample Host

```
POST https://verification.service.com:30443/hvs/v2/flavors
Authorization: Bearer <token>

{
    "connection_string": "<connection string>",
    "partial_flavor_types": ["PLATFORM", "OS", "HOST_UNIQUE"],
    "flavorgroup_names": []
}
```

Requires the permission `flavors:create`

???+ note 
    The HOST\_UNIQUE Flavor parts, used by Red Hat Enterprise Linux and VMWare ESXi host types, MUST be created for each registered host of that type, and should in general be imported from that host. This means that importing the HOST\_UNIQUE flavor should always be done for each host registered.

To import ONLY the HOST\_UNIQUE Flavor part from a host:

```
POST https://verification.service.com:30443/hvs/v2/flavors
Authorization: Bearer <token>

{
    "connection_string": "<connection string>",
    "partial_flavor_types": ["HOST_UNIQUE"],
    flavorgroup_names": []
}
```

Requires the permission `flavors:create`

### Creating a Flavor Manually

Flavors can be directly created (rather than importing from a sample
host) if the required information is known. If no Flavorgroup is
specified, the Flavor will be placed in the `automatic` group. Note that
the `label` is a required field and must be unique.

```
POST https://verification.service.com:30443/hvs/v2/flavors
Authorization: Bearer <token>

{
    "connection_string": "",
    "flavor_collection": {
        "flavors": [
            {
                "meta": {
                    "vendor": "INTEL",
                    "description": {
                        "flavor_part": "PLATFORM",
                        "label": "Intel Corporation_SE5C610.86B.01.01.1008.031920151331_TPM2.0",
                        "bios_name": "Intel Corporation",
                        "bios_version": "SE5C620.86B.00.01.0004.071220170215",
                        "tpm_version": "2.0"
                    }
                },
                "hardware": {
                    "processor_info": "…",
                    "processor_flags": "…",
                    "feature": {
                        "tpm": {
                            "enabled": true,
                            "pcr_banks": [
                                "SHA1",
                                "SHA256"
                            ]
                        },
                        "txt": {
                            "enabled": true
                        }
                    }
                },
                "pcrs": {
                    "SHA1": {
                        "pcr_0": {
                            "value": "d2ed125942726641a7260c4f92beb67d531a0def"
                        },
                        "pcr_17": {
                            "value": "1ec12004b371e3afd43d04155abde7476a3794fa",
                            "event": ...
                        }

```

Requires the permission `flavors:create`



## Creating the Default SOFTWARE Flavor (Linux Only)

As part of the new Application Integrity feature added in Intel® SecL-DC
version 1.5, a new default SOFTWARE Flavor part is provided so that the
Linux Trust Agent itself can be measured and included in the attestation
process. The default SOFTWARE Flavor includes a manifest for the static
files and folders in the Trust Agent. The manifest is automatically
deployed to each Linux Trust Agent during the provisioning step.

???+ note 
    The Linux Trust Agent **must** be rebooted after the Provisioning step is completed (typically Provisioning happens during installation, based on whether all of the required variables are set in the trustagent.env file). Rebooting allows the default SOFTWARE Flavor manifest to be measured and extended to the TPM PCRs. If the reboot is not performed, the system will require a SOFTWARE Flavor, but the measurements will not exist, and the system will appear Untrusted. If an un-rebooted host is used to create the SOFTWARE Flavor, the Flavor will be created based on measurements that do not exist, and will fail.

The `SOFTWARE` Flavor part should be created separately from the other
Flavor parts. Only one default `SOFTWARE` Flavor needs to be created for
each version of the Linux Trust Agent. If the `SOFTWARE` Flavor for the
same Trust Agent version is imported multiple times, subsequent imports
will fail as the Flavor already exists.

To import the `SOFTWARE` Flavor part from a host:

```
POST https://verification.service.com:30443/hvs/v2/flavors
Authorization: Bearer <token>

{
    "connection_string": "<connection string>",
    "partial_flavor_types": ["SOFTWARE"],
    flavorgroup_names": []
}
```

Requires the permission `flavors:create`



## Creating and Provisioning Asset Tags

Asset Tags represent a set of key/value pairs that can be associated
with a host in hardware. This enables usages around restricting
workflows to specific hosts based on tags, which could include location
information, compliance tags, etc.

ISecL creates Asset Tags by creating a certificate containing the list
of key/value pairs to be tagged to the host, with the host’s hardware
UUID as the certificate subject. A hash of this certificate is then
written to an NVRAM index in the host’s TPM. This value is included in
TPM quotes, and can be attested using an Asset Tag flavor that matches
up the expected value and the actual key/value pairs.

### Creating Asset Tag Certificates

Asset Tag certificates can be created with a single REST API call, with
any number of key/value pairs. Note that one certificate must be created
for each host to be tagged, even if they will all be tagged with
identical key/value pairs.

```
POST https://verification.service.com:30443/hvs/v2/tag-certificates
Authorization: Bearer <token>

{
    "hardware_uuid": "<hardware UUID of host to be tagged>",
    "selection_content": [
        {
            "name": "<key>",
            "value": "<value>"
        },
        {
            "name": "<key>",
            "value": "<value>"
        },
        {
            "name": "<key>",
            "value": "<value>"
        }
    ]
}
```

### Deploying Asset Tags

#### Red Hat Enterprise Linux

Asset Tags can be provisioned to a Windows or RHEL host via a REST API
request on the Verification Service that will in turn make a request to
the Trust Agent on the host to be tagged.

```
POST https://verification.service.com:30443/hvs/v2/rpc/deploy-tag-certificate
Authorization: Bearer <token>

{
    "certificate_id": "<certificate ID>"
}
```

#### VMWare

Since VMWare ESXi hosts do not use a Trust Agent, the process for
writing Asset Tags to a VMWare host is different from RHEL. A
new interface has been added to ESXi via a new `esxcli` command starting
in vSphere 6.5 Update 2 that allows the Asset Tag information to be
written to the TPM via a command-line command. The older process is also
described below.

The high-level workflow for using Asset Tags with VMWare ESXi is:

1.  Create the Asset Tag Certificate for the host.
2.  Calculate the Certificate Hash value.
3.  Provision the Certificate Hash value to the host TPM and reboot
4.  Create the Asset Tag Flavor.

???+ note 
    Asset Tag is currently not supported for VMWare hosts using TPM 2.0.

##### Calculate the Certificate Hash Value

Only the hash value of the Asset Tag Certificate can be provisioned to
the TPM, due to the low size of the NVRAM.

1.  Retrieve the Asset Tag Certificate. The Asset Tag Certificate can be
    retrieved either from the response when the Asset Tag certificate is
    created, or by using a GET API request to retrieve the certificate:

    ```
    GET https://verification.service.com:30443/hvs/v2/tag-certificates?subjectEqualTo=<HardwareUUID>
    Authorization: Bearer <token>
    ```

2.  Copy only the `certificate` value (this will be the certificate in
    encoded format) and write the data to a file on a Linux system.
    Remove any line breaks and save the file. Assuming the filename used
    is `tag-cert` use the following to generate the correct hash:

    ```shell
    cat tag-cert | base64 --decode | openssl dgst -sha1 | awk -F" " '{print $2}'
    ```

    This hash value will be what is actually written to the TPM NVRAM.

##### Provision the Certificate Hash to the Host TPM

Due to a new feature added in vSphere 6.5 Update 2, the process for
provisioning Asset Tags on VMWare ESXi hosts has been significantly
improved. Both the old and new process for provisioning Asset Tags is
documented below. Intel recommends using vSphere 6.5 Update 2 or later
due to the significant difference in the process.

###### **vSphere 6.5 Update 2 or Later**

Starting in ESXi 6.5u2, you can now use SSH to write Asset Tags directly
with no need for TPM clears, reboots, PXE, or BIOS access. SSH to the
ESXi host using root credentials. Then use the command:

1.  ```shell
    esxcli hardware tpm tag set -d <hash>
    ```

    You can use the following command to verify that the tag was written:

    ```shell
    esxcli hardware tpm tag get
    ```

2.  Reboot the host. After rebooting, the TPM PCR 22 will have the
    measured value of the hash.

###### **vSphere 6.5 Update 1 or Older**

There is no direct interface from VMWare vCenter or ESXi previous to
vSphere 6.5 Update 2 that will write the Tag information to the host
TPM.

Writing Asset Tag information to a TPM requires TPM ownership; VMWare
ESXi takes TPM ownership with a secret password at boot time. This means
that the process for writing Asset Tags to a VMWare host requires:

1.  Clear TPM ownership.

    * This can be done via the system BIOS, or using One Touch Activation
      through the IPMI interface (if enabled by the server OEM).

2. Reactivate TPM/TXT.
   * This can be done via the system BIOS, or using One Touch Activation
     through the IPMI interface (if enabled by the server OEM).

3. Booting to an OS that has the ability to issue TPM commands
   * Typically the provisioning OS used is Ubuntu or RHEL, booted
     temporarily using PXE.

4. Writing the Tag information
   * The TPM index 0x40000010 must be defined, and the hash of the Asset
     Tag certificate must be written to that index.

5. Clear TPM ownership.
   * This can be done via the system BIOS, or using One Touch Activation
     through the IPMI interface (if enabled by the server OEM).

6. Reactivate TPM/TXT
   * This can be done via the system BIOS, or using One Touch Activation
     through the IPMI interface (if enabled by the server OEM).

7. Boot back to VMWare ESXi.

When the system is rebooted to ESXi, the Trusted Boot process will
extend the value to PCR22, and this value can be used during
attestation.

##### Creating the Asset Tag Flavor (VMWare ESXi Only)

While for RHEL and Windows hosts the Asset Tag Flavor is automatically
created during the Tag Provisioning step, for VMWare ESXi hosts the
Flavor must be created by importing it from the host after the Tag has
been provisioned.

```
POST https://verification.service.com:30443/hvs/v2/flavors
Authorization: Bearer <token>

{   
    "connection_string": "<VMWare vCenter connection string>",
    "partial_flavor_types": ["ASSET_TAG"]
}
```

Once the Asset Tag Flavor is imported, the host can be attested
including Asset Tags as normal.



## Retrieving Current Attestation Reports

```
GET https://verification.service.com:30443/hvs/v2/reports?latestPerHost=true
Authorization: Bearer <token>
```



## Retrieving Current Host State Information

```
GET https://verification.service.com:30443/hvs/v2/host-status?latestPerHost=true
Authorization: Bearer <token>
```



## Upgrading Hosts in the Datacenter to a New BIOS or OS Version

Software and firmware updates are a common occurrence in the datacenter.
Automatic Flavor matching makes this process relatively simple:

1.  Create a new Flavor for the new version. This may be manually
    created or imported directly from a sample host that has already
    received the upgrade. Be sure to create new Flavors for each TPM
    version represented in your datacenter.

    ```
    POST https://verification.service.com:30443/hvs/v2/flavors
    Authorization: Bearer <token>

    {
        "connection_string": "<connection string>",
        "partial_flavor_types": ["PLATFORM", "OS", "HOST_UNIQUE"],
        flavorgroup_names": []
    }
    ```

2.  Update the hosts to the new software or firmware version as normal.
    On the next attestation attempt, the Verification Service will
    automatically match the updated hosts to the new Flavor.

3.  (Optional) If desired, delete the Flavor for the older version after
    the update is completed. This will cause any hosts that are still
    using the old version to attest as Untrusted. Which can easily flag
    hosts that missed the upgrade for remediation.

    ```
    DELETE https://verification.service.com:30443/hvs/v2/flavors/<flavorId>
    Authorization: Bearer <token>
    ```



## Removing Hosts From the Verification Service

Hosts can be deleted at any time. Reports for that host will remain in
the Verification Service database for audit purposes.

```
DELETE https://verification.service.com:30443/hvs/v2/hosts/<hostId>
Authorization: Bearer <token>
```

The `hostId` can be retrieved either at the time the host is created, or
by searching hosts using the host’s hostname.



## Removing Flavors

Flavors can be deleted; this will cause any hosts that match the deleted
Flavor to evaluate as Untrusted. This can be done if, for example, an
old BIOS version needs to be retired and should no longer exist in the
datacenter. By deleting the PLATFORM Flavor, hosts with the old BIOS
version will attest as Untrusted, flagging them for easy remediation.

```
DELETE https://verification.service.com:30443/hvs/v2/flavors/<flavorId>
```



## Invalidating Asset Tags
-----------------------

Asset Tags can be deleted in two ways.

Deleting the `ASSET_TAG` Flavor part will retain the Asset Tag
certificate in the database, but will cause the host using this Tag to
no longer use the Asset Tag for attestation (the Tag result will be
disregarded and no tags will be exposed in the attestation Reports).

```
DELETE https://verification.service.com:30443/hvs/v2/flavors/<assetTagflavorId>
Authorization: Bearer <token>
```

Deleting the actual Asset Tag certificate will remove the certificate
from the database, but will not actually affect attestation results (the
authority for attestation results is the Flavor).

```
DELETE https://verification.service.com:30443/hvs/v2/tag-certificates/<assetTagCertificateId>
Authorization: Bearer <token>
```



## Remediating an Untrusted attestation

Hosts can become Untrusted for a wide variety of causes. The first clue
to finding the root cause for an Untrusted attestation is the
attestation Report itself – the Report will show Trust results for the
`PLATFORM`, `OS`, `HOST_UNIQUE`, and `ASSET_TAG` Flavor parts individually,
along with the `OVERALL` trust. If the Report shows that the PLATFORM
Flavor part trust is “false” for example, it means that the PLATFORM
measurements did not match any Flavors in the host’s Flavorgroup.

Untrusted attestation Reports will contain **faults** that describe the
specific attestation rules that were not satisfied. This often shows
enough information to describe the cause of the Untrusted status. A
fault like **RequiredButNotDefined** means that a Flavor part is required
by the Flavorgroup policy, but no Flavors for that Flavor part exist in
the Flavorgroup (for example, generally Flavorgroups should always
require a PLATFORM Flavor part; if no PLATFORM Flavors are in the
Flavorgroup, hosts in the Flavorgroup will attest with this fault).

Other faults include:

**"PcrMatchesConstant"** - describes a rule that evaluates whether a TPM
PCR has a specific value

**"PcrEventLogIntegrity"** - the module event log is replayed during
attestation to verify that the resulting measurement matches the actual
value in the module PCR. If the replay does not match, it indicates the
event log cannot itself be trusted.

**"AikCertificateTrusted"** – This rule evaluates whether the TPM quote
was signed by the TPM associated with this host. As part of host
registration, the public half of the Attestation Identity Keypair is
captured, and this public key is used to verify the signature on TPM
quotes from that host.

See the Appendix for a full list of the rules evaluated during
Attestation.

The Flavor matching engine will use the most-similar Flavor for the
attestation Report in the case of an Untrusted result.

The fault will explain in a general sense what rule the host attestation
violated. To remediate, the rule will need to be satisfied. This could
mean creating a new Flavor to match the actual observed values, or it
could mean that the host has been tampered with and should have its BIOS
flashed or OS reloaded.



## Attestation Reporting
---------------------

Attestation results are delivered in the form of Host Reports. A Report
can delivered in several different formats, which can change the type of
data returned.

The preferred format for Host Reports is a SAML attestation. A
SAML-formatted report includes a chain or signatures that provides
auditability for the Report. The SAML attestation will include the base
trust status of the host, as well as the overall trust for each
individual Flavor used in the attestation. The Report will also contain
host information, such as TPM version, Operating System name and
version, BIOS version, etc. The SAML Report will not, however, contain
individual measurements and comparisons of values. This format of the
Report is ideal for securely communicating the trust status of a host
and for audit history.

Attestation Reports can also be retrieved in `json` format. This
format will not include the signature chain provided in the SAML
format, but will contain the actual measurement values and expected
Flavor values used for comparison. These reports are typically used for
remediation, because they will show specifically why a given Host
attested as Untrusted.

The format for a Report is determined by the `Accept` header in the
request.

Attestations are automatically generated in the Verification Service by
a repeating scheduled background process. This process looks for
Attestation Reports that are close to expiration, and triggers a new
Attestation Report. By default, Attestation Reports are valid for 24 hours, and the background refresh process will trigger a new
attestation when a Report is found to be within 3 minutes of expiration.

Starting in version 4.1, new attestation reports are automatically generated whenever the Trust Agent starts, including when a host reboots.  This affects only Trust Agent hosts (and so does not apply to VMware servers).  This means that a GET request to find the most recently generated report should reflect any possible changes to the host state from flavor changes, a reboot, etc.  Forcing a new report using a POST request will ensure the state is absolutely current, but should not be required for most circumstances.  New reports are generated automatically when:

- The Trust Agent starts (via service stop/start operations, including pod starts on Kubernetes)
- The Trust Agent host is rebooted
- The HVS detects that the existing report is nearing its expiration
- If the Workload Confidentiality feature is used, new reports are generated with every key request

A user can either retrieve the most recent currently valid Attestation
Report for a given host, or may trigger a new Attestation Report to be
generated. Typically, it is best to retrieve an existing Report for
performance reasons. Generating a new Attestation Report requires the
generation of a new TPM quote from the TPM of the host being attested;
TPM performance differs greatly between vendors, and a quote can take
anywhere between 2-7 seconds to generate.

### Sample Call – Generating a New Attestation Report

```
POST https://verification.service.com:30443/hvs/v2/reports
Authorization: Bearer <token>

{
    "host_name":"host-1"
}
```

Requires the permission `reports:create`

### Sample Call – Retrieving an Existing Attestation Report

```
GET https://verification.service.com:30443/hvs/v2/reports?hostName=HostName.server.com
Authorization: Bearer <token>
```

Below are the supported criteria options in order of precedence. If no
host filter criteria is specified, then results are returned for all
active hosts.

-   `id` - unique UUID of the report entry in the database

-   `hostId` - unique UUID of the host entry in the database

-   `hostName` - name of the host

-   `hostHardwareId` - hardware UUID of the host

-   `hostStatus` - current state of the host, which supports the following
    options:

    * CONNECTED - host is in connected state
    * QUEUE - host is in queue to be processed
    * CONNECTION\_FAILURE - connection failure
    * UNAUTHORIZED - unauthorized
    * AIK\_NOT\_PROVISIONED - AIK certificate is not provisioned
    * EC\_NOT\_PRESENT - endorsement certificate is not present
    * MEASURED\_LAUNCH\_FAILURE - TXT measured launch failure
    * TPM\_OWNERSHIP\_FAILURE - TPM ownership failureTPM\_NOT\_PRESENT - TPM is not present
    * UNSUPPORTED\_TPM - unsupported TPM version
    * UNKNOWN - unknown host state

Requires the permissions `reports:search`

Other search criteria may also be used. By default, the most recent
currently valid attestation is returned. However, different query
parameters can be used to retrieve all attestations for a specific host
over the last 30 days, for example.




