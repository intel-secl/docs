# SGX Attestation Infrastructure

The components documented in this section are used by the SGX Attestation Infrastructure and therefore by SKC, which leverages the SGX Attestation Infrastructure. Components that are exclusively used by SKC have (SKC Only) in the corresponding sub-section title.

## Definitions, Acronyms, and Abbreviation

* SGX -- Software Guard Extension

* TEE -- Trusted Execution Environment

* CSP -- Cloud Service Provider

* PCS -- Provisioning Certification Service

* CRLs -- Certificate Revocation Lists

* AAS -- Authentication and Authorization Service

* CRDs -- Custom Resource Definitions

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

## SGX Quote Verification Service

The SGX Quote Verification Service (SQVS) is typically deployed in the tenant environment, not the Cloud Service Provider (CSP) environment. SQVS performs the verification of SGX quotes on behalf of KBS. SQVS determines if the SGX quote signature is valid. It also determines if the SGX quote has been generated on a platform that is up to date on security patches (TCB). For the latter, SQVS uses the SGX Caching Service, which caches the SGX collateral information about Intel platform models. SQVS also parses the SGX quote and extracts the entities and returns them to KBS, which can then make additional policy decisions based on the values of the theses entities.

## Architecture Overview

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

![](images/image-20200727163158892.png)

The SGX Agent and the SGX services integrate with the Authentication and Authorization Service (AAS) and the Certificate Management Service (CMS). AAS and CMS are not represented on the diagram for clarity.
