# Intel® Security Libraries - Datacenter Foundational Security

**Product Guide**

**July 2021**

**Revision 4.0**

Notice: This document contains information on products in the design phase of development. The information here is subject to change without notice. Do not finalize a design with this information.

Intel technologies’ features and benefits depend on system configuration and may require enabled hardware, software, or service activation. Learn more at intel.com, or from the OEM or retailer.

No computer system can be absolutely secure. Intel does not assume any liability for lost or stolen data or systems or any damages resulting from such losses.

You may not use or facilitate the use of this document in connection with any infringement or other legal analysis concerning Intel products described herein. You agree to grant Intel a non-exclusive, royalty-free license to any patent claim thereafter drafted which includes subject matter disclosed herein.

No license (express or implied, by estoppel or otherwise) to any intellectual property rights is granted by this document.

The products described may contain design defects or errors known as errata which may cause the product to deviate from published
specifications. Current characterized errata are available on request.

This document contains information on products, services and/or processes in development. All information provided here is subject to
change without notice. Contact your Intel representative to obtain the latest Intel product specifications and roadmaps.

Intel disclaims all express and implied warranties, including without limitation, the implied warranties of merchantability, fitness for a particular purpose, and non-infringement, as well as any warranty arising from course of performance, course of dealing, or usage in trade.

Warning: Altering PC clock or memory frequency and/or voltage may (i) reduce system stability and use life of the system, memory and
processor; (ii) cause the processor and other system components to fail; (iii) cause reductions in system performance; (iv) cause additional heat or other damage; and (v) affect system data integrity. Intel assumes no responsibility that the memory, included if used with altered clock frequencies and/or voltages, will be fit for any particular purpose.
Check with memory manufacturer for warranty and additional details.

Tests document performance of components on a particular test, in specific systems. Differences in hardware, software, or configuration
will affect actual performance. Consult other sources of information to evaluate performance as you consider your purchase. For more complete information about performance and benchmark results, visit http://www.intel.com/performance.

Cost reduction scenarios described are intended as examples of how a given Intel- based product, in the specified circumstances and
configurations, may affect future costs and provide cost savings. Circumstances will vary. Intel does not guarantee any costs or cost
reduction.

Results have been estimated or simulated using internal Intel analysis or architecture simulation or modeling, and provided to you for
informational purposes. Any differences in your system hardware, software or configuration may affect your actual performance.

Intel does not control or audit third-party benchmark data or the web sites referenced in this document. You should visit the referenced web site and confirm whether referenced data are accurate.

Intel is a sponsor and member of the Benchmark XPRT Development Community, and was the major developer of the XPRT family of benchmarks. Principled Technologies is the publisher of the XPRT family of benchmarks. You should consult other information and performance tests to assist you in fully evaluating your contemplated purchases.

Copies of documents which have an order number and are referenced in this document may be obtained by calling 1-800-548-4725 or by visiting w[ww.intel.com/design/literature.htm.](http://www.intel.com/design/literature.htm)

Intel, the Intel logo, Intel TXT, and Xeon are trademarks of Intel Corporation in the U.S. and/or other countries.

\*Other names and brands may be claimed as the property of others.

Copyright © 2020, Intel Corporation. All Rights Reserved.

## Revision History

| Revision  Number | Description                     | Date          |
| ---------------- | ------------------------------- | ------------- |
| 1                | Updated for all GA Failures     | May 2019      |
| 1.5              | Updated for version 1.5 release | July 2019     |
| 1.6 BETA         | Updated for 1.6 BETA release    | November 2019 |
| 1.6              | Updated for version 1.6 release | December 2019 |
| 2.0              | Updated for version 2.0 release | February 2020 |
| 2.1              | Updated for version 2.1 release | April 2020    |
| 2.2              | Updated for version 2.2 release | June 2020     |
| 3.0              | Updated for version 3.0 release | August 2020   |
| 3.1              | Updated for version 3.1 release | October 2020  |
| 3.2              | Updated for version 3.2 release | November 2020 |
| 3.3              | Updated for version 3.3 release | December 2020 |
| 3.3.1            | Updated for version 3.3.1 release | January 2021 |
| 3.4 | Updated for version 3.4 release | February 2021 |
| 3.5 | Updated for version 3.5 release | March 2021 |
| 3.6 | Updated for version 3.6 release | May 2021 |
| 4.0 | Updated for version 4.0 release | July 2021 |


## Table of Contents

[TOC]

Introduction
============

### Overview

Intel Security Libraries for Datacenter is a collection of software applications and development libraries intended to help turn Intel platform security  features into real-world security use cases.

### Trusted Computing

Trusted Computing consists of a set of industry standards defined by the Trusted Computing Group to harden systems and data against attack. These
standards include verifying platform integrity, establishing identity, protection of keys and secrets, and more. One of the functions of Intel Security Libraries is to provide a “Trusted Platform,” using Intel security technologies to add visibility, auditability, and control to server platforms.

#### The Chain of Trust

In a Trusted Computing environment, a key concept is verification of the integrity of the underlying platform. Verifying platform integrity typically means cryptographic measurement and/or verification of firmware and software components. The process by which this measurement and verification takes place affects the overall strength of the assertion that the measured and verified components have not been altered. Intel refers to this process as the “*Chain of Trust*,” whereby at boot time, a sequence of cryptographic measurements and signature verification events happen in a defined order, such that
measurement/verification happens before execution, and each entity responsible for performing a measurement or verification is measured by
another step earlier in the process. Any break in this chain leads to an opportunity for an attacker to modify code and evade detection.

#### Hardware Root of Trust

The *Root of Trust,* the first link in the chain, can be one of several different options. Anything that happens in the boot process before the Root of Trust must be considered to be within the “trust boundary,” signifying components whose trustworthiness cannot be assessed. For this reason, it’s best to use a Root of Trust that starts as early in the system boot process as possible, so that the Chain of Trust during the boot process can cover as much as possible.

Multiple Root of Trust options exist, ranging from firmware to hardware. In general, a hardware Root of Trust will have a smaller “trust
boundary” than a firmware Root of Trust. A hardware Root of Trust will also have the benefit of immutability – where firmware can easily be
flashed and modified, hardware is much more difficult to tamper with.

##### Intel® Trusted Execution Technology (Intel® TXT)

Intel® Trusted Execution Technology is a hardware Root of Trust feature available on Intel® server platforms starting with the Grantley generation. Intel® TXT is enabled in the system BIOS (typically under the Processor \> Advanced tab), and requires Intel® VT-d and Intel VT-x features to be enabled as prerequisites (otherwise the option will be grayed out). Intel® TXT will ship “disabled” by default.

##### Intel® BootGuard (Intel® BtG)

Intel® BootGuard is a hardware Root of Trust feature available on Intel® server platforms starting with the Purley-Refresh generation. Unlike
Intel® TXT, Intel® BtG is configured in platform fuses, not in the system BIOS. Intel® BtG is fused into several “profiles” that determine the behavior of the feature. Intel® BtG supports both “verify” and “measure” profiles; in “verify” profiles, Intel® BtG will verify the signature of the platform Initial Boot Block (IBB). In “measure”profiles, Intel® BtG will hash the IBB and extend that measurement to a TPM PCR. It is recommended that Intel® BtG be fused into the “measure and verify” profile for maximum protection and auditability.

Because the Intel® BtG profile is configured using fuses, the server OEM/ODM will determine the profile used at manufacturing time. Please contact your server vendor to determine what Intel® BtG profiles are available in their product line.

Because Intel® BtG only measures/verifies the integrity of the IBB, it’s important to have an additional technology handle measurements later in the boot process. Intel® TXT can provide this function using tboot to invoke SINIT, and UEFI SecureBoot can alternatively provide similar functionality (note that Linux users should properly configure Shim and use a signed kernel for UEFI SecureBoot).

#### Supported Trusted Boot Options

Intel® SecL-DC supports several options for Trusted Computing, depending
on the features available on the platform.

<img src="Images\tboot options.png" style="zoom:150%;" />



#### Remote Attestation

Trusted computing consists primarily of two activities – measurement, and attestation. Measurement is the act of obtaining cryptographic representations for the system state. Attestation is the act of comparing those cryptographic measurements against expected values to determine whether the system booted into an acceptable state.

Attestation can be performed either locally, on the same host that is to be attested, or remotely, by an external authority. The trusted boot process can optionally include a local attestation involving the evaluation of a TPM-stored Launch Control Policy (LCP). In this case, the host’s TPM will compare the measurements that have been taken so far to a set of expected PCR values stored in the LCP; if there is a mismatch, the boot process is halted entirely.

Intel® SecL utilizes remote attestation, providing a remote Verification Service that maintains a database of expected measurements (or “flavors”), and compares the actual boot-time measurements from any number of hosts against its database to provide an assertion that the host booted into a “trusted” or “untrusted” state. Remote attestation is typically easier to centrally manage (as opposed to creating an LCP for each host and entering the policy into the host’s TPM), does not halt the boot process allowing for easier remediation, and separates the attack surface into separate components that must both be compromised to bypass security controls.

Both local and remote attestation can be used concurrently. However, Intel® SecL, and this document, will focus only on remote attestation. For more information on TPM Launch Control Policies, consult the *Intel Trusted Execution Technology (Intel TXT) Software Development Guide* (https://www.intel.com/content/dam/www/public/us/en/documents/guides/intel-txt-software-development-guide.pdf).

### Intel® Security Libraries for Datacenter Features

#### Platform Integrity

Platform Integrity is the use case enabled by the specific implementation of the Chain of Trust and Remote Attestation concepts. This involves the use of a Root of Trust to begin an unbroken chain of platform measurements at server boot time, with measurements extended to the Trusted Platform Module and compared against expected values to verify the integrity of measured components. This use case is foundational for other Intel® SecL use cases.

#### Data Sovereignty

Data Sovereignty builds on the Platform Integrity use case to allow physical TPMs to be written with Asset Tags containing any number of key/value pairs. This use case is typically used to identify the geographic location of the physical server, but can also be used to identify other attributes. For example, the Asset Tags provided by the Data Sovereignty use case could be used to identify hosts that meet specific compliance requirements and can run controlled workloads.

#### Application Integrity

Added in the Intel® SecL-DC 1.5 release, Application Integrity allows any files and folders on a Linux host system to be included in the Chain of Trust integrity measurements. These measurements are attested by the Verification Service along with the other platform measurements, and are included in determining the host’s overall Trust status. The measurements are performed by a measurement agent called tbootXM, which is built into initrd during Trust Agent installation. Because initrd is included in other Trusted Computing measurements, this allows Intel® SecL-DC to carry the Chain of Trust all the way to the Linux filesystem.

#### Workload Confidentiality for Virtual Machines and Containers

Added in the Intel® SecL-DC 1.6 release, Workload Confidentiality allows virtual machine and container images to be encrypted at rest, with key access tied to platform integrity attestation. Because security attributes contained in the platform integrity attestation report are used to control access to the decryption keys, this feature provides both protection for at-rest data, IP, code, etc in container or virtual machine images, and also enforcement of image-owner-controlled placement policies. When decryption keys are released, they are sealed to the physical TPM of the host that was attested, meaning that only a server that has successfully met the policy requirements for the image can actually gain access.

Workload Confidentiality begins with the Workload Policy Manager (WPM) and a qcow2 or container image that needs to be protected. The WPM is a lightweight application that will request a new key from the Key Broker, use that key to encrypt the image, and generate an Image Flavor. The image owner will then upload the encrypted image to their desired image storage service (for example, OpenStack Glance or a local container registry), and the image ID from the image storage will be uploaded along with the Image Flavor to the Intel® SecL Workload Service. When that image is used to launch a new VM or container, the Workload Agent will intercept the VM or container start and request the decryption key for that image from the Workload Service. The Workload Service will use the image ID and the Image Flavor to find the key transfer URL for the appropriate Key Broker, and will query the Verification Service for the latest Platform Integrity trust attestation report for the host. The Key Broker will use the attestation report to determine whether the host meets the policy requirements for the key transfer, and to verify that the report is signed by a Verification Service known to the Broker. If the report is genuine and meets the policy requirements, the image decryption key is sealed using an asymmetric key from that host’s TPM, and sent back to the Workload Service. The Workload Service then caches the key for 5 minutes (to avoid performance issues for multiple rapid launch requests; note that these keys are still wrapped using a sealing key unique to the hosts TPM, so multiple hosts would require multiple keys even for an identical image) and return the wrapped key to the Workload Agent on the host, which then uses the host TPM to unseal the image decryption key. The key is then used to create a new LUKS volume, and the image is decrypted into this volume.

This functionality means that a physical host must pass policy requirements in order to gain access to the image key, and the image will be encrypted at rest both in image storage and on the compute host.

Beginning with the Intel® SecL-DC version 2.1 release, the Key Broker now supports 3rd-party key managers that are KMIP-compliant. The Key Broker has been updated to use the “gemalto kmip-go” client.

#### Signed Flavors

Added in the Intel® SecL-DC 1.6 release, Flavor signing is an improvement to the existing handling of expected attestation measurements, called “Flavors.” This feature adds the ability to digitally sign Flavors so that the integrity of the expected measurements themselves can be verified when attestations occur. This also means that Flavors can be more securely transferred between different Verification Service instances.

Flavor signing is seamlessly added to the existing Flavor creation process (both importing from a sample host and “manually” creating a Flavor using the POST method to the /v2/flavors resource). When a Flavor is created, the Verification Service will sign it using a signing certificate signed by the Certificate Management Service (this is created during Verification Service setup). Each time that the Verification Service evaluates a Flavor, it will first verify the signature on that Flavor to ensure the integrity of the Flavor contents before it is used to attest the integrity of any host.

#### Trusted Virtual Kubernetes Worker Nodes

Added in the Intel® SecL-DC version 2.1 release, this feature provides a Chain of Trust solution extending to Kubernetes Worker Nodes deployed as Virtual Machines. This feature addresses Kubernetes deployments that use Virtual Machines as Worker Nodes, rather than using bare-metal servers.

When libvirt initiates a VM Start, the Intel® SecL-DC Workload Agent will create a report for the VM that associates the VM’s trust status with the trust status of the host launching the VM. This VM report will be retrievable via the Workload Service, and contains the hardware UUID of the physical server hosting the VM. This UUID can be correlated to the Trust Report of that server at the time of VM launch, creating an audit trail validating that the VM launched on a trusted platform. A new report is created for every VM Start, which includes actions like VM migrations, so that each time a VM is launched or moved a new report is generated ensuring an accurate trust status.

By using Platform Integrity and Data Sovereignty-based orchestration (or Workload Confidentiality with encrypted worker VMs) for the Virtual Machines to ensure that the virtual Kubernetes Worker nodes only launch on trusted hardware, these VM trust reports provide an auditing capability to extend the Chain of Trust to the virtual Worker Nodes.

Intel® Security Libraries Components
====================================

Certificate Management Service
------------------------------

Starting with Intel® SecL-DC 1.6, most non-TPM-related certificates used by Intel® SecL-DC applications will be issued by the new Certificate Management Service. This includes acting as a root CA and issuing TLS certificates for all of the various web services.

## Authentication and Authorization Service

Starting with Intel® SecL-DC 1.6, authentication and authorization for all Intel® SecL applications will be centrally managed by the new Authentication and Authorization Service (AAS). Previously, each application would manage its own users and permissions independently; this change allows authentication and authorization management to be centralized.



## Verification Service

The Verification Service component of Intel® Security Libraries performs the core Platform Integrity and Data Sovereignty functionality by acting as a remote attestation authority.

Platform security technologies like Intel® TXT, Intel® BootGuard, and UEFI SecureBoot extend measurements of platform components (such as the system BIOS/UEFI, OS kernel, etc) to a Trusted Platform module as the server boots. Known-good measurements for each of these components can be directly imported from a sample server. These expected measurements can then be compared against actual measurements from registered servers, allowing the Verification Service to attest to the "trustiness" of the platform, meaning whether the platform booted into a "known-good" state.



## Workload Service

The Workload Service acts as a management service for handling Workload Flavors (Flavors used for Virtual Machines and Containers). In the Intel® SecL-DC 1.6 release, the Workload Service uses Flavors to map decryption key IDs to image IDs. When a launch request for an encrypted workload image is intercepted by the Workload Agent, the Workload Service will handle mapping the image ID to the appropriate key ID and key request URL, and will initiate the key transfer request to the Key Broker.

## Trust Agent

The Trust Agent resides on physical servers and enables both remote attestation and the extended chain of trust capabilities. The Agent maintains ownership of the server's Trusted Platform Module, allowing secure attestation quotes to be sent to the Verification Service. Incorporating the Intel® SecL HostInfo and TpmProvider libraries, the Trust Agent serves to report on platform security capabilities and platform integrity measurements.

The Trust Agent is supported for Windows* Server 2016 Datacenter and Red Hat Enterprise Linux* (RHEL) 8.1 and later.



## Workload Agent

The Workload Agent is the component responsible for handling all of the functions needed for Workload Confidentiality for virtual machines and containers on a physical server. The Workload Agent uses libvirt hooks to identify VM lifecycle events (VM start, stop, hibernate, etc), and intercepts those events to perform needed functions like requesting decryption keys, creation and deletion of encrypted LUKS volumes, using the TPM to unseal decryption keys, etc. The WLA also performs analogous functionality for containers.



## Integration Hub

The Integration Hub acts as a middle-man between the Verification Service and one or more scheduler services (such as OpenStack* Nova), and "pushes" attestation information retrieved from the Verification Service to one or more scheduler services according to an assignment of hosts to specific tenants. In this way, Tenant A can receive attestation information for hosts that belong to Tenant A, but receive no information about hosts belonging to Tenant B.

The Integration Hub serves to disassociate the process of retrieving attestations from actual scheduler queries, so that scheduler services can adhere to best practices and retain better performance at scale. The Integration Hub will regularly query the Intel® SecL Verification Service for SAML attestations for each host. The Integration Hub maintains only the most recent currently valid attestation for each host, and will refresh attestations when they would expire. The Integration Hub will verify the signature of the SAML attestation for each host assigned to a tenant, then parse the attestation status and asset tag information, and then will securely push the parsed key/value pairs to the plugin endpoints enabled.

The Integration Hub features a plugin design for adding new scheduler endpoint types. Currently the Integration Hub supports OpenStack Nova and Kubernetes endpoint plugins. Other integration plugins may be added.



## Workload Policy Manager

The Workload Policy Manager is a Linux command line utility used by an image owner to encrypt VM (qcow2) or container images, and to create an Image Flavor used to provide the encryption key transfer URL during launch requests. The WPM utility will use an existing or request a new key from the Key Broker Service, use that key to encrypt the image, and output the Image Flavor in JSON format. The encrypted image can then be uploaded to the image store of choice (like OpenStack Glance), and the Image Flavor can be uploaded to the Workload Service. The ID of the image on the image storage system is then mapped to the Image Flavor in the WLS; when the image is used to launch a new instance, the WLS will find the Image Flavor associated with that image ID, and use the Image Flavor to determine the key transfer URL.



## Key Broker Service

The Key Broker Service is effectively a policy compliance engine. Its job is to manage key transfer requests, releasing keys only to servers that meet policy requirements. The Key Broker registers one or more SAML signing certificates from any Verification Services that it will trust. When a key transfer request is received, the request includes a trust attestation report signed by the Verification Service. If the signature matches a registered SAML key, the Broker will then look at the actual report to ensure the server requesting the key matches the image policy (currently only overall system trust is supported as a policy requirement). If the report indicates the policy requirements are met, the image decryption key is wrapped using a public key unique to the TPM of the host that was attested in the report, such that only the host that was attested can unseal the decryption key and gain access to the image.



Intel® Security Libraries Binary Installation
======================================

Intel® SecL services can be deployed as direct binary installations (on bare metal or in VMs), or can be deployed as containers.  This section details the binary-based installation of Intel SecL services; the next major section details container-based deployments.

It is recommended to deploy all control-plane services (CMS, AAS, HVS, WLS) as either containers or binaries, and not a mix of the two.

The Trust Agent/Workload Agent, KBS, and WPM can be installed as binaries or deployed as containers regardless of the installation method used for the control plane.

## Building Binary Installers

Intel® Security Libraries is distributed as open source code, and must be compiled into installation binaries before installation.

Instructions and sample scripts for building the Intel® SecL-DC components can be found here:

https://github.com/intel-secl/build-manifest

After the components have been built, the installation binaries can be found in the directories created by the build scripts.

```shell
<servicename>/out/<servicename>.bin
```

In addition, the build script will produce some sample database creation scripts that can be used during installation to configure database requirements (instructions are given in the installation sections):

create_db: `authservice/out/create_db.sh`

install_pgdb: `authservice/out/install_pgdb.sh`

In addition, sample Ansible roles to automatically build and deploy a testbed environment are provided:

https://github.com/intel-secl/utils/tree/v5.0/develop/tools/ansible-role

Also provided are sample API calls organized by workflows for Postman:

https://github.com/intel-secl/utils/tree/v5.0/develop/tools/api-collections

## Hardware Considerations

Intel® SecL-DC supports and uses a variety of Intel security features, but there are some key requirements to consider before beginning an installation. Most important among these is the Root of Trust configuration. This involves deciding what combination of TXT, Boot Guard, tboot, and UEFI Secure Boot to enable on platforms that will be attested using Intel® SecL.

Key points: 

\-   At least one "Static Root of Trust" mechanism must be used (TXT and/or BtG)

\-   For Legacy BIOS systems, tboot must be used (which requires TXT)

\-   For UEFI mode systems, UEFI SecureBoot must be used*

Use the chart below for a guide to acceptable configuration options. .

<img src="Images\hardware_considerations.png" alt="image-20200620161440899" style="zoom:150%;" />


Recommended Service Layout
--------------------------

The Intel® SecL-DC services can be installed in a variety of layouts, partially depending on the use cases desired and the OS of the server(s) to be protected. In general, the Intel® SecL-DC applications can be divided into management services that are deployed on the network on the management plane, and host or node components that must be installed on each protected server.

Management services can typically be deployed anywhere with network access to all of the protected servers. This could be a set of individual VMs per service; containers; or all installed on a single physical or virtual machine.

Node components must be installed on each protected physical server. Typically this is needed for Windows and Linux deployments.

### Platform Integrity 

The most basic use case enabled by Intel® SecL-DC, Platform Integrity requires only the Verification Service and, to protect Windows or Linux hosts, the Trust Agent. This also enables the Application Integrity use case by default for Linux systems.

The Integration Hub may be added to provide integration support for OpenStack or Kubernetes. The Hub is often installed on the same machine as the Verification Service, but optionally can be installed separately.

### Workload Confidentiality

Workload Confidentiality introduces a number of additional services and agents. For a POC environment, all of the management services can be installed on a single machine or VM. This includes:

Certificate Management Service (CMS)

Authorization and Authentication Service (AAS)

Host Verification Service (HVS)

Workload Service (WLS)

Integration Hub (HUB)

Key Broker Service (KBS) with backend key management 

Workload Policy Manager (WPM)

In a production environment, it is strongly suggested that the WPM and KBS be deployed (with their own CMS and AAS) separately for each image owner. For a Cloud Service Provider, this would mean that each customer/tenant who will use the Workload Confidentiality feature would have their own dedicated AAS/CMS/KBS/WPM operated on their own networks, not controlled by the CSP. This is because the Key Broker and WPM are the tools used to define the policies that will allow images to launch, and these policies and their enforcement should remain entirely under the control of the image owner.

The node components must be installed on each protected physical server:

Trust Agent (TA)

Workload Agent (WLA)

<img src="Images\service_layout" alt="image-20200620161752980" style="zoom:150%;" />



## Recommended Service Layout & Architecture - Containerized Deployment with K8s

The containerized deployment makes use of Kubernetes orchestrator for single node and multi node deployments.

A single-node deployment uses Microk8s to deploy the entire control plane in a pod on a single device.  This is best for POC or demo environments, but can also be used when integrating Intel SecL with another application that runs on a virtual machine - the single node deployment can run in the same VM as the integrated application to keep all functions local.

**Single Node:**

![k8s-single-node](.\Images\single-node-ws.jpg)

A multi-node deployment is a more typical Kubernetes architecture, where the Intel SecL management plane is simply deployed as a Pod, with the Intel SecL agents (the WLA and the TA, depending on use case) deployed as a DaemonSet.

**Multi Node:**

![K8s Deployment-fsws](.\images\k8s-deployment-ws.jpg)



**Services Deployments & Agent DaemonSets:**

Every service including databases will be deployed as separate K8s deployment with 1 replica, i.e(1 pod per deployment). Each deployment will be further exposed through k8s service and also will be having corresponding Persistent Volume Claims(PV) for configuration and log directories and mounted on persistent storage. In case of daemonsets/agents, the configuration and log directories will be mounted on respective Baremetal worker nodes.

![](.\images\service-deployment.jpg)

![Each pod consist of only one container with service](.\images\daemonsets-ws.jpg)

For stateful services which requires database like shvs, aas, scs, A separate database deployment will be created for each of such services. The data present on the database deployment will also made to persist on a NFS, through K8s persistent storage mechanism

![database deployment](.\Images\hvs-db.jpg)



**Networking within the Cluster:**

![Networking within cluster](.\images\inter-cluster-communication-ws.jpg)



**Networking Outside the Cluster:**

![Networking outside cluster](.\Images\within-cluster-communication-ws.jpg)

Installing/Configuring the Database
-----------------------------------

The Intel® SecL-DC Authentication and Authorization Service (AAS) requires a Postgresql 11 database. Scripts (install_pgdb.sh, create_db.sh) are provided with the AAS that will automatically add the Postgresql repositories and install/configure a sample database. If this script will not be used, a Postgresql 11 database must be installed by the user before executing the AAS installation.

### Using the Provided Database Installation Script

Install a sample Postgresql 11 database using the install_pgdb.sh script. This script will automatically install the Postgresql database and client packages required.

Add the Postgresql repository:

https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm

Create the `iseclpgdb.env` answer file:

```shell
ISECL_PGDB_IP_INTERFACES=localhost
ISECL_PGDB_PORT=5432
ISECL_PGDB_SAVE_DB_INSTALL_LOG=true
ISECL_PGDB_CERT_DNS=localhost
ISECL_PGDB_CERT_IP=127.0.0.1
```

Note that the values above assume that the database will be accessed locally. If the database server will be external to the Intel® SecL services, change these values to the hostname or FQDN and IP address where the client will access the database server.

Run the following command:

```shell
dnf module disable postgresql -y
```

Execute the installation script:

```shell
./install_pgdb.sh
```

> **Note**: the database installation only needs to be performed once if the same database server will be used for all services that require a database. Only the "create_db" step needs to be repeated if the database server will be shared.

### Provisioning the Database

Each Intel® SecL service that uses a database (the Authentication and Authorization Service, the Verification Service, the Integration Hub, the Workload Service) requires its own schema and access. After installation, the database must be created initialized and tables created. Execute the create_db.sh script to configure the database.

If a single shared database server will be used for each Intel® SecL service (for example, if all management plane services will be installed on a single VM), run the script multiple times, once for each service that requires a database.

If separate database servers will be used (for example, if the management plane services will reside on separate systems and will use their own local database servers), execute the script on each server hosting a database.

```shell
./create_db.sh <database name> <database_username> <database_password>
```

 For example:

```shell
./create_db.sh isecl_hvs_db hvs_db_username hvs_db_password
./create_db.sh isecl_aas_db aas_db_username aas_db_password
./create_db.sh isecl_wls_db wls_db_username wls_db_password
```

Note that the database name, username, and password details for each service must be used in the corresponding installation answer file for that service.

### Database Server TLS Certificate

The database client for Intel® SecL services requires the database TLS certificate to authenticate communication with the database server.

 If the database server for a service is located on the same server that the service will run on, only the path to this certificate is needed. If the provided Postgres scripts are used, the certificate will be located in `/usr/local/pgsql/data/server.crt`

 If the database server will be run separately from the Intel® SecL service(s), the certificate will need to be copied from the database server to the service machine before installing the Intel® SecL services.

The database client for Intel® SecL services will validate that the Subject Alternative Names in the database server’s TLS certificate contain the hostname(s)/IP address(es) that the clients will use to access the database server. If configuring a database without using the provided scripts, ensure that these attributes are present in the database TLS certificate.



## Using NATS with Intel SecL

Intel SecL-DC can utilize a NATS server to manage connectivity between the Host Verification Service and any number of deployed Trust Agent hosts.  This acts as an alternative to communication via REST APIs - in NATS mode, a connection is established with the NATS server, and messages are sent and received over that connection.

The NATS server should be deployed on the control plane and will need network connectivity to other control plane services as well as any Trust Agent hosts.

While NATS is not installed by Intel SecL directly, sample instructions for deploying a NATS server for use with Intel SecL can be found below (this is intended to be a sample only; please consult https://nats.io for official NATS documentation) :

```shell
##Download and install the NATS-server binary (see https://github.com/nats-io/nats-server/releases/latest) 
rpm -i https://github.com/nats-io/nats-server/releases/download/v2.3.0/nats-server-v2.3.0-amd64.rpm

##Install tar and unzip
yum install -y tar unzip 

##Install cfssl and cfssljson (for “Control-Plane Deployment” step #6).
wget https://github.com/cloudflare/cfssl/releases/download/v1.6.0/cfssljson_1.6.0_linux_amd64 -o /usr/bin/cfssljson && chmod +x /usr/bin/cfssljson 

wget https://github.com/cloudflare/cfssl/releases/download/v1.6.0/cfssl_1.6.0_linux_amd64 -o /usr/bin/cfssl && chmod +x /usr/bin/cfssl 
```

Use the following **additional** options in populate-users.env and run populate-users:

```ini
ISECL_INSTALL_COMPONENTS=TA,HVS,AAS,NATS
NATS_CERT_SAN_LIST=<NATS server ip>,<NATS server FQDN>,localhost 
NATS_CERT_COMMON_NAME=NATS TLS Certificate
```

Note that the ISECL_INSTALL_COMPONENTS list should reflect the actual components used in your deployment of Intel SecL, as dictated by the use cases to be enabled.  The important part here is to add the "NATS" element to the list.

To generate a long-lived token for use in environments where the Trust Agent may not be provisioned for an extended period of time beyond the usual lifetime of an authentication token, add the following to populate-users.env:

```ini
CUSTOM_CLAIMS_COMPONENTS=<List of services for which the long-lived installation token will be valid.  Typically this is only the Trust Agent, and so the only component in the list is typically "TA">
CUSTOM_CLAIMS_TOKEN_VALIDITY_SECS=<duration in seconds for the token to last>
CCC_ADMIN_USERNAME=<username for generating long-lived tokens>
CCC_ADMIN_PASSWORD=<password for generating long-lived tokens>
```

When populate-users is run, there will now be an additional "Custom Claims Token For TA:" bearer token.  

When installing the HVS, add the following to hvs.env:

```ini
NATS_SERVERS=nats://<server ip>:4222
```

Alternatively, if the HVS is already installed, add the following in /etc/hvs/config.yml :

```yaml
nats:
  servers:
  - nats://<NATS server IP>:4222
```

Be sure to restart the HVS after changing the configuration.

Download TLS certificates for the NATS server from the CMS:

```shell
export NATS_CERT_COMMON_NAME="NATS TLS Certificate" 
export CMS_ENDPOINT_URL=https://<CMS server ip or hostname>:8445/cms/v1 
export NATS_CERT_SAN_LIST=" <NATS server ip>,<NATS server FQDN>,localhost" 
./download-tls-certs.sh -d ./ -n "$NATS_CERT_COMMON_NAME" -u "$CMS_ENDPOINT_URL" -s "$NATS_CERT_SAN_LIST" -t $BEARER_TOKEN 
```

The download-tls-certs.sh script will conenct to the CMS and will out put two files:

```
server.pem
sslcert-key.pem
```

Create a “server.conf” configuration file for nats-server (be sure the paths to the .pem files are correct):

 

```json
listen: 0.0.0.0:4222 

tls: { 
  cert_file: "./server.pem" 
  key_file: "./sslcert-key.pem" 
} 
```



1. Append the operator/account credentials from the AAS installation to the server.conf (the following can be run if NATS will run on the same machine as the AAS):  

   ```shell
   cat /etc/authservice/nats/server.conf >> server.conf 
   ```

The final server.conf should look like the following:

```json
listen: 0.0.0.0:4222

tls: {
  cert_file: "/<path>/server.pem"
  key_file: "/<path>/sslcert-key.pem"
}

// Operator ISecL-operator
operator: eyJ0eXAiOiJKV1QiLCJhbGciOiJlZDI1NTE5LW5rZXkifQ.eyJleHAiOjE3ODE2MzI0NjUsImp0aSI6IkJWQVZUQkc1M01aMkFaSTVYUjRFNVlPS0xHTk5ZTE40SllYV0U3T1I1M0VQSDJOU0pFSEEiLCJpYXQiOjE2MjM5NTI0NjUsImlzcyI6Ik9DTENLS1UzS0lMWjZaRDRESDNWNTdUSkJESUdKSllMWk1RNEhKUU9DNFJFUVEyVkFUVU01SlA1IiwibmFtZSI6IklTZWNMLW9wZXJhdG9yIiwic3ViIjoiT0NMQ0tLVTNLSUxaNlpENERIM1Y1N1RKQkRJR0pKWUxaTVE0SEpRT0M0UkVRUTJWQVRVTTVKUDUiLCJuYXRzIjp7InR5cGUiOiJvcGVyYXRvciIsInZlcnNpb24iOjJ9fQ.PDlhAwk1cLHpbCCAJhKGKvv36J_NXc2PSsn6i3znmjDYHXG3C_HhO9zxsln9Bd9ViolRw_L10N1QwoMjzCBtBQ

resolver: MEMORY

resolver_preload: {
 // Account ISecL-account
 ADR7WNJ2EEEIYASP5YHDFDW6P3ICBFPXRRJVWU6CLGXOWVDIM7VIOXCM: eyJ0eXAiOiJKV1QiLCJhbGciOiJlZDI1NTE5LW5rZXkifQ.eyJleHAiOjE3ODE2MzI0NjUsImp0aSI6IjdVUlg0M0RSTUxEV0JEVTdMU0dDNTQ0UzZCRFFGVDc1TzZVWUE1QUdYNkxGV0FNWUNOTkEiLCJpYXQiOjE2MjM5NTI0NjUsImlzcyI6Ik9DTENLS1UzS0lMWjZaRDRESDNWNTdUSkJESUdKSllMWk1RNEhKUU9DNFJFUVEyVkFUVU01SlA1IiwibmFtZSI6IklTZWNMLWFjY291bnQiLCJzdWIiOiJBRFI3V05KMkVFRUlZQVNQNVlIREZEVzZQM0lDQkZQWFJSSlZXVTZDTEdYT1dWRElNN1ZJT1hDTSIsIm5hdHMiOnsibGltaXRzIjp7InN1YnMiOi0xLCJkYXRhIjotMSwicGF5bG9hZCI6LTEsImltcG9ydHMiOi0xLCJleHBvcnRzIjotMSwid2lsZGNhcmRzIjp0cnVlLCJjb25uIjotMSwibGVhZiI6LTF9LCJkZWZhdWx0X3Blcm1pc3Npb25zIjp7InB1YiI6e30sInN1YiI6e319LCJ0eXBlIjoiYWNjb3VudCIsInZlcnNpb24iOjJ9fQ.AB6eNFVE7KJspvX7DN-x_-L4mMNhPc-sDk01iOL-hEwYKfeoL9RAcdrOTwQX3CuJHMu-a3m5TpWflg1D4S1MCQ
}
```

Start NATS server:  

```shell
./nats-server -c server.conf 
```



Installing the Certificate Management Service
---------------------------------------------

### Required For

The CMS is REQUIRED for all use cases.

- Platform Integrity with Data Sovereignty and Signed Flavors

- Application Integrity

- Workload Confidentiality (both VMs and Containers)

### Supported Operating Systems

The Intel® Security Libraries Certificate Management Service supports:

Red Hat Enterprise Linux 8.2

Ubuntu 18.04

### Recommended Hardware

- 1 vCPUs

- RAM: 2 GB

- 10 GB

- One network interface with network access to all Intel® SecL-DC services

### Installation

To install the Intel® SecL-DC Certificate Management Service:

1. Copy the Certificate Management Service installation binary to the /root/ directory.

2. Create the `cms.env` installation answer file for an unattended installation:

   ```shell
   AAS_TLS_SAN=<comma-separated list of IPs and hostnames for the AAS>
   AAS_API_URL=https://<Authentication and Authorization Service IP or Hostname>:8444/aas/v1
   SAN_LIST=<Comma-separated list of IP addresses and hostnames for the CMS>,127.0.0.1,localhost 
   ```

   The SAN list will be used to authenticate the Certificate Signing Request from the AAS to the CMS. Only a CSR originating from a host matching the SAN list will be honored. Later, in the AAS `authservice.env` installation answer file, this same SAN list will be provided for the AAS installation. These lists must match, and must be valid for IPs and/or hostnames used by the AAS system. If both the AAS and CMS will be installed on the same system, "127.0.0.1,localhost" may be used. The SAN list variables also accept the wildcards “?” (for single-character wildcards) and "*" (for multiple-character wildcards) to allow address ranges or multiple FQDNs.

   The `AAS_API_URL` represents the URL for the AAS that will exist after the AAS is installed.

   For all configuration options and their descriptions, refer to the Intel® SecL Configuration section on the Certificate Management Service.

3. Execute the installer binary.

   ```shell
   ./cms-v4.0.0.bin
   ```

   When the installation completes, the Certificate Management Service is available. The services can be verified by running cms status from the command line.

   ```shell
cms status
   ```

   After installation is complete, the CMS will output a bearer token to the console. This token will be used with the AAS during installation to authenticate certificate requests to the CMS. If this token expires or otherwise needs to be recreated, use the following command:

   ```shell
cms setup cms_auth_token --force
   ```

   In addition, the SHA384 digest of the CMS TLS certificate will be needed for installation of the remaining Intel® SecL services. The digest can be obtained using the following command:
   
   ```shell
   cms tlscertsha384
   ```
   
   

Installing the Authentication and Authorization Service
-------------------------------------------------------

### Required For

The AAS is REQUIRED for all use cases.

* Platform Integrity with Data Sovereignty and Signed Flavors

* Application Integrity

*  Workload Confidentiality (both VMs and Containers)

### Prerequisites

The following must be completed before installing the Authentication and Authorization Service:

* The Certificate Management Service must be installed and available

* The Authentication and Authorization Service database must be available

### Package Dependencies

The Intel® SecL-DC Authentication and Authorization Service (AAS) requires a Postgresql 11 database. A script (install_pgdb.sh) is provided with the AAS that will automatically add the Postgresql repositories and install/configure a sample database. If this script will not be used, a Postgresql 11 database must be installed by the user before executing the AAS installation.

### Supported Operating Systems

The Intel® Security Libraries Authentication and Authorization Service supports:

Red Hat Enterprise Linux 8.2

Ubuntu 18.04

### Recommended Hardware

* 1 vCPUs

* RAM: 2 GB

* 10 GB

* One network interface with network access to all Intel® SecL-DC services

### Installation

To install the AAS, a bearer token from the CMS is required. This bearer token is output at the end of the CMS installation. However, if a new token is needed, simply use the following command from the CMS command line:

```shell
cms setup cms_auth_token --force
```

 Create the `authservice.env` installation answer file:

```shell
CMS_BASE_URL=https://<CMS IP or hostname>:8445/cms/v1/
CMS_TLS_CERT_SHA384=<CMS TLS certificate sha384>
AAS_DB_HOSTNAME=<IP or hostname of database server>
AAS_DB_PORT=<database port number; default is 5432>
AAS_DB_NAME=<database name>
AAS_DB_USERNAME=<database username>
AAS_DB_PASSWORD=<database password>
AAS_DB_SSLCERTSRC=<path to database TLS certificate; the default location is typically /usr/local/pgsql/data/server.crt>
AAS_ADMIN_USERNAME=<username for AAS administrative user>
AAS_ADMIN_PASSWORD=<password for AAS administrative user>
SAN_LIST=<comma-separated list of IPs and hostnames for the AAS; this should match the value for the AAS_TLS_SAN in the cms.env file from the CMS installation>
BEARER_TOKEN=<bearer token from CMS installation>
```

Execute the AAS installer:

```shell
./authservice-v4.0.0.bin
```

> **Note:** the `AAS_ADMIN` credentials specified in this answer file will have administrator rights for the AAS and can be used to create other users, create new roles, and assign roles to users.

### Creating Users

After installation is complete, a number of roles and user accounts must be generated. Most of these accounts will be service users, used by the various Intel® SecL services to work together. Another set of users will be used for installation permissions, and a final administrative user will be created to provide the initial authentication interface for the actual human user. The administrative user can be used to create additional users with appropriately restricted roles based on organizational needs.

Creating these required users and roles is facilitated by a script that will accept credentials and some configuration settings from an answer file and automate the process.

Create the `populate-users.env` file:

```shell
ISECL_INSTALL_COMPONENTS=KBS,TA,WLS,WPM,IHUB,HVS,WLA,AAS

AAS_API_URL=https://<AAS IP address or hostname>:8444/aas/v1
AAS_ADMIN_USERNAME=<AAS username>
AAS_ADMIN_PASSWORD=<AAS password>

HVS_CERT_SAN_LIST=<comma-separated list of IPs and hostnames for the Host Verification Service>
IH_CERT_SAN_LIST=<comma-separated list of IPs and hostnames for the Integration Hub>
WLS_CERT_SAN_LIST=<comma-separated list of IPs and hostnames for the Workload Service>
KBS_CERT_SAN_LIST=<comma-separated list of IPs and hostnames for the Key Broker Service>
TA_CERT_SAN_LIST=<comma-separated list of IPs and hostnames for the Trust Agent>

HVS_SERVICE_USERNAME=<Username for the HVS service user>
HVS_SERVICE_PASSWORD=<Password for the HVS service user>

IHUB_SERVICE_USERNAME=<Username for the Hub service user>
IHUB_SERVICE_PASSWORD=<Password for the Hub service user>

WPM_SERVICE_USERNAME=<Username for the WPM service user>
WPM_SERVICE_PASSWORD=<Password for the WPM service user>

WLS_SERVICE_USERNAME=<Username for the WLS service user>
WLS_SERVICE_PASSWORD=<Password for the WLS service user>

WLA_SERVICE_USERNAME=<Username for the WLA service user>
WLA_SERVICE_PASSWORD=<Password for the WLA service user>

GLOBAL_ADMIN_USERNAME=<Username for the global Administrator user
GLOBAL_ADMIN_PASSWORD=<Password for the global Administrator user

INSTALL_ADMIN_USERNAME=<Username for the installation user
INSTALL_ADMIN_PASSWORD=<Password for the global installation user
```

>  **Note**: The `ISECL_INSTALL_COMPONENTS` variable is a comma-separated list of the components that will be used in your environment. Not all services are required for every use case. If a given service will not be used in your deployment, simply delete the unnecessary service abbreviation from the `ISECL_INSTALL_COMPONENTS` list, and leave the SAN and credential variables for that service blank.

> **Note**: The SAN list variables each support wildcards( "*" and "?"). In particular, without wildcards the Trust Agent SAN list would need to explicitly list each hostname or IP address for all Trust Agents that will be installed, which is not generally feasible. Using wildcards, domain names and entire IP ranges can be included in the SAN list, which will allow any host matching those ranges to install the relevant service. The SAN list specified here must exactly match the SAN list for the applicable service in that service’s env installation file.

Execute the populate-users script:

```shell
./populate-users
```

> **Note:** The script can be executed with the `–output_json` argument to create the `populate-user.json`.This json output file will contain all of the users created by the script, along with usernames, passwords, and role assignments. This file can be used both as a record of the service and administrator accounts, and can be used as alternative inputs to recreate the same users with the same credentials in the future if needed. Be sure to protect this file if the `–output_json` argument is used.

The script will automatically generate the following users:

Verification Service User

Integration Hub Service User

Workload Policy Manager Service User

Workload Service User Name

Workload Service User

Global Admin User

Installation User

These user accounts will be used during installation of several of the Intel® SecL-DC applications. In general, whenever credentials are required by an installation answer file, the variable name should match the name of the corresponding variable used in the `populate-users.env` file.

The Global Admin user account has all roles for all services. This is a default administrator account that can be used to perform any task, including creating any other users. In general this account is useful for POC installations, but in production it should be used only to create user accounts with more restrictive roles. The administrator credentials should be protected and not shared.

The populate-users script will also output an installation token. This token has all privileges needed for installation of the Intel® SecL services, and uses the credentials provided with the `INSTALLATION_ADMIN_USERNAME` and password. The remaining Intel ® SecL-DC services require this token (set as the `BEARER_TOKEN` variable in the installation env files) to grant the appropriate privileges for installation. By default this token will be valid for two hours; the populate-users script can be rerun with the same `populate-users.env` file to regenerate the token if more time is required, or the `INSTALLATION_ADMIN_USERNAME` and password can be used to generate an authentication token.

 

Installing the Host Verification Service
-----------------------------------

This section details how to install the Intel® SecL-DC services. For instructions on running these services as containers, see the following section.

### Required For

The Host Verification Service is REQUIRED for all use cases.

* Platform Integrity with Data Sovereignty and Signed Flavors

* Application Integrity

* Workload Confidentiality (both VMs and Containers)

### Prerequisites

The following must be completed before installing the Verification Service:

* The Certificate Management Service must be installed and available

* The Authentication and Authorization Service must be installed and available
* The Verification Service database must be available

### Package Dependencies

The Intel® Security Libraries Verification Service requires the following packages and their dependencies:

- logback
- Postgres* client and server 11.6 (server component optional if an external Postgres database is used)
- unzip
- zip
- openssl
- wget
- net-tools
- python3-policycoreutils

If they are not already installed, the Verification Service installer attempts to install these automatically using the package manager. Automatic installation requires access to package repositories (the RHEL subscription repositories, the EPEL repository, or a suitable mirror), which may require an Internet connection. If the packages are to be installed from the package repository, be sure to update the repository package lists before installation.

### Supported Operating Systems

The Intel® Security Libraries Verification Service supports:

Red Hat Enterprise Linux 8.2

Ubuntu 18.04

### Recommended Hardware

* 4 vCPUs

* RAM: 8 GB

* 100 GB

* One network interface with network access to all managed servers

* (Optional) One network interface for Asset Tag provisioning (only required for “pull” tag provisioning; required to provision Asset Tags to VMware ESXi servers).

### Installation

To install the Verification Service, follow these steps:

1. Copy the Verification Service installation binary to the `/root` directory.

2. Create the `hvs.env` installation answer file.

   A sample minimal hvs.env file is provided below. For all configuration options and their descriptions, refer to the Intel® SecL Configuration section  on the Verification Service.

   ```shell
   # Authentication URL and service account credentials 
   AAS_API_URL=https://isecl-aas:8444/aas/v1
   HVS_SERVICE_USERNAME=<username>
   HVS_SERVICE_PASSWORD=<password>
   
   # CMS URL and CMS webserivce TLS hash for server verification 
   CMS_BASE_URL=https://isecl-cms:8445/cms/v1
   CMS_TLS_CERT_SHA384=<digest>
   
   # TLS Configuration
   SAN_LIST=127.0.0.1,192.168.1.1,hvs.server.com #comma-separated list of IP addresses and hostnames for the HVS to be used in the Subject Alternative Names list in the TLS Certificate
   
   # Installation admin bearer token for CSR approval request to CMS 
   BEARER_TOKEN=eyJhbGciOiJSUzM4NCIsImtpZCI6ImE…
   
   # Database 
   HVS_DB_NAME=<database name>
   HVS_DB_USERNAME=<database username>
   HVS_DB_PASSWORD=<database password>
   HVS_DB_SSLCERTSRC=/tmp/dbcert.pem  # Not required if VS_DB_SSLCERT is given
   ```

3. Execute the installer binary.

   ```shell
./hvs-v4.0.0.bin
   ```

   When the installation completes, the Verification Service is available. The services can be verified by running **hvs status** from the Verification Service command line.

   ```shell
   hvs status
   ```
   
   

Installing the Workload Service
-------------------------------

### Required For

The WLS is REQUIRED for the following use cases.

* Workload Confidentiality (both VMs and Containers)

### Prerequisites

The following must be completed before installing the Workload Service:

* The Certificate Management Service must be installed and available

* The Authentication and Authorization Service must be installed and available

* The Verification Service must be installed and available

* The Workload Service database must be available

### Supported Operating Systems

The Intel® Security Libraries Workload Service supports:

Red Hat Enterprise Linux 8.2

Ubuntu 18.04

### Recommended Hardware

### Installation

* Copy the Workload Service installation binary to the `/root` directory.

* Create the `workload-service.env` installation answer file

  ```shell
  WLS_DB_USERNAME=<database username>
  WLS_DB_PASSWORD=<database password>
  WLS_DB_HOSTNAME=<IP or hostname of database server>
  WLS_DB_PORT=<Database port; 5432 by default>
  WLS_DB=<name of the WLS database>
  WLS_DB_SSLCERTSRC=<path to database TLS certificate; the default location is typically /usr/local/pgsql/data/server.crt >
  HVS_URL=https://<Ip address or hostname of the Host verification Service>:8443/hvs/v2/
  WLS_SERVICE_USERNAME=<username for WLS service account>
  WLS_SERVICE_PASSWORD=<password for WLS service account>
  CMS_BASE_URL=https://<IP or hostname to CMS>:8445/cms/v1/
  CMS_TLS_CERT_SHA384=<sha384 of CMS TLS certificate>
  AAS_API_URL=https://<IP or hostname to AAS>:8444/aas/v1/
  SAN_LIST=<comma-separated list of IPs and hostnames for the WLS>
  BEARER_TOKEN=<Installation token from populate-users script>
  ```

* Execute the WLS installer binary:

  ```shell
  ./wls-v4.0.0.bin
  ```
  
  

Installing the Trust Agent for Linux
------------------------------------

### Required For

The Trust Agent for Linux is REQUIRED for all use cases.

* Platform Integrity with Data Sovereignty and Signed Flavors

* Application Integrity

* Workload Confidentiality (both VMs and Containers)

### Package Dependencies

The Trust Agent requires the following packages and their dependencies:

* Tboot (Optional, for TXT-based deployments **without** UEFI SecureBoot only) 

* openssl

* tar

* redhat-lsb

If they are not already installed, the Trust Agent installer attempts to install these automatically using the package manager. Automatic installation requires access to package repositories (the RHEL subscription repositories, the EPEL repository, or a suitable mirror), which may require an Internet connection. If the packages are to be installed from the package repository, be sure to update the repository package lists before installation.

Tboot will not be installed automatically. Instructions for installing and configuring tboot are documented later in this section.

### Supported Operating Systems

The Intel® Security Libraries Trust Agent for Linux supports:

Red Hat Enterprise Linux 8.2

Ubuntu 18.04

### Prerequisites

The following must be completed before installing the Trust Agent:

* Supported server hardware including an Intel® Xeon® processor with Intel Trusted Execution Technology activated in the system BIOS.

* Trusted Platform Module (version 2.0) installed and activated in the system BIOS, with cleared ownership status OR a known TPM ownership secret.

> **Note:** Starting in Intel SecL-DV 4.0, the Trust Agent will now default to using a null TPM owner secret, and does not require ownership permissions except during the provisioning step.   If ownership has already been taken when the Trust Agent will be provisioned, it will be necessary to either provide the ownership secret or clear the TPM ownership before provisioning.

* System must be booted to a tboot boot option OR use UEFI SecureBoot.

* (Provisioning step only) Intel® SecL Verification Service server installed and active.
* (Required for NATS mode only) A NATS server must be configured and available
* (REQUIRED for servers configured with TXT and tboot only) If the server is installed using an LVM, the LVM name must be identical for all Trust Agent systems. The Grub bootloader line that calls the Linux kernel will contain the LVM name of the root volume, and this line with all arguments is part of what is measured in the TXT/Tboot boot process. This will cause the OS Flavor measurements to differ between two otherwise identical hosts if their LVM names are different. Simply using a uniform name for the LVM during OS installation will resolve this possible discrepancy.
* (Optional, REQUIRED for Virtual Machine Confidentiality only):
  * QEMU/KVM must be installed
  * Libvirt must be installed

  

#### Tboot Installation

Tboot is required to build a complete Chain of Trust for Intel® TXT systems that are not using UEFI Secure Boot. Tboot acts to initiate the Intel® TXT SINIT ACM (Authenticated Code Module), which populates several TPM measurements including measurement of the kernel, grub command line, and initrd. Without either tboot or UEFI Secure Boot, the Chain of Trust will be broken because the OS-related components will be neither measured nor signature-verified prior to execution. Because tboot acts to initiate the Intel® TXT SINIT ACM, tboot is only required for platforms using Intel® TXT, and is not required for platforms using another hardware Root of Trust technology like Intel® Boot Guard.

Intel® SecL-DC requires tboot 1.10.1 or greater. This may be a later version of tboot than is available on public software repositories.  

The most current version of tboot can be found here:

https://sourceforge.net/projects/tboot/files/tboot/

Tboot requires configuration of the grub boot loader after installation. To install and configure tboot:

1. Install tboot

   ```shell
   yum install tboot
   ```

   If the package manager does not support a late enough version of tboot, it will need to be compiled from source and installed manually.  Instructions can be found here:

   https://sourceforge.net/p/tboot/wiki/Home/

   Note that the step "copy platform SINIT to /boot" should not be required, as datacenter platforms include the SINIT in the system BIOS package.

2. Ensure that `multiboot2.mod` and `relocator.mod` are available for grub2

   This step may not be necessary for all OS versions, for instance, this step is NA in case of Tboot installation on Ubuntu 18.04. In order to utilize tboot, grub2 requires these two modules from the grub2-efi-x64-modules package to be located in the correct directory (if they're absent, the host will throw a grub error when it tries to boot using tboot).

   These files must be present in this directory:

   ```
   /boot/efi/EFI/redhat/x86_64-efi/multiboot2.mod
   /boot/efi/EFI/redhat/x86_64-efi/relocator.mod
   ```

   If the files are not present in this directory, they can be moved from their installation location:

   ```shell
   cp /usr/lib/grub/x86_64-efi/multiboot2.mod /boot/efi/EFI/redhat/x86_64-efi/
   cp /usr/lib/grub/x86_64-efi/relocator.mod /boot/efi/EFI/redhat/x86_64-efi/
   ```

   Make a backup of your current `grub.cfg` file

   The below examples assume a RedHat OS that has been installed on a platform using UEFI boot mode. The grub path will be slightly different for platforms using a non-RedHat OS.

   ```shell
   cp /boot/efi/EFI/redhat/grub.cfg /boot/efi/EFI/redhat/grub.cfg.bak
   ```

3. Generate a new `grub.cfg` with the tboot boot option

    ```shell
    # For RHEL
    grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg
    
    # For Ubuntu
    grub-mkconfig -o /boot/efi/EFI/redhat/grub.cfg
    ```

4. Update the default boot option

   Ensure that the `GRUB_DEFAULT` value is set to the tboot option.

   a. Update `/etc/default/grub` and set the GRUB_DEFAULT value to `'tboot-1.10.1'`

   ```
   GRUB_DEFAULT='tboot-1.10.1'
   ```

   c. Regenerate `grub.cfg`:

   ```shell
   # For RHEL
   grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg
   
   # For Ubuntu
   grub-mkconfig -o /boot/efi/EFI/redhat/grub.cfg
   ```

5. Reboot the system

   Because measurement happens at system boot, a reboot is needed to boot to the tboot boot option and populate measurements in the TPM.

6. Verify a successful trusted boot with tboot

   Tboot provides the `txt-stat` command to show the tboot log. The first part of the output of this command can be used to verify a successful trusted launch. In the output below, note the “TXT measured launch” and “secrets flag set” at the bottom. Both of these should show "**TRUE**" if the tboot measured launch was successful. If either of these show "**FALSE**" the measured launch has failed. This usually simply indicates that the tboot boot option was not selected during boot.

   If the measured launch was successful, proceed to install the Trust Agent.

   ```
   Intel(r) TXT Configuration Registers:
           STS: 0x0001c091
               senter_done: TRUE
               sexit_done: FALSE
               mem_config_lock: FALSE
               private_open: TRUE
               locality_1_open: TRUE
               locality_2_open: TRUE
           ESTS: 0x00
               txt_reset: FALSE
           E2STS: 0x0000000000000006
               secrets: TRUE
           ERRORCODE: 0x00000000
           DIDVID: 0x00000001b0078086
               vendor_id: 0x8086
               device_id: 0xb007
               revision_id: 0x1
           FSBIF: 0xffffffffffffffff
           QPIIF: 0x000000009d003000
           SINIT.BASE: 0x6fec0000
           SINIT.SIZE: 262144B (0x40000)
           HEAP.BASE: 0x6ff00000
           HEAP.SIZE: 1048576B (0x100000)
           DPR: 0x0000000070000051
               lock: TRUE
               top: 0x70000000
               size: 5MB (5242880B)
           PUBLIC.KEY:
               9c 78 f0 d8 53 de 85 4a 2f 47 76 1c 72 b8 6a 11
               16 4a 66 a9 84 c1 aa d7 92 e3 14 4f b7 1c 2d 11
   
   ***********************************************************
            TXT measured launch: TRUE
            secrets flag set: TRUE
   ***********************************************************
   ```

### NATS Mode vs HTTP Mode

The Trust Agent can operate in either HTTP mode (default) or NATS mode.  This distinction controls how the Agent communicates with the HVS.  In HTTP mode, the TAgent presents a set of API endpoints for the HVS to access via individual TLS requests.  In NATS mode, the Agent and HVS are connected via a persistent session maintained via a NATS server; in this mode, the Trust Agent will not listen on any HTTP ports.

### Installation

Installation of the Trust Agent is split into two major steps: Installation, which covers the creation of system files and folders, and Provisioning, which involves the creation of keys and secrets and links the Trust Agent to a specific Verification Service. Both operations can be performed at the same time using an installation answer file. Without the answer file, the Trust Agent can be installed and left in an un-provisioned state regardless of whether a Verification Service is up and running, until such time as the datacenter administrator is ready to run the provisioning step and link the Trust Agent to a Verification Service.

To install the Trust Agent for Linux:

* Copy the Trust Agent installation binary to the /root/ directory.

* (Optional; required to perform Provisioning and Installation at the same time.) Create the` trustagent.env` answer file in the `/root` directory (for full configuration options, see section 9.2). The minimum configuration options for installation are provided below.

  For Platform Attestation only, provide the following in `trustagent.env` 

   ```shell
  HVS_URL=https://<Verification Service IP or Hostname>:8443/hvs/v2
  PROVISION_ATTESTATION=y
  GRUB_FILE=<path to grub.cfg>
  CURRENT_IP=<Trust Agent IP address>
  CMS_TLS_CERT_SHA384=<CMS TLS digest>
  BEARER_TOKEN=<Installation token from populate-users script>
  AAS_API_URL=https://<AAS IP or Hostname>:8444/aas/v1
  CMS_BASE_URL=https://<CMS IP or Hostname>:8445/cms/v1
  SAN_LIST=<Comma-separated list of IP addresses and hostnames for the TAgent matching the SAN list specified in the populate-users script; may include wildcards>
   ```

  For Workload Confidentiality with VM Encryption, add the following (**in addition to** the basic Platform Attestation sample):

   ```shell
  WLA_SERVICE_USERNAME=<Username for the WLA service user>
  WLA_SERVICE_PASSWORD=<Username for the WLA service user>
  WLS_API_URL=https://<WLS IP address or hostname>:5000/wls/
   ```

  For Workload Confidentiality with Container Encryption, add the following (**in addition to** the basic Platform Attestation sample):

   ```shell
  WLA_SERVICE_USERNAME=<Username for the WLA service user>
  WLA_SERVICE_PASSWORD=<Username for the WLA service user>
  WLS_API_URL=https://<WLS IP address or hostname>:5000/wls/
  REGISTRY_SCHEME_TYPE=https
  ##For the CRI-O container runtime:
  WA_WITH_CONTAINER_SECURITY_CRIO=yes
   ```
  
  For NATS mode, add the following (in addition to the basic Platform Attestation sample and any other optional features):
  
  ```
TA_SERVICE_MODE=outbound 
  NATS_SERVERS=<nats-server-ip>:4222 
TA_HOST_ID=<Any unique identifier for the host; this could be the server FQDN, a UUID, or any other unique identifier>
  ```
  
  Note that the TA_HOST_ID unique identifier will also be the ID used as part of the connection string to reach this Trust Agent host in NATS mode.
  
* Execute the Trust Agent installer and wait for the installation to complete.

  ```shell
  ./trustagent-v4.0.0.bin
  ```

If the `trustagent.env` answer file was provided with the minimum required options, the Trust Agent will be installed and also Provisioned to the Verification Service specified in the answer file.

If no answer file was provided, the Trust Agent will be installed, but will not be Provisioned. TPM-related functionality will not be available from the Trust Agent until the Provisioning step is completed.

The Trust Agent will add a new grub menu entry for application measurement. This new entry will include tboot if the existing grub contains tboot as the default boot option.

> **Note:**  If the Linux Trust Agent is installed without being Provisioned, the Trust Agent process will not actually run until the Provisioning step has been completed.

* Legacy BIOS systems using tboot ONLY) Update the grub boot loader:

  ```shell
  grub2-mkconfig -o /boot/grub2/grub.cfg
  ```

* After Provisioning is completed, the Linux Trust Agent must be rebooted so that the default SOFTWARE Flavor manifest can be measured and extended to the TPM. If the Workload Agent will also be installed on the system (see the next section), wait to reboot the server until after the Workload Agent has been installed, as this modifies the default SOFTWARE Flavor manifest.



Installing the Workload Agent
-----------------------------

### Required For

-   Workload Confidentiality (both VMs and Containers)

### Supported Operating Systems

The Intel® Security Libraries Workload Agent supports Red Hat Enterprise
Linux 8.2

### Prerequisites

The following must be completed before installing the Workload Agent:

-   Intel® SecL Trust Agent installed and active.

-   cryptsetup

-   (REQUIRED for Virtual Machine Confidentiality only):

-   QEMU/KVM must be installed

-   libvirt must be installed


### Installation

* Copy the Workload Agent installation binary to the /root/ directory

* Verify that the `trustagent.env` answer file is present. This file was
  necessary for installing/provisioning the Trust Agent. Note that the
  additional content required for Workload Confidentiality with either
  VM Encryption or Container Encryption must be included in the
  `trustagent.env` file (samples provided in the previous section) for
  use by the Workload Agent.

* Execute the Workload Agent installer binary.

  ```shell
  ./workload-agent-v4.0.0.bin
  ```

* Reboot the server. The Workload Agent populates files that are
  needed for the default `SOFTWARE` Flavor, and a reboot is required for
  those measurements to happen.

  

Trust Agent Provisioning
------------------------

"Provisioning" the Trust Agent involves connecting to a Verification
Service to download the Verification Service PrivacyCA certificate,
create a new Attestation Identity Keypair in the TPM, and verify or
create the TPM Endorsement Certificate and Endorsement Key. The
Verification Service PrivacyCA root certificate is used to sign the EC,
and the EC is used to generate the Attestation Identity Keypair. The AIK
is used by the Verification Service to verify the integrity of quotes
from the host’s TPM.  

Provisioning is the only time that the Trust Agent requires TPM ownership permissions.  If no TPM ownership secret is provided in the trustagent.env file, the Agent will use a null ownership secret to perform the provisioning steps.  If a TPM ownership secret is provided in the trustagent.env answer file, the Agent will attempt to use the specified secret.  If TPM ownership is in a clear state, the Agent will take ownership if a secret is specified.  If the TPM is already "owned," the Agent will try to use the specified secret; if the specified secret does not match the actual ownership password, the provisioning will fail.

Intel recommends using the default "null" ownership secret, as this makes it easy for other applications to also use the Trust Agent, and can prevent the need to clear ownership in the case of a need to re-provision.

Provisioning can be performed separately from installation (meaning you
can install the Trust Agent without Provisioning, and then Provision
later). If the `trustagent.env` answer file is present and has the
required Verification Service information during installation, the Agent
will automatically run the Provisioning steps.

> **Note:**The `trustagent.env` answer file must contain user credentials for a
> user with sufficient privileges. The minimum role required for
> performing provisioning is the "AttestationRegister" role.

> **Note:**If the Linux Trust Agent is installed without being Provisioned, the
> Trust Agent process will not actually run until the Provisioning
> step has been completed.

If the answer file is not present during installation, the Agent can be
provisioned later by adding the `trustagent.env` file and running the
following command:

`tagent setup -f <trustagent.env file
path>`



Trust Agent Registration
------------------------

Registration creates a host record with connectivity details and other
host information in the Verification Service database. This host record
will be used by the Verification Service to retrieve TPM attestation
quotes from the Trust Agent to generate an attestation report.

The Trust Agent can register the host with a Verification Service by
running the following command (the trustagent.env 
answer file must be present in the current working directory):

`tagent setup create-host`

Hosts can also be registered using a REST API request to the
Verification Service:

```http
POST <https://verification.service.com:8443/hvs/v2/hosts>

{
    "host_name": "<hostname of host to be registered>"
    "connection_string": "intel:https://<hostname or IP address>:1443",
    "flavorgroup_names": [],
    "description": "<description>"
}
```

> **Note:**When a new host is registered, the Verification Service will
> automatically attempt to match the host to appropriate Flavors. If
> appropriate Flavors are not found, the host will still be
> registered, but will be in an Untrusted state until/unless
> appropriate Flavors are added to the Verification Service.

<img src="Images\ta_registration.png" alt="image-20200621070159371" style="zoom:150%;" />





Importing the HOST\_UNIQUE Flavor
---------------------------------

RHEL and VMWare ESXi hosts have measured components that are unique to
each host. This means that a special HOST\_UNIQUE flavor part needs to
be imported for each RHEL and ESXi host, in addition to any other OS or
Platform Flavors.

> **Note:**Importing a Flavor requires user credentials for a user with
> sufficient privileges. The minimum role required for creating the
> HOST\_UNIQUE Flavor part is the “host\_unique\_flavor\_creator”
> role. This role can only create HOST\_UNIQUE Flavor parts, and
> cannot create any other Flavors.

On Red Hat Enterprise Linux hosts with the Trust Agent, this can be
performed from the Trust Agent command line (this requires the
trustagent.env answer file to be present in the current working
directory):

```shell
tagent setup create-host-unique-flavor
```

This can also be performed using a REST API (required for VMWare ESXi
hosts):

```http
POST https://verification.service.com:8443/hvs/v2/flavors

{ 
	"connection_string": "<Connection string>",
	"partial_flavor_types": ["HOST_UNIQUE"]
}
```



Installing the Intel® SecL Kubernetes Extensions and Integration Hub
-------------------------------------------------------------------------
Intel® SecL uses Custom Resource Definitions to add the ability to base
orchestration decisions on Intel® SecL security attributes to
Kubernetes. These CRDs allow Kubernetes administrators to configure pods
to require specific security attributes so that the Kubernetes Control Plane
Node will schedule those pods only on Worker Nodes that match the
specified attributes.

Two CRDs are required for integration with Intel® SecL – an extension
for the Control Plane nodes, and a scheduler extension. The extensions are deployed as a Kubernetes
deployment in the `isecl` namespace.

### Deploy Intel® SecL Custom Controller
------------------------------------------

    #Install skopeo to load container image for controller and scheduler from archive
    dnf install -y skopeo

1.  Copy `isecl-k8s-extensions-*.tar.gz` to Kubernetes Control plane machine and extract the contents
    
    ```shell
    #Copy
    scp /<build_path>/binaries/isecl-k8s-extensions-*.tar.gz <user>@<k8s_controller_machine>:/<path>/
    
    #Extract
    tar -xvzf /<path>/isecl-k8s-extensions-*.tar.gz
    cd /<path>/isecl-k8s-extensions/
    ```
    
2.  Create `hostattributes.crd.isecl.intel.com` CRD
    
    ```shell
    #1.14<=k8s_version<=1.16
    kubectl apply -f yamls/crd-1.14.yaml

    #1.16<=k8s_version<=1.18
    kubectl apply -f yamls/crd-1.17.yaml
    ```
    
3. Check whether the CRD is created

   ```shell
   kubectl get crds
   ```

4. Load the `isecl-controller` container image

   ```shell
   cd /<path>/isecl-k8s-extensions/
   skopeo copy oci-archive:<isecl-k8s-controller-*.tar> docker://<docker_private_registry_server>:5000/<imageName>:<tagName>
   ```
   
5. Udate image name as above in controller yaml "/opt/isecl-k8s-extensions/yamls/isecl-controller.yaml"
   ``` shell
      containers:
        - name: isecl-controller
          image: <docker_private_registry_server>:5000/<imageName>:<tagName>
   ```

6. Deploy `isecl-controller`

   ```shell
   kubectl apply -f yamls/isecl-controller.yaml
   ```

7. Check whether the isecl-controller is up and running

   ```shell
   kubectl get deploy -n isecl
   ```

8. Create clusterRoleBinding for ihub to get access to cluster nodes

   ```shell
   kubectl create clusterrolebinding isecl-clusterrole --clusterrole=system:node --user=system:serviceaccount:isecl:isecl
   ```

9. Fetch token required for ihub installation

   ```shell
   kubectl get secrets -n isecl
   
   #The below token will be used for ihub installation to update 'KUBERNETES_TOKEN' in ihub.env when configured with Kubernetes Tenant
   kubectl describe secret default-token-<name> -n isecl
   ```

10. Additional Optional Configurable fields for isecl-controller configuration in `isecl-controller.yaml`

| Field                 | Required   | Type     | Default | Description                                                  |
| --------------------- | ---------- | -------- | ------- | ------------------------------------------------------------ |
| LOG_LEVEL             | `Optional` | `string` | INFO    | Determines the log level                                     |
| LOG_MAX_LENGTH        | `Optional` | `int`    | 1500    | Determines the maximum length of characters in a line in log file |
| TAG_PREFIX            | `Optional` | `string` | isecl   | A custom prefix which can be applied to isecl attributes that are pushed from IH. For example, if the tag-prefix is **isecl.** and **trusted** attribute in CRD becomes **isecl.trusted**. |
| TAINT_UNTRUSTED_NODES | `Optional` | `string` | false   | If set to true. NoExec taint applied to the nodes for which trust status is set to false, Applicable only for HVS based attestation |


### Installing the Intel® SecL Integration Hub
------------------------------------------------------

> **Note:**The Integration Hub is only required to integrate Intel® SecL with
> third-party scheduler services, such as OpenStack Nova or
> Kubernetes. The Hub is not required for usage models that do not
> require Intel® SecL security attributes to be pushed to an
> integration endpoint.

### Required For

The Hub is REQUIRED for the following use cases.

-   Workload Confidentiality (both VMs and Containers)

The Hub is OPTIONAL for the following use cases (used only if
orchestration or other integration support is needed):

- Platform Integrity with Data Sovereignty and Signed Flavors
- Application Integrity

### Deployment Architecture Considerations for the Hub

A separate Hub instance is REQUIRED for each Cloud environment (also referred to as a Hub "tenant").  For example, if a single datacenter will have an OpenStack cluster and also two separate Kubernetes clusters, a total of three Hub instances must be installed, though additional instances of other Intel SecL services are not required (in the same example, only a single Verification Service is required).  Each Hub will manage a single orchestrator environment.  Each Hub instance should be installed on a separate VM or physical server

### Prerequisites

The Intel® Security Libraries Integration Hub can be run as a VM or as a
bare-metal server. The Hub may be installed on the same server (physical
or VM) as the Verification Service.

-   The Verification Service must be installed and available
-   The Authentication and Authorization Service must be installed and
    available
-   The Certificate Management Service must be installed and available
-   (REQUIRED for Kubernetes integration only) The Intel SecL Custom Resource Definitions must be installed and available (see the Integration section for details)

### Package Dependencies

The Intel® SecL Integration Hub requires a number of packages and their
dependencies:

If these are not already installed, the Integration Hub installer
attempts to install these packages automatically using the package
manager. Automatic installation requires access to package repositories
(the RHEL subscription repositories, the EPEL repository, or a suitable
mirror), which may require an Internet connection. If the packages are
to be installed from the package repository, be sure to update your
repository package lists before installation.

### Supported Operating Systems

The Intel Security Libraries Integration Hub supports Red Hat Enterprise
Linux 8.2

### Recommended Hardware

-   1 vCPUs

-   RAM: 2 GB

-   1 GB free space to install the Verification Service services.
    Additional free space is needed for the Integration Hub database and
    logs (database and log space requirements are dependent on the
    number of managed servers).

-   One network interface with network access to the Verification
    Service.

-   One network interface with network access to any integration
    endpoints (for example, OpenStack Nova).

### Installing the Integration Hub

To install the Integration Hub, follow these steps:

1. Copy the API Server certificate of the Kubernetes Controller to machine where Integration Hub will be installed to `/root/` directory
    >  **Note:**  In most  Kubernetes distributions the Kubernetes certificate and key is normally present under `/etc/kubernetes/pki`. However this might differ in case of some specific Kubernetes distributions.

    In ihub.env `KUBERNETES_TOKEN` token can be retrieved from Kubernetes using the following command:

    ```
    kubectl get secrets -n isecl -o jsonpath="{.items[?(@.metadata.annotations['kubernetes\.io/service-account\.name']=='default')].data.token}"|base64 --decode
    ```
    `KUBERNETES_CERT_FILE=/<any_path>/apiserver.crt` in this path can be specified any ex: `/root` which can taken care by IHUB during installation and copied to '/etc/ihub' directory.

2.  Create the `ihub.env` installation answer file. See the
    sample file below.


```shell
# Authentication URL and service account credentials
AAS_API_URL=https://isecl-aas:8444/aas/v1
IHUB_SERVICE_USERNAME=<Username for the Hub service user>
IHUB_SERVICE_PASSWORD=<Password for the Hub service user>

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
KUBERNETES_CERT_FILE=/root/apiserver.crt
KUBERNETES_TOKEN=eyJhbGciOiJSUzI1NiIsImtpZCI6Ik......

# Installation admin bearer token for CSR approval request to CMS - mandatory
BEARER_TOKEN=eyJhbGciOiJSUzM4NCIsImtpZCI6ImE…

```
3. Update the token obtained in  Step 8 of `Deploy Intel® SecL Custom Controller` along with other relevant tenant configuration options in `ihub.env`

4. Copy the Integration Hub installation binary to the `/root`
    directory & execute the installer binary.

   ```shell
   ./ihub-v4.0.0.bin
   ```

5. Copy the `/etc/ihub/ihub_public_key.pem` to Kubernetes Controller machine to `/<path>/secrets/` directory

   ```shell
   #On K8s-Controller machine
   mkdir -p /<path>/secrets
   
   #On IHUB machine, copy
   scp /etc/ihub/ihub_public_key.pem <user>@<k8s_controller_machine>:/<path>/secrets/hvs_ihub_public_key.pem
   ```



After installation, the Hub must be configured to integrate with a Cloud orchestration platform (for example, OpenStack or Kubernetes).  See the Integration section for details.

### Deploy Intel® SecL Extended Scheduler
------------------------------------------------------
1. Install `cfssl` and `cfssljson` on Kubernetes Control Plane

   ```shell
   #Install wget
   dnf install wget -y
   
   #Download cfssl to /usr/local/bin/
   wget -O /usr/local/bin/cfssl http://pkg.cfssl.org/R1.2/cfssl_linux-amd64
   chmod +x /usr/local/bin/cfssl
   
   #Download cfssljson to /usr/local/bin
   wget -O /usr/local/bin/cfssljson http://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
   chmod +x /usr/local/bin/cfssljson
   ```

2. Create TLS key-pair for `isecl-scheduler` service which is signed by Kubernetes `apiserver.crt`

   ```shell
   cd /<path>/isecl-k8s-extensions/
   chmod +x create_k8s_extsched_cert.sh
   
   #Set K8s_CONTROLLER_IP,HOSTNAME
   export CONTROLLER_IP=<k8s_machine_ip>
   export HOSTNAME=<k8s_machine_hostname>
   
   #Create TLS key-pair
   ./create_k8s_extsched_cert.sh -n "K8S Extended Scheduler" -s "$CONTROLLER_IP","$HOSTNAME" -c <k8s_ca_authority_cert> -k <k8s_ca_authority_key>
   ```

   > **Note:**  In most  Kubernetes distributions the Kubernetes certificate and key is normally present under `/etc/kubernetes/pki`. However this might differ in case of some specific Kubernetes distributions.

3. Copy the TLS key-pair generated to `/<path>/secrets/` directory

   ```shell
   cp /<path>/isecl-k8s-extensions/server.key /<path>/secrets/
   cp /<path>/isecl-k8s-extensions/server.crt /<path>/secrets/
   ```

4. Load the `isecl-scheduler` container image

   ```shell
   cd /<path>/isecl-k8s-extensions/
   skopeo copy oci-archive:<isecl-k8s-scheduler-*.tar> docker://<docker_private_registry_server>:5000/<imageName>:<tagName>
   ```
   
5. Update image name as above in scheduler yaml "/opt/isecl-k8s-extensions/yamls/isecl-scheduler.yaml"
   ``` shell
      containers:
        - name: isecl-scheduler
          image: <docker_private_registry_server>:5000/<imageName>:<tagName>
   ```
  
6. Create scheduler-secret for isecl-scheduler

   ```shell
   cd /<path>/
   kubectl create secret generic scheduler-certs --namespace isecl --from-file=secrets
   ```

7. The `isecl-scheduler.yaml` file includes support for both SGX and Workload Security put together. For only working with Workload Security scenarios , the following line needs to be made empty in the yaml file. The scheduler and controller yaml files are located under `/<path>/isecl-k8s-extensions/yamls`

   ```yaml
   - name: SGX_IHUB_PUBLIC_KEY_PATH
     value: ""
   ```

8. Deploy `isecl-scheduler`

   ```shell
   cd /<path>/isecl-k8s-extensions/
   kubectl apply -f yamls/isecl-scheduler.yaml      
   ```

9. Check whether the `isecl-scheduler` is up and running

   ```shell
   kubectl get deploy -n isecl
   ```

10. Additional optional fields for isecl-scheduler configuration in `isecl-scheduler.yaml`

| Field                    | Required   | Type     | Default | Description                                                  |
| ------------------------ | ---------- | -------- | ------- | ------------------------------------------------------------ |
| LOG_LEVEL                | `Optional` | `string` | INFO    | Determines the log level                                     |
| LOG_MAX_LENGTH           | `Optional` | `int`    | 1500    | Determines the maximum length of characters in a line in log file |
| TAG_PREFIX               | `Optional` | `string` | isecl.  | A custom prefix which can be applied to isecl attributes that are pushed from IH. For example, if the tag-prefix is ***\*isecl.\**** and ***\*trusted\**** attribute in CRD becomes ***\*isecl.trusted\****. |
| PORT                     | `Optional` | `int`    | 8888    | ISecl scheduler service port                                 |
| HVS_IHUB_PUBLIC_KEY_PATH | `Required` | `string` |         | Required for IHub with HVS Attestation                       |
| SGX_IHUB_PUBLIC_KEY_PATH | `Required` | `string` |         | Required for IHub with SGX Attestation                       |
| TLS_CERT_PATH            | `Required` | `string` |         | Path of tls certificate signed by kubernetes CA              |
| TLS_KEY_PATH             | `Required` | `string` |         | Path of tls key                                              |



#### Configuring kube-scheduler to establish communication with isecl-scheduler

> **Note:** The below is a sample when using `kubeadm` as the Kubernetes distribution, the scheduler configuration files would be different for any other Kubernetes distributions being used.

1.  Add a mount path to the
    `/etc/kubernetes/manifests/kube-scheduler.yaml` file for the Intel
    SecL scheduler extension:

    ```yaml
    - mountPath: /<path>/isecl-k8s-extensions/
      name: extendedsched
      readOnly: true
    ```

2. Add a volume path to the
   `/etc/kubernetes/manifests/kube-scheduler.yaml` file for the Intel
   SecL scheduler extension:

    ```yaml
    - hostPath:
        path: /<path>/isecl-k8s-extensions/
        type: ""
        name: extendedsched
    ```

3. Add `policy-config-file` path in the
   `/etc/kubernetes/manifests/kube-scheduler.yaml` file under `command` section:

   ```yaml
   - command:
     - kube-scheduler
     - --policy-config-file=/<path>/isecl-k8s-extensions/scheduler-policy.json
     - --bind-address=127.0.0.1
     - --kubeconfig=/etc/kubernetes/scheduler.conf
     - --leader-elect=true
   ```

4. Restart kubelet 

   ```
   systemctl restart kubelet
   ```

   Logs will be appended to older logs in

    /var/log/isecl-k8s-extensions

5. Whenever the CRD's are deleted and restarted for updates, the CRD's
    using the yaml files present under `/opt/isecl-k8s-extensions/yamls/`.
    Kubernetes Version 1.14-1.15 uses `crd-1.14.yaml` and 1.16-1.17 uses
    `crd-1.17.yaml`

    ```shell
    kubectl delete crd hostattributes.crd.isecl.intel.com
    kubectl apply -f /opt/isecl-k8s-extensions/yamls/crd-<version>.yaml
    ```

6. (Optional) Verify that the Intel ® SecL K8s extensions
    have been started:

    To verify the Intel SecL CRDs have been deployed:

    ```shell
    kubectl get -o json hostattributes.crd.isecl.intel.com
    ```

Installing the Key Broker Service
---------------------------------

### Required For

The KBS is REQUIRED for the following use cases:

-   Workload Confidentiality (both VMs and Containers)

### Prerequisites

The following must be completed before installing the Key Broker:

-   The Verification Service must be installed and available
-   The Authentication and Authorization Service must be installed and
    available
-   The Certificate Management Service must be installed and available
-   (Recommended; Required if a 3rd-party Key Management Server will
    be used) A KMIP 2.0-compliant 3rd-party Key management Server must
    be available.
-   The Key Broker will require the KMIP server’s client
        certificate, client key and root ca certificate.
-   The KMIP server's client certificate must contain a Subject Alternative Name that includes the KMIP server's hostname.
-   The Key Broker uses the gemalto kmip-go client to connect to a KMIP
        server
-   The Key Broker has been validated using the pykmip 0.9.1 KMIP
        server as a 3rd-party Key Management Server. While any general
        KMIP 2.0-compliant Key Management Server should work,
        implementation differences among KMIP providers may prevent
        functionality with specific providers.

### Package Dependencies

###   Supported Operating Systems

The Intel® Security Libraries Key Broker Service supports:

Red Hat Enterprise Linux 8.2

Ubuntu 18.04

### Recommended Hardware

### Installation

1.  Copy the Key Broker installation binary to the /root/ directory.

2. Create the installation answer file kbs.env:

   ```shell
   AAS_API_URL=https://<AAS IP or hostname>:8444/aas/v1
   CMS_BASE_URL=https://<CMS IP or hostname>:8445/cms/v1/
   ENDPOINT_URL=https://<KBS IP or hostname>:9443/kbs/v1/
   SAN_LIST=<comma-separated list of hostnames and IP addresses for the Key Broker>
   CMS_TLS_CERT_SHA384=<SHA384 hash of CMS TLS certificate>
   BEARER_TOKEN=<Installation token from populate-users script>
   
   ### OPTIONAL - KMIP configuration only
   KEY_MANAGER=KMIP
   KMIP_SERVER_IP=<IP address of KMIP server>
   KMIP_SERVER_PORT=<Port number of KMIP server>
   KMIP_VERSION=<KMIP protocol version>
   KMIP_USERNAME=<Username of KMIP server>
   KMIP_PASSWORD=<Password of KMIP server>
   ### KMIP_HOSTNAME must be used to provide, KMIP server certificate's SAN(IP/DNS) or valid COMMON NAME. Only FQDN names are allowed.
   KMIP_HOSTNAME=<Hostname of KMIP server>
   ### Retrieve the following certificates and keys from the KMIP server
   KMIP_CLIENT_KEY_PATH=<path>/client_key.pem
   KMIP_ROOT_CERT_PATH=<path>/root_certificate.pem
   KMIP_CLIENT_CERT_PATH=<path>/client_certificate.pem
   ```

3.  Execute the KBS installer.

    ```shell
    ./kbs-4.0.0.bin
    ```

#### Configure the Key Broker to use a KMIP-compliant Key Management Server

The Key Broker must be configured to use a 3rd-party KMIP key manager as part of installation using kbs.env installation variables.


To configure the Key Broker to point to a 3rd-party KMIP-compliant Key Management Server:

1.  Copy the KMIP server’s client certificate, client key and root ca
    certificate to the Key Broker system

2.  Configure the variables in kbs.env for kmip support as below during installation

    ```shell
    KEY_MANAGER=KMIP
    KMIP_SERVER_IP=<IP address of KMIP server>
    KMIP_SERVER_PORT=<Port number of KMIP server>
KMIP_HOSTNAME=<hostname of the KMIP server.  Must match the hostname used in the Subject Alternative Name fort eh KMIP server client certificate.>
    
    ## KMIP_VERSION variable can be used to mention KMIP protocol version.
    ## This is an OPTIONAL field, default value is set to '2.0'. KBS supports KMIP version '1.4' and '2.0'.
    KMIP_VERSION=<KMIP protocol version>
    
    ## KMIP_HOSTNAME can be used to configure TLS config with ServerName.
    ## KMIP server certificate should contain SAN(IP/DNS) or valid COMMON NAME and this value can be provided in KMIP_HOSTNAME. Only FQDN names are allowed.
    ## This is an OPTIONAL field, if KMIP_HOSTNAME is not provided then KMIP_SERVER_IP will be considered as ServerName in TLS configuration.
    KMIP_HOSTNAME=<Hostname of KMIP server>
    
    ## KMIP supports authentication mechanism to authenticate requestor. This is an OPTIONAL field.
    ## This feature can be added to KBS by updating kbs.env with KMIP_USERNAME and KMIP_PASSWORD.
    ## These are OPTIONAL variables. PyKMIP doesn't supports this feature. This feature is validated in Thales cipher trust manager.
    KMIP_USERNAME=<Username of KMIP server>
    KMIP_PASSWORD=<Password of KMIP server>
    
    ### Retrieve the following certificates and keys from the KMIP server
    KMIP_CLIENT_KEY_PATH=<path>/client_key.pem
    KMIP_ROOT_CERT_PATH=<path>/root_certificate.pem
    KMIP_CLIENT_CERT_PATH=<path>/client_certificate.pem
    ```
  ```
  
3.  The KBS configuration can be found in `/etc/kbs/config.yml`, KMIP configuration can be updated in this configuration

    ```shell
    kmip:
      version: "2.0"
      server-ip: "127.0.0.1"
      server-port: "5696"
      hostname: "localhost"
      kmip-username: "<kmip-username>"
      kmip-password: "<kmip-password>"
      client-key-path: "<path>/client-key.pem"
      client-cert-path: "<path>/client-certificate.pem"
      root-cert-path: "<path>/root-certificate.pem"
  ```

4.  Restart the Key Broker for the settings to take effect

    ```shell
    kbs stop
    kbs start
    ```

### Importing Verification Service Certificates

After installation, the Key Broker must import the SAML and PrivacyCA
certificates from any Verification Services it will trust. This provides
the Key Broker a way to ensure that only attestations that come from a
“known” Verification Service. The SAML and PrivacyCA certificates needed
can be found on the Verification Service.

#### Importing a SAML certificate

Display the SAML certificate:

```shell
cat /etc/hvs/certs/trustedca/saml-crt.pem
```

Use the SAML certificate output in the following POST call to the Key
Broker:

```
POST https://<Key Broker IP address or hostname>:9443/kbs/v1/saml-certificates
Content-Type: application/x-pem-file 
-----BEGIN CERTIFICATE-----
MIID9TCCAl2gAwIBAgIBCTANBgkqhkiG9w0BAQwFADBQMQswCQYDVQQGEwJVUzEL
MAkGA1UECBMCU0YxCzAJBgNVBAcTAlNDMQ4wDAYDVQQKEwVJTlRFTDEXMBUGA1UE
AxMOQ01TIFNpZ25pbmcgQ0EwHhcNMTkxMjExMTkzOTU1WhcNMjAxMjExMTkzOTU1
WjAYMRYwFAYDVQQDEw1tdHdpbHNvbi1zYW1sMIIBojANBgkqhkiG9w0BAQEFAAOC
AY8AMIIBigKCAYEArbrDpzR4Ry0MVhSJULHZoiVL020YqtyRH+R2NlVXTpJzqmEA
Ep2utfcP8+mSCT7DLpGBO6KACPCz3pmqj3wZyqZNTrG7IF2Z4Fuf641fPcxA3WVH
3lXz0L5Ep4jOUdfT8kj4hHxHJVJhDsW4J2fds2RGnn8bZG/QbmmGNRfqdxht0zMh
63ik8jBWNWHxYSRbck27FyTj9hDU+z+rFfIdNv1SiQ9FyndgOytK/m7ijoAetkSF
bCsauzUL7DFdRzTmB2GCF/Zd957V51GNpvan6uwqDTL6T4NFX2sqoVduu/WIyTpO
/6D2aA741CR3Bmk9945TSeDKZNz2HkihuE+d8ES68W1t4rvox/Noi74e0k35AqcQ
Q3P0DZpD+XaRapz5CHcOPwOpZ3A/8wN2f+CS2HqDx8FwABkh7l8OdiIWs8+TDQZe
1x4e/50jE/8zMR/tsAy1EXkm3OTOVxih0u18J84x4OT+rHAIcoQ+TOJ40aHrWGHg
kVCfiCUzYYT/W/RBAgMBAAGjEjAQMA4GA1UdDwEB/wQEAwIGwDANBgkqhkiG9w0B
AQwFAAOCAYEAP/ABHdPquBNrMOCU+v7SfMLmIfJymA15mCorMEiKZ1d7oNnoPP0G
pfyRA4TUiyFLCOLi4jIXWuu4Lt6RUz6bnzn8JRWD5ocIJGGxWjOA66xyS3o9iG7G
otOh1pzp5wlwPG7r8ZJ7Q26J+NuHpN1GW5U5Vjww1J9rEKnsKp45QHkG2nXEujdx
YXmKXtEG2gOMVjaLiqromf6VxbdNoKHZGEfqU3H5ymMgqIrnXl3MivA30CymCDLl
rJGRQSwOfzywPCnUOAVptBwLs2kwOtdvnq+BTK3q/dKKoNiFURj/mQ70egquW9ly
TOkYivmKqMZxZlq0//cre4K35aCW3ZArzGgNM8Pk0V/hZp8ZHrWLNAdo4w/Pj1oC
Yq7R0RQ8jQerkewYBfrv3O3e9c22h48fsHnun6F3sbcDjws/sWJIOcrPyqJE26HY
DmIKpvjqc0jI31ndBBwkb+RIBFkz1Ycob9rsW16uVqbjBFDjJ5QKOdXxhqulyboa
JAF53vmU+1jE
-----END CERTIFICATE-----
```

#### Importing a PrivacyCA Certificate

Use OpenSSL to display the PrivacyCA certificate content:

```shell
openssl x509 -in /etc/hvs/certs/trustedca/privacy-ca/privacy-ca-cert.pem
```

Use the PrivacyCA certificate output in the following POST call to the
Key Broker:

```
POST https://<Key Broker IP address or hostname>:9443/kbs/v1/tpm-identity-certificates
Content-Type: application/x-pem-file 
-----BEGIN CERTIFICATE-----
MIIHaDCCBdCgAwIBAgIGAW72eWZ9MA0GCSqGSIb3DQEBCwUAMBsxGTAXBgNVBAMT
EG10d2lsc29uLXBjYS1haWswHhcNMTkxMjExMTkzOTQxWhcNMjkxMjEwMTkzOTQx
WjAbMRkwFwYDVQQDExBtdHdpbHNvbi1wY2EtYWlrMIIBojANBgkqhkiG9w0BAQEF
AAOCAY8AMIIBigKCAYEAmWqBr2YiycZbF/QgFbxTr4YiHtueWBdW0sibtH1QRSbI
KtkbFsmr6J6QiLBaXcF7KVN6DaD0j5sU4cZSttqKwlSUnn07xjWJRP1EcvSaufO1
MarewgBpFQcI2T6aTs1ziV77BoKz0kWteURz1jT1KSwuattxTelpmgucDp98MqW/
uWsliHUVxh51JTE1yn7Vf1QCWz3a+NDH98Lgr5ks337yx3VBK59Dwtsmfsrd5tMn
IuV9Jw0Y2UEdDi004FXI4q64MsMpWA7t5ONRAU+VNU0Y3saXeNBDg9J363imOHIH
haP8ixDhqZ+Xb/TGafgFeEHBkJTv6bWpDqodbWVDbgZloxJzcNgtimQw3RbyrB3C
KijlEo5BQY6bOcdMG7gCq77u/fbOvLb5IXzS8ZDpwuwCQNnBP4UJXwAflO7COG7P
mpj9bTV1OtFiPtYFc4JdGdaf1Pl2zWGeR0c3PIzYQxqvtTVtFX+oRWRsgaEdxKf7
LJx4aIjXwP2s6PIiOSalAgMBAAGjggOwMIIDrDCCAbMGA1UdDgSCAaoEggGmMIIB
ojANBgkqhkiG9w0BAQEFAAOCAY8AMIIBigKCAYEAmWqBr2YiycZbF/QgFbxTr4Yi
HtueWBdW0sibtH1QRSbIKtkbFsmr6J6QiLBaXcF7KVN6DaD0j5sU4cZSttqKwlSU
nn07xjWJRP1EcvSaufO1MarewgBpFQcI2T6aTs1ziV77BoKz0kWteURz1jT1KSwu
attxTelpmgucDp98MqW/uWsliHUVxh51JTE1yn7Vf1QCWz3a+NDH98Lgr5ks337y
x3VBK59Dwtsmfsrd5tMnIuV9Jw0Y2UEdDi004FXI4q64MsMpWA7t5ONRAU+VNU0Y
3saXeNBDg9J363imOHIHhaP8ixDhqZ+Xb/TGafgFeEHBkJTv6bWpDqodbWVDbgZl
oxJzcNgtimQw3RbyrB3CKijlEo5BQY6bOcdMG7gCq77u/fbOvLb5IXzS8ZDpwuwC
QNnBP4UJXwAflO7COG7Pmpj9bTV1OtFiPtYFc4JdGdaf1Pl2zWGeR0c3PIzYQxqv
tTVtFX+oRWRsgaEdxKf7LJx4aIjXwP2s6PIiOSalAgMBAAEwDwYDVR0TAQH/BAUw
AwEB/zCCAeAGA1UdIwSCAdcwggHTgIIBpjCCAaIwDQYJKoZIhvcNAQEBBQADggGP
ADCCAYoCggGBAJlqga9mIsnGWxf0IBW8U6+GIh7bnlgXVtLIm7R9UEUmyCrZGxbJ
q+iekIiwWl3BeylTeg2g9I+bFOHGUrbaisJUlJ59O8Y1iUT9RHL0mrnztTGq3sIA
aRUHCNk+mk7Nc4le+waCs9JFrXlEc9Y09SksLmrbcU3paZoLnA6ffDKlv7lrJYh1
FcYedSUxNcp+1X9UAls92vjQx/fC4K+ZLN9+8sd1QSufQ8LbJn7K3ebTJyLlfScN
GNlBHQ4tNOBVyOKuuDLDKVgO7eTjUQFPlTVNGN7Gl3jQQ4PSd+t4pjhyB4Wj/IsQ
4amfl2/0xmn4BXhBwZCU7+m1qQ6qHW1lQ24GZaMSc3DYLYpkMN0W8qwdwioo5RKO
QUGOmznHTBu4Aqu+7v32zry2+SF80vGQ6cLsAkDZwT+FCV8AH5Tuwjhuz5qY/W01
dTrRYj7WBXOCXRnWn9T5ds1hnkdHNzyM2EMar7U1bRV/qEVkbIGhHcSn+yyceGiI
18D9rOjyIjkmpQIDAQABoR+kHTAbMRkwFwYDVQQDExBtdHdpbHNvbi1wY2EtYWlr
ggYBbvZ5Zn0wDQYJKoZIhvcNAQELBQADggGBAC3PEB8Av0PBJgrJMxzMbuf1FCdD
AUrfYmP81Hs0/v70efviMEF2s3GAyLHD9v+1nNFCQrjcNCar18k45BlcodBEmxKA
DZoioFykRtlha6ByVvuN6wD93KQbKsXPKhUp8X67fLuOcQgfc3BoDRlw/Ha1Ib6X
fliE+rQzLCOgClK7ZdTwl9Ok0VbR7Mbal/xShIqr2WopjBtal9p4RsnIxilTHI+m
qzbV8zvZXYfYtEb3MMMT5EnjIV8O498KKOjxohD2vqaxqItd58pOi6z/q5f4pLHc
DvdsJecJEoWb2bxWQdBgthMjX6AUV/B5G/LTfaPwVbTLdEc+S6Nrobf/TFYV0pvG
OzF3ltYag0fupuYJ991s/JhVwgJhCGq7YourDGkNIWAjt0Z2FWuQKnxWvmResgkS
WTeXt+1HCFSo5WcAZWV8R9FYv7tzFxPY8aoLj82sgrOE4IwRqaA8KMbq3anF4RCk
+D8k6etqMcNHFS8Fj6GlCd80mb4Q3sxuCiBvZw==
-----END CERTIFICATE-----
```



Installing the Workload Policy Manager
--------------------------------------

### Required For

The WPM is REQUIRED for the following use cases.

-   Workload Confidentiality (both VMs and Containers)

### Package Dependencies



### Supported Operating Systems

The Intel® Security Libraries Workload Policy Manager supports:

Red Hat Enterprise Linux 8.2

Ubuntu 18.04

### Recommended Hardware

-   2 vCPUs

-   RAM: 8 GB

-   100 GB

-   One network interface with network access to the Key Broker and
    Workload Service

-   Additional memory and disk space may be required depending on the
    size of images to be encrypted

### Installation

1.  Copy the WPM installer to the `/root` directory

2. Create the `wpm.env` answer file:

   ```shell
   KBS_BASE_URL=https://<IP address or hostname of the KBS>:9443/v1/
   WPM_SERVICE_USERNAME=<WPM_Service username from populate-users script>
   WPM_SERVICE_PASSWORD=<WPM Service password from populate-users script>
   CMS_TLS_CERT_SHA384=<Sha384 hash of the CMS TLS certificate>
   CMS_BASE_URL=https://<IP address or hostname for CMS>:8445/cms/v1/
   AAS_API_URL=https://<Hostname or IP address of the AAS>:8444/aas/v1
   BEARER_TOKEN=<Installation token from populate-users script>
   ```

   For  Container Encryption only, add the following line to the
   wpm.env installation answer file:

   ```shell
   ##For the CRI-O container runtime: 
   WPM_WITH_CONTAINER_SECURITY_CRIO=yes
   ```
   
3.  Execute the WPM installer:

    ```shell
    ./wpm-v4.0.0.bin
    ```



Intel® Security Libraries Container Deployment
==============

## Prerequisites for Containerized Deployment using Kubernetes 

### Supported Operating Systems

- RHEL 8.3
- Ubuntu 18.04

### Kubernetes Distributions

The included Intel SecL container deployment scripts can deploy using the following Kubernetes distributions:

- Single-node: `microk8s` (1.17.17)
  - A "single-node" deployment will deploy all of the Intel SecL services into a single microk8s deployment on a single **bare metal** container host.   Because all resources are deployed onto a single worker, only local resources are required.  This deployment type is best used for POCs and demos due to the requirement of a physical server.
- Multi-node: `kubeadm` (1.17.17)
  - A "multi-node" deployment will deploy the Intel SecL services using kubeadm as a Kubernetes pod, using NFS shared storage.  Any worker nodes that will be attested must be bare metal physical servers; agents will be deployed as privileged DaemonSets.

### Container Runtime

- CRIO-1.17.5 on RHEL 8.3
- CRIO-1.17.5 on Ubuntu 18.04

### Storage

- Single-node:`hostPath` storage for all services and agents
- Multi-node:`NFS` storage for all services, `hostPath ` storage for agents



## Building from Source

###  Prerequisites

The following steps need to be performed on a RHEL 8.3 Build machine (VM/Physical Node)

####  System Tools and Utilities

```shell
# RedHat Enterprise Linux 8.3
dnf install -y git wget tar python3 gcc gcc-c++ zip make yum-utils openssl-devel
dnf install -y https://dl.fedoraproject.org/pub/fedora/linux/releases/32/Everything/x86_64/os/Packages/m/makeself-2.4.0-5.fc32.noarch.rpm
ln -s /usr/bin/python3 /usr/bin/python
ln -s /usr/bin/pip3 /usr/bin/pip

# Ubuntu-18.04
apt update
apt remove -y gcc gcc-7
apt install -y python3-problem-report git wget tar python3 gcc-8 make makeself openssl libssl-dev libgpg-error-dev
cp /usr/bin/gcc-8 /usr/bin/gcc
ln -s /usr/bin/python3 /usr/bin/python
ln -s /usr/bin/pip3 /usr/bin/pip
```

####  Repo tool

```shell
tmpdir=$(mktemp -d)
git clone https://gerrit.googlesource.com/git-repo $tmpdir
install -m 755 $tmpdir/repo /usr/local/bin
rm -rf $tmpdir
```

####  Golang

```shell
wget https://dl.google.com/go/go1.14.4.linux-amd64.tar.gz
tar -xzf go1.14.4.linux-amd64.tar.gz
sudo mv go /usr/local
export GOROOT=/usr/local/go
export PATH=$GOROOT/bin:$PATH
rm -rf go1.14.4.linux-amd64.tar.gz
```

#### Docker

```shell
# RedHat Enterprise Linux-8.3
dnf module enable -y container-tools
dnf install -y yum-utils
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
dnf install -y docker-ce-19.03.13 docker-ce-cli-19.03.13

systemctl enable docker
systemctl start docker

# Ubuntu-18.04
apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
    
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

apt-get update
apt-get install docker-ce=5:19.03.13~3-0~ubuntu-bionic docker-ce-cli=5:19.03.13~3-0~ubuntu-bionic containerd.io

systemctl enable docker
systemctl start docker
```

Apply the below steps **only if** running behind a proxy

```shell
mkdir -p /etc/systemd/system/docker.service.d
touch /etc/systemd/system/docker.service.d/proxy.conf

#Add the below lines in proxy.conf
[Service]
Environment="HTTP_PROXY=<http_proxy>"
Environment="HTTPS_PROXY=<https_proxy>"
Environment="NO_PROXY=<no_proxy>"

systemctl daemon-reload
systemctl restart docker
```

###  Building OCI Container images and K8s Manifests

####  Foundational Security

- Sync the repos

  ```shell
  mkdir -p /root/intel-secl/build/fs && cd /root/intel-secl/build/fs
  repo init -u https://github.com/intel-secl/build-manifest.git -m manifest/fs.xml -b refs/tags/v5.0.0
  repo sync
  ```

- Run the `pre-requisites` setup script

  ```shell
  cd utils/build/foundational-security/
  chmod +x fs-prereq.sh
  ./fs-prereq.sh -s
  ```

- Install skopeo

  ```shell
  # RHEL 8.x
  dnf install -y skopeo
  
  # Ubuntu 18.04
  add-apt-repository ppa:projectatomic/ppa
  apt-get update
  apt-get install skopeo
  ```

- Build

  - Single-node: 

    - A "single-node" deployment will deploy all of the Intel SecL services into a single microk8s deployment on a single **bare metal** container host.   Because all resources are deployed onto a single worker, only local resources are required.  This deployment type is best used for POCs and demos due to the requirement of a physical server
    - Single node cluster with microk8s - `make k8s-aio`

    

  - Multi-node: `kubeadm` (1.17.17)

    - A "multi-node" deployment will deploy the Intel SecL services using kubeadm as a Kubernetes pod, using NFS shared storage.
    - Multi node cluster with kubeadm - `make k8s`

- After the build process is complete, the container images, Kubernetes manifests and deployment scripts can be found in the following folder:

  ```
  /root/intel-secl/build/fs/k8s/
  ```

####  Workload Confidentiality

Workload Confidentiality can be used with either the CRIO container runtime.

#####  Container Confidentiality with CRIO Runtime

- Sync the repos

  ```shell
  mkdir -p /root/intel-secl/build/cc-crio && cd /root/intel-secl/build/cc-crio
  repo init -u https://github.com/intel-secl/build-manifest.git -m manifest/cc-crio.xml -b refs/tags/v5.0.0
  repo sync
  ```

- Run the `pre-requisites` script

  ```shell
  cd utils/build/workload-security
  chmod +x ws-prereq.sh
  ./ws-prereq.sh -c
  ```

- Build

  ```shell
  cd /root/intel-secl/build/cc-crio
  
  #Single node cluster with microk8s
  make k8s-aio
  
  #Multi node cluster with kubeadm
  make k8s
  ```

- The container images, Kubernetes manifests and deployment scripts will output to the following folder:

  ```shell
  /root/intel-secl/build/cc-crio/k8s/
  ```

## Intel® SecL-DC Foundational Security K8s Deployment

This section details deployment of Intel SecL services as a Kubernetes deployment, using a privileged DaemonSet for the Trust Agent and Workload Agent.

###  Pre-requisites

- Install `openssl` on the Kubernetes control node

- Ensure a container registry is running locally or remotely.

  > **Note:** For the single-node `microk8s` deployment, a registry can be brought up by using microk8s addons. More details can be found in the Microk8s documentation. This is not mandatory; if a remote registry already exists, it can be used.
  >
  > **Note:** For multi-node `kubeadm` deployment, a container registry is required.

- Push all container images to container registry.

  ```shell
  # Without TLS enabled
  skopeo copy oci-archive:<oci-image-tar-name> docker://<registry-ip/hostname>:<registry-port>/<image-name>:<image-tag> --dest-tls-verify=false
  
  # With TLS enabled
  skopeo copy oci-archive:<oci-image-tar-name> docker://<registry-ip/hostname>:<registry-port>/image-name>:<image-tag>
  ```

  > **Note:**  For microk8s deployments, when the container registry is enabled locally, the OCI container images need to be copied to the node where registry is enabled and then the above command can be run. This is not required if a remote registry is available.

- On each worker node registered to the Kubernetes control plane, perform the following prerequisite steps:

  #### Foundational Security 

  - `Tboot-1.10.1` or later to be installed for non `SUEFI` servers. [Tboot installation Details](https://github.com/intel-secl/docs/blob/master/product-guides/Foundational%20%26%20Workload%20Security.md#tboot-installation)

  - Only for Ubuntu, run the following command

    ```shell
    $ modprobe msr
    ```

  #### Workload Security 

  ##### Container Confidentiality with CRIO runtime

  * `Tboot-1.10.1` or later to be installed for non `SUEFI` servers. [Tboot installation Details](https://github.com/intel-secl/docs/blob/master/product-guides/Foundational%20%26%20Workload%20Security.md#tboot-installation)
  * Copy `container-runtime` directory to each of the physical servers  

  - Run the `install-prereqs-crio.sh` script on the physical servers from `container-runtime`

  - Reboot the server

  - For Ubuntu worker nodes only, run the following command

    ```shell
    $ modprobe msr
    ```

###  Deploy

####  Single-Node Deployment

A single-node deployment utilizes Microk8s to deploy all services onto a single physical server.  This deployment type is best suited to POCs and demos.

#####  Pre-requisites

######  Setup

- `microk8s` must be installed on a physical server with a supported combination of platform integrity security technologies enabled.

- Copy all manifests and OCI container images as required to the Kubernetes control node

- Ensure a container registry is available

- The Kubernetes cluster admin can configure the existing bare metal worker nodes or register fresh bare metal worker nodes with labels. For example, a label like `node.type: TXT-ENABLED` or `node.type: SUEFI-ENABLED` respectively for `TXT/SUEFI` enabled servers can be used by the cluster admin to distinguish the baremetal worker node and the same label can be used in ISECL Agent pod configuration to schedule on all worker nodes marked with the label. The same label is being used as default in the Kubernetes manifests. This can be edited in `k8s/manifests/ta/daemonset.yml` , `k8s/manifests/wla/daemonset.yml`

  ```shell
  #Label node for TXT
  kubectl label node <node-name> node.type=TXT-ENABLED
  
  #Label node for SUEFI
  kubectl label node <node-name> node.type=SUEFI-ENABLED
  ```

- In a `microk8s` cluster, the `--allow-privileged=true` flag needs to be added to the `kube-apiserver` under `/var/snap/microk8s/current/args/kube-apiserver` and restart `kube-apiserver` with `systemctl restart snap.microk8s.daemon-apiserver` to allow running of privileged containers like `TRUST-AGENT` and `WORKLOAD-AGENT`

- For container confidentiality use cases, ensure a backend KMIP-2.0 compliant server like pykmip is up and running.

######  Manifests

- Update all the Kubernetes manifests with the image names to be pulled from the registry
- The `tolerations` and `node-affinity` for the isecl-scheduler and isecl-controller need to be updated in the respective manifests under the `manifests/k8s-extensions-controller`  and `manifests/k8s-extensions-scheduler` directories to `microk8s.io/cluster` based on the Kubernetes distribution (`kubeadm` and `microk8s,` respectively)

#####  Deploy steps

The bootstrap script will facilitate the deployment of all Intel SecL Foundational Security components based on the use cases to be enabled.

######  Update .env file

```shell
#Kubernetes Distribution - microk8s
K8S_DISTRIBUTION=microk8s
K8S_CONTROL_PLANE_IP=
K8S_CONTROL_PLANE_HOSTNAME=

# cms
CMS_BASE_URL=https://cms-svc.isecl.svc.cluster.local:8445/cms/v1
CMS_SAN_LIST=cms-svc.isecl.svc.cluster.local,<K8s control-plane IP/K8s control-plane Hostname>
CMS_K8S_ENDPOINT_URL=https://<k8s control-plane IP>:30445/cms/v1

# authservice
AAS_API_URL=https://aas-svc.isecl.svc.cluster.local:8444/aas/v1
AAS_API_CLUSTER_ENDPOINT_URL=https://<K8s control-plane IP>:30444/aas/v1
AAS_ADMIN_USERNAME=admin@aas
AAS_ADMIN_PASSWORD=aasAdminPass
AAS_DB_USERNAME=aasdbuser
AAS_DB_PASSWORD=aasdbpassword
AAS_DB_HOSTNAME=aasdb-svc.isecl.svc.cluster.local
AAS_DB_PORT="5432"
AAS_DB_NAME=aasdb
AAS_DB_SSLMODE=verify-full
AAS_SAN_LIST=aas-svc.isecl.svc.cluster.local,<K8s control-plane IP/K8s control-plane Hostname>
#NATS_ACCOUNT_NAME=ISecL-account

# Workload Service
WLS_SERVICE_USERNAME=admin@wls
WLS_SERVICE_PASSWORD=wlsAdminPass
WLS_DB_USERNAME=wlsdbuser
WLS_DB_PASSWORD=wlsdbpassword
WLS_DB_HOSTNAME=wlsdb-svc.isecl.svc.cluster.local
WLS_DB_NAME=wlsdb
WLS_DB_PORT="5432"
WLS_API_URL=https://wls-svc.isecl.svc.cluster.local:5000/wls/v1
WLS_CERT_SAN_LIST=wls-svc.isecl.svc.cluster.local

# Host Verification Service
HVS_SERVICE_USERNAME=admin@hvs
HVS_SERVICE_PASSWORD=hvsAdminPass
HVS_DB_USERNAME=hvsdbuser
HVS_DB_PASSWORD=hvsdbpassword
HVS_DB_HOSTNAME=hvsdb-svc.isecl.svc.cluster.local
HVS_DB_NAME=hvsdb
HVS_CERT_SAN_LIST=hvs-svc.isecl.svc.cluster.local,<K8s control-plane IP/K8s control-plane Hostname>
HVS_DB_PORT="5432"
HVS_URL=https://hvs-svc.isecl.svc.cluster.local:8443/hvs/v2/

#Nats Servers configuration for TA and HVS
#NATS_SERVERS=nats://<K8s control-plane IP/Hostname>:30222

# ihub bootstrap
IHUB_SERVICE_USERNAME=admin@hub
IHUB_SERVICE_PASSWORD=hubAdminPass
IH_CERT_SAN_LIST=ihub-svc.isecl.svc.cluster.local,<K8s control-plane IP/K8s control-plane Hostname>
# For microk8s
# K8S_API_SERVER_CERT=/var/snap/microk8s/current/certs/server.crt
K8S_API_SERVER_CERT=/var/snap/microk8s/current/certs/server.crt
# This is valid for multinode deployment, should be populated once ihub is deployed successfully
IHUB_PUB_KEY_PATH=
HVS_BASE_URL=https://hvs-svc.isecl.svc.cluster.local:8443/hvs/v2

# TrustAgent
# e.g TA_CERT_SAN_LIST=*.example.com,192.168.1.*
TA_CERT_SAN_LIST=
TPM_OWNER_SECRET=

# Workload Agent
WLA_SERVICE_USERNAME=wlauser@wls
WLA_SERVICE_PASSWORD=wlaAdminPass

# KBS
ENDPOINT_URL=https://kbs-svc.isecl.svc.cluster.local:9443/v1
KBS_CERT_SAN_LIST=kbs-svc.isecl.svc.cluster.local,<K8s control-plane IP>,<K8s control-plane Hostname>
KMIP_HOSTNAME=<KMIP IP/Hostname>
KMIP_SERVER_IP=
KMIP_SERVER_PORT=
# Retrieve the following KMIP server’s client certificate, client key and root ca certificate from the KMIP server.
# This key and certificates will be available in KMIP server, /etc/pykmip is the default path copy them to this system manifests/kbs/kmip-secrets path
KMIP_CLIENT_CERT_NAME=client_certificate.pem
KMIP_CLIENT_KEY_NAME=client_key.pem
KMIP_ROOT_CERT_NAME=root_certificate.pem

# ISecl Scheduler
# For microk8s
# K8S_CA_KEY=/var/snap/microk8s/current/certs/ca.key
# K8S_CA_CERT=/var/snap/microk8s/current/certs/ca.crt
K8S_CA_KEY=/var/snap/microk8s/current/certs/ca.key
K8S_CA_CERT=/var/snap/microk8s/current/certs/ca.crt

# populate users.env
ISECL_INSTALL_COMPONENTS="AAS,HVS,WLS,IHUB,KBS,WLA,TA,WPM"

#NATS_CERT_SAN_LIST=
#NATS_TLS_COMMON_NAME=

GLOBAL_ADMIN_USERNAME=
GLOBAL_ADMIN_PASSWORD=

INSTALL_ADMIN_USERNAME=
INSTALL_ADMIN_PASSWORD=

WPM_SERVICE_USERNAME=
WPM_SERVICE_PASSWORD=

CUSTOM_CLAIMS_COMPONENTS=
CCC_ADMIN_USERNAME=
CCC_ADMIN_PASSWORD=
```

######  Run scripts on Kubernetes Controller Node

- These bootstrap scripts are sample scripts to allow for a quick start of Intel SecL services and agents.

```shell
#Pre-reqs.sh
./pre-requisites.sh

#isecl-bootstrap-db-services
#Reference
#./isecl-bootstrap-db-services.sh: option requires an argument -- h
#Usage: ./isecl-bootstrap-db-services.sh [-help/up/purge]
#    -help          print help and exit
#    up        Bootstrap Database Services for Authservice, Workload Service and Host verification Service
#    purge     Delete Database Services for Authservice, Workload Service and Host verification Service

./isecl-bootstrap-db-services.sh up

#isecl-bootstrap
#Reference
#Usage: Usage: ./isecl-bootstrap.sh [-help/up/down/purge]
#    -help                                     Print help and exit
#    up   [all/<agent>/<service>/<usecase>]    Bootstrap ISecL K8s environment for specified agent/service/usecase
#    down [all/<agent>/<service>/<usecase>]    Delete ISecL K8s environment for specified agent/service/usecase [will not delete data, config, logs]
#    purge                                     Delete ISecL K8s environment with data, config, logs [only supported for single node deployments]

#    Available Options for up/down command:
#        agent      Can be one of tagent, wlagent
#        service    Can be one of cms, authservice, hvs, ihub, wls, kbs, isecl-controller, isecl-scheduler
#        usecase    Can be one of foundation-security, workload-security, isecl-orchestration-k8s

./isecl-bootstrap.sh up <all/usecase of choice>
```

- Perform the following steps for isecl-scheduler

```shell
#Copy scheduler-policy.json
cp manifests/k8s-extensions-scheduler/config/scheduler-policy.json /opt/isecl-k8s-extensions/

#Edit the kube-scheduler
vi /var/snap/microk8s/current/args/kube-scheduler

#Add the below line
--policy-config-file=/opt/isecl-k8s-extensions/scheduler-policy.json

#Restart kubelet
systemctl restart snap.microk8s.daemon-kubelet.service
```

####  Multi-Node Deployment

A multi-node deployment will deploy Intel SecL control plane services as a pod on a kubeadm Kubernetes cluster, using a DaemonSet to deploy agent components to worker nodes.  

#####  Pre-requisites

######  Setup

- `kubeadm` must be deployed

- Copy all manifests and OCI container images as required to the Kubernetes control node

- Intel SecL container images must be pushed to a container registry

- The Kubernetes cluster admin can configure the existing bare metal worker nodes or register fresh bare metal worker nodes with labels. For example, a label like `node.type: TXT-ENABLED` or `node.type: SUEFI-ENABLED` respectively for `TXT/SUEFI` enabled servers can be used by the cluster admin to distinguish the baremetal worker node and the same label can be used in ISECL Agent pod configuration to schedule on all worker nodes marked with the label. The same label is being used as default in the K8s manifests. This can be edited in `k8s/manifests/ta/daemonset.yml` , `k8s/manifests/wla/daemonset.yml`

  ```shell
  #Label node for TXT
  kubectl label node <node-name> node.type=TXT-ENABLED
  
  #Label node for SUEFI
  kubectl label node <node-name> node.type=SUEFI-ENABLED
  ```

- The `NFS` storage class is used in Kubernetes for data persistence. An NFS server with appropriate directory structure and permissions is required. Intel recommends creation of a separate user ID with permissions for all Intel SecL directories. Below are some samples for reference

  - Snapshot showing directory structure for which user needs to create on NFS volumes manually or using custom scripts.

  ![NFS structure](./Images/nfs-fsws-structure.jpg)

  - Snapshot showing ownership and permissions for directories for which user needs to manually grant the ownership.

  ![NFS Directory Permissions FS/WS Usecase](./Images/nfs-fsws-permissions.jpg)

  - Snapshot for configuring PV and PVC , user need to provide the NFS server IP or hostname and paths for each of the service directories. Sample manifest for creating `config-pv` for cms service

    ```yaml
    ---
    apiVersion: v1
    kind: PersistentVolume
    metadata:
        name: cms-config-pv
    spec:
        capacity:
        storage: 128Mi
        volumeMode: Filesystem
        accessModes:
        - ReadWriteMany
        persistentVolumeReclaimPolicy: Retain
        storageClassName: nfs
        nfs:
        path: /<NFS-vol-base-path>/isecl/cms/config
        server: <NFS Server IP/Hostname>
    ```

  - Sample manifest for creating config-pvc for cms service

    ```yaml
      ---
      apiVersion: v1
      kind: PersistentVolumeClaim
      metadata:
          name: cms-config-pvc
          namespace: isecl
      spec:
          storageClassName: nfs
          accessModes:
          - ReadWriteMany
          resources:
          requests:
              storage: 128Mi
    ```

  > **Note:** The user id specified in security context in `deployment.yml` for a given service and owner of the service related directories in NFS must be same

- For container confidentiality use cases, a backend KMIP-2.0 compliant server like pykmip must be availabe

######  Manifests

- Update Kubernetes manifests with the image names to be pulled from the registry
- The `tolerations` and `node-affinity` for the isecl-scheduler and isecl-controller nees to be updated in the respective manifests under the `manifests/k8s-extensions-controller`  and `manifests/k8s-extensions-scheduler` directories to `node-role.kubernetes.io/master`
- All NFS PV yaml files must be updated with the  `path: /<NFS-vol-path>`  and `server: <NFS Server IP/Hostname>` under each service manifest file for `config`, `logs` , `db-data`

#####  Deploy steps

######  Update .env file

```shell
#Kubernetes Distribution - kubeadm
K8S_DISTRIBUTION=kubeadm
K8S_CONTROL_PLANE_IP=
K8S_CONTROL_PLANE_HOSTNAME=

# cms
CMS_BASE_URL=https://cms-svc.isecl.svc.cluster.local:8445/cms/v1
CMS_SAN_LIST=cms-svc.isecl.svc.cluster.local,<K8s control-plane IP/K8s control-plane Hostname>
CMS_K8S_ENDPOINT_URL=https://<k8s control-plane IP>:30445/cms/v1

# authservice
AAS_API_URL=https://aas-svc.isecl.svc.cluster.local:8444/aas/v1
AAS_API_CLUSTER_ENDPOINT_URL=https://<K8s control-plane IP>:30444/aas/v1
AAS_ADMIN_USERNAME=admin@aas
AAS_ADMIN_PASSWORD=aasAdminPass
AAS_DB_USERNAME=aasdbuser
AAS_DB_PASSWORD=aasdbpassword
AAS_DB_HOSTNAME=aasdb-svc.isecl.svc.cluster.local
AAS_DB_PORT="5432"
AAS_DB_NAME=aasdb
AAS_DB_SSLMODE=verify-full
AAS_SAN_LIST=aas-svc.isecl.svc.cluster.local,<K8s control-plane IP/K8s control-plane Hostname>
#NATS_ACCOUNT_NAME=ISecL-account

# Workload Service
WLS_SERVICE_USERNAME=admin@wls
WLS_SERVICE_PASSWORD=wlsAdminPass
WLS_DB_USERNAME=wlsdbuser
WLS_DB_PASSWORD=wlsdbpassword
WLS_DB_HOSTNAME=wlsdb-svc.isecl.svc.cluster.local
WLS_DB_NAME=wlsdb
WLS_DB_PORT="5432"
WLS_API_URL=https://wls-svc.isecl.svc.cluster.local:5000/wls/v1
WLS_CERT_SAN_LIST=wls-svc.isecl.svc.cluster.local

# Host Verification Service
HVS_SERVICE_USERNAME=admin@hvs
HVS_SERVICE_PASSWORD=hvsAdminPass
HVS_DB_USERNAME=hvsdbuser
HVS_DB_PASSWORD=hvsdbpassword
HVS_DB_HOSTNAME=hvsdb-svc.isecl.svc.cluster.local
HVS_DB_NAME=hvsdb
HVS_CERT_SAN_LIST=hvs-svc.isecl.svc.cluster.local,<K8s control-plane IP/K8s control-plane Hostname>
HVS_DB_PORT="5432"
HVS_URL=https://hvs-svc.isecl.svc.cluster.local:8443/hvs/v2/

#Nats Servers configuration for TA and HVS
#NATS_SERVERS=nats://<K8s control-plane IP/Hostname>:30222

# ihub bootstrap
IHUB_SERVICE_USERNAME=admin@hub
IHUB_SERVICE_PASSWORD=hubAdminPass
IH_CERT_SAN_LIST=ihub-svc.isecl.svc.cluster.local,<K8s control-plane IP/K8s control-plane Hostname>
# For Kubeadm
# K8S_API_SERVER_CERT=/etc/kubernetes/pki/apiserver.crt
K8S_API_SERVER_CERT=/etc/kubernetes/pki/apiserver.crt
# This is valid for multinode deployment, should be populated once ihub is deployed successfully
IHUB_PUB_KEY_PATH=
HVS_BASE_URL=https://hvs-svc.isecl.svc.cluster.local:8443/hvs/v2

# TrustAgent
# e.g TA_CERT_SAN_LIST=*.example.com,192.168.1.*
TA_CERT_SAN_LIST=
TPM_OWNER_SECRET=

# Workload Agent
WLA_SERVICE_USERNAME=wlauser@wls
WLA_SERVICE_PASSWORD=wlaAdminPass

# KBS
ENDPOINT_URL=https://kbs-svc.isecl.svc.cluster.local:9443/v1
KBS_CERT_SAN_LIST=kbs-svc.isecl.svc.cluster.local,<K8s control-plane IP>,<K8s control-plane Hostname>
KMIP_HOSTNAME=<KMIP IP/Hostname>
KMIP_SERVER_IP=
KMIP_SERVER_PORT=
# Retrieve the following KMIP server’s client certificate, client key and root ca certificate from the KMIP server.
# This key and certificates will be available in KMIP server, /etc/pykmip is the default path copy them to this system manifests/kbs/kmip-secrets path
KMIP_CLIENT_CERT_NAME=client_certificate.pem
KMIP_CLIENT_KEY_NAME=client_key.pem
KMIP_ROOT_CERT_NAME=root_certificate.pem

# ISecl Scheduler
# For Kubeadm
# K8S_CA_KEY=/etc/kubernetes/pki/ca.key
# K8S_CA_CERT=/etc/kubernetes/pki/ca.crt
K8S_CA_KEY=/etc/kubernetes/pki/ca.key
K8S_CA_CERT=/etc/kubernetes/pki/ca.crt

# populate users.env
ISECL_INSTALL_COMPONENTS="AAS,HVS,WLS,IHUB,KBS,WLA,TA,WPM"

#NATS_CERT_SAN_LIST=
#NATS_TLS_COMMON_NAME=

GLOBAL_ADMIN_USERNAME=
GLOBAL_ADMIN_PASSWORD=

INSTALL_ADMIN_USERNAME=
INSTALL_ADMIN_PASSWORD=

WPM_SERVICE_USERNAME=
WPM_SERVICE_PASSWORD=

CUSTOM_CLAIMS_COMPONENTS=
CCC_ADMIN_USERNAME=
CCC_ADMIN_PASSWORD=
```

######  Run scripts on Kubernetes Controller Node

- The following bootstrap scripts are sample scripts to allow for a quick start of Intel SecL services and agents.

```shell
#Pre-reqs.sh
./pre-requisites.sh

#isecl-bootstrap-db-services
#Reference
#./isecl-bootstrap-db-services.sh: option requires an argument -- h
#Usage: ./isecl-bootstrap-db-services.sh [-help/up/purge]
#    -help          print help and exit
#    up        Bootstrap Database Services for Authservice, Workload Service and Host verification Service
#    purge     Delete Database Services for Authservice, Workload Service and Host verification Service

./isecl-bootstrap-db-services.sh up

#isecl-bootstrap
#Reference
#Usage: Usage: ./isecl-bootstrap.sh [-help/up/down/purge]
#    -help                                     Print help and exit
#    up   [all/<agent>/<service>/<usecase>]    Bootstrap ISecL K8s environment for specified agent/service/usecase
#    down [all/<agent>/<service>/<usecase>]    Delete ISecL K8s environment for specified agent/service/usecase [will not delete data, config, logs]
#    purge                                     Delete ISecL K8s environment with data, config, logs [only supported for single node deployments]

#    Available Options for up/down command:
#        agent      Can be one of tagent, wlagent
#        service    Can be one of cms, authservice, hvs, ihub, wls, kbs, isecl-controller, isecl-scheduler
#        usecase    Can be one of foundation-security, workload-security, isecl-orchestration-k8s

./isecl-bootstrap.sh up <all/usecase of choice>
```

- Copy the `ihub_public_key.pem` from NFS path -`<mnt>/isecl/ihub/config/ihub_public_key.pem ` to K8s Master
- Update the `isecl-k8s.env` for `IHUB_PUB_KEY_PATH` 
- Bring up the `isecl-k8s-scheduler` 

```
./isecl-bootstrap.sh up isecl-scheduler
```

- Create and update `scheduler-policy.json` path

```
mkdir -p /opt/isecl-k8s-extensions
cp manifests/k8s-extensions-scheduler/config/scheduler-policy.json /opt/isecl-k8s-extensions
```

- Configure kube-scheduler to establish communication with isecl-scheduler. Add `scheduler-policy.json` under kube-scheduler section, `mountPath` under container section and `hostPath` under volumes section in` /etc/kubernetes/manifests/kube-scheduler.yaml` as mentioned below

```yaml
spec:
  containers:
  - command:
    - kube-scheduler
    - --policy-config-file=/opt/isecl-k8s-extensions/scheduler-policy.json
containers:
    volumeMounts:
    - mountPath: /opt/isecl-k8s-extensions/
      name: extendedsched
      readOnly: true
volumes:
  - hostPath:
      path: /opt/isecl-k8s-extensions/
      type:
    name: extendedsched
```

> **Note:** Make sure to use proper indentation and don't delete existing `mountPath` and `hostPath` sections in `kube-scheduler.yaml`

- Restart `kubelet` which restart all the k8s services including kube-scheduler

```shell
systemctl restart kubelet
```

##  Default Service and Agent Mount Paths

###  Single Node Deployments

Single node Deployments use `hostPath` mounting pod(container) files directly on the host.

```yaml
#Certificate-Management-Service
Config: /etc/cms
Logs: /var/log/cms

#Authentication Authorization Service
Config: /etc/authservice
Logs: /var/log/authservice
Pg-data: /usr/local/kube/data/authservice/pgdata

#Host Attestation Service
Config: /etc/hvs
Logs: /var/log/hvs
Pg-data: /usr/local/kube/data/hvs

#Integration-Hub
Config: /etc/ihub
Log: /var/log/ihub

#Workload Service
Config: /etc/workload-service
Logs: /var/log/workload-service
Pg-data: /usr/local/kube/data/workload-service

#Key-Broker-Service
Config: /etc/kbs
Log: /var/log/kbs
Opt: /opt/kbs

#Trust Agent:
Config: /opt/trustagent/configuration
Logs:  /var/log/trustagent/
tpmrm: /dev/tpmrm0
txt-stat: /usr/sbin/txt-stat
ta-hostname-path: /etc/hostname
ta-hosts-path: /etc/hosts

#Workload Agent:
Config: /etc/workload-agent/
Logs: /var/log/workload-agent
TA Config: /opt/trustagent/configuration
WLA-Socket: /var/run/workload-agent
```

###  Multi Node Deployments

Multi node Deployments use Kubernetes persistent volume and persistent volume claims for mounting pod(container) files on NFS volumes for all services.  Agents will use `hostPath` for persistent storage. F

```yaml
#Certificate-Management-Service
Config: <NFS-vol-base-path>/isecl/cms/config
Logs: <NFS-vol-base-path>/isecl/cms/logs

#Authentication Authorization Service
Config: <NFS-vol-base-path>/isecl/aas/config
Logs: <NFS-vol-base-path>/isecl/aas/logs
Pg-data: <NFS-vol-base-path>/isecl/aas/db

#Host Attestation Service
Config: <NFS-vol-base-path>/isecl/hvs/config
Logs: <NFS-vol-base-path>/isecl/hvs/logs
Pg-data: <NFS-vol-base-path>/usr/local/kube/data/hvs

#Integration-Hub
Config: <NFS-vol-base-path>/isecl/ihub/config
Log: <NFS-vol-base-path>/isecl/ihub/logs

#Workload Service
Config: <NFS-vol-base-path>/isecl/wls/config
Logs: <NFS-vol-base-path>/isecl/wls/log
Pg-data: <NFS-vol-base-path>/usr/local/kube/data/wls

#Key-Broker-Service
Config: <NFS-vol-base-path>/isecl/kbs/config
Log: <NFS-vol-base-path>/isecl/kbs/logs
Opt: <NFS-vol-base-path>/isecl/kbs/kbs/opt

#Trust Agent:
Config: /opt/trustagent/configuration
Logs:  /var/log/trustagent/
tpmrm: /dev/tpmrm0
txt-stat: /usr/sbin/txt-stat
ta-hostname-path: /etc/hostname
ta-hosts-path: /etc/hosts

#Workload Agent:
Config: /etc/workload-agent/
Logs: /var/log/workload-agent
WLA-Socket: /var/run/workload-agent
```

##  Default Service Ports

For both single-node and multi-node deployments, the following ports are used:

```yaml
CMS: 30445
AAS: 30444
HVS: 30443
WLS: 30447
IHUB: None
KBS: 30448
K8s-scheduler: 30888
K8s-controller: None
TA: 31443
WLA: None
```





Authentication
==============

Beginning in the Intel® SecL-DC 1.6 release, authentication is centrally
managed by the Authentication and Authorization Service (AAS). This
service uses a Bearer Token authentication method, which replaces the
previous HTTP BASIC authentication. This service also centralizes the
creation of roles and users, allowing much easier management of users,
passwords, and permissions across all Intel® SecL-DC services.

To make an API request to an Intel® SecL-DC service, an authentication
token is now required. API requests must now include an Authorization
header with an appropriate token:

`Authorization: Bearer $TOKEN`

The token is issued by the AAS and will expire after a set amount of
time. This token may be used with any Intel® SecL-DC service, and will
carry the appropriate permissions for the role(s) assigned to the
account the token was generated for.



Create Token
------------

To request a new token from the AAS:

```
POST https://<AAS IP or hostname>:8444/aas/v1/token

{
    "username" : "<username>",
    "password" : "<password>"
}
```

The response will be a token that can be used in the Authorization
header for other requests. The length of time for which the token will
be valid is configured on the AAS using the key
`AAS\_JWT\_TOKEN\_DURATION\_MINS` (in the installation answer file during
installation) or `aas.jwt.token.duration.mins` (configured on the AAS
after installation). In both cases the value is the length of time in
minutes that issued tokens will remain valid before expiring.



User Management
---------------

Users in Intel® SecL-DC are no longer restrained to a specific service,
as they are now centrally managed by the Authentication and
Authorization Service. Any user may now be assigned roles for any
service, allowing user accounts to be fully defined by the tasks needed.

### Username and Password requirements

Passwords have the following constraints:

-   cannot be empty - i.e must at least have one character

-   maximum length of 255 characters

Usernames have the following requirements:

-   Format: username\[@host\_name\[domain\]\]

-   \[@host\_name\[domain\]\] is optional

-   username shall be minimum of 2 and maximum of 255 characters

-   username allowed characters are alphanumeric, ., -, \_ - but cannot
    start with -.

-   Domain name must meet requirements of a host name or fully qualified
    internet host name

-   Examples

    -   admin, admin\_wls, admin@wls, <admin@wls.intel.com>,
        <wls-admin@intel.com>

### Create User

```
POST https://<IP or hostname of AAS>:8444/aas/v1/users
Authorization: Bearer <token>

{
	"username" : "<username>",
	"password" : "<password>"
}
```

### Search Users by Username

```
GET https://<IP or hostname of AAS>:8444/aas/v1/users?name=<value>
```

### Change User Password

```
PATCH https://<IP or hostname of AAS>:8444/aas/v1/users/changepassword
Authorization: Bearer <token>
{
	"username": "<username>",
	"old_password": "<old_password>",
	"new_password": "<new_password>",
	"password_confirm": "<new_password>"
}
```

### Delete User

```
DELETE https://<IP or hostname of AAS>:8444/aas/v1/users/<User ID>
Authorization: Bearer <token>
```



Roles and Permissions
---------------------

Permissions in Intel® SecL-DC are managed by Roles. Roles are a set of
predefined permissions applicable to a specific service. Any number of
Roles may be applied to a User. While new Roles can be created, each
Intel® SecL service defines permissions that are applicable to specific
predetermined Roles. This means that only pre-defined Roles will
actually have any permissions. Role creation is intended to allow Intel®
SecL-DC services to define their permissions while allowing role and
user management to be centrally managed on the AAS. When a new service
is installed, it will use the Role creation functions to define roles
applicable for that service in the AAS.

### Create Role

```
POST https://<AAS IP or Hostname>:8444/aas/v1/roles
Authorization: Bearer <token>

{
    "service": "<Service name>",
    "name": "<Role Name>",
    "permissions": [<array of permissions>]
}
```

-   `service` field contains a minimum of 1 and maximum of 20 characters.
    Allowed characters are alphanumeric plus the special charecters -,
    \_, @, ., ,

-   `name` field contains a minimum of 1 and maximum of 40 characters.
    Allowed characters are alphanumeric plus the special characters -,
    \_, @, ., ,

-   `service` and `name` fields are mandatory

-   `context` field is optional and can contain up to 512 characters.
    Allowed characters are alphanumeric plus -, \_, @, ., ,,=,;,:,\*

-   `permissions` field is optional and allow up to a maximum of 512
    characters.

The Permissions array must a comma-separated list of permissions
formatted as `resource:action:`

Permissions required to execute specific API requests are listed with
the API resource and method definitions in the API documentation.

### Search Roles

```
GET https://<AAS IP or Hostname>:8444/aas/v1/roles?<parameter>=<value>
Authorization: Bearer <token>
```

Search parameters supported:

```
Service=<name of service>
Name=<role name>
Context=<context>
contextContains=<partial "context" string>
allContexts=<true or false>
filter=false
```

### Delete Role

```
DELETE https://<AAS IP or Hostname>:8444/aas/v1/roles/<role ID>
Authorization: Bearer <token>
```

### Assign Role to User

```
POST https://<AAS IP or Hostname>:8444/aas/v1/users/<user ID>/roles
Authorization: Bearer <token>

{
	"role_ids": ["<comma-separated list of role IDs>"]
}
```

### List Roles Assigned to User

```
GET https://<AAS IP or Hostname\>:8444/aas/v1/users/<user ID>/roles
Authorization: Bearer <token>
```

### Remove Role from User

```
DELETE https://<AAS IP or Hostname>:8444/aas/v1/users/<userID>/roles/<role ID>
Authorization: Bearer <token>
```

### Role Definitions

The following roles are created during installation (or by the
CreateUsers script) and exist by default.

| **Role Name**             | **Permissions**                                              | **Utility**                                                  |
| ------------------------- | :----------------------------------------------------------- | ------------------------------------------------------------ |
| TA:Administrator          | TA:\*:\*                                                     | Used by the Verification Service to access Trust Agent APIs, including retrieval of TPM quotes, provisioning Asset Tags and SOFTWARE Flavors, etc. |
| HVS:ReportSearcher        | HVS: \[reports:search:\*"]                                   | Used by the Integration Hub to retrieve attestation reports from the Verification Service |
| KBS:Keymanager            | KBS: \["keys:create:\*", "keys:transfer:\*"\]                | Used by the WPM to create and retrieve symmetric encryption keys to encrypt workload images |
| WLS:FlavorsImageRetrieval | WLS: image\_flavors:retrieve:\*                              | Used by the Workload Agent during Workload Confidentiality flows to retrieve the image Flavor |
| HVS: ReportCreator        | HVS: \["reports:create:\*"\]                                 | Used by the Workload Service to create new attestation reports on the Verification Service as part of Workload Confidentiality key retrievals. |
| Administrator             | \*:\*:\*                                                     | Global administrator role used for the initial administrator account. This role has all permissions across all services, including permissions to create new roles and users. |
| AAS: Administrator        | \*:\*:\*                                                     | Administrator role for the AAS only. Has all permissions for AAS resources, including the ability to create or delete users and roles. |
| AAS: RoleManager          | AAS: \[roles:create:\*, roles:retrieve:\*, roles:search:\*, roles:delete:\*\] | AAS role that allows all actions for Roles, but cannot create or delete Users or assign Roles to Users. |
| AAS: UserManager          | AAS: \[users:create:\*, users:retrieve:\*, users:store:\*, users:search:\*, users:delete:\*\] | AAS role with all permissions for Users, but has no ability to create Roles or assign Roles to Users. |
| AAS: UserRoleManager      | AAS: \[user\_roles:create:\*, user\_roles:retrieve:\*, user\_roles:search:\*, user\_roles:delete:\*, | AAS role with permissions to assign Roles to Users, but cannot create delete or modify Users or Roles. |
| HVS: AttestationRegister  | HVS: \[host\_tls\_policies:create:\*, hosts:create:\*, hosts:store:\*, hosts:search:\*, host\_unique\_flavors:create:\*, flavors:search:\*, tpm\_passwords:retrieve:\*, tpm\_passwords:create:\*, host\_aiks:certify:\* | Role used for Trust Agent provisioning. Used to create the installation token provided during installation. |
| HVS: Certifier            | HVS: host\_signing\_key\_certificates:create:\*              | Used for installation of the Workload Agent                  |



Connection Strings
==================

Connection Strings define a remote API resource endpoint that will be
used to communicate with the registered host for retrieving TPM quotes
and other host information. Connection Strings differ based on the type
of host.

Trust Agent 
-------------------------------

The Trust Agent connection string connects directly to the Trust Agent
on a given host. The Verification Service will use a service account
with the needed Trust Agent permissions to connect to the Trust Agent.
In previous Intel® SecL versions, each Trust Agent had its own unique
user access controls. Starting in the 1.6 release, all authentication
has been centralized with the new Authentication and Authorization
Service, eliminating the need for credentials to be provided for
connection strings connecting to Trust Agent resources.

By default, the Trust Agent uses "HTTP" mode, which uses the following connection string:

`intel:https://<HostNameOrIp>:1443`

The Trust Agent can also be used in NATS mode, which uses a slightly different connection string:

```
intel:nats://<unique host identifier, configured at Trust Agent installation>
```

The unique host identifier is a unique ID used by NATS to differentiate services when passing messages.  Any unique string is acceptable, but good examples can be the host's FQDN or hardware UUID.

VMware ESXi
-----------

### Importing VMware TLS Certificates

Before connecting to vCenter to register hosts or clusters, the vCenter TLS certificate needs to be imported to the Verification Service.  This must be done for each vCenter server that the Verification Service will connect to, for importing Flavors or registering hosts.

1. Download the root CA certs from vCenter:

   ```shell
wget --no-proxy "*" https://<vCenter IP or hostname>/certs/download.zip --no-check-certificate
   ```

   This downloads all the root CA certificates for you into `download.zip` file.

   ```shell
   unzip download.zip
   ```
   
   All of the certificates will be stored under `<pwd>/certs/`. Certs will be in `PEM` format.
   
   

2. Upload the certificates to the HVS

   ```http
   POST https://%3CIP%3E:8443/hvs/v2/ca-certificates
   
   {
       "name": "<cert name>",
       "type": "root",
       "certificate": "MIIELTCCAxW..."
   }
   ```

   > **Note** Please make sure that the certificate does not contain any other characters other than the base64 characters like that of `\n` or `-----BEGIN CERTIFICATE-----` etc.



3. After upload is successful, restart the HVS

   ```shell
   hvs restart
   ```

### Registering a VMware ESXi Host

The VMware ESXi connection string is actually directed to vCenter, not
the actual ESXi host. Many ESXi hosts managed by the same vCenter server
will use the same connection string. The username and password specified
are vCenter credentials, and the vCenter "Validate Session" privilege is
required for access.

```shell
vmware:https://<vCenterHostNameOrIp>:443/sdk;h=<hostname of ESXi host>;u=<username>;p=<password>
```



Platform Integrity Attestation
==============================

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

Host Registration
-----------------

Registration creates a host record with connectivity details and other
host information in the Verification Service database. This host record
will be used by the Verification Service to retrieve TPM attestation
quotes from the Trust Agent to generate an attestation report.

### Trust Agent

#### Registration via Trust Agent Command Line

The Trust Agent can register the host with a Verification Service by
running the following command:

```shell
tagent setup create-host 
```

> **Note**: The following environment variables must be exported for this command:
>
> HVS_URL=https://<hvs_IP/hostname>:8443/hvs/v2
> CURRENT_IP=<trustagent_IP>
> BEARER_TOKEN=<authentication token>
>
> **Note:** Because VMWare ESXi hosts do not use a Trust Agent, this method is
> not applicable for registration of ESXi hosts.

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
POST https://verification.service.com:8443/hvs/v2/hosts
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
POST https://verification.service.com:8443/hvs/v2/hosts
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



Flavor Creation for Automatic Flavor Matching
---------------------------------------------

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
`PLATFORM`, `OS`, `HOST_UNIQUE`, `SOFTWARE`, and `ASSET_TAG` measurements. These
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

> **Note**: See the Flavor Management section for additional details on how
> flavors can be managed, and how the Flavor matching engine works.
> The sample workflow provided here is intended to be an introduction
> only.

### Importing a Flavor from a Sample Host

```
POST https://verification.service.com:8443/hvs/v2/flavors
Authorization: Bearer <token>

{ 
    "connection_string": "<connection string>",
    "partial_flavor_types": ["PLATFORM", "OS", "HOST_UNIQUE"],
    "flavorgroup_names": []
}
```

Requires the permission `flavors:create`

> **Note**:The HOST\_UNIQUE Flavor parts, used by Red Hat Enterprise Linux and
> VMWare ESXi host types, MUST be created for each registered host of
> that type, and should in general be imported from that host. This
> means that importing the HOST\_UNIQUE flavor should always be done
> for each host registered.

To import ONLY the HOST\_UNIQUE Flavor part from a host:

```
POST https://verification.service.com:8443/hvs/v2/flavors
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
POST https://verification.service.com:8443/hvs/v2/flavors
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



Creating the Default SOFTWARE Flavor (Linux Only)
-------------------------------------------------

As part of the new Application Integrity feature added in Intel® SecL-DC
version 1.5, a new default SOFTWARE Flavor part is provided so that the
Linux Trust Agent itself can be measured and included in the attestation
process. The default SOFTWARE Flavor includes a manifest for the static
files and folders in the Trust Agent. The manifest is automatically
deployed to each Linux Trust Agent during the provisioning step.

> **Note**: The Linux Trust Agent **must** be rebooted after the
> Provisioning step is completed (typically Provisioning happens during
> installation, based on whether all of the required variables are set
> in the trustagent.env file). Rebooting allows the default SOFTWARE
> Flavor manifest to be measured and extended to the TPM PCRs. If the
> reboot is not performed, the system will require a SOFTWARE Flavor,
> but the measurements will not exist, and the system will appear
> Untrusted. If an un-rebooted host is used to create the SOFTWARE
> Flavor, the Flavor will be created based on measurements that do not
> exist, and will fail.

The `SOFTWARE` Flavor part should be created separately from the other
Flavor parts. Only one default `SOFTWARE` Flavor needs to be created for
each version of the Linux Trust Agent. If the `SOFTWARE` Flavor for the
same Trust Agent version is imported multiple times, subsequent imports
will fail as the Flavor already exists.

To import the `SOFTWARE` Flavor part from a host:

```
POST https://verification.service.com:8443/hvs/v2/flavors
Authorization: Bearer <token>

{ 
    "connection_string": "<connection string>",
    "partial_flavor_types": ["SOFTWARE"],
    flavorgroup_names": []
}
```

Requires the permission `flavors:create`



Creating and Provisioning Asset Tags
------------------------------------

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
POST https://verification.service.com:8443/hvs/v2/tag-certificates
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
POST https://verification.service.com:8443/hvs/v2/rpc/deploy-tag-certificate
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

> **Note**: Asset Tag is currently not supported for VMWare hosts
> using TPM 2.0.

##### Calculate the Certificate Hash Value

Only the hash value of the Asset Tag Certificate can be provisioned to
the TPM, due to the low size of the NVRAM.

1.  Retrieve the Asset Tag Certificate. The Asset Tag Certificate can be
    retrieved either from the response when the Asset Tag certificate is
    created, or by using a GET API request to retrieve the certificate:
    
    ```
    GET https://verification.service.com:8443/hvs/v2/tag-certificates?subjectEqualTo=<HardwareUUID>
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
POST https://verification.service.com:8443/hvs/v2/flavors
Authorization: Bearer <token>

{   
    "connection_string": "<VMWare vCenter connection string>",
    "partial_flavor_types": ["ASSET_TAG"]
}
```

Once the Asset Tag Flavor is imported, the host can be attested
including Asset Tags as normal.



Retrieving Current Attestation Reports
--------------------------------------

```
GET https://verification.service.com:8443/hvs/v2/reports?latestPerHost=true
Authorization: Bearer <token>
```



Retrieving Current Host State Information
-----------------------------------------

```
GET https://verification.service.com:8443/hvs/v2/host-status?latestPerHost=true
Authorization: Bearer <token>
```



Upgrading Hosts in the Datacenter to a New BIOS or OS Version
-------------------------------------------------------------

Software and firmware updates are a common occurrence in the datacenter.
Automatic Flavor matching makes this process relatively simple:

1.  Create a new Flavor for the new version. This may be manually
    created or imported directly from a sample host that has already
    received the upgrade. Be sure to create new Flavors for each TPM
    version represented in your datacenter.
    
    ```
    POST https://verification.service.com:8443/hvs/v2/flavors
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
    DELETE https://verification.service.com:8443/hvs/v2/flavors/<flavorId>
    Authorization: Bearer <token>
    ```

    

Removing Hosts From the Verification Service
--------------------------------------------

Hosts can be deleted at any time. Reports for that host will remain in
the Verification Service database for audit purposes.

```
DELETE https://verification.service.com:8443/hvs/v2/hosts/<hostId>
Authorization: Bearer <token>
```

The `hostId` can be retrieved either at the time the host is created, or
by searching hosts using the host’s hostname.



Removing Flavors
----------------

Flavors can be deleted; this will cause any hosts that match the deleted
Flavor to evaluate as Untrusted. This can be done if, for example, an
old BIOS version needs to be retired and should no longer exist in the
datacenter. By deleting the PLATFORM Flavor, hosts with the old BIOS
version will attest as Untrusted, flagging them for easy remediation.

```
DELETE https://verification.service.com:8443/hvs/v2/flavors/<flavorId>
```



Invalidating Asset Tags
-----------------------

Asset Tags can be deleted in two ways.

Deleting the `ASSET_TAG` Flavor part will retain the Asset Tag
certificate in the database, but will cause the host using this Tag to
no longer use the Asset Tag for attestation (the Tag result will be
disregarded and no tags will be exposed in the attestation Reports).

```
DELETE https://verification.service.com:8443/hvs/v2/flavors/<assetTagflavorId>
Authorization: Bearer <token>
```

Deleting the actual Asset Tag certificate will remove the certificate
from the database, but will not actually affect attestation results (the
authority for attestation results is the Flavor).

```
DELETE https://verification.service.com:8443/hvs/v2/tag-certificates/<assetTagCertificateId>
Authorization: Bearer <token>
```



Remediating an Untrusted attestation
------------------------------------

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



Attestation Reporting
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

Attestation Reports can also be retrieved in `json` or `xml` format. These
formats will not include the signature chain provided in the SAML
format, but will contain the actual measurement values and expected
Flavor values used for comparison. These reports are typically used for
remediation, because they will show specifically why a given Host
attested as Untrusted.

The format for a Report is determined by the `Accept` header in the
request.

Attestations are automatically generated in the Verification Service by
a repeating scheduled background process. This process looks for
Attestation Reports that are close to expiration, and triggers a new
Attestation Report. By default, Attestation Reports are valid for 90
minutes, and the background refresh process will trigger a new
attestation when a Report is found to be within 3 minutes of expiration.

A user can either retrieve the most recent currently valid Attestation
Report for a given host, or may trigger a new Attestation Report to be
generated. Typically, it is best to retrieve an existing Report for
performance reasons. Generating a new Attestation Report requires the
generation of a new TPM quote from the TPM of the host being attested;
TPM performance differs greatly between vendors, and a quote can take
anywhere between 2-7 seconds to generate.

### Sample Call – Generating a New Attestation Report

```
POST https://verification.service.com:8443/hvs/v2/reports
Authorization: Bearer <token>

{
    "host_name":"host-1"
}
```

Requires the permission `reports:create`

### Sample Call – Retrieving an Existing Attestation Report

```
GET https://verification.service.com:8443/hvs/v2/reports?hostName=HostName.server.com
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



Integration
-----------

Intel® SecL can be integrated with scheduler services (or potentially
other services) to provide additional security controls. For example, by
integrating Intel® SecL with the OpenStack scheduler service, the
OpenStack placement service can incorporate the Intel® SecL security
attributes into VM scheduling.

### The Integration Hub

The Integration Hub acts as the integration point between the Verification Service and a third party service. The primary purpose of the Hub is to collect and maintain up-to-date attestation information, and to “push” that information to the external service. The secondary purpose is to allow for multitenancy, the Verification Service does not allow for permissions to be applied for specific hosts, so a user with the “attestation” role can access all attestations for all hosts. By using separate Integration Hub instances for each Cloud environment (or "tenant"), the Hub will push attestations only for the associated hosts to a given tenant’s integration endpoints.

For example, Tenant A is using hosts 1-10 for an OpenStack environment. Tenant B is using hosts 11-15 for a Kubernetes environment. Two Hub instances must be configured, one managing tenant A's OpenStack cluster and a second instance managing Tenant B's Kubernetes environment.  Each integration Hub will automatically retrieve the list of hosts used by its configured orchestration endpoint, retrieve the attestation reports only for those hosts, and push the attestation attribute information to each configured endpoint. Neither tenant will have access to the Verification Service, and will not be able to see attestation or other host details regarding infrastructure used by other tenants.

Different integration endpoints can be added to the Integration Hub through a plugin architecture. By default, the Integration Hub includes plugins for OpenStack and Kubernetes (Kubernetes deployments require the additional installation of two Intel® SecL-DC Custom Resource Definitions on the Kube Control Plane).

<img src="Images\integration2" alt="image-20200621122316170" style="zoom:150%;" />

### Integration with OpenStack 

Starting in the Rocky release, OpenStack can now use “Traits” to provide qualitative data about Nova Compute hosts, and to establish Trait requirements for VM instances. The updated scheduler will place VMs
requiring a given Trait on Nova Compute nodes that meet the Trait requirements.

Intel SecL-DC uses the Integration Hub to continually push platform integrity and Asset Tag information to the OpenStack Traits resources.  This means the OpenStack scheduler natively supports workload scheduling incorporating Intel SecL-DC security attributes, including attestation report Trust status and Asset Tags. The OpenStack Placement Service will automatically attempt to place images with Trait requirements on compute nodes that have those Traits.

> **NOTE**: This control only applies to instances launched using the
> OpenStack scheduler, and the Traits functions will not affect
> manually-launched instances where a specific Compute Node is defined
> (since this does not use the scheduler at all). Intel SecL-DC uses
> existing OpenStack interfaces and does not modify OpenStack code.  The
> datacenter owner or OpenStack administrator is responsible for the
> security of the OpenStack workload scheduling process in general, and
> Intel recommends following published OpenStack security best
> practices.

#### Prerequisites

-   Verification Service must be installed and running.

-   OpenStack\* Rocky (or later) Nova, Glance, Horizon, and Keystone services must
    be installed and running

-   The Integration Hub must be installed and running.

#### Setting Image Traits

Image Traits define the policy for which Traits are required for that
Image to be launched on a Nova Compute node. By setting these Traits to
“required,” the OpenStack scheduler will require these same Traits to be
present on a Nova Compute node in order to launch instances of the
image.

To set the Image Traits for Intel SecL-DC, a specific naming convention
is used. This naming convention will match the Traits that the
Integration Hub will automatically push to OpenStack. Two types of
Traits are currently supported – one Trait is used to require that the
Compute Node be Trusted in the Attestation Report, and the other Trait
is used to require specific Asset Tag key/value pairs.

To require a `Trusted` Attestation Report:

```
CUSTOM_ISECL_TRUSTED=required
```

The naming convention for Asset Tags is more flexible, and any number of
these Traits can be used simultaneously.

> **Note**: All of the Traits must be present on the Compute Node for the
> scheduler to allow instances to land, so be sure not to set mutually
> exclusive Asset Tag values.

```
CUSTOM_ISECL_AT_TAG_<key>_<value>=required`
```

For example, to define a Trait that will require an Asset Tag where
`State = CA` use the following:

```
CUSTOM_ISECL_AT_TAG_STATE_CA= required
```

These Traits can be set using CLI commands for OpenStack Glance:

```shell
openstack image set --property trait:CUSTOM_ISECL_AT_STATE__CA=required <image_name>
```

```shell
openstack image set --property trait:CUSTOM_ISECL_TRUSTED=required <image_name>
```

To remove a Trait so that it is no longer required for an Image:

```shell
openstack image unset --property trait:CUSTOM_ISECL_AT_STATE__CA <image_name>
```

```shell
openstack image unset --property trait:CUSTOM_ISECL_TRUSTED <image_name>
```

#### Configuring the Integration Hub for Use with OpenStack

The Integration Hub must be configured with the API URLs and credentials for the OpenStack instance it will integrate with.  This can be done during installation using the "OPENSTACK_..." variables shown in the `ihub.env` answer file  sample (see the Installing the Integration Hub section).

However, this configuration can also be performed after installation using CLI commands:

```
ihub setup openstack --endpoint-url="http://openstack:5000/v3" --endpoint-user="username" --endpoint-pass="password"
```

Restart the Integration Hub after configuring the endpoint. Note that "endpoint name" should be replaced with any user-friendly name for the OpenStack instance you would prefer.

#### Scheduling Instances

Once Trait requirements are set for Images and the Integration Hub is
configured to push attributes to OpenStack, instances can be launched in
OpenStack as normal. As long as the OpenStack Nova scheduler is used to
schedule the workloads, only compliant Compute Nodes will be scheduled
to run instances of controlled Images.

> **NOTE**: This control only applies to instances launched using the
> OpenStack scheduler, and the Traits functions will not affect
> manually-launched instances where a specific Compute Node is defined
> (since this does not use the scheduler at all). Intel SecL-DC uses
> existing OpenStack interfaces and does not modify OpenStack code.  The
> datacenter owner or OpenStack administrator is responsible for the
> security of the OpenStack workload scheduling process in general, and
> Intel recommends following published OpenStack security best
> practices.

### Integration with Kubernetes

Through the use of Custom Resource Definitions for the Kubernetes
Control Plane, Intel® Security Libraries can make Kubernetes aware of Intel®
SecL security attributes and make them available for pod orchestration.
In this way, a security-sensitive pod can be launched only on `Trusted`
physical worker nodes, or on physical worker nodes that match specified
Asset Tag values.

> **NOTE**: This control only applies to pods launched using the
> Kubernetes scheduler, and these scheduling controls will not affect
> manually-launched instances where a specific worker node is defined
> (since this does not use the scheduler at all). Intel SecL-DC uses
> existing Kubernetes interfaces and does not modify Kubernetes code,
> using only the standard Custom Resource Definition mechanism to add
> this functionality to the Kubernetes Control Plane.  The datacenter owner or
> Kubernetes administrator is responsible for the security of the
> Kubernetes workload scheduling process in general, and Intel
> recommends following published Kubernetes security best practices.

#### Prerequisites

-   Verification Service must be installed and running.

-   Kubernetes Control Plane Node must be installed and running

-   The supported Kubernetes versions are from `1.14.8`-`1.17.3` and the
    integration is validated with `1.14.8` and `1.17.3`

-   Kubernetes Worker Nodes must be configured as physical hosts and
    attached to the Control Plane Node


#### Installing the Intel® SecL Custom Resource Definitions

Intel® SecL uses Custom Resource Definitions to add the ability to base
orchestration decisions on Intel® SecL security attributes to
Kubernetes. These CRDs allow Kubernetes administrators to configure pods
to require specific security attributes so that the Kubernetes Control Plane
Node will schedule those pods only on Worker Nodes that match the
specified attributes.

Two CRDs are required for integration with Intel® SecL – an extension
for the Control Plane nodes, and a scheduler extension. A single installer will
deploy both of these CRDs. The extensions are deployed as a Kubernetes
deployment in the `isecl` namespace.

> **Note:** Please refer detail steps given for  `3.15 Installing the Intel® SecL Kubernetes Extensions and Integration Hub` section.


#### Configuring Pods to Require Intel® SecL Attributes

1.  (Optional) Verify that the worker nodes have had their Intel® SecL
    security attributes populated:
    
    ```shell
    kubectl get nodes --show-labels
    ```
    
    The output should show the Trust staus and any Asset Tags applied to
    all of the registered Worker Nodes.
    
2.  Add the following to any Pod creation files:

    ```yaml
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: isecl.trusted
                    operator: In
                    values:
                      - "true"
                  - key: isecl.TAG_Country
                    operator: In
                    values:
                      - CA
                      - US
                  - key: isecl.TAG_Customer
                    operator: In
                    values:
                      - Coke
                      - Pepsi
                  - key: isecl.TAG_State
                    operator: In
                    values:
                      - CA
    ```


The `isecl.trusted` key defines the requirement for a Trusted host. Only
one of these keys should be used. The `isecl.TAG_` keys indicate Asset
Tags; if the workload should only launch on hosts with the `COUNTRY=US` Asset Tag, the pod should be launched with the matchExpression key
`isecl.TAG_COUNTRY` with the value `US`.

All of the matchExpression definitions must be true for a given worker
node to launch the pod – in the example above, the host must be
attested as Trusted with Asset Tags `Country=US` ,`Customer=Customer1` and `State=CA`. If the worker node has
additional Asset Tags beyond the ones required, the pod will still be
able to be launched on that node. However, if one of the specified
Tags is missing or has a different value, that worker node will not be
used for that pod.

#### Tainting Untrusted Worker Nodes

Optionally, the Intel® SecL Kubernetes CRDs can be configured to flag
worker nodes as `tainted` to prevent any pods from launching on them.
This restriction is applied regardless of whether the pod has a specific
trust policy – if a worker node is flagged as `tainted` no pods will be
launched on that worker.

This setting is disabled by default. To enable this setting:

1.  Edit the `isecl-controller.yaml` file under
    `/opt/isecl-k8s-extensions/yamls/isecl-controller.yaml` and set
    `TAINT_UNTRUSTED_NODES=true`

2.  Run
    
    ```shell
    kubectl apply -f /opt/isecl-k8s-extensions/yamls/isecl-controller.yaml
    ```

Worker nodes that attest as untrusted will be `tainted` with the
NoExecute flag and unable to launch pods.

If a worker was previously considered tainted and the untrusted state is
resolved, the Intel® SecL CRDs will remove the tainted flag and the
worker will be able to launch pods again.



Workload Confidentiality
========================

Workload Confidentiality builds upon Platform Attestation to protect
data in virtual machine and container images. At its core, this feature
is about allowing an image owner to set policies that define the
conditions under which their image will be allowed to run; if the policy
conditions are met, the decryption key will be provided, and if the
conditions are not met, the image will remain encrypted and
inaccessible. This provides a level of enforcement beyond integration
with orchestrators, and protects sensitive data when the image is at
rest.

Workload Encryption relies on Platform Attestation to define the
security attributes of hosts. When a protected image is launched, the
Workload Agent on the host launching the VM or container image will
detect the attempt (using either Libvirt hooks for VMs, or as a function
of CRI-O in the case of containers) and use the Image ID to find the Image Flavor on the Workload Service. The Workload Service will retrieve the current trust report for the host
launching the image, and use that report to make a key retrieval request
to the key transfer URL retrieved from the image flavor. The key
transfer URL refers to the URL to the image owner’s Key Broker Service,
along with the ID of the key needed.

In a typical production deployment, a Cloud Service Provider would
enable Intel® SecL-DC security controls by installing the Intel® SecL-DC
applications (with the exception of the Key Broker and Workload Policy
Manager), and configuring each workload host to be Trusted (as per the
Platform Integrity Attestation use case).

The owner of the workload image(s) to be protected (for example, the end
customer of the CSP) must install a Key Broker Service (which must be
available for network communication from the Workload Service hosted on
the CSP), the Workload Policy Manager, and their own Authentication and
Authorization Service and Certificate Management Service (these will
manage authentication and certificates for the KBS and WPM).

Any number of image owner customers with their own unique
KBS/WPM/AAS/CMS deployments may protect images that can be run by a
single CSP deployment.

The image owner will use the WPM to encrypt any image(s) to be
protected; the WPM will automatically create a new image encryption key
using the KBS, and will output the encrypted image and an Image Flavor.
The image owner can then upload the encrypted image to the CSP’s image
storage service, and then upload the Image Flavor to the CSP-hosted WLS.

<img src="Images\image-encryption.png" alt="image-20200622081105459" style="zoom:150%;" />

When a compute host at the CSP attempts to launch a protected image, the
WLA on the host will detect the launch request, and will issue a key
transfer request to the WLS. The WLS will use the image ID to retrieve
the Image Flavor, which contains the key retrieval URL for that image.
This URL is hosted on the KBS of the image owner (which is why the KBS
must be available to network requests from the WLS). The WLS will access
the HVS to retrieve the current Platform Integrity Attestation report
for the host, and will use this report to make a key transfer request to
the KBS at the key transfer URL.

The KBS will receive the request, verify that the Platform Integrity
Attestation report is signed using a known SAML signing key (verifying
that the report comes from a known and trusted HVS), and will then
verify that the report shows that the host is trusted.

If these requirements are met, the KBS will use the host’s Binding Key
(the public half of an asymmetric keypair generated by the host’s TPM
and included in the attestation report) as a Key Encryption Key to seal
the Image Encryption Key to the TPM of the host that was attested.

When the host receives the response to the key request, it will unseal
the Image Encryption Key using its TPM. Because the Key Encryption Key
is unique to this host’s TPM, only the actual host that was attested
will be able to gain access to the image.

With the Image Encryption Key, the host’s WLA will create the
appropriate encrypted volume(s) for the image and begin the launch as
normal.

The WLA does not retain the key on disk; if/when the host is rebooted or
the WLA is restarted, restarting the workloads based on protected images
will trigger new key requests based on new Platform Integrity
Attestation reports. In this way, if a host is compromised in a method
detectable by the Platform Integrity feature, protected images will be
unable to launch on this server.

<img src="Images\workload-decryption" alt="image-20200622081137301" style="zoom:150%;" />

Virtual Machine Confidentiality
-------------------------------

### Prerequisites

To enable Virtual Machine Confidentiality, the following Intel® SecL-DC
components must be installed and available:

-   Authentication and Authorization Service

-   Certificate Management Service

-   Key Broker Service

-   Host Verification Service

-   Workload Service

-   Trust Agent + Workload Agent (on each virtualization host)

-   Workload Policy Manager

See the Installation subsection on Recommended Service Layout for
recommendations on how/where to install each service.

It is strongly recommended to use a VM orchestration solution (for
example, OpenStack) with the Intel® SecL-DC Integration Hub to
schedule encrypted workloads on compute hosts that have already been
pre-checked for their Platform Integrity status. See the Platform
Integrity Attestation subsection on Integration with OpenStack for an
example.

You will need at least one QCOW2-format virtual machine image (for
quick testing purposes, a very small minimal premade image like CirrOS
is recommended; a good place to look for testing images is the
OpenStack Image Guide found here:
<https://docs.openstack.org/image-guide/obtain-images.html>).

One or more hypervisor compute nodes running QEMU/KVM is required.
Each of these nodes must have the Intel® SecL-DC Trust Agent and
Workload Agent installed, and they must be registered with the
Verification Service. Each of these servers should show as `trusted` see the Platform Integrity Attestation section for details. You should
have Flavors that match the system configuration for these hosts, and
attestation reports should show all Flavor parts as `trusted=true`
Hosts that are not trusted (including servers where there is no trust
status, like hosts with no Trust Agent) will fail to launch any
encrypted workloads.

### Workflow

#### Encrypting Images

```shell
wpm create-image-flavor -l <user-friendly unique label> -i <path to image file> -e <output path and filename for encrypted image> -o <output path for JSON image flavor>`
```

After generating the encrypted image with the WPM, the encrypted image
can be uploaded to the Image Storage service of choice (for example,
OpenStack Glance). Note that the ID of the image in this Image Storage
service must be retained and used for the next steps.

#### Uploading the Image Flavor

```
POST https://<Workload Service IP or Hostname>:5000/wls/flavors
Authorization: Bearer <token>

{<Image Flavor content from WPM output>}
```

Use the above API request to upload the Image Flavor to the WLS. The
Image Flavor will tell other Intel® SecL-DC components the Key Transfer
URL for this image.

#### Creating the Image Flavor to Image ID Association

The WLS needs to know the ID of the image as it exists in the image
storage service used by the CSP (for example, OpenStack Glance). Use the
below API request to create an association between the Image Flavor
created in the previous step and the image ID.

```
POST https://<Workload Service IP or Hostname>:5000/wls/images
Authorization: Bearer <token>

{
    "id": "<image ID on image storage>",
    "flavor_ids": ["<Image Flavor ID>"]
}
```

#### Launching Encrypted VMs

Instances of the protected images can now be launched as normal.
Encrypted images will only be accessible on hosts with a Platform
Integrity Attestation report showing the host is trusted.

If the VM is launched on a host that is not trusted, the launch will
fail, as the decryption key will not be provided.



Container Confidentiality
--------------------------------

### Container Confidentiality with Cri-o and Skopeo

#### Prerequisites

Container Confidentiality with Cri-o and Skopeo requires modified versions of both Cri-o and Skopeo.  Both of these are automatically built with the Intel SecL build scripts, and can be found here after the script has executed:

```
isecl/cc-crio/binaries/
```

[Skopeo](https://github.com/lumjjb/skopeo/tree/sample_integration)

- The patched version of Skopeo 0.1.41-dev must be installed on each Worker Node: https://github.com/lumjjb/skopeo/tree/sample_integration.

- The Skopeo wrapper that allows Skopeo to interface with the ISecL components must be installed on each Worker Node: https://github.com/lumjjb/skopeo/blob/sample_integration/vendor/github.com/lumjjb/seclkeywrap/keywrapper_secl.go.

- Copy the Skopeo wrapper into /usr/bin: 

  ```
  cp isecl/cc-crio/binaries/skopeo /usr/bin/skopeo
  ```

- Add the following to the crio.service definition to always start Cri-o with the Intel SecL policy parameters enabled:

  ```
  vi /usr/local/lib/systemd/system/crio.service
  ExecStart=/usr/local/bin/crio \
            $CRIO_CONFIG_OPTIONS \
            $CRIO_RUNTIME_OPTIONS \
            $CRIO_STORAGE_OPTIONS \
            $CRIO_NETWORK_OPTIONS \
            $CRIO_METRICS_OPTIONS \
            --decryption-secl-parameters secl:enabled
  ```

[Cri-o 1.17](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#cri-o)

- The patched version of Cri-o 1.17 must be installed on each Worker Node:   https://github.com/lumjjb/cri-o/blob/1.16_encryption_sample_integration.

- Copy the CRI-O binary from IsecL build script to /usr/bin/:

  ```
  cp isecl/cc-crio/binaries/crio /usr/bin/crio
  ```

  

- The Cri-o wrapper that allows Cri-o to interface with ISecL components must be installed on each Worker Node: https://github.com/lumjjb/cri-o/blob/1.16_encryption_sample_integration/vendor/github.com/lumjjb/seclkeywrap/keywrapper_secl.go.

- GoLang 1.14.4 must be installed on each Kubernetes Worker Node

- Crictl must be installed on each Kubernetes Worker Node

  ```
  $ VERSION="v1.17.0"
  $ wget https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-amd64.tar.gz
  $ sudo tar zxvf crictl-$VERSION-linux-amd64.tar.gz -C /usr/local/bin
  $ rm -f crictl-$VERSION-linux-amd64.tar.gz
  ```

- Kubernetes must be configured to use Cri-o and Skopeo
  
- Platform Integrity Attestation must be configured for the physical Kubernetes Worker Nodes.
  
  - This includes, at minimum, the CMS; AAS; HVS; KBS; WPM; and the Trust Agent must be installed on each Worker Node.  See the Installation section for details installing these services.
  - Each Kubernetes Worker Node should be Trusted in the attestation reports generated by the HVS.
  
- Only physical Worker Nodes are supported at this time.  

#### Workflow

###### Skopeo Commands

```
skopeo copy source-image destination-image

 Options:

--encryption-key [secl:asset_tag|keyfile] Specifies the encryption protocol. When using secl protocol, provide either "any" or an asset tag in the form "at_key:at_value"; only one asset tag can be used at this time. Alternatively, a specific key can be provided to be used for encryption.

--decryption-key [secl:enabled|keyfile] specifies the decryption Alternatively, a specific key can be provided to be used for decryption. This flag can be repeated if an image requires more than one key to be decrypted.
```

See https://github.com/lumjjb/skopeo/blob/sample_integration/docs/skopeo-copy.1.md for more details.

######  Examples

Copy a container image from a registry to a local server:

```
$ skopeo copy docker://docker.io/library/nginx:latest oci:nginx_local
```

To encrypt an image (this will allow the image to run only on Trusted platforms):

```
$ skopeo copy --encryption-key secl:any oci:nginx_local oci:nginx_secl_enc
```

To encrypt an image with an Asset Tag (this will allow the image to run only on Trusted platforms with the specified Asset tag):

```
$ skopeo copy --encryption-key secl:asset_tag_key:asset_tag_value oci:nginx_local oci:nginx_secl_enc_w_at
```

To decrypt an image:

```
$ skopeo copy --decryption-key secl:enabled oci:nginx_secl_enc oci:nginx_secl_dec
```

To copy an encrypted image without decryption:

```
$ skopeo copy oci:nginx_secl_enc oci:nginx_secl_enc_copy
```

To copy a local image to a remote registry:

```
$ skopeo copy oci:nginx_secl_enc docker://10.80.245.116/nginx_secl_enc:latest
```

###### Prepare an Image

Convert the image to an OCI image using Skopeo:

```
$ skopeo copy docker-daemon:custom-image:latest oci:custom-image:latest
```

Encrypt the image using Skopeo copy command

```
$ skopeo copy --encryption-key secl:any oci:custom-image:latest oci:custom-image:enc
```

Push the image to a registry:

```
$ skopeo copy oci:custom-image:enc docker://Registry.server.com:5000/custom-image:enc
```

Alternatively, encrypt the image and push it to a registry in a single step:

```
$ skopeo copy --encryption-key secl:any oci:custom-image:latest docker://registry.server.com:5000/custom-image:enc
```


##### Pulling and Encrypting a Container Image

Skopeo can be used to pull a container image from an external registry (a private Docker registry is used in the examples below). This image may be encrypted already, but if you wish to pull an image for encryption, it must be in plaintext format. Skopeo has a wrapper that can interact with the Workload Policy Manager. When trying to encrypt an image, Skopeo calls the WPM CLI fetch-key command. In the command, the KBS is called in order to create a new key. The return from the KBS includes the key retrieval URL, which is used when trying to decrypt. After the key is returned to the WPM, the WPM passes the key back to Skopeo. Skopeo uses the key to encrypt the image layer by layer as well as associate the encrypted image with the key's URL. Skopeo then uploads the encrypted image to a remote container registry.

The modified Cri-o and wrapper will modify the Cri-o commands to allow Intel SecL policies to be utilized.


#####  Launching an Encrypted Container Image

Cri-o allows for pulling and decryption of an encrypted container image from a container registry. When trying to pull and decrypt a container image, Cri-o has a hook that calls into the Workload Agent (WLA). The WLA will call into the Workload Service (WLS) and pass it the key URL associated with the encrypted image as well as the host's hardware UUID. These two serve as input to /keys endpoint of the WLS. The WLS initializes a HVS client in order to retrieve the host SAML report and then validates the report. If the host is trusted, the WLS will attempt to get the key. First, it will check if it's been cached alredy. If not, it will initialize a KBS client. The WLS uses this client to retrieve the key from the KBS. If the key is retrieved, it will be cached in the WLS temporarily so that the WLS will not need to requery the KBS if attempting to decrypt with the same key. The key is then passed back to the WLA as the return of the WLS's keys API. Finally, the key is returned to Cri-o, which uses the key to decrypt the container image layer by layer.

Containers of the protected images can now be launched as normal using Kubernetes pods and deployments. Encrypted images will only be accessible on hosts with a Platform Integrity Attestation report showing the host is trusted. If the Crio Container is launched on a host that is not trusted, the launch will fail, as the decryption key will not be provided.





Trusted Virtual Kubernetes Worker Nodes
=======================================

While the existing Platform Integrity Attestation functions support
bare-metal Kubernetes Worker Nodes, using Virtual Machines to host the
Worker Nodes is a common deployment architecture. This feature aims to
help extend the Chain of Trust to protect the integrity of Virtual
Machines, including virtual Kubernetes Worker Nodes. This feature
requires the foundational Platform integrity Attestation feature as a
prerequisite for the bare-metal servers hosting the virtual Worker
Nodes.

> **Note**: This feature requires a degree of separation between the VM
> and Kubernetes infrastructure. All physical, bare-metal servers should
> be virtualization hosts, and all Kubernetes Worker Nodes should be
> Virtual Machines running on those physical virtualization hosts.
> Kubernetes clusters should not use a mixture of both virtual and
> bare-metal Workers. The physical virtualization clusters should not
> include a mixture of hosts protected by Intel® SecL Platform integrity
> Attestation and hosts that are not protected. VM trust reports can
> only be generated for VM instances launched on hosts with Intel® SecL
> services enabled.
>
> **Also important to note is that this feature alone will not prevent
> any VMs from launching**. VMs will still be launched on Untrusted
> platforms unless additional steps are taken (for example, using
> OpenStack orchestration integration with Intel® SecL, or using the
> Workload Confidentiality feature to encrypt the Kubernetes Worker Node
> VM image). This feature generates VM attestation reports that can be
> used to audit compliance and extend the Chain of Trust, and relies on
> other datacenter policies and/or Intel® SecL features to enforce
> compliance.

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

Prerequisites
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

-   (Optional) Virtual Machine Images may be encrypted using the
    Workload Confidentiality feature. This adds a layer of cryptographic
    enforcement to the orchestration of virtual worker VMs, ensuring
    that the VMs can only be launched on compliant platforms.

Workflow
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

Sample VM Trust Report
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



Flavor Management
=================

Flavor Format Definitions
-------------------------

A Flavor is a standardized set of expectations that determines what
platform measurements will be considered “trusted.” Flavors are
constructed in a specific format, containing a metadata section
describing the Flavor, and then various other sections depending on the
Flavor type or Flavor part.

### Meta

The first part of a Flavor is the `meta` section:

```json
"meta":{
    "vendor": "INTEL",
    "description": {
        "flavor_part": "PLATFORM",
        "bios_name": "Intel Corporation",
        "bios_version": "SE5C620.86B.00.01.0004.071220170215",
        "tpm_version": "2.0"
    }
}
```

This section defines the Flavor part and any versioning information.

> **NOTE**:  Even when the BIOS or OS version remains the same, the actual
> measurements in the measured boot process will be different between
> TPM 1.2 and TPM 2.0, and so the TPM version is captured here as
> well. The attributes in the Meta section are used by the Flavor
> matching engine when matching Flavors to Hosts.  Note that TPM 1.2 is supported only for VMware ESXi hosts.

### Hardware

The `hardware` section is unique to PLATFORM flavor parts:

```json
"hardware": {
    "processor_info": "54 06 05 00 FF FB EB BF",
    "processor_flags": "fpu vme de …",
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
}
```

This part of the Flavor defines expected hardware attributes of the
host, and contains processor and TPM-related attributes.

### PCR banks (Algorithms)

TPMs can have one or more PCR banks enabled with different hash algorithms.  Intel SecL will always attempt to use the most secure algorithm available in the enabled PCR banks.

For example, if a given TPM has the following PCR banks enabled:

```
SHA1
SHA256
SHA384
```

The HVS will prefer the SHA384 PCR bank when creating flavors and performing attestations.  

The TPM vendor and version, platform OEM, and BIOS version and configuration each impact which PCR banks can potentially be enabled.  Some manufacturers will allow users to configure which banks are enabled/disabled in the BIOS.  Other manufacturers will enable only one PCR bank, and others will be permanently disabled.

Flavors will only utilize a single PCR bank, and when importing from a sample host the HVS will always prefer the strongest algorithm supported by the enabled TPM PCR banks.  In teh above example, a flavor imported from that host would use the SHA384 bank for all hash values.  This means that all hosts that will be attested using this flavor must also have SHA284 banks enabled in their TPMs.

Typically, among otherwise-identical servers this will not be an issue.  However, in a mixed environment it can be possible to have an OS flavor, for example, that needs to apply for some hosts that have SHA384 banks enabled, and other servers that only have SHA256 enabled and do not support SHA384.

In this circumstance, multiple flavors for the same OS version would need to be created - one for SHA384, and another for SHA256.



### PCRs

The last section of a Flavor is the “PCRs” section, which contains the
actual expected measurements for any PCRs. This section will contain PCR
measurements for each applicable algorithm supported by the TPM (SHA1
only for TPM 1.2, SHA256 and SHA1 sections for TPM 2.0).

Some PCRs simply have a value and nothing else. Other PCRs, however,
contain different `event` measurements. This indicates that separate
individual platform or OS components are independently measured and
extended to the same PCR. PCRs with event measurements will contain an
`Event` array that lists, in the correct order, all of the events in the
measurement event log that are extended to this PCR. When the
Verification Service attests a host against a given Flavor, each
measurement event is compared to the Flavor value, and all of the events
are replayed to confirm that a replay of all of the measurement
extensions do in fact result in the hash seen in the PCR value. In this
way, the Verification Service can ensure that the measurement event log
contents are secure, and the individual measurements can be attested so
that the cause for an Untrusted attestation can easily be seen.

The full PCRs section is not shown here due to length; see the sample
Flavor sections for a full sample.

```JSON
"pcrs": {
    "SHA1": {
        "pcr_0": {
            "value": "d2ed125942726641a7260c4f92beb67d531a0def"
        },
        "pcr_17": {
            "value": "1ec12004b371e3afd43d04155abde7476a3794fa",
            "event": [
                {
                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                    "value": "2fb7d57dcc5455af9ac08d82bdf315dbcc59a044",
                    "label": "HASH_START",
                    "info": {
                        "ComponentName": "HASH_START",
                        "EventName": "OpenSource.EventName"
                    }
                },
                ...
```

### Sample PLATFORM Flavor

The `PLATFORM` Flavor part encompasses measurements that are unique to a
specific platform, including the server OEM, BIOS version, etc. A
PLATFORM Flavor can be `shared` across all hosts of the same model that
have the same BIOS version.

```json
{
    "flavor_collection": {
        "flavors": [
            {
                "meta": {
                    "vendor": "INTEL",
                    "description": {
                        "flavor_part": " PLATFORM",
                        "bios_name": "Intel Corporation",
                        "bios_version": "SE5C620.86B.00.01.0004.071220170215",
                        "tpm_version": "2.0"
                    }
                },
                "hardware": {
                    "processor_info": "54 06 05 00 FF FB EB BF",
                    "processor_flags": "fpu vme de …",
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
                            "event": [
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "2fb7d57dcc5455af9ac08d82bdf315dbcc59a044",
                                    "label": "HASH_START",
                                    "info": {
                                        "ComponentName": "HASH_START",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "ffb1806465d2de1b7531fd5a2a6effaad7c5a047",
                                    "label": "BIOSAC_REG_DATA",
                                    "info": {
                                        "ComponentName": "BIOSAC_REG_DATA",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "3c585604e87f855973731fea83e21fab9392d2fc",
                                    "label": "CPU_SCRTM_STAT",
                                    "info": {
                                        "ComponentName": "CPU_SCRTM_STAT",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "9069ca78e7450a285173431b3e52c5c25299e473",
                                    "label": "LCP_CONTROL_HASH",
                                    "info": {
                                        "ComponentName": "LCP_CONTROL_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "5ba93c9db0cff93f52b521d7420e43f6eda2784f",
                                    "label": "LCP_DETAILS_HASH",
                                    "info": {
                                        "ComponentName": "LCP_DETAILS_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "5ba93c9db0cff93f52b521d7420e43f6eda2784f",
                                    "label": "STM_HASH",
                                    "info": {
                                        "ComponentName": "STM_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "3c585604e87f855973731fea83e21fab9392d2fc",
                                    "label": "OSSINITDATA_CAP_HASH",
                                    "info": {
                                        "ComponentName": "OSSINITDATA_CAP_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "3d42560dcf165a5557b3156a21583f2c6dbef10e",
                                    "label": "MLE_HASH",
                                    "info": {
                                        "ComponentName": "MLE_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "274f929dbab8b98a7031bbcd9ea5613c2a28e5e6",
                                    "label": "NV_INFO_HASH",
                                    "info": {
                                        "ComponentName": "NV_INFO_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "ca96de412b4e8c062e570d3013d2fccb4b20250a",
                                    "label": "tb_policy",
                                    "info": {
                                        "ComponentName": "tb_policy",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "d123e2f2b30f1effa8d9522f667af0dac4f48cfb",
                                    "label": "vmlinuz",
                                    "info": {
                                        "ComponentName": "vmlinuz",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "f3742133e1a0deb48177a74ed225418e5cf73fd1",
                                    "label": "initrd",
                                    "info": {
                                        "ComponentName": "initrd",
                                        "EventName": "OpenSource.EventName"
                                    }
                                }
                            ]
                        }
                    },
                    "SHA256": {
                        "pcr_0": {
                            "value": "db83f0e8a1773c21164c17986037cdf8afc1bbdc1b815772c6da1befb1a7f8a3"
                        },
                        "pcr_17": {
                            "value": "50bd58407a1893056eacff493245cfe785f045b2c0e1cc3e6e9eb5812d8d91bd",
                            "event": [
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "9301981c093654d5aa3430ba05c880a52eb22b9e18248f5f93e1fe1dab1cb947",
                                    "label": "HASH_START",
                                    "info": {
                                        "ComponentName": "HASH_START",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "2785d1ed65f6b5d4b555dc24ce5e068a44ce8740fe77e01e15a10b1ff66cca90",
                                    "label": "BIOSAC_REG_DATA",
                                    "info": {
                                        "ComponentName": "BIOSAC_REG_DATA",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "67abdd721024f0ff4e0b3f4c2fc13bc5bad42d0b7851d456d88d203d15aaa450",
                                    "label": "CPU_SCRTM_STAT",
                                    "info": {
                                        "ComponentName": "CPU_SCRTM_STAT",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "df3f619804a92fdb4057192dc43dd748ea778adc52bc498ce80524c014b81119",
                                    "label": "LCP_CONTROL_HASH",
                                    "info": {
                                        "ComponentName": "LCP_CONTROL_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                }
                            ]
                        }
                    }
                }
```

### Sample OS Flavor

An OS Flavor encompasses all of the measurements unique to a given OS.
This includes the OS kernel and other measurements.

```json
{
    "flavor_collection": {
        "flavors": [
            {
                "meta": {
                    "vendor": "INTEL",
                    "description": {
                        "flavor_part": "OS",
                        "os_name": "RedHatEnterpriseServer",
                        "os_version": "7.3",
                        "vmm_name": "",
                        "vmm_version": "",
                        "tpm_version": "2.0"
                    }
                },
                "pcrs": {
                    "SHA1": {
                        "pcr_17": {
                            "value": "1ec12004b371e3afd43d04155abde7476a3794fa",
                            "event": [
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "2fb7d57dcc5455af9ac08d82bdf315dbcc59a044",
                                    "label": "HASH_START",
                                    "info": {
                                        "ComponentName": "HASH_START",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "ffb1806465d2de1b7531fd5a2a6effaad7c5a047",
                                    "label": "BIOSAC_REG_DATA",
                                    "info": {
                                        "ComponentName": "BIOSAC_REG_DATA",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "3c585604e87f855973731fea83e21fab9392d2fc",
                                    "label": "CPU_SCRTM_STAT",
                                    "info": {
                                        "ComponentName": "CPU_SCRTM_STAT",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "9069ca78e7450a285173431b3e52c5c25299e473",
                                    "label": "LCP_CONTROL_HASH",
                                    "info": {
                                        "ComponentName": "LCP_CONTROL_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "5ba93c9db0cff93f52b521d7420e43f6eda2784f",
                                    "label": "LCP_DETAILS_HASH",
                                    "info": {
                                        "ComponentName": "LCP_DETAILS_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "5ba93c9db0cff93f52b521d7420e43f6eda2784f",
                                    "label": "STM_HASH",
                                    "info": {
                                        "ComponentName": "STM_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "3c585604e87f855973731fea83e21fab9392d2fc",
                                    "label": "OSSINITDATA_CAP_HASH",
                                    "info": {
                                        "ComponentName": "OSSINITDATA_CAP_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "3d42560dcf165a5557b3156a21583f2c6dbef10e",
                                    "label": "MLE_HASH",
                                    "info": {
                                        "ComponentName": "MLE_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "274f929dbab8b98a7031bbcd9ea5613c2a28e5e6",
                                    "label": "NV_INFO_HASH",
                                    "info": {
                                        "ComponentName": "NV_INFO_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "ca96de412b4e8c062e570d3013d2fccb4b20250a",
                                    "label": "tb_policy",
                                    "info": {
                                        "ComponentName": "tb_policy",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "d123e2f2b30f1effa8d9522f667af0dac4f48cfb",
                                    "label": "vmlinuz",
                                    "info": {
                                        "ComponentName": "vmlinuz",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "f3742133e1a0deb48177a74ed225418e5cf73fd1",
                                    "label": "initrd",
                                    "info": {
                                        "ComponentName": "initrd",
                                        "EventName": "OpenSource.EventName"
                                    }
                                }
                            ]
                        }
                    },
                    "SHA256": {
                        "pcr_17": {
                            "value": "50bd58407a1893056eacff493245cfe785f045b2c0e1cc3e6e9eb5812d8d91bd",
                            "event": [
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "9301981c093654d5aa3430ba05c880a52eb22b9e18248f5f93e1fe1dab1cb947",
                                    "label": "HASH_START",
                                    "info": {
                                        "ComponentName": "HASH_START",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "2785d1ed65f6b5d4b555dc24ce5e068a44ce8740fe77e01e15a10b1ff66cca90",
                                    "label": "BIOSAC_REG_DATA",
                                    "info": {
                                        "ComponentName": "BIOSAC_REG_DATA",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "67abdd721024f0ff4e0b3f4c2fc13bc5bad42d0b7851d456d88d203d15aaa450",
                                    "label": "CPU_SCRTM_STAT",
                                    "info": {
                                        "ComponentName": "CPU_SCRTM_STAT",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "df3f619804a92fdb4057192dc43dd748ea778adc52bc498ce80524c014b81119",
                                    "label": "LCP_CONTROL_HASH",
                                    "info": {
                                        "ComponentName": "LCP_CONTROL_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "6e340b9cffb37a989ca544e6bb780a2c78901d3fb33738768511a30617afa01d",
                                    "label": "LCP_DETAILS_HASH",
                                    "info": {
                                        "ComponentName": "LCP_DETAILS_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "6e340b9cffb37a989ca544e6bb780a2c78901d3fb33738768511a30617afa01d",
                                    "label": "STM_HASH",
                                    "info": {
                                        "ComponentName": "STM_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "67abdd721024f0ff4e0b3f4c2fc13bc5bad42d0b7851d456d88d203d15aaa450",
                                    "label": "OSSINITDATA_CAP_HASH",
                                    "info": {
                                        "ComponentName": "OSSINITDATA_CAP_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "26e1d98742f79c950dc637f8c067b0b72a1b0e8ff75db4e609c7e17321acf3f4",
                                    "label": "MLE_HASH",
                                    "info": {
                                        "ComponentName": "MLE_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "0f6e0c7a5944963d7081ea494ddff1e9afa689e148e39f684db06578869ea38b",
                                    "label": "NV_INFO_HASH",
                                    "info": {
                                        "ComponentName": "NV_INFO_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "27808f64e6383982cd3bcc10cfcb3457c0b65f465f779d89b668839eaf263a67",
                                    "label": "tb_policy",
                                    "info": {
                                        "ComponentName": "tb_policy",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "c89ad1d1e9adaa7ecfee2abce763b92472685f7d1b9f3799bf49974b66ed9638",
                                    "label": "vmlinuz",
                                    "info": {
                                        "ComponentName": "vmlinuz",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "81b88e268e697ccf1790d41b9de748a8f395acfb47aa67c9845479d4e8456f77",
                                    "label": "initrd",
                                    "info": {
                                        "ComponentName": "initrd",
                                        "EventName": "OpenSource.EventName"
                                    }
                                }
                            ]
                        }
                    }
                }
            }
        ]
    },
    "flavorgroup_name": "automatic"
}
```

### Sample HOST\_UNIQUE Flavor

Host-Unique flavors define measurements for a specific host. This can be
either a single large flavor that incorporates all of the host
measurements into a single flavor document used only to attest a single
host, or can be a small subset of measurements that are specific to a
single host. For example, some VMWare module measurements will change
from one host to the next, while most others will be shared assuming the
same ESXi build is used. The full Flavor requirement for such a host
would include Host-Unique flavors to cover the measurements that are
unique to only this one host, and would still use a generic PLATFORM and
OS flavor for the other measurements that would be identical for other
similarly configured hosts.

> **Note**:The HOST\_UNIQUE Flavors are unique to a specific host, and should
> always be imported directly from the specific host.

```json
{
    "flavors": [
        {
            "meta": {
                "id": "4d387cbd-f72b-4742-b4e5-c5b0ffed59e0",
                "vendor": "INTEL",
                "description": {
                    "flavor_part": "HOST_UNIQUE",
                    "source": "Purley11",
                    "bios_name": "Intel Corporation",
                    "bios_version": "SE5C620.86B.00.01.0004.071220170215",
                    "os_name": "RedHatEnterpriseServer",
                    "os_version": "7.4",
                    "tpm_version": "2.0",
                    "hardware_uuid": "00448C61-46F2-E711-906E-001560A04062"
                }
            },
            "pcrs": {
                "SHA256": {
                    "pcr_17": {
                        "value": "f9ef8c53ddfc8096d36eda5506436c52b4bfa2bd451a89aaa102f03181722176",
                        "event": [
                            {
                                "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                "value": "df3f619804a92fdb4057192dc43dd748ea778adc52bc498ce80524c014b81119",
                                "label": "LCP_CONTROL_HASH",
                                "info": {
                                    "ComponentName": "LCP_CONTROL_HASH",
                                    "EventName": "OpenSource.EventName"
                                }
                            },
                            {
                                "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                "value": "09f468dfc1d98a1fee86eb7297a56b0e097d57be66db4eae539061332da2e723",
                                "label": "initrd",
                                "info": {
                                    "ComponentName": "initrd",
                                    "EventName": "OpenSource.EventName"
                                }
                            }
                        ]
                    },
                    "pcr_18": {
                        "value": "c1f7bfdae5f270d9f13aa9620b8977951d6b759f1131fe9f9289317f3a56efa1",
                        "event": [
                            {
                                "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                "value": "df3f619804a92fdb4057192dc43dd748ea778adc52bc498ce80524c014b81119",
                                "label": "LCP_CONTROL_HASH",
                                "info": {
                                    "ComponentName": "LCP_CONTROL_HASH",
                                    "EventName": "OpenSource.EventName"
                                }
                            }
                        ]
                    }
                },
                "SHA1": {
                    "pcr_17": {
                        "value": "48695f747a3d494710bd14d20cb0a93c78a485cc",
                        "event": [
                            {
                                "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                "value": "9069ca78e7450a285173431b3e52c5c25299e473",
                                "label": "LCP_CONTROL_HASH",
                                "info": {
                                    "ComponentName": "LCP_CONTROL_HASH",
                                    "EventName": "OpenSource.EventName"
                                }
                            },
                            {
                                "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                "value": "b1f8db372e396bb128280821b7e0ac54a5ec2791",
                                "label": "initrd",
                                "info": {
                                    "ComponentName": "initrd",
                                    "EventName": "OpenSource.EventName"
                                }
                            }
                        ]
                    },
                    "pcr_18": {
                        "value": "983ec7db975ed31e2c85ef8e375c038d6d307efb",
                        "event": [
                            {
                                "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                "value": "9069ca78e7450a285173431b3e52c5c25299e473",
                                "label": "LCP_CONTROL_HASH",
                                "info": {
                                    "ComponentName": "LCP_CONTROL_HASH",
                                    "EventName": "OpenSource.EventName"
                                }
                            }
                        ]
                    }
                }
            }
        }
    ]
}
```

### Sample ASSET\_TAG Flavor

Asset Tag flavor parts are unique to Asset Tag attestation. These
flavors verify that the Asset Tag data in the host’s TPM correctly
matches the most recently created, currently valid Asset Tag certificate
that has been deployed to that host.

```json
{
    "meta": {
        "id": "b3e0c056-5b6c-4b6b-95c4-de5f1473cac0",
        "description": {
            "flavor_part": "ASSET_TAG",
            "hardware_uuid": "<Hardware UUID of the server to be tagged>"
        }
    },
    "external": {
        "asset_tag": {
            "tag_certificate": {
                "encoded": "<Tag certificate in base64 encoded format>",
                "issuer": "CN=assetTagService",
                "serial_number": 1519153541461,
                "subject": "<Hardware UUID of the server to be tagged>",
                "not_before": "2018-02-20T11:05:41-0800",
                "not_after": "2019-02-20T11:05:41-0800",
                "fingerprint_sha384": "46001d8472e56de423aac7c55f061404d27d50e497dfc21a861ef1965d7ac1e44887aee918fb5805385a3cbdf820899d",
                "attribute": [
                    {
                        "attr_type": {
                            "id": "2.5.4.789.2"
                        },
                        "attribute_values": [
                            {
                                "objects": {}
                            }
                        ]
                    },
                    {
                        "attr_type": {
                            "id": "2.5.4.789.2"
                        },
                        "attribute_values": [
                            {
                                "objects": {}
                            }
                        ]
                    },
                    {
                        "attr_type": {
                            "id": "2.5.4.789.2"
                        },
                        "attribute_values": [
                            {
                                "objects": {}
                            }
                        ]
                    }
                ]
            }
        }
    }
}
```



Flavor Templates
---------------

Added in the Intel SecL-DC 4.0 release, Flavor Templates expose the backend logic that determines which PCRs and event log measurements will be used for specific Flavor parts.  Where previously these rules were hardcoded on the backend, this new feature allows new templates to be added, and allows the customization or deletion of existing templates to suit specific business needs.

Flavor Templates are conditional rules that apply to a Flavor part cumulatively based on defined conditions.  For example a PLATFORM Flavor for a Linux host with Intel TXT enabled would normally include PCR0. If tboot is also enabled, elements from PCR 17 and 18 will be added to the PLATFORM flavor.  These are cumulative based on which conditions are true on a given host.

By default, Flavor Templates will come pre-populated in the HVS database to meet the same default behavior for previous releases.  

Flavor Templates can be added, removed, or edited to create customized rules.  For example, if there is a specific event log measurement that a user would like to add to an OS flavor, a new Flavor Template can be added for the OS Flavor part that defines a condition for applying the Template, along with the specific event log measurement that should be used when that condition is satisfied.

Flavor Templates are cumulative.  If a given host matches all of the conditions defined for Flavor Template A and Flavor Template B, both Templates will be applied when importing Flavors from that host.

### Sample Flavor Template

Below is a sample default Flavor Template used for RedHat Enterprise Linux servers with TPM2.0 and tboot enabled:

```json
{
   "id":"8798fb0c-2dfa-4464-8281-e650a30da7e6",
   "label":"default-linux-rhel-tpm20-tboot",
   "condition":[
      "//host_info/os_name//*[text()='RedHatEnterprise']",
      "//host_info/hardware_features/TPM/meta/tpm_version//*[text()='2.0']",
      "//host_info/tboot_installed//*[text()='true']"
   ],
   "flavor_parts":{
      "OS":{
         "meta":{
            "tpm_version":"2.0",
            "tboot_installed":true
         },
         "pcr_rules":[
            {
               "pcr":{
                  "bank":[
                     "SHA384",
                     "SHA256",
                     "SHA1"
                  ],
                  "index":17
               },
               "eventlog_includes":[
                  "vmlinuz"
               ]
            }
         ]
      },
      "PLATFORM":{
         "meta":{
            "tpm_version":"2.0",
            "tboot_installed":true
         },
         "pcr_rules":[
            {
               "pcr":{
                  "bank":[
                     "SHA384",
                     "SHA256",
                     "SHA1"
                  ],
                  "index":0
               },
               "pcr_matches":true
            },
            {
               "pcr":{
                  "bank":[
                     "SHA384",
                     "SHA256",
                     "SHA1"
                  ],
                  "index":17
               },
               "eventlog_equals":{
                  "excluding_tags":[
                     "LCP_CONTROL_HASH",
                     "initrd",
                     "vmlinuz"
                  ]
               }
            },
            {
               "pcr":{
                  "bank":[
                     "SHA384",
                     "SHA256",
                     "SHA1"
                  ],
                  "index":18
               },
               "eventlog_equals":{
                  "excluding_tags":[
                     "LCP_CONTROL_HASH",
                     "initrd",
                     "vmlinuz"
                  ]
               }
            }
         ]
      },
      "HOST_UNIQUE":{
         "meta":{
            "tpm_version":"2.0",
            "tboot_installed":true
         },
         "pcr_rules":[
            {
               "pcr":{
                  "bank":[
                     "SHA384",
                     "SHA256",
                     "SHA1"
                  ],
                  "index":17
               },
               "eventlog_includes":[
                  "LCP_CONTROL_HASH",
                  "initrd"
               ]
            },
            {
               "pcr":{
                  "bank":[
                     "SHA384",
                     "SHA256",
                     "SHA1"
                  ],
                  "index":18
               },
               "eventlog_includes":[
                  "LCP_CONTROL_HASH"
               ]
            }
         ]
      }
   }
}
```

### Flavor Template Definitions

A Flavor Template consists of several sections:

The "id" and "label" keys are unique identifiers.  The ID is generated automatically by the HVS when the Template is created; the label is user-specified and must be unique.

#### Conditions

The "condition" section contains a map of host-info elements to match when deciding to apply the Template.  For example:

```json
   "condition":[
      "//host_info/os_name//*[text()='RedHatEnterprise']",
      "//host_info/hardware_features/TPM/meta/tpm_version//*[text()='2.0']",
      "//host_info/tboot_installed//*[text()='true']"
   ]
```

This sample contains three conditions, each of which must be true for this Template to apply:

os_name: 'RedHatEnterprise'

tpm_version: 2.0

tboot_installed: true

This will apply for RedHat hosts with TPM2.0 and tboot enabled.  If a Flavor is imported from a VMware ESXi host, this template will not apply.  The condition paths directly refer to host-info elements collected from the host.  The full host-info details for a host can be viewed using the /hvs/v2/host-status API; below is a snippet of the host-info section (note that additional host-info elements may be added as new platform features are incorporated):

```json
               "host_info": {
                    "os_name": "RedHatEnterprise",
                    "os_version": "8.3",
                    "os_type": "linux",
                    "bios_version": "SE11111.111.11.11.1111.11111111111",
                    "vmm_name": "",
                    "vmm_version": "",
                    "processor_info": "54 06 05 00 FF FB EB BF",
                    "host_name": "hostname",
                    "bios_name": "Intel Corporation",
                    "hardware_uuid": "<UUID>",
                    "process_flags": "FPU VME DE PSE TSC MSR PAE MCE CX8 APIC SEP MTRR PGE MCA CMOV PAT PSE-36 CLFSH DS ACPI MMX FXSR SSE SSE2 SS HTT TM PBE",
                    "no_of_sockets": "72",
                    "tboot_installed": "false",
                    "is_docker_env": "false",
                    "hardware_features": {
                        "TXT": {
                            "enabled": "true"
                        },
                        "TPM": {
                            "enabled": "true",
                            "meta": {
                                "tpm_version": "2.0"
                            }
                        },
                        "CBNT": {
                            "enabled": "false",
                            "meta": {
                                "profile": "",
                                "msr": ""
                            }
                        },
                        "UEFI": {
                            "enabled": "false",
                            "meta": {
                                "secure_boot_enabled": true
                            }
                        }
                    },
                    "installed_components": [
                        "tagent"
                    ]
                }
```



#### Flavor Parts

This section of the template will define behaviors for each Flavor part.  Each different Flavor part is optional; if the new Template will only affect the OS Flavor part, only the OS Flavor part needs to be defined here.  Each Flavor part specified will have its own "meta" section where conditional host attributes will be defined.  These must match with host-info attributes; for example, in the sample above the OS part uses the following "meta" section elements:

```json
{
 "tpm_version":"2.0",
 "tboot_installed":true
}
```

These directly correspond to host-info elements from the hosts this Template will apply to.

#### PCR Rules

Each Flavor part section of a Flavor Template may contain 0 or more PCR rules that define PCRs to include.  Again using the OS Flavor part example from above, the default Template defines SHA1, SHA256, or SHA384 PCR banks; this tells the HVS to use the "best" available PCR bank algorithm, but that each of these algorithms is acceptable.  Alternatively, if the Template listed only the SHA384 PCR bank, the resulting Flavor would *require* the SHA384 PCR bank and would disregard any SHA256 or SHA1 banks, even if the SHA384 bank is unavailable and the SHA256 bank is enabled on the server.

The PCR Rules will also contain at least one PCR index, indicating which PCR the rule applies to.

##### pcr_matches

PCRs can require a direct PCR value match (when event logs are unnecessary and the final PCR hash is required to match a specific value), and/or can contain event log include/exclude/equals rules.

A direct PCR value match requirement is the easiest definitions, but should only be used when a specific PCR is known to always be the same on all hosts that the resulting Flavor will apply to:

```json
            {
               "pcr":{
                  "bank":[
                     "SHA384",
                     "SHA256",
                     "SHA1"
                  ],
                  "index":0
               },
               "pcr_matches":true
            }
```

This requires the value of PCR0 to exactly match, and will not examine specific event log details for this PCR index.

##### eventlog_includes

The following example shows how to require a specific event log entry to exist:

```json
            {
               "pcr":{
                  "bank":[
                     "SHA384",
                     "SHA256",
                     "SHA1"
                  ],
                  "index":17
               },
               "eventlog_includes":[
                  "vmlinuz"
               ]
            }
```

This rule will require the "vmlinuz" event log measurement to be present in the PCR17 event log.

##### excluding_tags

Specific event logs can also be excluded; in the below example, all events from PCR17 will be part of the resulting Flavor, but will exclude the LCP_CONTROL_HASH, initrd, and vmlinuz measurement events specifically.  This is often used when a specific PCR contains measurements that should apply to different Flavor parts; different rules need to be defined to ensure that the correct events are included in the right Flavor part, and events that will apply for different Flavor parts must be excluded.

```json
            {
               "pcr":{
                  "bank":[
                     "SHA384",
                     "SHA256",
                     "SHA1"
                  ],
                  "index":17
               },
               "eventlog_equals":{
                  "excluding_tags":[
                     "LCP_CONTROL_HASH",
                     "initrd",
                     "vmlinuz"
                  ]
               }
            }
```



Flavor Matching
---------------

Flavors are matched to host by objects called `Flavor Groups` A Flavor
Group represents a set of rules to satisfy for a set of flavors to be
matched to a host for attestation. For example, a Flavor Group can
require that a `PLATFORM` Flavor and an `OS` Flavor be used for attestation.
Without this level of association, a host that matches measurements for
only a `PLATFORM` flavor, for example, can be attested as Trusted, even
though the OS Flavor would attest the host as Untrusted.

Flavor matching can be automatic (the default), or can explicitly
specify a host to which the Flavor Group must apply.

Automatic flavor matching allows for more ease in datacenter lifecycle
management with updates and patches that may cause the appropriate
flavors to change over time. Automatic flavor matching will trigger a
new matching action when a new flavor is added, when an existing flavor
is deleted, or when a host is initially attested as Untrusted. The
system will automatically attempt to find a new set of flavors that
match the Flavor Group rules that will attest the host as Trusted. For
example, if a host in your datacenter has recently had a BIOS update,
the next attestation will cause the host to appear Untrusted (because
the `PLATFORM` measurements will now differ). Using automatic flavor
matching, the Verification Service will automatically search for a new
`PLATFORM` flavor that matches the actual BIOS version and measurement
seen on the host. If a new BIOS version is successfully found, the
Verification Service will use the new version for attestation, and the
host will appear Trusted. If no matching `PLATFORM` flavor is found, the
host will appear Untrusted. When automatic flavor matching is used,
think of the various flavors in the Verification Service as a collection
of valid configurations, and an attested host matching any combination
of those configurations (within the confines of the Flavor Group
requirements for which flavor types must be present) will be attested as
Trusted.

Host-based flavor matching explicitly maps a specific host to a flavor.
Host-based attestation requires that a host saves its entire
configuration in a composite flavor document in the system, and then
later validates against this flavor to detect any changes. In this case,
if a host received a BIOS upgrade, the host will attest as Untrusted,
and no attempt will be made to re-match a new flavor. An administrator
will need to explicitly specify a new flavor to be used for that host.

### When Does Flavor Matching Happen?

Generally speaking, a new Flavor match operation is triggered whenever a
host is registered, whenever a host is attested and would be untrusted,
and whenever a Flavor is added to or removed from a Flavor group.

When a new host is registered, the Verification Service will retrieve
the Host Report and derive the platform information needed for Flavor
matching (BIOS version, server OEM, OS type and version, TPM version,
etc.). The Verification Service then searches through the Flavors in the
same Flavor group that the host is in, and finds any Flavors that match
the platform information.

If a Flavor is deleted, the Verification Service finds any hosts that
are currently associated with that Flavor, and attempts to match them to
alternative Flavors.

If a Flavor is added, the Verification Service looks for any hosts in
the same Flavor group that are not currently matched to a Flavor of the
appropriate Flavor part, and checks to see whether those hosts should be
mapped to the new Flavor.

If a new Report is generated for a host and would not result in a
Trusted attestation, the Verification Service will first repeat the
Flavor matching process to be sure that no matching Flavors exist in the
host’s Flavor group that would result in a Trusted attestation. If the
Service still finds no matching Flavors, the host will appear as
Untrusted.

### Flavor Matching Performance

Flavor matching causes affected hosts to be moved into the `QUEUE` state
while the host and Flavor are evaluated to determine whether the host
and Flavor should be linked. Hosts can remain in the QUEUE state for
varying amounts of time based on the extent of the Flavor match
required. This means that the trust status of a host will not be
actually updated to reflect a new Flavor until after the process
finishes, which may take a few seconds or minutes depending on the
number of registered hosts, Flavors in the same Flavorgroup, etc.

If a new host is registered, only that host will be added to the queue,
and other hosts will be unaffected. The Verification Service will look
for only the `HOST\_UNIQUE` flavor part applicable to that specific host,
and then will look at all PLATFORM and OS Flavors in the same
Flavorgroup has the host, using the Flavor metadata and host info to
narrow the results. The Service will match the new host to the most
similar Flavors, and then move the host to the `CONNECTED` state and
generate a new trust report.

When a new PLATFORM or OS Flavor is created, the Service will instead
add all hosts in the same Flavorgroup as the new Flavors to the queue.
Each host in the queue will then be re-evaluated against every `PLATFORM` and `OS` Flavor in the Flavorgroup to determine the closest match.

This means that adding a new Flavor can cause more hosts to each spend
more time in the QUEUE state, as compared to adding a new host. For this
reason, as a best practice for initial population of Flavors and hosts
for a new deployment, it is suggested that Flavors be created before
registering hosts. This is not a concern after the initial population of
Flavors and hosts.

### Flavor Groups

Flavor Groups represent a collection of one or more Flavors that are
possible matches for a collection of one or more hosts. Flavor Groups
link to both Flavors and hosts – a host in Flavor Group "ABC" will only
be matched to Flavors in Flavor Group "ABC"

### Default Flavor Group

By default the Verification Service includes a Flavor Group named
`automatic` and another named `unique` During host registration, the
`automatic` Flavor Group is used as a default selection if no other
Flavor Group is specified.

#### automatic

The automatic Flavor Group is used as the default Flavor Group for all
hosts and all Flavor parts. If no other Flavor Groups are specified when
creating Flavors or Hosts, all Hosts and Flavors will be added to this
group. This is useful for datacenters that want to manage a single set
of acceptable configurations for all hosts.

#### unique

The unique Flavor Group is used to contain `HOST\_UNIQUE` Flavors. This
Flavorgroup is used by the backend software and should not be managed
manually.

### Flavor Match Policies

Flavor Match Policies are used to define how the Flavor Match engine
will match Flavors to hosts for attestation for a given Flavor Group.
Each Flavor part can have defined Flavor Match Policies within a given
Flavor Group.

```json
{
    "PLATFORM": {
        "any_of",
        "required"
    },
    "OS": {
        "all_of",
        "required_if_defined"
    },
    "HOST_UNIQUE": {
        "latest",
        "required_if_defined"
    },
    "ASSET_TAG": {
        "latest",
        "required_if_defined"
    },
    "SOFTWARE": {
        "all_of",
        "required_if_defined"
    }
}
```

The sample Policy above would require that a `PLATFORM` Flavor part be
matched, but any `PLATFORM` Flavor part in the Flavor Group may be
matched. The `OS` Flavor Part will only be required if there is an `OS` Flavor part in the Flavor Group; if there are no `OS` Flavor parts in the
Group, the match will not be required. If more than one `OS` Flavor part
exists in the Group, all of those `OS` parts will be required to match for
a host to be Trusted.

#### Default Flavor Match Policy

The `automatic` Flavor Group, and any Flavor Group created without
explicitly defining a Flavor Match Policy, will be created using the
following Flavor Match Policy. This is the default behavior for Flavor
Matching:

```json
{
    "PLATFORM": {
        "any_of",
        "required"
    },
    "OS": {
        "any_of",
        "required"
    },
    "HOST_UNIQUE": {
        "latest",
        "required_if_defined"
    },
    "ASSET_TAG": {
        "latest",
        "required_if_defined"
    },
    "SOFTWARE": {
        "all_of",
        "required_if_defined"
    }
}
```

#### ANY\_OF

The `ANY_OF` Policy allows any Flavor of the specified Flavor part to be
matched. If the Flavor Group contains OS Flavor 1 and OS Flavor 2, a
host will be Trusted if it matches either OS Flavor 1 or OS Flavor 2.

#### ALL\_OF

The `ALL_OF` Policy requires all Flavors of the specified Flavor Part in
the Flavor Group to be matched. For example, if Flavor Group X contains
PLATFORM Flavor Part 1 and `PLATFORM` Flavor Part 2, a host in Flavor
Group X will need to match both `PLATFORM` Flavor 1 and `PLATFORM` Flavor 2
to attest as Trusted. If the host matches only one of the Flavors, or
neither of them, the host will be attested as Untrusted.

#### LATEST

The `LATEST` Policy requires that the most recently created Flavor of the
specified Flavor part be used when matching to a host. For example:

```json
"ASSET_TAG": {
    "latest",
    "required_if_defined"
}
```

`ASSET_TAG` Flavor parts by default use the above Policy. This means that
if Asset Tag Flavors are in the Flavor Group, the most recently created
Asset Tag Flavor will be used. If no Asset Tag Flavors are present in
the Flavor Group, then this Flavor part will be ignored.

#### REQUIRED

The `REQUIRED` Policy requires a Flavor of the specified part to be
matched. For example:

```json
"PLATFORM": {
    "any_of",
    "required"
}
```

This policy means that a `PLATFORM` Flavor part must be used; if the
Flavor Group contains no `PLATFORM` Flavor parts, hosts in this Flavor
Group will always count as Untrusted.

#### REQUIRED\_IF\_DEFINED

The `REQUIRED_IF_DEFINED` Policy requires that a Flavor part be used if
a Flavor of that part exists. If no Flavor part of this type exists in
the Flavor Group, the Flavor part will not be required.

```json
"ASSET_TAG": {
    "latest",
    "required_if_defined"
}
```

`ASSET_TAG` Flavor parts by default use the above Policy. This means that
if Asset Tag Flavors are in the Flavor Group, the most recently created
Asset Tag Flavor will be used. If no Asset Tag Flavors are present in
the Flavor Group, then this Flavor part will be ignored.

### Flavor Match Event Triggers

Several events will cause the background queue service to attempt to
re-match Flavors and hosts:

1.  Host registration

    This event is the first time a host will be attempted to be matched to
    appropriate Flavors in the same Flavor Group, and affects only the host
    that was added (other hosts will not be re-matched to Flavors when you
    add a new host).

2.  Flavor creation

    When a new Flavor is added to a Flavor Group, the queue system will
    repeat the Flavor match operation for all hosts in the same Flavor Group
    as the new Flavor.

3.  Flavor deletion

    When a Flavor is deleted, the queue system will repeat the Flavor match
    operation for all hosts in the same Flavor Group as the deleted Flavor.

4.  Creation of a new Attestation Report

    When a new Attestation Report is generated, if the host would attest as
    Untrusted with the currently-matched Flavors, the host being attested
    will be re-matched as part of the Report generation process. This
    ensures that Reports are always generated using the best possible Flavor
    matches available in the database.

### Sample Flavorgroup API Calls

#### Create a New Flavorgroup

```
POST https://<Verification Service IP or Hostname>:8443/hvs/v2/flavorgroups
Authorization: Bearer <token>

{
    "name": "firstTest",
    "flavor_match_policy_collection": {
        "flavor_match_policies": [
            {
                "flavor_part": " PLATFORM",
                "match_policy": {
                    "match_type": "ANY_OF",
                    "required": "REQUIRED"
                }
            }
        ]
    }
}
```

Response:

```json
{
    "id": "a0950923-596b-41f7-b9ad-09f525929ba1",
    "name": "firstTest",
    "flavor_match_policy_collection": {
        "flavor_match_policies": [
            {
                "flavor_part": " PLATFORM",
                "match_policy": {
                    "match_type": "ANY_OF",
                    "required": "REQUIRED"
                }
            }
        ]
    }
}
```





SOFTWARE Flavor Management
--------------------------

### What is a SOFTWARE Flavor?

A `SOFTWARE` Flavor part defines the measurements expected for a specific
application, or a specific set of files and folders on the physical
host. `SOFTWARE` Flavors can be used to attest the boot-time integrity of
any static files or folders on a physical server.

A single server can have multiple SOFTWARE Flavors associated. Intel®
SecL-DC provides a `default` SOFTWARE Flavor that is deployed to each
Trust Agent server during the provisioning step. This default Flavor
includes the static files and folders of the Trust Agent itself, so that
the Trust Agent is measured during the server boot process, and its
integrity is included in the attestation of the other server
measurements.

Using `SOFTWARE` Flavors consists of two parts – creating the actual
`SOFTWARE` Flavor, and deploying the `SOFTWARE` Flavor manifest to the host.

### Creating a SOFTWARE Flavor part

Creating a new `SOFTWARE` Flavor requires creating a manifest of the files
and folders that need to be measured.

There are three different types of entries for the manifest:
`Directories`, `Symlinks`and `Files`.

#### Directories

A Directory defines measurement rules for measuring a directory.
Effectively this involves listing the contents of the directory and
hashing the results; in this way, a Directory measurement can verify
that no files have been added or removed from the directory specified,
but will not measure the integrity of individual files (ie, files can
change within the directory, but cannot be renamed, added, or removed).

Directory entries can use regular expressions to define explicit Include
and Exclude filters. For example, `Exclude=\*.log` would exclude all
files ending with .log from the measurement, meaning files with the .log
extension can be added or removed from the directory.

```xml
<Dir Type="dir" Include=".*" Exclude="" Path="/opt/trustagent/hypertext/WEB-INF">
```

#### Symlinks

A Symlink entry defines a symbolic link that will be measured. The
actual symbolic link is hashed, not the file or folder the symlink
points to. In this way, the measurement will detect the symbolic link
being modified to point to a different location, but the actual file or
folder pointed to can have its contents change.

```xml
<Symlink Path="/opt/trustagent/bin/tpm_nvinfo">
```

#### Files

Individual files can be explicitly specified for measurement as well.
Each file listed will be hashed and extended separately. This means that
if any file explicitly listed this way changes its contents or is
deleted or moved, the measurement will change, and the host will become
Untrusted.

```xml
<File Path="/opt/trustagent/bin/module_analysis_da.sh">
```

### Sample SOFTWARE Flavor Creation Call

Creating a new `SOFTWARE` Flavor requires specifying a sample host where
the application, files or folders that will be measured are currently
present. The measurements specified in the manifest will be captures
when this call is executed, and the Verification Service will
communicate with the Trust Agent and create a `SOFTWARE` Flavor based on
the file measurements.

The Connection String must point to the sample Trust Agent host. The
Label defines the name of the new Flavor (ideally this should be the
name of the application being measured for easier management).

```
POST https://<Verification Service IP or Hostname>:8443/hvs/v2/flavor-from-app-manifest
Authorization: Bearer <token>

<ManifestRequest xmlns="lib:wml:manifests-req:1.0">
    <connectionString>intel:https://trustagent.server.com:1443;u=trustagentUsername;p=trustagentPassword</connectionString>
    <Manifest xmlns="lib:wml:manifests:1.0" DigestAlg="SHA384" Label="Tomcat" Uuid="">+
        <Dir Type="dir" Include=".*" Exclude="" Path="/opt/trustagent/hypertext/WEB-INF" />
        <Symlink Path="/opt/trustagent/bin/tpm_nvinfo" />
        <File Path="/opt/trustagent/bin/module_analysis_da.sh" />
    </Manifest>
</ManifestRequest>
```

### Deploying a SOFTWARE Flavor Manifest to a Host

Once the SOFTWARE Flavor has been created, it can be deployed to any
number of Trust Agent servers. This requires the Flavor ID (returned
from Flavor creation) and the Host ID (returned from host registration).
The Verification Service will send a request to the appropriate Trust
Agent and create the manifest.

> **Note**: After the SOFTWARE Flavor manifest is deployed to a host,
> the host **must** be rebooted. This will allow the measurements
> specified in the Flavor to be taken and extended to the TPM. Until the
> host is rebooted, the host will now appear Untrusted, as it now
> requires measurements from a SOFTWARE Flavor that have not yet been
> extended to the TPM.

```
POST https://<Verification Service IP or Hostname>:8443/hvs/v2/rpc/deploy-software-manifest
Authorization: Bearer <token>

{  
   "flavor_id":"a6544ff4-6dc7-4c74-82be-578592e7e3ba",  
   "host_id":"a6544ff4-6dc7-4c74-82be-578592e7e3ba"
}
```

### SOFTWARE Flavor Matching

The default Flavor Match Policy for SOFTWARE Flavor parts is
`ALL_OF`,`REQUIRED_IF_DEFINED`. This means that all Software Flavors
defined in a Flavorgroup must match to all hosts in that Flavorgroup. If
no SOFTWARE Flavors are in the Flavorgroup, then hosts can still be
considered Trusted.

Because the default uses the `ALL_OF` Policy, it’s recommended to use
Flavorgroups dedicated to specific software loadouts. For example, if a
number of hosts will act as virtualization hosts and will have `SOFTWARE`
Flavors for the hypervisor and VM management applications, those hosts
should be placed in their own Flavorgroup as they will all run similar
or identical application loadouts. If another group of servers in the
datacenter will act as container hosts, these hosts might need `SOFTWARE` Flavors that include attestation of container runtimes and management
applications, and will have a very different application loadout from
the VM-based hosts. These should be placed in their own Flavorgroup, so
that the VM hosts are attested using the hypervisor-related `SOFTWARE` Flavors, and the container hosts are attested using the
container-related `SOFTWARE` Flavors.

As with other Flavor parts, hosts will be matched to Flavors in the same
Flavorgroup that the host is added to, and will not be matched to
Flavors in different Flavorgroups. Flavor matching will happen on the
same events as for other Flavor parts.

### Kernel Upgrades

Because the Application Integrity functionality involves adding a
measurement agent (`tbootXM`) to `initrd`, an additional process must be
followed when updating the OS kernel to ensure the new initrd also
contains the measurement agent. This is not required if Application
Integrity will not be used.

1.  Update grub to have the boot menu-entry created for the new kernel
    version in grub.cfg (`grub2-mkconfig -o \<path to grub file\>`)

2.  Reboot the host and boot into new kernel menu-entry.

3.  Generate a new initrd with tbootXM.
    (`/opt/tbootxm/bin/generate\_initrd.sh`)

4.  Copy the generated initrd to the boot drectory. (`cp
    /var/tbootxm/\<generated initrd file name\> /boot/`)

5.  Update the `TCB protection` menu-entry with the new kernel version.

    1.  Source `rustagent.env`, or

        ```json
        export GRUB_FILE=/boot/efi/EFI/redhat/grub.cfg
        ```

    2.  Run the configure\_host script:

        ```shell
        cd /opt/tbootxm/bin  
        ./configure_host.sh
        ```

6. Update the default boot menu-entry to have new kernel version. (edit
   `/etc/default/grub`)

7. Update the grub to reflect the updates. (`grub2-mkconfig -o \<path to
   grub file\>`)

8. Reboot the host and boot into TCB protection menu-entry.

After updating the system with the new `initrd`, the Software Flavor
should attest as Trusted. Note that changing `grub` and `initrd` does result
in a new `OS` Flavor measurements, so an updated `OS` Flavor should be
imported after updating the kernel and regenerating `initrd`.



Scalability and Sizing
======================

Configuration Maximums
----------------------

### Registered Hosts

The Intel® SecL Verification Service can support a maximum of 2000
registered hosts with a single Verification Service instance with
default settings and the specified minimum required hardware.

The Verification Service can support up to 100,000 registered hosts, but will require a minimum of 16 vCPUs and 16GB RAM.  The service is primarily CPU-limited.

### HDD Space

The HDD space recommendations below represent expected log and database
growth using default settings. Altering the database or log rotation
settings, or the SAML expiration setting, may change the amount of disk
space required. For default settings, 100 GB of disk space is
recommended.



Database Rotation Settings
--------------------------

The Intel® SecL Verification Service database will automatically rotate
the audit log table after one million records, and will retain up to ten
total rotations. These settings are user-configurable if a longer
retention period is needed.

**mtwilson.audit.log.num.rotations** - defines the maximum number of
rotations before the oldest rotation is deleted to make space for a new
rotation.

**mtwilson.audit.log.max.row.count** – defines the maximum number of
rows in the audit log table before a rotation will occur.



Log Rotation
------------

The Intel® SecL services (the Verification Service, Trust Agent, and
Integration Hub) use Logrotate to rotate logs automatically during a
daily cron job.

By default, logs are rotated once per month or when they exceed 1 GB in
size, whichever comes first, and 12 total rotations will be retained.



Intel Security Libraries Configuration Settings
===============================================

Verification Service
--------------------

### Installation Answer File Options

```
# Authentication URL and service account credentials - mandatory
AAS_API_URL=https://isecl-aas:8444/aas/v1
HVS_SERVICE_USERNAME=HVS_service
HVS_SERVICE_PASSWORD=password

# CMS URL and CMS webserivce TLS hash for server verification - mandatory
CMS_BASE_URL=https://isecl-cms:8445/cms/v1
CMS_TLS_CERT_SHA384=digest

# Installation admin bearer token for CSR approval request to CMS - mandatory
BEARER_TOKEN=eyJhbGciOiJSUzM4NCIsImtpZCI6ImE…

# Skip setup - optional
HVS_NOSETUP=false #default=false

# Logging options - optional
HVS_LOGLEVEL=info              # options: critical|error|warning|info|debug|trace, default='info'
HVS_LOG_MAX_LENGTH=300         # default=300
HVS_ENABLE_CONSOLE_LOG=false   # default=false

# HRRS configuration - optional
HRRS_REFRESH_PERIOD=2m0s       # default=2m0s
HRRS_REFRESH_LOOK_AHEAD=5m0s   # default=5m0s

# FVS configuration - optional
FVS_NUMBER_OF_VERIFIERS=20                    # default=20
FVS_NUMBER_OF_DATA_FETCHERS=20                # default=20
FVS_SKIP_FLAVOR_SIGNATURE_VERIFICATION=false  # default=false

# In case of trusted flavor storage, flavor signature verification can be skipped
# using following flag - optional
SKIP_FLAVOR_SIGNATURE_VERIFICATION=false # default=false

# TLS certificate configuration - optional
TLS_COMMON_NAME="HVS TLS Certificate"      # default="HVS TLS Certificate"
TLS_SAN_LIST=127.0.0.1,localhost           # default=127.0.0.1,localhost

# Server configuration - optional
HVS_PORT=8443                        # default=8443
HVS_SERVER_READ_TIMEOUT=30s          # default=30s
HVS_SERVER_READ_HEADER_TIMEOUT=10s   # default=10s
HVS_SERVER_WRITE_TIMEOUT=10s         # default=10s
HVS_SERVER_IDLE_TIMEOUT=10s          # default=10s
HVS_SERVER_MAX_HEADER_BYTES=1048576  # default=1048576

# Database - mandatory
HVS_DB_USERNAME=runner
HVS_DB_PASSWORD=test
HVS_DB_SSLCERTSRC=/tmp/dbcert.pem          # This doesn't need to be specified if HVS_DB_SSLCERT is given

# Database - optional
HVS_DB_HOSTNAME=localhost                  # default=localhost
HVS_DB_NAME=hvs-pg-db                      # default=hvs-pg-db
HVS_DB_PORT=5432                           # default=5432
HVS_DB_SSLMODE=verify-full                 # default=verify-full ;other options are like allow, prefer, require, verify-ca
HVS_DB_SSLCERT=/etc/hvs/hvsdbcert.pem      # default=/etc/hvs/hvsdbcert.pem

# Webservice configuration - Optional
HVS_PORT=8443
HVS_SERVER_READ_TIMEOUT=30s
HVS_SERVER_READ_HEADER_TIMEOUT=10s
HVS_SERVER_WRITE_TIMEOUT=10s
HVS_SERVER_IDLE_TIMEOUT=10s
HVS_SERVER_MAX_HEADER_BYTES=1048576

# Logging - Optional
HVS_LOG_MAX_LENGTH=300
HVS_ENABLE_CONSOLE_LOG=false

# Flavor Signing Configuration - Optional
FLAVOR_SIGNING_KEY_FILE=/etc/hvs/trusted-keys/flavor-signing.key
FLAVOR_SIGNING_CERT_FILE=/etc/hvs/certs/trustedca/flavor-signing.pem
FLAVOR_SIGNING_COMMON_NAME=HVS Flavor Signing Certificate

# SAML Configuration - Optional
SAML_KEY_FILE=/etc/hvs/trusted-keys/saml.key
SAML_CERT_FILE=/etc/hvs/certs/trustedca/saml-cert.pem
SAML_COMMON_NAME=HVS SAML Certificate

# Endorsement CA Configuration - Optional
ENDORSEMENT_CA_KEY_FILE=/etc/hvs/trusted-keys/endorsement-ca.key
ENDORSEMENT_CA_CERT_FILE=/etc/hvs/certs/trustedca/EndorsementCA.pem
ENDORSEMENT_CA_COMMON_NAME=HVS Endorsement Certificate
ENDORSEMENT_CA_ISSUER=intel-secl
ENDORSEMENT_CA_VALIDITY_YEARS=5

# Privacy CA Configuration - Optional
PRIVACY_CA_KEY_FILE=/etc/hvs/trusted-keys/privacy-ca.key
PRIVACY_CA_CERT_FILE=/etc/hvs/certs/trustedca/privacy-ca-cert.pem
PRIVACY_CA_COMMON_NAME=HVS Privacy Certificate
PRIVACY_CA_ISSUER=intel-secl
PRIVACY_CA_VALIDITY_YEARS=5

# Asset Tag Configuration - Optional
TAG_CA_KEY_FILE=/etc/hvs/trusted-keys/tag-ca.key
TAG_CA_CERT_FILE=/etc/hvs/certs/trustedca/tag-ca-cert.pem
TAG_CA_COMMON_NAME=HVS Tag Certificate
TAG_CA_ISSUER=intel-secl
TAG_CA_VALIDITY_YEARS=5

# Certificate Revocation Checks - Optional
ENABLE_EKCERT_REVOKE_CHECK=false # default=false - if true, revocation checks will be performed on EK certs
                                 # proxy settings (if applicable) must be provided - see next stion

```

### Note on Certificate Revocation Checks for TPM EK Certs

The ENABLE_EKCERT_REVOKE_CHECK setting has been added to toggle revocation checks for Endorsement Key Certs by Verification Service. The revocation check will be performed during the course of the AIK provisioning flow on the Trust Agent (`tagent setup provision-aik`).

At this stage, there are 3 possible outcomes:

1. No certs in the chain are found to be revoked: VS responds with a HTTP 200 response code
2. A certificate is found to be revoked: VS responds with a HTTP 400 response code
3. The revocation check failed due to a connection error: VS responds with a HTTP 500 response code 

If proxy settings are required to source CRL from external CAs, these can be added to the Verification Service service systemd via a drop-in config at /etc/systemd/system/hvs.service.d/proxy.conf or directly to `/opt/hvs/hvs.service` under the service section like so:

```
[Service]
Environment="https_proxy=<proxy server url>"
```

Run `systemctl daemon-reload` and restart the service after making this change.

### Configuration Options

The Verification Service configuration is stored in the
file `/etc/hvs/config.yml`:

```
tls:
  cert-file: /etc/hvs/tls-cert.pem
  key-file: /etc/hvs/tls.key
  common-name: HVS TLS Certificate
  san-list: 127.0.0.1,localhost
saml:
  common:
    cert-file: /etc/hvs/certs/trustedca/saml-cert.pem
    key-file: /etc/hvs/trusted-keys/saml.key
    common-name: HVS SAML Certificate
  issuer: AttestationService
  validity-days: 1
flavor-signing:
  cert-file: /etc/hvs/certs/trustedca/flavor-signing.pem
  key-file: /etc/hvs/trusted-keys/flavor-signing.key
  common-name: HVS Flavor Signing Certificate
privacy-ca:
  cert-file: /etc/hvs/certs/trustedca/privacy-ca/privacy-ca-cert.pem
  key-file: /etc/hvs/trusted-keys/privacy-ca.key
  common-name: HVS Privacy Certificate
  issuer: intel-secl
  validity-years: 5
endorsement-ca:
  cert-file: /etc/hvs/certs/endorsement/EndorsementCA.pem
  key-file: /etc/hvs/trusted-keys/endorsement-ca.key
  common-name: HVS Endorsement Certificate
  issuer: intel-secl
  validity-years: 5
tag-ca:
  cert-file: /etc/hvs/certs/trustedca/tag-ca-cert.pem
  key-file: /etc/hvs/trusted-keys/tag-ca.key
  common-name: HVS Tag Certificate
  issuer: intel-secl
  validity-years: 5
aik-certificate-validity-years: 5
server:
  port: 8898
  read-timeout: 30s
  read-header-timeout: 10s
  write-timeout: 30s
  idle-timeout: 10s
  max-header-bytes: 1048576
log:
  max-length: 30000
  enable-stdout: true
  level: TRACE
db:
  vendor: postgres
  host: localhost
  port: "5432"
  name: hvs_db
  username: root
  password: password
  ssl-mode: allow
  ssl-cert: /etc/hvs/hvsdbsslcert.pem
  conn-retry-attempts: 5
  conn-retry-time: 1
hrrs:
  refresh-period: 2m0s
  refresh-look-ahead: 5m0s
fvs:
  number-of-verifiers: 20
  number-of-data-fetchers: 20
  skip-flavor-signature-verification: true
enable-ekcert-revoke-check: false
```


### Command-Line Options

The Verification Service supports several command-line commands that can
be executed only as the Root user:

Syntax:

`hvs <command>`

#### Help

`hvs help`

Displays the list of available CLI commands.

#### Start

`hvs start`

Starts the services.

#### Stop

`hvs stop`

Stops the services.

#### Status

`hvs status`

Reports whether the service is currently running.

#### Uninstall

`hvs uninstall`

Uninstalls the service, including the deletion of all files and folders.
Database content is not removed. See section 14.1 for additional
details.

#### Version

`hvs version`

Reports the version of the service.

#### Erase-data

`hvs erase-data`

Deletes all non-user information from the database.  All data in the following tables will be deleted; the database schema will be preserved:

```
 flavor_host 
 flavor_flavorgroup
 flavorgroup_host
 queue
 report
 host_status
 flavorgroup
 host
 flavor
 host_credential
 tag_certificate
 audit_log_entry
 tls_policy
```

#### Setup

Usage of hvs setup:
        hvs setup <task> [--help] [--force] [-f <answer-file>]
                --help                      show help message for setup task
                --force                     existing configuration will be overwritten if this flag is set
                -f|--file <answer-file>     the answer file with required arguments

Available Tasks for setup:
        all                             Runs all setup tasks
        database                        Setup hvs database
        create-default-flavorgroup      Create default flavor groups in database
        create-dek                      Create data encryption key for HVS
        download-ca-cert                Download CMS root CA certificate
        download-cert-tls               Download CA certificate from CMS for tls
        download-cert-saml              Download CA certificate from CMS for saml
        download-cert-flavor-signing    Download CA certificate from CMS for flavor signing
        create-endorsement-ca           Generate self-signed endorsement certificate
        create-privacy-ca               Generate self-signed privacy certificate
        create-tag-ca                   Generate self-signed tag certificate
        update-service-config           Sets or Updates the Service configuration

### Directory Layout

The Host Verification Service installs by default to the following folders:

/etc/hvs/

This directory contains the config.yml configuration file, the database connection ssl cerificate, and the webservice TLS certificate.

/etc/hvs/

├── certs

│   ├── endorsement

│   │   ├── EndorsementCA-external.pem

│   │   └── EndorsementCA.pem

│   ├── trustedca

│   │   ├── flavor-signing.pem

│   │   ├── privacy-ca

│   │   │   └── privacy-ca-cert.pem

│   │   ├── root

│   │   │   ├── 58f6bcfcd.pem

│   │   │   ├── vmware-cert1.pem

│   │   │   └── vmware-cert2.pem

│   │   ├── saml-cert.pem

│   │   └── tag-ca-cert.pem

│   └── trustedjwt

│       └── f29aa4ab3.pem

├── config.yml

├── hvsdbsslcert.pem

├── tls-cert.pem

├── tls.key

└── trusted-keys

​    ├── endorsement-ca.key

​    ├── flavor-signing.key

​    ├── privacy-ca.key

​    ├── saml.key

​    └── tag-ca.key

Trust Agent
-----------

### Installation Answer File Options

| Key                               | Description                                                  | Sample Value                                                 |
| --------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| AAS\_API\_URL                     | API URL for Authentication Authorization Service (AAS).      | AAS\_API\_URL=https://{host}:{port}/aas/v1                   |
| AUTOMATIC\_PULL\_MANIFEST         | Instructs the installer to automatically pull application-manifests from HVS similar to tagent setup get-configured-manifest | AUTOMATIC\_PULL\_MANIFEST=Y                                  |
| AUTOMATIC\_REGISTRATION           | Instructs the installer to automatically register the host with HVS similar to running tagent setup create-host and tagent setup create-host-unique-flavor. | AUTOMATIC\_REGISTRATION=Y                                    |
| BEARER\_TOKEN                     | JWT from AAS that contains "install" permissions needed to access ISecL services during provisioning and registration | BEARER\_TOKEN=eyJhbGciOiJSUzM4NCIsjdkMTdiNmUz...             |
| CMS\_BASE\_URL                    | API URL for Certificate Management Service (CMS).            | CMS\_BASE\_URL=https://{host}:{port}/cms/v1                  |
| CMS\_TLS\_CERT\_SHA384            | SHA384 Hash sum for verifying the CMS TLS certificate.       | CMS\_TLS\_CERT\_SHA384=bd8ebf5091289958b5765da4...           |
| HVS\_API\_URL                     | The url used during setup to request information from HVS.   | HVS\_API\_URL=https://{host}:{port}/hvs/v2                   |
| PROVISION\_ATTESTATION            | When present, enables/disables whether tagent setup is called during installation. If trustagent.env is not present, the value defaults to no ('N'). | PROVISION\_ATTESTATION=Y                                     |
| SAN\_LIST                         | CSV list that sets the value for SAN list in the TA TLS certificate. Defaults to 127.0.0.1. | SAN\_LIST=10.123.100.1,201.102.10.22,mya.example.com         |
| TA\_TLS\_CERT\_CN                 | Sets the value for Common Name in the TA TLS certificate. Defaults to CN=trustagent. | TA\_TLS\_CERT\_CN=Acme Trust Agent 007                       |
| TPM\_OWNER\_SECRET                | Default is null.  Can be any string of characters.  Use the "hex:" prefix to force hex characters rather than a string.<br />hex:0164837f83..." | TPM\_OWNER\_SECRET=625d6...<br />Starting in Intel SecL-DC 4.0, this value will now default to null unless a secret is specified.  Using a null TPM ownership secret is recommended.  The Trust Agent now only requires TPM ownership during Trust Agent provisioning. |
| TPM\_QUOTE\_IPV4                  | When enabled (=y), uses the local system's ip address as a salt when processing a quote nonce. This field must align with the configuration of HVS. | TPM\_QUOTE\_IPV4=no                                          |
| TA\_SERVER\_READ\_TIMEOUT         | Sets tagent server ReadTimeout. Defaults to 30 seconds.      | TA\_SERVER\_READ\_TIMEOUT=30                                 |
| TA\_SERVER\_READ\_HEADER\_TIMEOUT | Sets tagent server ReadHeaderTimeout. Defaults to 30 seconds. | TA\_SERVER\_READ\_HEADER\_TIMEOUT=10                         |
| TA\_SERVER\_WRITE\_TIMEOUT        | Sets tagent server WriteTimeout. Defaults to 10 seconds.     | TA\_SERVER\_WRITE\_TIMEOUT=10                                |
| TA\_SERVER\_IDLE\_TIMEOUT         | Sets tagent server IdleTimeout. Defaults to 10 seconds.      | TA\_SERVER\_IDLE\_TIMEOUT=10                                 |
| TA\_SERVER\_MAX\_HEADER\_BYTES    | Sets tagent server MaxHeaderBytes. Defaults to 1MB(1048576)  | TA\_SERVER\_MAX\_HEADER\_BYTES=1048576                       |
| TA\_ENABLE\_CONSOLE\_LOG          | When set true, tagent logs are redirected to stdout. Defaults to false | TA\_ENABLE\_CONSOLE\_LOG=true                                |
| TRUSTAGENT\_LOG\_LEVEL            | The logging level to be saved in config.yml during installation ("trace", "debug", "info"). | TRUSTAGENT\_LOG\_LEVEL=debug                                 |
| TRUSTAGENT\_PORT                  | The port on which the trust-agent service will listen.       | TRUSTAGENT\_PORT=10433                                       |

### Configuration Options

The Trust Agent configuration settings are managed in
`/opt/trustagent/configuration/config.yml`

| **Setting**                                   | **Description**                                              |
| --------------------------------------------- | ------------------------------------------------------------ |
| tpmquoteipv4: true                            | When enabled, the Trust Agent will perform an additional hash of the nonce using the bytes from the Trust Agent server IP when returning TPM quotes. This should always be set to True. |
| logging:                                      |                                                              |
| loglevel: info                                | Defines the Trust Agent logging level                        |
| logenablestdout: false                        | If set to True, the Trust Agent will log to stdout. By default this is False and the logs are sent to /var/log/trustagent/trustagent.log |
| logentrymaxlength: 300                        | Defines the maximum length of a single log entry             |
| webservice:                                   |                                                              |
| port: 1443                                    | Defines the port on which the Trust Agent API server will listen |
| readtimeout: 30s                              |                                                              |
| readheadertimeout: 10s                        |                                                              |
| writetimeout: 10s                             |                                                              |
| idletimeout: 10s                              |                                                              |
| maxheaderbytes: 1048576                       |                                                              |
| hvs:                                          |                                                              |
| url: https://0.0.0.0:8443/hvs/v2              | Defines the baseurl for the Verification Service             |
| tpm:                                          |                                                              |
| aas:                                          |                                                              |
| baseurl: https://0.0.0.0:8444/aas/v1/         | Defines the base URL for the AAS                             |
| cms:                                          |                                                              |
| baseurl: https://0.0.0.0:8445/cms/v1          | Defines the base URL for the CMS                             |
| tlscertdigest: 330086b3...ae477c8502          | Defines the SHA383 hash of the CMS TLS certificate           |
| tls:                                          |                                                              |
| certsan: 10.1.2.3,server.domain.com,localhost | Comma-separated list of hostnames and IP addresses for the Trust Agent. Used in the Agent TLS certificate. |
| certcn: Trust Agent TLS Certificate           | Common Name for the Trust Agent TLS certificate              |

### Command-Line Options

Usage:

  tagent <command> [arguments]

Available Commands:

  help|-h|-help                    Show this help message.
  setup [all] [task]               Run setup task.
  uninstall                        Uninstall trust agent.
  version                          Print build version info.
  start                            Start the trust agent service.
  stop                             Stop the trust agent service.
  status                           Get the status of the trust agent service.
  fetch-ekcert-with-issuer         Print Tpm Endorsement Certificate in Base64 encoded string along with issuer

Setup command usage:  tagent setup [cmd] [-f <env-file>]

Available Tasks for 'setup', all commands support env file flag

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

### Directory Layout

#### Linux

The Linux Trust Agent installs by default to `/opt/trustagent`, with the
following subfolders:

##### Bin

Contains executables and scripts.

##### Configuration

Contains the `config.yml` configuration file, as well as certificates and
keystores. This includes the AIK public key blob after provitioning.

##### Var

Contains information gathered from the platform and SOFTWARE Flavor
manifests. All files with the name `manifest_*.xml` will be parsed to
define measurements during boot. Generally these should be automatically
provisioned from the Verification Service when creating/deploying
SOFTWARE Flavors.



Integration Hub
---------------

### Installation Answer File

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



### Configuration Options

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

### Command-Line Options

#### Available Commands

##### Help

`ihub -h | --help`

Displays the list of available CLI commands.

##### Start

`ihub start`

Starts the services.

##### Stop

`ihub stop`

Stops the services.

##### Status

`ihub status`

Reports whether the service is currently running.

##### Uninstall

`ihub uninstall [--purge]`

Uninstalls the service, including the deletion of all files and folders.
Database content is not removed. If the `--purge` option is used, database
content will be removed during the uninstallation.

##### Version

`ihub -v | --version`

Reports the version of the service.

##### Setup

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



Certificate Management Service
------------------------------

### Installation Answer File Options

| Key                                | Sample Value                                               | Description                                                  |
| ---------------------------------- | ---------------------------------------------------------- | ------------------------------------------------------------ |
| CMS\_NOSETUP                       | false                                                      | Determines whether “setup” will be executed after installation. Typically this is set to “false” to install and perform setup in one action. The “true” option is intended for building the service as a container, where the installation would be part of the image build, and setup would be performed when the container starts for the first time to generate any persistent data. |
| CMS\_PORT                          | 8445                                                       | Defines the HTTPS port the service will use.                 |
| AAS\_API\_URL                      | https://\<Hostname or IP address of the AAS\>:8444/aas/v1/ | URL to connect to the AAS, used during setup for authentication. |
| AAS\_TLS\_SAN                      | \<Comma-separated list of IPs/hostnames for the AAS\>      | SAN list populated in special JWT token, this token is used by AAS to get TLS certificate signed from CMS. SAN list in this token and CSR generated by AAS must match. |
| LOG\_ROTATION\_PERIOD              | hourly, daily, weekly, monthly, yearly                     | log rotation period, for more details refer- https://linux.die.net/man/8/logrotate |
| LOG\_COMPRESS                      | Compress                                                   | Old versions of log files are compressed with gzip, for more details refer- https://linux.die.net/man/8/logrotate |
| LOG\_DELAYCOMPRESS                 | delaycompress                                              | Postpone compression of the previous log file to the next rotation cycle, for more details refer- https://linux.die.net/man/8/logrotate |
| LOG\_COPYTRUNCATE                  | Copytruncate                                               | Truncate the original log file in place after creating a copy,'create' creates new one, for more details refer- https://linux.die.net/man/8/logrotate |
| LOG\_SIZE                          | 1K                                                         | Log files are rotated when they grow bigger than size bytes, for more details refer- https://linux.die.net/man/8/logrotate |
| LOG\_OLD                           | 12                                                         | Log files are rotated count times before being removed, for more details refer- https://linux.die.net/man/8/logrotate |
| CMS\_CA\_CERT\_VALIDITY            | 5                                                          | CMS Root Certificate Validity in years                       |
| CMS\_CA\_ORGANIZATION              | INTEL                                                      | CMS Certificate Organization                                 |
| CMS\_CA\_LOCALITY                  | US                                                         | CMS Certificate locality                                     |
| CMS\_CA\_PROVINCE                  | CA                                                         | CMS Certificate province                                     |
| CMS\_CA\_COUNTRY                   | USA                                                        | CMS Certificate country                                      |
| CMS\_TLS\_SAN\_LIST                |                                                            | Comma-separated list of IP addresses and hostnames to be added to the SAN list of CMS server |
| CMS\_SERVER\_READ\_TIMEOUT         | 30s                                                        | MS server - ReadTimeout is the maximum duration for reading the entire request, including the body. |
| CMS\_SERVER\_READ\_HEADER\_TIMEOUT | 10s                                                        | CMS server - ReadHeaderTimeout is the amount of time allowed to read request headers |
| CMS\_SERVER\_WRITE\_TIMEOUT        | 10s                                                        | CMS server - WriteTimeout is the maximum duration before timing out writes of the response. |
| CMS\_SERVER\_IDLE\_TIMEOUT         | 10s                                                        | CMS server - IdleTimeout is the maximum amount of time to wait for the next request when keep-alives are enabled. |
| CMS\_SERVER\_MAX\_HEADER\_BYTES    | 1048576                                                    | CMS server - MaxHeaderBytes controls the maximum number of bytes the server will read parsing the request header's keys and values, including the request line. |
| AAS\_JWT\_CN                       | AAS JWT Signing Certificate                                | CN of AAS JWT certificate, this gets populated in special JWT token. AAS must send JWT certificate CSR with this CN. |
| AAS\_TLS\_CN                       | AAS TLS Certificate                                        | CN of AAS TLS certificate, this gets populated in special JWT token. AAS must send TLS certificate CSR with this CN. |
| AAS\_TLS\_SAN                      |                                                            | SAN list populated in special JWT token, this token is used by AAS to get TLS certificate signed from CMS. SAN list in this token and CSR generated by AAS must match. |

### Configuration Options

The CMS configuration can be found in `/etc/cms/config.yml`

```yaml
port: 8445
loglevel: info
authserviceurl: https://<AAS IP or hostname>:8444/aas/v1/
cacertvalidity: 5
organization: INTEL
locality: SC
province: CA
country: US
keyalgorithm: rsa
keyalgorithmlength: 3072
rootcacertdigest: <sha384>
tlscertdigest: <sha384>
tokendurationmins: 20
aasjwtcn: ""
aastlscn: ""
aastlssan: ""
authdefender:
  maxattempts: 5
  intervalmins: 5
  lockoutdurationmins: 15
```

### Command-Line Options

#### Help

`cms help`

Displays the list of available CLI commands.

#### Start

`cms start`

Starts the services.

#### Stop

`cms stop`

Stops the service.

#### Status

`cms status`

Reports whether the service is currently running.

#### Uninstall

`cms uninstall`

Uninstalls the service, including the deletion of all files and folders.

#### Version

`cms version`

Reports the version of the service.

#### Tlscertsha384

Shows the SHA384 of the TLS certificate.

#### setup \[task\]

Usage of cms setup:
        cms setup <task> [--help] [--force] [-f <answer-file>]
                --help                      show help message for setup task
                --force                     existing configuration will be overwritten if this flag is set
                -f|--file <answer-file>     the answer file with required arguments

Available Tasks for setup:
    all                       Runs all setup tasks
    root-ca                   Creates a self signed Root CA key pair in /etc/cms/root-ca/ for quality of life
    intermediate-ca           Creates a Root CA signed intermediate CA key pair(signing, tls-server and tls-client) in /etc/cms/intermediate-ca/ for quality of life
    tls                       Creates an intermediate-ca signed TLS key pair in /etc/cms for quality of life
    cms-auth-token            Create its own self signed JWT key pair in /etc/cms/jwt for quality of life
    update-service-config     Sets or Updates the Service configuration

### Directory Layout

The Certificate Management Service installs by default to `/opt/cms` with
the following folders.

#### Bin

This folder contains executable scripts.

#### Cacerts

This folder contains the CMS root CA certificate.



Authentication and Authorization Service
----------------------------------------

### Installation Answer File Options

| Key                             | Sample Value                                 | Description                                                  |
| ------------------------------- | -------------------------------------------- | ------------------------------------------------------------ |
| CMS\_BASE\_URL                  | https://<cms IP or hostname\>/cms/v1/        | Required; Provides the URL for the CMS.                      |
| AAS\_NOSETUP                    | false                                        | Optional. Determines whether “setup” will be executed after installation. Typically this is set to “false” to install and perform setup in one action. The “true” option is intended for building the service as a container, where the installation would be part of the image build, and setup would be performed when the container starts for the first time to generate any persistent data. |
| AAS\_DB\_HOSTNAME               | localhost                                    | Required. Hostname or IP address of the AAS database         |
| AAS\_DB\_PORT                   | 5432                                         | Required. Database port number                               |
| AAS\_DB\_NAME                   | pgdb                                         | Required. Database name                                      |
| AAS\_DB\_USERNAME               | dbuser                                       | Required. Database username                                  |
| AAS\_DB\_PASSWORD               | dbpassword                                   | Required. Database password                                  |
| AAS\_DB\_SSLMODE                | verify-ca                                    | Defines the SSL mode for the connection to the database. If not specified, the database connection will not use certificate verification. If specified, certificate verification will be required for database connections. |
| AAS\_DB\_SSLCERTSRC             | /usr/local/pgsql/data/server.crt             | Optional, required if the`“AAS_DB_SSLMODE` is set to `verify-ca` Defines the location of the database SSL certificate. |
| AAS\_DB\_SSLCERT                | \<path\_to\_cert\_file\_on\_system\>         | Optional. The `AAS_DB_SSLCERTSRC` variable defines the source location of the database SSL certificate; this variable determines the local location. If the former option is used without specifying this option, the service will copy the SSL certificate to the default configuration directory. |
| AAS\_ADMIN\_USERNAME            | admin@aas                                    | Required. Defines a new AAS administrative user. This user will be able to create new users, new roles, and new role-user mappings. This user will have the `AAS:Administrator` role. |
| AAS\_ADMIN\_PASSWORD            | aasAdminPass                                 | Required. Password for the new AAS admin user.               |
| AAS\_JWT\_CERT\_SUBJECT         | "AAS JWT Signing Certificate"                | Optional. Defines the subject of the JWT signing certificate. |
| AAS\_JWT\_TOKEN\_DURATION\_MINS | 5                                            | Optional. Defines the amount of time in minutes that an issued token will be valid. |
| SAN\_LIST                       | 127.0.0.1,localhost,10.x.x.x                 | Comma-separated list of IP addresses and hostnames that will be valid connection points for the service. Requests sent to the service using an IP or hostname not in this list will be denied, even if it resolves to this service. |
| BEARER\_TOKEN                   | \<token\>                                    | Required. Token from the CMS generated during CMS setup that allows the AAS to perform initial setup tasks. |
| LOG\_LEVEL                      | Critical, error, warning, info, debug, trace | Optional. Defaults to INFO. Changes the log level used.      |

### Configuration Options

### Command-Line Options

Usage:
        authservice <command> [arguments]

Available Commands:
        -h|--help | help                 Show this help message
        setup <task>                     Run setup task
        start                            Start authservice
        status                           Show the status of authservice
        stop                             Stop authservice
        uninstall [--purge]              Uninstall authservice. --purge option needs to be applied to remove configuration and data files
        -v|--version | version           Show the version of authservice

Usage of authservice setup:
        authservice setup [task] [--help] [--force] [-f <answer-file>]
                --help                      show help message for setup task
                --force                     existing configuration will be overwritten if this flag is set
                -f|--file <answer-file>     the answer file with required arguments

        Available Tasks for setup:
                all                      Runs all setup tasks
                download-ca-cert         Download CMS root CA certificate
                download-cert-tls        Download CA certificate from CMS for tls
                database                 Setup authservice database
                admin                    Add authservice admin username and password to database and assign respective
                                         roles to the user
                jwt                      Create jwt signing key and jwt certificate signed by CMS
                update-service-config    Sets or Updates the Service configuration



### Directory Layout

The Verification Service installs by default to `/opt/authservice` with
the following folders.

#### Bin

Contains executable scripts and binaries.

#### dbscripts

Contains database scripts.



Workload Service
----------------

### Installation Answer File Options

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

### Configuration Options

The Workload Service configuration can be found in
`/etc/workload-service/config.yml`:

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

### Command-Line Options

The Workload Service supports several command-line commands that can be
executed only as the Root user:

Syntax:

`workload-service <command>`

####  Help

Available Commands:

help\|-help\|--help Show this help message

start Start workload-service

stop Stop workload-service

status Determine if workload-service is running

uninstall \[--purge\] Uninstall workload-service. --purge option needs
to be applied to remove configuration and data files

setup Setup workload-service for use

Setup command usage: workload-service \<command\> \[task...\]

Available tasks for setup:

download\_ca\_cert

\- Download CMS root CA certificate

\- Environment variable CMS\_BASE\_URL=\<url\> for CMS API url

download\_cert TLS

\- Generates Key pair and CSR, gets it signed from CMS

\- Environment variable CMS\_BASE\_URL=\<url\> for CMS API url

\- Environment variable BEARER\_TOKEN=\<token\> for authenticating with
CMS

\- Environment variable KEY\_PATH=\<key\_path\> to override default
specified in config

\- Environment variable CERT\_PATH=\<cert\_path\> to override default
specified in config

\- Environment variable WLS\_TLS\_CERT\_CN=\<COMMON NAME\> to override
default specified in config

\- Environment variable WLS\_CERT\_ORG=\<CERTIFICATE ORGANIZATION\> to
override default specified in config

\- Environment variable WLS\_CERT\_COUNTRY=\<CERTIFICATE COUNTRY\> to
override default specified in config

\- Environment variable WLS\_CERT\_LOCALITY=\<CERTIFICATE LOCALITY\> to
override default specified in config

\- Environment variable WLS\_CERT\_PROVINCE=\<CERTIFICATE PROVINCE\> to
override default specified in config

server Setup http server on given port

-Environment variable WLS\_PORT=\<port\> should be set

database Setup workload-service database

Required env variables are:

\- WLS\_DB\_HOSTNAME : database host name

\- WLS\_DB\_PORT : database port number

\- WLS\_DB\_USERNAME : database user name

\- WLS\_DB\_PASSWORD : database password

\- WLS\_DB : database schema name

hvsconnection Setup task for setting up the connection to the Host
Verification Service(HVS)

Required env variables are:

\- HVS\_URL : HVS URL

aasconnection Setup to create workload service user roles in AAS

\- AAS\_API\_URL : AAS API URL

\- BEARER\_TOKEN : Bearer Token

logs Setup workload-service log level

\- Environment variable WLS\_LOG\_LEVEL=\<log level\> should be set

####  start 

Start workload-service

####  stop 

Stop workload-service

####  status 

Determine if workload-service is running

####  uninstall 

\[--purge\] Uninstall workload-service. --purge option needs to be
applied to remove configuration and data files

####  setup 

Setup command usage:     workload-service setup [task] [--force]

Available tasks for setup:
   all                              Runs all setup tasks
                                    Required env variables:
                                        - get required env variables from all the setup tasks
                                    Optional env variables:
                                        - get optional env variables from all the setup tasks

   download_ca_cert                 Download CMS root CA certificate
                                    - Option [--force] overwrites any existing files, and always downloads new root CA cert
                                    Required env variables if WLS_NOSETUP=true or variables not set in config.yml:
                                        - AAS_API_URL=<url>                            : AAS API url
                                        - HVS_URL=<url>                                : HVS API Endpoint URL
                                        - WLS_SERVICE_USERNAME=<service username>      : WLS service username
                                        - WLS_SERVICE_PASSWORD=<service password>      : WLS service password
                                    Required env variables specific to setup task are:
                                        - CMS_BASE_URL=<url>                              : for CMS API url
                                        - CMS_TLS_CERT_SHA384=<CMS TLS cert sha384 hash>  : to ensure that WLS is talking to the right CMS instance

   download_cert TLS                Generates Key pair and CSR, gets it signed from CMS
                                    - Option [--force] overwrites any existing files, and always downloads newly signed WLS TLS cert
                                    Required env variables if WLS_NOSETUP=true or variable not set in config.yml:
                                        - CMS_TLS_CERT_SHA384=<CMS TLS cert sha384 hash>  : to ensure that WLS is talking to the right CMS instance
                                        - AAS_API_URL=<url>                            : AAS API url
                                        - HVS_URL=<url>                                : HVS API Endpoint URL
                                        - WLS_SERVICE_USERNAME=<service username>      : WLS service username
                                        - WLS_SERVICE_PASSWORD=<service password>      : WLS service password
                                    Required env variables specific to setup task are:
                                        - CMS_BASE_URL=<url>                       : for CMS API url
                                        - BEARER_TOKEN=<token>                     : for authenticating with CMS
                                        - SAN_LIST=<CSV List>                      : List of FQDNs to be added to the SAN field in TLS cert to override default specified in config
                                    Optional env variables specific to setup task are:
                                        - KEY_PATH=<key_path>                      : Path of file where TLS key needs to be stored
                                        - CERT_PATH=<cert_path>                    : Path of file/directory where TLS certificate needs to be stored
                                        - WLS_TLS_CERT_CN=<COMMON NAME>            : to override default specified in config

   database                         Setup workload-service database
                                    - Option [--force] overwrites existing database config
                                    Required env variables if WLS_NOSETUP=true or variable not set in config.yml:
                                        - CMS_BASE_URL=<url>                              : for CMS API url
                                        - CMS_TLS_CERT_SHA384=<CMS TLS cert sha384 hash>  : to ensure that WLS is talking to the right CMS instance
                                        - AAS_API_URL=<url>                               : AAS API url
                                        - HVS_URL=<url>                                   : HVS API Endpoint URL
                                        - WLS_SERVICE_USERNAME=<service username>         : WLS service username
                                        - WLS_SERVICE_PASSWORD=<service password>         : WLS service password
                                    Required env variables specific to setup task are:
                                        - WLS_DB_HOSTNAME=<db host name>                   : database host name
                                        - WLS_DB_PORT=<db port>                            : database port number
                                        - WLS_DB=<db name>                                 : database schema name
                                        - WLS_DB_USERNAME=<db user name>                   : database user name
                                        - WLS_DB_PASSWORD=<db password>                    : database password
                                    Optional env variables specific to setup task are:
                                        - WLS_DB_SSLMODE=<db sslmode>                      : database SSL Connection Mode <disable|allow|prefer|require|verify-ca|verify-full>
                                        - WLS_DB_SSLCERT=<ssl certificate path>            : database SSL Certificate target path. Only applicable for WLS_DB_SSLMODE=<verify-ca|verify-full>. If left empty, the cert will be copied to /etc/workload-service/wlsdbsslcert.pem
                                        - WLS_DB_SSLCERTSRC=<ssl certificate source path>  : database SSL Certificate source path. Mandatory if WLS_DB_SSLCERT does not already exist

   server                           Setup http server on given port
                                    - Option [--force] overwrites existing server config
                                    Required env variables if WLS_NOSETUP=true or variable not set in config.yml:
                                        - CMS_BASE_URL=<url>                              : for CMS API url
                                        - CMS_TLS_CERT_SHA384=<CMS TLS cert sha384 hash>  : to ensure that WLS is talking to the right CMS instance
                                        - AAS_API_URL=<url>                               : AAS API url
                                        - HVS_URL=<url>                                   : HVS API Endpoint URL
                                    Optional env variables specific to setup task are:
                                        - WLS_PORT=<port>    : WLS API listener port
                                        - WLS_SERVICE_USERNAME=<service username>         : WLS service username
                                        - WLS_SERVICE_PASSWORD=<service password>         : WLS service password

   hvsconnection                    Setup task for setting up the connection to the Host Verification Service(HVS)
                                    - Option [--force] overwrites existing HVS config
                                    Required env variables if WLS_NOSETUP=true or variable not set in config.yml:
                                        - CMS_BASE_URL=<url>                              : for CMS API url
                                        - CMS_TLS_CERT_SHA384=<CMS TLS cert sha384 hash>  : to ensure that WLS is talking to the right CMS instance
                                        - AAS_API_URL=<url>                               : AAS API url
                                        - WLS_SERVICE_USERNAME=<service username>         : WLS service username
                                        - WLS_SERVICE_PASSWORD=<service password>         : WLS service password
                                    Required env variable specific to setup task is:
                                        - HVS_URL=<url>      : HVS API Endpoint URL

   download_saml_ca_cert            Setup to download SAML CA certificates from HVS
                                    - Option [--force] overwrites existing certificate
                                                                        Required env variables if WLS_NOSETUP=true or variable not set in config.yml:
                                        - CMS_BASE_URL=<url>                              : for CMS API url
                                        - CMS_TLS_CERT_SHA384=<CMS TLS cert sha384 hash>  : to ensure that WLS is talking to the right CMS instance
                                        - AAS_API_URL=<url>                               : AAS API url
                                        - WLS_SERVICE_USERNAME=<service username>         : WLS service username
                                        - WLS_SERVICE_PASSWORD=<service password>         : WLS service password
                                                                        Required env variables specific to setup task are:
                                        - HVS_URL=<url>      : HVS API Endpoint URL
                                        - BEARER_TOKEN=<token> for authenticating with HVS

### Directory Layout

The Workload Service installs by default to /opt/wls with the following
folders.



Key Broker Service
------------------

### Installation Answer File Options

### Configuration Options

### Command-Line Options

The Key Broker Service supports several command-line commands that can
be executed only as the Root user:

Usage:
        kbs <command> [arguments]

Available Commands:
        help|-h|--help         						  	Show this help message
        version|-v|--version   					 	 Show the version of current kbs build
        setup <task>           							 Run setup task
        start                  									   Start kbs
        status                 							    	 Show the status of kbs
        stop                   									  Stop kbs
        uninstall [--purge]    							 Uninstall kbs
                --purge            								all configuration and data files will be removed if this flag is set

Usage of kbs setup:
        kbs setup <task> [--help] [--force] [-f <answer-file>]
                --help                      show help message for setup task
                --force                     existing configuration will be overwritten if this flag is set
                -f|--file <answer-file>     the answer file with required arguments

Available Tasks for setup:
        all                                 Runs all setup tasks
        download-ca-cert                    Download CMS root CA certificate
        download-cert-tls                   Download CA certificate from CMS for tls
        create-default-key-transfer-policy  Create default key transfer policy for KBS
        update-service-config               Sets or Updates the Service configuration

### Directory Layout

The Verification Service installs by default with the following folders:

#### /opt/kbs/bin

Contains KBS binaries

#### /etc/kbs/

Contains KBS configuration files

#### /var/log/kbs/

Contains KBS logs





Workload Agent
--------------

### Installation Answer File Options

### Configuration Options

### Command-Line Options

Available Commands:

####  Help

wlagent help\|-help\|--help Show help message

####  setup 

wlagent setup \[task\] Run setup task

##### Available Tasks for setup

###### SigningKey 

Generate a TPM signing key

###### BindingKey 

Generate a TPM binding key

###### RegisterSigningKey 

Register a signing key with the host verification service

-   Environment variable BEARER\_TOKEN=\<token\> for authenticating with
    Verification service

###### RegisterBindingKey 

Register a binding key with the host verification service

-   Environment variable BEARER\_TOKEN=\<token\> for authenticating with
    Verification service

#### start 

Start wlagent

#### stop 

Stop wlagent

#### status 

Reports the status of wlagent service

#### uninstall 

Uninstall wlagent

#### uninstall --purge 

Uninstalls workload agent and deletes the existing configuration
directory

#### version 

Reports the version of the workload agent



### Directory Layout

The Workload Agent installs by default to /opt/workload-agent with the
following folders.

#### Bin

Contains scripts and executable binaries.



Workload Policy Manager
-----------------------

### Installation Answer File Options

| Key                    | Sample Value                                            | Description                                                  |
| ---------------------- | ------------------------------------------------------- | ------------------------------------------------------------ |
| KBS_BASE_URL           | https://\<IP address or hostname of the KBS\>:9443/v1/  | Required. Defines the baseurl for the Key Broker Service. The WPM uses this URL to request new encryption keys when encrypting images. |
| CMS\_TLS\_CERT\_SHA384 |                                                         | Required. SHA384 hash of the CMS TLS certificate             |
| CMS\_BASE\_URL         | https://\<IP address or hostname for CMS\>:8445/cms/v1/ | Required. Defines the base URL for the CMS owned by the image owner. Note that this CMS may be different from the CMS used for other components. |
| AAS\_API\_URL          | https://\<IP address or hostname for AAS\>:8444/aas/v1  | Required. Defines the baseurl for the AAS owned by the image owner. Note that this AAS may be different from the AAS used for other components. |
| BEARER\_TOKEN          | \<token\>                                               | Required; token from CMS with permissions used for installation. |
| WPM\_LOG\_LEVEL        | INFO (default), DEBUG                                   | Optional; defines the log level for the WPM. Defaults to INFO. |
| WPM\_SERVICE_PASSWORD  |                                                         | Defines the credentials for the WPM to use to access the KBS |
| WPM\_SERVICE_USERNAME  |                                                         | Defines the credentials for the WPM to use to access the KBS |

### Configuration Options

### Command-Line Options

The Workload Policy Manager supports several command-line commands that
can be executed only as the Root user:

Syntax:

wpm \<command\>

#### create-image-flavor

Creates a new image flavor and encrypts a source image. Output is the
image flavor in JSON format and the encrypted image.

usage: wpm create-image-flavor \[-l label\] \[-i in\] \[-o out\] \[-e
encout\] \[-k key\]

-l, --label image flavor label

-i, --in input image file path

-o, --out (optional) output image flavor file path

if not specified, will print to the console

-e, --encout (optional) output encrypted image file path

if not specified, encryption is skipped

-k, --key (optional) existing key ID

if not specified, a new key is generated

#### create-software-flavor

Not currently supported; intended for future functionality.

#### Uninstall

Removes the WPM.

#### --help

Displays help text

#### --version

Displays the WPM version

#### Setup

usage : wpm setup \[\<tasklist\>\]

\<tasklist\>-space separated list of tasks

##### wpm setup

##### wpm setup CreateEnvelopeKey

##### wpm setup RegisterEnvelopeKey

##### wpm setup download\_ca\_cert \[--force\]

\- Download CMS root CA certificate

\- Option \[--force\] overwrites any existing files, and always
downloads new root CA cert

\- Environment variable CMS\_BASE\_URL=\<url\> for CMS API url

##### wpm setup download\_cert Flavor-Signing \[--force\]

\- Generates Key pair and CSR, gets it signed from CMS

\- Option \[--force\] overwrites any existing files, and always
downloads newly signed Flavor Signing cert

\- Environment variable CMS\_BASE\_URL=\<url\> for CMS API url

\- Environment variable BEARER\_TOKEN=\<token\> for authenticating with
CMS

\- Environment variable KEY\_PATH=\<key\_path\> to override default
specified in config

\- Environment variable CERT\_PATH=\<cert\_path\> to override default
specified in config

\- Environment variable WPM\_FLAVOR\_SIGN\_CERT\_CN=\<COMMON NAME\> to
override default specified in config

\- Environment variable WPM\_CERT\_ORG=\<CERTIFICATE ORGANIZATION\> to
override default specified in config

\- Environment variable WPM\_CERT\_COUNTRY=\<CERTIFICATE COUNTRY\> to
override default specified in config

\- Environment variable WPM\_CERT\_LOCALITY=\<CERTIFICATE LOCALITY\> to
override default specified in config

\- Environment variable WPM\_CERT\_PROVINCE=\<CERTIFICATE PROVINCE\> to
override default specified in config



Certificate and Key Management
==============================

Host Verification Service Certificates and Keys
-----------------------------------------------

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



TLS Certificates
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



# Upgrades

***NOTE:***  Before performing any upgrade, Intel strongly recommends backing up the database for the HVS, WLS, and AAS.  See Postgres documentation for detailed options for backing up databases.  Below is a sample method for backing up an entire database server:

```
Backup to tar file:
pg_dump --dbname <database_name> --username=<database username> -F -t > <database_backup_file>.tar
Restore from tar file:
pg_restore --dbname=<database_name> --username=<database username><database_backup_file>.tar	
```

Some upgrades may involve changes to database content, and a backup will ensure that data is not lost in the case of an error during the upgrade process.

## Backward Compatibility

In general Intel SecL services are made to be backward-compatible within a given major release (for example, the 3.6 HVS should be compatible with the 3.5 Trust Agent) in an upgrade priority order (see below).  Major version upgrades may require coordinated upgrades across all services.

## Upgrade Order

Upgrades should be performed in the following order to prevent misconfiguration or any service unavailability:

1) CMS, AAS

2)  HVS

3) WLS, IHUB

4) KBS, Trust Agents, Workload Agents, WPM

Upgrading in this order will make each service unavailable only for the duration of the upgrade for that service.  

## Upgrade Process

### Binary Installations

For services installed directly (not deployed as containers), the upgrade process simply requires executing the new-version installer on the same machine where the old-version is running.  The installer will re-use the same configuration elements detected in the existing version's config file.  No additional answer file is required unless configuration settings will change.





Appendix
========

PCR Definitions
---------------

### Red Had Enterprise Linux

#### TPM 2.0

<table>
<thead>
<tr class="header">
<th>PCR</th>
<th>Measurement Parameters</th>
<th>Description</th>
<th>Operating System</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>PCR 0</td>
<td><p>BIOS ROM and Flash Image</p>
<p>Initial Boot Block (Intel® BootGuard only)</p></td>
<td><p>This PCR is based solely on the BIOS version, and remains identical across all hosts using the same BIOS. This PCR is used as the PLATFORM Flavor.</p>
<p>(Intel® BootGuard only): Extends measurements based on the Intel® BootGuard profile configuration and production vs non-production ACM flags; ACM signature; BootGuard key manifest hash; Boot Policy Manifest Signature</p></td>
<td><ul>
<li><p>All</p></li>
</ul></td>
</tr>
<tr class="even">
<td>PCR 7</td>
<td>Intel® BootGuard configuration and profiles</td>
<td>Describes the success of the IBB measurement event.</td>
<td><ul>
<li><p>All (Intel® BootGuard only)</p></li>
</ul></td>
</tr>
<tr class="odd">
<td>PCR 17</td>
<td>ACM</td>
<td>BIOS AC registration information<br />
Digest of Processor S-CRTM<br />
Digest of Policycontrol<br />
Digest of all matching elements used by the policy<br />
Digest of STM<br />
Digest of Capability field of OsSinitData<br />
Digest of MLE<br />
For TA hosts, this PCR includes measurements of the OS, InitRD, and UUID. This changes with every install due to InitRD and UUID change.</td>
<td><ul>
<li><p>VMware ESXi</p></li>
<li><p>Red Hat Enterprise Linux</p></li>
</ul></td>
</tr>
<tr class="even">
<td>PCR 18</td>
<td>MLE [Tboot +VMM]</td>
<td>Digest of public key modulus used to verify SINIT signature<br />
Digest of Processor S-CRTM<br />
Digest of Capability field of OSSinitData table<br />
Digest of PolicyControl field of used policy<br />
Digest of LCP</td>
<td><ul>
<li><p>VMware ESXi</p></li>
<li><p>Red Hat Enterprise Linux</p></li>
</ul></td>
</tr>
<tr class="odd">
<td>PCR 19</td>
<td><blockquote>
<p>OS Specific.</p>
</blockquote>
<ul>
<li><p>ESX and Trust Agent — non Kernel modules</p></li>
<li><p>Citrix Xen — OS</p></li>
<li><p>+ Init RD + UUID</p></li>
</ul></td>
<td><p>For ESXi and Trust Agent hosts, this PCR contains individual measurements of all of the non-Kernel modules.</p>
<p>For Linux hosts, this PCR is a measurement of the OS, InitRD, and UUID.</p></td>
<td><ul>
<li><p>VMware ESXi</p></li>
<li><p>Red Hat Enterprise Linux</p></li>
</ul></td>
</tr>
</tbody>
</table>

### VMWare ESXi

#### TPM 1.2

<table>
<thead>
<tr class="header">
<th>PCR</th>
<th>Measurement Parameters</th>
<th>Description</th>
<th>Operating System</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>PCR 0</td>
<td>BIOS ROM and Flash Image</td>
<td>This PCR is based solely on the BIOS version, and remains identical across all hosts using the same BIOS. This PCR is used as the PLATFORM Flavor.</td>
<td><ul>
<li><p>All</p></li>
</ul></td>
</tr>
<tr class="even">
<td>PCR 17</td>
<td>ACM</td>
<td>This PCR measures the SINIT ACM, and is hardware platform-specific. This PCR is part of the PLATFORM Flavor.</td>
<td><ul>
<li><p>VMware ESXi</p></li>
<li><p>Red Hat Enterprise Linux</p></li>
</ul></td>
</tr>
<tr class="odd">
<td>PCR 18</td>
<td>MLE [Tboot +VMM]</td>
<td>This PCR measures the tboot and hypervisor version. In ESXi hosts, only the tboot version is measured.</td>
<td><ul>
<li><p>VMware ESXi</p></li>
<li><p>Red Hat Enterprise Linux</p></li>
</ul></td>
</tr>
<tr class="even">
<td>PCR 19</td>
<td><blockquote>
<p>OS Specific.</p>
</blockquote>
<ul>
<li><p>ESX and Trust Agent — non Kernel modules</p></li>
<li><p>Citrix Xen — OS</p></li>
<li><p>+ Init RD + UUID</p></li>
</ul></td>
<td><p>For ESXi and Trust Agent hosts, this PCR contains individual measurements of all of the non-Kernel modules.</p>
<p>For Citrix Xen hosts, this PCR is a measurement of the OS, InitRD, and UUID.</p></td>
<td><ul>
<li><p>VMware ESXi</p></li>
<li><p>Red Hat Enterprise Linux</p></li>
</ul></td>
</tr>
<tr class="odd">
<td>PCR 20</td>
<td><p>For ESXi only.</p>
<p>VM Kernel and VMK Boot</p></td>
<td>This PCR is used only by ESXi hosts and is blank for all other host types.</td>
<td><ul>
<li><p>VMware ESXi</p></li>
</ul></td>
</tr>
<tr class="even">
<td>PCR 22</td>
<td>Asset Tag</td>
<td>This PCR contains the measurement of the SHA1 of the Asset Tag Certificate provisioned to the TPM, if any.</td>
<td><ul>
<li><p>VMware ESXi</p></li>
<li></li>
</ul></td>
</tr>
</tbody>
</table>

#### TPM 2.0

VMWare supports TPM 2.0 with Intel TXT starting in vSphere 6.7 Update 1.
Earlier versions will support TPM 1.2 only.

<table>
<thead>
<tr class="header">
<th>PCR</th>
<th>Measurement Parameters</th>
<th>Description</th>
<th>Operating System</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>PCR 0</td>
<td>BIOS ROM and Flash Image</td>
<td>This PCR is based solely on the BIOS version, and remains identical across all hosts using the same BIOS. This PCR is used as part of the PLATFORM flavor.</td>
<td><ul>
<li><p>All</p></li>
</ul></td>
</tr>
<tr class="even">
<td>PCR 17</td>
<td>ACM</td>
<td>This PCR measures the SINIT ACM, and is hardware platform-specific. This PCR is part of the PLATFORM Flavor.</td>
<td><ul>
<li><p>VMware ESXi</p></li>
<li><p>Red Hat Enterprise Linux</p></li>
</ul></td>
</tr>
<tr class="odd">
<td>PCR 18</td>
<td>MLE [Tboot +VMM]</td>
<td>This PCR measures the tboot and hypervisor version. In ESXi hosts, only the tboot version is measured. This PCR is part of the PLATFORM Flavor.</td>
<td><ul>
<li><p>VMware ESXi</p></li>
<li><p>Red Hat Enterprise Linux</p></li>
</ul></td>
</tr>
<tr class="even">
<td>PCR 19</td>
<td><blockquote>
<p>OS Specific.</p>
</blockquote>
<ul>
<li><p>ESX and Trust Agent — non Kernel modules</p></li>
<li><p>Citrix Xen — OS</p></li>
<li><p>+ Init RD + UUID</p></li>
</ul></td>
<td>For ESXi this PCR contains individual measurements of all of the non-Kernel modules – this includes all of the VIBs installed on the ESXi host. This is part of the OS flavor. Note that two ESXi hosts with the same version of ESXi installed may require different OS flavors if different VIBs are installed.</td>
<td><ul>
<li><p>VMware ESXi</p></li>
<li><p>Red Hat Enterprise Linux</p></li>
</ul></td>
</tr>
<tr class="odd">
<td>PCR 20</td>
<td><p>For ESXi only.</p>
<p>VM Kernel and VMK Boot</p></td>
<td>This PCR is used only by ESXi hosts for some host-specific measurements, and is part of the host-unique flavor.</td>
<td><ul>
<li><p>VMware ESXi</p></li>
</ul></td>
</tr>
<tr class="even">
<td>PCR 22</td>
<td>Asset Tag</td>
<td>Asset Tag is not currently supported for TPM 2.0 with ESXi.</td>
<td><ul>
<li><p>VMware ESXi</p></li>
<li></li>
</ul></td>
</tr>
</tbody>
</table>


## Attestation Rules

<table>
<thead>
<tr class="header">
<th>Platform</th>
<th>TPM</th>
<th>Flavor Type</th>
<th>Rules to be verified</th>
<th>Comments</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>RHEL</td>
<td>2.0</td>
<td>HARDWARE</td>
<td><p>PcrMatchesConstant rule for PCR 0</p>
<p>PcrEventLogIncludes rule for PCR 17 (LCP_DETAILS_HASH, BIOSAC_REG_DATA, OSSINITDATA_CAP_HASH, STM_HASH, MLE_HASH, NV_INFO_HASH, tb_policy, CPU_SCRTM_STAT, HASH_START, LCP_CONTROL_HASH)</p>
<p>PcrEventLogIntegrity rule for PCR 17</p></td>
<td><p>Evaluation of PcrEventLogIncludes would not include initrd and vmlinuz modules. They would be handled in host_specific flavor.</p>
<p>Evaluation of PcrEventLogIntegrity rule would also include OS modules (initrd &amp; vmlinuz)</p></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td>OS</td>
<td>PcrEventLogIntegrity rule for PCR 17</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td>ASSET_TAG</td>
<td>AssetTagMatches rule</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td>HOST_SPECIFIC</td>
<td>PcrEventLogIncludes rule for PCR 17 (initrd &amp; vmlinuz)</td>
<td></td>
</tr>
<tr class="odd">
<td>VMware ESXi</td>
<td>1.2</td>
<td>PLATFORM</td>
<td><p>PcrMatchesConstant rule for PCR 0</p>
<p>PcrMatchesConstant rule for PCR 17</p></td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td>OS</td>
<td><p>PcrMatchesConstant rule for PCR 18</p>
<p>PcrMatchesConstant rule for PCR 20</p>
<p>PcrEventLogEqualsExcluding rule for PCR 19 (excludes dynamic modules based on component name)</p>
<p>PcrEventLogIntegrity rule for PCR 19</p></td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td>ASSET_TAG</td>
<td>PcrMatchesConstant rule for PCR 22</td>
<td></td>
</tr>
<tr class="even">
<td>VMware ESXi</td>
<td>2.0</td>
<td></td>
<td>NOT SUPPORTED</td>
<td></td>
</tr>
<tr class="odd">
<td>Windows</td>
<td>1.2</td>
<td>PLATFORM</td>
<td>PcrMatchesConstant rule for PCR 0</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td>OS</td>
<td><p>PcrMatchesConstant rule for PCR 13</p>
<p>PcrMatchesConstant rule for PCR 14</p></td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td>ASSET_TAG</td>
<td>AssetTagMatches rule</td>
<td></td>
</tr>
<tr class="even">
<td>Windows</td>
<td>2.0</td>
<td>PLATFORM</td>
<td>PcrMatchesConstant rule for PCR 0</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td>OS</td>
<td><p>PcrMatchesConstant rule for PCR 13</p>
<p>PcrMatchesConstant rule for PCR 14</p></td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td>ASSET_TAG</td>
<td>AssetTagMatches rule</td>
<td>AssetTagMatches rule needs to be updated to verify the key-value pairs after verifying the tag certificate.</td>
</tr>
</tbody>
</table>
