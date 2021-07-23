# Intel® Security Libraries - Datacenter SGX Attestation Infrastructure and Secure Key Caching

## Product Guide

### July 2021

### Revision 4.0

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

Copyright © 2021, Intel Corporation. All Rights Reserved.

Revision History

[[_TOC_]]

# 1 Introduction 

## 1.1 Overview

The SGX Attestation infrastructure and Secure Key Caching (SKC) are part of the Intel Security Libraries for datacenter (ISecL-DC). Intel Security Libraries for Datacenter is a collection of software applications and development libraries intended to help turn Intel platform security features into real-world security use cases.

The SGX Attestation infrastructure provides an end to end support for registering SGX hosts and provisioning them with SGX material (PCK certificates) and SGX collateral (security patches information - TCB Information - and Certificate Revocation Lists - CRLs).

The SGX Attestation infrastructure also provides support for generating SGX quotes for SGX enclaves hosted by workloads and verifying them by a remote attesting application. The remote attesting application can also use the SGX Attestation infrastructure to enforce enclave policies (like requiring a specific enclave signer). Optionally, the SGX Attestation Infrastructure allows to integrate with Cloud Orchestrators like Openstack and Kubernetes.

SKC leverages the SGX Attestation Infrastructure to support the Secure Key Caching (SKC) use case.SKC provides the key protection at rest and in-use use case using the Intel Software Guard Extensions technology (SGX). SGX implements the Trusted Execution Environment (TEE) paradigm.

Using the SKC Client -- a set of libraries -- applications can retrieve keys from the ISecL-DC Key Broker Service (KBS) and load them to an SGX-protected memory (called SGX enclave) in the application memory space. KBS performs the SGX enclave attestation to ensure that the application will store the keys in a genuine SGX enclave. Application keys are wrapped with an enclave public key by KBS prior to transferring to the application enclave. Consequently, application keys are protected from infrastructure admins, malicious applications and compromised HW/BIOS/OS/VMM. SKC does not require the refactoring of the application because it supports a standard PKCS\#11 interface.

## Trusted Execution Environment

A Trusted Execution Environment (TEE) provides a computer secure area where code and data can be loaded with the assurance that their confidentiality and integrity are protected. TEEs have various applications in areas where the confidentiality of the data and the integrity of the code are of the highest importance. One examples of a TEE usage is the protection of sensitive secrets like cryptographic keys and authentication strings. These secrets can be preserved within a TEE, ensuring that they never get exposed to software that is outside of the TEE. Operations involving these secrets can be performed within the TEE, removing the need to ever extract the secrets outside of the TEE. Another example is the processing of data with privacy concerns. This is often the case in financial, healthcare and artificial intelligence (AI) applications. Putting sensitive operations inside a TEE allows organizations to support business processes without compromising the privacy expectations of their customers.

### Intel Software Guard Extensions

Intel Software Guard Extensions (SGX) is an Intel platform security feature that implements the TEE paradigm. A portion of RAM called EPC (Enclave Page Cache) is used by applications to load secure isolated areas called SGX enclaves. Code and data inside SGX enclaves are encrypted and only decrypted inside the Intel CPU. From the host application perspective, an SGX enclave looks like a dynamic library. Any part of the application that is not contained in an SGX enclave is considered untrusted while the SGX enclave is considered trusted. Communications between the untrusted part and the trusted part (the SGX enclave) of an application uses a special calls called ECALLS and call from the enclave to the untrusted part of the application use OCALLS. A signed claim called SGX quote can be generated for an enclave. The SGX quote may contain a measurement of the code and the data of the enclave. An SGX quote allows to prove to a remote verifier (relying party) that an application includes the expected SGX enclave.

#### SGX ECDSA Attestation 

SGX ECDSA attestation is the process that allows an application (relying party) to verify that a remote piece of code and data that it's interacting with is contained in a genuine Intel SGX enclave. The remote enclave can generate a signed claim called an SGX quote. A valid SGX quote signature generated on an SGX enabled platform can be chained up to a trusted Intel signing key. The SGX quote contains the measurement of the enclave (MREnclave), the enclave developer's signature (MRSigner), the security patch level of the platform (Trusted Computing Base or TCB) and any user data that the enclave wants to include in the quote. Typically, the user data in an SGX quote contains the hash of the public key part of a public/private key pair generated inside the enclave. The public key is transferred along with the SGX quote to the relying party. The latter generates a Symmetric Wrapping Key (SWK) and wraps it with the public key of the enclave. The wrapped SWK is provisioned into the SGX enclave, which can unwrap it since it has the corresponding private key. The relying party can then provision secrets into the SGX enclave after wrapping them with the SWK. For an enclave to generate an SGX quote, a PCK certificate for the host platform must be obtained from Intel SGX Provisioning Certification Service (PCS).

#### PCK Certificates Provisioning

To generate an SGX quote for an enclave, a PCK certificate must be obtained from SGX Provisioning Certification Service (PCS). Requiring a workload to retrieve the PCK certificate from Intel PCS at the time of the SGX quote generation can be detrimental to the workload. Network connectivity issues can prevent the connection to Intel PCS. To remove the dependency on network connectivity, the PCK certificates of the data center platforms are fetched before running any workload. This is achieved by extracting SGX related information from the platform using the PCK ID Retrieval tool and pushing it to a Caching Service running in the same data center. The Caching Service then retrieves the PCK certificates of all the platforms that pushed SGX information to it from Intel PCS. Network connectivity issues are not a problem since the Caching Service can retry if needed. When an SGX workload needs its PCK certificate, it can just get it from the Caching Service.

## Key Protection

Cryptographic keys are high value assets that must be protected against disclosure and corruption. Key disclosure or corruption expose the key owner to data confidentiality breaches, impersonation and denial of service. The industry has good solutions to protect keys at rest.

A popular solution is to store keys in a central secure Key Management System (KMS), and applications retrieve them at runtime. However, this solution does not protect keys once they are in RAM and used to perform cryptographic operations. Keys in RAM can be disclosed because of software vulnerabilities like Heartbleed or because of memory snapshots. Therefore, keys are not protected in use. This concern can be addressed by having the application send the payload that needs cryptographic processing to the KMS where the processing happens instead. By doing this, the key is never exposed in RAM. However, this solution incurs an overhead caused by the network round trip to the KMS.

Another solution is to store keys in Hardware Security Modules (HSMs)

### HSM

A Hardware Security Module or HSM is a separate hardware part that can be attached to a server. HSMs provide APIs to create and load keys. HSMs also support APIs to perform cryptographic operations using keys stored inside them. The typical flow for using an HSM is to create or load a key in the HSM in a secure environment then take the HSM to the server where the workload runs and attach it to this server. The application then performs cryptographic operations using the key inside the HSM. This ensures that the key is never exposed in RAM. Therefore, HSMs protect keys both at rest and in-use. The drawback with HSMs is that they can be a costly hardware add-on to the server, and they require physical access to the server to get attached to it (via the USB port for example).

Most HSMs support the PKCS\#11 cryptographic programming interface.

### PKCS\#11

PKCS\#11 is the standard cryptographic programming interface supported by HSMs. The PKCS\#11 interface is defined using a C-style definition, but many languages support bindings exist. Although applications can directly use the PKCS\#11 programming interface, most applications use other cryptographic interfaces like openssl. Fortunately, openssl supports a PKCS\#11 engine mechanism that converts openssl calls to PKCS\#11calls. This allows applications written against the openssl cryptographic interface to use an HSM supporting the PKCS\#11 interface without code change. Popular applications that use openssl but can still use an HSM to protect the key include Nginx and Apache.

## Features

### SGX Attestation Infrastructure 

The SGX Attestation Infrastructure allows to fetch PCK certificates and SGX collateral from Intel SGX Provisioning Certification Service (PCS). It makes the PCK certificates available to workloads that use the SKC Client, which allows them to generate SGX quotes. The SGX Attestation Infrastructure also includes components that perform the verification of SGX quotes.

### SGX Support in Orchestrators

The SGX Attestation Infrastructure can optionally push the SGX information on compute nodes to cloud orchestrators so that SGX workloads (like SKC) can be scheduled on compute nodes that support SGX. Currently, the Kubernetes and Openstack orchestrators are supported.

### Key Protection

SKC leverages the SGX Attestation Infrastructure to protect keys in an SGX enclave at rest and in use. Applications use the SKC Client -- a set of libraries -- to retrieves keys at runtime from KBS. KBS performs an SGX enclave attestation. If the attestation is successful, KBS generates a Symmetric Wrapping Key (SWK), wraps it with the enclave public key and provisions it into the enclave, which can unwrap it since it has the corresponding private key. Application can then be provisioned into the SGX enclave after being wrapped with the SWK. Application keys are therefore never exposed to any software outside of the enclave.

# SGX Attestation Infrastructure and SKC Components

The components documented in this section are used by the SGX Attestation Infrastructure and therefore by SKC, which leverages the SGX Attestation Infrastructure. Components that are exclusively used by SKC have (SKC Only) in the corresponding sub-section title.

## Certificate Management Service

All the certificates used by SKC services and by the SGX Agent are issued by the Certificate Management Service (CMS). CMS has a root CA certificate and all the SKC services and the SGX Agent certificates chain up to the CMS root CA.

CMS is an infrastructure service and is shared with other Intel® SecL-DC components.

## Authentication and Authorization Service

The authentication and authorization for all SKC services and the SGX Agent are centrally managed by the Authentication and Authorization Service (AAS).

AAS is an infrastructure service and is shared with other Intel® SecL-DC components.

## SGX Caching Service

The SGX Caching Service (SCS) allows to retrieve the PCK certificates of the data center server platforms from Intel SGX Provisioning Certification Service (PCS). SCS retrieves also platform models collateral. The collateral consists of the security patches (TCBInfo) that have been issued for Intel platform models. Finally, SCS retrieves the Certificate Revocation Lists (CRLs).

Since the Caching Service stores all the TCBInfo of all the platform models in the datacenter, the SGX Quote Verification Service (SQVS) uses it to determine the TCB status of the platforms in the data center.

The SKC Client retrieves its PCK certificate from the Caching Service when it generates an SGX quote.

SCS can be deployed in both Cloud Service Provider (CSP) and tenant environments. In the CSP environment, SCS is used to fetch PCK certificates for compute nodes in the data center. In the tenant environment, it's used to cache SGX collateral information used in verifying SGX quotes.

## SGX Host Verification Service

If SGX Host Verification Service API URL is specified in SGX Agent env file, then SGX Agent will push the platform enablement info and TCB status to SHVS at regular interval, else, Agent pushes the platform enablement info and TCB status to SHVS periodically. The SGX enablement information consists of SGX discovery information (SGX supported, SGX enabled, FLC enabled and EPC memory size).

## SGX Agent

The SGX Agent resides on physical servers and pushes SGX platform specific values to SGX Caching Service (SCS).

If SGX Host Verification Service (SHVS) URL is specified in SGX Agent env file, SGX Agent would fetch the TCB status from SCS and updates SHVS with platform enablement info and TCB status periodically.

## Integration Hub

The Integration Hub (IHUB) allows to support SGX in Kubernetes and Open stack. IHUB pulls the list of hosts details from Kubernetes and then using the host information it pulls the SGX Data from SGX Host Verification Service and pushes it to Kubernetes. IHUB performs these steps on a regular basis so that the most recent SGX information about nodes is reflected in Kubernetes and Openstack. This integration allows Kubernetes and Openstack to schedule VMs and containers that need to run SGX workloads on compute nodes that support SGX. The SGX data that IHUB pushes to Kubernetes consists of SGX enabled/disabled, SGX supported/not supported, FLC enabled/not enabled, EPC memory size, TCB status up to date/not up to date and platform-data expiry time.

## Key Broker Service (SKC Only)

The Key Broker Service (KBS) is typically deployed in the tenant environment, not the Cloud Service Provider (CSP) environment. KBS is effectively a policy compliance engine. Its job is to manage key transfer requests from SKC Clients, releasing keys only to those that meet policy requirements. A user admin can create and register keys in KBS. He can also create key policies and assign them to keys. A key policy specifies the conditions that the SKC Client must fulfill for keys that have the policy assigned to them to be released. Most of the information about an SKC Client is contained in the SGX quote that it sends to KBS. The SGX quote also contains a hash of the enclave's public key. KBS gets the public key along the quote so the hash in the quote allows to verify that the public key is genuine. If the SGX quote verification (attestation) is successful, KBS generates a Symmetric Wrapping Key (SWK), wraps it with the enclave public key and provisions it into the enclave, which can unwrap it since it has the corresponding private key. Application can then be provisioned into the SGX enclave after being wrapped with the SWK. Application keys are therefore never exposed to any software outside of the enclave. 

KBS is shared with other Intel® SecL-DC components.

## SGX Quote Verification Service

The SGX Quote Verification Service (SQVS) is typically deployed in the tenant environment, not the Cloud Service Provider (CSP) environment. SQVS performs the verification of SGX quotes on behalf of KBS. SQVS determines if the SGX quote signature is valid. It also determines if the SGX quote has been generated on a platform that is up to date on security patches (TCB). For the latter, SQVS uses the SGX Caching Service, which caches the SGX collateral information about Intel platform models. SQVS also parses the SGX quote and extracts the entities and returns them to KBS, which can then make additional policy decisions based on the values of the theses entities.

## The Workload SGX Dependencies

This is a set of dependencies needed by SGX workloads. 

## The SKC Client (Secure Key Caching Use case Only)

The SKC Client refers to a suite of libraries that applications that require key protection must link with. It's comprised of the SKC Library, which is an Intel® SecL-DC component and the Intel Crypto Toolkit. the SKC Client uses the workload SGX dependencies component.  The SKC Library supports the PKCS\#11 interface and is therefore considered as a PKCS\#11 module from the host application perspective. The SKC Library uses Intel Crypto Toolkit to protect keys in an SGX enclave. When a key is requested by the host application, the SKC Library sends a request to the Key Broker Service (KBS) along with an SGX quote that is generated by the Crypto Toolkit. KBS releases the key after verifying the quote and evaluating the attributes contained in the quote. The key policy can also specify conditions that can't be verified with the SGX quote alone.

The SKC Client is typically deployed inside a tenant VM or container. It can also be used on bare metal. In all these deployments, the underlying platform is typically owned by a Cloud Service Provider (CSP) and is considered untrusted.

# Definitions, Acronyms, and Abbreviation

SKC -- Secure Key Caching

SGX -- Software Guard Extension

TEE -- Trusted Execution Environment

HSM -- Hardware Security Module

KBS -- Key Broker Service

CSP -- Cloud Service Provider

PCS -- Provisioning Certification Service

CRLs -- Certificate Revocation Lists 

AAS -- Authentication and Authorization Service

SWK -- Symmetric Wrapping Key

CRDs -- Custom Resource Definitions 

# Architecture Overview 

As indicated in the Features section, SKC provides 3 features essentially:

-   SGX Attestation Support: this is the feature that CSPs provide to tenants who need to run SGX workloads that require attestation.
-   SGX Support  in Orchestrators: this feature allows to discover SGX support in physical servers and related information:
    -   SGX supported.

    -   SGX enabled.

    -   Size of RAM reserved for SGX. It's called Enclave Page Cache (EPC).

    -   Flexible Launch Control enabled.
-   Key Protection: this is the feature used by tenants using a CSP to run workloads with key protection requirements.

The high-level architectures of these features are presented in the next sub-sections.

## SGX Attestation Support and SGX Support in Orchestrators

The diagram below shows the infrastructure that CSPs need to deploy to support SGX attestation and optionally, integration with orchestrators (Kubernetes and OpenStack). 

THE SGX Agent pushes platform information to SGX Caching Service (SCS), which uses it to get the PCK Certificate and other SGX collateral from the Intel SGX Provisioning Certification Service (PCS) and caches them locally. When a workload on the platform needs to generate an SGX Quote, it retrieves the PCK Certificate of the platform from SCS.

If SGX Host Verification Service (SHVS) URL is configured, the SGX Agent fetches the TCB Status from SCS and updates SHVS with SGX platform enablement information and TCB status periodically. The platform information is made available to Kubernetes and Openstack via the SGX Hub (IHUB), which pulls it from SHVS.

The SGX Quote Verification Service (SQVS) allows attesting applications to verify SGX quotes and extract the SGX quote attributes to verify compliance with a user-defined SGX enclave policy. SQVS uses the SGX Caching Service to retrieve SGX collateral needed to verify SGX quotes from the Intel SGX Provisioning Certification Service (PCS). SQVS typically runs in the attesting application owner network environment. Typically, a separate instance of the SGX Caching Service is setup in the attesting application owner network environment. 

![](Images/image-20200727163158892.png)

The SGX Agent and the SGX services integrate with the Authentication and Authorization Service (AAS) and the Certificate Management Service (CMS). AAS and CMS are not represented on the diagram for clarity.

##  Key Protection

Key Protection leverages the SGX Attestation support and optionally, the SGX support in orchestrators.

Key Protection is implemented by the SKC Client -- a set of libraries - which must be linked with a tenant workload, like Nginx, deployed in a CSP environment and the Key Broker Service (KBS) deployed in the tenant's enterprise environment. The SKC Client retrieves the keys needed by the workload from KBS after proving that the key can be protected in an SGX enclave as shown in the diagram below.

![](Images/image-20200727163116765.png)

Step 6 is optional (keys can be stored in KBS). Keys policies in step 2 are called Key Transfer Policies and are created by an Admin and assigned to Application keys.

##  SKC Virtualization (Supported only on RHEL 8.2, not supported on Ubuntu 18.04)

Virtualization enabled on SGX Host machines, uses SGX key features. With Virtualization being enabled on SGX host, SKC Library which uses Intel crypto tool kit to protect keys in SGX Enclave can be configured on Virtual Machines which are created on SGX Hosts. This enhancement further provides the privilege for a workload on a VM  allowing successful Secure Key transfer flow which meets the policy requirements. Hence virtualization on SGX Hosts supports key transfer flow for Workload on bare metal, Workload inside a VM, Workload in a container and Workload in a container inside a VM enabled on SGX Host.


# Intel® Security Libraries Installation

## Building from Source

Intel® Security Libraries is distributed as open source code and must be compiled into installation binaries before installation.

Instructions and sample scripts for building the Intel® SecL-DC components can be found here (Section 1 to 6)

https://github.com/intel-secl/docs/blob/v5.0/develop/quick-start-guides/SGX%20Infrastructure%20-%20RHEL.md

After the components have been built, the installation binaries and database scripts can be found in the binaries directory created by the build scripts.

Generated component binaries/installers are:

- CMS: cms-v4.0.0.bin
- AAS: authservice-v4.0.0.bin
- SCS: scs-v4.0.0.bin
- SHVS: shvs-v4.0.0.bin
- IHUB: ihub-v4.0.0.bin
- SQVS: sqvs-v4.0.0.bin
- KBS: kbs-v4.0.0.bin
- K8S-Extensions: isecl-k8s-extensions-v4.0.0.tar.gz
- SGX-Agent: agent_untar.sh, sgx_agent.sha2 and sgx_agent.tar
- SKC-Library: skclib_untar.sh, skc_library.sha2 and skc_library.tar

DB scripts:

- Postgres installation script: install_pgdb.sh
- AAS, SCS and SHVS DB creation script: create_db.sh

## Building from Source - OCI images & K8s Manifests

Intel® Security Libraries is distributed as open source code and must be compiled into OCI images before installation.

Instructions and sample scripts for building the Intel® SecL-DC components  as containerized images for Kubernetes deployments can be found in [Quick Start guide](https://github.com/intel-secl/docs/blob/v5.0/develop/quick-start-guides/SGX%20Infrastructure%20-%20Containerization.md#build)

After the components have been built, the OCI images and pre-req scripts can be found in the `K8s` directory created by the build scripts.

Generated components `OCI images`  under `<working directory>/k8s/container-images`:

* Authentication Authorization Service: `aas-v4.0.0.tar`
* Certificate Management Service: `cms-v4.0.0.tar`
* Integration Hub: `ihub-v4.0.0.tar`
* Key Broker Service: `kbs-v4.0.0.tar`
* K8s Extensions Custom Controller: `isecl-k8s-controller-v4.0.0.tar`
* K8s Extensions Extended Scheduler: `isecl-k8s-scheduler-v4.0.0.tar`
* SGX Caching Service: `scs-v4.0.0.tar`
* SGX Quote Verification Service: `sqvs-v4.0.0.tar`
* SGX Host Verification Service: `shvs-v4.0.0.tar`
* SGX Agent: `sgx-agent-v4.0.0.tar`
* SKC Library: `skc-lib-4.0.0.tar`

Generated Components `K8s Manifests` directories under `<working directory>/k8s/manifests`:

* Authentication Authorization Service Database: `aas-db`
* SGX Caching Service Database: `scs-db`
* SGX Host Verification Service Database: `shvs-db`
* Certificate Management Service: `cms`
* Authentication Authorization Service: `aas`
* Integration Hub: `ihub`
* Key Broker Service: `kbs`
* SGX Caching Service: `scs`
* SGX Host Verification Service: `shvs`
* SGX Quote Verification Service: `sqvs`
* K8s Extensions Custom Controller: `k8s-extensions-controller`
* K8s Extensions Extended Scheduler: `k8s-extensions-scheduler`
* SGX Agent: `sgx_agent`
* SKC Library: `skc_library`

Bootstrap scripts and answer file under `<working directory>/k8s/manifests`:

* Pre-req: `pre-requisites.sh`
* Bootstrap DB: `skc-bootstrap-db-services.sh`
* Answer file: `isecl-skc-k8s.env`
* Bootstrap: `skc-bootstrap.sh`

## Hardware Considerations

### Supported **Hardware**

-   Intel® Xeon® SP products those support SGX

### BIOS Requirements

-   Intel® SGX-TEM BIOS requirements are outlined in the latest Intel® SGX Platforms BIOS Writer's Guide

-   Intel® SGX should be enabled in BIOS menu (Intel® SGX is Disabled by default on Ice Lake).

-   Intel® SGX BIOS requirements include exposing Flexible Launch Control menu.


### OS Requirements (Intel® SGX does not supported on 32-bit OS):

-   Linux: RHEL 8.2

-   Linux: Ubuntu 18.04

### Requirements for Containerized Deployment with K8s

#### Operating System:

* RHEL 8.2
* Ubuntu 18.04

#### Kubernetes

* Single-node: `microk8s` (1.17.17)
* Multi-node: `kubeadm` (1.17.17)

#### Container Runtime

* Docker 19.03.13 on RHEL 8.2
* Docker 19.03.5 on Ubuntu 18.04

#### Storage:

* `hostPath` for Single Node `microk8s` for all services and agents
* `NFS` for Multi Node `kubeadm` for all services, `hostPath ` for sgx_agent and skc_library

##  Recommended Service Layout

The Intel® SecL-DC services can be installed in a variety of layouts, partially depending on the use cases desired and the OS of the server(s) to be protected. In general, the Intel® SecL-DC applications can be divided into management services that are deployed on the network on the management plane, and host or node components that must be installed on each protected server.

Management services can typically be deployed anywhere with network access to all the protected servers. This could be a set of individual VMs per service; containers; or all installed on a single physical or virtual machine.

Node components must be installed on each protected physical server.

Typically, this is needed for Linux deployments.

For a POC environment, all the management services can be installed on a single machine or VM.

This Includes:

-   Certificate Management Service (CMS)

-   Authentication and Authorization Service (AAS)

-   SGX Caching Service (SCS)

-   SGX Host Verification Service (SHVS)

-   Integration HUB (IHUB)

-   Key Broker Service (KBS) with backend key management

-   SGX Quote Verification Service (SQVS)

SGX Agent & SKC Library needs to be installed on SGX Enabled Machine.

Isecl-K8s-extensions must be installed on separate VM.

## Recommended Service Layout & Architecture - Containerized Deployment with K8s

The containerized deployment makes use of Kubernetes orchestrator for single node and multi node deployments. The supported deployment models are as below:

**Single Node:**

![k8s-single-node](./Images/k8s-single-node.png)



**Multi Node:**

![K8s Deployment-sqx](./Images/k8s-deployment-sgx.jpg)



**Services Deployments & Agent DaemonSets:**

Every service including databases will be deployed as separate K8s deployment with 1 replica, i.e(1 pod per deployment). Each deployment will be further exposed through k8s service and also will be having corresponding Persistent Volume Claims(PV) for configuration and log directories and mounted on persistent storage. In case of daemonsets/agents, the configuration and log directories will be mounted on respective Baremetal worker nodes.

![Each pod consist of only one container with service](./Images/pod-deployment.jpg)

For stateful services which requires database like shvs, aas, scs, A separate database deployment will be created for each of such services. The data present on the database deployment will also made to persist on a NFS, through K8s persistent storage mechanism

![database deployment](./Images/shvs-db.jpg)



**Networking within the Cluster:**

![Networking within cluster](./Images/within-cluster-communication-sgx.jpg)



**Networking Outside the Cluster:**

![Networking outside cluster](./Images/inter-cluster-communication-sgx.jpg)

**SKC Virtualization:**

![](.\Images\sgx-virtualization-model.jpg)

Follow the [Installation of Containerized Services and Agent in K8s Cluster](#Installation of Containerized Services and Agent in K8s Cluster) for installation instructions once deployment model is chosen

### Using the provided Database Installation Script 

Install a sample Postgresql 11 database using the install_pgdb.sh script provided in binaries directory. This script will automatically install the Postgresql database and client packages required.

Create the iseclpgdb.env answer file:

ISECL_PGDB_IP_INTERFACES=localhost

ISECL_PGDB_PORT=5432

ISECL_PGDB_SAVE_DB_INSTALL_LOG=true

ISECL_PGDB_CERT_DNS=localhost

ISECL_PGDB_CERT_IP=127.0.0.1

Note that the values above assume that the database will be accessed locally. If the database server will be external to the Intel® SecL services, change these values to the hostname or FQDN and IP address where the client will access the database server.

### Provisioning the Database

Each Intel® SecL service that uses a database (the Authentication and Authorization Service, the SGX host Verification Service, the SGX caching Service,) requires its own schema and access. The database must be created and initialized. Execute the install_pgdb.sh script to install the database.

If a single shared database server will be used for each Intel® SecL service (for example, if all management plane services will be installed on a single VM), run install_pgdb.sh script only once and create_db.sh script for each component that uses a database.

If separate database servers will be used (for example, if the management plane services will reside on separate systems and will use their own local database servers), execute the install_pgdb.sh script on each server hosting a database and create_db.sh script for each component that uses a DB. 

Command to install postgres DB:
```
./install_pgdb.sh
```
Command to create DB for AAS/SCS/SHVS:
```
./create_db.sh <DB Name> <DB Username> <DB Password>
```

Note that the db name, db username and db user password should match with the respective component environment files.

###  Database Server TLS Certificate

The database client for Intel® SecL services requires the database TLS certificate to authenticate communication with the database server.

If the database server for a service is located on the same server that the service will run on, only the path to this certificate is needed. If the provided Postgres scripts are used, the certificate will be in /usr/local/pgsql/data/server.crt.

If the database server will be run separately from the Intel® SecL service(s), the certificate will need to be copied from the database server to the service machine before installing the Intel® SecL services.

The database client for Intel® SecL services will validate that the Subject Alternative Names in the database server's TLS certificate contain the hostname(s)/IP address(es) that the clients will use to access the database server. If configuring a database without using the provided scripts, ensure that these attributes are present in the database TLS certificate.

## Installation of Containerized Services and Agent in K8s Cluster

The containerized deployment utilizes K8s orchestrator to deploy SGX components. The deployments are fairly automated once the pre-reqs are in place for K8s cluster deployments. 

> **Note:** The K8s manifests are declarative in nature and the same can be modified as required for SGX services deployments for single node and multi node deployments. Modifications would require specific steps to ensure services and agents get updated as per the required configuration. More details for the same present in Setup Task Flows for K8s Deployments, Configuration Update Flows for K8s Deployments and [Intel Security Libraries Configuration Settings ](#intel-security-libraries-configuration-settings)

### Pre-requisites

* Ensure based on the deployment model , `microk8s` or `kubeadm` in installed. Supported versions in [Requirements for Containerized Deployment with K8s](#requirements-for-containerized-deployment-with-k8s)
* Docker runtime is configured for each of these deployments. Supported versions in [Requirements for Containerized Deployment with K8s](#requirements-for-containerized-deployment-with-k8s)
* The build would generate a script for platform dependencies under `<working directory>/k8s/platform dependencies`
* Follow the deployment pre-reqs as given in the [Quick Start guide](https://github.com/intel-secl/docs/blob/v5.0/develop/quick-start-guides/SGX%20Infrastructure%20-%20Containerization.md#deployment) based on the chosen deployment model

### Deploy Steps

* The deploy steps are detailed in the [Quick Start guide](https://github.com/intel-secl/docs/blob/v5.0/develop/quick-start-guides/SGX%20Infrastructure%20-%20Containerization.md#deploy-steps) based on the deployment model. Follow the instructions for the deployment using the scripts

### Additional Details

* [Default Service and Agent Mount Paths - Single Node](https://github.com/intel-secl/docs/blob/v5.0/develop/quick-start-guides/SGX%20Infrastructure%20-%20Containerization.md#default-service-and-agent-mount-paths)
* [Default Service and Agent Mount Paths - Multi Node](https://github.com/intel-secl/docs/blob/v5.0/develop/quick-start-guides/SGX%20Infrastructure%20-%20Containerization.md#multi-node-deployments)
* [Default Service Ports](https://github.com/intel-secl/docs/blob/v5.0/develop/quick-start-guides/SGX%20Infrastructure%20-%20Containerization.md#default-service-ports)
* [NFS Setup Pre-reqs - Multi Node](https://github.com/intel-secl/docs/blob/v5.0/develop/quick-start-guides/SGX%20Infrastructure%20-%20Containerization.md#setup-1)

## Installing the Certificate Management Service

### Required For

The CMS is REQUIRED for all use cases.

### Supported Operating System 

The Intel® Security Libraries Certificate Management Service supports Red Hat Enterprise Linux 8.2 and Ubuntu 18.04.5 LTS(Bionic Beaver).

### Recommended Hardware

-   1 vCPUs

-   RAM: 2 GB

-   10 GB

-   One network interface with network access to all Intel® SecL-DC services

### Installation

To install the Intel® SecL-DC Certificate Management Service:

1.  Copy the Certificate Management Service installation binary to the /root/ directory.

2.  Create the `cms.env` installation answer file for an unattended installation:

AAS_TLS_SAN=\< Comma-Separated list of IPs and hostnames for the AAS\>

AAS_API_URL=https://\< Authentication and Authorization Service IP or Hostname\>:8444/aas/v1

SAN_LIST=< Comma-Separated list of IP addresses and hostnames for the CMS>

The SAN list will be used to authenticate the Certificate Signing Request from the AAS to the CMS. Only a CSR originating from a host matching the SAN list will be honored. Later, in the AAS authservice.env installation answer file, this same SAN list will be provided for the AAS installation. These lists must match and must be valid for IPs and/or hostnames used by the AAS system. The SAN list variables also accept the wildcards "?" (for single-character wildcards) and "\*" (for multiple-character wildcards) to allow address ranges or multiple FQDNs.

The AAS_API_URL represents the URL for the AAS that will exist after the AAS is installed.

For all configuration options and their descriptions, refer to the Intel® SecL Configuration section on the Certificate Management Service.

3.  Execute the installer binary.

./cms-v4.0.0.bin

When the installation completes, the Certificate Management Service is available. The services can be verified by running cms status from the command line.

\# cms status

After installation is complete, the CMS will output a bearer token to the console. This token will be used with the AAS during installation to authenticate certificate requests to the CMS. If this token expires or otherwise needs to be recreated, use the following command:

cms setup cms-auth-token \--force

In addition, the SHA384 digest of the CMS TLS certificate will be needed for installation of the remaining Intel® SecL services. The digest can be obtained using the following command:

cms tlscertsha384


## Installing the Authentication and Authorization Service 

### Required For

The AAS is REQUIRED for all use cases.

### Prerequisites

The following must be completed before installing the Authentication and Authorization Service:

-   The Certificate Management Service must be installed and available

-   The Authentication and Authorization Service database must be available

### Package Dependencies

The Intel® SecL-DC Authentication and Authorization Service (AAS) requires a Postgresql 11 database. script (install_pgdb.sh) is provided with the AAS that will install Postgresql repositories.

### Supported Operating Systems

The Intel® Security Libraries Authentication and Authorization Service supports Red Hat Enterprise Linux 8.2 and Ubuntu 18.04.5 LTS(Bionic Beaver).

### Recommended Hardware 

-   1 vCPUs

-   RAM: 2 GB

-   10 GB

-   One network interface with network access to all Intel® SecL-DC services

### Installation


Before AAS is installed, Database needs to be created. Use the following commands to install postgres and create AAS DB

copy install_pgdb.sh and create_db.sh to /root/ directory
```
./install_pgdb.sh

./create_db.sh <db name> <db_user> <db_password>
```

To install the AAS, a bearer token from the CMS is required. This bearer token is output at the end of the CMS installation. However, if a new token is needed, simply use the following command from the CMS command line:

cms setup cms-auth-token --force

Create the authservice.env installation answer file in /root/ directory as below:

      CMS_BASE_URL=https://< CMS IP or hostname>:8445/cms/v1/
    
      CMS_TLS_CERT_SHA384=<CMS TLS certificate sha384>
      
      AAS_DB_SSLMODE=verify-full
    
      AAS_DB_HOSTNAME=<IP or hostname of database server>
    
      AAS_DB_PORT=<database port number; default is 5432>
    
      AAS_DB_NAME=<database name>
    
      AAS_DB_USERNAME=<database username>
    
      AAS_DB_PASSWORD=<database password>
    
      AAS_DB_SSLCERTSRC=<path to database TLS certificate; the default location is typically /usr/local/pgsql/data/server.crt>
    
      AAS_ADMIN_USERNAME=<username for AAS administrative user>
    
      AAS_ADMIN_PASSWORD=<password for AAS administrative user>
    
      AAS_JWT_TOKEN_DURATION_MINS=2880
    
      SAN_LIST=<comma-separated list of IPs and hostnames for the AAS; this should match the value for the AAS_TLS_SAN in the cms.env file from the CMS installation>
    
      BEARER_TOKEN=<bearer token from CMS installation>

Execute the AAS installer:

./authservice-v4.0.0.bin

Note: the AAS_ADMIN credentials specified in this answer file will have administrator rights for the AAS and can be used to create other users, create new roles, and assign roles to users.

### Creating Users

Before deployment is initiated, user account and roles must be generated for each component.  Most of these accounts will be service users, used by the various Intel® SecL SKC services to work together.

Creating these required users and roles is facilitated by the populate-user script.

### Creating Users and Roles 

During installation of each services, number of user accounts and roles specific to services must be generated. Most of these accounts will be service users, which is used by the various services to function together. Another set of users will be used for installation permissions, and administrative user will be created to provide the initial authentication interface for the actual user based on the organizational requirements. Creating these required users and roles is facilitated by a script that will accept credentials and configuration settings from an answer file and automate the process.
Create the `populate-users.env` file using the following values:

```shell
# SKC Components include AAS,SCS,SHVS,SQVS,SIH and SKBS.
ISECL_INSTALL_COMPONENTS=AAS,SCS,SHVS,SQVS,SIH,SKBS

AAS_API_URL=https://<AAS IP address or hostname>:8444/aas/v1
AAS_ADMIN_USERNAME=<AAS username>
AAS_ADMIN_PASSWORD=<AAS password>

IH_CERT_SAN_LIST=<comma-separated list of IPs and hostnames for the Integration Hub>
KBS_CERT_SAN_LIST=<comma-separated list of IPs and hostnames for the Key Broker Service>
SCS_CERT_SAN_LIST=<comma-separated list of IPs and hostnames for the SGX Caching Service>
SQVS_CERT_SAN_LIST=<comma-separated list of IPs and hostnames for the SGX Quote Verification Service>
SHVS_CERT_SAN_LIST=<comma-separated list of IPs and hostnames for the SGX Host Verification Service>

IHUB_SERVICE_USERNAME=<Username for the Hub service user>
IHUB_SERVICE_PASSWORD=<Password for the Hub service user>

SCS_SERVICE_USERNAME=<Username for the SCS service user>
SCS_SERVICE_PASSWORD=<Password for the SCS service user>

SHVS_SERVICE_USERNAME=<Username for the SHVS service user>
SHVS_SERVICE_PASSWORD=<Password for the SHVS service user>

KBS_SERVICE_USERNAME=<Username for the KBS service user>
KBS_SERVICE_PASSWORD=<Password for the KBS service user>

INSTALL_ADMIN_USERNAME=<Username for the Admin user>
INSTALL_ADMIN_PASSWORD=<Password for the Admin user>

CCC_ADMIN_USERNAME=<Username for the Custom Claims Creator Admin user>
CCC_ADMIN_PASSWORD=<Password for the Custom Claims Creator Admin user>

```
> **Note**: The `ISECL_INSTALL_COMPONENTS` variable is a comma-separated list of all the components that will be used in your environment. Not all services are required for every use case. Include only services which are required specific to the use case.

> **Note**: The SAN list variables each support wildcards( "*" and "?"). Using wildcards, domain names and entire IP ranges can be included in the SAN list, which will allow any host matching those ranges to install the relevant service. The SAN list specified here must exactly match the SAN list for the applicable service in that service’s env installation file.

Execute the populate-users script:
```shell
./populate-users
```

The script will automatically generate the following users:

-   Authentication and Authorization Service (AAS)
-   SGX Caching Service (SCS)
-   SGX Host Verificatin Service (SHVS)
-   Integration HUB (IHUB)
-   Key Broker Service (KBS) with backend key management
-   Installation Admin User
-   CCC Admin User

These user accounts will be used during installation of each components of SGX Attestation or SKC. In general, whenever credentials are required by an installation answer file, the variable name should match the name of the corresponding variable used in the `populate-users.env` file.

The populate-users script will also output an installation token. This token has all privileges needed for installation of the services, and uses the credentials provided with the `INSTALL_ADMIN_USERNAME` and `INSTALL_ADMIN_PASSWORD`. The remaining Intel ® SecL-DC services require this token (set as the `BEARER_TOKEN` variable in the installation env files) to grant the appropriate privileges for installation. By default this token will be valid for two hours; the populate-users script can be rerun with the same `populate-users.env` file to regenerate the token if more time is required, or the `INSTALL_ADMIN_USERNAME` and `INSTALL_ADMIN_PASSWORD` can be used to generate an authentication token. 


## Installing the Caching Service

### Required For

The SCS is REQUIRED for the following use cases.

### Prerequisites (CSP & Enterprise)

The following must be completed before installing the SGX Caching Service

-   The Certificate Management Service must be installed and available

-   The Authentication and Authorization Service must be installed and available

-   User needs to subscribe to Intel\'s Provisioning Certificate Service to obtain an API Key

-   The SGX Caching Service database must be available

### Package Dependencies

The Intel® SecL-DC SGX Caching Service (SCS) requires a

Postgresql 11 database. A set of scripts (install_pgdb.sh and create_db.sh) is provided with the SCS

that will automatically add the Postgresql repositories and install/configure a

sample database. If this script will not be used, a Postgresql 11 database

must be installed by the user before executing the SCS installation.

### Supported Operating System 

The Intel® Security Libraries SGX Caching Service supports Red Hat Enterprise Linux 8.2 and Ubuntu 18.04.5 LTS(Bionic Beaver).

### Recommended Hardware 

-   4 vCPUs

-   RAM: 8 GB

-   100 GB

-   One network interface with network access to all managed servers. Internet connection is needed for connecting to Intel PCS Server.

### Installation

Before SCS is installed, Database needs to be created. Use the following commands to install postgres and create SCS DB

copy install_pgdb.sh and create_db.sh to /root/ directory
```
./install_pgdb.sh (if services reside on separate VM)

./create_db.sh <db name> <db_user> <db_password>
```

1. Copy the SCS installation binary to the /root/ directory.

2. Create the scs.env installation answer file in /root/ directory as below:

       SCS_DB_USERNAME=<database username>
        
       SCS_DB_PASSWORD=<database password>
        
       SCS_DB_HOSTNAME=<IP or hostname of database server>
        
       SCS_DB_PORT=<Database port; 5432 by default>
        
       SCS_DB_NAME=<name of the SCS database; pgscsdb by default>
        
       SCS_DB_SSLCERTSRC=<path to database TLS certificate; the default location is typically /usr/local/pgsql/data/server.crt>
        
       INTEL_PROVISIONING_SERVER=<hostname of INTEL PCS Server>
        
       INTEL_PROVISIONING_SERVER_API_KEY=<subscription key>
        
       SCS_REFRESH_HOURS=<time in hours to refresh SGX collaterals; 1 hour by default>
        
       SCS_ADMIN_USERNAME=<username for SCS service account>
        
       SCS_ADMIN_PASSWORD=<password for SCS service account>
        
       CMS_BASE_URL=https://<IP or hostname to CMS>:8445/cms/v1/
        
       CMS_TLS_CERT_SHA384=<sha384 of CMS TLS certificate>
        
       AAS_API_URL=https://<IP or hostname to AAS>:8444/aas/v1
       
       RETRY_COUNT=3
       
       WAIT_TIME=1
        
       SAN_LIST=<comma-separated list of IPs and hostnames for the SCS>
        
       BEARER_TOKEN=<Installation token> 

Update the BEARER_TOKEN with the TOKEN obtained after running populate-users.sh script

Execute the SCS installer binary:

./scs-v4.0.0.bin

## Installing the SGX Host Verification Service 

### Required For

If SGX Host Verification Service API URL is specified in SGX Agent env file, then SGX Agent will push the platform enablement information and TCB status to SHVS at regular interval.

### Prerequisites

The following must be completed before installing the SGX Host Verification Service:

-   The Certificate Management Service must be installed and available

-   The Authentication and Authorization Service must be installed and available

-   The SGX Host Verification Service database must be available


### Package Dependencies

The Intel® Security Libraries SGX Host Verification Service requires the following packages and their dependencies:

-   Postgres\* client and server 11.6 (server component optional if an external Postgres database is used)

-   Golang packages

If they are not already installed, the SGX Host Verification Service installer attempts to install these automatically using the package manager. Automatic installation requires access to package repositories (the RHEL/Ubuntu subscription repositories, the EPEL repository, or a suitable mirror), which may require an Internet connection. If the packages are to be installed from the package repository, be sure to update the repository package lists before installation.

### Supported Operating Systems 

The Intel® Security Libraries SGX Host Verification Service supports Red Hat Enterprise Linux 8.2 and Ubuntu 18.04.5 LTS(Bionic Beaver).

### Recommended Hardware

-   4 vCPUs

-   RAM: 8 GB

-   100 GB

-   One network interface with network access to all managed servers

###  Installation

Before SHVS is installed, Database needs to be created. Use the following commands to install postgres and create SHVS DB

copy install_pgdb.sh and create_db.sh to /root/ directory
```
./install_pgdb.sh (if services reside on separate VM)

./create_db.sh <db name> <db_user> <db_password>
```

To install the SGX Host Verification Service, follow these steps:

-   Copy the SGX Host Verification Service installation binary to the /root/ directory.

-   Create the shvs.env installation answer file in /root/ directory as below.

A sample minimal shvs.env file is provided below. For all configuration options and their descriptions, refer to the Intel® SecL Configuration section on the SGX Host Verification Service.

     SHVS_DB_HOSTNAME=<hostname or IP address to database server> 
    
     SHVS_DB_USERNAME=<Database administrative username> 
    
     SHVS_DB_PORT=<Database port, default is 5432>
    
     SHVS_DB_PASSWORD=<Database password> 
    
     SHVS_DB_NAME=<Database schema>
    
     SHVS_ADMIN_USERNAME=<SGX Host Verification Service username> 
    
     SHVS_ADMIN_PASSWORD=<SGX HostVerification Service password> 
    
     CMS_TLS_CERT_SHA384=<Certificate Management Service TLS digest> 
    
     SHVS_DB_SSLCERTSRC=/usr/local/pgsql/data/server.crt
     
     SHVS_SCHEDULER_TIMER=10
     
     #Maximum allowed time before a platform enablement record in SHVS database is considered as stale
     SHVS_HOST_PLATFORM_EXPIRY_TIME=240
     
     SHVS_AUTO_REFRESH_TIMER=120
    
     BEARER_TOKEN=<Installation token> 
    
     AAS_API_URL=https://<Authentication and Authorization Service IP or Hostname>:8444/aas/v1
    
     CMS_BASE_URL=https://<Certificate Management Service IP or Hostname>:8445/cms/v1/
    
     SCS_BASE_URL=https://<SGX Caching Service IP or Hostname>:9000/scs/sgx/
    
     SAN_LIST=<Comma-separated list of IP addresses and hostnames for the SHVS> 

Update the BEARER_TOKEN with the TOKEN obtained after running populate-users.sh script

Execute the installer binary.

./shvs-v4.0.0.bin

When the installation completes, the SGX Host Verification Service is available. The service can be verified by running **shvs** status from the SGX Host Verification Service command line.

\# shvs status

## Installing the SGX Agent 

### Required for

The SGX Agent is REQUIRED for all use cases. 

The SGX Agent pushes SGX platform data to SGX Caching Service (SCS). SGX Agent gets current TCB Status for the platform from SCS. If SGX Host Verification Service (SHVS) URL is configured, the SGX Agent pushes platform enablement information and TCB Status to SHVS.

### Prerequisites 

-   The following must be completed before installing the SGX Agent:

    -   Certificate Management Service, Authentication and Authorization Service,SGX Caching Service and SGX Host Verification Service must be installed and available.
    -   Make sure system date and time of SGX machine and CSP machine both are in sync. Also, if the system is configured to read the RTC time in the local time zone, then use RTC in UTC by running`timedatectl set-local-rtc 0` command on both the machine. Otherwise SGX Agent deployment will fail with certificate expiry error. 

### Package Dependencies

The Intel® Security Libraries SGX Agent Service requires the following packages and their dependencies:

-   Golang packages

### Supported Operating Systems

Red Hat Enterprise Linux 8.2 and Ubuntu 18.04.5 LTS(Bionic Beaver).

**Recommended Hardware**

Intel® Xeon® SP (Ice Lake-SP)

### Installation
```
Copy sgx_agent.tar, sgx_agent.sha2 and agent_untar.sh from binaries directoy to a directory in SGX compute node

./agent_untar.sh

Edit agent.conf with the following
  - CSP system IP address where CMS, AAS, SHVS and SCS services deployed
  - Network Port numbers for CMS, AAS, SCS and SHVS
  - CSP Admin credentials (same which are provided in service configuration file. for ex: csp_skc.conf, orchestrator.conf or skc.conf)
  - Token validity period in days
  - CMS TLS SHA Value (Run "cms tlscertsha384" on CSP system)

Save and Close

Note: In case orchestration support is not needed, please comment/delete SHVS_IP in agent.conf available in same folder

./deploy_sgx_agent.sh
```
##  Installing the SQVS

### Required for

SGX ECDSA Attestation / SGX Quote Verification by KBS

### Prerequisites 

-   The following must be completed before installing the SQVS:

    -   Certificate Management Service, Authentication and Authorization Service and SGX Caching Service must be installed and available.

### Package Dependencies

The Intel® Security Libraries Quote Verification Service requires the following packages and their dependencies:

-   Golang packages

### Supported Operating Systems

Red Hat Enterprise Linux 8.2 and Ubuntu 18.04.5 LTS(Bionic Beaver).

### Recommended Hardware 

-   4 vCPUs

-   RAM: 8 GB

-   100 GB

-   One network interface with network access to all managed servers

### Installation

To install the SQVS Service, follow these steps:

1.  Copy the SQVS installation binary to the ~/root directory

2.  Copy the Root CA Certificate of the Intel PCS Service (Refer to below Note) from sgx-verification-service/dist/linux/ directory as trusted_rootca.pem to the /tmp directory

NOTE: Retrieve appropriate Trusted RootCA certificate files for SGX platform (trusted_rootca.pem for pre-production systems using IceLake Sandbox PCS, trusted_rootca_icx_prod.pem for production systems using IceLake Live PCS and trusted_rootca_clx_prod.pem for CascadeLake production systems using Live PCS Server) from dist/linux directory in SQVS repository

3.  Create the sqvs.env installation answer file in /root/ directory as below

A sample minimal sqvs.env file is provided below. For all configuration options and their descriptions, refer to the Configuration section on the SGX Quote Verification Service.

       SGX_TRUSTED_ROOT_CA_PATH=< Path where trusted root ca cert for PCS is stored, by default /tmp/trusted_rootca.pem > 
    
       SCS_BASE_URL=https://< SCS IP or Hostname >:9000/scs/sgx/certification/v1
    
       CMS_TLS_CERT_SHA384=< Certificate Management Service TLS digest > 
    
       BEARER_TOKEN=< Installation token > 
    
       AAS_API_URL=https://< Authentication and Authorization Service IP or Hostname >:8444/aas/v1
    
       CMS_BASE_URL=https://< Certificate Management Service IP or Hostname >:8445/cms/v1/ 
    
       SAN_LIST=< *Comma-separated list of IP addresses and hostnames for the SQVS* > 
       
       SQVS_NOSETUP=false
       
       SQVS_PORT=12000
       
       SQVS_LOGLEVEL=info
       
       SQVS_INCLUDE_TOKEN=true

Update the BEARER_TOKEN with the TOKEN obtained after running populate-users.sh script



3.  Execute the sqvs installer binary.

./sqvs-v4.0.0.bin

When the installation completes, the SGX Quote Verification Service is available. The service can be verified by sqvs status from the sqvs command line.

\# sqvs status


## Setup K8S Cluster and Deploy Isecl-k8s-extensions

* Setup master and worker node for k8s. Worker node should be setup on SGX enabled host machine. Master node can be any system.

* To setup k8 cluster on RHEL 8.2, follow https://phoenixnap.com/kb/how-to-install-kubernetes-on-centos 

* To setup k8 cluster on Ubuntu 18.04, follow the "Install, Enable and start the Docker daemon" section in Ubuntu Quick Start Guide - https://github.com/intel-secl/docs/blob/v5.0/develop/quick-start-guides/SGX%20Infrastructure%20-%20Ubuntu.md
Once done, go to https://vitux.com/install-and-deploy-kubernetes-on-ubuntu/ and follow from step 3 onwards.

* Once the master/worker setup is done, follow below steps on Master Node:

### Untar packages and push OCI images to registry

* Copy tar output isecl-k8s-extensions-*.tar.gz from build system's binaries folder to /opt/ directory on the Master Node and extract the contents.
  
  ```shell
    cd /opt/
    tar -xvzf isecl-k8s-extensions-*.tar.gz
    cd isecl-k8s-extensions/
  ```
  
* Configure private registry

* Push images to private registry using skopeo command, (this can be done from build vm also)
  
  ```shell
     skopeo copy oci-archive:isecl-k8s-controller-v4.0.0-<commitid>.tar docker://<registryIP>:<registryPort>/isecl-k8s-controller:v4.0.0
     skopeo copy oci-archive:isecl-k8s-scheduler-v4.0.0-<commitid>.tar docker://<registryIP>:<registryPort>/isecl-k8s-scheduler:v4.0.0
  ```
  
* Add the image names in isecl-controller.yml and isecl-scheduler.yml in /opt/isecl-k8s-extensions/yamls with full image name including registry IP/hostname (e.g <registryIP>:<registryPort>/isecl-k8s-scheduler:v4.0.0). It will automatically pull the images from registry.
  

##### Deploy isecl-controller

* Create hostattributes.crd.isecl.intel.com crd

```
    kubectl apply -f yamls/crd-1.17.yaml
```

* Check whether the crd is created

```
    kubectl get crds
```

* Deploy isecl-controller

```
    kubectl apply -f yamls/isecl-controller.yaml
```

* Check whether the isecl-controller is up and running

```
    kubectl get deploy -n isecl
```

* Create clusterrolebinding for ihub to get access to cluster nodes

```
    kubectl create clusterrolebinding isecl-clusterrole --clusterrole=system:node --user=system:serviceaccount:isecl:isecl
```

* Fetch token required for ihub installation and follow below steps to update ihub.env,

```
    kubectl get secrets -n isecl
    kubectl describe secret default-token-<name> -n isecl
```

For IHUB installation, make sure to update below configuration in /root/binaries/env/ihub.env before installing ihub on CSP VM:

* Copy /etc/kubernetes/pki/apiserver.crt from master node to /root on CSP VM. Update KUBERNETES_CERT_FILE.
* Get k8s token in master, using above commands and update KUBERNETES_TOKEN
* Update the value of CRD name

```
	KUBERNETES_CRD=custom-isecl-sgx
```

##### Deploy isecl-scheduler

The isecl-scheduler default configuration is provided for common cluster support in /opt/isecl-k8s-extensions/yamls/isecl-scheduler.yaml.

Variables HVS_IHUB_PUBLIC_KEY_PATH and SGX_IHUB_PUBLIC_KEY_PATH are by default set to default paths. 

Please use and set only required variables based on the use case.

For example, if only sgx based attestation is required then remove/comment HVS_IHUB_PUBLIC_KEY_PATH variables.

* Install cfssl and cfssljson on Kubernetes Control Plane
```
    #Download cfssl to /usr/local/bin/
    wget -O /usr/local/bin/cfssl http://pkg.cfssl.org/R1.2/cfssl_linux-amd64
    chmod +x /usr/local/bin/cfssl

    #Download cfssljson to /usr/local/bin
    wget -O /usr/local/bin/cfssljson http://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
    chmod +x /usr/local/bin/cfssljson

```

* Create tls key pair for isecl-scheduler service, which is signed by k8s apiserver.crt

```
    cd /opt/isecl-k8s-extensions/
    chmod +x create_k8s_extsched_cert.sh
    ./create_k8s_extsched_cert.sh -n "K8S Extended Scheduler" -s "<K8_MASTER_IP>","<K8_MASTER_HOST>" -c /etc/kubernetes/pki/ca.crt -k /etc/kubernetes/pki/ca.key
```

* After iHub deployment, copy /etc/ihub/ihub_public_key.pem from ihub to /opt/isecl-k8s-extensions/ directory on k8 master vm. Also, copy tls key pair generated in previous step to secrets directory.

```
    mkdir secrets
    cp /opt/isecl-k8s-extensions/server.key secrets/
    cp /opt/isecl-k8s-extensions/server.crt secrets/
	mv /opt/isecl-k8s-extensions/ihub_public_key.pem /opt/isecl-k8s-extensions/sgx_ihub_public_key.pem
    cp /opt/isecl-k8s-extensions/sgx_ihub_public_key.pem secrets/
```

Note: Prefix the attestation type for ihub_public_key.pem before copying to secrets folder.

* Create kubernetes secrets scheduler-secret for isecl-scheduler

```
    kubectl create secret generic scheduler-certs --namespace isecl --from-file=secrets
```

* Deploy isecl-scheduler

```
    kubectl apply -f yamls/isecl-scheduler.yaml
```

* Check whether the isecl-scheduler is up and running

```
    kubectl get deploy -n isecl
```

##### Configure kube-scheduler to establish communication with isecl-scheduler

* Add scheduler-policy.json under kube-scheduler section, mountPath under container section and hostPath under volumes section in /etc/kubernetes/manifests/kube-scheduler.yaml as mentioned below

```
spec:
  containers:
  - command:
    - kube-scheduler
    - --policy-config-file=/opt/isecl-k8s-extensions/scheduler-policy.json
```

```
  containers:
    volumeMounts:
    - mountPath: /opt/isecl-k8s-extensions/
      name: extendedsched
      readOnly: true
```

```
  volumes:
  - hostPath:
      path: /opt/isecl-k8s-extensions/
      type:
    name: extendedsched
```

Note: Make sure to use proper indentation and don't delete existing mountPath and hostPath sections in kube-scheduler.yaml.

* Restart Kubelet which restart all the k8s services including kube base schedular

```
	systemctl restart kubelet
```

* Check if CRD Data is populated

```
	kubectl get -o json hostattributes.crd.isecl.intel.com
```

##  Installing the Integration Hub

**Note:** The Integration Hub is only required to integrate Intel® SecL with third-party scheduler services, such as Kubernetes. The Integration Hub is not required for usage models that do not require Intel® SecL security attributes to be pushed to an integration endpoint.

### Required For

The Integration Hub is REQUIRED the for enabling support for orchestration support.

### Prerequisites

The Intel® Security Libraries Integration Hub can be run on a VM or on a bare-metal server. The Integration Hub may be installed on the same server (physical or VM) as the SGX Host Verification Service.

-   SGX Caching Service must be installed and available.

-   The SGX Host Verification Service must be installed and available

-   The Authentication and Authorization Service must be installed and available

-   The Certificate Management Service must be installed and available


### Package Dependencies

The Intel® SecL Integration Hub requires a number of packages and their dependencies:

-   Golang packages

If these are not already installed, the Integration Hub installer attempts to install these packages automatically using the package manager. Automatic installation requires access to package repositories (the RHEL/Ubuntu subscription repositories, the EPEL repository, or a suitable mirror), which may require an Internet connection. If the packages are to be installed from the package repository, be sure to update your repository package lists before installation.

### Supported Operating Systems

The Intel Security Libraries Integration Hub supports Red Hat Enterprise Linux 8.2 and Ubuntu 18.04.5 LTS(Bionic Beaver).

### Recommended Hardware

-   1 vCPUs

-   RAM: 2 GB

-   1 GB free space to install the Integration Hub (database and log space requirements are dependent on the number of managed servers).

-   One network interface with network access to the SGX Host Verification Service.

-   One network interface with network access to any integration endpoints (for example, OpenStack Nova).

#### Installing the Integration Hub

To install the SGX Integration Hub, follow these steps:

1. Copy the Integration Hub installation binary to the /root/ directory.

2. Create the ihub.env installation answer file in /root/ directory as below
```
    IHUB_SERVICE_USERNAME=< IHUB service user username > 
    IHUB_SERVICE_PASSWORD=< IHUB service user password > 
    SHVS_BASE_URL=< https://< SHVS IP or Hostname >:13000/sgx-hvs/v2
    CMS_TLS_CERT_SHA384=< CMS TLS digest > 
    BEARER_TOKEN=<Installation token> 
 
    AAS_API_URL=https://< AAS IP or Hostname >:8444/aas/v1
    CMS_BASE_URL=https://< CMS IP or Hostname >:8445/cms/v1
    POLL_INTERVAL_MINUTES=2
    TLS_SAN_LIST=< comma separated list of IPs and hostnames for the IHUB >
    TENANT=< tenant-type e.g. KUBERNETES or OPENSTACK >

    # Kubernetes Integration Credentials - required for Kubernetes integration only
    KUBERNETES_URL=< https://< Kubernetes IP >:6443/>
    KUBERNETES_CRD=custom-isecl-sgx
    KUBERNETES_TOKEN=< K8S token >
    KUBERNETES_CERT_FILE =< Path of Kubernetes master node certificate >

    # OpenStack Integration Credentials - required for OpenStack integration only
    OPENSTACK_AUTH_URL=<OpenStack Keystone URL; typically http://openstack-ip:5000/>
    OPENSTACK_PLACEMENT_URL=<OpenStack Nova Placement API URL; typically http://openstack-ip:8778/>
    OPENSTACK_USERNAME=< OpenStack username >
    OPENSTACK_PASSWORD=< OpenStack password >
```
3.  Create Integrated Hub Service user account and Roles. A sample script is provided in the appendix section for reference

Update the BEARER_TOKEN with the TOKEN obtained after running populate-users.sh script

4.  Execute the installer binary.

./ihub-v4.0.0.bin

In case installation fails, its recommended to run the following command to clear failed service instance

```
systemctl reset-failed 
```

Copy IHUB public key to the master node and restart kubelet.

```
    scp -r /etc/ihub/ihub_public_key.pem <master-node IP>:/opt/isecl-k8s-extensions/
    systemctl restart kubelet
```

Run this command to validate if the data has been pushed to CRD: 

```
    kubectl get -o json hostattributes.crd.isecl.intel.com
```

Run this command to validate that the labels have been populated:

```
    kubectl get nodes --show-labels.
```

Sample labels:

```
    EPC-Memory=2.0GB,FLC-Enabled=true,SGX-Enabled=true,SGX-Supported=true,SgxTrustExpiry=2020-11-09T08.07.43Z,TCBUpToDate=true
```

Create sample yml file for nginx workload and add SGX labels to it such as:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    name: nginx
spec:
  affinity:
    nodeAffinity:
     requiredDuringSchedulingIgnoredDuringExecution:
       nodeSelectorTerms:
       - matchExpressions:
         - key: SGX-Enabled
           operator: In
           values:
           - "true"
         - key: EPC-Memory
           operator: In
           values:
           - "2.0GB"
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```

Validate if pod can be launched on the node. Run following commands:

```
    kubectl apply -f pod.yml
    kubectl get pods
    kubectl describe pods nginx 
```

Pod should be in running state and launched on the host as per values in pod.yml. Validate running below commands on sgx host:

```
	docker ps
```

## Integration with OpenStack (Supported only on RHEL 8.2, not supported on Ubuntu 18.04)

OpenStack can now use “Traits” to provide qualitative data about Nova Compute hosts to establish Trait requirements. The Integration Hub continually push SGX data to the OpenStack Traits resources. This means OpenStack scheduler natively supports workload scheduling incorporating SGX Host information, including SGX enabled/disabled, SGX supported/not supported, FLC enabled/not enabled, EPC memory size, TCB status upto date/not. The OpenStack Placement Service will automatically attempt to place images with Trait requirements on compute nodes that have those Traits.

NOTE: This control only applies to instances launched using the OpenStack scheduler, and the Traits functions will not affect manually-launched instances where a specific Compute Node is defined (since this does not use the scheduler at all). Intel SecL-DC uses existing OpenStack interfaces and does not modify OpenStack code.  The datacenter owner or OpenStack administrator is responsible for the security of the OpenStack workload scheduling process in general, and Intel recommends following published OpenStack security best practices.

Setting Image Traits
Image Traits define the policy for which Traits are required for that instance to be launched on a Nova Compute node.By setting these Traits to “required” the OpenStack scheduler will require the same Traits to be present on a Nova Compute node in order to launch instances. To set the Image Traits for Intel SecL-DC,a specific naming convention is used. This naming convention will match the Traits that the Integration Hub will automatically push to OpenStack. Two types of Traits are currently supported – one Trait is used to require that the Compute Node must be SGX supported and the other Trait is used to require specific SGXkey/value pairs.
Required Image trait for SGX Enabled Host:
```
CUSTOM_ISECL_SGX_ENABLED_TRUE=required
```
These Traits can be set using CLI commands for OpenStack Glance:
```
openstack image set --property trait:CUSTOM_ISECL_SGX_ENABLED_TRUE=required <image name>
```
To veiw the Traits that has been set:
```
openstack image show
```
List the set of resources mapped to the Openstack
```
openstack resource provider list
```
To view the traits enabled for the SGX Host:
```
openstack resource provider trait list <uuid of the host which the openstack resoruce provider lists>
```
Create the instances
```
openstack server create --flavor tiny --image <image name> --net vmnet <vm instance name>
```
Instances should be created and the status should be "Active". Instance should be launched successfully.
```
openstack server list
```
To remove a Trait that is not required for an Image:
```
openstack image unset --property trait:CUSTOM_ISECL_SGX_ENABLED_TRUE 
openstack image unset --property trait:CUSTOM_ISECL_SGX_ENABLED_FALSE 
```
Scheduling Instances
Once Trait requirements are set for Images and the Integration Hub is configured to push attributes to OpenStack, instances can be launched in OpenStack as normal. As long as the OpenStack Nova scheduler is used to schedule the workloads, only compliant Compute Nodes will be scheduled to run instances of controlled Images.

NOTE: This control only applies to instances launched using the OpenStack scheduler and the Traits functions will not affect manually-launched instances where a specific Compute Node is defined (since this does not use the scheduler at all). Intel SecL-DC uses existing OpenStack interfaces and does not modify OpenStack code. The datacenter owner or OpenStack administrator is responsible for the security of the
OpenStack workload scheduling process in general and Intel recommends following published OpenStack security best practices.


##  Installing the Key Broker Service

### Required for

The KBS is REQUIRED for

\- Storing Application Keys and Verifying the SGX Quote

### Prerequisites

The following must be completed before installing the Key Broker:

-   The Authentication and Authorization Service must be installed and available

-   The Certificate Management Service must be installed and available

If a 3rd-party Key Management server will be used following must be completed before installing the Key Broker:

-   A KMIP 3rd-party Key management Server must be available.

-   The Key Broker will require the KMIP server’s client certificate, client key and root ca certificate. This key and certificate will be available in KMIP server.

NOTE: The Key Broker has been validated using the pykmip 0.9.1 KMIP server as a 3rd-party Key Management Server. While any general KMIP 2.0-compliant Key Management Server should work, implementation differences among KMIP providers may prevent functionality with specific providers.

### Package Dependencies

N/A

### Supported Operating Systems

supports Red Hat Enterprise Linux 8.2 and Ubuntu 18.04.5 LTS(Bionic Beaver).

### Recommended Hardware

NA

### Installation

1. Copy the Key Broker installation binary to the /root/ directory.

2. Create the installation answer file kbs.env /root/ directory as below:

       KBS_SERVICE_USERNAME=< KBS service user username > 
        
       KBS_SERVICE_PASSWORD=< KBS service user password > 
       
       SERVER_PORT=9443
        
       AAS_API_URL=https://<AAS IP or hostname>:8444/aas/v1
        
       CMS_BASE_URL=https://<CMS IP or hostname>:8445/cms/v1/
        
       SQVS_URL=https://<SQVS IP or hostname>:12000/svs/v1
       
       ### KEY_MANAGER is set to KMIP
        
       KEY_MANAGER=KMIP
       
       ENDPOINT_URL=https://<KBS Hostname>:9443/v1
       
       TLS_COMMON_NAME="KBS TLS Certificate"
       
       SKC_CHALLENGE_TYPE="SGX"
        
       CMS_TLS_CERT_SHA384=<SHA384 hash of CMS TLS certificate>
        
       TLS_SAN_LIST=<KBS Hostname/IP>
        
       BEARER_TOKEN=<Installation token from AAS>
       
       ## Session Expiry Time Between KBS and SKC Library in Minutes
       SESSION_EXPIRY_TIME=60
       
       KMIP_SERVER_IP=<IP address of KMIP server>
       
       KMIP_SERVER_PORT=<Port number of KMIP server>
       
       ## KMIP_VERSION variable can be used to mention KMIP protocol version.
       ## This is an OPTIONAL field, default value is set to '2.0'. KBS supports KMIP version '1.4' and '2.0'.
       KMIP_VERSION=<kmip version>
       
       ## KMIP_HOSTNAME can be used to configure TLS config with ServerName.
       ## KMIP server certificate should contain SAN(IP/DNS) or valid COMMON NAME and this value can be provided in KMIP_HOSTNAME. Only FQDN names are allowed.
       ## This is an OPTIONAL field; if KMIP_HOSTNAME is not provided then KMIP_SERVER_IP will be considered as ServerName in TLS configuration.
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

Update the BEARER_TOKEN with the TOKEN obtained after running populate-users.sh script

4.  Execute the KBS installer.

./kbs-4.0.0.bin

NOTE: When a 3rd-party Key Management Server is used, KBS supports only association of RSA key. User needs to create RSA key in backend kmip server and note the Private key Id. User also needs to create the tls certificate for the private key (it will be used during key transfer).
rsa-create.py available in kbs_scripts can be used to create the private key and generate the certificate by providing the kmip server ip and certificates path in the script.

##  Installing the SKC Library

### Required For 

The SKC_Library enables secure transfer of application keys from KBS after performing SGX attestation. It stores the keys in the SGX enclave and performs crypto operations ensuring the keys are never exposed in use, at rest and in transit outside of enclave.

### Package Dependencies 

The Intel® Security Libraries SKC Library requires the following packages and their dependencies

Openssl

Curl

### Supported Operation System 

The Intel® Security Libraries SKC Library supports Red Hat Enterprise Linux 8.2 and Ubuntu 18.04.5 LTS(Bionic Beaver).

### Recommended Hardware 

-   Icelake Server with SGX enabled in BIOS

-   RAM: 8 GB

-   100 GB

-   One network interface with network access to the Key Broker

### Installation
```
Copy skc_library.tar, skc_library.sha2 and skclib_untar.sh from binaries directoy to a directory in SGX compute node

./skclib_untar.sh

Update create_roles.conf with the following
  - IP address of AAS deployed on Enterprise system
  - Admin account credentials of AAS deployed on Enterprise system. These credentials should match with the AAS admin credentials provided in authservice.env on enterprise side.
  - Permission string to be embedded into skc_libraty client TLS Certificate
  - For Each SKC Library installation on a SGX compute node, please change SKC_USER and SKC_USER_PASSWORD

Save and Close

./skc_library_create_roles.sh
Copy the token printed on console.

Update skc_library.conf with the following
  - IP address for CMS and KBS services deployed on Enterprise system
  - CSP_CMS_IP should point to the IP of CMS service deployed on CSP system
  - CSP_SCS_IP should point to the IP of SCS service deployed on CSP system
  - Hostname of the Enterprise system where KBS is deployed
  - Network Port numbers for CMS and SCS services deployed on CSP system
  - Network Port numbers for CMS and KBS services deployed on Enterprise system
  - For Each SKC Library installation on a SGX compute node, please change SKC_USER (should be same as SKC_USER provided in create_roles.conf)
  - SKC_TOKEN with the token copied from previous step

Save and Close

./deploy_skc_library.sh
```
#### Deploying SKC Library as a Container (Supported only on RHEL 8.2, not supported on Ubuntu 18.04)
```
Use the following steps to configure SKC library running in a container and to validate key transfer in container on bare metal and inside a VM on SGX enabled hosts.

Note: All the configuration files required for SKC Library container are modified in the resources directory only 

1. Docker should be installed, enabled and services should be active

2. In the build System, SKC Library tar file "<skc-lib*>.tar" required to load is located in the "/root/workspace/skc_library" directory.  

3. Copy "resources" folder from "workspace/skc_library/container/resources" to the "/root/" directory of SGX host. Inside the resources folder all the key transfer flow related files will be available.

4. Update sgx_default_qcnl.conf file inside resources folder with SCS IP and SCS port and update hosts file present in same folder with KBS IP and hostname.

5. Generate the RSA key in the kbs host and copy it to SGX host.

6. Refer to openssl and nginx sub sections of QSG in the "Configuration for NGINX testing" to configure nginx.conf and openssl.conf present resource in the directory.

7. Update keyID in the keys.txt and nginx.conf. 

8. Under [core] section of pkcs11-apimodule.ini in the "/root/resources/" directory add preload_keys=/root/keys.txt.

9. Update skc_library.conf with IP addresses where SKC services are deployed.

10. On the SGX Compute node, load the skc library docker image provided in the tar file. 
   docker load < <SKC_Library>.tar
   
11. Provide valid paramenets in the docker run command and execute the docker run command. Update the genertaed RSA Key ID and <keys>.crt in the resources directory.
    docker run -p 8080:2443 -p 80:8080 --mount type=bind,source=/root/<KBS_cert>.crt,target=/root/<KBS_cert>.crt --mount type=bind,source=/root/resources/sgx_default_qcnl.conf,target=/etc/sgx_default_qcnl.conf --mount type=bind,source=/root/resources/nginx.conf,target=/etc/nginx/nginx.conf --mount type=bind,source=/root/resources/keys.txt,target=/root/keys.txt,readonly --mount type=bind,source=/root/resources/pkcs11-apimodule.ini,target=/opt/skc/etc/pkcs11-apimodule.ini,readonly --mount type=bind,source=/root/resources/openssl.cnf,target=/etc/pki/tls/openssl.cnf --mount type=bind,source=/root/resources/skc_library.conf,target=/skc_library.conf --add-host=<SHC_HOSTNAME>:<SGX_HOST_IP> --add-host=<KBS_Hostname>:<KBS host IP> --mount type=bind,source=/dev/sgx,target=/dev/sgx --cap-add=SYS_MODULE --privileged=true <SKC_LIBRARY_IMAGE_NAME>
    
    Note: In the above docker run command, source refers to the actual path of the files located on the host and the target always refers to the files which would be mounted inside the container
  
12. Restore index.html for the transferred key inside the container
    Get the container id using "docker ps" command
    docker exec -it <container_id> /bin/sh 
   
    Download index.html
    wget https://localhost:2443 --no-check-certificate
```

# Authentication

Authentication is centrally managed by the Authentication and Authorization Service (AAS). This service uses a Bearer Token authentication method. This service also centralizes the creation of roles and users, allowing much easier management of users, passwords, and permissions across all Intel® SecL-DC services.

To make an API request to an Intel® SecL-DC service, an authentication token is required. API requests must now include an Authorization header with a valid token

The token is issued by AAS and expires after a set amount of time. This token may be used with any Intel® SecL-DC service and will carry the appropriate permissions for the role(s) assigned to the account the token was generated for.

The SKC solution involves AAS deployments for 2 different domains: the CSP domain and the tenant domain. There is no trust relationship between the 2 deployments.

In SKC, the accounts of the SGX Services are created at install time. However, CSP admin users must obtain AAS tokens to invoke admin APIs in the SGX Host Verification Service (SHVS), the SGX Hub, the SGX Caching Service (SCS) and AAS.

Similarly, the tenant admin needs AAS tokens to invoke Create, Read, Update and Delete (CRUD) APIs in KBS and admin APIs in AAS.

The following sections present how to use AAS APIs to create tokens and manage users.

## Create Token

To request a new token from the AAS:

POST https://\<AAS IP or hostname\>:8444/aas/v1/token

{

\"username\" : \"\<username\>\",

\"password\" : \"\<password\>\"

}

The response will be a token that can be used in the Authorization header for other requests. The length of time for which the token will be valid is configured on the AAS using the key ~AAS_JWT_TOKEN_DURATION_MINS~ (in the installation answer file during installation) or aas.jwt.token.duration.mins (configured on the AAS after installation). In both cases the value is the length of time in minutes that issued tokens will remain valid before expiring.

## User Management

Users in Intel® SecL-DC are centrally managed by the Authentication and Authorization Service (AAS). Any user may be assigned roles for any service, allowing user accounts to be fully defined by the tasks needed

### Username and Password Requirement

Passwords have the following constraints:

-   cannot be empty - ie must at least have one character

-   maximum length of 255 characters

Usernames have the following requirements:

-   Format: username\[\@host_name\[domain\]\]

-   \[\@host_name\[domain\]\] is optional

-   username shall be minimum of 2 and maximum of 255 characters

-   username allowed characters are alphanumeric, ., -, \_ - but cannot start with -.

-   Domain name must meet requirements of a host name or fully qualified internet host name

-   (Update it relevant to SKC)

### Create User 

POST https://\<IP or hostname of AAS\>:8444/aas/v1/users

Authorization: Bearer \<token\>

{

\"username\" : \"\<username\>\",

\"password\" : \"\<password\>\"

}

### Search Users by Username

GET https://\<IP or hostname of AAS\>:8444/aas/v1/users?name=\<value\>

Authorization: Bearer \<token\>

### Change User Password

PATCH https://\<IP or hostname of AAS\>:8444/aas/v1/users/changepassword

{

\"username\": \"\<username\>\",

\"old_password\": \"\<old_password\>\",

\"new_password\": \"\<new_password\>\",

\"password_confirm\": \"\<new_password\>\" }

### Delete User

DELETE https://\<IP or hostname of AAS\>:8444/aas/v1/users/\<User ID\>

Authorization: Bearer \<token\>

## Roles and Permission

Permissions in Intel® SecL-DC are managed by Roles. Roles are a set of predefined permissions applicable to a specific service. Any number of Roles may be applied to a User. While new Roles can be created, each Intel® SecL service defines permissions that are applicable to specific predetermined Roles. This means that only pre-defined Roles will actually have any permissions. Role creation is intended to allow Intel® SecL-DC services to define their permissions while allowing role and user management to be centrally managed on the AAS. When a new service is installed, it will use the Role creation functions to define roles applicable for that service in the AAS.

### Create Roles

POST https://\<AAS IP or Hostname\>:8444/aas/v1/roles

Authorization: Bearer \<token\>

{

\"service\": \"\<Service name\>\",

\"name\": \"\<Role Name\>\".

"permissions": \[\<array of permissions\>\]

}

-   Service field contains a minimum of 1 and maximum of 20 characters. Allowed characters are alphanumeric plus the special charecters -, \_, @, ., ,

-   Name field contains a minimum of 1 and maximum of 40 characters. Allowed characters are alphanumeric plus the special characters -, \_, @, ., ,

-   Service and Name fields are mandatory

-   Context field is optional and can contain up to 512 characters. Allowed characters are alphanumeric plus -, \_, @, ., ,,=,;,:,\*

-   Permissions field is optional and allow up to a maximum of 512 characters.

The Permissions array must a comma-separated list of permissions formatted as resource:action:

Permissions required to execute specific API requests are listed with the API resource and method definitions in the API documentation.

### Search Roles

GET https://\<AAS IP or Hostname\>:8444/aas/v1/roles?\<parameter\>=\<value\>

Authorization: Bearer \<token\>

Search parameters supported:

Service=\<name of service\>

Name=\<role name\>

Context=\<context\>

contextContains=\<partial "context" string\>

allContexts=\<true or false\> filter=false

### Delete Role

DELETE https://\<AAS IP or Hostname\>:8444/aas/v1/roles/\<role ID\> Authorization:

Bearer \<token\>

### Assign Role to User

POST https://\<AAS IP or Hostname\>:8444/aas/v1/users/\<user ID\>/roles

Authorization: Bearer \<token\>

{

\"role_ids\": \[\"\<comma-separated list of role IDs\>\"\]

}

### List Roles Assigned to User 

GET https://\<AAS IP or Hostname\>:8444/aas/v1/users/\<user ID\>/roles

Authorization: Bearer \<token\>

### Remove Role from User 

DELETE https://\<AAS IP or Hostname\>:8444/aas/v1/users/\<user ID\>/roles/\<role ID\>

Authorization: Bearer \<token\>

### Role Definitions 

Following are the set of roles which are required during installation and runtime.

| Role Name                                                    | Permissions                                                  | Utility                                                      |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| < SHVS:HostDataUpdater: >                                    |                                                              | Used by the SGX_Agent to push host data to the SHVS          |
| < SHVS:HostsListReader: >                                    |                                                              | Used by the IHUB to retrieve the list of hosts from SHVS     |
| < SHVS:HostDataReader: >                                     |                                                              | Used by the IHUB to retrieve platform-data from SHVS         |
| < CMS:CertApprover:CN=SHVS TLS Certificate;SAN=<san list>;CERTTYPE=TLS> |                                                   | Used by the SHVS to retrieve TLS Certificate from CMS        |
| < CMS:CertApprover:CN=Integration HUB TLS Certificate;SAN=<san list>;CERTTYPE=TLS> |                                        | Used by the IHUB to retrieve TLS Certificate from CMS        |
| < SCS:HostDataUpdater: >                                     |                                                              | Used by the SGX_Agent to push the platform-info to SCS       |
| < SCS:HostDataReader: >                                      |                                                              | Used by the SGX_Agent to retrieve the TCB status info from SCS|
| < SCS:CacheManager: >                                        |                                                              | Used by the SCS admin to refresh the platform info           |
| < CMS:CertApprover:CN=SCS TLS Certificate;SAN=<san list>;CERTTYPE=TLS> |                                                    | Used by the SCS to retrieve TLS Certificate from CMS         |
| < KBS:KeyTransfer:permissions=nginx,USA >                    |                                                              | Used by the SKC Library user for Key Transfer                |
| < CMS:CertApprover:CN=skcuser;CERTTYPE=TLS-Client>           |                                                              | Used by the SKC Library user to retrieve TLS-Client Certificate from CMS |
| < CMS:CertApprover:CN=KBS TLS Certificate;SAN=<san list>;CERTTYPE=TLS> |                                                    | Used by the KBS to retrieve TLS Certificate from CMS         |
| AAS: Administrator                                           | *:*:*                                                        | Administrator role for the AAS only. Has all permissions for AAS resources, including the ability to create or delete users and roles |
| AAS: RoleManager                                             | AAS: [roles:create:*, roles:retrieve:*, roles:search:*, roles:delete:*] | AAS role that allows all actions for Roles but cannot create or delete Users or assign Roles to Users. |
| AAS: UserManager                                             | AAS: [users:create:*, users:retrieve:*, users:store:*, users:search:*, users:delete:*] | AAS role with all permissions for Users but has no ability to create Roles or assign Roles to Users. |
| AAS: UserRoleManager                                         | AAS: [user_roles:create:*, user_roles:retrieve:*, user_roles:search:*, user_roles:delete:*] | AAS role with permissions to assign Roles to Users but cannot create delete or modify Users or Roles. |
| < SHVS:HostListManager:>                                     |                                                              | Used by the SHVS admin to delete the hosts.                  |
| < SQVS:QuoteVerifier: >                                      |                                                              | Used by the KBS service user for quote verification          |


## SGX Agent

The SGX Agent communicates with SGX Caching Service (SCS) and SGX Host Verification Service (SHVS) directly. Authentication has been centralized with the new Authentication and Authorization Service.

# SGX Features Provisioning

## Host Registration 

Host Registration creates a host record with host information in the SGX Host Verification Service database when SGX Agent update SGX enablement information for the first time.

# Setup Task Flows for K8s Deployments

Setup tasks flows have been updated to have K8s native flow to be more agnostic to K8s workflows. Following would be the flow for setup task. The setup task options are detailed under [Intel Security Libraries Configuration Settings](#intel-security-libraries-configuration-settings) for each service and agent

- User would create a new `configMap`  object with the environment variables specific to the setup task. The Setup task variables would be documented in the Product Guide
- Users can provide variables for more than one setup task
- If variables involve BEARER_TOKEN then user need to delete the corresponding secret using `kubectl delete secret <service_name>-secret -n isecl` , update the new token inside secrets.txt file and re-create the secret using `kubectl create secret generic <service_name>-secret --from-file=secrets.txt --namespace=isecl`, else update the variables in configMap.yml
- Users need to add `SETUP_TASK: "<setup task name>/<comma separated setup task name>"` in the same `configMap`
- Provide a unique name to the new `configMap`
- Provide the same name in the `deployment.yaml` under `configMapRef` section
- Deploy the specific service again with  ` kubectl kustomize . | kubectl  apply -f -`

# Configuration Update Flows for K8s Deployments

Configuration Update flows have been updated to have K8s native flow to be more agnostic to K8s workflows using `configMap` only. Following would be the flow for configuration update. The config task options are are detailed under [Intel Security Libraries Configuration Settings](#intel-security-libraries-configuration-settings) for each service and agent

- User would create a new `configMap`  object using existing one and update the new values. The list of config variables would be documented in the Product Guide
- Users can update variables for more than one
- Users need to add `SETUP_TASK: "update-service-config"` in the same `configMap`
- Provide a unique name to the new `configMap`
- Provide the same name in the `deployment.yaml` under `configMapRef` section
- Deploy the specific service again with  ` kubectl  kustomize . | kubectl  apply -f -`

> **Note:** Incase of agents, setup tasks or configuration updates done through above flows will be applied for all the agents running on different BMs. In order to run setup task or update configuration for individual agents, then user need to perform `kubectl exec -it <pod_name> /bin/bash` into a particular agent pod and run the specific setup task.

# Intel Security Libraries Configuration Settings 

> **Note:** All the answer file options would remain common for containerized K8s deployments with the except of URLS where Kubernetes DNS would be used. The respective `configMap.yml` for each service and agent would carry the defaults for the same when built under `<working directory>/k8s/manifests/<service/agent/db names>`

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


### Configuration Options 

The SGX Host Verification Service configuration is in path /etc/shvs/config.yml.

### Command-Line Options 

The SGX Host Verification Service supports several command-line options that can be executed only as the Root user:

Syntax:

shvs \<command\>

#### Available Commands

#### Help 

shvs help

Displays the list of available CLI commands.

#### Start 

shvs start

Starts the SGX Host Verification service

#### Stop 

shvs stop

Stops the SGX Host Verification service

#### Status 

shvs status

Reports whether the service is currently running.

#### Uninstall 

shvs uninstall \[\--purge\]

Removes the service. Use \--purge option to remove configuration directory(/etc/shvs/)

#### Version 

shvs version

Shows the version of the service.

#### Setup \[task\]

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

### Directory Layout 

The SGX Host Verification Service installs by default to /opt/shvs with the following folders.

#### Bin 

This folder contains executable scripts.

#### Configuration 

This folder /etc/shvs contains certificates, keys, and configuration files.

#### Logs 

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

`#### Setup \[task\]

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

ihub \<command\>

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

ihub uninstall \[\--purge\] \[\--exec\]

Removes the service. Use \--purge option to remove configuration directory(/etc/ihub/). Use \--exec option to remove ihub instance specific directories

#### Version 

ihub version

Reports the version of the service.

#### Setup \[task\]

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

cms \<command\>

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

cms uninstall \[\--purge\]

Uninstalls the service, including the deletion of all files and folders.

#### Version 

cms version

Reports the version of the service.

#### Tlscertsha384

cms tlscertsha384

Shows the SHA384 digest of the TLS certificate.

#### Setup \[task\] 

Runs a specific setup task.

Syntax:

cms setup [task]

Available Tasks for setup:

##### cms setup server \[\--port=\<port\>\] 

-   Setup http server on \<port\>

-   Environment variable CMS_PORT=\<port\> can be set alternatively

#####  cms setup root_ca \[\--force\] 

-   Create its own self signed Root CA keypair in /etc/cms for quality of life

-   Option \[\--force\] overwrites any existing files, and always generate new Root CA keypair

##### cms setup tls \[\--force\] \[\--host_names=\<host_names\>\] 

-   Create its own root_ca signed TLS keypair in /etc/cms for quality of life

-   Option \[\--force\] overwrites any existing files, and always generate root_ca signed TLS keypair

-   Argument \<host_names\> is a list of host names used by local machine, seperated by comma

-   Environment variable CMS_HOST_NAMES=\<host_names\> can be set alternatively

##### cms setup cms-auth-token \[\--force\]

-   Create its own self signed JWT keypair in /etc/cms/jwt for quality of life

-   Option \[\--force\] overwrites any existing files, and always generate new

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

authservice uninstall \[\--purge\]

Removes the service. Use the "\--purge" flag to also delete all data.

#### Version

authservice version

Shows the version of the service.

#### Setup \[task\]

Executes a specific setup task. Can be used to change the current configuration.

Syntax:

authservice setup [task]

Available Tasks for setup:

##### authservice setup all 

 Runs all setup tasks

##### authservice setup database \[-force\] \[-arguments=\<argument_value\>\] 

-   Available arguments are:

-   db-host alternatively, set environment variable AAS_DB_HOSTNAME

-   db-port alternatively, set environment variable AAS_DB_PORT

-   db-user alternatively, set environment variable AAS_DB_USERNAME

-   db-pass alternatively, set environment variable AAS_DB_PASSWORD

-   db-name alternatively, set environment variable AAS_DB_NAME

-   db-sslmode \<disable\|allow\|prefer\|require\|verify-ca\|verify-full\> alternatively, set environment variable AAS_DB_SSLMODE

-   db-sslcert path to where the certificate file of database. Only applicable for db-sslmode=\<verify-ca\|verify-full. If left empty, the cert will be copied to /etc/authservice/tdcertdb.pem alternatively, set environment variable AAS_DB_SSLCERT

-   db-sslcertsrc \<path to where the database ssl/tls certificate file\>

mandatory if db-sslcert does not already exist alternatively, set environment variable AAS_DB_SSLCERTSRC

-   Run this command with environment variable AAS_DB_REPORT_MAX_ROWS and AAS_DB_REPORT_NUM_ROTATIONS can update db rotation arguments

##### authservice setup server \[\--port=\<port\>\] 

-   Setup http server on \<port\>

-   Environment variable AAS_PORT=\<port\> can be set alternatively authservice setup tls \[\--force\] \[\--host_names=\<host_names\>\]

-   Use the key and certificate provided in /etc/threat-detection if files exist

-   Otherwise create its own self-signed TLS keypair in /etc/authservice for quality of life

-   Option \[\--force\] overwrites any existing files, and always generate self-signed keypair

-   Argument \<host_names\> is a list of host names used by local machine, seperated by comma

-   Environment variable AAS_TLS_HOST_NAMES=\<host_names\> can be set alternatively

##### authservice setup admin \[\--user=\<username\>\] \[-pass=\<password\>\] 

-   Environment variable AAS_ADMIN_USERNAME=\<username\> can be set alternatively

-   Environment variable AAS_ADMIN_PASSWORD=\<password\> can be set alternatively

##### authservice setup jwt 

-   Create jwt signing key and jwt certificate signed by CMS

-   Environment variable CMS_BASE_URL=\<url\> for CMS API url

-   Environment variable AAS_JWT_CERT_CN=\<CERTIFICATE SUBJECT\> AAS JWT

Certificate Subject

-   Environment variable AAS_JWT_INCLUDE_KEYID=\<KEY ID\> AAS include key id in JWT Token

-   Environment variable AAS_JWT_TOKEN_DURATION_MINS=\<DURATION\> JWT Token validation minutes

-   Environment variable BEARER_TOKEN=\<token\> for authenticating with CMS


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
| ENDPOINT_URL         | https://< KBS Hostname >:9443/v1              | KBS Endpoint URL                                   |
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

kbs uninstall \[\--purge\]

Removes the service

#### Version

kbs version

Displays the version of the service

#### Setup \[task\]

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

```

### Directory Layout

The SGX Quote Verification Service installs by default to /opt/sqvs with the following folders.

#### Bin

This folder contains executable scripts.

#### Configuration

This folder /etc/sqvs contains certificates, keys, and configuration files.

#### Logs

This folder contains log files: /var/log/sqvs


# Uninstallation 

This section describes steps used for uninstalling Intel SecL-DC services.


## Certificate Management Service

To uninstall the Certificate Management Service, run the following command:

cms uninstall \--purge

Removes following directories:

1.  /opt/cms

2.  /run/cms

3.  /var/log/cms

4.  /etc/cms


## Authentication and Authorization Service

To uninstall the Authentication and Authorization Service, run the following command:

authservice uninstall \--purge

Removes following directories:

1.  /opt/authservice

2.  /run/authservice

3.  /var/log/authservice

4.  /etc/authservice


## SGX Host Verification Service 

To uninstall the SGX Host Verification Service, run the following command:

shvs uninstall \--purge

Removes following directories:

1.  /opt/shvs

2.  /run/shvs

3.  /var/log/shvs

4.  /etc/shvs


## SGX_Agent 

To uninstall the SGX Agent, run the following command: 

sgx_agent uninstall \--purge

Removes following directories:

1.  /opt/sgx_agent

2.  /run/sgx_agent

3.  /var/log/sgx_agent

4.  /etc/sgx_agent

## Integration Hub 

To uninstall the Integration Hub, run the following command:

ihub uninstall \--purge

Removes the following directories:

1.  /opt/ihub

2.  /run/ihub

3.  /var/log/ihub

4.  /etc/ihub

## SGX Caching Service

To uninstall the SGX Caching Service , run the following command:

scs uninstall \--purge

Removes the following directories:

1.  /opt/scs

2.  /run/scs

3.  /var/log/scs

4.  /etc/scs

## SGX Quote Verification Service 

To uninstall the SGX Quote Verification Service, run the following command:

sqvs uninstall \--purge

Removes the following directories:

1.  /opt/sqvs

2.  /run/sqvs

3.  /var/log/sqvs

4.  /etc/sqvs

## Key Broker Service

kbs uninstall \--purge

To uninstall the Key Broker Service , run the following command:

Removes the following directories:

1.  /opt/kbs

2.  /run/kbs

3.  /var/log/kbs

4.  /etc/kbs


## SKC Library

To uninstall the SKC Library, run the following command:

./opt/skc/devops/scripts/uninstall.sh

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



# Binary Upgrades

***NOTE:***  Before performing any upgrade, Intel strongly recommends backing up the database for the SHVS, SCS, and AAS.  See Postgres documentation for detailed options for backing up databases.  Below is a sample method for backing up an entire database server:

```
Backup to tar file:
pg_dump --dbname <database_name> --username=<database username> -F t > <database_backup_file>.tar
Restore from tar file:
pg_restore --dbname=<database_name> --username=<database username><database_backup_file>.tar	
```

Some upgrades may involve changes to database content, and a backup will ensure that data is not lost in the case of an error during the upgrade process.

## Backward Compatibility

In general Intel SecL services are made to be backward-compatible within a given major release (for example, the 3.6 SHVS should be compatible with the 3.5 SGX Agent) in an upgrade priority order (see below).  Major version upgrades may require coordinated upgrades across all services.

## Upgrade Order

Upgrades should be performed in the following order to prevent misconfiguration or any service unavailability:

1) CMS, AAS

2) SCS, SHVS

3) SQVS, KBS, SGX Agent

Upgrading in this order will make each service unavailable only for the duration of the upgrade for that service.  

## Upgrade Process

### Binary Installations

For services installed directly (not deployed as containers), the upgrade process simply requires executing the new-version installer on the same machine where the old-version is running.  The installer will re-use the same configuration elements detected in the existing version's config file.  No additional answer file is required.

# Container Upgrades

Container upgrades will be supported only on multi node deployments and will be based on recreate strategy from v3.6 to v4.0. All services except KBS can be upgraded just by updating the image name and tag to newer version in respective deployment.yml files.

## Backup and roll back applicable to all services

1.	Take back up of data in NFS mount for all services from NFS server system <NFS-Mount-Path>/isecl
2.	If upgrade is not successful, then update deployment.yml files with older images(v3.6) for 4.0 upgrade path and restore the backed up NFS data at same mount path.
3.	Bring up individual components with ./isecl-bootstrap.sh up <service>.

**Note:** If in case service fails to start or gets crashed, then copy all the backed up data as per folder structure like how was it earlier. Bring up DB instance pointing to backed up data and bring up component deployment by pointing to older version of container image.

## Backward Compatibility

In general Intel SecL services are made to be backward-compatible within a given major release (for example, the  SHVS should be compatible with the 3.5 SGX Agent) in an upgrade priority order (see below). Major version upgrades may require coordinated upgrades across all services.

## Upgrade Order

Upgrades should be performed in the following order to prevent misconfiguration or any service unavailability:

1.	CMS, AAS
2.	SCS, SHVS
3.	SQVS, KBS, SGX Agent
4.	ISECL-CONTROLLER, IHUB, ISECL-SCHEDULER

Upgrading in this order will make each service unavailable only for the duration of the upgrade for that service.

## Upgrade Process

### Container Installations

Assuming all services for any use-case are up and running.

Push all the newer version of container images to registry. All oci images will be in k8s/container-images.

e.g
```
skopeo copy oci-archive:<oci-image-tar-name> docker://<registry-ip/hostname>:<registry-port>/<image-name>:<image-tag> --dest-tls-verify=false
```

#### Individual services upgrade except KBS

1.	Update the image name in existing deployment.yml of respective service.
2.	Redeploy by running the below command, By doing "kubectl apply -f" on deployment.yml with change in image name will terminate service with older image version and bring up service with new image version.

```
cd /k8s/manifests/<service-manifests-folder> &&  kubectl kustomize . | kubectl apply -f -
e.g cd /k8s/manifests/cms && kubectl kustomize . | kubectl apply -f -
```

#### KBS Upgrade

1.	Update the image name with new image name/image tag in existing deployment.yml.
2.	Copy upgrade-job.yml from builds k8s/manifests/kbs/upgrade-job.yml into control plane node k8s/manifests/kbs/
```
scp <build-vm>:<build-dir>/k8s/manifests/kbs/upgrade-job.yml <control-plane-vm>:<existing dir>/k8s/manifests/kbs/upgrade-job.yml
```
3.	Add the following variables in kbs/configMap.yml
```
KMIP_HOSTNAME: <kmip fqdn or ip>
```
Run the command
```
kubectl apply -f kbs/configMap.yml --namespace=isecl
```
4.	(Optional) Add the following variables in kbs/secrets.yml if required KMIP_USERNAME and KMIP_PASSWORD Run the following commands
```
kubectl delete secret -n isecl kbs-secret
kubectl create secret generic kbs-secret --from-file=kbs/secrets.txt --namespace=isecl
```
5.	Update "image-name" and "image-tag" and existing version of deployed kbs in "current deployed version" in k8s/manifests/kbs/upgrade-job.yml
```
      containers:
        - name: kbs
          image: <image-name>:<image-tag>
          imagePullPolicy: Always
          securityContext:
            runAsUser: 1001
            runAsGroup: 1001
          env:
            - name: CONFIG_DIR
              value: "/etc/kbs"
            - name: COMPONENT_VERSION
              value: <current deployed version>
```
6.	Run the upgrade job,
```
cd k8s/manifests
kubectl delete deployment -n isecl kbs-deployment
kubectl apply -f kbs/upgrade-job.yml
```
7.	Check the status of kbs-upgrade job for completion.
```
kubectl get jobs -n isecl
kubectl logs -n isecl kbs-upgrade-<pod id>
```
8.	Update the image name in kbs/deployment.yml to newer version and deploy the latest kbs
```
kubectl apply -f kbs/deployment.yml or cd kbs && kubectl kustomize . | kubectl apply -f -
```

# Appendix 

### SGX Attestation flow
```
To Deploy SampleApp:
  Copy sample_apps.tar, sample_apps.sha2 and sampleapps_untar.sh from binaries directory to a directory in SGX compute node and untar it using './sample_apps_untar.sh'
  Install Intel® SGX SDK for Linux*OS into /opt/intel/sgxsdk using './install_sgxsdk.sh'
  Install SGX dependencies using './deploy_sgx_dependencies.sh'
Note: Make sure to deploy SQVS with includetoken configuration as false. 

To Verify the SampleApp flow:
  Update sample_apps.conf with the following
   - IP address for SQVS services deployed on Enterprise system
   - IP address for SCS services deployed on CSP system
   - ENTERPRISE_CMS_IP should point to the IP of CMS service deployed on Enterprise system
   - Network Port numbers for SCS services deployed on CSP system
   - Network Port numbers for SQVS and CMS services deployed on Enterprise system
   - Set RUN_ATTESTING_APP to yes if user wants to run both apps in same machine
  Run SampleApp using './run_sample_apps.sh'
  Check the output of attestedApp and attestingApp under out/attested_app_console_out.log and out/attesting_app_console_out.log files 

```

### Creating RSA Keys in Key Broker Service

**Steps to run KMIP Server**

Note: Below mentioned steps are provided as script (install_pykmip.sh and pykmip.service) as part of kbs_script folder which will install KMIP Server as daemon. Refer to ‘Install KMIP Server as daemon’ section.

```
1. Install python3 and vim-common
   For RHEL 8.2
   # dnf -y install python3-pip vim-common  
   For Ubuntu 18.04     
   # apt -y install python3-pip vim-common    
   ln -s /usr/bin/python3 /usr/bin/python  > /dev/null 2>&1
   ln -s /usr/bin/pip3 /usr/bin/pip  > /dev/null 2>&1

2. Install pykmip
   # pip3 install pykmip==0.9.1

3. In the /etc/ directory create pykmip and policies folders
   mkdir -p /etc/pykmip/policies

4. Configure pykmip server using server.conf
   Update hostname in the server.conf

5. Copy the following to /etc/pykmip/ from kbs_script folder available under binaries directory
   create_certificates.py, run_server.py, server.conf

6. Create certificates
   > cd /etc/pykmip
   > python3 create_certificates.py <KMIP Host IP/KMIP Host FQDN>

7. Kill running KMIP Server processes and wait for 10 seconds until all the KMIP Server processes are killed. 
   > ps -ef | grep run_server.py | grep -v grep | awk '{print $2}' | xargs kill

8. Run pykmip server using run_server.py script
   > python3 run_server.py &

```

**Install KMIP Server as daemon**

```
1. cd into /root/binaries/kbs_script folder 

2. Configure pykmip server using server.conf
   Update hostname in the server.conf

3. Run the install_pykmip.sh script and KMIP server will be installed as daemon process
   ./install_pykmip.sh

```

**Create RSA key in PyKMIP and generate certificate**

NOTE: This step is required only when PyKMIP script is used as a backend KMIP server.

```
1. Update Host IP in /root/binaries/kbs_script rsa_create.py script
2. In the kbs_script folder, Run rsa_create.py script
    > cd /root/binaries/kbs_script
    > python3 rsa_create.py

This script will generate “Private Key ID” and “Server certificate”, which should be provided in the kbs.conf file for “KMIP_KEY_ID” and “SERVER_CERT”.

```
**Configuration Update to create Keys in KBS**
    
	cd into /root/binaries/kbs_script folder
	
	**To register keys with KBS KMIP**
	
	Update the following variables in kbs.conf:
	
	    KMIP_KEY_ID (Private key ID registered in KMIP server)
	    
	    SERVER_CERT (Server certificate for created private key)
				
		Enterprise system IP address where CMS, AAS and KBS services are deployed
	    
		Port of CMS, AAS and KBS services deployed on enterprise system
	
	    AAS admin and Enterprise admin credentials

NOTE: If KMIP_KEY_ID is not provided then RSA key register will be done with keystring.

Update sgx_enclave_measurement_anyof value in transfer_policy_request.json with enclave measurement value obtained using sgx_sign utility. Refer to "Extracting SGX Enclave values for Key Transfer Policy" section.

**Create RSA Key**

	Execute the command
	
	./run.sh reg

Copy the generated cert file to SGX Compute node where skc_library is deployed. Also make a note of the key id generated.

## Configuration for NGINX testing on RHEL 8.2

**Note:** Below mentioned OpenSSL and NGINX configuration updates are provided as patches (nginx.patch and openssl.patch) as part of skc_library deployment script. Patch can be applied with default nginx and openssl file. In case nginx/openssl contains any external changes then refer manual step.

**Apply Patch**
        Execute the command with nginx version - nginx 1.14.1 (Rhel) and openssl version- Openssl 1.1.1g (Rhel)

        patch -b /etc/nginx/nginx.conf < nginx.patch
        patch -b /etc/ssl/openssl.cnf < openssl.patch

**OpenSSL Configuration**

Update openssl configuration file /etc/pki/tls/openssl.cnf with below changes:

[openssl_def]
engines = engine_section

[engine_section]
pkcs11 = pkcs11_section

[pkcs11_section]
engine_id = pkcs11

dynamic_path =/usr/lib64/engines-1.1/pkcs11.so

MODULE_PATH =/opt/skc/lib/libpkcs11-api.so

init = 0

**Nginx Configuration**

Update nginx configuration file /etc/nginx/nginx.conf with below changes:

ssl_engine pkcs11;

Update the location of certificate with the location where it was copied into the SGX compute node. 

ssl_certificate "add absolute path of crt file"; 

Update the fields(token, object and pin-value) with the values given in keys.txt for the KeyID corresponding to the certificate.

ssl_certificate_key "engine:pkcs11:pkcs11:token=KMS;object=RSAKEY;pin-value=1234";

**SKC Configuration**

 Create keys.txt in /root folder. This provides key preloading functionality in skc_library. 

Any number of keys can be added in keys.txt. Each PKCS11 URL should contain different Key IDs which need to be transferred from KBS along with respective object tag for each key id specified

Token, object and pin-value given in PKCS11 url entry in keys.txt should match with the one in nginx.conf.

The keyID should match the keyID of RSA key created in KBS. File location should match on pkcs11-apimodule.ini; 

	pkcs11:token=KMS;id=164b41ae-be61-4c7c-a027-4a2ab1e5e4c4;object=RSAKEY;type=private;pin-value=1234;
	
	Sample /opt/skc/etc/pkcs11-apimodule.ini file
	
	[core]
	preload_keys=/root/keys.txt
	keyagent_conf=/opt/skc/etc/key-agent.ini
	mode=SGX
	debug=true
	
	[SGX]
	module=/opt/intel/cryptoapitoolkit/lib/libp11sgx.so

## Configuration for NGINX testing for Ubuntu 18.04

**Note:** Below mentioned OpenSSL and NGINX configuration updates are provided as patches (nginx.patch and openssl.patch) as part of skc_library deployment script. Patch can be applied with default nginx and openssl file. In case nginx/openssl contains any external changes then refer manual step.

**Apply Patch**
        Execute the command with nginx version - nginx 1.14.0 (Ubuntu) and openssl version- Openssl 1.1.1 (Ubuntu)

        patch -b /etc/nginx/nginx.conf < nginx_ubuntu.patch
        patch -b /etc/ssl/openssl.cnf < openssl_ubuntu.patch

**OpenSSL**

In the /etc/ssl/openssl.cnf file, look for the below line:
[ new_oids ]

Just before the line [ new_oids ], add the below section:

openssl_conf = openssl_def

[openssl_def]
engines = engine_section
oid_section = new_oids

[engine_section]
pkcs11 = pkcs11_section

[pkcs11_section]
engine_id = pkcs11
dynamic_path =/usr/lib/x86_64-linux-gnu/engines-1.1/pkcs11.so
MODULE_PATH =/opt/skc/lib/libpkcs11-api.so
init = 0

**Nginx**

Update nginx configuration file /etc/nginx/nginx.conf with below changes:

ssl_engine pkcs11;

Update the location of certificate with the loaction where it was copied into the skc_library machine.

ssl_certificate "add absolute path of crt file";

Update the fields(token, object and pin-value) with the values given in keys.txt for the KeyID corresponding to the certificate.

ssl_certificate_key "engine:pkcs11:pkcs11:token=KMS;object=RSAKEY;pin-value=1234";

**SKC Configuration**

 Create keys.txt in /root folder. This provides key preloading functionality in skc_library.

  Any number of keys can be added in keys.txt. Each PKCS11 URL should contain different Key ID which need to be transferred from KBS along with respective object tag for each key id specified

  Sample PKCS11 url is as below

  pkcs11:token=KMS;id=164b41ae-be61-4c7c-a027-4a2ab1e5e4c4;object=RSAKEY;type=private;pin-value=1234;

  Token, object and pin-value given in PKCS11 url entry in keys.txt should match with the one in nginx.conf.

  The keyID should match the keyID of RSA key created in KBS. File location should match with preload_keys directive in pkcs11-apimodule.ini;

  Sample /opt/skc/etc/pkcs11-apimodule.ini file

        [core]
        preload_keys=/root/keys.txt
        keyagent_conf=/opt/skc/etc/key-agent.ini
        mode=SGX
        debug=true
    
       [SGX]
        module=/opt/intel/cryptoapitoolkit/lib/libp11sgx.so

# KBS key-transfer flow validation

On SGX compute node, Execute below commands for KBS key-transfer:


Note: Before initiating key transfer make sure, PYKMIP server is running.

```
    pkill nginx
```

Remove any existing pkcs11 token

```
    rm -rf /opt/intel/cryptoapitoolkit/tokens/*
```

Initiate Key tranfer from KBS

```
    systemctl restart nginx
```

Changing group ownership and permissions of pkcs11 token
```
    chown -R root:intel /opt/intel/cryptoapitoolkit/tokens/
    chmod -R 770 /opt/intel/cryptoapitoolkit/tokens/
```

Establish tls session with the nginx using the key transferred inside the enclave

```
    wget https://localhost:2443 --no-check-certificate
```

# Note on Key Transfer Policy

Key transfer policy is used to enforce a set of policies which need to be compiled with before the secret can be securely provisioned onto a sgx enclave

A typical Key Transfer Policy would look as below
```
        "sgx_enclave_issuer_anyof":["83d719e77deaca1470f6baf62a4d774303c899db69020f9c70ee1dfc08c7ce9e"],
        "sgx_enclave_issuer_product_id_anyof":[0],
        "sgx_enclave_measurement_anyof":["ad46749ed41ebaa2327252041ee746d3791a9f2431830fee0883f7993caf316a"],
        "tls_client_certificate_issuer_cn_anyof":["CMSCA", "CMS TLS Client CA"],
        "client_permissions_allof":["nginx","USA"],
        "sgx_enforce_tcb_up_to_date":false
```

**sgx_enclave_issuer_anyof** establishes the signing identity provided by an authority who has signed the sgx enclave. in other words the owner of the enclave

**sgx_enclave_measurement_anyof** represents the cryptographic hash of the enclave log (enclave code, data)

**sgx_enforce_tcb_up_to_date** - If set to true, Key Broker service will provision the key only of the platform generating the quote conforms to the latest Trusted Computing Base

**client_permissions_allof** - Special permission embedded into the skc_library client TLS certificate which can enforce additional restrictons on who can get access to the key,
    In above example: the key is provisioned only to the nginx workload and platform which is tagged with value for ex: USA


# Note on SKC Library Deployment

SKC Library Deployment (Binary as well as container) needs to performed with root privilege

For binary deployment of SKC client Library, only one instance of Workload can use SKC Client Library. The config information for SKC client library is bound to the workload.
In future, Multiple workloads might be supported
For container deployment, since configmaps are used, each container instance of workload gets its own private SKC Client Library config information

The SKC Client Library TLS client certificate private key is stored in the configuration directories and can be read only with elevated root privileges
keys.txt (set of PKCS11 URIs for the keys to be securely provisioned into an SGX enclave) can only be modified with elevated privileges

## Extracting SGX Enclave values for Key Transfer Policy

Values that are specific to the enclave such as sgx_enclave_issuer_anyof, sgx_enclave_measurement_anyof and sgx_enclave_issuer_product_id_anyof can be retrived using `sgx_sign` utility that is available as part of Intel SGX SDK.

Run `sgx_sign` utility on the signed enclave (This command should be run on the build system).
```
    /opt/intel/sgxsdk/bin/x64/sgx_sign dump -enclave <path to the signed enclave> -dumpfile info.txt
```

- For `sgx_enclave_issuer_anyof`, in info.txt, search for "mrsigner->value" . E.g mrsigner->value :
  ```
  mrsigner->value: "0x83 0xd7 0x19 0xe7 0x7d 0xea 0xca 0x14 0x70 0xf6 0xba 0xf6 0x2a 0x4d 0x77 0x43 0x03 0xc8 0x99 0xdb 0x69 0x02 0x0f 0x9c 0x70 0xee 0x1d 0xfc 0x08 0xc7 0xce 0x9e"
  ```
  Remove the whitespace and 0x characters from the above string and add it to the policy file. E.g :
  ```
  "sgx_enclave_issuer_anyof":["83d719e77deaca1470f6baf62a4d774303c899db69020f9c70ee1dfc08c7ce9e"]
  ```
- For `sgx_enclave_measurement_anyof`, in info.txt, search for metadata->enclave_css.body.enclave_hash.m . E.g metadata->enclave_css.body.enclave_hash.m :
  ```
  metadata->enclave_css.body.enclave_hash.m:
  0xad 0x46 0x74 0x9e 0xd4 0x1e 0xba 0xa2 0x32 0x72 0x52 0x04 0x1e 0xe7 0x46 0xd3
  0x79 0x1a 0x9f 0x24 0x31 0x83 0x0f 0xee 0x08 0x83 0xf7 0x99 0x3c 0xaf 0x31 0x6a
  ```
  Remove the whitespace and 0x characters from the above string and add it to the policy file. E.g :
  ```
  "sgx_enclave_measurement_anyof":["ad46749ed41ebaa2327252041ee746d3791a9f2431830fee0883f7993caf316a"]
  ```
Please note that the SGX Enclave measurement value will depend on the toolchain used to build and link the SGX enclave. Hence the SGX Enclave measurement value would differ across OS flavours.
For more details please refer https://github.com/intel/linux-sgx/tree/master/linux/reproducibility
