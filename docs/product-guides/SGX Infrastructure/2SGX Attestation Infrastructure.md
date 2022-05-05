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

## TEE Caching Service

The TEE Caching Service (TCS) allows to retrieve the PCK certificates of the data center server platforms from Intel SGX Provisioning Certification Service (PCS). TCS retrieves and caches SGX platform collaterals. The collateral consists of the security patches (TCBInfo) that have been issued for Intel platform models. Finally, TCS retrieves and caches the Certificate Revocation Lists (CRLs).

Since the Caching Service stores the TCBInfo of all the platform models in the datacenter, the Quote Verification Service (QVS) uses it to determine the TCB status of the platforms in the data center.

The SKC Client retrieves its PCK certificate from the TEE Caching Service when it generates an SGX quote.

TCS can be deployed in both Cloud Service Provider (CSP) and tenant environments. In the CSP environment, TCS is used to fetch PCK certificates for compute nodes in the data center. In the tenant environment, it's used to cache SGX collateral information used in verifying SGX quotes.

## Feature Discovery Service

If Feature Discovery Service API URL is specified in FDA env file, then FDA will push the platform enablement info and TCB status to FDS at regular intervals, else, FDA pushes the platform enablement info and TCB status to FDS periodically. The SGX enablement information consists of SGX discovery information (SGX supported, SGX enabled, FLC enabled and EPC memory size).

## Feature Discovery Agent

The Feature Discovery Agent resides on physical servers and pushes SGX platform specific values to TEE Caching Service (TCS).

If Feature Discovery Service (FDS) URL is specified in Feature Doscovery Agent env file, SGX Agent would fetch the TCB status from TCS and updates FDS with platform enablement info and TCB status periodically.

## Integration Hub

The Integration Hub (IHUB) allows to support SGX in Kubernetes. IHUB pulls the list of hosts details from Kubernetes and then using the host information it pulls the SGX Data from Feature Discovery Service and pushes it to Kubernetes. IHUB performs these steps on a regular basis so that the most recent SGX information about nodes is reflected in Kubernetes. This integration allows Kubernetes to schedule containers that need to run SGX workloads on compute nodes that support SGX. The SGX data that IHUB pushes to Kubernetes consists of SGX enabled/disabled, SGX supported/not supported, FLC enabled/not enabled, EPC memory size, TCB status up to date/not up to date and platform-data expiry time.

## Quote Verification Service

The Quote Verification Service (QVS) is typically deployed in the tenant environment, not the Cloud Service Provider (CSP) environment. SQVS performs the verification of SGX quotes on behalf of KBS. QVS determines if the SGX quote signature is valid. It also determines if the SGX quote has been generated on a platform that is up to date on security patches (TCB). For the latter, QVS uses the TEE Caching Service, which caches the SGX collateral information about Intel platform models. QVS also parses the SGX quote and extracts the entities and returns them to KBS, which can then make additional policy decisions based on the values of the theses entities.

## Architecture Overview

As indicated in the Features section, SKC provides 3 features essentially:

-   SGX Attestation Support: this is the feature that CSPs provide to tenants who need to run SGX workloads that require attestation.
-   SGX Support in Orchestrators: this feature allows to discover SGX support in physical servers and related information:
    -   SGX supported.

    -   SGX enabled.

    -   Size of RAM reserved for SGX. It's called Enclave Page Cache (EPC).

    -   Flexible Launch Control enabled.
-   Key Protection: this is the feature used by tenants using a CSP to run workloads with key protection requirements.

The high-level architectures of these features are presented in the next sub-sections.

## SGX Attestation Support and SGX Support in Orchestrators

The diagram below shows the infrastructure that CSPs need to deploy to support SGX attestation and optionally, integration with orchestrators (Kubernetes).

THE Feature Discovery Agent pushes platform information to TEE Caching Service (SCS), which uses it to get the PCK Certificate and other SGX collateral from the Intel SGX Provisioning Certification Service (PCS) and caches them locally. When a workload on the platform needs to generate an SGX Quote, it retrieves the PCK Certificate of the platform from TCS.

If SFeature Discovery Service (FDS) URL is configured, the Feature Discovery Agent fetches the TCB Status from TCS and updates FDS with SGX platform enablement information and TCB status periodically. The platform information is made available to Kubernetes  via the SGX Hub (IHUB), which pulls it from FDS.

The Quote Verification Service (QVS) allows attesting applications to verify SGX quotes and extract the SGX quote attributes to verify compliance with a user-defined SGX enclave policy. SQVS uses the SGX Caching Service to retrieve SGX collateral needed to verify SGX quotes from the Intel SGX Provisioning Certification Service (PCS). SQVS typically runs in the attesting application owner network environment. Typically, a separate instance of the SGX Caching Service is setup in the attesting application owner network environment.

![](images/image-20200727163158892.png)

The Feature Discovery Agent and the SGX services integrate with the Authentication and Authorization Service (AAS) and the Certificate Management Service (CMS). AAS and CMS are not represented on the diagram for clarity.
