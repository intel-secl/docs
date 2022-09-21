# Intel® Security Libraries for Data Center (Intel® SecL-DC)

## Table of Contents

- [Intel® Security Libraries for Data Center (Intel® SecL-DC)](#intel-security-libraries-for-data-center-intel-secl-dc)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Architecture](#architecture)
  - [Use Cases](#use-cases)
    - [Foundational Security & Launch Time Protection](#foundational-security--launch-time-protection)
  - [License](#license)
  - [Contributing](#contributing)
  - [Legalities](#legalities)

  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Architecture](#architecture)
  - [Use Cases](#use-cases)

    - [Foundational Security & Launch Time Protection](#foundational-security--launch-time-protection)

  - [License](#license)
  - [Contributing](#contributing)
  - [Legalities](#legalities)

  - [Table of Contents](#table-of-contents)

  - [Overview](#overview)
  - [Architecture](#architecture)
  - [Use Cases](#use-cases)

    - [Foundational Security & Launch Time Protection](#foundational-security--launch-time-protection)
  
  - [License](#license)

  - [Contributing](#contributing)
  - [Legalities](#legalities)

    - [Foundational Security & Launch Time Protection](#foundational-security--launch-time-protection)
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

[Foundational and Workload Security Product Guide](https://intel-secl.github.io/docs/5.0/)

[Foundational & Workload Security Swagger Documents](https://github.com/intel-secl/docs/tree/v5.0/develop/swagger-docs/foundational-and-workload-security)


## License

[BSD 3-Clause License](https://opensource.org/licenses/BSD-3-Clause)

## Contributing

<https://github.com/intel-secl/intel-secl/>


## Legalities
