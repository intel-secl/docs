# **SGX Attestation Infrastructure and Secure Key Caching (SKC) Quick Start Guide for Ubuntu**

Table of Contents

- [**SGX Attestation Infrastructure and Secure Key Caching (SKC) Quick Start Guide for Ubuntu**](#sgx-attestation-infrastructure-and-secure-key-caching-skc-quick-start-guide-for-ubuntu)
  - [**1. Hardware & OS Requirements**](#1-hardware--os-requirements)
      - [Four Hosts or VMs](#four-hosts-or-vms)
      - [SGX Enabled Host](#sgx-enabled-host)
    - [OS Requirements](#os-requirements)
  - [**2. Network Requirements**](#2-network-requirements)
        - [Build System](#build-system)
        - [CSP Managed Services](#csp-managed-services)
        - [Enterprise Managed Services](#enterprise-managed-services)
        - [SGX Enabled Host](#sgx-enabled-host-1)
  - [**3. Ubuntu Package Requirements**](#3-ubuntu-package-requirements)
  - [**4. Deployment Model**](#4-deployment-model)
  - [**5. Ubuntu System Tools and Utilities**](#5-ubuntu-system-tools-and-utilities)
  - [**6. Build Services, Libraries and Install packages**](#6-build-services-libraries-and-install-packages)
  - [**7. Deployment & Usecase Workflow Tools Installation**](#7-deployment--usecase-workflow-tools-installation)
    - [Usecases Workflow Tools Installation](#usecases-workflow-tools-installation)
  - [**8. Deployment**](#8-deployment)
    - [Deployment Using Ansible](#deployment-using-ansible)
    - [Download the Ansible Role](#download-the-ansible-role)
    - [Update Ansible Inventory](#update-ansible-inventory)
    - [Create and Run Playbook](#create-and-run-playbook)
    - [Usecase Setup Options](#usecase-setup-options)
  - [Deployment Using Binaries](#deployment-using-binaries)
    - [Setup K8S Cluster and Deploy Isecl-k8s-extensions](#setup-k8s-cluster-and-deploy-isecl-k8s-extensions)
    - [Untar packages and push OCI images to registry](#untar-packages-and-push-oci-images-to-registry)
        - [Deploy isecl-controller](#deploy-isecl-controller)
        - [Deploy isecl-scheduler](#deploy-isecl-scheduler)
        - [Configure kube-scheduler to establish communication with isecl-scheduler](#configure-kube-scheduler-to-establish-communication-with-isecl-scheduler)
      - [Deploying SKC Services on Single System](#deploying-skc-services-on-single-system)
      - [Deploy CSP SKC Services](#deploy-csp-skc-services)
      - [Deploy Enterprise SKC Services](#deploy-enterprise-skc-services)
      - [Deploy SGX Agent](#deploy-sgx-agent)
      - [Deploy SKC Library](#deploy-skc-library)
  - [**9. Usecase Workflows with Postman API Collections**](#9-usecase-workflows-with-postman-api-collections)
    - [Use Case Collections](#use-case-collections)
    - [Download Postman API Collections](#download-postman-api-collections)
    - [Running API Collections](#running-api-collections)
  - [**10. System User Configuration**](#10-system-user-configuration)
  - [Appendix](#appendix)
    - [SGX Attestation flow](#sgx-attestation-flow)
    - [Creating RSA Keys in Key Broker Service](#creating-rsa-keys-in-key-broker-service)
  - [Configuration for NGINX testing](#configuration-for-nginx-testing)
- [KBS key-transfer flow validation](#kbs-key-transfer-flow-validation)
  - [Note on Key Transfer Policy](#note-on-key-transfer-policy)
  - [Note on SKC Library Deployment](#note-on-skc-library-deployment)
  - [Extracting SGX Enclave values for Key Transfer Policy](#extracting-sgx-enclave-values-for-key-transfer-policy)

## **1. Hardware & OS Requirements**

#### Four Hosts or VMs

   Build System

   CSP managed Services

   Enterprise Managed Services

   Orchestrator Node Setup

#### SGX Enabled Host

### OS Requirements

   UBUNTU 18.04. SKC Solution is built, installed and tested with root privileges. Please ensure that all the following instructions are executed with root privileges

## **2. Network Requirements**

Internet access is required for the following

##### Build System

##### CSP Managed Services

##### Enterprise Managed Services

##### SGX Enabled Host


**Setting Proxy and No Proxy**

```
export http_proxy=http://<proxy-url>:<proxy-port>
export https_proxy=http://<proxy-url>:<proxy-port>
export no_proxy=0.0.0.0,127.0.0.1,localhost,<CSP IP>,<Enterprise IP>, <SGX Compute Node IP>, <KBS system Hostname>
```

**Firewall Settings**

Ensure that all the SKC service ports are accessible with firewall


## **3. Ubuntu Package Requirements**

Access required for the postgresql repo in all systems, through below steps:

**Create the file repository configuration**
```
sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
```

**Import the repository signing key**
```
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
```

**Update the package lists**
```
apt-get update
```

## **4. Deployment Model**

![deploy_model](./images/isecl_deploy_model.PNG)

* Build + Deployment Machine

* CSP - ISecL Services Machine

* Physical Server as per supported configurations

* Enterprise - ISecL Services Machine


## **5. Ubuntu System Tools and Utilities**

**Ubuntu System Tools and utils**

```
apt-get install -y software-properties-common git gcc zip wget make python3 python3-yaml python3-pip tar lsof jq nginx curl libssl-dev
ln -s /usr/bin/python3 /usr/bin/python
ln -s /usr/bin/pip3 /usr/bin/pip
add-apt-repository ppa:projectatomic/ppa
apt-get update
apt-get install skopeo
apt-get install makeself
export PATH=/usr/local/bin:$PATH
```

> Note: After skopeo installation, if /etc/containers/policy.json file not available then follow below steps:
```
mkdir /etc/containers
touch /etc/containers/policy.json 

#Add below contents in policy.json
{
    "default": [
        {
            "type": "insecureAcceptAnything"
        }
    ],
    "transports":
        {
            "docker-daemon":
                {
                    "": [{"type":"insecureAcceptAnything"}]
                }
        }
}
```

***Repo Tool***

```
tmpdir=$(mktemp -d)
git clone https://gerrit.googlesource.com/git-repo $tmpdir
install -m 755 $tmpdir/repo /usr/local/bin
rm -rf $tmpdir
```

***Golang Installation***

```
wget https://dl.google.com/go/go1.14.1.linux-amd64.tar.gz
tar -xzf go1.14.1.linux-amd64.tar.gz
sudo mv go /usr/local
export GOROOT=/usr/local/go
export PATH=$GOROOT/bin:$PATH
rm -rf go1.14.1.linux-amd64.tar.gz
```

***To Generate Swagger Documentation for components, Java Runtime needs to be installed***

```
apt-get install -y default-jre
```

## **6. Build Services, Libraries and Install packages**

Note: currently, the repos contain the source code of both the SGX Attestation Infrastructure and SKC. Make will build and package all the binaries and installation scripts  but the SGX Attestation Infrastructure can be installed and deployed separately. SKC cannot be installed without the SGX Attestation Infrastructure. 

The rest of this document will indicate steps that are only needed for SKC. 

**Pulling Source Code**

```
mkdir -p /root/workspace && cd /root/workspace
repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v4.0.2 -m manifest/skc.xml
repo sync
```

**Install, Enable and start the Docker daemon**

  ```shell
  wget https://download.docker.com/linux/ubuntu/dists/bionic/pool/stable/amd64/containerd.io_1.2.10-3_amd64.deb 
  dpkg -i containerd.io_1.2.10-3_amd64.deb
  wget "https://download.docker.com/linux/ubuntu/dists/bionic/pool/stable/amd64/docker-ce-cli_19.03.5~3-0~ubuntu-bionic_amd64.deb"
  dpkg -i docker-ce-cli_19.03.5~3-0~ubuntu-bionic_amd64.deb
  wget "https://download.docker.com/linux/ubuntu/dists/bionic/pool/stable/amd64/docker-ce_19.03.5~3-0~ubuntu-bionic_amd64.deb"
  dpkg -i docker-ce_19.03.5~3-0~ubuntu-bionic_amd64.deb

  systemctl enable docker
  systemctl start docker

  ```

**Ignore the below steps if not running behind a proxy**

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

**Building All SKC Components**

```
make

```

**Copy Binaries to a clean folder**

```
For CSP/Enterprise Deployment Model, copy the generated binaries directory to the /root directory on the CSP/Enterprise system
For Single system model, copy the generated binaries directory to the /root directory on the deployment system
```

## **7. Deployment & Usecase Workflow Tools Installation**

The below installation is required on the Build & Deployment system only and the Platform(Windows,Linux or MacOS) for Usecase Workflow Tool Installation

**Deployment Tools Installation**

* Install Ansible on Build Machine

  ```shell
  pip3 install ansible==2.9.10
  ```

* Install `sshpass` for ansible to connect to remote hosts using SSH

  ```shell
  apt-get install sshpass
  ```

* Create directory for ansible default configuration and hosts file

  ```shell
  mkdir -p /etc/ansible/
  touch /etc/ansible/ansible.cfg
  ```

* Copy the `ansible.cfg` contents from https://raw.githubusercontent.com/ansible/ansible/v2.9.10/examples/ansible.cfg and paste it under `/etc/ansible/ansible.cfg`


### Usecases Workflow Tools Installation

* Postman client should be [downloaded](https://www.postman.com/downloads/) on supported platforms or on the web to get started with the usecase collections.

  > **Note:** The Postman API Network will always have the latest released version of the API Collections. For all releases, refer the github repository for [API Collections](https://github.com/intel-secl/utils/tree/v4.0.2/develop/tools/api-collections)


## **8. Deployment**

### Deployment Using Ansible

The below details would enable the deployment through Ansible Role for Intel® SecL-DC Secure Key Caching Usecase. However the services can still be installed manually using the Product Guide. More details on Ansible Role for Intel® SecL-DC in [Ansible-Role](https://github.com/intel-secl/utils/tree/v4.0.2/develop/tools/ansible-role) repository.

### Download the Ansible Role 

The role can be cloned locally from git and the contents can be copied to the roles folder used by your ansible server 

```shell
#Create directory for using ansible deployment
mkdir -p /root/intel-secl/deploy/

#Clone the repository
cd /root/intel-secl/deploy/ && git clone https://github.com/intel-secl/utils.git

#Checkout to specific release version
cd utils/
git checkout <release-version of choice>
cd tools/ansible-role

#Update ansible.cfg roles_path to point to path(/root/intel-secl/deploy/utils/tools/ansible-role/roles/)
```

### Update Ansible Inventory

The following inventory can be used and created under `/etc/ansible/hosts`

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

> **Note:** Go through the `Additional Examples and Tips` section for specific workflow samples

**Option 1**

Update the PCS Server key with following vars in `<path to ansible role>/ansible-role/defaults/main.yml`

```yaml
  intel_provisioning_server_api_key_sandbox: <pcs server key>
```

Create playbook with following contents

```yaml
- hosts: all
  remote_user: root
  become: yes
  become_user: root
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
  remote_user: root
  become: yes
  become_user: root
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


### Usecase Setup Options

| Usecase                      | Variable                                                     |
| ---------------------------- | ------------------------------------------------------------ |
| Secure Key Caching           | `setup: secure-key-caching` in playbook or via `--extra-vars` as `setup=secure-key-caching`in CLI |
| SGX Orchestration Kubernetes | `setup: sgx-orchestration-kubernetes` in playbook or via `--extra-vars` as `setup=sgx-orchestration-kubernetes`in CLI |
| SKC No Orchestration         | `setup: skc-no-orchestration` in playbook or via `--extra-vars` as `setup=skc-no-orchestration`in CLI |
| SGX Attestation Kubernetes   | `setup: sgx-attestation-kubernetes` in playbook or via `--extra-vars` as `setup=sgx-attestation-kubernetes`in CLI |
| SGX Attestation No Orchestration | `setup: sgx-attestation-no-orchestration` in playbook or via `--extra-vars` as `setup=sgx-attestation-no-orchestration`in CLI |


> **Note:**  Orchestrator installation is not bundled with the role and need to be done independently. Also, components dependent on the orchestrator like `isecl-k8s-extensions` and `integration-hub` are installed either partially or not installed

## Deployment Using Binaries

### Setup K8S Cluster and Deploy Isecl-k8s-extensions

* Setup master and worker node for k8s. Worker node should be setup on SGX enabled host machine. Master node can be any system.

* To setup k8 cluster follow https://phoenixnap.com/kb/install-kubernetes-on-ubuntu Once the master/worker setup is done, follow below steps on Master Node:

### Untar packages and push OCI images to registry

* Copy tar output isecl-k8s-extensions-*.tar.gz from build system's binaries folder to /opt/ directory on the Master Node and extract the contents.
  
  ```shell
    cd /opt/
    tar -xvzf isecl-k8s-extensions-*.tar.gz
    cd isecl-k8s-extensions/
  ```
  
* Configure private registry

* Push images to private registry using skopeo command, (this can be done from build vm also)
  
  ```shell
     skopeo copy oci-archive:isecl-k8s-controller-v4.0.2-<commitid>.tar docker://<registryIP>:<registryPort>/isecl-k8s-controller:v4.0.2
     skopeo copy oci-archive:isecl-k8s-scheduler-v4.0.2-<commitid>.tar docker://<registryIP>:<registryPort>/isecl-k8s-scheduler:v4.0.2
  ```
  
* Add the image names in isecl-controller.yml and isecl-scheduler.yml in /opt/isecl-k8s-extensions/yamls with full image name including registry IP/hostname (e.g <registryIP>:<registryPort>/isecl-k8s-scheduler:v4.0.2). It will automatically pull the images from registry.
  

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

#### Deploying SKC Services on Single System
```
Copy the binaries directory generated in the build system to the /root/ directory on the deployment system
Update orchestrator.conf with the following
  - Deployment system IP address
  - SAN List (a list of ip address and hostname for the deployment system)
  - Network Port numbers for CMS, AAS, SCS and SHVS
  - Install Admin and CSP Admin credentials
  - TENANT as KUBERNETES
  - System IP address where Kubernetes deployed
  - Netowrk Port Number of Kubernetes
  - Database name, Database username and password for SHVS
Update enterprise_skc.conf with the following
  - Deployment system IP address
  - SAN List (a list of ip address and hostname for the deployment system)
  - Network Port numbers for CMS, AAS, SCS, SQVS and KBS
  - Install Admin and CSP Admin credentials
  - Database name, Database username and password for AAS and SCS services
  - Intel PCS Server API URL and API Keys
  - Key Manager can be set to either Directory or KMIP
  - KMIP server configuration if KMIP is set
Save and Close
./install_skc.sh
```

In case installation fails, its recommended to run the following command to clear failed service instance

```
systemctl reset-failed 
```

#### Deploy CSP SKC Services
```
Copy the binaries directory generated in the build system system to the /root/ directory on the CSP system
Update csp_skc.conf with the following
  - CSP system IP Address
  - SAN List (a list of ip address and hostname for the CSP system)
  - Network Port numbers for CMS, AAS, SCS and SHVS
  - Install Admin and CSP Admin credentials
  - TENANT as KUBERNETES 
  - System IP address where Kubernetes is deployed
  - Netowrk Port Number of Kubernetes
  - Database name, Database username and password for AAS, SCS and SHVS services
  - Intel PCS Server API URL and API Keys
Save and Close
./install_csp_skc.sh
```

In case installation fails, its recommended to run the following command to clear failed service instance

```
systemctl reset-failed 
```

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
#### Deploy Enterprise SKC Services
```
Copy the binaries directory generated in the build system to the /root/ directory on Enterprise system
Update enterprise_skc.conf with the following
  - Enterprise system IP address
  - SAN List (a list of ip address and hostname for the Enterprise system)
  - Network Port numbers for CMS, AAS, SCS, SQVS and KBS
  - Install Admin credentials
  - Database name, Database username and passwords for AAS and SCS services
  - Intel PCS Server API URL and API Keys
  - Key Manager can be set to either Directory or KMIP
  - KMIP server configuration if KMIP is set
Save and Close
./install_enterprise_skc.sh
```

#### Deploy SGX Agent
```
Copy sgx_agent.tar, sgx_agent.sha2 and agent_untar.sh from binaries directoy to a directory in SGX compute node

./agent_untar.sh

Edit agent.conf with the following
  - CSP system IP address where CMS, AAS, SHVS and SCS services deployed
  - CSP Admin credentials (same which are provided in service configuration file. for ex: csp_skc.conf, orchestrator.conf or skc.conf)
  - Network Port numbers for CMS, AAS, SCS and SHVS
  - Token validity period in days
  - CMS TLS SHA Value (Run "cms tlscertsha384" on CSP system)

Save and Close

Note: In case orchestration support is not needed, please comment/delete SHVS_IP in agent.conf available in same folder

./deploy_sgx_agent.sh
```

#### Deploy SKC Library
```
Copy skc_library.tar, skc_library.sha2 and skclib_untar.sh from binaries directoy to a directory in SGX compute node

./skclib_untar.sh

Update create_roles.conf with the following
  - IP address of AAS deployed on Enterprise system
  - Admin account credentials of AAS deployed on Enterprise system. These credentials should match with the AAS admin credentials provided in authservice.env on enterprise side.
  - Permission string to be embedded into skc_libraty client TLS Certificate
  - For Each SKC Library installation on a SGX compute node, please change SKC_USER and SKC_USER_PASSWORD

Save and Close

./skc_library_create_roles.sh
Copy the token printed on console.

Update skc_library.conf with the following
  - IP address for CMS and KBS services deployed on Enterprise system
  - CSP_CMS_IP should point to the IP of CMS service deployed on CSP system
  - CSP_SCS_IP should point to the IP of SCS service deployed on CSP system
  - Hostname of the Enterprise system where KBS is deployed
  - Network Port numbers for CMS and SCS services deployed on CSP system
  - Network Port numbers for CMS and KBS services deployed on Enterprise system
  - For Each SKC Library installation on a SGX compute node, please change SKC_USER (should be same as SKC_USER provided in create_roles.conf)
  - SKC_TOKEN with the token copied from previous step

Save and Close

./deploy_skc_library.sh
```

## **9. Usecase Workflows with Postman API Collections**

The below allow to get started with workflows within Intel® SecL-DC for Foundational and Workload Security Usecases. More details available in [API Collections](https://github.com/intel-secl/utils/tree/v4.0.2/develop/tools/api-collections) repository

### Use Case Collections

| Use case                     | Sub-Usecase | API Collection     |
| ---------------------------- | ----------- | ------------------ |
| Secure Key Caching           | -           | ✔️                  |
| SGX Discovery, Provisioning and Orchestration | -           | ✔️                  |
| SGX Discovery and Provisioning           | -           | ✔️                  |


### Download Postman API Collections

* Postman API Network for latest released collections: https://explore.postman.com/intelsecldc

  or 

* Github repo for allreleases

  ```shell
  #Clone the github repo for api-collections
  git clone https://github.com/intel-secl/utils.git
  
  #Switch to specific release-version of choice
  cd utils/
  git checkout <release-version of choice>
  
  #Import Collections from
  cd tools/api-collections
  ```

>  **Note:**  The postman-collections are also available when cloning the repos via build manifest under `utils/tools/api-collections`


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


## **10. System User Configuration**

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
        default = matching 
```

* Make sure system date and time of SGX machine and CSP machine both are in sync. Also, if the system is configured to read the RTC time in the local time zone, then use RTC in UTC by running `timedatectl set-local-rtc 0` command on both the machine. Otherwise SGX Agent deployment will fail with certificate expiry error. 

## Appendix

### SGX Attestation flow
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

Note: Below mentioned steps are provided as script (install_pykmip.sh and pykmip.service) as part of kbs_script folder which will install KMIP Server as daemon. Refer to ‘Install KMIP Server as daemon’ section.

```
1. Install python3 and vim-common
   #  apt -y install python3-pip vim-common   
   ln -s /usr/bin/python3 /usr/bin/python  > /dev/null 2>&1
   ln -s /usr/bin/pip3 /usr/bin/pip  > /dev/null 2>&1

2. Install pykmip
   # pip3 install pykmip==0.9.1

3. In the /etc/ directory create pykmip and policies folders
   mkdir -p /etc/pykmip/policies

4. Configure pykmip server using server.conf
   Update hostname in the server.conf

5. Copy the following to /etc/pykmip/ from kbs_script folder available under binaries directory
   create_certificates.py, run_server.py, server.conf

6. Create certificates
   > cd /etc/pykmip
   > python3 create_certificates.py <KMIP Host IP/KMIP Host FQDN>

7. Kill running KMIP Server processes and wait for 10 seconds until all the KMIP Server processes are killed. 
   > ps -ef | grep run_server.py | grep -v grep | awk '{print $2}' | xargs kill

8. Run pykmip server using run_server.py script
   > python3 run_server.py &

```

**Install KMIP Server as daemon**

```
1. cd into /root/binaries/kbs_script folder 

2. Configure pykmip server using server.conf
   Update hostname in the server.conf

3. Run the install_pykmip.sh script and KMIP server will be installed as daemon process
   ./install_pykmip.sh

```
**Create RSA key in PyKMIP and generate certificate**

NOTE: This step is required only when PyKMIP script is used as a backend KMIP server.

```
1. Update Host IP in /root/binaries/kbs_script rsa_create.py script
2. In the kbs_script folder, Run rsa_create.py script
    > cd /root/binaries/kbs_script
    > python3 rsa_create.py

This script will generate “Private Key ID” and “Server certificate”, which should be provided in the kbs.conf file for “KMIP_KEY_ID” and “SERVER_CERT”.

```
**Configuration Update to create Keys in KBS**
    
	cd into /root/binaries/kbs_script folder
	
    **To register keys with KBS KMIP**

    Update the following variables in kbs.conf:

        KMIP_KEY_ID (Private key ID registered in KMIP server)

        SERVER_CERT (Server certificate for created private key)

                Enterprise system IP address where CMS, AAS and KBS services are deployed

                Port of CMS, AAS and KBS services deployed on enterprise system

            AAS admin and Enterprise admin credentials

NOTE: If KMIP_KEY_ID is not provided then RSA key register will be done with keystring.

Update sgx_enclave_measurement_anyof value in transfer_policy_request.json with enclave measurement value obtained using sgx_sign utility. Refer to "Extracting SGX Enclave values for Key Transfer Policy" section.

**Create RSA Key**

	Execute the command
	
	./run.sh reg

copy the generated cert file to SGX Compute node where skc_library is deployed. Also make a note of the key id generated

## Configuration for NGINX testing

**Note:** Below mentioned OpenSSL and NGINX configuration updates are provided as patches (nginx.patch and openssl.patch) as part of skc_library deployment script. Patch can be applied with default nginx and openssl file. In case nginx/openssl contains any external changes then refer manual step.

**Apply Patch**
        Execute the command with nginx version - nginx 1.14.0 (Ubuntu) and openssl version- Openssl 1.1.1 (Ubuntu)

        patch -b /etc/nginx/nginx.conf < nginx_ubuntu.patch
        patch -b /etc/ssl/openssl.cnf < openssl_ubuntu.patch

**OpenSSL**

In the /etc/ssl/openssl.cnf file, look for the below line:
[ new_oids ]

Just before the line [ new_oids ], add the below section:

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

**Nginx**

Update nginx configuration file /etc/nginx/nginx.conf with below changes:

ssl_engine pkcs11;

Update the location of certificate with the loaction where it was copied into the skc_library machine. 

ssl_certificate "add absolute path of crt file";

Update the fields(token, object and pin-value) with the values given in keys.txt for the KeyID corresponding to the certificate.

ssl_certificate_key "engine:pkcs11:pkcs11:token=KMS;object=RSAKEY;pin-value=1234";

**SKC Configuration**

 Create keys.txt in /root folder. This provides key preloading functionality in skc_library.

  Any number of keys can be added in keys.txt. Each PKCS11 URL should contain different Key ID which need to be transferred from KBS along with respective object tag for each key id specified

  Sample PKCS11 url is as below
  
  pkcs11:token=KMS;id=164b41ae-be61-4c7c-a027-4a2ab1e5e4c4;object=RSAKEY;type=private;pin-value=1234;
  
  Token, object and pin-value given in PKCS11 url entry in keys.txt should match with the one in nginx.conf.

  The keyID should match the keyID of RSA key created in KBS. File location should match with preload_keys directive in pkcs11-apimodule.ini; 

  Sample /opt/skc/etc/pkcs11-apimodule.ini file
	
	[core]
	preload_keys=/root/keys.txt
	keyagent_conf=/opt/skc/etc/key-agent.ini
	mode=SGX
	debug=true
	
	[SGX]
	module=/opt/intel/cryptoapitoolkit/lib/libp11sgx.so

# KBS key-transfer flow validation

On SGX Compute node, Execute below commands for KBS key-transfer:


Note: Before initiating key transfer make sure, PYKMIP server is running.

```
    pkill nginx
```

Remove any existing pkcs11 token

```
    rm -rf /opt/intel/cryptoapitoolkit/tokens/*
```

Initiate Key transfer from KBS

```
    systemctl restart nginx
```

Changing group ownership and permissions of pkcs11 token

```
    chown -R root:intel /opt/intel/cryptoapitoolkit/tokens/
```

```
    chmod -R 770 /opt/intel/cryptoapitoolkit/tokens/
```

Establish a tls session with the nginx using the key transferred inside the enclave

```
    wget https://localhost:2443 --no-check-certificate
```

## Note on Key Transfer Policy

Key transfer policy is used to enforce a set of policies which need to be compiled with before the secret can be securely provisioned onto a sgx enclave

A typical Key Transfer Policy would look as below
```
        "sgx_enclave_issuer_anyof":["83d719e77deaca1470f6baf62a4d774303c899db69020f9c70ee1dfc08c7ce9e"],
        "sgx_enclave_issuer_product_id_anyof":[0],
        "sgx_enclave_measurement_anyof":["ad46749ed41ebaa2327252041ee746d3791a9f2431830fee0883f7993caf316a"],
        "tls_client_certificate_issuer_cn_anyof":["CMSCA", "CMS TLS Client CA"],
        "client_permissions_allof":["nginx","USA"],
        "sgx_enforce_tcb_up_to_date":false
```

**sgx_enclave_issuer_anyof** establishes the signing identity provided by an authority who has signed the sgx enclave. in other words the owner of the enclave

**sgx_enclave_measurement_anyof** represents the cryptographic hash of the enclave log (enclave code, data)

**sgx_enforce_tcb_up_to_date** - If set to true, Key Broker service will provision the key only of the platform generating the quote conforms to the latest Trusted Computing Base

**client_permissions_allof** - Special permission embedded into the skc_library client TLS certificate which can enforce additional restrictons on who can get access to the key,
    In above example: the key is provisioned only to the nginx workload and platform which is tagged with value for ex: USA


## Note on SKC Library Deployment

SKC Library Deployment (Binary as well as container) needs to performed with root privilege

For binary deployment of SKC client Library, only one instance of Workload can use SKC Client Library. The config information for SKC client library is bound to the workload.
In future, Multiple workloads might be supported
For container deployment, since configmaps are used, each container instance of workload gets its own private SKC Client Library config information

The SKC Client Library TLS client certificate private key is stored in the configuration directories and can be read only with elevated root privileges
keys.txt (set of PKCS11 URIs for the keys to be securely provisioned into an SGX enclave) can only be modified with elevated privileges


## Extracting SGX Enclave values for Key Transfer Policy

Values that are specific to the enclave such as sgx_enclave_issuer_anyof, sgx_enclave_measurement_anyof and sgx_enclave_issuer_product_id_anyof can be retrived using `sgx_sign` utility that is available as part of Intel SGX SDK.

Run `sgx_sign` utility on the signed enclave (This command should be run on the build system).
```
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
