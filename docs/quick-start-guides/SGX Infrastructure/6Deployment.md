#  Deployment

## Deploy Linux Stacks for Intel SGX

* To setup and deployment of the Linux* Stacks for Intel® SGX, follow https://download.01.org/intelsgxstack/2021-07-28/Getting_Started.pdf
* Install Kubernetes Stack on control-plane node and SGX compute node using Linux SGX Stacks Ansible. Control-plane node can be any system, Compute node should be setup on SGX enabled host machine.
* Once the Kubernetes stack deployed in control-plane and SGX compute node, follow the below steps for deploying Intel® SecL-DC Secure Key Caching Usecase through Ansible Role.

## Deployment Using Binaries

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
???+ note 
    In case orchestration support is not needed, please comment/delete SHVS_IP in agent.conf available in same folder
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

???+ note 
    For deploying SKC Library as a container refer [Deploy SKC Library as a container](#deploying-skc-library-as-a-container) section in Appendix

## Deployment & Usecase Workflow Tools Installation

The below installation is required on the Build & Deployment system only and the Platform(Windows,Linux or MacOS) for Usecase Workflow Tool Installation

Deployment Tools Installation

### Usecases Workflow Tools Installation

* Postman client should be [downloaded](https://www.postman.com/downloads/) on supported platforms or on the web to get started with the usecase collections.

???+ note 
    The Postman API Network will always have the latest released version of the API Collections. For all releases, refer the github repository for [API Collections](https://github.com/intel-secl/utils/tree/v4.1/develop/tools/api-collections)


## Deployment Using Ansible

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


???+ note 
    Orchestrator installation is not bundled with the role and need to be done independently. Also, components dependent on the orchestrator like `isecl-k8s-extensions` and `integration-hub` are installed either partially or not installed


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

???+ note 
    Ansible requires `ssh` and `root` user access to remote machines. The following command can be used to ensure ansible can connect to remote machines with host key check `
  ```shell
  ssh-keyscan -H <ip_address> >> /root/.ssh/known_hosts
  ```

### Create and Run Playbook

The following are playbook and CLI example for deploying Intel® SecL-DC binaries based on the supported deployment models and usecases. The below example playbooks can be created as `site-bin-isecl.yml`

???+ note 
    If running behind a proxy, update the proxy variables under `<path to ansible role>/ansible-role/vars/main.yml` and run as below

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

???+ note 
    If any service installation fails due to any misconfiguration, just uninstall the specific service manually , fix the misconfiguration in ansible and rerun the playbook. The successfully installed services won't be reinstalled.


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
     skopeo copy oci-archive:isecl-k8s-controller-v4.1-<commitid>.tar docker://<registryIP>:<registryPort>/isecl-k8s-controller:v4.1
     skopeo copy oci-archive:isecl-k8s-scheduler-v4.1-<commitid>.tar docker://<registryIP>:<registryPort>/isecl-k8s-scheduler:v4.1
  ```
  
* Add the image names in isecl-controller.yml and isecl-scheduler.yml in /opt/isecl-k8s-extensions/yamls with full image name including registry IP/hostname (e.g <registryIP>:<registryPort>/isecl-k8s-scheduler:v4.1.0). It will automatically pull the images from registry.

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
???+ note 
    Prefix the attestation type for ihub_public_key.pem before copying to secrets folder.
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

???+ note 
    Make sure to use proper indentation and don't delete existing mountPath and hostPath sections in kube-scheduler.yaml.
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

???+ note 
    Openstack Support only validated with RHEL 8.1/8.2

* Setup Compute and Controller node for Openstack. Compute node should be setup on SGX host machine, Controller node can be any system. After the compute/controller setup is done, follow the below steps:

* IHUB should be installed and configured with Openstack

???+ note 
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
???+ note 
    To unset the trait, use the following CLI commands:
```
 openstack image unset --property trait:CUSTOM_ISECL_SGX_ENABLED_TRUE <image name>

 openstack image unset --property trait:CUSTOM_ISECL_SGX_ENABLED_FALSE <image name>
```