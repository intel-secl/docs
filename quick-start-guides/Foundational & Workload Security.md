# Quick Start Guide - Foundational & Workload Security

Table of Contents
-----------------
   
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Quick Start Guide - Foundational & Workload Security](#quick-start-guide---foundational--workload-security)
  - [Table of Contents](#table-of-contents)
  - [**1. Hardware & OS Requirements**](#1-hardware--os-requirements)
    - [Physical Server requirements](#physical-server-requirements)
    - [OS Requirements](#os-requirements)
    - [User Access](#user-access)
  - [**2. Deployment Model**](#2-deployment-model)
  - [**3. Build Services and packages**](#3-build-services-and-packages)
    - [Pre-requisites](#pre-requisites)
    - [Building](#building)
      - [Foundational Security Usecase](#foundational-security-usecase)
      - [Workload Security Usecase](#workload-security-usecase)
        - [VM Confidentiality](#vm-confidentiality)
        - [Container Confidentiality with CRIO Runtime](#container-confidentiality-with-crio-runtime)
  - [**4. Deployment**](#4-deployment)
    - [Pre-requisites](#pre-requisites-1)
      - [Installing Ansible](#installing-ansible)
    - [Download the Ansible Role](#download-the-ansible-role)
    - [Usecase Setup Options](#usecase-setup-options)
    - [Update Ansible Inventory](#update-ansible-inventory)
    - [Create and Run Playbook](#create-and-run-playbook)
    - [Additional Examples & Tips](#additional-examples--tips)
      - [TBoot Installation](#tboot-installation)
      - [TPM is already owned](#tpm-is-already-owned)
      - [UEFI SecureBoot enabled](#uefi-secureboot-enabled)
      - [In case of Misconfigurations](#in-case-of-misconfigurations)
  - [**5. Usecase Workflows API Collections**](#5-usecase-workflows-api-collections)
    - [Pre-requisites](#pre-requisites-2)
    - [Use Case Collections](#use-case-collections)
    - [Downloading API Collections](#downloading-api-collections)
    - [Running API Collections](#running-api-collections)
  - [**Appendix**](#appendix)
    - [Running behind Proxy](#running-behind-proxy)
    - [Git Config Sample (~/.gitconfig)](#git-config-sample-gitconfig)
    - [Rebuilding Repos](#rebuilding-repos)
    - [Installing the Intel® SecL Kubernetes Extensions and Integration Hub](#installing-the-intel-secl-kubernetes-extensions-and-integration-hub)
      - [Deploy Intel® SecL Custom Controller](#deploy-intel-secl-custom-controller)
      - [Installing the Intel® SecL Integration Hub](#installing-the-intel-secl-integration-hub)
      - [Deploy Intel® SecL Extended Scheduler](#deploy-intel-secl-extended-scheduler)
      - [Configuring kube-scheduler to establish communication with isecl-scheduler](#configuring-kube-scheduler-to-establish-communication-with-isecl-scheduler)

<!-- /code_chunk_output -->



## **1. Hardware & OS Requirements**

### Physical Server requirements

* Intel® SecL-DC  supports and uses a variety of Intel security features, but there are some key requirements to consider before beginning an installation. Most important among these is the Root of Trust configuration. This involves deciding what combination of TXT, Boot Guard, tboot, and UEFI Secure Boot to enable on platforms that will be attested using Intel® SecL.

  > **Note:** At least one "Static Root of Trust" mechanism must be used (TXT and/or BtG). For Legacy BIOS systems, tboot must be used. For UEFI mode systems, UEFI SecureBoot must be used* Use the chart below for a guide to acceptable configuration options. Only dTPM is supported on Intel® SecL-DC platform hardware. 

  ![hardware-options](./images/trusted-boot-options.PNG)

### OS Requirements

* `RHEL 8.3` OS
* `rhel-8-for-x86_64-baseos-rpms` and `rhel-8-for-x86_64-appstream-rpms` repositories need to be enabled on build machine and remote machines
* Date and time should be in sync across the machines


### User Access

* The services need to be built & installed as `root` user. Ensure root privileges are present for the user to work with Intel® SecL-DC.
  
> **Note:** When using Ansible role for deployment, Ansible needs to be able to talk to remote machines as root user for successful deployment

* All Intel® SecL-DC service & agent ports should be allowed in firewall rules. 


## **2. Deployment Model**

![deploy_model](./images/isecl_deploy_model.PNG)

* Build + Deployment Machine

* CSP - ISecL Services Machine

* CSP - Physical Server as per supported configurations

* Enterprise - ISecL Services Machine
  

## **3. Build Services and packages**

The below steps needs to be carried out on the Build and Deployment VM

### Pre-requisites

* The repos can be built only as `root` user

* RHEL 8.3 Machine for building repos

* Enable the following RHEL repos:

  * `rhel-8-for-x86_64-appstream-rpms`
  * `rhel-8-for-x86_64-baseos-rpms`

* Install basic utilities for getting started

  ```shell
  dnf install git wget tar python3 yum-utils
  ```

* Create symlink for python3

  ```shell
  ln -s /usr/bin/python3 /usr/bin/python
  ln -s /usr/bin/pip3 /usr/bin/pip
  ```

* Install repo tool

  ```shell
  tmpdir=$(mktemp -d)
  git clone https://gerrit.googlesource.com/git-repo $tmpdir
  install -m 755 $tmpdir/repo /usr/local/bin
  rm -rf $tmpdir
  ```

* Golang installation
  
```shell
  wget https://dl.google.com/go/go1.16.7.linux-amd64.tar.gz
  tar -xzf go1.16.7.linux-amd64.tar.gz
  sudo mv go /usr/local
  export GOROOT=/usr/local/go
  export PATH=$GOROOT/bin:$PATH
  rm -rf go1.16.7.linux-amd64.tar.gz
  ```
  
### Building

#### Foundational Security Usecase

* Sync the repo

  ```shell
  mkdir -p /root/intel-secl/build/fs && cd /root/intel-secl/build/fs
  repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v5.0.0 -m manifest/fs.xml
  repo sync
  ```

* Run the `pre-requisites` setup script

  ```shell
  cd utils/build/foundational-security/
  chmod +x fs-prereq.sh
  ./fs-prereq.sh -s
  ```

* Build all repos

  ```shell
  cd /root/intel-secl/build/fs/
  make binaries
  ```

* Built Binaries

  ```shell
  /root/intel-secl/build/fs/binaries
  ```

#### Workload Security Usecase

##### VM Confidentiality

* Sync the repo

  ```shell
  mkdir -p /root/intel-secl/build/vmc && cd /root/intel-secl/build/vmc
  repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v5.0.0 -m manifest/vmc.xml
  repo sync
  ```

* Run the pre-req script

  ```shell
  cd utils/build/workload-security
  chmod +x ws-prereq.sh
  ./ws-prereq.sh -v
  ```
  
* Build repo

  ```shell
  cd /root/intel-secl/build/vmc/
  make all
  ```

* Built Binaries
  ```shell
  /root/intel-secl/build/vmc/binaries/
  ```

##### Container Confidentiality with CRIO Runtime

* Sync the repo

  ```shell
  mkdir -p /root/intel-secl/build/cc-crio && cd /root/intel-secl/build/cc-crio
  repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v5.0.0 -m manifest/cc-crio.xml
  repo sync
  ```

* Run the `pre-requisites` script

  ```shell
  cd utils/build/workload-security
  chmod +x ws-prereq.sh
  ./ws-prereq.sh -c
  ```
  
* Enable and start the Docker daemon

  ```shell
  systemctl enable docker
  systemctl start docker
  ```

* Ignore the below steps if not running behind a proxy

  ```shell
  mkdir -p /etc/systemd/system/docker.service.d
  touch /etc/systemd/system/docker.service.d/proxy.conf
  
  #Add the below lines in proxy.conf
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

* Download go dependencies

  ```shell
  cd /root/
  go get github.com/cpuguy83/go-md2man
  mv /root/go/bin/go-md2man /usr/bin/
  ```

* Build the repos

  ```shell
  cd /root/intel-secl/build/cc-crio
  make binaries
  ```
  
  > **Note:** The crio use case uses containerd that is bundled with `docker-ce-20.10.8` during build time. As of this release , the version being used is `containerd-1.4.4`. If the remote docker-ce repo gets updated for newer containerd version, then the version of containerd might be incompatible for building crio use case. It is recommended to use the version 1.4.4 in that case.
  
* Built binaries
  
  ```shell
  /root/intel-secl/build/cc-crio/binaries/
  ```

## **4. Deployment**

The below details would enable the deployment through Ansible Role for Intel® SecL-DC Foundational & Workload Security Usecases. However the services can still be installed manually using the Product Guide. More details on Ansible Role for Intel® SecL-DC in [Ansible-Role](https://github.com/intel-secl/utils/tree/v5.0/develop/tools/ansible-role) repository.


### Pre-requisites

* The Ansible Server is required to use this role to deploy Intel® SecL-DC services based on the supported deployment model. The Ansible server is recommended to be installed on the Build machine itself. 
* The role has been tested with Ansible Version `2.9.10`

#### Installing Ansible

* Install Ansible on Build Machine

  ```shell
  pip3 install ansible==2.9.10
  ```

* Install `epel-release` repository and install `sshpass` for ansible to connect to remote hosts using SSH

  ```shell
  dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
  dnf install sshpass
  ```

* Create directory for ansible default configuration and hosts file

  ```shell
  mkdir -p /etc/ansible/
  touch /etc/ansible/ansible.cfg
  ```

* Copy the default `ansible.cfg` contents from https://raw.githubusercontent.com/ansible/ansible/v2.9.10/examples/ansible.cfg and paste it under `/etc/ansible/ansible.cfg`


### Download the Ansible Role

The role can be cloned locally from git and the contents can be copied to the roles folder used by your ansible server 

```shell
#Create directory for using ansible deployment
mkdir -p /root/intel-secl/deploy/

#Clone the repository
cd /root/intel-secl/deploy/ && git clone https://github.com/intel-secl/utils.git

#Checkout to specific release-version
cd utils/
git checkout <release-version of choice>
cd tools/ansible-role

#Update ansible.cfg roles_path to point to path(/root/intel-secl/deploy/utils/tools/)
```

### Usecase Setup Options

| Usecase                                                      | Variable                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Host Attestation                                             | `setup: host-attestation` in playbook or via `--extra-vars` as `setup=host-attestation` in CLI |
| Application Integrity                                        | `setup: application-integrity` in playbook or via `--extra-vars` as `setup=application-integrity` in CLI |
| Data Fencing & Asset Tags                                    | `setup: data-fencing` in playbook or via `--extra-vars` as `setup=data-fencing` in CLI |
| Trusted Workload Placement - VM            | `setup: trusted-workload-placement-vm` in playbook or via `--extra-vars` as `setup=trusted-workload-placement-vm` in CLI |
| Trusted Workload Placement - Containers                      | `setup: trusted-workload-placement-containers` in playbook or via `--extra-vars` as `setup=trusted-workload-placement-containers` in CLI |
| Launch Time Protection - VM Confidentiality                  | `setup: workload-conf-vm` in playbook or via `--extra-vars` as `setup=workload-conf-vm` in CLI |
| Launch Time Protection - Container Confidentiality with CRIO Runtime | `setup: workload-conf-containers-crio` in playbook or via `--extra-vars` as `setup=workload-conf-containers-crio`in CLI |

> **Note:**  Orchestrator installation is not bundled with the role and need to be done independently. Also, components dependent on the orchestrator like `isecl-k8s-extensions` and `integration-hub` are installed either partially or not installed
>
> **Note:** `Key Broker Service` is not configured with KMIP compliant KMS when installing through ansible role  

### Update Ansible Inventory

In order to deploy Intel® SecL-DC binaries, the following inventory can be used and the required inventory vars as below need to be set. The below example inventory can be created under `/etc/ansible/hosts`

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

> **Note:** Ansible requires `ssh` and `root` user access to remote machines. The following command can be used to ensure ansible can connect to remote machines with host key check. Ensure the existing keys of the machines are cleared to enable fresh keyscan.
  ```shell
  ssh-keyscan -H <ip_address/hostname> >> /root/.ssh/known_hosts
  ```

### Create and Run Playbook

The following are playbook and CLI example for deploying Intel® SecL-DC binaries based on the supported deployment models and usecases. The below example playbooks can be created as `site-bin-isecl.yml`

> **Note :** If running behind a proxy, update the proxy variables under `vars/main.yml` and run as below

> **Note:** Go through the `Additional Examples and Tips` section for specific workflow samples

**Option 1**

```yaml
- hosts: all
  gather_facts: yes
  any_errors_fatal: true
  vars:
    setup: <setup var from supported usecases>
    binaries_path: <path where built binaries are copied to>
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
ansible-playbook <playbook-name> \
--extra-vars setup=<setup var from supported usecases> \
--extra-vars binaries_path=<path where built binaries are copied to>
```

### Additional Examples & Tips

#### TBoot Installation

Tboot needs to be built by the user from tboot source and the `tboot.gz` & `tboot-syms` files needs to be copied under the `binaries` folder. The supported version of Tboot as of 4.0 release is `tboot-1.10.1`.The options must then be provided during runtime in the playbook:

```shell
ansible-playbook <playbook-name> \
--extra-vars setup=<setup var from supported usecases> \
--extra-vars binaries_path=<path where built binaries are copied to> \
--extra-vars tboot_gz_file=<path where built binaries are copied to>/tboot.gz
--extra-vars tboot_syms_file=<path where built binaries are copied to>/tboot-syms
```

or 

Update the following in `vars/main.yml`

```yaml
# The TPM Storage Root Key(SRK) Password to be used if TPM is already owned
tboot_gz_file: "<binaries_path>/tboot.gz"
tboot_syms_file: "<binaries_path>/tboot-syms"
```

#### TPM is already owned

If the Trusted Platform Module(TPM) is already owned, the owner secret(SRK) can be provided directly during runtime in the playbook:

```shell
ansible-playbook <playbook-name> \
--extra-vars setup=<setup var from supported usecases> \
--extra-vars binaries_path=<path where built binaries are copied to> \
--extra-vars tpm_secret=<tpm owner secret>
```

or

Update the following vars in `vars/main.yml`

```yaml
# The TPM Storage Root Key(SRK) Password to be used if TPM is already owned
tpm_owner_secret: <tpm_secret>
```

#### UEFI SecureBoot enabled

If UEFI mode and UEFI SecureBoot feature is enabled, the following option can be used to during runtime in the playbook

```shell
ansible-playbook <playbook-name> \
--extra-vars setup=<setup var from supported usecases> \
--extra-vars binaries_path=<path where built binaries are copied to> \
--extra-vars uefi_secureboot=yes \
--extra-vars grub_file_path=<uefi mode grub file path>
```

or

Update the following vars in `vars/main.yml`

```yaml
# UEFI mode or UEFI SecureBoot mode
# ['no' - UEFI mode, 'yes' - UEFI SecureBoot mode]
uefi_secureboot: 'yes'

# The grub file path for UEFI Mode systems
# [/boot/efi/EFI/redhat/grub.cfg - UEFI Mode]
grub_file_path: /boot/efi/EFI/redhat/grub.cfg
```

#### In case of Misconfigurations 

If any service installation fails due to any misconfiguration, just uninstall the specific service manually , fix the misconfiguration in ansible and rerun the playbook. The successfully installed services wont be reinstalled.


## **5. Usecase Workflows API Collections**

The below allow to get started with workflows within Intel® SecL-DC for Foundational and Workload Security Usecases. More details available in [API Collections](https://github.com/intel-secl/utils/tree/v5.0/develop/tools/api-collections) repository

### Pre-requisites

* Postman client should be [downloaded](https://www.postman.com/downloads/) on supported platforms or on the web to get started with the usecase collections.

  >  **Note:** The Postman API Network will always have the latest released version of the API Collections. For all releases, refer the github repository for [API Collections](https://github.com/intel-secl/utils/tree/v5.0/develop/tools/api-collections)

### Use Case Collections

| Use case               | Sub-Usecase                                   | API Collection     |
| ---------------------- | --------------------------------------------- | ------------------ |
| Foundational Security  | Host Attestation(RHEL & VMWARE)                              | ✔️                  |
|                        | Data Fencing  with Asset Tags(RHEL & VMWARE)                 | ✔️                  |
|                        | Trusted Workload Placement (VM & Containers)  | ✔️ |
|                        | Application Integrity                         | ✔️                  |
| Launch Time Protection | VM Confidentiality                            | ✔️                  |
|                        | Container Confidentiality with CRIO Runtime   | ✔️                  |

> **Note: ** `Foundational Security - Host Attestation` is a pre-requisite for all usecases beyond Host Attestation. E.g: For working with `Launch Time Protection - VM Confidentiality` , Host Attestation flow must be run as a pre-req before trying VM Confidentiality

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


## **Appendix**

### Running behind Proxy

```shell
#Set proxy in ~/.bash_profile
export http_proxy=<proxy-url>
export https_proxy=<proxy-url>
export no_proxy=<ip_address/hostname>
```

### Git Config Sample (~/.gitconfig)

```
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
cd /root/isec/fs
rm -rf * .repo
```

### Installing the Intel® SecL Kubernetes Extensions and Integration Hub

Intel® SecL uses Custom Resource Definitions to add the ability to base
orchestration decisions on Intel® SecL security attributes to
Kubernetes. These CRDs allow Kubernetes administrators to configure pods
to require specific security attributes so that the Kubernetes Control Plane
Node will schedule those pods only on Worker Nodes that match the
specified attributes.

Two CRDs are required for integration with Intel® SecL – an extension
for the Control Plane nodes, and a scheduler extension. The extensions are deployed as a Kubernetes
deployment in the `isecl` namespace.


#### Deploy Intel® SecL Custom Controller

1.  Copy `isecl-k8s-extensions-*.tar.gz` to Kubernetes Control plane machine and extract the contents
    
    ```shell
    #Copy
    scp /<build_path>/binaries/isecl-k8s-extensions-*.tar.gz <user>@<k8s_master_machine>:/<path>/
    
    #Extract
    tar -xvzf /<path>/isecl-k8s-extensions-*.tar.gz
    cd /<path>/isecl-k8s-extensions/
    ```
    
2. Create `hostattributes.crd.isecl.intel.com` CRD

   ```shell
   #1.14<=k8s_version<=1.16
   kubectl apply -f yamls/crd-1.14.yaml
   
   #1.16<=k8s_version<=1.18
   kubectl apply -f yamls/crd-1.17.yaml
   ```

3. Check whether the CRD is created

   ```shell
   kubectl get crds
   ```

4. Load the `isecl-controller` docker image

   ```shell
   #Install skopeo to load docker image for controller and scheduler from archive
   dnf install -y skopeo
   
   #Push image to registry
   cd /<path>/isecl-k8s-extensions/
   skopeo copy oci-archive:<isecl-k8s-controller-*.tar> docker://<docker_private_registry_server>:5000/<imageName>:<tagName>
   ```

5. Update image name as above in `/opt/isecl-k8s-extensions/yamls/isecl-controller.yaml`
   ``` shell
      containers:
        - name: isecl-controller
          image: <docker_private_registry_server>:5000/<imageName>:<tagName>
   ```

6. Deploy `isecl-controller`

   ```shell
   kubectl apply -f yamls/isecl-controller.yaml
   ```

7. Check whether the isecl-controller is up and running

   ```shell
   kubectl get deploy -n isecl
   ```

8. Create `clusterRoleBinding` for ihub to get access to cluster nodes

   ```shell
   kubectl create clusterrolebinding isecl-clusterrole --clusterrole=system:node --user=system:serviceaccount:isecl:isecl
   ```

9. Fetch token required for ihub installation

   ```shell
   kubectl get secrets -n isecl
   
   #The below token will be used for ihub installation when configured with Kubernetes Tenant
   kubectl describe secret default-token-<name> -n isecl
   ```

10. Additional Optional Configurable fields for isecl-controller configuration in `isecl-controller.yaml`

| Field                 | Required   | Type     | Default | Description                                                  |
| --------------------- | ---------- | -------- | ------- | ------------------------------------------------------------ |
| LOG_LEVEL             | `Optional` | `string` | INFO    | Determines the log level                                     |
| LOG_MAX_LENGTH        | `Optional` | `int`    | 1500    | Determines the maximum length of characters in a line in log file |
| TAG_PREFIX            | `Optional` | `string` | isecl   | A custom prefix which can be applied to isecl attributes that are pushed from IH. For example, if the tag-prefix is **isecl.** and **trusted** attribute in CRD becomes **isecl.trusted**. |
| TAINT_UNTRUSTED_NODES | `Optional` | `string` | false   | If set to true. NoExec taint applied to the nodes for which trust status is set to false, Applicable only for HVS based attestation |



#### Installing the Intel® SecL Integration Hub

1. Copy the API Server certificate of K8s Master to machine where Integration Hub will be installed to `/root/` directory

   >  **Note:**  In most  Kubernetes distributions the Kubernetes certificate and key is normally present under `/etc/kubernetes/pki`. However this might differ in case of some specific Kubernetes distributions.

2. Update the token obtained in  Step 8 of `Deploy Intel® SecL Custom Controller` along with other relevant tenant configuration options in `ihub.env`

3. Install Integration Hub

4. Copy the `/etc/ihub/ihub_public_key.pem` to Kubernetes Master machine to `/<path>/secrets/` directory

   ```shell
   #On K8s-Master machine
   mkdir -p /<path>/secrets
   
   #On IHUB machine, copy
   scp /etc/ihub/ihub_public_key.pem <user>@<k8s_master_machine>:/<path>/secrets/hvs_ihub_public_key.pem
   ```



#### Deploy Intel® SecL Extended Scheduler

1. Install `cfssl` and `cfssljson` on Kubernetes Control Plane

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

2. Create TLS key-pair for `isecl-scheduler` service which is signed by Kubernetes `apiserver.crt`

   ```shell
   cd /<path>/isecl-k8s-extensions/
   chmod +x create_k8s_extsched_cert.sh
   
   #Set K8s_MASTER_IP,HOSTNAME
   export MASTER_IP=<k8s_machine_ip>
   export HOSTNAME=<k8s_machine_hostname>
   
   #Create TLS key-pair
   ./create_k8s_extsched_cert.sh -n "K8S Extended Scheduler" -s "$MASTER_IP","$HOSTNAME" -c <k8s_ca_authority_cert> -k <k8s_ca_authority_key>
   ```

   > **Note:**  In most  Kubernetes distributions the Kubernetes certificate and key is normally present under `/etc/kubernetes/pki`. However this might differ in case of some specific Kubernetes distributions.

3. Copy the TLS key-pair generated to `/<path>/secrets/` directory

   ```shell
   cp /<path>/isecl-k8s-extensions/server.key /<path>/secrets/
   cp /<path>/isecl-k8s-extensions/server.crt /<path>/secrets/
   ```

4. Load the `isecl-scheduler` docker image

   ```shell
   #Push image to registry
   cd /<path>/isecl-k8s-extensions/
   skopeo copy oci-archive:<isecl-k8s-scheduler-*.tar> docker://<docker_private_registry_server>:5000/<imageName>:<tagName>
   ```

5. Update image name as above in `/opt/isecl-k8s-extensions/yamls/isecl-scheduler.yaml`
   ``` shell
      containers:
        - name: isecl-scheduler
          image: <docker_private_registry_server>:5000/<imageName>:<tagName>
   ```

6. Create scheduler-secret for isecl-scheduler

   ```shell
   cd /<path>/
   kubectl create secret generic scheduler-certs --namespace isecl --from-file=secrets
   ```

7. The `isecl-scheduler.yaml` file includes support for both SGX and Workload Security put together. For only working with Workload Security scenarios , the following line needs to be made empty in the yaml file. The scheduler and controller yaml files are located under `/<path>/isecl-k8s-extensions/yamls`

   ```yaml
   - name: SGX_IHUB_PUBLIC_KEY_PATH
     value: ""
   ```

8. Deploy `isecl-scheduler`

   ```shell
   cd /<path>/isecl-k8s-extensions/
   kubectl apply -f yamls/isecl-scheduler.yaml      
   ```

9. Check whether the `isecl-scheduler` is up and running

   ```shell
   kubectl get deploy -n isecl
   ```

10. Additional optional fields for isecl-scheduler configuration in `isecl-scheduler.yaml`

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

> **Note:** The below is a sample when using `kubeadm` as the Kubernetes distribution, the scheduler configuration files would be different for any other Kubernetes distributions being used.

1.  Add a mount path to the
    `/etc/kubernetes/manifests/kube-scheduler.yaml` file for the Intel
    SecL scheduler extension:

    ```yaml
    - mountPath: /<path>/isecl-k8s-extensions/
      name: extendedsched
      readOnly: true
    ```

2. Add a volume path to the
   `/etc/kubernetes/manifests/kube-scheduler.yaml` file for the Intel
   SecL scheduler extension:

   ```yaml
   - hostPath:
       path: /<path>/isecl-k8s-extensions/
       type: ""
     name: extendedsched
   ```

3. Add `policy-config-file` path in the
   `/etc/kubernetes/manifests/kube-scheduler.yaml` file under `command` section:

   ```yaml
   - command:
     - kube-scheduler
     - --policy-config-file=/<path>/isecl-k8s-extensions/scheduler-policy.json
     - --bind-address=127.0.0.1
     - --kubeconfig=/etc/kubernetes/scheduler.conf
     - --leader-elect=true
   ```

4. Restart kubelet 

   ```shell
   systemctl restart kubelet
   ```
