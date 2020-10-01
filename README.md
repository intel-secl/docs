# Intel® Security Libraries for Data Center (Intel® SecL-DC)

[[_TOC_]]

## Overview

Intel® Security Libraries for Data Center (Intel® SecL-DC) enables security use cases for data center using Intel® hardware security technologies.

Hardware-based cloud security solutions provide a higher level of protection as compared to software-only security measures. There are many Intel platform security technologies, which can be used to secure customers' data. Customers have found adopting and deploying these technologies at a broad scale challenging, due to the lack of solution integration and deployment tools. Intel® Security Libraries for Data Centers (Intel® SecL - DC) was built to aid our customers in adopting and deploying Intel Security features, rooted in silicon, at scale.

Intel® SecL-DC is an open-source remote attestation implementation comprising a set of building blocks that utilize Intel Security features to discover, attest, and enable critical foundation security and confidential computing use-cases. It applies the remote attestation fundamentals and standard specifications to maintain a platform data collection service, and an efficient verification engine to perform comprehensive trust evaluations. These trust evaluations can be used to govern different trust and security policies applied to any given workload.

For more details please visit : https://01.org/intel-secl

## Architecture

The below diagram depicts the high level architecture of the Intel®SecL-DC,

[![isecl-arch](https://github.com/intel-secl/intel-secl/raw/master/docs/diagrams/isecl-arch.png)](https://github.com/intel-secl/intel-secl/blob/master/docs/diagrams/isecl-arch.png)



## Use Cases

### Foundational Security & Launch Time Protection

Foundational Security

Workload Security 

Foundational and Workload Security Product Guide https://gitlab.devtools.intel.com/sst/isecl/intel-secl-collaterals/-/blob/collaterals/Product%20Guide%20-%20Intel%C2%AE%20Security%20Libraries%20-%20Datacenter%20Foundational%20Security.md

Foundational Security Quick Start Guide https://gitlab.devtools.intel.com/sst/isecl/intel-secl-collaterals/-/blob/collaterals/Quick%20Start%20Guide%20-%20Intel%C2%AE%20Security%20Libraries%20-%20Datacenter.md

Workload Security Quick Start Guide https://gitlab.devtools.intel.com/sst/isecl/intel-secl-collaterals/-/blob/collaterals/Quick%20Start%20Guide%20-%20Intel%C2%AE%20Security%20Libraries%20-%20Datacenter.md

Foundational Security Swagger Documents

Workload Security Swagger Document

### Secure Key Caching

Secure Key Caching (SKC) is part of the Intel Security Libraries for datacenter (ISecL-DC). Intel Security Libraries for Datacenter is a collection of software applications and development libraries intended to help turn Intel platform security features into real-world security use cases. SKC provides the key protection at rest and in-use use case using the Intel Software Guard Extensions technology (SGX). SGX implements the Trusted Execution Environment (TEE) paradigm.

Using the SKC Client -- a set of libraries -- applications can retrieve keys from the ISecL-DC Key Broker Service (KBS) and load them to an SGX-protected memory (called SGX enclave) in the application memory space. KBS performs the SGX enclave attestation to ensure that the application will store the keys in a genuine SGX enclave. Application keys are wrapped with an enclave public key by KBS prior to transferring to the application enclave. Consequently, application keys are protected from infrastructure admins, malicious applications and compromised HW/BIOS/OS/VMM. SKC does not require the refactoring of the application because it supports a standard PKCS#11 interface.

The SKC use case requires the provisioning of host servers to respond to SGX attestation requests. The SKC solution provides a framework to support SGX attestation.

Link to Product Guide 

https://gitlab.devtools.intel.com/sst/isecl/docs/-/blob/v3.1/develop/product-guides/Product%20Guide%20-%20Intel%C2%AE%20Security%20Libraries%20-%20Secure%20Key%20Caching.md

Link to Quick Start Guide

https://gitlab.devtools.intel.com/sst/isecl/docs/-/blob/v3.1/develop/quick-start-guides/Quick%20Start%20Guide%20-%20Intel%C2%AE%20Security%20Libraries%20-%20Secure%20Key%20Caching.md

## License 

## Support 

## Reporting Issues

## Contributing 

## Legalities
