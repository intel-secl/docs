# Introduction

[Intel® SecL-DC libraries](https://github.com/intel-secl/docs/-/blob/v4.1/develop/product-guides/Secure%20Key%20Caching.md#sgx-attestation-infrastructure-and-skc-components) support SGX attestation as a base use case. For a detailed description of SGX Attestation, please refer to [SGX ECDSA Attestation](https://github.com/intel-secl/docs/-/blob/v4.1/product-guides/SGX%20Infrastructure.md#sgx-ecdsa-attestation)

Intel® SecL-DC libraries also support following usecases which build upon the SGX Attestation usecase
* Secure Key Caching
* SGX Orchestration Support for Kubernetes and Openstack


##  Glossary

| CMS        | Certificate Management Service           | Responsible for Providing TLS Certificates for other components                                                                  |
|------------|------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------|
| AAS        | Authentication and Authorization Service | Authenticates Service User and Authorizes User against set of predefined roles                                                   |
| Agent      | SGX Agent                                | Performs SGX Feature Discovery and Extracts SGX Platform Data from a SGX Compute Node                                            |
| IHUB       | integration Hub                          | Service to populate SGX Platform Discovery and Enablement Data to Cloud Orchestrator Services like Kubernetes and Openstack      |
| KBS        | Key Broker Service                       | Broker Service which integrates with backend key management solution for accessing secrets over KMIP Interface                   |
| SCS        | SGX Caching Service                      | Responsible for Providing PCK Certifcates and other SGX Platform Collaterals                                                     |
| SQVS       | SGX Quote Verification Service           | Responsible fo Verifying SGX ECDSA Quote                                                                                         |
| SKC Client | SKC Library                              | A Client Library which connects to Key Broker Service to get the Secrets over secure channel and stores secret in a SGX Enclave  |
| SHVS       | SGX Host Verification Service            | Aggregates SGX Platform Discovery and Enablement Data for all the SGX Compute Nodes in a cluster                                 |


## Intel® SecL-DC libraries Services Deployment Matrix for All Supported Usecases
|                       Usecases            | CMS | AAS | SGX Agent | SCS | SQVS | SKC Library | KBS | SHVS | IHUB |
|-----------------------------------|-----|-----|-----------|-----|------|-------------|-----|------|------|
| SGX Attestation                   | ✔   | ✔   | ✔         | ✔   | ✔    |             |     |      |      |
| Secure Key Caching                | ✔   | ✔   | ✔         | ✔   | ✔    | ✔           | ✔   |      |      |
| Orchestration Support             | ✔   | ✔   | ✔         | ✔   |      |             |     | ✔    | ✔    |
