# Intel® Security Libraries Components

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

The Workload Policy Manager is a Linux command line utility used by an image owner to encrypt VM (qcow2) or container images, and to create an Image Flavor used to provide the encryption key transfer URL during launch requests. The WPM utility will use an existing or request a new key from the Key Broker Service, use that key to encrypt the image, and output the Image Flavor in JSON format. The encrypted image can then be uploaded to the image store of choice (like OpenStack Glance), and the Image Flavor can be uploaded to the Workload Service. The ID of the image on the image storage system is then mapped to the Image Flavor in the WLS; when the image is used to launch a new instance, the WLS will find the Image Flavor associated with that image ID, and use the Image Flavor to determine the key transfer URL.

* Key Broker Service

The Key Broker Service is effectively a policy compliance engine. Its job is to manage key transfer requests, releasing keys only to servers that meet policy requirements. The Key Broker registers one or more SAML signing certificates from any Verification Services that it will trust. When a key transfer request is received, the request includes a trust attestation report signed by the Verification Service. If the signature matches a registered SAML key, the Broker will then look at the actual report to ensure the server requesting the key matches the image policy (currently only overall system trust is supported as a policy requirement). If the report indicates the policy requirements are met, the image decryption key is wrapped using a public key unique to the TPM of the host that was attested in the report, such that only the host that was attested can unseal the decryption key and gain access to the image.

## Intel® Security Libraries Binary Installation


Intel® SecL services can be deployed as direct binary installations (on bare metal or in VMs), or can be deployed as containers.  This section details the binary-based installation of Intel SecL services; the next major section details container-based deployments.

It is recommended to deploy all control-plane services (CMS, AAS, HVS, WLS) as either containers or binaries, and not a mix of the two.

The Trust Agent/Workload Agent, KBS, and WPM can be installed as binaries or deployed as containers regardless of the installation method used for the control plane.

## Building Binary Installers

Intel® Security Libraries is distributed as open source code, and must be compiled into installation binaries before installation.

Instructions and sample scripts for building the Intel® SecL-DC components can be found here:

https://github.com/intel-secl/build-manifest

After the components have been built, the installation binaries can be found in the directories created by the build scripts.

```shell
<servicename>/out/<servicename>.bin
```

In addition, the build script will produce some sample database creation scripts that can be used during installation to configure database requirements (instructions are given in the installation sections):

create_db: `authservice/out/create_db.sh`

install_pgdb: `authservice/out/install_pgdb.sh`

In addition, sample Ansible roles to automatically build and deploy a testbed environment are provided:

https://github.com/intel-secl/utils/tree/v4.0.2/develop/tools/ansible-role

Also provided are sample API calls organized by workflows for Postman:

https://github.com/intel-secl/utils/tree/v4.0.2/develop/tools/api-collections

## Hardware Considerations

Intel® SecL-DC supports and uses a variety of Intel security features, but there are some key requirements to consider before beginning an installation. Most important among these is the Root of Trust configuration. This involves deciding what combination of TXT, Boot Guard, tboot, and UEFI Secure Boot to enable on platforms that will be attested using Intel® SecL.

Key points:

\-   At least one "Static Root of Trust" mechanism must be used (TXT and/or BtG)

\-   For Legacy BIOS systems, tboot must be used (which requires TXT)

\-   For UEFI mode systems, UEFI SecureBoot must be used*

Use the chart below for a guide to acceptable configuration options. .

![hardware-considerations](./images/hardware_considerations.png)

## Recommended Service Layout

The Intel® SecL-DC services can be installed in a variety of layouts, partially depending on the use cases desired and the OS of the server(s) to be protected. In general, the Intel® SecL-DC applications can be divided into management services that are deployed on the network on the management plane, and host or node components that must be installed on each protected server.

Management services can typically be deployed anywhere with network access to all of the protected servers. This could be a set of individual VMs per service; containers; or all installed on a single physical or virtual machine.

Node components must be installed on each protected physical server. Typically this is needed for Windows and Linux deployments.

### Platform Integrity

The most basic use case enabled by Intel® SecL-DC, Platform Integrity requires only the Verification Service and, to protect Windows or Linux hosts, the Trust Agent. This also enables the Application Integrity use case by default for Linux systems.

The Integration Hub may be added to provide integration support for OpenStack or Kubernetes. The Hub is often installed on the same machine as the Verification Service, but optionally can be installed separately.

### Workload Confidentiality

Workload Confidentiality introduces a number of additional services and agents. For a POC environment, all of the management services can be installed on a single machine or VM. This includes:

* Certificate Management Service (CMS)

* Authorization and Authentication Service (AAS)

* Host Verification Service (HVS)

* Workload Service (WLS)

* Integration Hub (HUB)

* Key Broker Service (KBS) with backend key management

* Workload Policy Manager (WPM)

In a production environment, it is strongly suggested that the WPM and KBS be deployed (with their own CMS and AAS) separately for each image owner. For a Cloud Service Provider, this would mean that each customer/tenant who will use the Workload Confidentiality feature would have their own dedicated AAS/CMS/KBS/WPM operated on their own networks, not controlled by the CSP. This is because the Key Broker and WPM are the tools used to define the policies that will allow images to launch, and these policies and their enforcement should remain entirely under the control of the image owner.

The node components must be installed on each protected physical server:

* Trust Agent (TA)

* Workload Agent (WLA)

![service-layout](./images/service_layout.png)



## Recommended Service Layout & Architecture - Containerized Deployment with K8s

The containerized deployment makes use of Kubernetes orchestrator for single node and multi node deployments.

A single-node deployment uses Microk8s to deploy the entire control plane in a pod on a single device.  This is best for POC or demo environments, but can also be used when integrating Intel SecL with another application that runs on a virtual machine - the single node deployment can run in the same VM as the integrated application to keep all functions local.

**Single Node:**

![k8s-single-node](./images/single-node-ws.jpg)

A multi-node deployment is a more typical Kubernetes architecture, where the Intel SecL management plane is simply deployed as a Pod, with the Intel SecL agents (the WLA and the TA, depending on use case) deployed as a DaemonSet.

**Multi Node:**

![K8s Deployment-fsws](./images/k8s-deployment-ws.jpg)

**Services Deployments & Agent DaemonSets:**

Every service including databases will be deployed as separate K8s deployment with 1 replica, i.e(1 pod per deployment). Each deployment will be further exposed through k8s service and also will be having corresponding Persistent Volume Claims(PV) for configuration and log directories and mounted on persistent storage. In case of daemonsets/agents, the configuration and log directories will be mounted on respective Baremetal worker nodes.

![](./images/service-deployment.jpg)

![Each pod consist of only one container with service](./images/daemonsets-ws.jpg)

For stateful services which requires database like shvs, aas, scs, A separate database deployment will be created for each of such services. The data present on the database deployment will also made to persist on a NFS, through K8s persistent storage mechanism

![database deployment](./images/hvs-db.jpg)

**Networking within the Cluster:**

![Networking within cluster](./images/inter-cluster-communication-ws.jpg)

**Networking Outside the Cluster:**

![Networking outside cluster](./images/within-cluster-communication-ws.jpg)

## Installing/Configuring the Database

The Intel® SecL-DC Authentication and Authorization Service (AAS) requires a Postgresql 11 database. Scripts (install_pgdb.sh, create_db.sh) are provided with the AAS that will automatically add the Postgresql repositories and install/configure a sample database. If this script will not be used, a Postgresql 11 database must be installed by the user before executing the AAS installation.

### Using the Provided Database Installation Script

Install a sample Postgresql 11 database using the install_pgdb.sh script. This script will automatically install the Postgresql database and client packages required.

Add the Postgresql repository:

https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm

Create the `iseclpgdb.env` answer file:

```shell
ISECL_PGDB_IP_INTERFACES=localhost
ISECL_PGDB_PORT=5432
ISECL_PGDB_SAVE_DB_INSTALL_LOG=true
ISECL_PGDB_CERT_DNS=localhost
ISECL_PGDB_CERT_IP=127.0.0.1
```

Note that the values above assume that the database will be accessed locally. If the database server will be external to the Intel® SecL services, change these values to the hostname or FQDN and IP address where the client will access the database server.

Run the following command:

```shell
dnf module disable postgresql -y
```

Execute the installation script:

```shell
./install_pgdb.sh
```

???+ note 
    The database installation only needs to be performed once if the same database server will be used for all services that require a database. Only the "create_db" step needs to be repeated if the database server will be shared.

### Provisioning the Database

Each Intel® SecL service that uses a database (the Authentication and Authorization Service, the Verification Service, the Integration Hub, the Workload Service) requires its own schema and access. After installation, the database must be created initialized and tables created. Execute the create_db.sh script to configure the database.

If a single shared database server will be used for each Intel® SecL service (for example, if all management plane services will be installed on a single VM), run the script multiple times, once for each service that requires a database.

If separate database servers will be used (for example, if the management plane services will reside on separate systems and will use their own local database servers), execute the script on each server hosting a database.

```shell
./create_db.sh <database name> <database_username> <database_password>
```

 For example:

```shell
./create_db.sh isecl_hvs_db hvs_db_username hvs_db_password
./create_db.sh isecl_aas_db aas_db_username aas_db_password
./create_db.sh isecl_wls_db wls_db_username wls_db_password
```

Note that the database name, username, and password details for each service must be used in the corresponding installation answer file for that service.

### Database Server TLS Certificate

The database client for Intel® SecL services requires the database TLS certificate to authenticate communication with the database server.

 If the database server for a service is located on the same server that the service will run on, only the path to this certificate is needed. If the provided Postgres scripts are used, the certificate will be located in `/usr/local/pgsql/data/server.crt`

 If the database server will be run separately from the Intel® SecL service(s), the certificate will need to be copied from the database server to the service machine before installing the Intel® SecL services.

The database client for Intel® SecL services will validate that the Subject Alternative Names in the database server’s TLS certificate contain the hostname(s)/IP address(es) that the clients will use to access the database server. If configuring a database without using the provided scripts, ensure that these attributes are present in the database TLS certificate.

## Using NATS with Intel SecL

Intel SecL-DC can utilize a NATS server to manage connectivity between the Host Verification Service and any number of deployed Trust Agent hosts.  This acts as an alternative to communication via REST APIs - in NATS mode, a connection is established with the NATS server, and messages are sent and received over that connection.

The NATS server should be deployed on the control plane and will need network connectivity to other control plane services as well as any Trust Agent hosts.

While NATS is not installed by Intel SecL directly, sample instructions for deploying a NATS server for use with Intel SecL can be found below (this is intended to be a sample only; please consult https://nats.io for official NATS documentation) :

```shell
##Download and install the NATS-server binary (see https://github.com/nats-io/nats-server/releases/latest)
rpm -i https://github.com/nats-io/nats-server/releases/download/v2.3.0/nats-server-v2.3.0-amd64.rpm

##Install tar and unzip
yum install -y tar unzip

##Install cfssl and cfssljson (for “Control-Plane Deployment” step #6).
wget https://github.com/cloudflare/cfssl/releases/download/v1.6.0/cfssljson_1.6.0_linux_amd64 -o /usr/bin/cfssljson && chmod +x /usr/bin/cfssljson

wget https://github.com/cloudflare/cfssl/releases/download/v1.6.0/cfssl_1.6.0_linux_amd64 -o /usr/bin/cfssl && chmod +x /usr/bin/cfssl
```

User has two methods to generate custom claims token for TA.

* Run populate-user.sh whenever it is required.
* Alternatively use AAS API directly to generate custom claims token.

#### Generating Custom Claims Token using populate-users script
</br>
Use the following **additional** options in populate-users.env and run populate-users:

```ini
ISECL_INSTALL_COMPONENTS=TA,HVS,AAS,NATS
NATS_CERT_SAN_LIST=<NATS server ip>,<NATS server FQDN>,localhost
NATS_CERT_COMMON_NAME=NATS TLS Certificate
```

???+ note 
    The ISECL_INSTALL_COMPONENTS list should reflect the actual components used in your deployment of Intel SecL, as dictated by the use cases to be enabled.  The important part here is to add the "NATS" element to the list.

To generate a long-lived token for use in environments where the Trust Agent may not be provisioned for an extended period of time beyond the usual lifetime of an authentication token, add the following to populate-users.env:

```ini
CUSTOM_CLAIMS_COMPONENTS=<List of services for which the long-lived installation token will be valid.  Typically this is only the Trust Agent, and so the only component in the list is typically "TA">
CUSTOM_CLAIMS_TOKEN_VALIDITY_SECS=<duration in seconds for the token to last>
CCC_ADMIN_USERNAME=<username for generating long-lived tokens>
CCC_ADMIN_PASSWORD=<password for generating long-lived tokens>
```

When populate-users script is run, there will now be an additional "Custom Claims Token For TA:" bearer token.

#### Generating Custom Claims Token using AAS API
</br>
To generate custom claims using AAS API, following are the steps to be followed:

* Create CCC_ADMIN_USERNAME and CCC_ADMIN_PASSWORD entry in the database along with the roles using the populate-users script.
```
CCC_ADMIN_USERNAME=<username for generating long-lived tokens>
CCC_ADMIN_PASSWORD=<password for generating long-lived tokens>
```
* Get the JWT from AAS to get access to the `/custom-claims-token`. Sample call to get the JWT from `/token` using the CCC_ADMIN_USERNAME/PASSWORD has been provided below:

```
curl -s -X POST "${AAS_BASE_URL}/token" --header 'Content-Type: application/json' --data-raw '{"username" : "'"$CCC_ADMIN_USERNAME"'", "password" : "'"$CCC_ADMIN_PASSWORD"'"}' -k
```
* Once the JWT has been received, admin can get the custom claims token using the JWT ($bearer_token) received in step 2. A sample curl command to get the custom claims token has been provided below:
```
curl -s -X POST "${AAS_BASE_URL}/custom-claims-token" \
--header 'Content-Type: application/json' \
--header "Authorization: Bearer $bearer_token" \
--data-raw '{
    "subject": "AAS-JWT-Issuer",
    "validity_seconds": '"$((TOKEN_VALIDITY_SECONDS))"',
    "claims": {
        "roles": [
            {
                "service": "HVS",
                "name": "AttestationRegister"
            },
            {
                "service": "AAS",
                "name": "CredentialCreator",
                "context": "type=TA"
            },
            {
                "service": "CMS",
                "name": "CertApprover",
                "context": "CN=Trust Agent TLS Certificate;SAN=*;certType=TLS"
            }
        ],
         "permissions": [
            {
                "service": "HVS",
                "rules": ["hosts:store:*", "hosts:search:*", "host_unique_flavors:create:*", "flavors:search:*",
                                        "host_aiks:certify:*", "tpm_endorsements:create:*", "tpm_endorsements:search:*"]
            },
            {
                "service": "AAS",
                "rules": ["credential:create:*"]
            }
         ]
    }
}' -k
```

Note:

-  - The "rules" under "permissions" describe which permissions are embedded in the token and describe what actions will be allowed by HVS.  The list in this example are the permissions required for the Trust-Agent.  For example, Trust-Agent command line tools like "tagent setup", "tagent setup create-host", "tagent setup create-host-unique-flavor" use HVS' REST API and therefore need a token with sufficient permissions. The CCC admin is responsible for creating tokens with the appropriate permissions.  For example, providing "hosts:store:*" allows the token permissions to add hosts to HVS (which may not be desirable in all circumstances).  For more information about permissions please refer to AAS swagger doc.
-  TOKEN_VALIDITY_SECONDS should be provided as per requirement
-  "-k" from curl request can be removed by adding AAS TLS certificate to the trusted ca directory of the respective OS

#### Installing HVS
When installing the HVS, add the following to hvs.env:

```ini
NATS_SERVERS=nats://<server ip>:4222
```

Alternatively, if the HVS is already installed, add the following in /etc/hvs/config.yml :

```yaml
nats:
  servers:
  - nats://<NATS server IP>:4222
```

Be sure to restart the HVS after changing the configuration.

Download TLS certificates for the NATS server from the CMS:

```shell
export NATS_CERT_COMMON_NAME="NATS TLS Certificate"
export CMS_ENDPOINT_URL=https://<CMS server ip or hostname>:8445/cms/v1
export NATS_CERT_SAN_LIST=" <NATS server ip>,<NATS server FQDN>,localhost"
./download-tls-certs.sh -d ./ -n "$NATS_CERT_COMMON_NAME" -u "$CMS_ENDPOINT_URL" -s "$NATS_CERT_SAN_LIST" -t $BEARER_TOKEN
```

The download-tls-certs.sh script will conenct to the CMS and will out put two files:

```
server.pem
sslcert-key.pem
```

Create a “server.conf” configuration file for nats-server (be sure the paths to the .pem files are correct):



```json
listen: 0.0.0.0:4222

tls: {
  cert_file: "./server.pem"
  key_file: "./sslcert-key.pem"
}
```

* Append the operator/account credentials from the AAS installation to the server.conf (the following can be run if NATS will run on the same machine as the AAS):  

   ```shell
   cat /etc/authservice/nats/server.conf >> server.conf
   ```

The final server.conf should look like the following:

```json
listen: 0.0.0.0:4222

tls: {
  cert_file: "/<path>/server.pem"
  key_file: "/<path>/sslcert-key.pem"
}

// Operator ISecL-operator
operator: eyJ0eXAiOiJKV1QiLCJhbGciOiJlZDI1NTE5LW5rZXkifQ.eyJleHAiOjE3ODE2MzI0NjUsImp0aSI6IkJWQVZUQkc1M01aMkFaSTVYUjRFNVlPS0xHTk5ZTE40SllYV0U3T1I1M0VQSDJOU0pFSEEiLCJpYXQiOjE2MjM5NTI0NjUsImlzcyI6Ik9DTENLS1UzS0lMWjZaRDRESDNWNTdUSkJESUdKSllMWk1RNEhKUU9DNFJFUVEyVkFUVU01SlA1IiwibmFtZSI6IklTZWNMLW9wZXJhdG9yIiwic3ViIjoiT0NMQ0tLVTNLSUxaNlpENERIM1Y1N1RKQkRJR0pKWUxaTVE0SEpRT0M0UkVRUTJWQVRVTTVKUDUiLCJuYXRzIjp7InR5cGUiOiJvcGVyYXRvciIsInZlcnNpb24iOjJ9fQ.PDlhAwk1cLHpbCCAJhKGKvv36J_NXc2PSsn6i3znmjDYHXG3C_HhO9zxsln9Bd9ViolRw_L10N1QwoMjzCBtBQ

resolver: MEMORY

resolver_preload: {
 // Account ISecL-account
 ADR7WNJ2EEEIYASP5YHDFDW6P3ICBFPXRRJVWU6CLGXOWVDIM7VIOXCM: eyJ0eXAiOiJKV1QiLCJhbGciOiJlZDI1NTE5LW5rZXkifQ.eyJleHAiOjE3ODE2MzI0NjUsImp0aSI6IjdVUlg0M0RSTUxEV0JEVTdMU0dDNTQ0UzZCRFFGVDc1TzZVWUE1QUdYNkxGV0FNWUNOTkEiLCJpYXQiOjE2MjM5NTI0NjUsImlzcyI6Ik9DTENLS1UzS0lMWjZaRDRESDNWNTdUSkJESUdKSllMWk1RNEhKUU9DNFJFUVEyVkFUVU01SlA1IiwibmFtZSI6IklTZWNMLWFjY291bnQiLCJzdWIiOiJBRFI3V05KMkVFRUlZQVNQNVlIREZEVzZQM0lDQkZQWFJSSlZXVTZDTEdYT1dWRElNN1ZJT1hDTSIsIm5hdHMiOnsibGltaXRzIjp7InN1YnMiOi0xLCJkYXRhIjotMSwicGF5bG9hZCI6LTEsImltcG9ydHMiOi0xLCJleHBvcnRzIjotMSwid2lsZGNhcmRzIjp0cnVlLCJjb25uIjotMSwibGVhZiI6LTF9LCJkZWZhdWx0X3Blcm1pc3Npb25zIjp7InB1YiI6e30sInN1YiI6e319LCJ0eXBlIjoiYWNjb3VudCIsInZlcnNpb24iOjJ9fQ.AB6eNFVE7KJspvX7DN-x_-L4mMNhPc-sDk01iOL-hEwYKfeoL9RAcdrOTwQX3CuJHMu-a3m5TpWflg1D4S1MCQ
}
```

Start NATS server:  

```shell
./nats-server -c server.conf
```

## Installing the Certificate Management Service

### Required For

The CMS is REQUIRED for all use cases.

- Platform Integrity with Data Sovereignty and Signed Flavors

- Application Integrity

- Workload Confidentiality (both VMs and Containers)

### Supported Operating Systems

The Intel® Security Libraries Certificate Management Service supports:

- Red Hat Enterprise Linux 8.2

- Ubuntu 18.04

### Recommended Hardware

- 1 vCPUs

- RAM: 2 GB

- 10 GB

- One network interface with network access to all Intel® SecL-DC services

### Installation

To install the Intel® SecL-DC Certificate Management Service:

* Copy the Certificate Management Service installation binary to the /root/ directory.

* Create the `cms.env` installation answer file for an unattended installation:

   ```shell
   AAS_TLS_SAN=<comma-separated list of IPs and hostnames for the AAS>
   AAS_API_URL=https://<Authentication and Authorization Service IP or Hostname>:8444/aas/v1
   SAN_LIST=<Comma-separated list of IP addresses and hostnames for the CMS>,127.0.0.1,localhost
   ```

   The SAN list will be used to authenticate the Certificate Signing Request from the AAS to the CMS. Only a CSR originating from a host matching the SAN list will be honored. Later, in the AAS `authservice.env` installation answer file, this same SAN list will be provided for the AAS installation. These lists must match, and must be valid for IPs and/or hostnames used by the AAS system. If both the AAS and CMS will be installed on the same system, "127.0.0.1,localhost" may be used. The SAN list variables also accept the wildcards “?” (for single-character wildcards) and "*" (for multiple-character wildcards) to allow address ranges or multiple FQDNs.

   The `AAS_API_URL` represents the URL for the AAS that will exist after the AAS is installed.

   For all configuration options and their descriptions, refer to the Intel® SecL Configuration section on the Certificate Management Service.

* Execute the installer binary.

   ```shell
   ./cms-v4.0.2.bin
   ```

   When the installation completes, the Certificate Management Service is available. The services can be verified by running cms status from the command line.

   ```shell
    cms status
   ```

   After installation is complete, the CMS will output a bearer token to the console. This token will be used with the AAS during installation to authenticate certificate requests to the CMS. If this token expires or otherwise needs to be recreated, use the following command:

   ```shell
   cms setup cms_auth_token --force
   ```

   In addition, the SHA384 digest of the CMS TLS certificate will be needed for installation of the remaining Intel® SecL services. The digest can be obtained using the following command:

   ```shell
   cms tlscertsha384
   ```
## Installing the Authentication and Authorization Service

### Required For

The AAS is REQUIRED for all use cases.

* Platform Integrity with Data Sovereignty and Signed Flavors

* Application Integrity

*  Workload Confidentiality (both VMs and Containers)

### Prerequisites

The following must be completed before installing the Authentication and Authorization Service:

* The Certificate Management Service must be installed and available

* The Authentication and Authorization Service database must be available

### Package Dependencies

The Intel® SecL-DC Authentication and Authorization Service (AAS) requires a Postgresql 11 database. A script (install_pgdb.sh) is provided with the AAS that will automatically add the Postgresql repositories and install/configure a sample database. If this script will not be used, a Postgresql 11 database must be installed by the user before executing the AAS installation.

### Supported Operating Systems

The Intel® Security Libraries Authentication and Authorization Service supports:

Red Hat Enterprise Linux 8.2

Ubuntu 18.04

### Recommended Hardware

* 1 vCPUs

* RAM: 2 GB

* 10 GB

* One network interface with network access to all Intel® SecL-DC services

### Installation

To install the AAS, a bearer token from the CMS is required. This bearer token is output at the end of the CMS installation. However, if a new token is needed, simply use the following command from the CMS command line:

```shell
cms setup cms_auth_token --force
```

 Create the `authservice.env` installation answer file:

```shell
CMS_BASE_URL=https://<CMS IP or hostname>:8445/cms/v1/
CMS_TLS_CERT_SHA384=<CMS TLS certificate sha384>
AAS_DB_HOSTNAME=<IP or hostname of database server>
AAS_DB_PORT=<database port number; default is 5432>
AAS_DB_NAME=<database name>
AAS_DB_USERNAME=<database username>
AAS_DB_PASSWORD=<database password>
AAS_DB_SSLCERTSRC=<path to database TLS certificate; the default location is typically /usr/local/pgsql/data/server.crt>
AAS_ADMIN_USERNAME=<username for AAS administrative user>
AAS_ADMIN_PASSWORD=<password for AAS administrative user>
SAN_LIST=<comma-separated list of IPs and hostnames for the AAS; this should match the value for the AAS_TLS_SAN in the cms.env file from the CMS installation>
BEARER_TOKEN=<bearer token from CMS installation>
```

Execute the AAS installer:

```shell
./authservice-v4.0.2.bin
```

???+ note 
    The `AAS_ADMIN` credentials specified in this answer file will have administrator rights for the AAS and can be used to create other users, create new roles, and assign roles to users.

### Creating Users

After installation is complete, a number of roles and user accounts must be generated. Most of these accounts will be service users, used by the various Intel® SecL services to work together. Another set of users will be used for installation permissions, and a final administrative user will be created to provide the initial authentication interface for the actual human user. The administrative user can be used to create additional users with appropriately restricted roles based on organizational needs.

Creating these required users and roles is facilitated by a script that will accept credentials and some configuration settings from an answer file and automate the process.

Create the `populate-users.env` file:

```shell
ISECL_INSTALL_COMPONENTS=KBS,TA,WLS,WPM,IHUB,HVS,WLA,AAS

AAS_API_URL=https://<AAS IP address or hostname>:8444/aas/v1
AAS_ADMIN_USERNAME=<AAS username>
AAS_ADMIN_PASSWORD=<AAS password>

HVS_CERT_SAN_LIST=<comma-separated list of IPs and hostnames for the Host Verification Service>
IH_CERT_SAN_LIST=<comma-separated list of IPs and hostnames for the Integration Hub>
WLS_CERT_SAN_LIST=<comma-separated list of IPs and hostnames for the Workload Service>
KBS_CERT_SAN_LIST=<comma-separated list of IPs and hostnames for the Key Broker Service>
TA_CERT_SAN_LIST=<comma-separated list of IPs and hostnames for the Trust Agent>

HVS_SERVICE_USERNAME=<Username for the HVS service user>
HVS_SERVICE_PASSWORD=<Password for the HVS service user>

IHUB_SERVICE_USERNAME=<Username for the Hub service user>
IHUB_SERVICE_PASSWORD=<Password for the Hub service user>

WPM_SERVICE_USERNAME=<Username for the WPM service user>
WPM_SERVICE_PASSWORD=<Password for the WPM service user>

WLS_SERVICE_USERNAME=<Username for the WLS service user>
WLS_SERVICE_PASSWORD=<Password for the WLS service user>

WLA_SERVICE_USERNAME=<Username for the WLA service user>
WLA_SERVICE_PASSWORD=<Password for the WLA service user>

GLOBAL_ADMIN_USERNAME=<Username for the global Administrator user
GLOBAL_ADMIN_PASSWORD=<Password for the global Administrator user

INSTALL_ADMIN_USERNAME=<Username for the installation user
INSTALL_ADMIN_PASSWORD=<Password for the global installation user
```

???+ note 
    The `ISECL_INSTALL_COMPONENTS` variable is a comma-separated list of the components that will be used in your environment. Not all services are required for every use case. If a given service will not be used in your deployment, simply delete the unnecessary service abbreviation from the `ISECL_INSTALL_COMPONENTS` list, and leave the SAN and credential variables for that service blank.

???+ note 
    The SAN list variables each support wildcards( "*" and "?"). In particular, without wildcards the Trust Agent SAN list would need to explicitly list each hostname or IP address for all Trust Agents that will be installed, which is not generally feasible. Using wildcards, domain names and entire IP ranges can be included in the SAN list, which will allow any host matching those ranges to install the relevant service. The SAN list specified here must exactly match the SAN list for the applicable service in that service’s env installation file.

Execute the populate-users script:

```shell
./populate-users
```

???+ note 
    The script can be executed with the `–output_json` argument to create the `populate-user.json`.This json output file will contain all of the users created by the script, along with usernames, passwords, and role assignments. This file can be used both as a record of the service and administrator accounts, and can be used as alternative inputs to recreate the same users with the same credentials in the future if needed. Be sure to protect this file if the `–output_json` argument is used.

The script will automatically generate the following users:

* Verification Service User

* Integration Hub Service User

* Workload Policy Manager Service User

* Workload Service User Name

* Workload Service User

* Global Admin User

* Installation User

These user accounts will be used during installation of several of the Intel® SecL-DC applications. In general, whenever credentials are required by an installation answer file, the variable name should match the name of the corresponding variable used in the `populate-users.env` file.

The Global Admin user account has all roles for all services. This is a default administrator account that can be used to perform any task, including creating any other users. In general this account is useful for POC installations, but in production it should be used only to create user accounts with more restrictive roles. The administrator credentials should be protected and not shared.

The populate-users script will also output an installation token. This token has all privileges needed for installation of the Intel® SecL services, and uses the credentials provided with the `INSTALLATION_ADMIN_USERNAME` and password. The remaining Intel ® SecL-DC services require this token (set as the `BEARER_TOKEN` variable in the installation env files) to grant the appropriate privileges for installation. By default this token will be valid for two hours; the populate-users script can be rerun with the same `populate-users.env` file to regenerate the token if more time is required, or the `INSTALLATION_ADMIN_USERNAME` and password can be used to generate an authentication token.



## Installing the Host Verification Service

This section details how to install the Intel® SecL-DC services. For instructions on running these services as containers, see the following section.

### Required For

The Host Verification Service is REQUIRED for all use cases.

* Platform Integrity with Data Sovereignty and Signed Flavors

* Application Integrity

* Workload Confidentiality (both VMs and Containers)

### Prerequisites

The following must be completed before installing the Verification Service:

* The Certificate Management Service must be installed and available

* The Authentication and Authorization Service must be installed and available
* The Verification Service database must be available

### Package Dependencies

The Intel® Security Libraries Verification Service requires the following packages and their dependencies:

- logback
- Postgres* client and server 11.6 (server component optional if an external Postgres database is used)
- unzip
- zip
- openssl
- wget
- net-tools
- python3-policycoreutils

If they are not already installed, the Verification Service installer attempts to install these automatically using the package manager. Automatic installation requires access to package repositories (the RHEL subscription repositories, the EPEL repository, or a suitable mirror), which may require an Internet connection. If the packages are to be installed from the package repository, be sure to update the repository package lists before installation.

### Supported Operating Systems

The Intel® Security Libraries Verification Service supports:

* Red Hat Enterprise Linux 8.2

* Ubuntu 18.04

### Recommended Hardware

* 4 vCPUs

* RAM: 8 GB

* 100 GB

* One network interface with network access to all managed servers

* (Optional) One network interface for Asset Tag provisioning (only required for “pull” tag provisioning; required to provision Asset Tags to VMware ESXi servers).

### Installation

To install the Verification Service, follow these steps:

* Copy the Verification Service installation binary to the `/root` directory.

* Create the `hvs.env` installation answer file.

   A sample minimal hvs.env file is provided below. For all configuration options and their descriptions, refer to the Intel® SecL Configuration section  on the Verification Service.

   ```shell
   # Authentication URL and service account credentials
   AAS_API_URL=https://isecl-aas:8444/aas/v1
   HVS_SERVICE_USERNAME=<username>
   HVS_SERVICE_PASSWORD=<password>

   # CMS URL and CMS webserivce TLS hash for server verification
   CMS_BASE_URL=https://isecl-cms:8445/cms/v1
   CMS_TLS_CERT_SHA384=<digest>

   # TLS Configuration
   SAN_LIST=127.0.0.1,192.168.1.1,hvs.server.com #comma-separated list of IP addresses and hostnames for the HVS to be used in the Subject Alternative Names list in the TLS Certificate

   # Installation admin bearer token for CSR approval request to CMS
   BEARER_TOKEN=eyJhbGciOiJSUzM4NCIsImtpZCI6ImE…

   # Database
   HVS_DB_NAME=<database name>
   HVS_DB_USERNAME=<database username>
   HVS_DB_PASSWORD=<database password>
   HVS_DB_SSLCERTSRC=/tmp/dbcert.pem  # Not required if VS_DB_SSLCERT is given
   ```

* Execute the installer binary.

   ```shell
   ./hvs-v4.0.2.bin
   ```

   When the installation completes, the Verification Service is available. The services can be verified by running **hvs status** from the Verification Service command line.

   ```shell
   hvs status
   ```

## Installing the Workload Service

### Required For

The WLS is REQUIRED for the following use cases.

* Workload Confidentiality (both VMs and Containers)

### Prerequisites

The following must be completed before installing the Workload Service:

* The Certificate Management Service must be installed and available

* The Authentication and Authorization Service must be installed and available

* The Verification Service must be installed and available

* The Workload Service database must be available

### Supported Operating Systems

The Intel® Security Libraries Workload Service supports:

* Red Hat Enterprise Linux 8.2

* Ubuntu 18.04

### Recommended Hardware

### Installation

* Copy the Workload Service installation binary to the `/root` directory.

* Create the `workload-service.env` installation answer file

  ```shell
  WLS_DB_USERNAME=<database username>
  WLS_DB_PASSWORD=<database password>
  WLS_DB_HOSTNAME=<IP or hostname of database server>
  WLS_DB_PORT=<Database port; 5432 by default>
  WLS_DB=<name of the WLS database>
  WLS_DB_SSLCERTSRC=<path to database TLS certificate; the default location is typically /usr/local/pgsql/data/server.crt >
  HVS_URL=https://<Ip address or hostname of the Host verification Service>:8443/hvs/v2/
  WLS_SERVICE_USERNAME=<username for WLS service account>
  WLS_SERVICE_PASSWORD=<password for WLS service account>
  CMS_BASE_URL=https://<IP or hostname to CMS>:8445/cms/v1/
  CMS_TLS_CERT_SHA384=<sha384 of CMS TLS certificate>
  AAS_API_URL=https://<IP or hostname to AAS>:8444/aas/v1/
  SAN_LIST=<comma-separated list of IPs and hostnames for the WLS>
  BEARER_TOKEN=<Installation token from populate-users script>
  ```

* Execute the WLS installer binary:

  ```shell
  ./wls-v4.0.2.bin
  ```



## Installing the Trust Agent for Linux

### Required For

The Trust Agent for Linux is REQUIRED for all use cases.

* Platform Integrity with Data Sovereignty and Signed Flavors

* Application Integrity

* Workload Confidentiality (both VMs and Containers)

### Package Dependencies

The Trust Agent requires the following packages and their dependencies:

* Tboot (Optional, for TXT-based deployments **without** UEFI SecureBoot only)

* openssl

* tar

* redhat-lsb

If they are not already installed, the Trust Agent installer attempts to install these automatically using the package manager. Automatic installation requires access to package repositories (the RHEL subscription repositories, the EPEL repository, or a suitable mirror), which may require an Internet connection. If the packages are to be installed from the package repository, be sure to update the repository package lists before installation.

Tboot will not be installed automatically. Instructions for installing and configuring tboot are documented later in this section.

### Supported Operating Systems

The Intel® Security Libraries Trust Agent for Linux supports:

* Red Hat Enterprise Linux 8.2

* Ubuntu 18.04

### Prerequisites

The following must be completed before installing the Trust Agent:

* Supported server hardware including an Intel® Xeon® processor with Intel Trusted Execution Technology activated in the system BIOS.

* Trusted Platform Module (version 2.0) installed and activated in the system BIOS, with cleared ownership status OR a known TPM ownership secret.

???+ note
	Starting in Intel SecL-DV 4.0, the Trust Agent will now default to using a null TPM owner secret, and does not require ownership permissions except during the provisioning step.   If ownership has already been taken when the Trust Agent will be provisioned, it will be necessary to either provide the ownership secret or clear the TPM ownership before provisioning.

* System must be booted to a tboot boot option OR use UEFI SecureBoot.

* (Provisioning step only) Intel® SecL Verification Service server installed and active.
* (Required for NATS mode only) A NATS server must be configured and available
* (REQUIRED for servers configured with TXT and tboot only) If the server is installed using an LVM, the LVM name must be identical for all Trust Agent systems. The Grub bootloader line that calls the Linux kernel will contain the LVM name of the root volume, and this line with all arguments is part of what is measured in the TXT/Tboot boot process. This will cause the OS Flavor measurements to differ between two otherwise identical hosts if their LVM names are different. Simply using a uniform name for the LVM during OS installation will resolve this possible discrepancy.
* (Optional, REQUIRED for Virtual Machine Confidentiality only):
  * QEMU/KVM must be installed
  * Libvirt must be installed



### Tboot Installation

Tboot is required to build a complete Chain of Trust for Intel® TXT systems that are not using UEFI Secure Boot. Tboot acts to initiate the Intel® TXT SINIT ACM (Authenticated Code Module), which populates several TPM measurements including measurement of the kernel, grub command line, and initrd. Without either tboot or UEFI Secure Boot, the Chain of Trust will be broken because the OS-related components will be neither measured nor signature-verified prior to execution. Because tboot acts to initiate the Intel® TXT SINIT ACM, tboot is only required for platforms using Intel® TXT, and is not required for platforms using another hardware Root of Trust technology like Intel® Boot Guard.

**Important Note:** SGX Attestation fails when SGX is enabled on a host booted using tboot

**Root Cause:** tboot requires the "noefi" kernel parameter to be passed during boot, in order to not use unmeasured EFI runtime services. As a result, the kernel does not expose EFI variables to user-space. SGX Attestation requires these EFI variables to fetch Platform Manifest data.

**Workaround:**

The EFI variables required by SGX are only needed during the SGX provisioning/registration phase. Once this step is completed successfully, access to the EFI variables is no longer required. This means this issue can be worked around by installing the SGX agent without booting to tboot, then rebooting the system to tboot. SGX attestation will then work as expected while booted to tboot.

1. Enable SGX and TXT in the platform BIOS

2. Perform SGX Factory Reset and boot into the “plain” distribution kernel (without tboot or TCB)

3. Install tboot and ISecL components (SGX Agent, Trust Agent and Workload Agent)

4. The SGX Agent installation fetches the SGX Platform Manifest data and caches it

5. Reboot the system into the tboot kernel mode.

6. Verify that TXT measured launch was successful:

​    txt-stat |grep "TXT measured launch"

7. The SGX and Platform Integrity Attestation use cases should now work as normal.



Intel® SecL-DC requires tboot 1.10.1 or greater. This may be a later version of tboot than is available on public software repositories.  

The most current version of tboot can be found here:

https://sourceforge.net/projects/tboot/files/tboot/

Tboot requires configuration of the grub boot loader after installation. To install and configure tboot:

* Install tboot

   ```shell
   yum install tboot
   ```

   If the package manager does not support a late enough version of tboot, it will need to be compiled from source and installed manually.  Instructions can be found here:

   https://sourceforge.net/p/tboot/wiki/Home/

   Note that the step "copy platform SINIT to /boot" should not be required, as datacenter platforms include the SINIT in the system BIOS package.

* Ensure that `multiboot2.mod` and `relocator.mod` are available for grub2

   This step may not be necessary for all OS versions, for instance, this step is NA in case of Tboot installation on Ubuntu 18.04. In order to utilize tboot, grub2 requires these two modules from the grub2-efi-x64-modules package to be located in the correct directory (if they're absent, the host will throw a grub error when it tries to boot using tboot).

   These files must be present in this directory:

   ```
   /boot/efi/EFI/redhat/x86_64-efi/multiboot2.mod
   /boot/efi/EFI/redhat/x86_64-efi/relocator.mod
   ```

   If the files are not present in this directory, they can be moved from their installation location:

   ```shell
   cp /usr/lib/grub/x86_64-efi/multiboot2.mod /boot/efi/EFI/redhat/x86_64-efi/
   cp /usr/lib/grub/x86_64-efi/relocator.mod /boot/efi/EFI/redhat/x86_64-efi/
   ```

   Make a backup of your current `grub.cfg` file

   The below examples assume a RedHat OS that has been installed on a platform using UEFI boot mode. The grub path will be slightly different for platforms using a non-RedHat OS.

   ```shell
   cp /boot/efi/EFI/redhat/grub.cfg /boot/efi/EFI/redhat/grub.cfg.bak
   ```

* Generate a new `grub.cfg` with the tboot boot option

    ```shell
    # For RHEL
    grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg

    # For Ubuntu
    grub-mkconfig -o /boot/efi/EFI/redhat/grub.cfg
    ```

* Update the default boot option

   Ensure that the `GRUB_DEFAULT` value is set to the tboot option.

   a. Update `/etc/default/grub` and set the GRUB_DEFAULT value to `'tboot-1.10.1'`

   ```
   GRUB_DEFAULT='tboot-1.10.1'
   ```

   c. Regenerate `grub.cfg`:

   ```shell
   # For RHEL
   grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg

   # For Ubuntu
   grub-mkconfig -o /boot/efi/EFI/redhat/grub.cfg
   ```

* Reboot the system

   Because measurement happens at system boot, a reboot is needed to boot to the tboot boot option and populate measurements in the TPM.

* Verify a successful trusted boot with tboot

   Tboot provides the `txt-stat` command to show the tboot log. The first part of the output of this command can be used to verify a successful trusted launch. In the output below, note the “TXT measured launch” and “secrets flag set” at the bottom. Both of these should show "**TRUE**" if the tboot measured launch was successful. If either of these show "**FALSE**" the measured launch has failed. This usually simply indicates that the tboot boot option was not selected during boot.

   If the measured launch was successful, proceed to install the Trust Agent.

   ```
   Intel(r) TXT Configuration Registers:
           STS: 0x0001c091
               senter_done: TRUE
               sexit_done: FALSE
               mem_config_lock: FALSE
               private_open: TRUE
               locality_1_open: TRUE
               locality_2_open: TRUE
           ESTS: 0x00
               txt_reset: FALSE
           E2STS: 0x0000000000000006
               secrets: TRUE
           ERRORCODE: 0x00000000
           DIDVID: 0x00000001b0078086
               vendor_id: 0x8086
               device_id: 0xb007
               revision_id: 0x1
           FSBIF: 0xffffffffffffffff
           QPIIF: 0x000000009d003000
           SINIT.BASE: 0x6fec0000
           SINIT.SIZE: 262144B (0x40000)
           HEAP.BASE: 0x6ff00000
           HEAP.SIZE: 1048576B (0x100000)
           DPR: 0x0000000070000051
               lock: TRUE
               top: 0x70000000
               size: 5MB (5242880B)
           PUBLIC.KEY:
               9c 78 f0 d8 53 de 85 4a 2f 47 76 1c 72 b8 6a 11
               16 4a 66 a9 84 c1 aa d7 92 e3 14 4f b7 1c 2d 11

   ***********************************************************
            TXT measured launch: TRUE
            secrets flag set: TRUE
   ***********************************************************
   ```

### NATS Mode vs HTTP Mode

The Trust Agent can operate in either HTTP mode (default) or NATS mode.  This distinction controls how the Agent communicates with the HVS.  In HTTP mode, the TAgent presents a set of API endpoints for the HVS to access via individual TLS requests.  In NATS mode, the Agent and HVS are connected via a persistent session maintained via a NATS server; in this mode, the Trust Agent will not listen on any HTTP ports.

### Installation

Installation of the Trust Agent is split into two major steps: Installation, which covers the creation of system files and folders, and Provisioning, which involves the creation of keys and secrets and links the Trust Agent to a specific Verification Service. Both operations can be performed at the same time using an installation answer file. Without the answer file, the Trust Agent can be installed and left in an un-provisioned state regardless of whether a Verification Service is up and running, until such time as the datacenter administrator is ready to run the provisioning step and link the Trust Agent to a Verification Service.

To install the Trust Agent for Linux:

* Copy the Trust Agent installation binary to the /root/ directory.

* (Optional; required to perform Provisioning and Installation at the same time.) Create the` trustagent.env` answer file in the `/root` directory (for full configuration options, see section 9.2). The minimum configuration options for installation are provided below.

  For Platform Attestation only, provide the following in `trustagent.env`

   ```shell
  HVS_URL=https://<Verification Service IP or Hostname>:8443/hvs/v2
  PROVISION_ATTESTATION=y
  GRUB_FILE=<path to grub.cfg>
  CURRENT_IP=<Trust Agent IP address>
  CMS_TLS_CERT_SHA384=<CMS TLS digest>
  BEARER_TOKEN=<Installation token from populate-users script>
  AAS_API_URL=https://<AAS IP or Hostname>:8444/aas/v1
  CMS_BASE_URL=https://<CMS IP or Hostname>:8445/cms/v1
  SAN_LIST=<Comma-separated list of IP addresses and hostnames for the TAgent matching the SAN list specified in the populate-users script; may include wildcards>
   ```

  For Workload Confidentiality with VM Encryption, add the following (**in addition to** the basic Platform Attestation sample):

   ```shell
  WLA_SERVICE_USERNAME=<Username for the WLA service user>
  WLA_SERVICE_PASSWORD=<Username for the WLA service user>
  WLS_API_URL=https://<WLS IP address or hostname>:5000/wls/
   ```

  For Workload Confidentiality with Container Encryption, add the following (**in addition to** the basic Platform Attestation sample):

   ```shell
  WLA_SERVICE_USERNAME=<Username for the WLA service user>
  WLA_SERVICE_PASSWORD=<Username for the WLA service user>
  WLS_API_URL=https://<WLS IP address or hostname>:5000/wls/
  REGISTRY_SCHEME_TYPE=https
  ##For the CRI-O container runtime:
  WA_WITH_CONTAINER_SECURITY_CRIO=yes
   ```

  For NATS mode, add the following (in addition to the basic Platform Attestation sample and any other optional features):

  ```
  TA_SERVICE_MODE=outbound
  NATS_SERVERS=<nats-server-ip>:4222
  TA_HOST_ID=<Any unique identifier for the host; this could be the server FQDN, a UUID, or any other unique identifier>
  ```

  Note that the TA_HOST_ID unique identifier will also be the ID used as part of the connection string to reach this Trust Agent host in NATS mode.

* Execute the Trust Agent installer and wait for the installation to complete.

  ```shell
  ./trustagent-v4.0.2.bin
  ```

If the `trustagent.env` answer file was provided with the minimum required options, the Trust Agent will be installed and also Provisioned to the Verification Service specified in the answer file.

If no answer file was provided, the Trust Agent will be installed, but will not be Provisioned. TPM-related functionality will not be available from the Trust Agent until the Provisioning step is completed.

The Trust Agent will add a new grub menu entry for application measurement. This new entry will include tboot if the existing grub contains tboot as the default boot option.

???+ note 
    If the Linux Trust Agent is installed without being Provisioned, the Trust Agent process will not actually run until the Provisioning step has been completed.

* Legacy BIOS systems using tboot ONLY) Update the grub boot loader:

  ```shell
  grub2-mkconfig -o /boot/grub2/grub.cfg
  ```

* After Provisioning is completed, the Linux Trust Agent must be rebooted so that the default SOFTWARE Flavor manifest can be measured and extended to the TPM. If the Workload Agent will also be installed on the system (see the next section), wait to reboot the server until after the Workload Agent has been installed, as this modifies the default SOFTWARE Flavor manifest.



## Installing the Workload Agent

### Required For

-   Workload Confidentiality (both VMs and Containers)

### Supported Operating Systems

The Intel® Security Libraries Workload Agent supports:

* Red Hat Enterprise

* Linux 8.2

### Prerequisites

The following must be completed before installing the Workload Agent:

-   Intel® SecL Trust Agent installed and active.

-   cryptsetup

-   (REQUIRED for Virtual Machine Confidentiality only):

-   QEMU/KVM must be installed

-   libvirt must be installed


### Installation

* Copy the Workload Agent installation binary to the /root/ directory

* Verify that the `trustagent.env` answer file is present. This file was
  necessary for installing/provisioning the Trust Agent. Note that the
  additional content required for Workload Confidentiality with either
  VM Encryption or Container Encryption must be included in the
  `trustagent.env` file (samples provided in the previous section) for
  use by the Workload Agent.

* Execute the Workload Agent installer binary.

  ```shell
  ./workload-agent-v4.0.2.bin
  ```

* Reboot the server. The Workload Agent populates files that are
  needed for the default `SOFTWARE` Flavor, and a reboot is required for
  those measurements to happen.



## Trust Agent Provisioning

"Provisioning" the Trust Agent involves connecting to a Verification
Service to download the Verification Service PrivacyCA certificate,
create a new Attestation Identity Keypair in the TPM, and verify or
create the TPM Endorsement Certificate and Endorsement Key. The
Verification Service PrivacyCA root certificate is used to sign the EC,
and the EC is used to generate the Attestation Identity Keypair. The AIK
is used by the Verification Service to verify the integrity of quotes
from the host’s TPM.  

Provisioning is the only time that the Trust Agent requires TPM ownership permissions.  If no TPM ownership secret is provided in the trustagent.env file, the Agent will use a null ownership secret to perform the provisioning steps.  If a TPM ownership secret is provided in the trustagent.env answer file, the Agent will attempt to use the specified secret.  If TPM ownership is in a clear state, the Agent will take ownership if a secret is specified.  If the TPM is already "owned," the Agent will try to use the specified secret; if the specified secret does not match the actual ownership password, the provisioning will fail.

Intel recommends using the default "null" ownership secret, as this makes it easy for other applications to also use the Trust Agent, and can prevent the need to clear ownership in the case of a need to re-provision.

Provisioning can be performed separately from installation (meaning you
can install the Trust Agent without Provisioning, and then Provision
later). If the `trustagent.env` answer file is present and has the
required Verification Service information during installation, the Agent
will automatically run the Provisioning steps.

???+ note 
    The `trustagent.env` answer file must contain user credentials for a user with sufficient privileges. The minimum role required for performing provisioning is the "AttestationRegister" role.

???+ note 
    If the Linux Trust Agent is installed without being Provisioned, the Trust Agent process will not actually run until the Provisioning step has been completed.

If the answer file is not present during installation, the Agent can be
provisioned later by adding the `trustagent.env` file and running the
following command:

`tagent setup -f <trustagent.env file
path>`



## Trust Agent Registration

Registration creates a host record with connectivity details and other
host information in the Verification Service database. This host record
will be used by the Verification Service to retrieve TPM attestation
quotes from the Trust Agent to generate an attestation report.

The Trust Agent can register the host with a Verification Service by
running the following command (the trustagent.env
answer file must be present in the current working directory):

`tagent setup create-host`

Hosts can also be registered using a REST API request to the
Verification Service:

```http
POST <https://verification.service.com:8443/hvs/v2/hosts>

{
    "host_name": "<hostname of host to be registered>"
    "connection_string": "intel:https://<hostname or IP address>:1443",
    "flavorgroup_names": [],
    "description": "<description>"
}
```

???+ note 
    When a new host is registered, the Verification Service will automatically attempt to match the host to appropriate Flavors. If appropriate Flavors are not found, the host will still be registered, but will be in an Untrusted state until/unless appropriate Flavors are added to the Verification Service.

![ta-registration](./images/ta_registration.png)

## Importing the HOST\_UNIQUE Flavor

RHEL and VMWare ESXi hosts have measured components that are unique to
each host. This means that a special HOST\_UNIQUE flavor part needs to
be imported for each RHEL and ESXi host, in addition to any other OS or
Platform Flavors.

???+ note 
    Importing a Flavor requires user credentials for a user with sufficient privileges. The minimum role required for creating the HOST\_UNIQUE Flavor part is the “host\_unique\_flavor\_creator” role. This role can only create HOST\_UNIQUE Flavor parts, and cannot create any other Flavors.

On Red Hat Enterprise Linux hosts with the Trust Agent, this can be
performed from the Trust Agent command line (this requires the
trustagent.env answer file to be present in the current working
directory):

```shell
tagent setup create-host-unique-flavor
```

This can also be performed using a REST API (required for VMWare ESXi
hosts):

```http
POST https://verification.service.com:8443/hvs/v2/flavors

{
	"connection_string": "<Connection string>",
	"partial_flavor_types": ["HOST_UNIQUE"]
}
```



## Installing the Intel® SecL Kubernetes Extensions and Integration Hub

Intel® SecL uses Custom Resource Definitions to add the ability to base
orchestration decisions on Intel® SecL security attributes to
Kubernetes. These CRDs allow Kubernetes administrators to configure pods
to require specific security attributes so that the Kubernetes Control Plane
Node will schedule those pods only on Worker Nodes that match the
specified attributes.

Two CRDs are required for integration with Intel® SecL – an extension
for the Control Plane nodes, and a scheduler extension. The extensions are deployed as a Kubernetes
deployment in the `isecl` namespace.

## Deploy Intel® SecL Custom Controller

    #Install skopeo to load container image for controller and scheduler from archive
    dnf install -y skopeo

*  Copy `isecl-k8s-extensions-*.tar.gz` to Kubernetes Control plane machine and extract the contents

    ```shell
    #Copy
    scp /<build_path>/binaries/isecl-k8s-extensions-*.tar.gz <user>@<k8s_controller_machine>:/<path>/

    #Extract
    tar -xvzf /<path>/isecl-k8s-extensions-*.tar.gz
    cd /<path>/isecl-k8s-extensions/
    ```

*  Create `hostattributes.crd.isecl.intel.com` CRD

    ```shell
    #1.14<=k8s_version<=1.16
    kubectl apply -f yamls/crd-1.14.yaml

    #1.16<=k8s_version<=1.18
    kubectl apply -f yamls/crd-1.17.yaml
    ```

* Check whether the CRD is created

   ```shell
   kubectl get crds
   ```

* Load the `isecl-controller` container image

   ```shell
   cd /<path>/isecl-k8s-extensions/
   skopeo copy oci-archive:<isecl-k8s-controller-*.tar> docker://<docker_private_registry_server>:5000/<imageName>:<tagName>
   ```

* Udate image name as above in controller yaml "/opt/isecl-k8s-extensions/yamls/isecl-controller.yaml"
   ``` shell
      containers:
        - name: isecl-controller
          image: <docker_private_registry_server>:5000/<imageName>:<tagName>
   ```

* Deploy `isecl-controller`

   ```shell
   kubectl apply -f yamls/isecl-controller.yaml
   ```

* Check whether the isecl-controller is up and running

   ```shell
   kubectl get deploy -n isecl
   ```

* Create clusterRoleBinding for ihub to get access to cluster nodes

   ```shell
   kubectl create clusterrolebinding isecl-clusterrole --clusterrole=system:node --user=system:serviceaccount:isecl:isecl
   ```

* Fetch token required for ihub installation

   ```shell
   kubectl get secrets -n isecl

   #The below token will be used for ihub installation to update 'KUBERNETES_TOKEN' in ihub.env when configured with Kubernetes Tenant
   kubectl describe secret default-token-<name> -n isecl
   ```

* Additional Optional Configurable fields for isecl-controller configuration in `isecl-controller.yaml`

| Field                 | Required   | Type     | Default | Description                                                  |
| --------------------- | ---------- | -------- | ------- | ------------------------------------------------------------ |
| LOG_LEVEL             | `Optional` | `string` | INFO    | Determines the log level                                     |
| LOG_MAX_LENGTH        | `Optional` | `int`    | 1500    | Determines the maximum length of characters in a line in log file |
| TAG_PREFIX            | `Optional` | `string` | isecl   | A custom prefix which can be applied to isecl attributes that are pushed from IH. For example, if the tag-prefix is **isecl.** and **trusted** attribute in CRD becomes **isecl.trusted**. |
| TAINT_UNTRUSTED_NODES | `Optional` | `string` | false   | If set to true. NoExec taint applied to the nodes for which trust status is set to false, Applicable only for HVS based attestation |


## Installing the Intel® SecL Integration Hub

???+ note 
    The Integration Hub is only required to integrate Intel® SecL with third-party scheduler services, such as OpenStack Nova or Kubernetes. The Hub is not required for usage models that do not require Intel® SecL security attributes to be pushed to an integration endpoint.

### Required For

The Hub is REQUIRED for the following use cases.

-   Workload Confidentiality (both VMs and Containers)

The Hub is OPTIONAL for the following use cases (used only if
orchestration or other integration support is needed):

- Platform Integrity with Data Sovereignty and Signed Flavors
- Application Integrity

### Deployment Architecture Considerations for the Hub

A separate Hub instance is REQUIRED for each Cloud environment (also referred to as a Hub "tenant").  For example, if a single datacenter will have an OpenStack cluster and also two separate Kubernetes clusters, a total of three Hub instances must be installed, though additional instances of other Intel SecL services are not required (in the same example, only a single Verification Service is required).  Each Hub will manage a single orchestrator environment.  Each Hub instance should be installed on a separate VM or physical server

### Prerequisites

The Intel® Security Libraries Integration Hub can be run as a VM or as a
bare-metal server. The Hub may be installed on the same server (physical
or VM) as the Verification Service.

-   The Verification Service must be installed and available
-   The Authentication and Authorization Service must be installed and
    available
-   The Certificate Management Service must be installed and available
-   (REQUIRED for Kubernetes integration only) The Intel SecL Custom Resource Definitions must be installed and available (see the Integration section for details)

### Package Dependencies

The Intel® SecL Integration Hub requires a number of packages and their
dependencies:

If these are not already installed, the Integration Hub installer
attempts to install these packages automatically using the package
manager. Automatic installation requires access to package repositories
(the RHEL subscription repositories, the EPEL repository, or a suitable
mirror), which may require an Internet connection. If the packages are
to be installed from the package repository, be sure to update your
repository package lists before installation.

### Supported Operating Systems

The Intel Security Libraries Integration Hub supports Red Hat Enterprise
Linux 8.2

### Recommended Hardware

-   1 vCPUs

-   RAM: 2 GB

-   1 GB free space to install the Verification Service services.
    Additional free space is needed for the Integration Hub database and
    logs (database and log space requirements are dependent on the
    number of managed servers).

-   One network interface with network access to the Verification
    Service.

-   One network interface with network access to any integration
    endpoints (for example, OpenStack Nova).

### Installing the Integration Hub

To install the Integration Hub, follow these steps:

* Copy the API Server certificate of the Kubernetes Controller to machine where Integration Hub will be installed to `/root/` directory

???+ note 
    In most  Kubernetes distributions the Kubernetes certificate and key is normally present under `/etc/kubernetes/pki`. However this might differ in case of some specific Kubernetes distributions.

    In ihub.env `KUBERNETES_TOKEN` token can be retrieved from Kubernetes using the following command:

    ```
    kubectl get secrets -n isecl -o jsonpath="{.items[?(@.metadata.annotations['kubernetes\.io/service-account\.name']=='default')].data.token}"|base64 --decode
    ```
    `KUBERNETES_CERT_FILE=/<any_path>/apiserver.crt` in this path can be specified any ex: `/root` which can taken care by IHUB during installation and copied to '/etc/ihub' directory.

*  Create the `ihub.env` installation answer file. See the
    sample file below.


```shell
# Authentication URL and service account credentials
AAS_API_URL=https://isecl-aas:8444/aas/v1
IHUB_SERVICE_USERNAME=<Username for the Hub service user>
IHUB_SERVICE_PASSWORD=<Password for the Hub service user>

# CMS URL and CMS webserivce TLS hash for server verification
CMS_BASE_URL=https://isecl-cms:8445/cms/v1
CMS_TLS_CERT_SHA384=<TLS hash>

# TLS Configuration
TLS_SAN_LIST=127.0.0.1,192.168.1.1,hub.server.com #comma-separated list of IP addresses and hostnames for the Hub to be used in the Subject Alternative Names list in the TLS Certificate

# Verification Service URL
HVS_BASE_URL=https://isecl-hvs:8443/hvs/v2
ATTESTATION_TYPE=HVS

#Integration tenant type.  Currently supported values are "KUBENETES" or "OPENSTACK"
TENANT=<KUBERNETES or OPENSTACK>

# OpenStack Integration Credentials - required for OpenStack integration only
OPENSTACK_AUTH_URL=<OpenStack Keystone URL; typically http://openstack-ip:5000/>
OPENSTACK_PLACEMENT_URL=<OpenStack Nova API URL; typically http://openstack-ip:8778/>
OPENSTACK_USERNAME=<OpenStack username>
OPENSTACK_PASSWORD=<OpenStack password>

# Kubernetes Integration Credentials - required for Kubernetes integration only
KUBERNETES_URL=https://kubernetes:6443/
KUBERNETES_CRD=custom-isecl
KUBERNETES_CERT_FILE=/root/apiserver.crt
KUBERNETES_TOKEN=eyJhbGciOiJSUzI1NiIsImtpZCI6Ik......

# Installation admin bearer token for CSR approval request to CMS - mandatory
BEARER_TOKEN=eyJhbGciOiJSUzM4NCIsImtpZCI6ImE…

```
* Update the token obtained in  Step 8 of `Deploy Intel® SecL Custom Controller` along with other relevant tenant configuration options in `ihub.env`

* Copy the Integration Hub installation binary to the `/root`
    directory & execute the installer binary.

   ```shell
   ./ihub-v4.0.2.bin
   ```

* Copy the `/etc/ihub/ihub_public_key.pem` to Kubernetes Controller machine to `/<path>/secrets/` directory

   ```shell
   #On K8s-Controller machine
   mkdir -p /<path>/secrets

   #On IHUB machine, copy
   scp /etc/ihub/ihub_public_key.pem <user>@<k8s_controller_machine>:/<path>/secrets/hvs_ihub_public_key.pem
   ```



After installation, the Hub must be configured to integrate with a Cloud orchestration platform (for example, OpenStack or Kubernetes).  See the Integration section for details.

## Deploy Intel® SecL Extended Scheduler

* Install `cfssl` and `cfssljson` on Kubernetes Control Plane

   ```shell
   #Install wget
   dnf install wget -y

   #Download cfssl to /usr/local/bin/
   wget -O /usr/local/bin/cfssl http://pkg.cfssl.org/R1.2/cfssl_linux-amd64
   chmod +x /usr/local/bin/cfssl

   #Download cfssljson to /usr/local/bin
   wget -O /usr/local/bin/cfssljson http://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
   chmod +x /usr/local/bin/cfssljson
   ```

* Create TLS key-pair for `isecl-scheduler` service which is signed by Kubernetes `apiserver.crt`

   ```shell
   cd /<path>/isecl-k8s-extensions/
   chmod +x create_k8s_extsched_cert.sh

   #Set K8s_CONTROLLER_IP,HOSTNAME
   export CONTROLLER_IP=<k8s_machine_ip>
   export HOSTNAME=<k8s_machine_hostname>

   #Create TLS key-pair
   ./create_k8s_extsched_cert.sh -n "K8S Extended Scheduler" -s "$CONTROLLER_IP","$HOSTNAME" -c <k8s_ca_authority_cert> -k <k8s_ca_authority_key>
   ```

???+ note 
    In most  Kubernetes distributions the Kubernetes certificate and key is normally present under `/etc/kubernetes/pki`. However this might differ in case of some specific Kubernetes distributions.

* Copy the TLS key-pair generated to `/<path>/secrets/` directory

   ```shell
   cp /<path>/isecl-k8s-extensions/server.key /<path>/secrets/
   cp /<path>/isecl-k8s-extensions/server.crt /<path>/secrets/
   ```

* Load the `isecl-scheduler` container image

   ```shell
   cd /<path>/isecl-k8s-extensions/
   skopeo copy oci-archive:<isecl-k8s-scheduler-*.tar> docker://<docker_private_registry_server>:5000/<imageName>:<tagName>
   ```

* Update image name as above in scheduler yaml "/opt/isecl-k8s-extensions/yamls/isecl-scheduler.yaml"
   ``` shell
      containers:
        - name: isecl-scheduler
          image: <docker_private_registry_server>:5000/<imageName>:<tagName>
   ```

* Create scheduler-secret for isecl-scheduler

   ```shell
   cd /<path>/
   kubectl create secret generic scheduler-certs --namespace isecl --from-file=secrets
   ```

* The `isecl-scheduler.yaml` file includes support for both SGX and Workload Security put together. For only working with Workload Security scenarios , the following line needs to be made empty in the yaml file. The scheduler and controller yaml files are located under `/<path>/isecl-k8s-extensions/yamls`

   ```yaml
   - name: SGX_IHUB_PUBLIC_KEY_PATH
     value: ""
   ```

* Deploy `isecl-scheduler`

   ```shell
   cd /<path>/isecl-k8s-extensions/
   kubectl apply -f yamls/isecl-scheduler.yaml      
   ```

* Check whether the `isecl-scheduler` is up and running

   ```shell
   kubectl get deploy -n isecl
   ```

* Additional optional fields for isecl-scheduler configuration in `isecl-scheduler.yaml`

| Field                    | Required   | Type     | Default | Description                                                  |
| ------------------------ | ---------- | -------- | ------- | ------------------------------------------------------------ |
| LOG_LEVEL                | `Optional` | `string` | INFO    | Determines the log level                                     |
| LOG_MAX_LENGTH           | `Optional` | `int`    | 1500    | Determines the maximum length of characters in a line in log file |
| TAG_PREFIX               | `Optional` | `string` | isecl.  | A custom prefix which can be applied to isecl attributes that are pushed from IH. For example, if the tag-prefix is ***\*isecl.\**** and ***\*trusted\**** attribute in CRD becomes ***\*isecl.trusted\****. |
| PORT                     | `Optional` | `int`    | 8888    | ISecl scheduler service port                                 |
| HVS_IHUB_PUBLIC_KEY_PATH | `Required` | `string` |         | Required for IHub with HVS Attestation                       |
| SGX_IHUB_PUBLIC_KEY_PATH | `Required` | `string` |         | Required for IHub with SGX Attestation                       |
| TLS_CERT_PATH            | `Required` | `string` |         | Path of tls certificate signed by kubernetes CA              |
| TLS_KEY_PATH             | `Required` | `string` |         | Path of tls key                                              |



#### Configuring kube-scheduler to establish communication with isecl-scheduler

???+ note 
    The below is a sample when using `kubeadm` as the Kubernetes distribution, the scheduler configuration files would be different for any other Kubernetes distributions being used.

*  Add a mount path to the
    `/etc/kubernetes/manifests/kube-scheduler.yaml` file for the Intel
    SecL scheduler extension:

    ```yaml
    - mountPath: /<path>/isecl-k8s-extensions/
      name: extendedsched
      readOnly: true
    ```

* Add a volume path to the
   `/etc/kubernetes/manifests/kube-scheduler.yaml` file for the Intel
   SecL scheduler extension:

    ```yaml
    - hostPath:
        path: /<path>/isecl-k8s-extensions/
        type: ""
        name: extendedsched
    ```

* Add `policy-config-file` path in the
   `/etc/kubernetes/manifests/kube-scheduler.yaml` file under `command` section:

   ```yaml
   - command:
     - kube-scheduler
     - --policy-config-file=/<path>/isecl-k8s-extensions/scheduler-policy.json
     - --bind-address=127.0.0.1
     - --kubeconfig=/etc/kubernetes/scheduler.conf
     - --leader-elect=true
   ```

* Restart kubelet

   ```
   systemctl restart kubelet
   ```

   Logs will be appended to older logs in

    /var/log/isecl-k8s-extensions

* Whenever the CRD's are deleted and restarted for updates, the CRD's
    using the yaml files present under `/opt/isecl-k8s-extensions/yamls/`.
    Kubernetes Version 1.14-1.15 uses `crd-1.14.yaml` and 1.16-1.17 uses
    `crd-1.17.yaml`

    ```shell
    kubectl delete crd hostattributes.crd.isecl.intel.com
    kubectl apply -f /opt/isecl-k8s-extensions/yamls/crd-<version>.yaml
    ```

* (Optional) Verify that the Intel ® SecL K8s extensions
    have been started:

    To verify the Intel SecL CRDs have been deployed:

    ```shell
    kubectl get -o json hostattributes.crd.isecl.intel.com
    ```

## Installing the Key Broker Service

### Required For

The KBS is REQUIRED for the following use cases:

-   Workload Confidentiality (both VMs and Containers)

### Prerequisites

The following must be completed before installing the Key Broker:

-   The Verification Service must be installed and available
-   The Authentication and Authorization Service must be installed and
    available
-   The Certificate Management Service must be installed and available
-   (Recommended; Required if a 3rd-party Key Management Server will
    be used) A KMIP 2.0-compliant 3rd-party Key management Server must
    be available.
-   The Key Broker will require the KMIP server’s client
        certificate, client key and root ca certificate.
-   The KMIP server's client certificate must contain a Subject Alternative Name that includes the KMIP server's hostname.
-   The Key Broker uses the gemalto kmip-go client to connect to a KMIP
        server
-   The Key Broker has been validated using the pykmip 0.9.1 KMIP
        server as a 3rd-party Key Management Server. While any general
        KMIP 2.0-compliant Key Management Server should work,
        implementation differences among KMIP providers may prevent
        functionality with specific providers.

### Package Dependencies

###   Supported Operating Systems

The Intel® Security Libraries Key Broker Service supports:

Red Hat Enterprise Linux 8.2

Ubuntu 18.04

### Recommended Hardware

### Installation

*  Copy the Key Broker installation binary to the /root/ directory.

* Create the installation answer file kbs.env:

   ```shell
   AAS_API_URL=https://<AAS IP or hostname>:8444/aas/v1
   CMS_BASE_URL=https://<CMS IP or hostname>:8445/cms/v1/
   ENDPOINT_URL=https://<KBS IP or hostname>:9443/kbs/v1/
   SAN_LIST=<comma-separated list of hostnames and IP addresses for the Key Broker>
   CMS_TLS_CERT_SHA384=<SHA384 hash of CMS TLS certificate>
   BEARER_TOKEN=<Installation token from populate-users script>

   ### OPTIONAL - KMIP configuration only
   KEY_MANAGER=KMIP
   KMIP_SERVER_IP=<IP address of KMIP server>
   KMIP_SERVER_PORT=<Port number of KMIP server>
   KMIP_VERSION=<KMIP protocol version>
   KMIP_USERNAME=<Username of KMIP server>
   KMIP_PASSWORD=<Password of KMIP server>
   ### KMIP_HOSTNAME must be used to provide, KMIP server certificate's SAN(IP/DNS) or valid COMMON NAME. Only FQDN names are allowed.
   KMIP_HOSTNAME=<Hostname of KMIP server>
   ### Retrieve the following certificates and keys from the KMIP server
   KMIP_CLIENT_KEY_PATH=<path>/client_key.pem
   KMIP_ROOT_CERT_PATH=<path>/root_certificate.pem
   KMIP_CLIENT_CERT_PATH=<path>/client_certificate.pem
   ```

*  Execute the KBS installer.

    ```shell
    ./kbs-4.0.0.bin
    ```

#### Configure the Key Broker to use a KMIP-compliant Key Management Server

The Key Broker must be configured to use a 3rd-party KMIP key manager as part of installation using kbs.env installation variables.


To configure the Key Broker to point to a 3rd-party KMIP-compliant Key Management Server:

1.  Copy the KMIP server’s client certificate, client key and root ca
    certificate to the Key Broker system

2.  Configure the variables in kbs.env for kmip support as below during installation

    ```shell
    KEY_MANAGER=KMIP
    KMIP_SERVER_IP=<IP address of KMIP server>
    KMIP_SERVER_PORT=<Port number of KMIP server>
    KMIP_HOSTNAME=<hostname of the KMIP server.  Must match the hostname used in the Subject Alternative Name fort eh KMIP server client certificate.>

    ## KMIP_VERSION variable can be used to mention KMIP protocol version.
    ## This is an OPTIONAL field, default value is set to '2.0'. KBS supports KMIP version '1.4' and '2.0'.
    KMIP_VERSION=<KMIP protocol version>

    ## KMIP_HOSTNAME can be used to configure TLS config with ServerName.
    ## KMIP server certificate should contain SAN(IP/DNS) or valid COMMON NAME and this value can be provided in KMIP_HOSTNAME. Only FQDN names are allowed.
    ## This is an OPTIONAL field, if KMIP_HOSTNAME is not provided then KMIP_SERVER_IP will be considered as ServerName in TLS configuration.
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
    ```


3.  The KBS configuration can be found in `/etc/kbs/config.yml`, KMIP configuration can be updated in this configuration

    ```shell
    kmip:
      version: "2.0"
      server-ip: "127.0.0.1"
      server-port: "5696"
      hostname: "localhost"
      kmip-username: "<kmip-username>"
      kmip-password: "<kmip-password>"
      client-key-path: "<path>/client-key.pem"
      client-cert-path: "<path>/client-certificate.pem"
      root-cert-path: "<path>/root-certificate.pem"
  ```

4.  Restart the Key Broker for the settings to take effect

    ```shell
    kbs stop
    kbs start
    ```

### Importing Verification Service Certificates

After installation, the Key Broker must import the SAML and PrivacyCA
certificates from any Verification Services it will trust. This provides
the Key Broker a way to ensure that only attestations that come from a
“known” Verification Service. The SAML and PrivacyCA certificates needed
can be found on the Verification Service.

#### Importing a SAML certificate

Display the SAML certificate:

```shell
cat /etc/hvs/certs/trustedca/saml-crt.pem
```

Use the SAML certificate output in the following POST call to the Key
Broker:

```
POST https://<Key Broker IP address or hostname>:9443/kbs/v1/saml-certificates
Content-Type: application/x-pem-file
-----BEGIN CERTIFICATE-----
MIID9TCCAl2gAwIBAgIBCTANBgkqhkiG9w0BAQwFADBQMQswCQYDVQQGEwJVUzEL
MAkGA1UECBMCU0YxCzAJBgNVBAcTAlNDMQ4wDAYDVQQKEwVJTlRFTDEXMBUGA1UE
AxMOQ01TIFNpZ25pbmcgQ0EwHhcNMTkxMjExMTkzOTU1WhcNMjAxMjExMTkzOTU1
WjAYMRYwFAYDVQQDEw1tdHdpbHNvbi1zYW1sMIIBojANBgkqhkiG9w0BAQEFAAOC
AY8AMIIBigKCAYEArbrDpzR4Ry0MVhSJULHZoiVL020YqtyRH+R2NlVXTpJzqmEA
Ep2utfcP8+mSCT7DLpGBO6KACPCz3pmqj3wZyqZNTrG7IF2Z4Fuf641fPcxA3WVH
3lXz0L5Ep4jOUdfT8kj4hHxHJVJhDsW4J2fds2RGnn8bZG/QbmmGNRfqdxht0zMh
63ik8jBWNWHxYSRbck27FyTj9hDU+z+rFfIdNv1SiQ9FyndgOytK/m7ijoAetkSF
bCsauzUL7DFdRzTmB2GCF/Zd957V51GNpvan6uwqDTL6T4NFX2sqoVduu/WIyTpO
/6D2aA741CR3Bmk9945TSeDKZNz2HkihuE+d8ES68W1t4rvox/Noi74e0k35AqcQ
Q3P0DZpD+XaRapz5CHcOPwOpZ3A/8wN2f+CS2HqDx8FwABkh7l8OdiIWs8+TDQZe
1x4e/50jE/8zMR/tsAy1EXkm3OTOVxih0u18J84x4OT+rHAIcoQ+TOJ40aHrWGHg
kVCfiCUzYYT/W/RBAgMBAAGjEjAQMA4GA1UdDwEB/wQEAwIGwDANBgkqhkiG9w0B
AQwFAAOCAYEAP/ABHdPquBNrMOCU+v7SfMLmIfJymA15mCorMEiKZ1d7oNnoPP0G
pfyRA4TUiyFLCOLi4jIXWuu4Lt6RUz6bnzn8JRWD5ocIJGGxWjOA66xyS3o9iG7G
otOh1pzp5wlwPG7r8ZJ7Q26J+NuHpN1GW5U5Vjww1J9rEKnsKp45QHkG2nXEujdx
YXmKXtEG2gOMVjaLiqromf6VxbdNoKHZGEfqU3H5ymMgqIrnXl3MivA30CymCDLl
rJGRQSwOfzywPCnUOAVptBwLs2kwOtdvnq+BTK3q/dKKoNiFURj/mQ70egquW9ly
TOkYivmKqMZxZlq0//cre4K35aCW3ZArzGgNM8Pk0V/hZp8ZHrWLNAdo4w/Pj1oC
Yq7R0RQ8jQerkewYBfrv3O3e9c22h48fsHnun6F3sbcDjws/sWJIOcrPyqJE26HY
DmIKpvjqc0jI31ndBBwkb+RIBFkz1Ycob9rsW16uVqbjBFDjJ5QKOdXxhqulyboa
JAF53vmU+1jE
-----END CERTIFICATE-----
```

#### Importing a PrivacyCA Certificate

Use OpenSSL to display the PrivacyCA certificate content:

```shell
openssl x509 -in /etc/hvs/certs/trustedca/privacy-ca/privacy-ca-cert.pem
```

Use the PrivacyCA certificate output in the following POST call to the
Key Broker:

```
POST https://<Key Broker IP address or hostname>:9443/kbs/v1/tpm-identity-certificates
Content-Type: application/x-pem-file
-----BEGIN CERTIFICATE-----
MIIHaDCCBdCgAwIBAgIGAW72eWZ9MA0GCSqGSIb3DQEBCwUAMBsxGTAXBgNVBAMT
EG10d2lsc29uLXBjYS1haWswHhcNMTkxMjExMTkzOTQxWhcNMjkxMjEwMTkzOTQx
WjAbMRkwFwYDVQQDExBtdHdpbHNvbi1wY2EtYWlrMIIBojANBgkqhkiG9w0BAQEF
AAOCAY8AMIIBigKCAYEAmWqBr2YiycZbF/QgFbxTr4YiHtueWBdW0sibtH1QRSbI
KtkbFsmr6J6QiLBaXcF7KVN6DaD0j5sU4cZSttqKwlSUnn07xjWJRP1EcvSaufO1
MarewgBpFQcI2T6aTs1ziV77BoKz0kWteURz1jT1KSwuattxTelpmgucDp98MqW/
uWsliHUVxh51JTE1yn7Vf1QCWz3a+NDH98Lgr5ks337yx3VBK59Dwtsmfsrd5tMn
IuV9Jw0Y2UEdDi004FXI4q64MsMpWA7t5ONRAU+VNU0Y3saXeNBDg9J363imOHIH
haP8ixDhqZ+Xb/TGafgFeEHBkJTv6bWpDqodbWVDbgZloxJzcNgtimQw3RbyrB3C
KijlEo5BQY6bOcdMG7gCq77u/fbOvLb5IXzS8ZDpwuwCQNnBP4UJXwAflO7COG7P
mpj9bTV1OtFiPtYFc4JdGdaf1Pl2zWGeR0c3PIzYQxqvtTVtFX+oRWRsgaEdxKf7
LJx4aIjXwP2s6PIiOSalAgMBAAGjggOwMIIDrDCCAbMGA1UdDgSCAaoEggGmMIIB
ojANBgkqhkiG9w0BAQEFAAOCAY8AMIIBigKCAYEAmWqBr2YiycZbF/QgFbxTr4Yi
HtueWBdW0sibtH1QRSbIKtkbFsmr6J6QiLBaXcF7KVN6DaD0j5sU4cZSttqKwlSU
nn07xjWJRP1EcvSaufO1MarewgBpFQcI2T6aTs1ziV77BoKz0kWteURz1jT1KSwu
attxTelpmgucDp98MqW/uWsliHUVxh51JTE1yn7Vf1QCWz3a+NDH98Lgr5ks337y
x3VBK59Dwtsmfsrd5tMnIuV9Jw0Y2UEdDi004FXI4q64MsMpWA7t5ONRAU+VNU0Y
3saXeNBDg9J363imOHIHhaP8ixDhqZ+Xb/TGafgFeEHBkJTv6bWpDqodbWVDbgZl
oxJzcNgtimQw3RbyrB3CKijlEo5BQY6bOcdMG7gCq77u/fbOvLb5IXzS8ZDpwuwC
QNnBP4UJXwAflO7COG7Pmpj9bTV1OtFiPtYFc4JdGdaf1Pl2zWGeR0c3PIzYQxqv
tTVtFX+oRWRsgaEdxKf7LJx4aIjXwP2s6PIiOSalAgMBAAEwDwYDVR0TAQH/BAUw
AwEB/zCCAeAGA1UdIwSCAdcwggHTgIIBpjCCAaIwDQYJKoZIhvcNAQEBBQADggGP
ADCCAYoCggGBAJlqga9mIsnGWxf0IBW8U6+GIh7bnlgXVtLIm7R9UEUmyCrZGxbJ
q+iekIiwWl3BeylTeg2g9I+bFOHGUrbaisJUlJ59O8Y1iUT9RHL0mrnztTGq3sIA
aRUHCNk+mk7Nc4le+waCs9JFrXlEc9Y09SksLmrbcU3paZoLnA6ffDKlv7lrJYh1
FcYedSUxNcp+1X9UAls92vjQx/fC4K+ZLN9+8sd1QSufQ8LbJn7K3ebTJyLlfScN
GNlBHQ4tNOBVyOKuuDLDKVgO7eTjUQFPlTVNGN7Gl3jQQ4PSd+t4pjhyB4Wj/IsQ
4amfl2/0xmn4BXhBwZCU7+m1qQ6qHW1lQ24GZaMSc3DYLYpkMN0W8qwdwioo5RKO
QUGOmznHTBu4Aqu+7v32zry2+SF80vGQ6cLsAkDZwT+FCV8AH5Tuwjhuz5qY/W01
dTrRYj7WBXOCXRnWn9T5ds1hnkdHNzyM2EMar7U1bRV/qEVkbIGhHcSn+yyceGiI
18D9rOjyIjkmpQIDAQABoR+kHTAbMRkwFwYDVQQDExBtdHdpbHNvbi1wY2EtYWlr
ggYBbvZ5Zn0wDQYJKoZIhvcNAQELBQADggGBAC3PEB8Av0PBJgrJMxzMbuf1FCdD
AUrfYmP81Hs0/v70efviMEF2s3GAyLHD9v+1nNFCQrjcNCar18k45BlcodBEmxKA
DZoioFykRtlha6ByVvuN6wD93KQbKsXPKhUp8X67fLuOcQgfc3BoDRlw/Ha1Ib6X
fliE+rQzLCOgClK7ZdTwl9Ok0VbR7Mbal/xShIqr2WopjBtal9p4RsnIxilTHI+m
qzbV8zvZXYfYtEb3MMMT5EnjIV8O498KKOjxohD2vqaxqItd58pOi6z/q5f4pLHc
DvdsJecJEoWb2bxWQdBgthMjX6AUV/B5G/LTfaPwVbTLdEc+S6Nrobf/TFYV0pvG
OzF3ltYag0fupuYJ991s/JhVwgJhCGq7YourDGkNIWAjt0Z2FWuQKnxWvmResgkS
WTeXt+1HCFSo5WcAZWV8R9FYv7tzFxPY8aoLj82sgrOE4IwRqaA8KMbq3anF4RCk
+D8k6etqMcNHFS8Fj6GlCd80mb4Q3sxuCiBvZw==
-----END CERTIFICATE-----
```



## Installing the Workload Policy Manager

### Required For

The WPM is REQUIRED for the following use cases.

-   Workload Confidentiality (both VMs and Containers)

### Package Dependencies



### Supported Operating Systems

The Intel® Security Libraries Workload Policy Manager supports:

* Red Hat Enterprise Linux 8.2

* Ubuntu 18.04

### Recommended Hardware

-   2 vCPUs

-   RAM: 8 GB

-   100 GB

-   One network interface with network access to the Key Broker and
    Workload Service

-   Additional memory and disk space may be required depending on the
    size of images to be encrypted

### Installation

* Copy the WPM installer to the `/root` directory

* Create the `wpm.env` answer file:

   ```shell
   KBS_BASE_URL=https://<IP address or hostname of the KBS>:9443/v1/
   WPM_SERVICE_USERNAME=<WPM_Service username from populate-users script>
   WPM_SERVICE_PASSWORD=<WPM Service password from populate-users script>
   CMS_TLS_CERT_SHA384=<Sha384 hash of the CMS TLS certificate>
   CMS_BASE_URL=https://<IP address or hostname for CMS>:8445/cms/v1/
   AAS_API_URL=https://<Hostname or IP address of the AAS>:8444/aas/v1
   BEARER_TOKEN=<Installation token from populate-users script>
   ```

   For  Container Encryption only, add the following line to the
   wpm.env installation answer file:

   ```shell
   ##For the CRI-O container runtime:
   WPM_WITH_CONTAINER_SECURITY_CRIO=yes
   ```

* Execute the WPM installer:

    ```shell
    ./wpm-v4.0.2.bin
    ```