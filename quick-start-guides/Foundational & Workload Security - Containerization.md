# Quick start Guide - Foundation & Workload Security - Containerization


Table of Contents
-----------------


<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Quick start Guide - Foundation & Workload Security - Containerization](#quick-start-guide---foundation--workload-security---containerization)
  - [Table of Contents](#table-of-contents)
  - [Hardware & OS Requirements](#hardware--os-requirements)
    - [Physical Server requirements](#physical-server-requirements)
    - [Machines](#machines)
    - [OS Requirements](#os-requirements)
    - [Container Runtime](#container-runtime)
    - [K8s Distributions](#k8s-distributions)
    - [Storage](#storage)
  - [Network Requirements](#network-requirements)
    - [Build System](#build-system)
    - [CSP Managed Services](#csp-managed-services)
    - [Enterprise Managed Services](#enterprise-managed-services)
    - [TXT/SUEFI Enabled Host](#txtsuefi-enabled-host)
    - [Firewall Settings](#firewall-settings)
  - [Rpms & Debs Requirement](#rpms--debs-requirement)
    - [RHEL](#rhel)
    - [Ubuntu](#ubuntu)
  - [Deployment Model](#deployment-model)
    - [Single Node](#single-node)
    - [Multi Node](#multi-node)
  - [Build](#build)
    - [Pre-requisites](#pre-requisites)
      - [Development Tools and Utilities](#development-tools-and-utilities)
      - [Repo tool](#repo-tool)
      - [Golang](#golang)
      - [Docker](#docker)
    - [Build OCI Container images and K8s Manifests](#build-oci-container-images-and-k8s-manifests)
      - [Foundational Security](#foundational-security)
      - [Workload Security](#workload-security)
        - [Container Confidentiality with CRIO Runtime](#container-confidentiality-with-crio-runtime)
  - [Deployment](#deployment)
    - [Pre-requisites](#pre-requisites-1)
    - [Deploy](#deploy)
      - [Single-Node](#single-node-1)
        - [Pre-requisites](#pre-requisites-2)
          - [Setup](#setup)
          - [Manifests](#manifests)
        - [Deploy steps](#deploy-steps)
          - [Update `isecl-k8s.env` file](#update-isecl-k8senv-file)
          - [Run scripts on K8s control-plane](#run-scripts-on-k8s-control-plane)
      - [Multi-Node](#multi-node-1)
        - [Pre-requisites](#pre-requisites-3)
          - [Setup](#setup-1)
          - [Manifests](#manifests-1)
        - [Deploy steps](#deploy-steps-1)
          - [Update `isecl-k8s.env` file](#update-isecl-k8senv-file-1)
          - [Run scripts on K8s control-plane](#run-scripts-on-k8s-control-plane-1)
  - [Default Service and Agent Mount Paths](#default-service-and-agent-mount-paths)
    - [Single Node Deployments](#single-node-deployments)
    - [Multi Node Deployments](#multi-node-deployments)
  - [Default Service Ports](#default-service-ports)
  - [Usecase Workflows API Collections](#usecase-workflows-api-collections)
    - [Pre-requisites](#pre-requisites-4)
    - [Use Case Collections](#use-case-collections)
    - [Downloading API Collections](#downloading-api-collections)
    - [Running API Collections](#running-api-collections)
  - [Appendix](#appendix)
    - [Hardware feature detection](#hardware-feature-detection)
    - [Running behind Proxy](#running-behind-proxy)
    - [Git Config Sample (~/.gitconfig)](#git-config-sample-gitconfig)
    - [Rebuilding Repos](#rebuilding-repos)
    - [Setup Task Flow](#setup-task-flow)
    - [Configuration Update Flow](#configuration-update-flow)
    - [Cleanup workflows](#cleanup-workflows)
      - [Single-node](#single-node-2)
      - [Multi-node](#multi-node-2)

<!-- /code_chunk_output -->


## Hardware & OS Requirements

### Physical Server requirements

* Intel® SecL-DC  supports and uses a variety of Intel security features, but there are some key requirements to consider before beginning an installation. Most important among these is the Root of Trust configuration. This involves deciding what combination of TXT, Boot Guard, tboot, and UEFI Secure Boot to enable on platforms that will be attested using Intel® SecL.

  > **Note:** At least one "Static Root of Trust" mechanism must be used (TXT and/or BtG). For Legacy BIOS systems, tboot must be used. For UEFI mode systems, UEFI SecureBoot must be used* Use the chart below for a guide to acceptable configuration options. Only dTPM is supported on Intel® SecL-DC platform hardware. 

  ![hardware-options](./images/trusted-boot-options.PNG)

### Machines

* Build Machine

* K8s control-plane Node Setup on CSP (VMs/Physical Nodes + TXT/SUEFI enabled Physical Nodes)

* K8s control-plane Node Setup on Enterprise (VMs/Physical Nodes)

### OS Requirements

* `RHEL 8.3`/`Ubuntu 18.04` for build

* `RHEL 8.3`/`Ubuntu 18.04` for K8s cluster deployments

  >  **Note:** Foundational & Workload Security solution is built, installed and tested with root privileges. Please ensure that all the following instructions are executed with root privileges

### Container Runtime

* Docker-20.10.8
* CRIO-1.17.5

### K8s Distributions

* Single Node: 

  A single-node deployment uses `microk8s` to deploy the entire control plane in a pod on a single hardware machine. This is best for POC or demo environments, but can also be used when integrating Intel SecL with another application that runs on a virtual machine

* Multi Node: 

  A multi-node deployment is a more typical Kubernetes architecture, where the Intel SecL management plane is simply deployed as a Pod, with the Intel SecL agents (the WLA and the TA, depending on use case) deployed as a DaemonSet. `kubeadm` is the supported multi node distribution as of today.

### Storage

* `hostPath` in case of single-node `microk8s` for all services and agents
* `NFS` in case of multi-node `kubeadm` for services and `hostPath` for Trust Agent and Workload Agent


## Network Requirements

### Build System

Internet access required

### CSP Managed Services

Internet access required for Foundation & Workload Security services deployed on CSP system/Compute Node;

### Enterprise Managed Services

Internet access required for Foundation & Workload Security services on Enterprise system;

### TXT/SUEFI Enabled Host

Internet access required to access KBS running on Enterprise environment

### Firewall Settings

Ensure that all the FS,WS service ports are accessible with firewall

## Rpms & Debs Requirement

### RHEL

* `rhel-8-for-x86_64-appstream-rpms`
* `rhel-8-for-x86_64-baseos-rpms`
* `codeready-builder-for-rhel-8-x86_64-rpms`
* `epel-release-latest-8.noarch.rpm`

### Ubuntu

None



## Deployment Model

### Single Node

The single Node uses `microk8s` as a supported K8s distribution

![k8s-single-node](./images/k8s-single-deploy-fsws.png)

### Multi Node

The multi node supports `kubeadm` as a supported K8s distribution

![K8s Deployment-fsws](./images/k8s-multi-deploy-fsws.png)

## Build

### Pre-requisites

The below steps need to be done on `RHEL 8.3`/`Ubuntu-18.04` Build machine (VM/Physical Node)

#### Development Tools and Utilities

```shell
# RedHat Enterprise Linux 8.3
dnf install -y git wget tar python3 gcc gcc-c++ zip make yum-utils openssl-devel
dnf install -y https://dl.fedoraproject.org/pub/fedora/linux/releases/32/Everything/x86_64/os/Packages/m/makeself-2.4.0-5.fc32.noarch.rpm
ln -s /usr/bin/python3 /usr/bin/python
ln -s /usr/bin/pip3 /usr/bin/pip

# Ubuntu-18.04
apt update
apt remove -y gcc gcc-7
apt install -y python3-problem-report git wget tar python3 gcc-8 make makeself openssl libssl-dev libgpg-error-dev
cp /usr/bin/gcc-8 /usr/bin/gcc
ln -s /usr/bin/python3 /usr/bin/python
ln -s /usr/bin/pip3 /usr/bin/pip
```

#### Repo tool

```shell
tmpdir=$(mktemp -d)
git clone https://gerrit.googlesource.com/git-repo $tmpdir
install -m 755 $tmpdir/repo /usr/local/bin
rm -rf $tmpdir
```

#### Golang

```shell
wget https://dl.google.com/go/go1.16.7.linux-amd64.tar.gz
tar -xzf go1.16.7.linux-amd64.tar.gz
sudo mv go /usr/local
export GOROOT=/usr/local/go
export PATH=$GOROOT/bin:$PATH
rm -rf go1.16.7.linux-amd64.tar.gz
```

#### Docker

```shell
# RedHat Enterprise Linux-8.3
dnf module enable -y container-tools
dnf install -y yum-utils
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
dnf install -y docker-ce-20.10.8 docker-ce-cli-20.10.8

systemctl enable docker
systemctl start docker

# Ubuntu-18.04
apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
    
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

apt-get update

# Ubuntu 18.04 
apt-get -y install docker-ce=5:20.10.8~3-0~ubuntu-bionic docker-ce-cli=5:20.10.8~3-0~ubuntu-bionic containerd.io

# Ubuntu 20.04 
apt-get -y install docker-ce=5:20.10.8~3-0~ubuntu-focal docker-ce-cli=5:20.10.8~3-0~ubuntu-focal containerd.io

systemctl enable docker
systemctl start docker
```

Apply the below steps **only if** running behind a proxy

```shell
mkdir -p /etc/systemd/system/docker.service.d
touch /etc/systemd/system/docker.service.d/proxy.conf

#Add the below lines in proxy.conf
[Service]
Environment="HTTP_PROXY=<http_proxy>"
Environment="HTTPS_PROXY=<https_proxy>"
Environment="NO_PROXY=<no_proxy>"

systemctl daemon-reload
systemctl restart docker
```

### Build OCI Container images and K8s Manifests

#### Foundational Security

* Sync the repos

  ```shell
  mkdir -p /root/intel-secl/build/fs && cd /root/intel-secl/build/fs
  repo init -u https://github.com/intel-secl/build-manifest.git -m manifest/fs.xml -b refs/tags/v5.0.0
  repo sync
  ```

* Run the `pre-requisites` setup script

  ```shell
  cd utils/build/foundational-security/
  chmod +x fs-prereq.sh
  ./fs-prereq.sh -s
  ```

* Install skopeo

  ```shell
  # RHEL 8.x
  dnf install -y skopeo

  # Ubuntu 18.04
  echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_18.04/ /" | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
  curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_18.04/Release.key | sudo apt-key add -
  apt-get update
  apt-get -y upgrade
  apt-get -y install skopeo
  ```
  
* Build

  ```shell
  cd /root/intel-secl/build/fs/
  
  #Single node cluster with microk8s
  make k8s-aio
  
  #Multi node cluster with kubeadm
  make k8s
  ```

* Built Container images,K8s manifests and deployment scripts

  ```shell
  /root/intel-secl/build/fs/k8s/
  ```

#### Workload Security
##### Container Confidentiality with CRIO Runtime

* Sync the repos

  ```shell
  mkdir -p /root/intel-secl/build/cc-crio && cd /root/intel-secl/build/cc-crio
  repo init -u https://github.com/intel-secl/build-manifest.git -m manifest/cc-crio.xml -b refs/tags/v5.0.0
  repo sync
  ```

* Run the `pre-requisites` script

  ```shell
  cd utils/build/workload-security
  chmod +x ws-prereq.sh
  ./ws-prereq.sh -c
  ```
  
* Build

  ```shell
  cd /root/intel-secl/build/cc-crio
  
  #Single node cluster with microk8s
  make k8s-aio
  
  #Multi node cluster with kubeadm
  make k8s
  ```

* Built Container images,K8s manifests and deployment scripts

  ```shell
  /root/intel-secl/build/cc-crio/k8s/
  ```

## Deployment

### Pre-requisites

* Install `openssl` on K8s control-plane

* Ensure a docker registry is running locally or remotely. 

  > **Note:** For single node `microk8s` deployment, a registry can be brought up by using microk8s add-ons. More details present in Microk8s documentation. This is not mandatory, if a remote registry already exists, the same can be  used as well for single-node
  >
  > **Note:** For multi-node `kubeadm` deployment, a docker registry needs to be setup by the user

* Push all container images to docker registry. Example below

  ```shell
  # Without TLS enabled
  skopeo copy oci-archive:<oci-image-tar-name> docker://<registry-ip/hostname>:<registry-port>/<image-name>:<image-tag> --dest-tls-verify=false
  
  # With TLS enabled
  skopeo copy oci-archive:<oci-image-tar-name> docker://<registry-ip/hostname>:<registry-port>/image-name>:<image-tag>
  ```

  > **Note:**  In case of microk8s deployment, when docker registry is enabled locally, the OCI container images need to be copied to the node where registry is enabled and then the above example command can be run. The same would not be required when registry is remotely installed

* On each worker node with `TXT/BTG` enabled and registered to K8s control-plane, the following pre-req needs to be done on `RHEL-8.3`/`Ubuntu-18.04` systems

  * Foundational Security

    * `Tboot-1.10.1` or later to be installed for non `SUEFI` servers. [Tboot installation Details](https://github.com/intel-secl/docs/v5.0/develop/product-guides/Foundational%20%26%20Workload%20Security.md#tboot-installation)

    * Only for `Ubuntu-18.04`, run the following commands

      ```shell
      $ modprobe msr
      ```

  * Workload Security
    * Container Confidentiality with CRIO(>=v1.21) runtime 
      
      * `Tboot-1.10.1`  or later to be installed for non `SUEFI` servers. [Tboot installation Details](https://github.com/intel-secl/docs/blob/v4.0.1/develop/product-guides/Foundational%20%26%20Workload%20Security.md#tboot-installation) 
      
      * Reboot the server
      
      * Only for `Ubuntu-18.04`, run the following command
      
          ```shell
          $ modprobe msr
          ```

### Deploy

#### Single-Node

##### Pre-requisites

###### Setup

* `microk8s` being the default supported single node K8s distribution, users would need to install microk8s on a Physical server

* Copy all manifests and OCI container images as required to KK8s control-plane

* Ensure docker registry is running locally or remotely

* The K8s cluster admin should configure the existing bare metal worker nodes or register fresh bare metal worker nodes with labels. 
  For example, a label like `node.type: TXT-ENABLED` or `node.type: SUEFI-ENABLED` respectively for `TXT/SUEFI` enabled servers 
  can be used by the cluster admin to distinguish the baremetal worker node and the same label can be used in ISECL Agent pod configuration 
  to schedule on all worker nodes marked with the label. The same label is being used as default in the K8s manifests. 
  This can be edited in `k8s/manifests/ta/daemonset.yml` , `k8s/manifests/wla/daemonset.yml` 
  
  Refer Section in [appendix for Feature Detection](#hardware-feature-detection)
  - `node.type: TXT-ENABLED` should be labeled for nodes installed with tboot, where event logs will be collected from tboot measurements.
  - `node.type: SUEFI-ENABLED` should be labeled for nodes with SUEFI enabled, where event logs will be efi logs.
  
  ```shell
  #Label node for TXT
  kubectl label node <node-name> node.type=TXT-ENABLED
  
  #Label node for SUEFI
  kubectl label node <node-name> node.type=SUEFI-ENABLED
  ```
  
* In case of `microk8s` cluster, the `--allow-privileged=true` flag needs to be added to the `kube-apiserver` under `/var/snap/microk8s/current/args/kube-apiserver` and restart `kube-apiserver` with `systemctl restart snap.microk8s.daemon-apiserver` to allow running of privileged containers 
  like `TRUST-AGENT` and `WORKLOAD-AGENT`
  
* Ensure a backend KMIP-2.0 compliant server like pykmip is up and running.

###### Manifests

* Update all the K8s manifests with the image names to be pulled from the registry

* The `tolerations` and `node-affinity` in case of isecl-scheduler and isecl-controller needs to be updated in the respective manifests under the `manifests/k8s-extensions-controller`  and `manifests/k8s-extensions-scheduler` directories to `microk8s.io/cluster` based on k8s distributions of `kubeadm` and `microk8s` respectively
##### Deploy steps

The bootstrap script would facilitate the deployment of all FS,WS components at a use case level. Sample one given below.

###### Update `isecl-k8s.env` file


```shell
#Kubernetes Distribution - microk8s
K8S_DISTRIBUTION=microk8s
K8S_CONTROL_PLANE_IP=
K8S_CONTROL_PLANE_HOSTNAME=

# cms
CMS_BASE_URL=https://cms-svc.isecl.svc.cluster.local:8445/cms/v1
CMS_SAN_LIST=cms-svc.isecl.svc.cluster.local,<K8s control-plane IP/K8s control-plane Hostname>
CMS_K8S_ENDPOINT_URL=https://<k8s control-plane IP>:30445/cms/v1

# authservice
AAS_API_URL=https://aas-svc.isecl.svc.cluster.local:8444/aas/v1
AAS_API_CLUSTER_ENDPOINT_URL=https://<K8s control-plane IP>:30444/aas/v1
AAS_ADMIN_USERNAME=admin@aas
AAS_ADMIN_PASSWORD=aasAdminPass
AAS_DB_USERNAME=aasdbuser
AAS_DB_PASSWORD=aasdbpassword
AAS_DB_HOSTNAME=aasdb-svc.isecl.svc.cluster.local
AAS_DB_PORT="5432"
AAS_DB_NAME=aasdb
AAS_DB_SSLMODE=verify-full
AAS_SAN_LIST=aas-svc.isecl.svc.cluster.local,<K8s control-plane IP/K8s control-plane Hostname>
#NATS_ACCOUNT_NAME=ISecL-account

# Workload Service
WLS_SERVICE_USERNAME=admin@wls
WLS_SERVICE_PASSWORD=wlsAdminPass
WLS_API_URL=https://wls-svc.isecl.svc.cluster.local:5000/wls/v1
WLS_CERT_SAN_LIST=wls-svc.isecl.svc.cluster.local

# Host Verification Service
HVS_SERVICE_USERNAME=admin@hvs
HVS_SERVICE_PASSWORD=hvsAdminPass
HVS_DB_USERNAME=hvsdbuser
HVS_DB_PASSWORD=hvsdbpassword
HVS_DB_HOSTNAME=hvsdb-svc.isecl.svc.cluster.local
HVS_DB_NAME=hvsdb
HVS_CERT_SAN_LIST=hvs-svc.isecl.svc.cluster.local,<K8s control-plane IP/K8s control-plane Hostname>
HVS_DB_PORT="5432"
HVS_URL=https://hvs-svc.isecl.svc.cluster.local:8443/hvs/v2/

#Nats Servers configuration for TA and HVS
#NATS_SERVERS=nats://<K8s control-plane IP/Hostname>:30222

# ihub bootstrap
IHUB_SERVICE_USERNAME=admin@hub
IHUB_SERVICE_PASSWORD=hubAdminPass
IH_CERT_SAN_LIST=ihub-svc.isecl.svc.cluster.local,<K8s control-plane IP/K8s control-plane Hostname>
# For microk8s
# K8S_API_SERVER_CERT=/var/snap/microk8s/current/certs/server.crt
K8S_API_SERVER_CERT=/var/snap/microk8s/current/certs/server.crt
# This is valid for multinode deployment, should be populated once ihub is deployed successfully
IHUB_PUB_KEY_PATH=
HVS_BASE_URL=https://hvs-svc.isecl.svc.cluster.local:8443/hvs/v2

# TrustAgent
# e.g TA_CERT_SAN_LIST=*.example.com,192.168.1.*
TA_CERT_SAN_LIST=
TPM_OWNER_SECRET=

# Workload Agent
WLA_SERVICE_USERNAME=wlauser@wls
WLA_SERVICE_PASSWORD=wlaAdminPass

# KBS
ENDPOINT_URL=https://kbs-svc.isecl.svc.cluster.local:9443/v1
KBS_CERT_SAN_LIST=kbs-svc.isecl.svc.cluster.local,<K8s control-plane IP>,<K8s control-plane Hostname>
KMIP_HOSTNAME=<KMIP IP/Hostname>
KMIP_SERVER_IP=
KMIP_SERVER_PORT=
# Retrieve the following KMIP server’s client certificate, client key and root ca certificate from the KMIP server.
# This key and certificates will be available in KMIP server, /etc/pykmip is the default path copy them to this system manifests/kbs/kmip-secrets path
KMIP_CLIENT_CERT_NAME=client_certificate.pem
KMIP_CLIENT_KEY_NAME=client_key.pem
KMIP_ROOT_CERT_NAME=root_certificate.pem

# ISecl Scheduler
# For microk8s
# K8S_CA_KEY=/var/snap/microk8s/current/certs/ca.key
# K8S_CA_CERT=/var/snap/microk8s/current/certs/ca.crt
K8S_CA_KEY=/var/snap/microk8s/current/certs/ca.key
K8S_CA_CERT=/var/snap/microk8s/current/certs/ca.crt

# populate users.env
ISECL_INSTALL_COMPONENTS="AAS,HVS,WLS,IHUB,KBS,WLA,TA,WPM"

#NATS_CERT_SAN_LIST=
#NATS_TLS_COMMON_NAME=

GLOBAL_ADMIN_USERNAME=
GLOBAL_ADMIN_PASSWORD=

INSTALL_ADMIN_USERNAME=
INSTALL_ADMIN_PASSWORD=

WPM_SERVICE_USERNAME=
WPM_SERVICE_PASSWORD=

CUSTOM_CLAIMS_COMPONENTS=
CCC_ADMIN_USERNAME=
CCC_ADMIN_PASSWORD=
```

> **Note:** Ensure to update `KMIP_CLIENT_CERT_NAME`, `KMIP_CLIENT_KEY_NAME`, `KMIP_ROOT_CERT_NAME` in the env from `/etc/pykmip` of pykmip by copying the key and certs to this system under `manifests/kbs/kmip-secrets` path

###### Run scripts on K8s control-plane

* The bootstrap scripts are sample scripts to allow for a quick start of FS,WS services and agents. Users are free to modify the script or directly use the K8s manifests as per their deployment model requirements

```shell
#Pre-reqs.sh
./pre-requisites.sh

#isecl-bootstrap-db-services
#Reference
#Usage: ./isecl-bootstrap-db-services.sh [-help/up/purge]
#    -help          print help and exit
#    up        Bootstrap Database Services for Authservice, Workload Service and Host #verification Service
#    purge     Delete Database Services for Authservice, Workload Service and Host #verification Service

./isecl-bootstrap-db-services.sh up

#isecl-bootstrap
#Reference
#Usage: ./isecl-bootstrap.sh [-help/up/down/purge]
#    -help                                     Print help and exit
#    up   [all/<agent>/<service>/<usecase>]    Bootstrap ISecL K8s environment for #specified agent/service/usecase
#    down [all/<agent>/<service>/<usecase>]    Delete ISecL K8s environment for specified #agent/service/usecase [will not delete data, config, logs]
#    purge                                     Delete ISecL K8s environment with data, #config, logs [only supported for single node deployments]

#    Available Options for up/down command:
#        agent      Can be one of tagent, wlagent
#        service    Can be one of cms, authservice, hvs, ihub, wls, kbs, isecl-#controller, isecl-scheduler
#        usecase    Can be one of foundational-security, workload-security, isecl-#orchestration-k8s, csp, enterprise

./isecl-bootstrap.sh up <all/usecase of choice>
```

> **Note:** An error to create asymmetric key would mean the following line, `RANDFILE = $ENV::HOME/.rnd` needs to be commented under `/etc/ssl/openssl.cnf`  

* Update the `IHUB_PUB_KEY_PATH` in `isecl-k8s.env` to `/etc/ihub/ihub_public_key.pem`
* Bring up isecl-scheduler

```shell
./isecl-bootstrap.sh up isecl-scheduler
```

* Copy `scheduler-policy.json`

```shell
mkdir -p /opt/isecl-k8s-extensions
cp manifests/k8s-extensions-scheduler/config/scheduler-policy.json /opt/isecl-k8s-extensions/
```

* Edit `kube-scheduler` and restart kubelet

```shell
#Edit the kube-scheduler
vi /var/snap/microk8s/current/args/kube-scheduler

#Add the below line
--policy-config-file=/opt/isecl-k8s-extensions/scheduler-policy.json

#Restart kubelet
systemctl restart snap.microk8s.daemon-kubelet.service
```

#### Multi-Node

##### Pre-requisites

###### Setup

* `kubeadm` being the default supported multi-node K8s distribution, users would need to install a kubeadm K8s control-plane node setup

* Copy all manifests and OCI container images as required to K8s control-plane

* Ensure images are pushed to registry locally or remotely

* The K8s cluster admin should configure the existing bare metal worker nodes or register fresh bare metal worker nodes with labels. 
  For example, a label like `node.type: TXT-ENABLED` or `node.type: SUEFI-ENABLED` respectively for `TXT/SUEFI` enabled servers 
  can be used by the cluster admin to distinguish the baremetal worker node and the same label can be used in ISECL Agent pod configuration 
  to schedule on all worker nodes marked with the label. The same label is being used as default in the K8s manifests. 
  This can be edited in `k8s/manifests/ta/daemonset.yml` , `k8s/manifests/wla/daemonset.yml` 

  Refer Section in [appendix for Feature Detection](#hardware-feature-detection)
  - `node.type: TXT-ENABLED` should be used for nodes with tboot installed, where event logs will be collected from tboot measurements.
  - `node.type: SUEFI-ENABLED` should be used for nodes with SUEFI enabled, where event logs will be EFI logs.
    ```shell
    #Label node for TXT
    kubectl label node <node-name> node.type=TXT-ENABLED

    #Label node for SUEFI
    kubectl label node <node-name> node.type=SUEFI-ENABLED
    ```

* `NFS` storage class is used in kubernetes environment for data persistence and supported in ISecL FS/WS usecases. User needs to setup NFS server and create directory structure along with granting permission for a given user id. From security point of view, its been recommended to create a separate user id and grant the permission for all isecl directories for this user id. Below are some samples for reference

  * Snapshot showing directory structure for which user needs to create on NFS volumes manually or using custom scripts.

  ![NFS Directory Structure FS/WS Usecase](./images/nfs-fsws-structure.png)

  * Snapshot showing ownership and permissions for directories for which user needs to manually grant the ownership.

  ![NFS Directory Permissions FS/WS Usecase](./images/nfs-fsws-permissions.png)

  * Snapshot for configuring PV and PVC , user need to provide the NFS server IP or hostname and paths for each of the service directories. Sample manifest for creating `config-pv` for cms service

    ```yaml
    ---
    apiVersion: v1
    kind: PersistentVolume
    metadata:
        name: cms-config-pv
    spec:
        capacity:
        storage: 128Mi
        volumeMode: Filesystem
        accessModes:
        - ReadWriteMany
        persistentVolumeReclaimPolicy: Retain
        storageClassName: nfs
        nfs:
        path: /<NFS-vol-base-path>/isecl/cms/config
        server: <NFS Server IP/Hostname>
    ```

  * Sample manifest for creating config-pvc for cms service

    ```yaml
    ---
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: cms-config-pvc
      namespace: isecl
    spec:
      storageClassName: nfs
      accessModes:
      - ReadWriteMany
      resources:
      requests:
          storage: 128Mi
    ```

  > **Note:** The user id specified in security context in `deployment.yml` for a given service and owner of the service related directories in NFS must be same
  
* Ensure a backend KMIP-2.0 compliant server like pykmip is up and running.

###### Manifests

* Update all the K8s manifests with the image names to be pulled from the registry

* The `tolerations` and `node-affinity` in case of isecl-scheduler and isecl-controller needs to be updated in the respective manifests under the `manifests/k8s-extensions-controller`  and `manifests/k8s-extensions-scheduler` directories to `node-role.kubernetes.io/master`
* All NFS PV yaml files needs to be updated with the  `path: /<NFS-vol-path>`  and `server: <NFS Server IP/Hostname>` under each service manifest file for `config`, `logs` , `db-data`

##### Deploy steps

###### Update `isecl-k8s.env` file

```shell
#Kubernetes Distribution - kubeadm
K8S_DISTRIBUTION=kubeadm
K8S_CONTROL_PLANE_IP=
K8S_CONTROL_PLANE_HOSTNAME=

# cms
CMS_BASE_URL=https://cms-svc.isecl.svc.cluster.local:8445/cms/v1
CMS_SAN_LIST=cms-svc.isecl.svc.cluster.local,<K8s control-plane IP/K8s control-plane Hostname>
CMS_K8S_ENDPOINT_URL=https://<k8s control-plane IP>:30445/cms/v1

# authservice
AAS_API_URL=https://aas-svc.isecl.svc.cluster.local:8444/aas/v1
AAS_API_CLUSTER_ENDPOINT_URL=https://<K8s control-plane IP>:30444/aas/v1
AAS_ADMIN_USERNAME=admin@aas
AAS_ADMIN_PASSWORD=aasAdminPass
AAS_DB_USERNAME=aasdbuser
AAS_DB_PASSWORD=aasdbpassword
AAS_DB_HOSTNAME=aasdb-svc.isecl.svc.cluster.local
AAS_DB_PORT="5432"
AAS_DB_NAME=aasdb
AAS_DB_SSLMODE=verify-full
AAS_SAN_LIST=aas-svc.isecl.svc.cluster.local,<K8s control-plane IP/K8s control-plane Hostname>
#NATS_ACCOUNT_NAME=ISecL-account

# Workload Service
WLS_SERVICE_USERNAME=admin@wls
WLS_SERVICE_PASSWORD=wlsAdminPass
WLS_API_URL=https://wls-svc.isecl.svc.cluster.local:5000/wls/v1
WLS_CERT_SAN_LIST=wls-svc.isecl.svc.cluster.local

# Host Verification Service
HVS_SERVICE_USERNAME=admin@hvs
HVS_SERVICE_PASSWORD=hvsAdminPass
HVS_DB_USERNAME=hvsdbuser
HVS_DB_PASSWORD=hvsdbpassword
HVS_DB_HOSTNAME=hvsdb-svc.isecl.svc.cluster.local
HVS_DB_NAME=hvsdb
HVS_CERT_SAN_LIST=hvs-svc.isecl.svc.cluster.local,<K8s control-plane IP/K8s control-plane Hostname>
HVS_DB_PORT="5432"
HVS_URL=https://hvs-svc.isecl.svc.cluster.local:8443/hvs/v2/

#Nats Servers configuration for TA and HVS
#NATS_SERVERS=nats://<K8s control-plane IP/Hostname>:30222

# ihub bootstrap
IHUB_SERVICE_USERNAME=admin@hub
IHUB_SERVICE_PASSWORD=hubAdminPass
IH_CERT_SAN_LIST=ihub-svc.isecl.svc.cluster.local,<K8s control-plane IP/K8s control-plane Hostname>
# For Kubeadm
# K8S_API_SERVER_CERT=/etc/kubernetes/pki/apiserver.crt
K8S_API_SERVER_CERT=/etc/kubernetes/pki/apiserver.crt
# This is valid for multinode deployment, should be populated once ihub is deployed successfully
IHUB_PUB_KEY_PATH=
HVS_BASE_URL=https://hvs-svc.isecl.svc.cluster.local:8443/hvs/v2

# TrustAgent
# e.g TA_CERT_SAN_LIST=*.example.com,192.168.1.*
TA_CERT_SAN_LIST=
TPM_OWNER_SECRET=

# Workload Agent
WLA_SERVICE_USERNAME=wlauser@wls
WLA_SERVICE_PASSWORD=wlaAdminPass

# KBS
ENDPOINT_URL=https://kbs-svc.isecl.svc.cluster.local:9443/v1
KBS_CERT_SAN_LIST=kbs-svc.isecl.svc.cluster.local,<K8s control-plane IP>,<K8s control-plane Hostname>
KMIP_HOSTNAME=<KMIP IP/Hostname>
KMIP_SERVER_IP=
KMIP_SERVER_PORT=
# Retrieve the following KMIP server’s client certificate, client key and root ca certificate from the KMIP server.
# This key and certificates will be available in KMIP server, /etc/pykmip is the default path copy them to this system manifests/kbs/kmip-secrets path
KMIP_CLIENT_CERT_NAME=client_certificate.pem
KMIP_CLIENT_KEY_NAME=client_key.pem
KMIP_ROOT_CERT_NAME=root_certificate.pem

# ISecl Scheduler
# For Kubeadm
# K8S_CA_KEY=/etc/kubernetes/pki/ca.key
# K8S_CA_CERT=/etc/kubernetes/pki/ca.crt
K8S_CA_KEY=/etc/kubernetes/pki/ca.key
K8S_CA_CERT=/etc/kubernetes/pki/ca.crt

# populate users.env
ISECL_INSTALL_COMPONENTS="AAS,HVS,WLS,IHUB,KBS,WLA,TA,WPM"

#NATS_CERT_SAN_LIST=
#NATS_TLS_COMMON_NAME=

GLOBAL_ADMIN_USERNAME=
GLOBAL_ADMIN_PASSWORD=

INSTALL_ADMIN_USERNAME=
INSTALL_ADMIN_PASSWORD=

WPM_SERVICE_USERNAME=
WPM_SERVICE_PASSWORD=

CUSTOM_CLAIMS_COMPONENTS=
CCC_ADMIN_USERNAME=
CCC_ADMIN_PASSWORD=
```

> **Note:** Ensure to update `KMIP_CLIENT_CERT_NAME`, `KMIP_CLIENT_KEY_NAME`, `KMIP_ROOT_CERT_NAME` in the env from `/etc/pykmip` of pykmip by copying the key and certs to this system under `manifests/kbs/kmip-secrets` path

###### Run scripts on K8s control-plane

* The bootstrap scripts are sample scripts to allow for a quick start of FS,WS services and agents. Users are free to modify the script or directly use the K8s manifests as per their deployment model requirements

```shell
#Pre-reqs.sh
./pre-requisites.sh

#isecl-bootstrap-db-services
#Reference
#Usage: ./isecl-bootstrap-db-services.sh [-help/up/purge]
#    -help          print help and exit
#    up        Bootstrap Database Services for Authservice, Workload Service and Host verification Service
#    purge     Delete Database Services for Authservice, Workload Service and Host verification Service

./isecl-bootstrap-db-services.sh up

#isecl-bootstrap
#Reference
#Usage: ./isecl-bootstrap.sh [-help/up/down/purge]
#    -help                                     Print help and exit
#    up   [all/<agent>/<service>/<usecase>]    Bootstrap ISecL K8s environment for #specified agent/service/usecase
#    down [all/<agent>/<service>/<usecase>]    Delete ISecL K8s environment for specified #agent/service/usecase [will not delete data, config, logs]
#    purge                                     Delete ISecL K8s environment with data, config, logs [only supported for single node deployments]

#    Available Options for up/down command:
#        agent      Can be one of tagent, wlagent
#        service    Can be one of cms, authservice, hvs, ihub, wls, kbs, isecl-#controller, isecl-scheduler
#        usecase    Can be one of foundational-security, workload-security, isecl-#orchestration-k8s, csp, enterprise

./isecl-bootstrap.sh up <all/usecase of choice>
```

> **Note:** An error to create asymmetric key would mean the following line, `RANDFILE = $ENV::HOME/.rnd` needs to be commented under `/etc/ssl/openssl.cnf`  


* Copy the `ihub_public_key.pem` from NFS path -`<mnt>/isecl/ihub/config/ihub_public_key.pem ` to K8s control-plane
* Update the `isecl-k8s.env` for `IHUB_PUB_KEY_PATH`
* Bring up the `isecl-k8s-scheduler`

```shell
./isecl-bootstrap.sh up isecl-scheduler
```


* Create and update `scheduler-policy.json` path

```shell
mkdir -p /opt/isecl-k8s-extensions
cp manifests/k8s-extensions-scheduler/config/scheduler-policy.json /opt/isecl-k8s-extensions
```

* Configure kube-scheduler to establish communication with isecl-scheduler. Add `scheduler-policy.json` under kube-scheduler section, `mountPath` under container section and `hostPath` under volumes section in ` /etc/kubernetes/manifests/kube-scheduler.yaml` as mentioned below

```yaml
spec:
  containers:
  - command:
    - kube-scheduler
    - --policy-config-file=/opt/isecl-k8s-extensions/scheduler-policy.json
```

```yaml
containers:
    volumeMounts:
    - mountPath: /opt/isecl-k8s-extensions/
      name: extendedsched
      readOnly: true
```

```yaml
volumes:
  - hostPath:
      path: /opt/isecl-k8s-extensions/
      type:
    name: extendedsched
```

> **Note:** Make sure to use proper indentation and don't delete existing `mountPath` and `hostPath` sections in `kube-scheduler.yaml`

* Restart `kubelet` which restart all the k8s services including kube-scheduler

```shell
systemctl restart kubelet
```


## Default Service and Agent Mount Paths

### Single Node Deployments

Single node Deployments use `hostPath` mounting pod(container) files directly on host. Following is the complete list of the files being mounted on host

```yaml
#Certificate-Management-Service
Config: /etc/cms
Logs: /var/log/cms

#Authentication Authorization Service
Config: /etc/authservice
Logs: /var/log/authservice
Pg-data: /usr/local/kube/data/authservice/pgdata

#Host Attestation Service
Config: /etc/hvs
Logs: /var/log/hvs
Pg-data: /usr/local/kube/data/hvs

#Integration-Hub
Config: /etc/ihub
Log: /var/log/ihub

#Workload Service
Config: /etc/wls
Logs: /var/log/wls

#Key-Broker-Service
Config: /etc/kbs
Log: /var/log/kbs
Opt: /opt/kbs

#Trust Agent:
Config: /opt/trustagent/configuration
Logs:  /var/log/trustagent/
tpmrm: /dev/tpmrm0
txt-stat: /usr/sbin/txt-stat
ta-hostname-path: /etc/hostname
ta-hosts-path: /etc/hosts

#Workload Agent:
Config: /etc/workload-agent/
Logs: /var/log/workload-agent
TA Config: /opt/trustagent/configuration
WLA-Socket: /var/run/workload-agent
```

### Multi Node Deployments

Multi node Deployments use k8s persistent volume and persistent volume claims for mounting pod(container) files on NFS volumes for all services, agents will continue to use `hostPath`. Following is a sample list of the files being mounted on NFS base volumes

```yaml
#Certificate-Management-Service
Config: <NFS-vol-base-path>/isecl/cms/config
Logs: <NFS-vol-base-path>/isecl/cms/logs

#Authentication Authorization Service
Config: <NFS-vol-base-path>/isecl/aas/config
Logs: <NFS-vol-base-path>/isecl/aas/logs
Pg-data: <NFS-vol-base-path>/isecl/aas/db

#Host Attestation Service
Config: <NFS-vol-base-path>/isecl/hvs/config
Logs: <NFS-vol-base-path>/isecl/hvs/logs
Pg-data: <NFS-vol-base-path>/usr/local/kube/data/hvs

#Integration-Hub
Config: <NFS-vol-base-path>/isecl/ihub/config
Log: <NFS-vol-base-path>/isecl/ihub/logs

#Workload Service
Config: <NFS-vol-base-path>/isecl/wls/config
Logs: <NFS-vol-base-path>/isecl/wls/log

#Key-Broker-Service
Config: <NFS-vol-base-path>/isecl/kbs/config
Log: <NFS-vol-base-path>/isecl/kbs/logs
Opt: <NFS-vol-base-path>/isecl/kbs/kbs/opt

#Trust Agent:
Config: /opt/trustagent/configuration
Logs:  /var/log/trustagent/
tpmrm: /dev/tpmrm0
txt-stat: /usr/sbin/txt-stat
ta-hostname-path: /etc/hostname
ta-hosts-path: /etc/hosts

#Workload Agent:
Config: /etc/workload-agent/
Logs: /var/log/workload-agent
WLA-Socket: /var/run/workload-agent
```



## Default Service Ports

For both Single node and multi-node deployments following ports are used. All services exposing APIs will use the below ports

```yaml
CMS: 30445
AAS: 30444
HVS: 30443
WLS: 30447
IHUB: None
KBS: 30448
K8s-scheduler: 30888
K8s-controller: None
TA: 31443
WLA: None
```



## Usecase Workflows API Collections

The below allow to get started with workflows within Intel® SecL-DC for Foundational and Workload Security Usecases. More details available in [API Collections](https://github.com/intel-secl/utils/tree/v5.0/develop/tools/api-collections) repository

### Pre-requisites

* Postman client should be [downloaded](https://www.postman.com/downloads/) on supported platforms or on the web to get started with the usecase collections.

  >  **Note:** The Postman API Network will always have the latest released version of the API Collections. For all releases, refer the github repository for [API Collections](https://github.com/intel-secl/utils/tree/v5.0/develop/tools/api-collections)

### Use Case Collections

| Use case               | Sub-Usecase                                   | API Collection     |
| ---------------------- | --------------------------------------------- | ------------------ |
| Foundational Security  | Host Attestation(RHEL & VMWARE)                              | ✔️                  |
|                        | Data Fencing  with Asset Tags(RHEL & VMWARE)                 | ✔️                  |
|                        | Trusted Workload Placement (Containers)  | ✔️ |
| Workload Security | Container Confidentiality with CRIO Runtime | ✔️                  |

### Downloading API Collections

* Postman API Network for latest released: https://explore.postman.com/intelsecldc

  or 

* Github repo for all releases

  ```shell
  #Clone the github repo for api-collections
  git clone https://github.com/intel-secl/utils.git
  
  #Switch to specific release-version of choice
  cd utils/
  git checkout <release-version of choice>
  
  #Import Collections from
  cd tools/api-collections
  ```

> **Note:**  The postman-collections are also available when cloning the repos via build manifest under `utils/tools/api-collections`

### Running API Collections

* Import the collection into Postman API Client

  > **Note:** This step is required only when not using Postman API Network and downloading from Github

  ![importing-collection](./images/importing_collection.gif)

* Update env as per the deployment details for specific usecase

  ![updating-env](./images/updating_env.gif)

* View Documentation

  ![view-docs](./images/view_documentation.gif)

* Run the workflow

  ![running-collection](./images/running_collection.gif)


## Appendix

### Hardware feature detection
For checking whether a system is SUEFI enabled
```bootctl | grep "Secure Boot: enabled"```

For checking system installed with tboot
```txt-stat | grep "TXT measured launch: TRUE" ```

### Running behind Proxy

```shell
#Set proxy in ~/.bash_profile
export http_proxy=<proxy-url>
export https_proxy=<proxy-url>
export no_proxy=<ip_address/hostname>
```

### Git Config Sample (~/.gitconfig)

```ini
[user]
        name = <username>
        email = <email-id>
[color]
        ui = auto
 [push]
        default = matching 
```

### Rebuilding Repos

In order to rebuild repos, ensure the following steps are followed as a pre-requisite

```shell
# Clean all go-mod packages
rm -rf ~/go/pkg/mod/*

#Navigate to specific folder where repos are built, example below
cd /root/isecl/build/fs
rm -rf .repo *
#Rebuild as before
repo init ...
repo sync
make ...
```

### Setup Task Flow

Setup tasks flows have been updated to have K8s native flow to be more agnostic to K8s workflows. Following would be the flow for setup task

- User would create a new `configMap`  object with the environment variables specific to the setup task. The Setup task variables would be documented in the Product Guide
- Users can provide variables for more than one setup task
- Users need to add `SETUP_TASK: "<setup task name>/<comma separated setup task name>"` in the same `configMap`
- Provide a unique name to the new `configMap`
- Provide the same name in the `deployment.yaml` under `configMapRef` section
- Deploy the specific service again with  ` kubectl kustomize . | kubectl  apply -f -`

### Configuration Update Flow

Configuration Update flows have been updated to have K8s native flow to be more agnostic to K8s workflows using `configMap` only. Following would be the flow for configuration update

- User would create a new `configMap`  object using existing one and update the new values. The list of config variables would be documented in the Product Guide
- Users can update variables for more than one
- Users need to add `SETUP_TASK: "update-service-config"` in the same `configMap`
- Provide a unique name to the new `configMap`
- Provide the same name in the `deployment.yaml` under `configMapRef` section
- Deploy the specific service again with  ` kubectl  kustomize . | kubectl  apply -f -`

> **Note:** Incase of agents, setup tasks or configuration updates done through above flows will be applied for all the agents running on different BMs. In order to run setup task or update configuration for individual agents, then user need to perform `kubectl exec -it <pod_name> /bin/bash` into a particular agent pod and run the specific setup task.

### Cleanup workflows

#### Single-node

In order to cleanup and setup fresh again on single node without data, config from previous deployment

```shell
#Purge all data and pods,deploy,cm,secrets
./isecl-bootstrap.sh purge <all/usecase of choice>

#Purge all db data,pods,deploy,cm,secrets
./isecl-bootstrap-db-services.sh purge

#Comment/Remove the following lines from /var/snap/microk8s/current/args/kube-scheduler
--policy-config-file=/opt/isecl-k8s-extensions/scheduler-policy.json

#Restart kubelet
systemctl restart snap.microk8s.daemon-kubelet.service

#Setup fresh
./isecl-bootstrap-db-services.sh up
./isecl-bootstrap.sh up <all/usecase of choice>

#Reconfigure K8s-scheduler
vi /var/snap/microk8s/current/args/kube-scheduler

#Add the below line
--policy-config-file=/opt/isecl-k8s-extensions/scheduler-policy.json

#Restart kubelet
systemctl restart snap.microk8s.daemon-kubelet.service
```

In order to cleanup and setup fresh again on single node with data, config from previous deployment

```shell
#Down all data and pods,deploy,cm,secrets with deleting config,data,logs
./isecl-bootstrap.sh down <all/usecase of choice>

#Comment/Remove the following lines from /var/snap/microk8s/current/args/kube-scheduler
--policy-config-file=/opt/isecl-k8s-extensions/scheduler-policy.json

#Restart kubelet
systemctl restart snap.microk8s.daemon-kubelet.service

#Setup fresh
./isecl-bootstrap-db-services.sh up
./isecl-bootstrap.sh up <all/usecase of choice>

#Reconfigure K8s-scheduler
vi /var/snap/microk8s/current/args/kube-scheduler

#Add the below line
--policy-config-file=/opt/isecl-k8s-extensions/scheduler-policy.json

#Restart kubelet
systemctl restart snap.microk8s.daemon-kubelet.service
```

In order to clean single service and bring up again on single node without data, config from previous deployment
```shell script
./isecl-bootstrap.sh down <service-name>
rm -rf /etc/<service-name>
rm -rf /var/log/<service-name>

#Only in case of KBS, perform one more step along with above 2 steps
rm -rf /opt/kbs
./isecl-bootstrap.sh up <service-name>
```

In order to redeploy again on single node with data, config from previous deployment
```shell
./isecl-bootstrap.sh down <service-name>
./isecl-bootstrap.sh up <service-name>
```

#### Multi-node

In order to cleanup and setup fresh again on multi-node with data,config from previous deployment

```shell
#Purge all data and pods,deploy,cm,secrets
./isecl-bootstrap.sh down <all/usecase of choice>

#Delete 'scheduler-policy.json'
rm -rf /opt/isecl-k8s-extensions

#Comment/Remove '--policy-config-file=...' from /etc/kubernetes/manifests/kube-scheduler.yaml as below
 containers:
  - command:
    - kube-scheduler
    - --policy-config-file=/opt/isecl-k8s-extensions/scheduler-policy.json

#Comment/Remove '- mountPath: ...' from /etc/kubernetes/manifests/kube-scheduler.yaml as below
containers:
    volumeMounts:
    - mountPath: /opt/isecl-k8s-extensions/
      name: extendedsched
      readOnly: true
      
#Comment/Remove '-hostPath:...' from /etc/kubernetes/manifests/kube-scheduler.yaml as below
volumes:
  - hostPath:
      path: /opt/isecl-k8s-extensions/
      type:
    name: extendedsched

#Restart kubelet
systemctl restart kubelet

#Purge all db data,pods,deploy,cm,secrets
./isecl-bootstrap-db-services.sh purge

#Cleanup all data from NFS share --> User controlled

#Cleanup data from each worker node --> User controlled
rm -rf /etc/workload-agent
rm -rf /var/log/workload-agent
rm -rf /opt/trustagent
rm -rf /var/log/trustagent

#Setup fresh
./isecl-bootstrap-db-services.sh up
./isecl-bootstrap.sh up <all/usecase of choice>

#Reconfigure K8s-scheduler
containers:
  - command:
    - kube-scheduler
    - --policy-config-file=/opt/isecl-k8s-extensions/scheduler-policy.json

containers:
    volumeMounts:
    - mountPath: /opt/isecl-k8s-extensions/
      name: extendedsched
      readOnly: true

volumes:
  - hostPath:
      path: /opt/isecl-k8s-extensions/
      type:
    name: extendedsched

#Restart kubelet
systemctl restart kubelet
```

In order to cleanup and setup fresh again on multi-node without removing data,config from previous deployment

```shell
#Down all pods,deploy,cm,secrets with removing persistent data
./isecl-bootstrap.sh down <all/usecase of choice>

#Delete 'scheduler-policy.json'
rm -rf /opt/isecl-k8s-extensions

#Comment/Remove '--policy-config-file=...' from /etc/kubernetes/manifests/kube-scheduler.yaml as below
 containers:
  - command:
    - kube-scheduler
    - --policy-config-file=/opt/isecl-k8s-extensions/scheduler-policy.json

#Comment/Remove '- mountPath: ...' from /etc/kubernetes/manifests/kube-scheduler.yaml as below
containers:
    volumeMounts:
    - mountPath: /opt/isecl-k8s-extensions/
      name: extendedsched
      readOnly: true
      
#Comment/Remove '-hostPath:...' from /etc/kubernetes/manifests/kube-scheduler.yaml as below
volumes:
  - hostPath:
      path: /opt/isecl-k8s-extensions/
      type:
    name: extendedsched
    
#Restart kubelet
systemctl restart kubelet

#Setup fresh
./isecl-bootstrap-db-services.sh up
./isecl-bootstrap.sh up <all/usecase of choice>

#Setup fresh
./isecl-bootstrap-db-services.sh up
./isecl-bootstrap.sh up <all/usecase of choice>

#Copy ihub_public_key.pem from <NFS-PATH>/ihub/config/ to K8s control-plane and update IHUB_PUB_KEY_PATH in isecl-isecl-k8s.env

#Bootstrap isecl-scheduler
./isecl-bootstrap.sh up isecl-scheduler

#Reconfigure K8s-scheduler
containers:
  - command:
    - kube-scheduler
    - --policy-config-file=/opt/isecl-k8s-extensions/scheduler-policy.json

containers:
    volumeMounts:
    - mountPath: /opt/isecl-k8s-extensions/
      name: extendedsched
      readOnly: true

volumes:
  - hostPath:
      path: /opt/isecl-k8s-extensions/
      type:
    name: extendedsched
    
#Restart kubelet
systemctl restart kubelet
```

In order to clean single service and bring up again on multi node without data, config from previous deployment
```./isecl-bootstrap.sh down <service-name>```
log into nfs system 

```shell script
rm -rf /<nfs-mount-path>/isecl/<service-name>/config
rm -rf /<nfs-mount-path>/isecl/<service-name>/logs

#Only in case of KBS, perform one more step along with above 2 steps
rm -rf /<nfs-mount-path>/isecl/<service-name>/opt
```
log into K8s control-plane
```./isecl-bootstrap.sh up <service-name>```

In order to redeploy again on multi node with data, config from previous deployment
```shell
./isecl-bootstrap.sh down <service-name>
./isecl-bootstrap.sh up <service-name>
```