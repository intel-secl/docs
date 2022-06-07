Intel Security Libraries for Data Center
=========================================

# ***Intel(R) SecL-DC version 5.0 GA***

Foundational Security/Workload Security Use Cases:

- All bare-metal binary installation deployments have been deprecated.
- The Workload Confidentiality use case is now supported again for conatiner workloads only
- Intel SecL now supports Endorsement Certificate (EC) Pre-Registration.  This feature allows a hardware owner to pre-register TPM ECs for any devices that will later be registered to the HVS for attestation.  This feature is intended for Edge and IoT devices that may be provisioned centrally and then installed in remote locations, and is intended to prevent unauthorized devices from being registered.
- 

SKC/SGX Attestation Use Cases:

- All SGX/SKC use cases now support containerized deployment using Helm
- All bare-metal binary installation deployments have been deprecated.  

# ***Intel(R) SecL-DC version 4.2 BETA***

Foundational Security/Workload Security Use Cases:

- Intel SecL-DC no longer supports bare-metal binary installation.  All deployment of Intel SecL components will now require Kubernetes and use Helm charts for deployment.
- Because bare-metal deployments are no longer supported, the following use cases no longer apply and will be deprecated for all future releases:
  - Application Integrity
  - Virtual Machine Confidentiality
- The following use cases are unsupported for this BETA release only.  These will not be deprecated going forward, but are unsupported for the 4.2 BETA release
  - Workload Confidentiality 

SKC/ SGX Attestation Use Cases:

- Helm deployments are not currently supported for SKC / SGX use cases.  No changes have been made for this Beta release.

# ***Intel(R) SecL-DC version 4.1 GA***

Foundational/WL Use Cases:

- New attestation reports will now be triggered whenever a Trust Agent starts, including as a result of a host reboot.  Note that this does not affect VMWare hosts.  
	-	As part of this feature, the HVS /reports API now has an additional option for asynchronous report generation - see the Swagger API 
		documentation for details.  Existing calls to /reports will work with no needed changes.
	-	A new long-lived token is required to give the Trust Agent permissions to generate new asynchronous reports on startup.  To support this 
		feature in an environment upgraded from an earlier release, use the 4.1 version of the populate-users script.  Use the new resulting 	
		installation token to run the "tagent setup download-api-token" setup task on each Trust Agent host at the end of the upgrade process.
	-	The new long-lived token expires after one year.  To regenerate the token, generate a new installation bearer token using the 
		populate-users script, and use that token to run the "tagent setup download-api-token" setup task on each Trust Agent host.

- New options for tainting Kubernetes worker nodes have been added to the Intel SecL Kubernetes controller.  These new options are in addition to the previously-supported ability to taint workers on an untrusted attestation.  These new options provide an admission-control capability to help ensure that security-sensitive pods are not scheduled on workers that have not yet proved integrity.
	-	Worker nodes can now be automatically tainted on reboot, clearing the taint based only on a trusted attestation
	-	Worker nodes can now be automatically tainted when attached to the Kubernetes cluster, clearing the taint based only on a trusted
 		attestation.
	-	As with all Kubernetes worker node tainting, pods can be configured with "tolerations" to be scheduled even on tainted workers.  


# ***Intel(R) SecL-DC version 4.0 GA***

Foundational/WL Use Cases:

- Support for Client Intel® PTT (fTPM) has been added. ISecL has always supported the server implementation of PTT. However, the client implementation has a difference in the endorsement certificate hierarchies that would cause the TPM authenticity verification to fail during Trust Agent provisioning. This alternative implementation is now supported.

- Support for TPM SHA384 PCR banks has been added. Newer TPMs support SHA384, and ISecL has added support for this algorithm. The addition of another PCR bank algorithm has also forced changes in the way the HVS will behave when it encounters multiple available PCR banks. By default, the most secure PCR bank will be preferred when importing flavors or performing attestations. If a host only has one bank enabled, that bank will be used. If a host has multiple banks enabled, the HVS will choose the “best” available algorithm and disregard the others. In most cases this will not
be noticeable. In the specific case where the same flavor will be used on hosts with different PCR banks enabled however, this may result in “untrusted” results:
  - Importing a flavor from a host with SHA256 and SHA384 enabled will generate a flavor using SHA384.
  - If host is attested that only has a SHA256 bank enabled and does not have SHA384, the host will appear untrusted because no flavors can match the SHA256 hashes available, even if the servers are otherwise identical.
  - This specific scenario requires separate flavors for the SHA256 systems. A datacenter may require a RHEL 8.4 SHA384 flavor, and also a RHEL 8.4 SHA256 flavor. The OS can be identical, but the measurements used at attestation time need to match the measurements in the flavors.

- Platform-info gathering from the Trust Agent has been changed to now pull platform details directly from UEFI and ACPI tables, instead of working through intermediary applications like dmidecode.
  - Previous Trust Agent dependencies like dmidecode have been removed
  - Additional PCR event logs are now available that were previously invisible. This allows far more visibility and granularity in remote attestations and flavor definitions.

- Intel SecL now exposes Flavor templates. Previously the definitions for which PCRs and events would belong in each flavor part were hard-coded. Given specific host conditions (which security technologies are enabled, etc) and the specific flavor part (PLATFORM, OS, etc), the HVS would use specific PCRs and events. These definitions were previously not changeable by the end user. This new feature adds the capability to create new templates that cumulatively apply definitions to flavor parts based on what’s detected on the host. This allows an administrator to define which PCRs and events should be used for each flavor part, and to control those definitions based on conditions present on the host.

- The Key Broker now requires that the KMIP client certificate include Subject Alternative Names with the KMIP servers

???NOTE: 
  This requirement may mean that older KMIP client certificates will no longer be valid after an upgrade to version 4.0, if the previously generated certificates did not contain the needed SAN.

- Support for the filesystem key manager in the Key Broker Service has been removed. This feature was provided as a POC-level function to help customers get started using the Key Broker and was not intended for production use. This functionality has been removed. The Key Broker now requires a 3rd-party KMIP key manager.

- The Trust Agent and HVS now support communication in NATS mode. This does not replace the default HTTP communication but is provided as an alternative. By default, the Trust Agent exposes REST API endpoints through an open port, and communication is initiated by the HVS to make API requests to the Trust Agent. NATS mode provides an option for an alternative messaging system. In NATS mode the Trust Agent does not expose any API endpoints. Instead, the Agent will establish a connection to a NATS server that acts as a messaging system. The HVS will also route Trust Agent communication via the NATS server, allowing the same functionality provided in HTTP mode. NATS is a third-party application; additional documentation can be found at https://nats.io.
Sample configuration information for a NATS Server are provided in the Intel® Security Libraries Product Guide.

- The Trust Agent has changed how it utilizes TPM ownership. Previously the Agent would assert TPM ownership at
service startup. This is no longer required. Instead, TPM ownership is not only required at the time of TPM provisioning,
when the AIK is generated and endorsed by the HVS. This makes the Trust Agent much easier to use for administrators
who also need to use their TPMs for purposes outside of Intel® Security Libraries. The ownership secret is also no longer
stored in the config.yml file. As part of this change, the Trust Agent now supports and defaults to a NULL TPM ownership
secret. If the TPM_OWNERSHIP_SECRET variable is provided in trustagent.env, the installer will use the specified secret.
If no secret is specified, or if the value of the variable is empty, the Agent will use a NULL TPM ownership secret by
default. This change is intended to allow administrators to control and manage the platform TPM and how ownership is
used. Defaulting to a NULL secret also means that the Trust Agent will no longer require clearing the TPM ownership in
any but the rarest of cases. The Trust Agent will also now support TPM ownership secrets other than the previous 20
byte hex secret. Any ASCII string can now be utilized. To force a string to be treated as hex, prefix the secret with "hex:".
Again this allows significantly greater control over the secret for the platform administrator.

???NOTE: 
  This change will require upgrades to version 4.0 to re-provision the Trust Agent. This means that for upgrades to version 4.0, the installation bearer token must be provided. In addition, if the TPM owner secret is already set, the secret must be provided at provisioning time as well (which will typically be when the upgrade is executed). This can also be an opportunity to clear the TPM ownership and use the NULL secret or one of the other new options.

- Support for the Docker container runtime interface has been deprecated for the container confidentiality feature.
Intel® SecL now supports only the CRI-O container runtime interface for container confidentiality.

Known Issues:
???
  Important Note:** SGX Attestation fails when SGX is enabled on a host booted using tboot
  
**Root Cause:** 
tboot requires the "noefi" kernel parameter to be passed during boot, in order to not use an unmeasured EFI runtime services. As a result, the kernel does not expose EFI variables to user-space. SGX Attestation requires these EFI variables to fetch Platform Manifest data.
**Workaround:**
The EFI variables required by SGX are only needed during the SGX provisioning/registration phase. Once this step is completed successfully, access to the EFI variables is no longer required. This means this issue can be worked around by installing the SGX agent without booting to tboot, then rebooting the system to tboot. SGX attestation will then work as expected while booted to tboot.
1. Enable SGX and TXT in the platform BIOS
2. Perform SGX Factory Reset and boot into the “plain” distribution kernel (without tboot or TCB)
3. Install tboot and ISecL components (SGX Agent, Trust Agent and Workload Agent)
4. The SGX Agent installation fetches the SGX Platform Manifest data and caches it
5. Reboot the system into the tboot kernel mode.
6. Verify that TXT measured launch was successful:
txt-stat |grep "TXT measured launch"
7. The SGX and Platform Integrity Attestation use cases should now work as normal.


# ***Intel(R) SecL-DC Version 3.6 GA***

Foundational/WL Use Cases:

- Containerization Deployment of Foundational and Workload Security Use Cases supported and validated with RHEL and Ubuntu OS

- Added support for pyKMIP integration with Workload Security Use Cases

- Additional performance and scalability improvements

- Common Integration Hub to support both Foundational/WL Use Cases and SKC/SGX Attestation Use Cases SKC/ SGX Attestation Use Cases:
- Additional performance and scalability improvements
- SGX Sample Application with quote verification signature added

Known Issues:
- While upgrading components normally requires no installation answer files, the Integration Hub when upgrading from version 3.5 to 3.6 will require an answer file containing “HVS_BASE_URL=https://hvs.server.com:8443/hvs/v1" variable. This is required because the variable and its corresponding configuration setting for the Hub changed between these release versions. No other variables or env files are otherwise required for upgrades.

# ***Intel(R) SecL-DC Version 3.5 GA***

Foundational/WL Use Cases:
- Additional performance and scalability improvements

- Added new filter criteria to the /v2/hosts API. Hosts can now be searched by trust status, and the response data when retrieving host details can now optionally also include the host-status and Trusted state. See the HVS Swaggerdoc for details.

- Host searches will now return data in a consistent order (based on the timestamp when the host was
registered), and can be sorted by ascending or descending order. See the HVS Swaggerdoc for details.

- The CLI command "setup server" has been replaced by "setup update-service-config" across all Foundational
Security services. See the Product Guide for details.

SKC/ SGX Attestation USe Cases:

- Containerization Deployment of SKC and SGX Attestation Use Cases supported

- Added support for pyKMIP integration with SKC Use Case; See Quick Start Guide for deployment details

- Additional performance and scalability improvements

- Added new filter criteria to the /v2/hosts API. See the HVS Swagger docs for details.

- SGX Sample Application and Verifier Enhancements

Known Issues:
- Sometimes the SGX Compute node may becomes inaccessible after the secure key transfer. See guidance in Product Guide.
- Running SHVS Setup Task after changing a config value fails. See guidance in Product Guide.

