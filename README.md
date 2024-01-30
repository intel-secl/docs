# Intel® Security Libraries for Data Center (Intel® SecL-DC)

## Table of Contents

- [Intel® Security Libraries for Data Center (Intel® SecL-DC)](#intel-security-libraries-for-data-center-intel-secl-dc)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Architecture](#architecture)
  - [Use Cases](#use-cases)
    - [Foundational Security & Launch Time Protection](#foundational-security--launch-time-protection)
    - [SGX Infrastructure and Orchestration](#sgx-infrastructure-and-orchestration)
  - [License](#license)
  - [Contributing](#contributing)
  - [Legalities](#legalities)

## Overview

Intel® Security Libraries for Data Center (Intel® SecL-DC) enables security use cases for data center using Intel® hardware security technologies.

Hardware-based cloud security solutions provide a higher level of protection as compared to software-only security measures. There are many Intel platform security technologies, which can be used to secure customers' data. Customers have found adopting and deploying these technologies at a broad scale challenging, due to the lack of solution integration and deployment tools. Intel® Security Libraries for Data Centers (Intel® SecL - DC) was built to aid our customers in adopting and deploying Intel Security features, rooted in silicon, at scale.

Intel® SecL-DC is an open-source remote attestation implementation comprising a set of building blocks that utilize Intel Security features to discover, attest, and enable critical foundation security and confidential computing use-cases. It applies the remote attestation fundamentals and standard specifications to maintain a platform data collection service, and an efficient verification engine to perform comprehensive trust evaluations. These trust evaluations can be used to govern different trust and security policies applied to any given workload.

For more details please visit : <https://01.org/intel-secl>

## Architecture

The below diagram depicts the high level architecture of the Intel®SecL-DC,

[![isecl-arch](https://github.com/intel-secl/intel-secl/raw/v4.0.0/docs/diagrams/isecl-arch.png)](https://github.com/intel-secl/intel-secl/raw/v4.0.0/docs/diagrams/isecl-arch.png)

## Use Cases

### Foundational Security & Launch Time Protection

Foundational and Workload Security refers to a collection of software security solutions provided by Intel SecL-DC that leverage Intel silicon to provide boot-time integrity attestation of platform components. Starting with a Hardware Root of Trust, a chain of measurements of system components that includes the system BIOS/UEFI and OS kernel is extended to a Trusted Platform Module for remote attestation against expected measurements. Use cases include auditing the integrity of Cloud platforms, Asset or Geolocation Tagging, Platform Integrity-aware Cloud orchestration, and VM and container encryption. This acts as a firm, hardware-rooted foundation upon which to build a Cloud platform with auditable integrity verification.

[Foundational and Workload Security Product Guide](https://intel-secl.github.io/docs/5.1/)

[Foundational & Workload Security Swagger Documents](https://github.com/intel-secl/docs/tree/v5.1/develop/swagger-docs/foundational-and-workload-security)

### SGX Infrastructure and Orchestration

The SGX Attestation infrastructure and Secure Key Caching (SKC) are part of the Intel Security Libraries for datacenter (ISecL-DC). The SGX Attestation infrastructure provides an end to end support for registering SGX hosts and provisioning them with SGX material (PCK certificates) and SGX collateral (security patches information - TCB Information - and Certificate Revocation Lists - CRLs).

The SGX Attestation infrastructure also provides support for generating SGX quotes for SGX enclaves hosted by workloads and verifying them by a remote attesting application. The remote attesting application can also use the SGX Attestation infrastructure to enforce enclave policies (like requiring a specific enclave signer). Optionally, the SGX Attestation Infrastructure allows to integrate with Cloud Orchestrators like Openstack and Kubernetes.

SKC leverages the SGX Attestation Infrastructure to support the Secure Key Caching (SKC) use case.SKC provides the key protection at rest and in-use use case using the Intel Software Guard Extensions technology (SGX). SGX implements the Trusted Execution Environment (TEE) paradigm.

Using the SKC Client -- a set of libraries -- applications can retrieve keys from the ISecL-DC Key Broker Service (KBS) and load them to an SGX-protected memory (called SGX enclave) in the application memory space. KBS performs the SGX enclave attestation to ensure that the application will store the keys in a genuine SGX enclave. Application keys are wrapped with an enclave public key by KBS prior to transferring to the application enclave. Consequently, application keys are protected from infrastructure admins, malicious applications and compromised HW/BIOS/OS/VMM. SKC does not require the refactoring of the application because it supports a standard PKCS\#11 interface.

[SGX Infrastructure and Orchestration Product Guide](https://intel-secl.github.io/docs/5.1/)

[SGX Infrastructure and Orchestration Swagger Documents](https://github.com/intel-secl/docs/tree/v5.1/develop/swagger-docs/sgx-infrastructure-and-orchestration)

## License

[BSD 3-Clause License](https://opensource.org/licenses/BSD-3-Clause)

## Contributing 

<https://github.com/intel-secl/intel-secl/>


## Legalities
