# Introduction

## Overview

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