# Intel® Security Libraries for Data Center (Intel® SecL-DC)

## Table of Contents

- [Intel® Security Libraries for Data Center (Intel® SecL-DC)](#intel-security-libraries-for-data-center-intel-secl-dc)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Architecture](#architecture)
  - [Use Cases](#use-cases)
    - [Foundational Security & Launch Time Protection](#foundational-security--launch-time-protection)
    - [SGX Attestation Infrastructure](#sgx-attestation-infrastructure)
    - [Secure Key Caching](#secure-key-caching)
  - [License](#license)
  - [Contributing](#contributing)
  - [Legalities](#legalities)

  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Architecture](#architecture)
  - [Use Cases](#use-cases)

    - [Foundational Security & Launch Time Protection](#foundational-security--launch-time-protection)
    - [SGX Attestation Infrastructure](#sgx-attestation-infrastructure)
    - [Secure Key Caching](#secure-key-caching)

  - [License](#license)
  - [Contributing](#contributing)
  - [Legalities](#legalities)

  - [Table of Contents](#table-of-contents)

  - [Overview](#overview)
  - [Architecture](#architecture)
  - [Use Cases](#use-cases)

    - [Foundational Security & Launch Time Protection](#foundational-security--launch-time-protection)
    - [SGX Attestation Infrastructure](#sgx-attestation-infrastructure)
    - [Secure Key Caching](#secure-key-caching)

  - [License](#license)

  - [Contributing](#contributing)
  - [Legalities](#legalities)

    - [Foundational Security & Launch Time Protection](#foundational-security--launch-time-protection)
    - [SGX Attestation Infrastructure](#sgx-attestation-infrastructure)
    - [Secure Key Caching](#secure-key-caching)

  - [License](#license)

  - [Contributing](#contributing)
  - [Known Issues](#known-issues)
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

[Foundational and Workload Security Product Guide](https://github.com/intel-secl/docs/blob/v4.1/develop/product-guides/Foundational%20%26%20Workload%20Security.md)

[Foundational & Workload Security Quick Start Guide](https://github.com/intel-secl/docs/blob/v4.1/develop/quick-start-guides/Foundational%20%26%20Workload%20Security.md)

[Foundational & Workload Security Swagger Documents](https://github.com/intel-secl/docs/tree/v4.1/develop/swagger-docs/foundational-and-workload-security)

### SGX Attestation Infrastructure

The SGX Attestation infrastructure provides an end to end support for registering SGX hosts and provisioning them with SGX material (PCK certificates) and SGX collateral (security patches information - TCB Information - and Certificate Revocation Lists - CRLs).

The SGX Attestation infrastructure also provides support for generating SGX quotes for SGX enclaves hosted by workloads and verifying them by a remote attesting application. The remote attesting application can also use the SGX Attestation infrastructure to enforce enclave policies (like requiring a specific enclave signer).

Optionally, the SGX Attestation Infrastructure allows to integrate with Cloud Orchestrators like Openstack and Kubernetes.

The SGX Attestation infrastructure does not make any assumption on the user SGX workload and enclave policy.

[SGX Attestation Infrastructure and Secure Key Caching Product Guide](https://github.com/intel-secl/docs/blob/v4.1/develop/product-guides/SGX%20Infrastructure.md)

[SGX Attestation Infrastructure and Secure Key Caching Quick Start Guide](https://github.com/intel-secl/docs/tree/v4.1/develop/quick-start-guides)

[SGX Attestation Infrastructure and Secure Key Caching Swagger Documents](https://github.com/intel-secl/docs/tree/v4.1/develop/swagger-docs/sgx-infrastructure)

As a demonstration of SGX Attestation infrastructure artifacts, [a sample SGX attestation app is available here.](https://github.com/intel-secl/utils/tree/v4.1/develop/tools/sample-sgx-attestation)

### Secure Key Caching

Secure Key Caching (SKC) leverages the SGX Attestation Infrastructure to support the Secure Key Caching (SKC) use case.

SKC provides key protection at rest and in-use using Intel Software Guard Extensions (SGX). SGX implements the Trusted Execution Environment (TEE) paradigm.

Using the SKC Client -- a set of libraries -- applications can retrieve keys from the Intel SecL-DC Key Broker Service (KBS) and load them to an SGX-protected memory (called SGX enclave) in the application memory space. KBS performs the SGX enclave attestation to ensure that the application will store the keys in a genuine SGX enclave. The attestation involves the KBS verification of a signed SGX quote generated by the SKC Client. The SGX quote contains the hash of the public key of an enclave generated RSA key pair.

Application keys are wrapped with a Symmetric Wrapping Key (SWK) by KBS prior to transferring to the SGX enclave. The SWK is generated by KBS and wrapped with the enclave RSA public key, which ensures that the SWK is only known to KBS and the enclave . Consequently, application keys are protected from infrastructure admins, malicious applications and compromised HW/BIOS/OS/VMM. SKC does not require the refactoring of the application because it supports a standard PKCS#11 interface.

[SGX Attestation Infrastructure and Secure Key Caching Product Guide](https://github.com/intel-secl/docs/blob/v4.1/develop/product-guides/SGX%20Infrastructure.md)

[SGX Attestation Infrastructure and Secure Key Caching Quick Start Guide](https://github.com/intel-secl/docs/tree/v4.1/develop/quick-start-guides/)

[SGX Attestation Infrastructure and Secure Key Caching Swagger Documents](https://github.com/intel-secl/docs/tree/v4.1/develop/swagger-docs/sgx-infrastructure)

## License

[BSD 3-Clause License](https://opensource.org/licenses/BSD-3-Clause)

## Contributing

<https://github.com/intel-secl/intel-secl/>

## Known Issues

Container build for SGX Attestation and Secure Key caching use cases are not supported in Ubuntu.

## Legalities
