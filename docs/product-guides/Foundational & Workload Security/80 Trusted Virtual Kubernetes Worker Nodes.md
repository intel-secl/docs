# Trusted Virtual Kubernetes Worker Nodes

While the existing Platform Integrity Attestation functions support
bare-metal Kubernetes Worker Nodes, using Virtual Machines to host the
Worker Nodes is a common deployment architecture. This feature aims to
help extend the Chain of Trust to protect the integrity of Virtual
Machines, including virtual Kubernetes Worker Nodes. This feature
requires the foundational Platform integrity Attestation feature as a
prerequisite for the bare-metal servers hosting the virtual Worker
Nodes.

???+ note 
    This feature requires a degree of separation between the VM and Kubernetes infrastructure. All physical, bare-metal servers should be virtualization hosts, and all Kubernetes Worker Nodes should be Virtual Machines running on those physical virtualization hosts. Kubernetes clusters should not use a mixture of both virtual and bare-metal Workers. The physical virtualization clusters should not include a mixture of hosts protected by Intel® SecL Platform integrity Attestation and hosts that are not protected. VM trust reports can only be generated for VM instances launched on hosts with Intel® SecL services enabled.
    **Also important to note is that this feature alone will not prevent any VMs from launching**. VMs will still be launched on Untrusted platforms unless additional steps are taken (for example, using OpenStack orchestration integration with Intel® SecL, or using the Workload Confidentiality feature to encrypt the Kubernetes Worker Node VM image). This feature generates VM attestation reports that can be used to audit compliance and extend the Chain of Trust, and relies on other datacenter policies and/or Intel® SecL features to enforce compliance.

When libvirt initiates a VM Start, the Intel® SecL-DC Workload Agent
will create a report for the VM that associates the VM’s trust status
with the trust status of the host launching the VM. This VM report will
be retrievable via the Workload Service, and contains the hardware UUID
of the physical server hosting the VM. This UUID can be correlated to
the Trust Report of that server at the time of VM launch, creating an
audit trail validating that the VM launched on a trusted platform. A new
report is created for every VM Start, which includes actions like VM
migrations, so that each time a VM is launched or moved a new report is
generated ensuring an accurate trust status.

By using Platform Integrity and Data Sovereignty-based orchestration (or
Workload Confidentiality with encrypted worker VMs) for the Virtual
Machines to ensure that the virtual Kubernetes Worker nodes only launch
on trusted hardware, these VM trust reports provide an auditing
capability to extend the Chain of Trust to the virtual Worker Nodes.

Optionally, the Kubernetes Worker Node VM images can be encrypted and
protected as per the Workload Confidentiality feature of Intel® SecL.
This adds a layer of enforcement – rather than simply reporting whether
the VM started on a Trusted platform (and is therefore Trusted),
Workload Confidentiality ensures that the Worker Node VM image can only
be decrypted on compliant platforms.

In both cases (with VM image encryption and without), the VM Trust
Reports are accessed through the Workload Service:

```
GET https://<Workload Service IP or Hostname>:5000/wls/reports?instance_id=<instance ID>
Authorization: Bearer <token>
```

This query will return the latest VM trust report for the provided
Instance ID (the Instance ID is the VM’s ID as it is identified by
Libvirt; in OpenStack this would correspond directly to the OpenStack
Instance ID).

As a best practice, Intel® recommends using an orchestration layer (such
as OpenStack) integrated with Intel® SecL to launch VMs only on Trusted
platforms. See the previous section, “Integration” under the “Platform
Integrity Attestation” feature for details.

As an additional layer of protection, the Kubernetes Worker Node VM
images can be encrypted using the Workload Confidentiality feature. This
adds cryptographic enforcement to the workload orchestration and ensures
instances of the Worker Node images will only be launched on Trusted
platforms.

## Prerequisites
-------------

-   All physical, bare-metal servers should be virtualization hosts.
    Virtualization hosts must be Linux platforms using Libvirt.

-   All Kubernetes Worker Nodes should be Virtual Machines running on
    those physical virtualization hosts.

-   Kubernetes clusters must not use a mixture of both virtual and
    bare-metal Workers.

-   The physical virtualization clusters must not include a mixture of
    hosts protected by Intel® SecL Platform integrity Attestation and
    hosts that are not protected. VM trust reports can only be generated
    for VM instances launched on hosts with Intel® SecL services
    enabled.

-   The Intel® SecL Platform integrity Attestation feature must be used
    to protect all physical virtualization hosts. These platforms must
    all be registered with the Verification Service, must have the Trust
    Agent installed and running, and must be Trusted. See the Platform
    integrity Attestation section for details.

-   In addition to the services required by Platform Integrity
    Attestation, the Workload Agent must be installed on each physical
    virtualization host, and the Workload Service must be installed on
    the management plane.

-   (Optional; recommended) Virtual Machines should be orchestrated
    using an Intel® SecL-supported orchestrator, such as OpenStack. This
    will help launch the VMs only on compliant platforms.

-   (Optional) Virtual Machine images may be encrypted using the
    Workload Confidentiality feature. This adds a layer of cryptographic
    enforcement to the orchestration of virtual worker VMs, ensuring
    that the VMs can only be launched on compliant platforms.

## Workflow
--------

There are no additional steps required to enable this feature; if the
Workload Agent is running on the physical virtualization host, VM trust
reports will automatically be generated at every VM Start. Intel®
strongly recommends using an orchestration integration for the VM
management layer (for example, the provided Integration Hub integration
with OpenStack) to help ensure that the worker node VMs only launch on
Trusted physical hosts. If no orchestration is used, the platform
service provider should ensure that all physical hosts are always in a
Trusted state and take action to ensure Untrusted platforms cannot
launch VMs.

The primary benefit of the Trusted Virtual Kubernetes Worker Node
feature is auditability of the Chain of Trust. By retrieving the VM
Trust Report from the Workload Service for a given Worker Node instance,
auditors can verify that the VM launched on a Trusted platform. The VM
trust report also includes the hardware UUID of the physical host. This
UUID, along with the time that the VM instance was launched, can be used
to pull the correlating physical host trust report from the Verification
Service to provide proof of compliance.

To retrieve a VM trust report from the Workload Service:

```
GET https://<Workload Service IP or Hostname>:5000/wls/reports?instance_id=<instance ID>
Authorization: Bearer <token>
```

This will return the latest report for the specified instance ID.

## Sample VM Trust Report
----------------------

A sample VM Trust Report from the Workload Service is below. The report
is generated by the Workload Agent and signed using the host’s TPM, then
stored in the Workload Service. The report contains some key attributes:

**instance\_id**: This is the ID of the instance. In OpenStack, this
would correlate directly to the Instance ID for the VM.

**image\_id**: This is the ID for the source image used to launch the
instance. In OpenStack, this correlates directly to the Image ID for the
VM.

**host\_hardware\_uuid**: The hardware UUID of the physical host that
started the VM. This attribute identifies which host performed the VM
start and attested the VM. This UUID can be used to query the
Verification Service to retrieve attestations of the host. By
correlating the VM Trust Report with the Host Trust Report, we can
verify that this instance was started on a Trusted platform.

**image\_encrypted**: True or False based on whether the source image
was protected using the Workload Confidentiality feature.

**trusted**: True or False, based on whether the VM instance was started
on a Trusted platform. Because the report is generated at every `vm
start` through Libvirt, a new report will be generated whenever the VM
is turned on or migrated, reflecting the state of the VM and its host at
every opportunity for the state to change.

```xml
<Response xmlns="http://wls.server.com/wls/reports">
    <instance_manifest>
        <instance_info>
            <instance_id>bd06385a-5530-4644-a510-e384b8c3323a</instance_id>
            <host_hardware_uuid>00964993-89c1-e711-906e-00163566263e</host_hardware_uuid>
            <image_id>773e22da-f687-47ca-89e7-5df655c60b7b</image_id>
        </instance_info>
        <image_encrypted>true</image_encrypted>
    </instance_manifest>
    <policy_name>Intel VM Policy</policy_name>
    <results>
        <e>
            <rule>
                <rule_name>EncryptionMatches</rule_name>
                <markers>
                    <e>IMAGE</e>
                </markers>
                <expected>
                    <name>encryption_required</name>
                    <value>true</value>
                </expected>
            </rule>
            <flavor_id>3a3e1ccf-2618-4a0d-8426-fb7acb1ebabc</flavor_id>
            <trusted>true</trusted>
        </e>
    </results>
    <trusted>true</trusted>
    <data>eyJpbnN0YW5jZV9tYW5pZmVzdC…data>
        <hash_alg>SHA-256</hash_alg>
        <cert>-----BEGIN CERTIFICATE-----
…
-----END CERTIFICATE-----</cert>
        <signature>…</signature>
    </Response>

```
