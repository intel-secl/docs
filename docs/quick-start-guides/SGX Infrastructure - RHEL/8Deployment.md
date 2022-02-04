# Deployment

## Deployment Using Ansible

The below details would enable the deployment through Ansible Role for Intel® SecL-DC Secure Key Caching Usecase. However the services can still be installed manually using the Product Guide. More details on Ansible Role for Intel® SecL-DC in [Ansible-Role](https://github.com/intel-secl/utils/tree/v4.0.2/develop/tools/ansible-role) repository.

## Download the Ansible Role 

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

## Update Ansible Inventory

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

???+ note 
    Ansible requires `ssh` and `root` user access to remote machines. The following command can be used to ensure ansible can connect to remote machines with host key check `
    ```shell
    ssh-keyscan -H <ip_address> >> /root/.ssh/known_hosts
    ```


## Create and Run Playbook

The following are playbook and CLI example for deploying Intel® SecL-DC binaries based on the supported deployment models and usecases. The below example playbooks can be created as `site-bin-isecl.yml`

???+ note 
    If running behind a proxy, update the proxy variables under `<path to ansible role>/ansible-role/vars/main.yml` and run as below

???+ note 
    Go through the `Additional Examples and Tips` section for specific workflow samples

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


## Usecase Setup Options

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

## Deployment Using Binaries

### Setup K8S Cluster and Deploy Isecl-k8s-extensions

* Setup master and worker node for k8s. Worker node should be setup on SGX enabled host machine. Master node can be any system.

* To setup k8 cluster follow https://phoenixnap.com/kb/how-to-install-kubernetes-on-centos Once the master/worker setup is done, follow below steps on Master Node:

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

#### Deploying SKC Services on Single System
```
Copy the binaries directory generated in the build system to the /root/ directory on the deployment system
Update orchestrator.conf with the following
  - Deployment system IP address
  - SAN List (a list of ip address and hostname for the deployment system)
  - Network Port numbers for CMS, AAS, SCS and SHVS
  - Install Admin and CSP Admin credentials
  - TENANT as KUBERNETES or OPENSTACK (based on the orchestrator chosen)
  - System IP address where Kubernetes or Openstack is deployed
  - Netowrk Port Number of Kubernetes or Openstack Keystone/Placement Service
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

In case ihub installation fails, its recommended to run the following command to clear failed service instance

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
  - TENANT as KUBERNETES or OPENSTACK (based on the orchestrator chosen)
  - System IP address where Kubernetes or Openstack is deployed
  - Netowrk Port Number of Kubernetes or Openstack Keystone/Placement Service
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
#### Openstack Setup and Associate Traits

* Setup Compute and Controller node for Openstack. Compute node should be setup on SGX host machine, Controller node can be any system. After the compute/controller setup is done, follow the below steps:

* IHUB should be installed and configured with Openstack

  Note:
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
* Veiw the Traits that has been set:
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
 Note : To unset the trait, use the following CLI commands:
```
 openstack image unset --property trait:CUSTOM_ISECL_SGX_ENABLED_TRUE <image name>

 openstack image unset --property trait:CUSTOM_ISECL_SGX_ENABLED_FALSE <image name>
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
#### Deploying SKC Library as a Container 
```
Use the following steps to configure SKC library running in a container and to validate key transfer in container on bare metal and inside a VM on SGX enabled hosts.

Note: All the configuration files required for SKC Library container are modified in the resources directory only 

1. Docker should be installed, enabled and services should be active

2.To get the SKC library tar file, run "make skc_library_k8s".
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

