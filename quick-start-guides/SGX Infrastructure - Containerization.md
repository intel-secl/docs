# Quick start Guide - SGX Containerization 

Table of Contents
=================

   * [Quick start Guide - SGX Containerization](#quick-start-guide---sgx-containerization)
   * [Table of Contents](#table-of-contents)
      * [Hardware &amp; OS Requirements](#hardware--os-requirements)
         * [Machines](#machines)
         * [OS Requirements](#os-requirements)
         * [Container Runtime](#container-runtime)
         * [K8s Distributions](#k8s-distributions)
         * [Storage](#storage)
      * [Network Requirements](#network-requirements)
         * [Build System](#build-system)
         * [CSP Managed Services](#csp-managed-services)
         * [Enterprise Managed Services](#enterprise-managed-services)
         * [SGX Enabled Host](#sgx-enabled-host)
         * [Firewall Settings](#firewall-settings)
      * [RHEL RPMs Requirements](#rhel-rpms-requirements)
      * [Deployment Model](#deployment-model)
         * [Single Node](#single-node)
         * [Multi Node](#multi-node)
      * [Build](#build)
         * [Pre-requisites](#pre-requisites)
            * [Development Tools and Utilities](#development-tools-and-utilities)
            * [Repo tool](#repo-tool)
            * [Golang](#golang)
            * [Docker](#docker)
            * [Skopeo](#skopeo)
         * [Build OCI Container images and K8s Manifests](#build-oci-container-images-and-k8s-manifests)
            * [Single Node](#single-node-1)
            * [Multi Node](#multi-node-1)
      * [Deploy Linux Stacks for Intel SGX](#deploy-linux-stacks-for-intel-sgx)
      * [Deployment](#deployment)
         * [Pre-requisites](#pre-requisites-1)
         * [Deploy](#deploy)
            * [Single-Node](#single-node-2)
               * [Pre-requisites](#pre-requisites-2)
                  * [Setup](#setup)
                  * [Manifests](#manifests)
               * [Deploy steps](#deploy-steps)
                  * [Update .env file](#update-env-file)
                  * [Run scripts on K8s control-plane](#run-scripts-on-k8s-control-plane)
            * [Multi-Node](#multi-node-2)
               * [Pre-requisites](#pre-requisites-3)
                  * [Setup](#setup-1)
                  * [Manifests](#manifests-1)
               * [Deploy steps](#deploy-steps-1)
                  * [Update .env file](#update-env-file-1)
                  * [Run scripts on K8s control-plane](#run-scripts-on-K8s-control-plane-1)
      * [Default Service and Agent Mount Paths](#default-service-and-agent-mount-paths)
         * [Single Node Deployments](#single-node-deployments)
         * [Multi Node Deployments](#multi-node-deployments)
      * [Default Service Ports](#default-service-ports)
      * [Usecase Workflows API Collections](#usecase-workflows-api-collections)
         * [Pre-requisites](#pre-requisites-4)
         * [Use Case Collections](#use-case-collections)
         * [Downloading API Collections](#downloading-api-collections)
         * [Running API Collections](#running-api-collections)
      * [Appendix](#appendix)
         * [Running behind Proxy](#running-behind-proxy)
         * [Git Config Sample (~/.gitconfig)](#git-config-sample-gitconfig)
         * [Rebuilding Repos](#rebuilding-repos)
         * [SGX Attestation Flow](#sgx-attestation-flow)
         * [SKC Key Transfer Flow](#skc-key-transfer-flow)
            * [Generating keys](#generating-keys)
            * [Setup configurations](#setup-configurations)
            * [Initiate Key Transfer Flow](#initiate-key-transfer-flow)
         * [Other Workloads](#other-workloads)
         * [SGX Discovery Flow](#sgx-discovery-flow)
         * [SKC Virtualization Flow](#skc-virtualization-flow)
            * [Configuration for NGINX testing On SGX Virtualization Setup](#Configuration for NGINX testing On SGX Virtualization Setup)
         * [Setup Task Flow](#setup-task-flow)
         * [Configuration Update Flow](#configuration-update-flow)
         * [Cleanup workflows](#cleanup-workflows)
            * [Single-node](#single-node-3)
            * [Multi-node](#multi-node-3)

## Hardware & OS Requirements

### Machines

* RHEL 8.2 Build Machine

* RHEL 8.4 Build Machine for stack based deployment

* K8s control-plane Node Setup on CSP (VMs/Physical Nodes + SGX enabled Physical Nodes)

* K8s control-plane Setup on Enterprise (VMs/Physical Nodes)

### OS Requirements

* RHEL 8.2 for build

* RHEL 8.4 for build for stack based deployment

* RHEL 8.2 or Ubuntu 18.04/20.04 for K8s cluster deployments

* RHEL 8.4 for stack based K8s cluster deployments 

  >  **Note:** SKC Solution is built, installed and tested with root privileges. Please ensure that all the following instructions are executed with root privileges

### Container Runtime

* Docker

### K8s Distributions

* Single Node: `microk8s`
* Multi Node: `kubeadm`

### Storage

* `hostPath` in case of single-node `microk8s` for all services and agents
* `NFS` in case of multi-node `kubeadm` for services and `hostPath` for sgx_agent and skc_library


## Network Requirements

### Build System

Internet access required

### CSP Managed Services

Internet access required for SGX Caching Service deployed on CSP system/SGX Compute Node;

### Enterprise Managed Services

Internet access required for SGX Caching Service deployed on Enterprise system;

### SGX Enabled Host

Internet access required to access KBS running on Enterprise environment

### Firewall Settings

Ensure that all the SKC service ports are accessible with firewall



## RHEL RPMs Requirements

Access required for the following rpms in all systems

* BaseOS

* Appstream

* CodeReady



## Deployment Model

### Single Node

The single Node uses `microk8s` as a supported K8s distribution

![k8s-single-node](./images/k8s-single-node.png)

### Multi Node

The multi node supports `kubeadm` as a supported K8s distribution

![K8s Deployment-sqx](./images/k8s-deployment-sgx.jpg)

![K8s Multi Deployment-sqx](./images/k8s-multi-deployment-sgx.jpg)

## Build

### Pre-requisites

The below steps need to be done on RHEL 8.2 or RHEL 8.4(stack based deployment) Build Machine (VM/Physical Node)

#### Development Tools and Utilities

```shell
dnf install -y git wget tar python3 gcc gcc-c++ zip tar make yum-utils openssl-devel
dnf install -y https://dl.fedoraproject.org/pub/fedora/linux/releases/32/Everything/x86_64/os/Packages/m/makeself-2.4.0-5.fc32.noarch.rpm
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
On RHEL 8.2:
    dnf module enable -y container-tools
    dnf install -y yum-utils
    dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
    dnf install -y docker-ce-20.10.8 docker-ce-cli-20.10.8

On RHEL 8.4:
    dnf module enable -y container-tools
    dnf install -y yum-utils
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

#Ignore the below steps if not running behind a proxy
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

#### Skopeo

For RHEL 8.2 OS

```shell
dnf install -y skopeo
```
For RHEL 8.4 OS	

```shell
dnf install -y skopeo --nobest
```

For Ubuntu 18.04 OS

```shell
echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_18.04/ /" | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_18.04/Release.key | sudo apt-key add -
apt-get update
apt-get -y upgrade
apt-get -y install skopeo
```

For Ubuntu 20.04 OS

```shell
echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_20.04/ /" | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_20.04/Release.key | sudo apt-key add -
apt-get update
apt-get -y upgrade
apt-get -y install skopeo
```

### Build OCI Container images and K8s Manifests

The build process for OCI containers images and K8s manifests for RHEL 8.2 & Ubuntu 18.04/20.04 deployments must be done on RHEL 8.2 machine only

The build process for OCI containers images and K8s manifests for RHEL 8.4 must be done on RHEL 8.4 machine only

#### Single Node

* Sync the repos

  ```shell
  mkdir -p /root/intel-secl/build/skc-k8s-single-node && cd /root/intel-secl/build/skc-k8s-single-node
  repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v5.0.0 -m manifest/skc.xml
  repo sync
  ```

* Build - Distribution Based Deployment

  ```shell
  make k8s-aio
  ```

* Build - Stack Based Deployment

  ```shell
  make k8s-aio-stacks
  ```

* Built Container images,K8s manifests and deployment scripts

  ```shell
  /root/intel-secl/build/skc-k8s-single-node/k8s/
  ```

#### Multi Node

* Sync the repos

  ```shell
  mkdir -p /root/intel-secl/build/skc-k8s-multi-node && cd /root/intel-secl/build/skc-k8s-multi-node
  repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v5.0.0 -m manifest/skc.xml
  repo sync
  ```

* Build - Distribution Based Deployment

  ```shell
  make k8s
  ```
* Build - Stack Based Deployment

  ```shell
  make k8s-stacks
  ```

* Built Container images,K8s manifests and deployment scripts

  ```shell
  /root/intel-secl/build/skc-k8s-multi-node/k8s/
  ```
## Deploy Linux Stacks for Intel SGX
	
* To setup and deployment of the Linux* Stacks for Intel® SGX, follow https://download.01.org/intelsgxstack/2021-07-28/Getting_Started.pdf

## Deployment

### Pre-requisites

* Install `openssl` on K8s control-plane
  > **Note:** For Ubuntu OS, comment `RANDFILE = $ENV::HOME/.rnd` line in /etc/ssl/openssl.cnf

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

* On each worker node with SGX enabled and registered to K8s control-plane, the following pre-req needs to be done

  * `RHEL 8.2 or Ubuntu 18.04/20.04 or RHEL 8.4 stack based` enabled K8s worker node with SGX:
    * Pre requisite scripts will be available under `k8s/platform-dependencies/` on build machine
    * Copy the platform-dependencies script to SGX enabled worker nodes on K8s
    * Execute  `./agent_untar.sh`  
    * Execute `./agent_container_prereq.sh` for deploying all pre-reqs required for agent
    
* Ensure a backend KMIP-2.0 compliant server like pykmip is up and running.
    * Retrieve KMIP server's key and certificates i.e. client_certificate.pem, client_key.pem and root_certificate.pem files from /etc/pykmip (default path) to K8s control-plane node under `k8s/manifests/kbs/kmip-secrets/` path.
    > **Note:** Under `k8s/manifests/kbs/` , if kmip-secrets folder not available then please create it before copying above key and certs 

* Ensure that system date and time are in sync across all servers (K8s control-plane node, worker node, nfs server, kmip server)

### Deploy

#### Single-Node

##### Pre-requisites 

###### Setup

* `microk8s` being the default supported single node K8s distribution, users would need to install microk8s on a Physical server

* Copy all manifests and OCI container images as required to K8s control-plane

* Ensure docker registry is running locally or remotely

* The K8s cluster admin configure the existing bare metal worker nodes or register fresh bare metal worker nodes with labels. For example, a label like `node.type: SGX-ENABLED` can be used by the cluster admin to distinguish the baremetal worker node and the same label can be used in ISECL Agent pod configuration to schedule on all worker nodes marked with the label. The same label is being used as default in the K8s manifests. This can be edited in `k8s/manifests/sgx_agent/daemonset.yml` , `k8s/manifests/skc_library/deployment.yml`
  ```shell
  #Label node
  kubectl label node <node-name> node.type=SGX-ENABLED
  ```
  
* In case of `microk8s` cluster, the `--allow-privileged=true` flag needs to be added to the `kube-apiserver` under `/var/snap/microk8s/current/args/kube-apiserver` and restart `kube-apiserver` with `systemctl restart snap.microk8s.daemon-apiserver` to allow running of privileged containers 
  like `SGX-AGENT` and `SKC-LIBRARY`
  
* Enable external API communication for `microk8s` to talk to external Intel PCS server

  ```shell
  kubectl get configmap -n kube-system
  kubectl edit configmap -n kube-system coredns
  #edit coredns config map and change the name server as:
  "forward . /etc/resolv.conf"
  #Delete coredns pod, it will relaunch
  ```

###### Manifests

* Update all the K8s manifests with the image names to be pulled from the registry

* The key field in `tolerations` and `nodeAffinity` in case of isecl-scheduler and isecl-controller needs to be updated in the respective manifests under the `manifests/k8s-extensions-controller`  and `manifests/k8s-extensions-scheduler` directories to `microk8s.io/cluster` based on k8s distributions of `kubeadm` and `microk8s` respectively
##### Deploy steps

The bootstrap script would facilitate the deployment of all SGX components at a usecase level. Sample one given below.

###### Update .env file

```shell
#Kubernetes Distribution microk8s or kubeadm
K8S_DISTRIBUTION=microk8s
K8S_MASTER_IP=<K8s control-plane IP>
K8S_MASTER_HOSTNAME=<K8s control-plane hostname>

# cms
CMS_SAN_LIST=cms-svc.isecl.svc.cluster.local,<K8s control-plane IP>,<K8s control-plane Hostname>

# authservice
AAS_API_CLUSTER_ENDPOINT_URL=https://<K8s control-plane IP>>:30444/aas/v1
AAS_ADMIN_USERNAME=admin@aas
AAS_ADMIN_PASSWORD=aasAdminPass
AAS_DB_USERNAME=aasdbuser
AAS_DB_PASSWORD=aasdbpassword
AAS_SAN_LIST=aas-svc.isecl.svc.cluster.local,<K8s control-plane IP>,<K8s control-plane Hostname>

# SGX Caching Service
SCS_ADMIN_USERNAME=scsuser@scs
SCS_ADMIN_PASSWORD=scspassword
SCS_DB_USERNAME=scsdbuser
SCS_DB_PASSWORD=scsdbpassword
# To be generated by user via Intel PCS server in case of new users, else the exisiting Provisioning key can be used
INTEL_PROVISIONING_SERVER_API_KEY=<provisioning server key>
SCS_CERT_SAN_LIST=scs-svc.isecl.svc.cluster.local,<K8s control-plane IP>,<K8s control-plane Hostname>

# SGX host verification service
SHVS_ADMIN_USERNAME=shvsuser@shvs
SHVS_ADMIN_PASSWORD=shvspassword
SHVS_DB_USERNAME=shvsdbuser
SHVS_DB_PASSWORD=shvsdbpassword
SHVS_CERT_SAN_LIST=shvs-svc.isecl.svc.cluster.local,K8s control-plane IP/Hostname>

# SGX Quote Verification Service bootstrap
SQVS_INCLUDE_TOKEN="true"
SQVS_NOSETUP="false"
#SGX_TRUSTED_ROOT_CA_FILE 
#For production Icelake CPUs SGX_TRUSTED_ROOT_CA_FILE = trusted_rootca_icx_prod.pem
#For production CascadeLake CPUs SGX_TRUSTED_ROOT_CA_FILE = trusted_rootca_clx_prod.pem
#For pre production Icelake CPUs SGX_TRUSTED_ROOT_CA_FILE = trusted_rootca.pem
SGX_TRUSTED_ROOT_CA_FILE=
SQVS_CERT_SAN_LIST=sqvs-svc.isecl.svc.cluster.local,<K8s control-plane IP>,<K8s control-plane Hostname>

# SGX Quote Verification Service bootstrap
SQVS_INCLUDE_TOKEN="true"
SQVS_NOSETUP="false"
#SGX_TRUSTED_ROOT_CA_FILE 
#For production Icelake CPUs SGX_TRUSTED_ROOT_CA_FILE = trusted_rootca_icx_prod.pem
#For production CascadeLake CPUs SGX_TRUSTED_ROOT_CA_FILE = trusted_rootca_clx_prod.pem
#For pre production Icelake CPUs SGX_TRUSTED_ROOT_CA_FILE = trusted_rootca.pem
SGX_TRUSTED_ROOT_CA_FILE=
SQVS_CERT_SAN_LIST=sqvs-svc.isecl.svc.cluster.local,<K8s control-plane IP>,<K8s control-plane Hostname>


# ihub bootstrap
IHUB_SERVICE_USERNAME=admin@hub
IHUB_SERVICE_PASSWORD=hubAdminPass
IH_CERT_SAN_LIST=ihub-svc.isecl.svc.cluster.local,<K8s control-plane IP>
# For microk8s
# K8S_API_SERVER_CERT=/var/snap/microk8s/current/certs/server.crt
K8S_API_SERVER_CERT=/var/snap/microk8s/current/certs/server.crt
IHUB_PUB_KEY_PATH=/etc/ihub/ihub_public_key.pem

# KBS bootstrap credentials
KBS_SERVICE_USERNAME=admin@kbs
KBS_SERVICE_PASSWORD=kbsAdminPass
# For SKC Virtualization use case set ENDPOINT_URL=https://<K8s control-plane IP>>:30448/v1
ENDPOINT_URL=https://kbs-svc.isecl.svc.cluster.local:9443/kbs/v1
KBS_CERT_SAN_LIST=kbs-svc.isecl.svc.cluster.local,<K8s control-plane IP>,<K8s control-plane Hostname>

KMIP_SERVER_IP=<KMIP Server IP>
KMIP_SERVER_PORT=<KMIP Server Port>
KMIP_HOSTNAME=<KMIP IP/Hostname>
#Provide either FQDN or KMIP SERVER IP for kmip hostname
#Default port for kmip server is 5696

# ISecl Scheduler
# For microk8s
# K8S_CA_KEY=/var/snap/microk8s/current/certs/ca.key
# K8S_CA_CERT=/var/snap/microk8s/current/certs/ca.crt
K8S_CA_KEY=/var/snap/microk8s/current/certs/ca.key
K8S_CA_CERT=/var/snap/microk8s/current/certs/ca.crt

#Skc Library
#Comment during fresh deploy,Uncomment this and update key id when deploying SKC Library
#KBS_PUBLIC_CERTIFICATE=<key id>.crt

SCS_SERVICE_USERNAME=scsuser@scs
SCS_SERVICE_PASSWORD=scspassword

SHVS_SERVICE_USERNAME=shvsuser@shvs
SHVS_SERVICE_PASSWORD=shvspassword

SQVS_SERVICE_USERNAME=sqvsuser@sqvs
SQVS_SERVICE_PASSWORD=sqvspassword

CCC_ADMIN_USERNAME=ccc_admin
CCC_ADMIN_PASSWORD=password

GLOBAL_ADMIN_USERNAME=global_admin_user
GLOBAL_ADMIN_PASSWORD=globalAdminPass

INSTALL_ADMIN_USERNAME=superadmin
INSTALL_ADMIN_PASSWORD=superAdminPass
```

###### Run scripts on K8s control-plane

* The bootstrap scripts are sample scripts to allow for a quick start of SKC services and agents. Users are free to modify the script or directly use the K8s manifests as per their deployment model requirements

```shell
#Pre-reqs.sh
./pre-requisites.sh

#skc-bootstrap-db-services
#Reference
#Usage: ./skc-bootstrap-db-services.sh [-help/up/purge]
#    -help          print help and exit
#    up        Bootstrap Database Services for Authservice, SGX Caching Service and SGX Host verification Service
#    purge     Delete Database Services for Authservice, SGX Caching Service and SGX Host verification Service
./skc-bootstrap-db-services.sh up

#skc-bootstrap
#Reference
#Usage: ./skc-bootstrap.sh [-help/up/down/purge]
#    -help                                     Print help and exit
#    up   [all/<agent>/<service>/<usecase>]    Bootstrap SKC K8s environment for specified agent/service/usecase
#    down [all/<agent>/<service>/<usecase>]    Delete SKC K8s environment for specified agent/service/usecase [will not delete data,config,logs]
#    purge                                     Delete SKC K8s environment with data,config,logs [only supported for single node deployments]

#    Available Options for up/down command:
#        agent      Can be one of sagent,skclib
#        service    Can be one of cms,authservice,scs,shvs,ihub,sqvs,kbs,isecl-controller,isecl-scheduler
#        usecase    Can be one of secure-key-caching,sgx-attestation,sgx-orchestration-k8s,sgx-virtualization,csp,enterprise
./skc-bootstrap.sh up <all/usecase of choice>
```


* Create and update `scheduler-policy.json` path

```shell
mkdir -p /opt/isecl-k8s-extensions
cp manifests/k8s-extensions-scheduler/config/scheduler-policy.json /opt/isecl-k8s-extensions
```


* Configure kube-scheduler to establish communication with isecl-scheduler.

```shell
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

* The K8s cluster admin configure the existing bare metal worker nodes or register fresh bare metal worker nodes with labels. For example, a label like `node.type: SGX-ENABLED` can be used by the cluster admin to distinguish the baremetal worker node and the same label can be used in ISECL Agent pod configuration to schedule on all worker nodes marked with the label. The same label is being used as default in the K8s manifests. This can be edited in `k8s/manifests/sgx_agent/daemonset.yml` , `k8s/manifests/skc_library/deployment.yml`
  ```shell
  #Label node
  kubectl label node <node-name> node.type=SGX-ENABLED
  ```

* `NFS` storage class is used in kubernetes environment for data persistence and supported in SKC. User needs to setup NFS server and create directory structure along with granting permission for a given user id. From security point of view, its been recommended to create a separate user id and grant the permission for all isecl directories for this user id. Below are some samples for reference

  * Snapshot showing directory structure for which user needs to create on NFS volumes manually or using custom scripts.

  ![NFS Directory Structure SKC Usecase](./images/nfs_skc_directory_structure.JPG)

  * Snapshot showing ownership and permissions for directories for which user needs to manually grant the ownership.

  ![NFS Directory Permissions SKC Usecase](./images/nfs_skc_permissions_dir.JPG)

  * Snapshot for configuring PV and PVC , user need to provide the NFS server IP or hostname and paths for each of the service directories. Sample manifest for creating `config-pv` for cms service

  ```
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

  Sample manifest for creating config-pvc for cms service

  ```
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

###### Manifests

* Update all the K8s manifests with the image names to be pulled from the registry

* The key field in `tolerations` and `nodeAffinity` in case of isecl-scheduler and isecl-controller needs to be updated in the respective manifests under the `manifests/k8s-extensions-controller`  and `manifests/k8s-extensions-scheduler` directories to `node-role.kubernetes.io/master
* All NFS PV yaml files needs to be updated with the  `path: /<NFS-vol-path>`  and `server: <NFS Server IP/Hostname>` under each service manifest file for `config`, `logs` , `db-data`

##### Deploy steps

###### Update .env file

```shell
#Kubernetes Distribution microk8s or kubeadm
K8S_DISTRIBUTION=kubeadm
K8S_CONTROL_PLANE_IP=<K8s control-plane IP>
K8S_CONTROL_PLANE_HOSTNAME=<K8s control-plane Hostname>

# cms
CMS_SAN_LIST=cms-svc.isecl.svc.cluster.local,<K8s control-plane IP>,<K8s control-plane Hostname>

# authservice
AAS_API_CLUSTER_ENDPOINT_URL=https://<K8s control-plane IP>:30444/aas/v1
AAS_ADMIN_USERNAME=admin@aas
AAS_ADMIN_PASSWORD=aasAdminPass
AAS_DB_USERNAME=aasdbuser
AAS_DB_PASSWORD=aasdbpassword
AAS_SAN_LIST=aas-svc.isecl.svc.cluster.local,<K8s control-plane IP>,<K8s control-plane Hostname>

# SGX Caching Service
SCS_ADMIN_USERNAME=scsuser@scs
SCS_ADMIN_PASSWORD=scspassword
SCS_DB_USERNAME=scsdbuser
SCS_DB_PASSWORD=scsdbpassword
# To be generated by user via Intel PCS server in case of new users, else the exisiting Provisioning key can be used
INTEL_PROVISIONING_SERVER_API_KEY=<provisioning server key>
SCS_CERT_SAN_LIST=scs-svc.isecl.svc.cluster.local,<K8s control-plane IP>,<K8s control-plane Hostname>

# SGX host verification service
SHVS_ADMIN_USERNAME=shvsuser@shvs
SHVS_ADMIN_PASSWORD=shvspassword
SHVS_DB_USERNAME=shvsdbuser
SHVS_DB_PASSWORD=shvsdbpassword
SHVS_CERT_SAN_LIST=shvs-svc.isecl.svc.cluster.local,<K8s control-plane IP>,<K8s control-plane Hostname>

# SGX Quote Verification Service bootstrap
SQVS_INCLUDE_TOKEN="true"
SQVS_NOSETUP="false"
#SGX_TRUSTED_ROOT_CA_FILE 
#For production Icelake CPUs SGX_TRUSTED_ROOT_CA_FILE = trusted_rootca_icx_prod.pem
#For production CascadeLake CPUs SGX_TRUSTED_ROOT_CA_FILE = trusted_rootca_clx_prod.pem
#For pre production Icelake CPUs SGX_TRUSTED_ROOT_CA_FILE = trusted_rootca.pem
SGX_TRUSTED_ROOT_CA_FILE=
SQVS_CERT_SAN_LIST=sqvs-svc.isecl.svc.cluster.local,<K8s control-plane IP>,<K8s control-plane Hostname>

# ihub bootstrap
IHUB_SERVICE_USERNAME=admin@hub
IHUB_SERVICE_PASSWORD=hubAdminPass
IH_CERT_SAN_LIST=ihub-svc.isecl.svc.cluster.local,<control-plane IP/Hostname>
# For Kubeadm
# K8S_API_SERVER_CERT=/etc/kubernetes/pki/apiserver.crt
K8S_API_SERVER_CERT=/etc/kubernetes/pki/apiserver.crt
# This is not valid for multinode deployment, should be populated once ihub is deployed successfully
IHUB_PUB_KEY_PATH=<skip this during initial deploy of K8s multi-node>

# KBS bootstrap credentials
KBS_SERVICE_USERNAME=admin@kbs
KBS_SERVICE_PASSWORD=kbsAdminPass
# For SKC Virtualization use case set ENDPOINT_URL=https://<K8s control-plane IP>:30448/v1
ENDPOINT_URL=https://kbs-svc.isecl.svc.cluster.local:9443/kbs/v1
KBS_CERT_SAN_LIST=kbs-svc.isecl.svc.cluster.local,<K8s control-plane IP>, <k8s control-plane Hostname>

KMIP_SERVER_IP=<KMIP Server IP>
KMIP_SERVER_PORT=<KMIP Server Port>
KMIP_HOSTNAME=<KMIP IP/Hostname>
#Provide either FQDN or KMIP SERVER IP for kmip hostname
#Default port for kmip server is 5696


# ISecl Scheduler
# For Kubeadm
# K8S_CA_KEY=/etc/kubernetes/pki/ca.key
# K8S_CA_CERT=/etc/kubernetes/pki/ca.crt
K8S_CA_KEY=/etc/kubernetes/pki/ca.key
K8S_CA_CERT=/etc/kubernetes/pki/ca.crt

#Skc Library
#Comment during fresh deploy,Uncomment this and update key id when deploying SKC Library
#KBS_PUBLIC_CERTIFICATE=<key id>.crt

SCS_SERVICE_USERNAME=scsuser@scs
SCS_SERVICE_PASSWORD=scspassword

SHVS_SERVICE_USERNAME=shvsuser@shvs
SHVS_SERVICE_PASSWORD=shvspassword

SQVS_SERVICE_USERNAME=sqvsuser@sqvs
SQVS_SERVICE_PASSWORD=sqvspassword

CCC_ADMIN_USERNAME=ccc_admin
CCC_ADMIN_PASSWORD=password

GLOBAL_ADMIN_USERNAME=global_admin_user
GLOBAL_ADMIN_PASSWORD=globalAdminPass

INSTALL_ADMIN_USERNAME=superadmin
INSTALL_ADMIN_PASSWORD=superAdminPass

```

###### Run scripts on K8s control-plane

* The bootstrap scripts are sample scripts to allow for a quick start of SKC services and agents. Users are free to modify the script or directly use the K8s manifests as per their deployment model requirements

```shell
#If using sample script provided for creating nfs directories
#Copy the script to the base path of the NFS location configured
chmod +x create-skc-dirs.nfs.sh
./create-skc-dirs.nfs.sh

#Pre-reqs.sh
./pre-requisites.sh

#skc-bootstrap-db-services
#Reference
#Usage: ./skc-bootstrap-db-services.sh [-help/up/purge]
#    -help          print help and exit
#    up        Bootstrap Database Services for Authservice, SGX Caching Service and SGX Host verification Service
#    purge     Delete Database Services for Authservice, SGX Caching Service and SGX Host verification Service
./skc-bootstrap-db-services.sh up

#skc-bootstrap
#Reference
#Usage: ./skc-bootstrap.sh [-help/up/down/purge]
#    -help                                     Print help and exit
#    up   [all/<agent>/<service>/<usecase>]    Bootstrap SKC K8s environment for specified agent/service/usecase
#    down [all/<agent>/<service>/<usecase>]    Delete SKC K8s environment for specified agent/service/usecase [will not delete data,config,logs]
#    purge                                     Delete SKC K8s environment with data,config,logs [only supported for single node deployments]

#    Available Options for up/down command:
#        agent      Can be one of sagent,skclib
#        service    Can be one of cms,authservice,scs,shvs,ihub,sqvs,kbs,isecl-controller,isecl-scheduler
#        usecase    Can be one of secure-key-caching,sgx-attestation,sgx-orchestration-k8s,sgx-virtualization,csp,enterprise
./skc-bootstrap.sh up <all/usecase of choice>

#NOTE: The isecl-scheduler will not be deployed in case of multi-node K8s as there is dependency on the NFS server IHUB public key to be copied from NFS to K8s control-plane to allow the successful installation of isecl-scheduler. Post update of the isecl-k8s-skc.env for IHUB_PUB_KEY_PATH on K8s control-plane, user needs to run the following
./skc-bootstrap.sh up isecl-scheduler
```

* Create and update `scheduler-policy.json` path

```shell
mkdir -p /opt/isecl-k8s-extensions
cp manifests/k8s-extensions-scheduler/config/scheduler-policy.json /opt/isecl-k8s-extensions
```

* Configure kube-scheduler to establish communication with isecl-scheduler. Add `scheduler-policy.json` under kube-scheduler section, `mountPath` under container section and `hostPath` under volumes section in` /etc/kubernetes/manifests/kube-scheduler.yaml` as mentioned below

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

#SGX Host Attestation Service
Config: /etc/shvs
Logs: /var/log/shvs
Pg-data: /usr/local/kube/data/shvs

#SGX Caching Service
Config: /etc/scs
Logs: /var/log/scs
Pg-data: /usr/local/kube/data/scs

#Integration-Hub
Config: /etc/ihub
Log: /var/log/ihub

#SGX Quote Verificaton Service
Config: /etc/sqvs
Logs: /var/log/sqvs

#Key-Broker-Service
Config: /etc/kbs
Log: /var/log/kbs
Opt: /opt/kbs

#SGX Library:
Config: /root/lib/resources/sgx_default_qcnl.conf
openssl-config: /etc/pki/tls/openssl.cnf
pkcs11-config: /opt/skc/etc/pkcs11-apimodule.ini
kms-npm-config: /opt/skc/etc/kms_npm.ini
sgx-stm-config: /opt/skc/etc/sgx_stm.ini
kms-cert: /root/9e9db4b5-5893-40fe-b6c4-d54ec6609c55.crt
haproxy-hosts: /etc/hosts
nginx-config: /etc/nginx/nginx.conf
skc-lib-config: /root/lib/skc_library.conf
kms-key: /tmp/keys.txt
nginx-logs: /var/log/sqvs/

#SGX Agent:
Config: /etc/sgx-agent
Logs: /var/log/sgx-agent
EFI: /sys/firmware/efi/efivars
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

#SGX Host Attestation Service
Config: <NFS-vol-base-path>/isecl/shvs/config
Logs: <NFS-vol-base-path>/isecl/shvs/logs
Pg-data: <NFS-vol-base-path>/usr/local/kube/data/shvs

#Integration-Hub
Config: <NFS-vol-base-path>/isecl/ihub/config
Log: <NFS-vol-base-path>/isecl/ihub/logs

#SGX Quote Verificaton Service
Config: <NFS-vol-base-path>/isecl/sqvs/config
Logs: <NFS-vol-base-path>/isecl/sqvs/log

#Key-Broker-Service
Config: <NFS-vol-base-path>/isecl/kbs/config
Log: <NFS-vol-base-path>/isecl/kbs/logs
Opt: <NFS-vol-base-path>/isecl/kbs/kbs/opt

#SGX Agent:
Config: /etc/sgx-agent
Logs: /var/log/sgx-agent
EFI: /sys/firmware/efi/efivars
```



## Default Service Ports

For both Single node and multi-node deployments following ports are used. All services exposing APIs will use the below ports

```yaml
CMS: 30445
AAS: 30444
SCS: 30501
SHVS: 30500
SQVS: 30502
IHUB: None
KBS: 30448
K8s-scheduler: 30888
K8s-controller: None
SKC Library: None
SGXAgent: 31001
```



## Usecase Workflows API Collections

The below allow to get started with workflows within Intel® SecL-DC for Foundational and Workload Security Usecases. More details available in [API Collections](https://github.com/intel-secl/utils/tree/v5.0/develop/tools/api-collections) repository

### Pre-requisites

* Postman client should be [downloaded](https://www.postman.com/downloads/) on supported platforms or on the web to get started with the usecase collections.

  >  **Note:** The Postman API Network will always have the latest released version of the API Collections. For all releases, refer the github repository for [API Collections](https://github.com/intel-secl/utils/tree/v5.0/develop/tools/api-collections)

### Use Case Collections


| Use case                     | Sub-Usecase | API Collection     |
| ---------------------------- | ----------- | ------------------ |
| Secure Key Caching           | -           | ✔️                  |
| SGX Discovery, Provisioning and Orchestration | -           | ✔️                  |

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
cd /root/isecl/build/skc-k8s-single-node
rm -rf .repo *
#Rebuild as before
repo init ...
repo sync
#To rebuild single node
make k8s-aio
#To rebuild multi node
make k8s
```
### SGX Attestation Flow
```
To Deploy SampleApp:
  Copy sample_apps.tar, sample_apps.sha2 and sampleapps_untar.sh from k8s/sample_apps directory to a directory in SGX compute node and untar it using './sample_apps_untar.sh'
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
### SKC Key Transfer Flow

Below steps to be followed post successful deployment with Single-Node/Multi-Node deployment

#### Generating keys

* From cluster node, copy `k8s/manifests/kbs/rsa_create.py` to KMIP server, update KMIP server IP in rsa_create.py inside single quote.

* On cluster node, navigate to `k8s/manifests/kbs/` 

* Update `kbs.conf` with `SYSTEM_IP`, `AAS_PORT`, `KBS_PORT` and `CMS_PORT`
  * `SYSTEM_IP` : K8s control-plane IP
  * `AAS_PORT` : k8s exposed port for AAS (default 30444)
  * `KBS_PORT` : k8s exposed port for KBS (default 30448)
  * `CMS_PORT` : k8s exposed port for CMS (default 30445)


* Generate the key using `./run.sh reg`

#### Setup configurations

* Copy generated key to `k8s/manifests/skc_library/resources`  

* Update `create_roles.conf` under `k8s/manifests/skc_library/resources` for `AAS_IP` to K8s control-plane & `AAS_PORT` to K8s pod exposed service port (Default:`30444`)

* Execute `skc_library_create_roles.sh` to create skc token

* Update below files available under `k8s/manifests/skc_library/resources` with required values
  * `hosts`
    * Update K8s control-plane IP and K8s control-plane Host Name where we generate KBS key
  * `keys.txt`
    * Update KBS Key ID
  * `nginx.conf`
    * Update KBS Key ID in both `ssl_certificate` and `ssl_certificate_key` lines.
  * `sgx_default_qcnl.conf`
    * Update K8s control-plane IP and SCS K8s Service Port
  * `kms_npm.ini`
    * Update K8s control-plane IP and KBS K8s Service Port
  * `skc_library.conf`
    * Update `KBS_HOSTNAME`, `KBS_IP` and `KBS_PORT` with K8s control-plane Hostname, K8s control-plane IP and  KBS K8s Service Port.
    * Update `CMS_IP` and `CMS_PORT` with K8s control-plane IP and CMS K8s Service Port.
    * Update `CSP_SCS_IP` and `CSP_SCS_PORT` with K8s control-plane IP and SCS K8s Service Port.
  
  >  **Note: ** Update skc token in the `skc_library.conf` generated in previous step
  
* Update `mountPath` and `subPath` with generated key id, along with image name in `k8s/manifests/skc_library/deployment.yml`  

* Update `isecl-skc-k8s.env` by uncommenting `KBS_PUBLIC_CERTIFICATE=<key id>.crt` and update the `<key id>` value

#### Initiate Key Transfer Flow

* Deploy the SKC Library using `./skc-bootstrap.sh up skclib`
* It will initiate the key transfer
>  **Note: ** To enable debug log, make `debug=true` in pkcs11-apimodule.ini, kms_npm.ini and sgx_stm.ini files available under `k8s/manifests/skc_library/resources`

* Establish tls session with the nginx using the key transferred inside the enclave
   `wget https://<K8s control-plane IP>:30443 --no-check-certificate`

#### Other Workloads

Secure Key Caching use case employs nginx web server as a workload to demonstrate confidential computing. If the need is to use other custom workloads, then following reference Dockerfile and entrypoint scripts can be used as starting point to deploy custom workload as a container.

Reference `Dockerfile` for Nginx with skc library
```
# Copyright (C) 2021 Intel Corporation
# SPDX-License-Identifier: BSD-3-Clause

FROM centos:8

# Set env variables
ENV SGX_INSTALL_DIR /opt/intel
ENV SKC_LIBRARY_BIN_NAME skc_library_*.bin

# Copy binaries and SGX local repo
COPY dist/image/bin/pkcs11.so /usr/lib64/engines-1.1/
COPY dist/image/bin/libp11.so.3.4.3 /usr/lib64/
COPY dist/image/bin/sgx_rpm_local_repo ${PWD}/sgx_rpm_local_repo
COPY dist/image/bin/${SKC_LIBRARY_BIN_NAME} $PWD
COPY dist/image/entrypoint.sh $PWD
RUN chmod 700 /entrypoint.sh
COPY dist/image/credential_agent.sh $PWD
RUN chmod 700 /credential_agent.sh
COPY dist/image/sgxssl ${SGX_INSTALL_DIR}/sgxssl
COPY dist/image/cryptoapitoolkit ${SGX_INSTALL_DIR}/cryptoapitoolkit

#Install dependencies and SGX components
RUN dnf install -y https://dl.fedoraproject.org/pub/fedora/linux/releases/33/Everything/x86_64/os/Packages/l/libgda-5.2.9-6.fc33.x86_64.rpm \
    https://dl.fedoraproject.org/pub/fedora/linux/releases/33/Everything/x86_64/os/Packages/l/libgda-sqlite-5.2.9-6.fc33.x86_64.rpm \
    https://download-ib01.fedoraproject.org/pub/epel/8/Everything/x86_64/Packages/j/jsoncpp-1.8.4-6.el8.x86_64.rpm \
    https://download-ib01.fedoraproject.org/pub/epel/8/Everything/x86_64/Packages/j/jsoncpp-devel-1.8.4-6.el8.x86_64.rpm
RUN dnf install -y yum-utils jq protobuf opensc nginx cronie sudo ncurses
RUN groupadd intel && \
    usermod -G intel nginx && \
    ln -sf /usr/lib64/libp11.so.3.4.3 /usr/lib64/libp11.so && \
    ln -sf /usr/lib64/engines-1.1/pkcs11.so /usr/lib64/engines-1.1/libpkcs11.so && \
    ln -sf /usr/lib64/libjsoncpp.so /usr/lib64/libjsoncpp.so.0
RUN yum-config-manager --add-repo file://${PWD}/sgx_rpm_local_repo && \
    dnf install -y --nogpgcheck libsgx-launch libsgx-uae-service libsgx-urts libsgx-ae-qve libsgx-dcap-ql libsgx-dcap-ql-devel \
    libsgx-dcap-default-qpl-devel libsgx-dcap-default-qpl && \
    rm -rf sgx_rpm_local_repo /etc/yum.repos.d/*sgx_rpm_local_repo.repo

# Install SKC-Library and copy entrypoint script
RUN ./${SKC_LIBRARY_BIN_NAME}
ENTRYPOINT ["./entrypoint.sh"]
```

Sample entrypoint script `entrypoint.sh`

```
#!/bin/bash

./credential_agent.sh
grep -qi "daemon off" /etc/nginx/nginx.conf || echo 'daemon off;' >> /etc/nginx/nginx.conf
nginx
```

### SGX Discovery Flow

* Create below sample pod.yml file for nginx workload and add SGX labels as node affinity to it.
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

* Launch the pod and validate if pod can be launched on the node. Run following commands:
```
    kubectl apply -f pod.yml
    kubectl get pods
    kubectl describe pod nginx
```

* Pod should be in running state and launched on the host as per values in yml. Validate by running below command on worker node:
```
	docker ps
```



### SKC Virtualization Flow

* Update the `populate-users.env` under `aas/scripts` directory for `SCS_CERT_SAN_LIST` and `SHVS_CERT_SAN_LIST` to add the SGX enabled node IP's
* Deploy all services on SGX Virtualization control plane
* Ensure to update `agent.conf` for deploying SGX agent service
  and run `./deploy_sgx_agent.sh` to deploy `SGX_AGENT` as binary deployment
* Deploy `SKC LIBRARY` on SGX Host, update `skc_library.conf` and execute
   `./deploy_skc_library.sh`
* Follow the [Generating keys](#generating-keys) section to generate the keys

* Copy the generated Key to SGX Host
* Update `keys.txt` and `nginx.conf`

* Initiate key transfer

#### Configuration for Key Transfer On SGX Virtualization Setup

> **Note:** Below mentioned OpenSSL and NGINX configuration updates are provided as patches (nginx.patch and openssl.patch) as part of skc_library deployment script.

##### OpenSSL

Update openssl configuration file `/etc/pki/tls/openssl.cnf` with below changes

```ini
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

##### Nginx

Update nginx configuration file `/etc/nginx/nginx.conf` with below changes

ssl_engine pkcs11;

Update the location of certificate with the loaction where it was copied into the skc_library machine. 

ssl_certificate "add absolute path of crt file";

Update the KeyID with the KeyID received when RSA key was generated in KBS

ssl_certificate_key "engine:pkcs11:pkcs11:token=KMS;id=164b41ae-be61-4c7c-a027-4a2ab1e5e4c4;object=RSAKEY;type=private;pin-value=1234";

##### SKC Configuration

* Create `keys.txt` in /root folder. This provides key preloading functionality in skc_library.

  > **Note:** Any number of keys can be added in keys.txt. Each PKCS11 URL should contain different Key ID which need to be transferred from KBS along with respective object tag for each key id specified

* Sample PKCS11 url is as below

  ```
  pkcs11:token=KMS;id=164b41ae-be61-4c7c-a027-4a2ab1e5e4c4;object=RSAKEY;type=private;pin-value=1234;
  ```

  > **Note:**  Last PKCS11 url entry in `keys.txt` should match with the one in `nginx.conf`

* The keyID should match the keyID of RSA key created in KBS. Other contents should match with `nginx.conf`. File location should match with `preload_keys directive` in `pkcs11-apimodule.ini`

* Sample `/opt/skc/etc/pkcs11-apimodule.ini` file
  	

  ```ini
  [core]
  preload_keys=/root/keys.txt
  keyagent_conf=/opt/skc/etc/key-agent.ini
  mode=SGX
  debug=true
  
  [SW]
  module=/usr/lib64/pkcs11/libsofthsm2.so
  
  [SGX]
  module=/opt/intel/cryptoapitoolkit/lib/libp11sgx.so
  ```

##### Key-transfer flow validation

* On SGX Compute node, Execute below commands for KBS key-transfer:

  ```shell
  pkill nginx
  ```

* Remove any existing pkcs11 token

  ```shell
   rm -rf /opt/intel/cryptoapitoolkit/tokens/*  
  ```

* Initiate Key transfer from KBS

  ```shell
  systemctl restart nginx
  ```

* Changing group ownership and permissions of pkcs11 token

  ```shell
  chown -R root:intel /opt/intel/cryptoapitoolkit/tokens/
  ```

  ```shell
  chmod -R 770 /opt/intel/cryptoapitoolkit/tokens/    
  ```

* Establish a tls session with the nginx using the key transferred inside the enclave

  ```shell
  wget https://localhost:2443 --no-check-certificate
  ```

##### Note on Key Transfer Policy

Key Transfer Policy is used to enforce a set of policies which need to be complied before the secret can be securely provisioned onto a sgx enclave

Sample Key Transfer Policy:
```yaml
"sgx_enclave_issuer_anyof":["cd171c56941c6ce49690b455f691d9c8a04c2e43e0a4d30f752fa5285c7ee57f"],
"sgx_enclave_issuer_product_id_anyof":[0],
"sgx_enclave_measurement_anyof":["7df0b7e815bd4b4af41239038d04a740daccf0beb412a2056c8d900b45b621fd"],
"tls_client_certificate_issuer_cn_anyof":["CMSCA", "CMS TLS Client CA"],
"client_permissions_allof":["nginx","USA"],
"sgx_enforce_tcb_up_to_date":false
```
   a.    `sgx_enclave_issuer_anyof` - Establishes the signing identity provided by an authority who has signed the SGX enclave. In other words the owner of the enclave.

   b.    `sgx_enclave_measurement_anyof` - Represents the cryptographic hash of the enclave log (enclave code, data)

   c.    `sgx_enforce_tcb_up_to_date` - If set to true, Key Broker service will provision the key only of the platform generating the quote conforms to the latest Trusted Computing Base

   d.    `client_permissions_allof `- Special permission embedded into the skc_library client TLS certificate which can enforce additional restrictions on who can get access to the key. In the above example, key is provisioned only to the nginx workload and platform which is tagged with value for ex: USA

##### Note on SKC Library Deployment

SKC Library Deployment needs to performed with root privilege

Each container instance of workload gets its own private SKC Client Library config information

The SKC Client Library TLS client certificate private key is stored in the configuration directories and can be read only with elevated root privileges
keys.txt (set of PKCS11 URIs for the keys to be securely provisioned into an SGX enclave) can only be modified with elevated privileges


##### Extracting SGX Enclave values for Key Transfer Policy

Values that are specific to the enclave such as `sgx_enclave_issuer_anyof`, `sgx_enclave_measurement_anyof` and `sgx_enclave_issuer_product_id_anyof` can be retrieved using `sgx_sign` utility that is available as part of Intel SGX SDK.

* Run `sgx_sign` utility on the signed enclave (This command should be run on the build system).

  ```shell
  /opt/intel/sgxsdk/bin/x64/sgx_sign dump -enclave <path to the signed enclave> -dumpfile info.txt
  ```

* For `sgx_enclave_issuer_anyof`, in info.txt, search for `mrsigner->value` . E.g.. mrsigner->value :
  ```shell
  mrsigner->value: "0x83 0xd7 0x19 0xe7 0x7d 0xea 0xca 0x14 0x70 0xf6 0xba 0xf6 0x2a 0x4d 0x77 0x43 0x03 0xc8 0x99 0xdb 0x69 0x02 0x0f 0x9c 0x70 0xee 0x1d 0xfc 0x08 0xc7 0xce 0x9e"
  ```
  Remove the whitespace and 0x characters from the above string and add it to the policy file. E.g.. :
  ```yaml
  "sgx_enclave_issuer_anyof":["83d719e77deaca1470f6baf62a4d774303c899db69020f9c70ee1dfc08c7ce9e"]
  ```

* For `sgx_enclave_measurement_anyof`, in info.txt, search for `metadata->enclave_css.body.enclave_hash.m` . E.g. metadata->enclave_css.body.enclave_hash.m :
  ```shell
  metadata->enclave_css.body.enclave_hash.m:
  0xad 0x46 0x74 0x9e 0xd4 0x1e 0xba 0xa2 0x32 0x72 0x52 0x04 0x1e 0xe7 0x46 0xd3
  0x79 0x1a 0x9f 0x24 0x31 0x83 0x0f 0xee 0x08 0x83 0xf7 0x99 0x3c 0xaf 0x31 0x6a
  ```
  Remove the whitespace and 0x characters from the above string and add it to the policy file. E.g :
  ```shell
  "sgx_enclave_measurement_anyof":["ad46749ed41ebaa2327252041ee746d3791a9f2431830fee0883f7993caf316a"]
  ```
Please note that the SGX Enclave measurement value will depend on the toolchain used to build and link the SGX enclave. Hence the SGX Enclave measurement value would differ across OS flavours.
For more details please refer https://github.com/intel/linux-sgx/tree/master/linux/reproducibility



### Setup Task Flow

Setup tasks flows have been updated to have K8s native flow to be more agnostic to K8s workflows. Following would be the flow for setup task

- User would create a new `configMap`  object with the environment variables specific to the setup task. The Setup task variables would be documented in the Product Guide
- Users can provide variables for more than one setup task
- If variables involve BEARER_TOKEN then user need to delete the corresponding secret using `kubectl delete secret <service_name>-secret -n isecl` , update the new token inside secrets.txt file and re-create the secret using `kubectl create secret generic <service_name>-secret --from-file=secrets.txt --namespace=isecl`, else update the variables in configMap.yml
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
./skc-bootstrap.sh purge <all/usecase of choice>

#Purge all db data,pods,deploy,cm,secrets
./skc-bootstrap-db-services.sh purge

#Comment/Remove the following lines from /var/snap/microk8s/current/args/kube-scheduler
--policy-config-file=/opt/isecl-k8s-extensions/scheduler-policy.json

#Restart kubelet
systemctl restart snap.microk8s.daemon-kubelet.service

#Setup fresh
./skc-bootstrap-db-services.sh up
./skc-bootstrap.sh up <all/usecase of choice>

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
./skc-bootstrap.sh down <all/usecase of choice>

#Comment/Remove the following lines from /var/snap/microk8s/current/args/kube-scheduler
--policy-config-file=/opt/isecl-k8s-extensions/scheduler-policy.json

#Restart kubelet
systemctl restart snap.microk8s.daemon-kubelet.service

#Setup fresh
./skc-bootstrap-db-services.sh up
./skc-bootstrap.sh up <all/usecase of choice>

#Reconfigure K8s-scheduler
vi /var/snap/microk8s/current/args/kube-scheduler

#Add the below line
--policy-config-file=/opt/isecl-k8s-extensions/scheduler-policy.json

#Restart kubelet
systemctl restart snap.microk8s.daemon-kubelet.service
```

In order to clean single service and bring up again on single node without data, config from previous deployment
```shell script
./skc-bootstrap.sh down <service-name>
rm -rf /etc/<service-name>
rm -rf /var/log/<service-name>

# Only in case of KBS, perform one more step along with above 2 steps
rm -rf /opt/kbs
./skc-bootstrap.sh up <service-name>
```

In order to redeploy again on single node with data, config from previous deployment
```shell
./skc-bootstrap.sh down <service-name>
./skc-bootstrap.sh up <service-name>
```

#### Multi-node

In order to cleanup and setup fresh again on multi-node with data,config from previous deployment

```shell
#Purge all data and pods,deploy,cm,secrets
./skc-bootstrap.sh down <all/usecase of choice>

#Delete 'scheduler-policy.json'
rm -rf /opt/isecl-k8s-extensions

#Comment/Remove '- --policy-config-file=...' from /etc/kubernetes/manifests/kube-scheduler.yaml as below
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
      
#Comment/Remove '- hostPath:...' from /etc/kubernetes/manifests/kube-scheduler.yaml as below
volumes:
  - hostPath:
      path: /opt/isecl-k8s-extensions/
      type:
    name: extendedsched

#Restart kubelet
systemctl restart kubelet

#Purge all db data,pods,deploy,cm,secrets
./skc-bootstrap-db-services.sh purge

#Cleanup all data from NFS share --> User controlled

#Cleanup data from each worker node --> User controlled
rm -rf /etc/sgx_agent
rm -rf /var/log/sgx_agent

#Setup fresh
./skc-bootstrap-db-services.sh up
./skc-bootstrap.sh up <all/usecase of choice>

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
./skc-bootstrap.sh down <all/usecase of choice>

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
./skc-bootstrap-db-services.sh up
./skc-bootstrap.sh up <all/usecase of choice>

#Setup fresh
./skc-bootstrap-db-services.sh up
./skc-bootstrap.sh up <all/usecase of choice>

#Copy ihub_public_key.pem from <NFS-PATH>/ihub/config/ to K8s control-plane and update IHUB_PUB_KEY_PATH in isecl-skc-k8s.env

#Bootstrap isecl-scheduler
./skc-bootstrap.sh up isecl-scheduler

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
```./skc-bootstrap.sh down <service-name>```
log into nfs system 
```shell script

rm -rf /<nfs-mount-path>/isecl/<service-name>/config
rm -rf /<nfs-mount-path>/isecl/<service-name>/logs

# Only in case of KBS, perform one more step along with above 2 steps
rm -rf /<nfs-mount-path>/isecl/<service-name>/opt
```
log into K8s control-plane
```./skc-bootstrap.sh up <service-name>```

In order to redeploy again on multi node with data, config from previous deployment
```shell
./skc-bootstrap.sh down <service-name>
./skc-bootstrap.sh up <service-name>

```
