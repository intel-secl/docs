# Quick Start Guide - SGX Attestation Infrastructure and Secure Key Caching (SKC)

Table of Contents
-----------------

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Quick Start Guide - SGX Attestation Infrastructure and Secure Key Caching (SKC)](#quick-start-guide---sgx-attestation-infrastructure-and-secure-key-caching-skc)
  - [Table of Contents](#table-of-contents)
  - [**1. Introduction**](#1-introduction)
    - [Glossary](#glossary)
    - [Intel® SecL-DC libraries Services Deployment Matrix for All Supported Usecases](#intel-secl-dc-libraries-services-deployment-matrix-for-all-supported-usecases)
  - [**2. Hardware Requirements**](#2-hardware-requirements)
  - [**3. OS Requirements**](#3-os-requirements)
    - [User Access](#user-access)
    - [Proxy Settings](#proxy-settings)
  - [**4. Build Services**](#4-build-services)
    - [Pre-requisites](#pre-requisites)
    - [Building SGX Attestation Usecase](#building-sgx-attestation-usecase)
    - [Building Orchestration Usecase](#building-orchestration-usecase)
    - [Building Secure Key Caching Usecase](#building-secure-key-caching-usecase)
    - [Building Sample Application](#building-sample-application)
    - [Building All Usecases - Distribution Based Deployment](#building-all-usecases-distribution-based-deployment)
    - [Building All Usecases - Stack Based Deployment](#building-all-usecases-stack-based-deployment)
  - [**5. Deployment Model**](#5-deployment-model)
- [**6. Deployment**](#6-deployment)
  - [**Deploy Linux Stacks for Intel SGX**](#deploy-linux-stacks-for-intel-sgx)
  - [**Deployment Using Binaries**](#deployment-using-binaries)
    - [Deploying Control Plane Services on a Enterprise network](#deploying-control-plane-services-on-a-enterprise-network)
      - [Deploying Services for SGX Attestation Usecase](#deploying-services-for-sgx-attestation-usecase)
    - [Deploying Control Plane Services on Enterprise and Cloud Service Networks](#deploying-control-plane-services-on-enterprise-and-cloud-service-networks)
      - [Deploy Enterprise SKC Services](#deploy-enterprise-skc-services)
      - [Deploy CSP SKC Services](#deploy-csp-skc-services)
      - [Deploying Services for Secure Key Caching Usecase](#deploying-services-for-secure-key-caching-usecase)
      - [Deploying Services for Orchestration Usecase](#deploying-services-for-orchestration-usecase)
      - [Deploying Services for All Usecases](#deploying-services-for-all-usecases)
      - [Deploy SGX Agent](#deploy-sgx-agent)
      - [Deploy SKC Library](#deploy-skc-library)
  - [**Deployment & Usecase Workflow Tools Installation**](#deployment--usecase-workflow-tools-installation)
    - [Usecases Workflow Tools Installation](#usecases-workflow-tools-installation)
  - [**Deployment Using Ansible**](#deployment-using-ansible)
    - [Pre-requisites](#pre-requisites-1)
    - [Installing Ansible on Build Machine](#installing-ansible-on-build-machine)
    - [Download the Ansible Role](#download-the-ansible-role)
    - [Usecase Setup Options](#usecase-setup-options)
    - [Update Ansible Inventory](#update-ansible-inventory)
    - [Create and Run Playbook](#create-and-run-playbook)
      - [Setup K8S Cluster and Deploy Isecl-k8s-extensions](#setup-k8s-cluster-and-deploy-isecl-k8s-extensions)
        - [Untar packages and push OCI images to registry](#untar-packages-and-push-oci-images-to-registry)
        - [Deploy isecl-controller](#deploy-isecl-controller)
        - [Deploy isecl-scheduler](#deploy-isecl-scheduler)
        - [Configure kube-scheduler to establish communication with isecl-scheduler](#configure-kube-scheduler-to-establish-communication-with-isecl-scheduler)
        - [Validate POD launch](#validate-pod-launch)
      - [Openstack Setup and Associate Traits](#openstack-setup-and-associate-traits)
  - [**7. Usecase Workflows API Collections**](#7-usecase-workflows-api-collections)
    - [Pre-requisites](#pre-requisites-2)
    - [Use Case Collections](#use-case-collections)
    - [Downloading API Collections](#downloading-api-collections)
    - [Running API Collections](#running-api-collections)
  - [**8. System User Configuration**](#8-system-user-configuration)
  - [**9. Appendix**](#9-appendix)
    - [SGX Attestation flow](#sgx-attestation-flow)
    - [Creating RSA Keys in Key Broker Service](#creating-rsa-keys-in-key-broker-service)
    - [Configuration for NGINX testing](#configuration-for-nginx-testing)
    - [KBS key-transfer flow validation](#kbs-key-transfer-flow-validation)
    - [Note on Key Transfer Policy](#note-on-key-transfer-policy)
    - [Note on SKC Library Deployment](#note-on-skc-library-deployment)
    - [Deploy SKC Library as a container](#deploying-skc-library-as-a-container)
    - [Extracting SGX Enclave values for Key Transfer Policy](#extracting-sgx-enclave-values-for-key-transfer-policy)
<!-- /code_chunk_output -->

## **1. Introduction**

[Intel® SecL-DC libraries](https://github.com/intel-secl/docs/-/blob/v4.1/develop/product-guides/Secure%20Key%20Caching.md#sgx-attestation-infrastructure-and-skc-components) support SGX attestation as a base use case. For a detailed description of SGX Attestation, please refer to [SGX ECDSA Attestation](https://github.com/intel-secl/docs/-/blob/v4.1/product-guides/SGX%20Infrastructure.md#sgx-ecdsa-attestation)

Intel® SecL-DC libraries also support following usecases which build upon the SGX Attestation usecase
* Secure Key Caching
* SGX Orchestration Support for Kubernetes and Openstack


###  Glossary

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


### Intel® SecL-DC libraries Services Deployment Matrix for All Supported Usecases
|                       Usecases            | CMS | AAS | SGX Agent | SCS | SQVS | SKC Library | KBS | SHVS | IHUB |
|-----------------------------------|-----|-----|-----------|-----|------|-------------|-----|------|------|
| SGX Attestation                   | ✔   | ✔   | ✔         | ✔   | ✔    |             |     |      |      |
| Secure Key Caching                | ✔   | ✔   | ✔         | ✔   | ✔    | ✔           | ✔   |      |      |
| Orchestration Support             | ✔   | ✔   | ✔         | ✔   |      |             |     | ✔    | ✔    |


## **2. Hardware Requirements**

For Enterprise Network Deployment

- Build System (RHEL 8.2/8.4 Ubuntu 18.04/20.04 system)
- Deployment System (SGX Agent and SKC Library need to be deployed on SGX Compute Node). Other Services can be installed anywhere

> Note: SGX Compute node can be used to build and deploy all services

For Cloud Service Provider (CSP) and Enterprise Network Deployment

- Build System (RHEL 8.2/8.4 Ubuntu 18.04/20.04 system)
- SGX Compute Node on CSP Network Deployment System (For deploying AAS, CMS, SCS, SGX Agent and SKC Client)
- Enterprise Network Deployment System (For deploying AAS, CMS, SCS, SQVS)
- Deployment system for orchestration support


## **3. OS Requirements**

Ensure that you have one of the following required operating systems:

* Ubuntu* 18.04/20.04 LTS
* Red Hat Enterprise Linux Server release 8.2 64bits
* Red Hat Enterprise Linux Server release 8.4 64bits for Stack based deployment

Date and time should be synced across all systems. If a system is configured to read the RTC time in the local time zone, then use RTC in UTC by running `timedatectl set-local-rtc 0` command on all deployment systems.

### User Access

* All services need to be built & deployed as `root` user.
> **Note:** When using Ansible role for deployment, Ansible needs to be able to talk to remote machines as root user for successful deployment
* All Intel® SecL-DC service & agent ports should be allowed in firewall rules. 

### Proxy Settings

If system is behind a proxy, configure system to use proxy. 
Sample configuration
```
export http_proxy=http://<proxy-url>:<proxy-port>
export https_proxy=http://<proxy-url>:<proxy-port>
export no_proxy=0.0.0.0,127.0.0.1,localhost,<Deployment System IP>, <SGX Compute Node IP>, <KBS system Hostname>
```

## **4. Build Services**

The below steps needs to be carried out on the Build and Deployment VM

### Pre-requisites

* Install basic utilities for getting started

On Red Hat Enterprise Linux 8.2/8.4

Enable the Following package repositories

  * `rhel-8-for-x86_64-appstream-rpms`
  * `rhel-8-for-x86_64-baseos-rpms`
  * `codeready-builder-for-rhel-8-for-x86_64-rpms`

```shell
dnf install git wget tar python3 gcc gcc-c++ zip tar make yum-utils curl openssl-devel

# RHEL-8.2
	dnf install skopeo -y
# RHEL-8.4
	dnf install skopeo -y --nobest
  
dnf install https://download-ib01.fedoraproject.org/pub/epel/8/Everything/x86_64/Packages/m/makeself-2.4.2-1.el8.noarch.rpm
ln -s /usr/bin/python3 /usr/bin/python
ln -s /usr/bin/pip3 /usr/bin/pip
export PATH=/usr/local/bin:$PATH
```
On Ubuntu

```shell
apt-get install -y software-properties-common git gcc zip wget make python3 python3-yaml python3-pip tar lsof jq nginx curl libssl-dev
ln -s /usr/bin/python3 /usr/bin/python
ln -s /usr/bin/pip3 /usr/bin/pip

sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
apt-get update

# Ubuntu-18.04
echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_18.04/ /" | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_18.04/Release.key | sudo apt-key add -
apt-get update
apt-get -y upgrade
apt-get -y install skopeo

# Ubuntu-20.04
echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_20.04/ /" | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_20.04/Release.key | sudo apt-key add -
apt-get update
apt-get -y upgrade
apt-get -y install skopeo

apt-get install makeself
export PATH=/usr/local/bin:$PATH
```

* Install Repo Tool

```shell
tmpdir=$(mktemp -d)
git clone https://gerrit.googlesource.com/git-repo $tmpdir
install -m 755 $tmpdir/repo /usr/local/bin
rm -rf $tmpdir
```

* Golang Installation

```shell
wget https://dl.google.com/go/go1.16.7.linux-amd64.tar.gz
tar -xzf go1.16.7.linux-amd64.tar.gz
mv go /usr/local
export GOROOT=/usr/local/go
export PATH=$GOROOT/bin:$PATH
rm -rf go1.16.7.linux-amd64.tar.gz
```

* Install, Enable and start the Docker daemon
  ```shell
  On RHEL 8.2:
    dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
    dnf install -y docker-ce-20.10.8 docker-ce-cli-20.10.8

  On RHEL 8.4:
    dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
    dnf install -y docker-ce-20.10.9 docker-ce-cli-20.10.9

  On Ubuntu 18.04:
    wget https://download.docker.com/linux/ubuntu/dists/bionic/pool/stable/amd64/containerd.io_1.4.11-1_amd64.deb
    dpkg -i containerd.io_1.4.11-1_amd64.deb
    wget "https://download.docker.com/linux/ubuntu/dists/bionic/pool/stable/amd64/docker-ce-cli_20.10.8~3-0~ubuntu-bionic_amd64.deb"
    dpkg -i docker-ce-cli_20.10.8~3-0~ubuntu-bionic_amd64.deb
    wget "https://download.docker.com/linux/ubuntu/dists/bionic/pool/stable/amd64/docker-ce_20.10.8~3-0~ubuntu-bionic_amd64.deb"
    dpkg -i docker-ce_20.10.8~3-0~ubuntu-bionic_amd64.deb

  On Ubuntu 20.04:
    wget https://download.docker.com/linux/ubuntu/dists/focal/pool/stable/amd64/containerd.io_1.4.11-1_amd64.deb
    dpkg -i containerd.io_1.4.11-1_amd64.deb
    wget "https://download.docker.com/linux/ubuntu/dists/focal/pool/stable/amd64/docker-ce-cli_20.10.8~3-0~ubuntu-focal_amd64.deb"
    dpkg -i docker-ce-cli_20.10.8~3-0~ubuntu-focal_amd64.deb
    wget "https://download.docker.com/linux/ubuntu/dists/focal/pool/stable/amd64/docker-ce_20.10.8~3-0~ubuntu-focal_amd64.deb"
    dpkg -i docker-ce_20.10.8~3-0~ubuntu-focal_amd64.deb

  systemctl enable docker
  systemctl start docker
  ```

* Ignore the below steps if not running behind a proxy
  ```shell
  mkdir -p /etc/systemd/system/docker.service.d
  touch /etc/systemd/system/docker.service.d/proxy.conf
  
  #Add the below lines in proxy.conf and set proxy server details if proxy is used
  [Service]
  Environment="HTTP_PROXY=<http_proxy>"
  Environment="HTTPS_PROXY=<https_proxy>"
  Environment="NO_PROXY=<no_proxy>"
  ```

  ```shell
  #Reload docker
  systemctl daemon-reload
  systemctl restart docker
  ```

* Pulling Source Code

```shell
mkdir -p /root/workspace && cd /root/workspace
repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v4.1.0 -m manifest/skc.xml
repo sync
```

### Building SGX Attestation Usecase
For buiding SGX Attestation usecase only - Distribution Based Deployment
```shell
make attest
```

For buiding SGX Attestation usecase only - Stack Based Deployment
```shell
make attest_stacks
```

### Building Orchestration Usecase
For buiding Orchestration usecase - Distribution Based Deployment
```shell
make orchestrator
```

For buiding Orchestration usecase - Stack Based Deployment
```shell
make orchestrator_stacks
```

### Building Secure Key Caching Usecase
For buiding Secure Key Caching usecase  - Distribution Based Deployment
```shell
make skc
```

For buiding Secure Key Caching usecase - Stack Based Deployment
```shell
make skc_stacks
```

### Building Sample Application
For buiding Sample Application for SGX Attestation - Distribution Based Deployment
```shell
make app
```

For buiding Sample Application for SGX Attestation - Stack Based Deployment
```shell
make app_stacks
```

### Building All Usecases - Distribution Based Deployment
```shell
make
```

### Building All Usecases - Stack Based Deployment
```shell
make sgx-stack
```

* Built Binaries
```shell
cd /root/workspace/binaries
```
Copy the binaries directory generated in the build system to the /root/ directory on the deployment system


## **5. Deployment Model**

Following deployment models are supported

* Enterprise Network Deployment Model

In this model, all services will be deployed on a single enterprise network

    Deploy all services on a SGX Compute Node

    Deploy SGX Agent and SKC Client on SGX Compute Node and all other services on another system


* Cloud Service Provider and Enterprise Network Model

In this model, some services are deployed on CSP network and other services on Enterprise Network.

    Deploy CMS, AAS, SCS, SGX Agent, SKC Client on Cloud Service Provider Network

    Deploy CMS, AAS, SCS and SQVS on Enterprise Network
    
Following diagram illustrates the CSP/Enterprise Network Model

 ![deploy_model](./images/isecl_deploy_model.PNG)


For most of the requirements, Enterprise Network Deployment model should suffice.

# **6. Deployment**

## **Deploy Linux Stacks for Intel SGX**

* To setup and deployment of the Linux* Stacks for Intel® SGX, follow https://download.01.org/intelsgxstack/2021-07-28/Getting_Started.pdf
* Install Kubernetes Stack on control-plane node and SGX compute node using Linux SGX Stacks Ansible. Control-plane node can be any system, Compute node should be setup on SGX enabled host machine.
* Once the Kubernetes stack deployed in control-plane and SGX compute node, follow the below steps for deploying Intel® SecL-DC Secure Key Caching Usecase through Ansible Role.

## **Deployment Using Binaries**

 The Control Plane Services comprise of AAS, CMS, SCS, SQVS, KBS, SHVS and IHUB. These services can be deployed as below

 1. Deploy All Services on a SGX Compute Node
 2. Deploy SGX Agent on a SGX Compute Node and deploy all Control Plane Services on a VM
 3. Deploy SGX Agent on a SGX Compute Node. Deploy some Control Plane Services on an enterprise network and others on Cloud Service Provider network.
 
### Deploying Control Plane Services on a Enterprise network

#### Deploying Services for SGX Attestation Usecase
On Deployment System
```shell
cd /root/binaries/
```

Update enterprise_skc.conf

  - SYSTEM_IP (Deployment System IP address)
  - SYSTEM_SAN (Subject Alternative Name List contains list of all domain names for the deployment system)
  - Network Port numbers for CMS, AAS, SCS and SQVS (Default Ports can be specified as mentioned in config file)
  - INSTALL_ADMIN_USERNAME and INSTALL_ADMIN_PASSWORD (Admin Credentials for installation of services)
  - CCC_ADMIN_USERNAME and CCC_ADMIN_PASSWORD (User credentials for generating long-lived tokens)
  - Database name, Database username and password for AAS and SCS services
  - Intel PCS Server API URL and API Keys (Refer config file for instructions on getting a API Key)


If ECDSA Quote verification response needs to be signed, please set SIGN_QUOTE_RESPONSE=true in /root/binaries/env/sqvs.env

For ICX Production CPUs, copy sgx-verification-service/dist/linux/rootca_icx_prod.pem as trusted_rootca.pem in /root/binaries/env diretory

```shell
./install_sgx_infra.sh
```

### Deploying Control Plane Services on Enterprise and Cloud Service Networks

#### Deploy Enterprise SKC Services
On Deployment System
```shell
cd /root/binaries/
```

Update enterprise_skc.conf

  - Deployment System IP address
  - SAN List (Subject Alternative Name List contains list of all domain names for the deployment system)
  - Network Port numbers for CMS, AAS, SCS, SQVS and KBS (Default Ports can be specified as mentioned in config file)
  - INSTALL_ADMIN_USERNAME and INSTALL_ADMIN_PASSWORD (Admin Credentials for installation of services)
  - CCC_ADMIN_USERNAME and CCC_ADMIN_PASSWORD (User credentials for generating long-lived tokens)
  - Database name, Database username and password for AAS and SCS services
  - Intel PCS Server API URL and API Keys (Refer config file for instructions on getting a API Key)  
  - Key Manager (can be set to either Directory or KMIP)
  - KMIP server configuration if KMIP is set

If ECDSA Quote verification response needs to be signed, please set SIGN_QUOTE_RESPONSE=true in /root/binaries/env/sqvs.env

For ICX Production CPUs, copy sgx-verification-service/dist/linux/rootca_icx_prod.pem as trusted_rootca.pem in /root/binaries/env diretory

```shell
./install_enterprise_skc.sh
```

#### Deploy CSP SKC Services
On Deployment System
```shell
cd /root/binaries/
```

Update csp_skc.conf

  - CSP system IP Address
  - SAN List (a list of ip address and hostname for the CSP system)
  - Network Port numbers for CMS, AAS, SCS and SHVS
  - Install Admin and CSP Admin credentials
  - TENANT as KUBERNETES or OPENSTACK (based on the orchestrator chosen)
  - System IP address where Kubernetes or Openstack is deployed
  - Network Port Number of Kubernetes or Openstack Keystone/Placement Service
  - Database name, Database username and password for AAS, SCS and SHVS services
  - Intel PCS Server API URL and API Keys (Refer config file for instructions on getting a API Key)  

```shell
./install_csp_skc.sh
```

#### Deploying Services for Secure Key Caching Usecase
On Deployment System
```shell
cd /root/binaries/
```

Update enterprise_skc.conf

  - Deployment System IP address
  - SAN List (Subject Alternative Name List contains list of all domain names for the deployment system)
  - Network Port numbers for CMS, AAS, SCS, SQVS and KBS (Default Ports can be specified as mentioned in config file)
  - INSTALL_ADMIN_USERNAME and INSTALL_ADMIN_PASSWORD (Admin Credentials for installation of services)
  - CCC_ADMIN_USERNAME and CCC_ADMIN_PASSWORD (User credentials for generating long-lived tokens)
  - Database name, Database username and password for AAS and SCS services
  - Intel PCS Server API URL and API Keys (Refer config file for instructions on getting a API Key)  
  - Key Manager (can be set to either Directory or KMIP)
  - KMIP server configuration if KMIP is set

If ECDSA Quote verification response needs to be signed, please set SIGN_QUOTE_RESPONSE=true in /root/binaries/env/sqvs.env

For ICX Production CPUs, copy sgx-verification-service/dist/linux/rootca_icx_prod.pem as trusted_rootca.pem in /root/binaries/env diretory


```shell
./install_enterprise_skc.sh
```

#### Deploying Services for Orchestration Usecase
On Deployment System
```shell
cd /root/binaries/
```

Update orchestrator.conf
  
  - Deployment System IP address
  - SAN List (Subject Alternative Name List contains list of all domain names for the deployment system)
  - Network Port numbers for CMS, AAS, SCS, SQVS and KBS (Default Ports can be specified as mentioned in config file)
  - INSTALL_ADMIN_USERNAME and INSTALL_ADMIN_PASSWORD (Admin Credentials for installation of services)
  - CCC_ADMIN_USERNAME and CCC_ADMIN_PASSWORD (User credentials for generating long-lived tokens)
  - Database name, Database username and password for SHVS
  - TENANT as KUBERNETES or OPENSTACK (based on the orchestrator chosen)
  - System IP address where Kubernetes or Openstack is deployed
  - Network Port Number of Kubernetes or Openstack Keystone/Placement Service
  
Update enterprise_skc.conf
  - Deployment system IP address
  - SAN List (Subject Alternative Name List contains list of all domain names for the deployment system)
  - Network Port numbers for CMS, AAS, SCS and SQVS
  - INSTALL_ADMIN_USERNAME and INSTALL_ADMIN_PASSWORD (Admin Credentials for installation of services)
  - CCC_ADMIN_USERNAME and CCC_ADMIN_PASSWORD (User credentials for generating long-lived tokens)
  - Database name, Database username and password for AAS and SCS services
  - Intel PCS Server API URL and API Keys (Refer config file for instructions on getting a API Key)  

```shell
./install_orchestrator.sh
```

#### Deploying Services for All Usecases
On Deployment System
```shell
cd /root/binaries/
```

Update orchestrator.conf

  - Deployment System IP address
  - SAN List (Subject Alternative Name List contains list of all domain names for the deployment system)
  - Network Port numbers for CMS, AAS, SCS, SQVS and KBS (Default Ports can be specified as mentioned in config file)
  - INSTALL_ADMIN_USERNAME and INSTALL_ADMIN_PASSWORD (Admin Credentials for installation of services)
  - CCC_ADMIN_USERNAME and CCC_ADMIN_PASSWORD (User credentials for generating long-lived tokens)
  - Database name, Database username and password for SHVS
  - TENANT as KUBERNETES or OPENSTACK (based on the orchestrator chosen)
  - System IP address where Kubernetes or Openstack is deployed
  - Network Port Number of Kubernetes or Openstack Keystone/Placement Service

Update enterprise_skc.conf
  - Deployment System IP address
  - SAN List (Subject Alternative Name List contains list of all domain names for the deployment system)
  - Network Port numbers for CMS, AAS, SCS, SQVS and KBS (Default Ports can be specified as mentioned in config file)
  - INSTALL_ADMIN_USERNAME and INSTALL_ADMIN_PASSWORD (Admin Credentials for installation of services)
  - CCC_ADMIN_USERNAME and CCC_ADMIN_PASSWORD (User credentials for generating long-lived tokens)
  - Database name, Database username and password for AAS and SCS services
  - Intel PCS Server API URL and API Keys (Refer config file for instructions on getting a API Key)  
  - Key Manager (can be set to either Directory or KMIP)
  - KMIP server configuration if KMIP is set

```shell
./install_skc.sh
```

#### Deploy SGX Agent
Copy sgx_agent.tar, sgx_agent.sha2 and agent_untar.sh from binaries directoy to a directory in SGX compute node
```shell
./agent_untar.sh
```
Update agent.conf
> Note:  In case orchestration support is not needed, please comment/delete SHVS_IP in agent.conf available in same folder
  - CSP system IP address where CMS, AAS, SHVS and SCS services deployed
  - CSP Admin credentials (same which are provided in service configuration file. for ex: csp_skc.conf, orchestrator.conf or skc.conf)
  - Network Port numbers for CMS, AAS, SCS and SHVS
  - Token validity period in days
  - CMS TLS SHA Value (Run "cms tlscertsha384" on CSP system)

```shell
./deploy_sgx_agent.sh
```

#### Deploy SKC Library
Copy skc_library.tar, skc_library.sha2 and skclib_untar.sh from binaries directoy to a directory in SGX compute node
```shell
./skclib_untar.sh
```
Update create_roles.conf
  - IP address of AAS deployed on Enterprise system
  - Admin account credentials of AAS deployed on Enterprise system. These credentials should match with the AAS admin credentials provided in authservice.env on enterprise side.
  - For Each SKC Library installation on a SGX compute node, please change SKC_USER and SKC_USER_PASSWORD

```shell
./skc_library_create_roles.sh
```
Copy the token printed on console.

Update skc_library.conf
  - IP address for CMS and KBS services deployed on Enterprise network
  - CSP_CMS_IP should point to the IP of CMS service deployed on CSP network
  - CSP_SCS_IP should point to the IP of SCS service deployed on CSP network
  - Hostname of the Enterprise system where KBS is deployed
  - Network Port numbers for CMS and SCS services deployed on CSP system
  - Network Port numbers for CMS and KBS services deployed on Enterprise system
  - For Each SKC Library installation on a SGX compute node, please change SKC_USER (should be same as SKC_USER provided in create_roles.conf)
  - SKC_TOKEN with the token copied from previous step

```shell
./deploy_skc_library.sh
```

>> Note: For deploying SKC Library as a container refer [Deploy SKC Library as a container](#deploying-skc-library-as-a-container) section in Appendix

## **Deployment & Usecase Workflow Tools Installation**

The below installation is required on the Build & Deployment system only and the Platform(Windows,Linux or MacOS) for Usecase Workflow Tool Installation

**Deployment Tools Installation**

### Usecases Workflow Tools Installation

* Postman client should be [downloaded](https://www.postman.com/downloads/) on supported platforms or on the web to get started with the usecase collections.

  > **Note**: The Postman API Network will always have the latest released version of the API Collections. For all releases, refer the github repository for [API Collections](https://github.com/intel-secl/utils/tree/v4.1/develop/tools/api-collections)


## **Deployment Using Ansible**

The below details would enable the deployment through Ansible Role for Intel® SecL-DC Secure Key Caching Usecase. However the services can still be installed manually using the Product Guide. More details on Ansible Role for Intel® SecL-DC in [Ansible-Role](https://github.com/intel-secl/utils/tree/v4.1/develop/tools/ansible-role) repository.

### Pre-requisites

* The Ansible Server is required to use this role to deploy Intel® SecL-DC services based on the supported deployment model. The Ansible server is recommended to be installed on the Build machine itself. 
* The role has been tested with Ansible Version `2.9.10`

### Installing Ansible on Build Machine

  ```shell
  pip3 install ansible==2.9.10
  ```

* Install `epel-release` repository and install `sshpass` for ansible to connect to remote hosts using SSH

On Red Hat Enterprise Linux 8.2/8.4:
  ```shell
  dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
  dnf install sshpass
  ```
  
On Ubuntu 18.04/20.04:
  ```shell
  apt-get install sshpass
  ```

* Create directory for ansible default configuration and hosts file

  ```shell
  mkdir -p /etc/ansible/
  touch /etc/ansible/ansible.cfg
  ```

* Copy the `ansible.cfg` contents from https://raw.githubusercontent.com/ansible/ansible/v2.9.10/examples/ansible.cfg and paste it under `/etc/ansible/ansible.cfg`


### Download the Ansible Role 

The role can be cloned locally from git and the contents can be copied to the roles folder used by your ansible server 

#Create directory for using ansible deployment
```shell
mkdir -p /root/intel-secl/deploy/
```

#Clone the repository
```shell
cd /root/intel-secl/deploy/ && git clone https://github.com/intel-secl/utils.git
```

#Checkout to specific release version
```shell
cd utils/
git checkout <release-version of choice>
cd tools/ansible-role
```

#Update ansible.cfg roles_path to point to path(/root/intel-secl/deploy/utils/tools/ansible-role/roles/)


### Usecase Setup Options

| Usecase                      | Variable                                                     |
| ---------------------------- | ------------------------------------------------------------ |
| Secure Key Caching           | `setup: secure-key-caching` in playbook or via `--extra-vars` as `setup=secure-key-caching`in CLI |
| SGX Orchestration Kubernetes | `setup: sgx-orchestration-kubernetes` in playbook or via `--extra-vars` as `setup=sgx-orchestration-kubernetes`in CLI |
| SGX Attestation Kubernetes   | `setup: sgx-attestation-kubernetes` in playbook or via `--extra-vars` as `setup=sgx-attestation-kubernetes`in CLI |
| SGX Orchestration Openstack  | `setup: sgx-orchestration-openstack` in playbook or via `--extra-vars` as `setup=sgx-orchestration-openstack`in CLI |
| SGX Attestation Openstack    | `setup: sgx-attestation-openstack` in playbook or via `--extra-vars` as `setup=sgx-attestation-openstack`in CLI |
| SKC No Orchestration         | `setup: skc-no-orchestration` in playbook or via `--extra-vars` as `setup=skc-no-orchestration`in CLI |
| SGX Attestation No Orchestration | `setup: sgx-attestation-no-orchestration` in playbook or via `--extra-vars` as `setup=sgx-attestation-no-orchestration`in CLI |


> **Note:**  Orchestrator installation is not bundled with the role and need to be done independently. Also, components dependent on the orchestrator like `isecl-k8s-extensions` and `integration-hub` are installed either partially or not installed


### Update Ansible Inventory

The following inventory can be used and created under `/etc/ansible/hosts`

On RHEL 8.2/8.4:
```
[CSP]
<machine1_ip/hostname>

[Enterprise]
<machine2_ip/hostname>

[Node]
<machine3_ip/hostname>

[CSP:vars]
isecl_role=csp
ansible_user=root
ansible_password=<password>

[Enterprise:vars]
isecl_role=enterprise
ansible_user=root
ansible_password=<password>

[Node:vars]
isecl_role=node
ansible_user=root
ansible_password=<password>
```

On Ubuntu 18.04/20.04:
```
[CSP]
<machine1_ip/hostname>

[Enterprise]
<machine2_ip/hostname>

[Node]
<machine3_ip/hostname>

[CSP:vars]
isecl_role=csp
ansible_user=<ubuntu_user>
ansible_sudo_pass=<password>
ansible_password=<password>

[Enterprise:vars]
isecl_role=enterprise
ansible_user=<ubuntu_user>
ansible_sudo_pass=<password>
ansible_password=<password>

[Node:vars]
isecl_role=node
ansible_user=<ubuntu_user>
ansible_sudo_pass=<password>
ansible_password=<password>
```

> **Note:** Ansible requires `ssh` and `root` user access to remote machines. The following command can be used to ensure ansible can connect to remote machines with host key check `
  ```shell
  ssh-keyscan -H <ip_address> >> /root/.ssh/known_hosts
  ```

### Create and Run Playbook

The following are playbook and CLI example for deploying Intel® SecL-DC binaries based on the supported deployment models and usecases. The below example playbooks can be created as `site-bin-isecl.yml`

> **Note:** If running behind a proxy, update the proxy variables under `<path to ansible role>/ansible-role/vars/main.yml` and run as below

**Option 1**

Update the PCS Server key with following vars in `<path to ansible role>/ansible-role/defaults/main.yml`

```yaml
  intel_provisioning_server_api_key_sandbox: <pcs server key>
```

Create playbook with following contents

```yaml
- hosts: all
  gather_facts: yes
  any_errors_fatal: true
  vars:
    setup: <setup var from supported usecases>
    binaries_path: <path where built binaries are copied to>
    backend_pykmip: "<yes/no to install pykmip server along with KMIP KBS>"
  roles:   
  - ansible-role
  environment:
    http_proxy: "{{http_proxy}}"
    https_proxy: "{{https_proxy}}"
    no_proxy: "{{no_proxy}}"
```

and

```shell
ansible-playbook <playbook-name>
```

OR


**Option 2:**

Create playbook with following contents

```yaml
- hosts: all
  gather_facts: yes
  any_errors_fatal: true
  roles:   
  - ansible-role
  environment:
    http_proxy: "{{http_proxy}}"
    https_proxy: "{{https_proxy}}"
    no_proxy: "{{no_proxy}}"
```

and

```shell
ansible-playbook <playbook-name> --extra-vars setup=<setup var from supported usecases> --extra-vars binaries_path=<path where built binaries are copied to> --extra-vars intel_provisioning_server_api_key=<pcs server key> --extra-vars backend_pykmip=yes
```

> **Note:** If any service installation fails due to any misconfiguration, just uninstall the specific service manually , fix the misconfiguration in ansible and rerun the playbook. The successfully installed services won't be reinstalled.


#### Setup K8S Cluster and Deploy Isecl-k8s-extensions

Note: For Stack based deployment, setup master and worker node for k8s is part of Linux Stacks for SGX deployment.

* Setup master and worker node for k8s. Worker node should be setup on SGX enabled host machine. Master node can be any system.

* To setup k8 cluster for RHEL, follow https://phoenixnap.com/kb/how-to-install-kubernetes-on-centos.

* To setup k8 cluster for Ubuntu, follow https://phoenixnap.com/kb/install-kubernetes-on-ubuntu 

Once the master/worker setup is done, follow below steps on Master Node:

##### Untar packages and push OCI images to registry

* Copy tar output isecl-k8s-extensions-*.tar.gz from build system's binaries folder to /opt/ directory on the Control-plane Node and extract the contents.
  
  ```shell
    cd /opt/
    tar -xvzf isecl-k8s-extensions-*.tar.gz
    cd isecl-k8s-extensions/
  ```
* Configure private registry

* Push images to private registry using skopeo command, (this can be done from build vm also)
  
  ```shell
     skopeo copy oci-archive:isecl-k8s-controller-v5.0.0-<commitid>.tar docker://<registryIP>:<registryPort>/isecl-k8s-controller:v5.0.0
     skopeo copy oci-archive:isecl-k8s-scheduler-v5.0.0-<commitid>.tar docker://<registryIP>:<registryPort>/isecl-k8s-scheduler:v5.0.0
  ```
  
* Add the image names in isecl-controller.yml and isecl-scheduler.yml in /opt/isecl-k8s-extensions/yamls with full image name including registry IP/hostname (e.g <registryIP>:<registryPort>/isecl-k8s-scheduler:v5.0.0). It will automatically pull the images from registry.

##### Deploy isecl-controller
* Create hostattributes.crd.isecl.intel.com crd
```
    kubectl apply -f yamls/crd-1.17.yaml
```
* Check whether the crd is created
```
    kubectl get crds
```
* Deploy isecl-controller
```
    kubectl apply -f yamls/isecl-controller.yaml
```
* Check whether the isecl-controller is up and running
```
    kubectl get deploy -n isecl
```
* Create clusterrolebinding for ihub to get access to cluster nodes
```
    kubectl create clusterrolebinding isecl-clusterrole --clusterrole=system:node --user=system:serviceaccount:isecl:isecl
```
* Fetch token required for ihub installation and follow below IHUB installation steps,
```
    kubectl get secrets -n isecl
    kubectl describe secret default-token-<name> -n isecl
```

For IHUB installation, make sure to update below configuration in /root/binaries/env/ihub.env before installing ihub on CSP system:
* Copy /etc/kubernetes/pki/apiserver.crt from master node to /root on CSP system. Update KUBERNETES_CERT_FILE.
* Get k8s token in master, using above commands and update KUBERNETES_TOKEN
* Update the value of CRD name
```
	KUBERNETES_CRD=custom-isecl-sgx
```

##### Deploy isecl-scheduler
* The isecl-scheduler default configuration is provided for common cluster support in /opt/isecl-k8s-extensions/yamls/isecl-scheduler.yaml. Variables HVS_IHUB_PUBLIC_KEY_PATH and SGX_IHUB_PUBLIC_KEY_PATH are by default set to default paths. Please use and set only required variables based on the use case. 
For example, if only sgx based attestation is required then remove/comment HVS_IHUB_PUBLIC_KEY_PATH variables.

* Install cfssl and cfssljson on Kubernetes Control Plane
```
    #Download cfssl to /usr/local/bin/
    wget -O /usr/local/bin/cfssl http://pkg.cfssl.org/R1.2/cfssl_linux-amd64
    chmod +x /usr/local/bin/cfssl

    #Download cfssljson to /usr/local/bin
    wget -O /usr/local/bin/cfssljson http://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
    chmod +x /usr/local/bin/cfssljson
```

* Create tls key pair for isecl-scheduler service, which is signed by k8s apiserver.crt
```
    cd /opt/isecl-k8s-extensions/
    chmod +x create_k8s_extsched_cert.sh
    ./create_k8s_extsched_cert.sh -n "K8S Extended Scheduler" -s "<K8_MASTER_IP>","<K8_MASTER_HOST>" -c /etc/kubernetes/pki/ca.crt -k /etc/kubernetes/pki/ca.key
```
* After iHub deployment, copy /etc/ihub/ihub_public_key.pem from ihub to /opt/isecl-k8s-extensions/ directory on k8 master system. Also, copy tls key pair generated in previous step to secrets directory.
```
    mkdir secrets
    cp /opt/isecl-k8s-extensions/server.key secrets/
    cp /opt/isecl-k8s-extensions/server.crt secrets/
    mv /opt/isecl-k8s-extensions/ihub_public_key.pem /opt/isecl-k8s-extensions/sgx_ihub_public_key.pem
    cp /opt/isecl-k8s-extensions/sgx_ihub_public_key.pem secrets/
```
Note: Prefix the attestation type for ihub_public_key.pem before copying to secrets folder.
* Create kubernetes secrets scheduler-secret for isecl-scheduler
```
    kubectl create secret generic scheduler-certs --namespace isecl --from-file=secrets
```
* Deploy isecl-scheduler
```
    kubectl apply -f yamls/isecl-scheduler.yaml
```
* Check whether the isecl-scheduler is up and running
```
    kubectl get deploy -n isecl
```

##### Configure kube-scheduler to establish communication with isecl-scheduler
* Add scheduler-policy.json under kube-scheduler section, mountPath under container section and hostPath under volumes section in /etc/kubernetes/manifests/kube-scheduler.yaml as mentioned below
```
spec:
  containers:
  - command:
    - kube-scheduler
    - --policy-config-file=/opt/isecl-k8s-extensions/scheduler-policy.json
```

```
  containers:
    volumeMounts:
    - mountPath: /opt/isecl-k8s-extensions/
      name: extendedsched
      readOnly: true
```

```
  volumes:
  - hostPath:
      path: /opt/isecl-k8s-extensions/
      type:
    name: extendedsched
```

Note: Make sure to use proper indentation and don't delete existing mountPath and hostPath sections in kube-scheduler.yaml.
* Restart Kubelet which restart all the k8s services including kube base schedular
```
	systemctl restart kubelet
```

* Check if CRD data is populated
```
	kubectl get -o json hostattributes.crd.isecl.intel.com
```

##### Validate POD launch
Create sample yml file for nginx workload and add SGX labels to it such as:
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    name: nginx
spec:
  affinity:
    nodeAffinity:
     requiredDuringSchedulingIgnoredDuringExecution:
       nodeSelectorTerms:
       - matchExpressions:
         - key: SGX-Enabled
           operator: In
           values:
           - "true"
         - key: EPC-Memory
           operator: In
           values:
           - "2.0GB"
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```

Validate if pod can be launched on the node. Run following commands:

```
    kubectl apply -f pod.yml
    kubectl get pods
    kubectl describe pods nginx
```

Pod should be in running state and launched on the host as per values in pod.yml. Validate by running below command on sgx host:
```
	docker ps
```

#### Openstack Setup and Associate Traits

> **Note**: Openstack Support only validated with RHEL 8.1/8.2

* Setup Compute and Controller node for Openstack. Compute node should be setup on SGX host machine, Controller node can be any system. After the compute/controller setup is done, follow the below steps:

* IHUB should be installed and configured with Openstack

  > **Note**:
  * While using deployment scripts to install the components, in the env directory of the binaries folder comment "KUBERNETES_TOKEN" in the ihub.env before installation.
  * Openstack compute node and build VM should have the same OS package repositories, else there will be package mismatch for SKC library.
  
* On the openstack controller, if resource provider is not listing the resources then install the "osc-placement"
```
  pip3 install osc-placement
```
* source the admin-openrc credentials to gain access to user-only CLI commands and export the os_placement_API_version
```
   source admin-openrc
```
* List the set of resources mapped to the Openstack
```
  openstack resource provider list
```
* Set the required traits for SGX Hosts
```
  #For example 'cirros' image can be used for the instances
  openstack image set --property trait:CUSTOM_ISECL_SGX_ENABLED_TRUE=required <image name>
```
* View the Traits that has been set:
```
  #The trait should be set and assinged to the respective image successfully. For example 'cirros' image can be used for the instances 
   openstack image show <image name>
```
* Verify the trait is enabled for the SGX Host:
```
  openstack resource provider trait list <uuid of the host which the openstack resoruce provider lists>

  #SGX Supported, SGX TCB upto Date, SGX FLC enabled, SGX EPC size attritubes of the SGX host for which the 'required' trait set to TRUE or FALSE is displayed. For example,if required trait is set as TRUE:
  
  CUSTOM_ISECL_SGX_ENABLED_TRUE
  CUSTOM_ISECL_SGX_SUPPORTED_TRUE
  CUSTOM_ISECL_SGX_TCBUPTODATE_FALSE
  CUSTOM_ISECL_SGX_FLC_ENABLED_TRUE
  CUSTOM_ISECL_SGX_EPC_SIZE_2_0_GB

  For example, if the required trait is set as FALSE
  CUSTOM_ISECL_SGX_ENABLED_FALSE
  CUSTOM_ISECL_SGX_SUPPORTED_TRUE
  CUSTOM_ISECL_SGX_TCBUPTODATE_FALSE
  CUSTOM_ISECL_SGX_FLC_ENABLED_FALSE
  CUSTOM_ISECL_SGX_EPC_SIZE_0_B
```
* Create the instances
```
  openstack server create --flavor tiny --image <image name> --net vmnet <vm instance name>

  Instances should be created and the status should be "Active". Instance should be launched successfully.
  openstack server list
```
 > **Note** : To unset the trait, use the following CLI commands:
```
 openstack image unset --property trait:CUSTOM_ISECL_SGX_ENABLED_TRUE <image name>

 openstack image unset --property trait:CUSTOM_ISECL_SGX_ENABLED_FALSE <image name>
```


## **7. Usecase Workflows API Collections**

The below allow to get started with workflows within Intel® SecL-DC for Foundational and Workload Security Usecases. More details available in [API Collections](https://github.com/intel-secl/utils/tree/v4.1/develop/tools/api-collections) repository

### Pre-requisites

* Postman client should be [downloaded](https://www.postman.com/downloads/) on supported platforms or on the web to get started with the usecase collections.

  >  **Note:** The Postman API Network will always have the latest released version of the API Collections. For all releases, refer the github repository for [API Collections](https://github.com/intel-secl/utils/tree/v4.1/develop/tools/api-collections)

### Use Case Collections

| Use case                     | Sub-Usecase | API Collection     |
| ---------------------------- | ----------- | ------------------ |
| Secure Key Caching           | -           | ✔️                  |
| SGX Discovery, Provisioning and Orchestration | -           | ✔️                  |
| SGX Discovery and Provisioning           | -           | ✔️                  |


### Downloading API Collections

* Postman API Network for latest released collections: https://explore.postman.com/intelsecldc

  or 

* Github repo for allreleases

  ```shell
  git clone https://github.com/intel-secl/utils.git
  cd utils/
  git checkout <release-version of choice>
  cd tools/api-collections
  ```

>  **Note**:  The postman-collections are also available when cloning the repos via build manifest under `utils/tools/api-collections`


### Running API Collections

* Import the collection into Postman API Client

  > Note**: This step is required only when not using Postman API Network and downloading from Github

  ![importing-collection](./images/importing_collection.gif)

* Update env as per the deployment details for specific usecase

  ![updating-env](./images/updating_env.gif)

* View Documentation

  ![view-docs](./images/view_documentation.gif)

* Run the workflow

  ![running-collection](./images/running_collection.gif)


## **8. System User Configuration**

**Build System**

**Setup ~/.gitconfig to update the git user details. A sample config is provided below**

GIT Configuration**

```
[user]
        name = John Doe
        email = john.doe@abc.com
[color]
        ui = auto
 [push]
        default = matching +
```

* Make sure system date and time of SGX machine and CSP machine both are in sync. Also, if the system is configured to read the RTC time in the local time zone, then use RTC in UTC by running `timedatectl set-local-rtc 0` command on both the machine. Otherwise SGX Agent deployment will fail with certificate expiry error.

## **9. Appendix**

###  SGX Attestation flow

```
To Deploy SampleApp:
  Copy sample_apps.tar, sample_apps.sha2 and sampleapps_untar.sh from binaries directory to a directory in SGX compute node and untar it using './sample_apps_untar.sh'
  Install Intel® SGX SDK for Linux*OS into /opt/intel/sgxsdk using './install_sgxsdk.sh'
  Install SGX dependencies using './deploy_sgx_dependencies.sh'
Note: Make sure to deploy SQVS with includetoken configuration as false. 

To Verify the SampleApp flow:
  Update sample_apps.conf with the following
   - IP address for SQVS services deployed on Enterprise system
   - IP address for SCS services deployed on CSP system
   - ENTERPRISE_CMS_IP should point to the IP of CMS service deployed on Enterprise system
   - Network Port numbers for SCS services deployed on CSP system
   - Network Port numbers for SQVS and CMS services deployed on Enterprise system
   - Set RUN_ATTESTING_APP to yes if user wants to run both apps in same machine
  Run SampleApp using './run_sample_apps.sh'
  Check the output of attestedApp and attestingApp under out/attested_app_console_out.log and out/attesting_app_console_out.log files
 
```

### Creating RSA Keys in Key Broker Service

**Steps to run KMIP Server**

> **Note**: Below mentioned steps are provided as script (install_pykmip.sh and pykmip.service) as part of kbs_script folder which will install KMIP Server as daemon.

1. Install python3 and vim-common
```shell
   dnf -y install python3-pip vim-common
   ln -s /usr/bin/python3 /usr/bin/python  > /dev/null 2>&1
   ln -s /usr/bin/pip3 /usr/bin/pip  > /dev/null 2>&1
```

2. Install pykmip
```shell
   pip3 install pykmip==0.9.1
```

3. In the /etc/ directory create pykmip and policies folders
```shell
   mkdir -p /etc/pykmip/policies
```

4. Configure pykmip server using server.conf
   Update hostname in the server.conf

5. Copy the following to /etc/pykmip/ from kbs_script folder available under binaries directory
   create_certificates.py, run_server.py, server.conf

6. Create certificates
```shell
   cd /etc/pykmip
   python3 create_certificates.py <KMIP Host IP/KMIP Host FQDN>
```

7. Kill running KMIP Server processes and wait for 10 seconds until all the KMIP Server processes are killed. 
```shell
   ps -ef | grep run_server.py | grep -v grep | awk '{print $2}' | xargs kill
```

8. Run pykmip server using run_server.py script
```shell
   python3 run_server.py &
```

**Install KMIP Server as daemon**

```
1. cd into /root/binaries/kbs_script folder

2. Configure pykmip server using server.conf
   Update hostname in the server.conf

3. Run the install_pykmip.sh script and KMIP server will be installed as daemon process
   ./install_pykmip.sh
```

Update KMIP_IP in /root/binaries/kbs_script/rsa_create.py

**Configuration Update to create Keys in KBS**
```shell
cd /root/binaries/kbs_script
```

To register keys with KBS KMIP

Update the following variables in kbs.conf:

```    
Enterprise system IP address where CMS, AAS and KBS services are deployed
        
Port of CMS, AAS and KBS services deployed on enterprise system
    
AAS admin and Enterprise admin credentials
```        

Update sgx_enclave_measurement_anyof value in transfer_policy_request.json with enclave measurement value obtained using sgx_sign utility. Refer to "Extracting SGX Enclave values for Key Transfer Policy" section.

**Create RSA Key**

```shell
   ./run.sh reg
```

Copy the generated cert file to SGX Compute node where skc_library is deployed. Also make a note of the key id generated.

### Configuration for NGINX testing

**Note:** Below mentioned OpenSSL and NGINX configuration updates are provided as patches (nginx.patch and openssl.patch) as part of skc_library deployment script. Patch can be applied with default nginx and openssl file. In case nginx/openssl contains any external changes then refer manual step.

**Apply Patch (Rhel)**

  Execute the command with nginx version - nginx 1.14.1 and openssl version- Openssl 1.1.1g

```shell
patch -b /etc/nginx/nginx.conf < nginx.patch
patch -b /etc/pki/tls/openssl.cnf < openssl.patch
```
**Apply Patch (Ubuntu)**

  Execute the command with nginx version - nginx 1.14.0 and openssl version- Openssl 1.1.1

```shell
patch -b /etc/nginx/nginx.conf < nginx_ubuntu.patch
patch -b /etc/ssl/openssl.cnf < openssl_ubuntu.patch
```

OpenSSL Patch adds following section in openssl.cnf

**OpenSSL (Rhel)**

```shell
[openssl_def]
engines = engine_section

[engine_section]
pkcs11 = pkcs11_section

[pkcs11_section]
engine_id = pkcs11

dynamic_path =/usr/lib64/engines-1.1/pkcs11.so

MODULE_PATH =/opt/skc/lib/libpkcs11-api.so

init = 0
```

**OpenSSL (Ubuntu)**

```shell
openssl_conf = openssl_def

[openssl_def]
engines = engine_section
oid_section = new_oids

[engine_section]
pkcs11 = pkcs11_section

[pkcs11_section]
engine_id = pkcs11
dynamic_path =/usr/lib/x86_64-linux-gnu/engines-1.1/pkcs11.so
MODULE_PATH =/opt/skc/lib/libpkcs11-api.so
init = 0
```

**Nginx**

Update nginx configuration file /etc/nginx/nginx.conf with below changes:
```shell
ssl_engine pkcs11;
```

Update the location of certificate with the loaction where it was copied into the skc_library machine. 
```shell
ssl_certificate "add absolute path of crt file";
```

Update the fields(token, object and pin-value) with the values given in keys.txt for the KeyID corresponding to the certificate.
```shell
ssl_certificate_key "engine:pkcs11:pkcs11:token=KMS;object=RSAKEY;pin-value=1234";
```

**SKC Configuration**

 Create keys.txt in /root folder. This provides key preloading functionality in skc_library.

  Any number of keys can be added in keys.txt. Key IDs are specified in PKCS11 URL format. Each PKCS11 URL should contain different Key ID, along with respective object tag

  Sample PKCS11 url is as below
 ``` 
  pkcs11:token=KMS;id=164b41ae-be61-4c7c-a027-4a2ab1e5e4c4;object=RSAKEY;type=private;pin-value=1234;
 ```

  Token, object and pin-value given in PKCS11 url entry in keys.txt should match with the one in nginx.conf.

  The keyID should match the keyID of RSA key created in KBS. File location should match with preload_keys directive in pkcs11-apimodule.ini; 

  Sample /opt/skc/etc/pkcs11-apimodule.ini file

```shell	
[core]
preload_keys=/root/keys.txt
keyagent_conf=/opt/skc/etc/key-agent.ini
mode=SGX
debug=false
	
[SGX]
module=/opt/intel/cryptoapitoolkit/lib/libp11sgx.so
```

### KBS key-transfer flow validation

On SGX Compute node, Execute below commands for KBS key-transfer:


> **Note**: Before initiating key transfer make sure, PYKMIP server is running.

```shell
    pkill nginx
```

Remove any existing pkcs11 token

```shell
    rm -rf /opt/intel/cryptoapitoolkit/tokens/*
```

Install Nginx and add Group (Rhel)

```
    yum install -y nginx
    groupadd intel
    usermod -G intel nginx

```

Install Nginx and add Group (Ubuntu)

```
    apt install -y nginx
    groupadd intel
    usermod -G intel www-data

```

Initiate Key transfer from KBS

```shell
    systemctl restart nginx
```

Changing group ownership and permissions of pkcs11 token

```shell
    chown -R root:intel /opt/intel/cryptoapitoolkit/tokens/
    chmod -R 770 /opt/intel/cryptoapitoolkit/tokens/
```

Establish a tls session with the nginx using the key transferred inside the enclave

```shell
    wget https://localhost:2443 --no-check-certificate
```

### Note on Key Transfer Policy

Key transfer policy is used to enforce a set of policies which need to be compiled with before the secret can be securely provisioned onto a sgx enclave

A typical Key Transfer Policy would look as below
```
        "sgx_enclave_issuer_anyof":["83d719e77deaca1470f6baf62a4d774303c899db69020f9c70ee1dfc08c7ce9e"],
        "sgx_enclave_issuer_product_id":0,
        "sgx_enclave_measurement_anyof":["ad46749ed41ebaa2327252041ee746d3791a9f2431830fee0883f7993caf316a"],
        "sgx_enclave_svn_minimum":1,
        "tls_client_certificate_issuer_cn_anyof":["CMSCA", "CMS TLS Client CA"],
        "client_permissions_allof":["nginx","USA"],
        "attestation_type_anyof":["SGX","SW"],
        "sgx_enforce_tcb_up_to_date":false
```
**sgx_enclave_issuer_anyof** establishes the signing identity provided by an authority who has signed the sgx enclave. in other words the owner of the enclave

**sgx_enclave_measurement_anyof** represents the cryptographic hash of the enclave log (enclave code, data)

**sgx_enforce_tcb_up_to_date** - If set to true, Key Broker service will provision the key only of the platform generating the quote conforms to the latest Trusted Computing Base

**client_permissions_allof** - Special permission embedded into the skc_library client TLS certificate which can enforce additional restrictons on who can get access to the key,
    In above example: the key is provisioned only to the nginx workload and platform which is tagged with value for ex: USA

> **Note**: sgx_enclave_issuer_anyof and sgx_enclave_issuer_product_id are mandatory fields which need to be specified in a Key Transfer policy

### Note on SKC Library Deployment

SKC Library Deployment (Binary as well as container) needs to performed with root privilege

For binary deployment of SKC client Library, only one instance of Workload can use SKC Client Library. The config information for SKC client library is bound to the workload.
In future, Multiple workloads might be supported For container deployment, since configmaps are used, each container instance of workload gets its own private SKC Client Library config information

The SKC Client Library TLS client certificate private key is stored in the configuration directories and can be read only with elevated root privileges
keys.txt (set of PKCS11 URIs for the keys to be securely provisioned into an SGX enclave) can only be modified with elevated privileges

### Deploy SKC Library as a Container 
```
Use the following steps to configure SKC library running in a container and to validate key transfer in container on bare metal and inside a VM on SGX enabled hosts.

Note: All the configuration files required for SKC Library container are modified in the resources directory only 

1. Docker should be installed, enabled and services should be active
2.To get the SKC library tar file, run "make skc_library_k8s" for distribution based deployment or run "make skc_library_k8s_stacks" for stack based deployment.
  In the build System, SKC Library tar file "<skc-lib*>.tar" required to load is located in the "/root/workspace/skc_library" directory.  
3. Copy "resources" folder from "workspace/skc_library/container/resources" to the "/root/" directory of SGX host. Inside the resources folder all the key transfer flow related files will be available.
4. Update sgx_default_qcnl.conf file inside resources folder with SCS IP and SCS port and also update the kms_npm.ini with KBS IP and KBS PORT and update hosts file present in same folder with KBS IP and hostname.
5. To create user and role for skc library, update the create_roles.conf, and run ./skc_library_create_roles.sh, which is inside the resources folder.
6. Generate the RSA key in the kbs host and copy the generated KBS certificate to SGX host under /root/.
7. Refer to openssl and nginx sub sections of Quick Start Guide in the "Configuration for NGINX testing" to configure nginx.conf and openssl.conf files which are under resource directory.
8. Update keyID in the keys.txt and nginx.conf. 
9. Under [core] section of pkcs11-apimodule.ini in the "/root/resources/" directory add preload_keys=/root/keys.txt.
10. Update skc_library.conf with IP addresses where SKC services are deployed.
11. On the SGX Compute node, load the skc library docker image provided in the tar file. 
   docker load < <SKC_Library>.tar
12. Provide valid paramenets in the docker run command and execute the docker run command. Update the genertaed RSA Key ID and <keys>.crt in the resources directory.
    docker run -p 8080:2443 -p 80:8080 --mount type=bind,source=/root/<KBS_cert>.crt,target=/root/<KBS_cert>.crt --mount type=bind,source=/root/resources/sgx_default_qcnl.conf,target=/etc/sgx_default_qcnl.conf --mount type=bind,source=/root/resources/nginx.conf,target=/etc/nginx/nginx.conf --mount type=bind,source=/root/resources/keys.txt,target=/root/keys.txt,readonly --mount type=bind,source=/root/resources/pkcs11-apimodule.ini,target=/opt/skc/etc/pkcs11-apimodule.ini,readonly --mount type=bind,source=/root/resources/kms_npm.ini,target=/opt/skc/etc/kms_npm.ini,readonly --mount type=bind,source=/root/resources/sgx_stm.ini,target=/opt/skc/etc/sgx_stm.ini,readonly --mount type=bind,source=/root/resources/openssl.cnf,target=/etc/pki/tls/openssl.cnf --mount type=bind,source=/root/resources/skc_library.conf,target=/skc_library.conf --add-host=<SGX_HOSTNAME>:<SGX_HOST_IP> --add-host=<KBS_Hostname>:<KBS host IP> --mount type=bind,source=/dev/sgx,target=/dev/sgx --cap-add=SYS_MODULE --privileged=true <SKC_LIBRARY_IMAGE_NAME>
    
    Note: In the above docker run command, source refers to the actual path of the files located on the host and the target always refers to the files which would be mounted inside the container
  
13. Establish a tls session with the nginx using the key transferred inside the enclave
    Get the container id using "docker ps" command
    docker exec -it <container_id> /bin/sh

    #Follow the steps only if proxy setup is required
    export http_proxy=http://<proxy-url>:<proxy-port>
    export https_proxy=http://<proxy-url>:<proxy-port>
    export no_proxy=0.0.0.0,127.0.0.1,localhost,<CSP IP>,<Enterprise IP>, <SGX Compute Node IP>, <KBS system Hostname>
    
    #Install wget
    dnf install wget
    wget https://localhost:2443 --no-check-certificate
```

### Extracting SGX Enclave values for Key Transfer Policy

Values that are specific to the enclave such as sgx_enclave_issuer_anyof, sgx_enclave_measurement_anyof and sgx_enclave_issuer_product_id_anyof can be retrived using `sgx_sign` utility that is available as part of Intel SGX SDK.

Run `sgx_sign` utility on the signed enclave (This command should be run on the build system).
```shell
    /opt/intel/sgxsdk/bin/x64/sgx_sign dump -enclave <path to the signed enclave> -dumpfile info.txt
```

- For `sgx_enclave_issuer_anyof`, in info.txt, search for "mrsigner->value" . E.g mrsigner->value :
  ```
  mrsigner->value: "0x83 0xd7 0x19 0xe7 0x7d 0xea 0xca 0x14 0x70 0xf6 0xba 0xf6 0x2a 0x4d 0x77 0x43 0x03 0xc8 0x99 0xdb 0x69 0x02 0x0f 0x9c 0x70 0xee 0x1d 0xfc 0x08 0xc7 0xce 0x9e"
  ```
  Remove the whitespace and 0x characters from the above string and add it to the policy file. E.g :
  ```
  "sgx_enclave_issuer_anyof":["83d719e77deaca1470f6baf62a4d774303c899db69020f9c70ee1dfc08c7ce9e"]
  ```
- For `sgx_enclave_measurement_anyof`, in info.txt, search for metadata->enclave_css.body.enclave_hash.m . E.g metadata->enclave_css.body.enclave_hash.m :
  ```
  metadata->enclave_css.body.enclave_hash.m:
  0xad 0x46 0x74 0x9e 0xd4 0x1e 0xba 0xa2 0x32 0x72 0x52 0x04 0x1e 0xe7 0x46 0xd3
  0x79 0x1a 0x9f 0x24 0x31 0x83 0x0f 0xee 0x08 0x83 0xf7 0x99 0x3c 0xaf 0x31 0x6a
  ```
  Remove the whitespace and 0x characters from the above string and add it to the policy file. E.g :
  ```
  "sgx_enclave_measurement_anyof":["ad46749ed41ebaa2327252041ee746d3791a9f2431830fee0883f7993caf316a"]
  ```
Please note that the SGX Enclave measurement value will depend on the toolchain used to build and link the SGX enclave. Hence the SGX Enclave measurement value would differ across OS flavours.
For more details please refer https://github.com/intel/linux-sgx/tree/master/linux/reproducibility
