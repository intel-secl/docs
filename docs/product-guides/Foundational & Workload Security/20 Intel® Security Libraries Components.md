# Intel® Security Libraries Components


* Certificate Management Service

Starting with Intel® SecL-DC 1.6, most non-TPM-related certificates used by Intel® SecL-DC applications will be issued by the new Certificate Management Service. This includes acting as a root CA and issuing TLS certificates for all of the various web services.

* Authentication and Authorization Service

Starting with Intel® SecL-DC 1.6, authentication and authorization for all Intel® SecL applications will be centrally managed by the new Authentication and Authorization Service (AAS). Previously, each application would manage its own users and permissions independently; this change allows authentication and authorization management to be centralized.

* Verification Service

The Verification Service component of Intel® Security Libraries performs the core Platform Integrity and Data Sovereignty functionality by acting as a remote attestation authority.

Platform security technologies like Intel® TXT, Intel® BootGuard, and UEFI SecureBoot extend measurements of platform components (such as the system BIOS/UEFI, OS kernel, etc) to a Trusted Platform module as the server boots. Known-good measurements for each of these components can be directly imported from a sample server. These expected measurements can then be compared against actual measurements from registered servers, allowing the Verification Service to attest to the "trustiness" of the platform, meaning whether the platform booted into a "known-good" state.

* Workload Service

The Workload Service acts as a management service for handling Workload Flavors (Flavors used for Virtual Machines and Containers). In the Intel® SecL-DC 1.6 release, the Workload Service uses Flavors to map decryption key IDs to image IDs. When a launch request for an encrypted workload image is intercepted by the Workload Agent, the Workload Service will handle mapping the image ID to the appropriate key ID and key request URL, and will initiate the key transfer request to the Key Broker.

* Trust Agent

The Trust Agent resides on physical servers and enables both remote attestation and the extended chain of trust capabilities. The Agent maintains ownership of the server's Trusted Platform Module, allowing secure attestation quotes to be sent to the Verification Service. Incorporating the Intel® SecL HostInfo and TpmProvider libraries, the Trust Agent serves to report on platform security capabilities and platform integrity measurements.

The Trust Agent is supported for Windows* Server 2016 Datacenter and Red Hat Enterprise Linux* (RHEL) 8.1 and later.

* Workload Agent

The Workload Agent is the component responsible for handling all of the functions needed for Workload Confidentiality for virtual machines and containers on a physical server. The Workload Agent uses libvirt hooks to identify VM lifecycle events (VM start, stop, hibernate, etc), and intercepts those events to perform needed functions like requesting decryption keys, creation and deletion of encrypted LUKS volumes, using the TPM to unseal decryption keys, etc. The WLA also performs analogous functionality for containers.

* Integration Hub

The Integration Hub acts as a middle-man between the Verification Service and one or more scheduler services (such as OpenStack* Nova), and "pushes" attestation information retrieved from the Verification Service to one or more scheduler services according to an assignment of hosts to specific tenants. In this way, Tenant A can receive attestation information for hosts that belong to Tenant A, but receive no information about hosts belonging to Tenant B.

The Integration Hub serves to disassociate the process of retrieving attestations from actual scheduler queries, so that scheduler services can adhere to best practices and retain better performance at scale. The Integration Hub will regularly query the Intel® SecL Verification Service for SAML attestations for each host. The Integration Hub maintains only the most recent currently valid attestation for each host, and will refresh attestations when they would expire. The Integration Hub will verify the signature of the SAML attestation for each host assigned to a tenant, then parse the attestation status and asset tag information, and then will securely push the parsed key/value pairs to the plugin endpoints enabled.

The Integration Hub features a plugin design for adding new scheduler endpoint types. Currently the Integration Hub supports OpenStack Nova and Kubernetes endpoint plugins. Other integration plugins may be added.

* Workload Policy Manager

The Workload Policy Manager is a Linux command line utility used by an image owner to encrypt container images and to create an Image Flavor used to provide the encryption key transfer URL during launch requests. The WPM utility will use an existing or request a new key from the Key Broker Service, use that key to encrypt the image, and output the Image Flavor in JSON format. The encrypted image can then be uploaded to the image store of choice (like OpenStack Glance), and the Image Flavor can be uploaded to the Workload Service. The ID of the image on the image storage system is then mapped to the Image Flavor in the WLS; when the image is used to launch a new instance, the WLS will find the Image Flavor associated with that image ID, and use the Image Flavor to determine the key transfer URL.

* Key Broker Service

The Key Broker Service is effectively a policy compliance engine. Its job is to manage key transfer requests, releasing keys only to servers that meet policy requirements. The Key Broker registers one or more SAML signing certificates from any Verification Services that it will trust. When a key transfer request is received, the request includes a trust attestation report signed by the Verification Service. If the signature matches a registered SAML key, the Broker will then look at the actual report to ensure the server requesting the key matches the image policy (currently only overall system trust is supported as a policy requirement). If the report indicates the policy requirements are met, the image decryption key is wrapped using a public key unique to the TPM of the host that was attested in the report, such that only the host that was attested can unseal the decryption key and gain access to the image.

