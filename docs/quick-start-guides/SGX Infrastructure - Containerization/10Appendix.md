# Appendix

## Running behind Proxy

```shell
#Set proxy in ~/.bash_profile
export http_proxy=<proxy-url>
export https_proxy=<proxy-url>
export no_proxy=<ip_address/hostname>
```

## Git Config Sample (~/.gitconfig)

```ini
[user]
        name = <username>
        email = <email-id>
[color]
        ui = auto
 [push]
        default = matching 
```

## Rebuilding Repos

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

## SGX Attestation Flow
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

## SKC Key Transfer Flow

Below steps to be followed post successful deployment with Single-Node/Multi-Node deployment

### Generating keys

* From cluster node, copy `k8s/manifests/kbs/rsa_create.py` to KMIP server, update KMIP server IP in rsa_create.py inside single quote and execute it using `python3 rsa_create.py`. It will generate the KMIP KEY ID and server.crt. Copy server.crt to cluster node.

* On cluster node, navigate to `k8s/manifests/kbs/` 

* Update `kbs.conf` with `SYSTEM_IP`, `AAS_PORT`, `KBS_PORT`, `CMS_PORT`, `KMIP_KEY_ID` and `SERVER_CERT`
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
  
???+ note 
    Update skc token in the `skc_library.conf` generated in previous step
  
* Update `mountPath` and `subPath` with generated key id, along with image name in `k8s/manifests/skc_library/deployment.yml`  

* Update `isecl-skc-k8s.env` by uncommenting `KBS_PUBLIC_CERTIFICATE=<key id>.crt` and update the `<key id>` value

### Initiate Key Transfer Flow

* Deploy the SKC Library using `./skc-bootstrap.sh up skclib`
* It will initiate the key transfer
???+ note 
    To enable debug log, make `debug=true` in pkcs11-apimodule.ini, kms_npm.ini and sgx_stm.ini files available under `k8s/manifests/skc_library/resources`

* Establish tls session with the nginx using the key transferred inside the enclave
   `wget https://<K8s control-plane IP>:30443 --no-check-certificate`
 
#### Other Workloads

Secure Key Caching use case employs nginx web server as a workload to demonstrate confidential computing. If the need is to use other custom workloads, then following reference Dockerfile and entrypoint scripts can be used as starting point to deploy custom workload as a container.

Reference `Dockerfile` for Nginx with skc library
```
# Copyright (C) 2021 Intel Corporation
# SPDX-License-Identifier: BSD-3-Clause

FROM ubuntu:focal

LABEL org.label-schema.name="SKC Library" \
      org.label-schema.vendor="Intel Corporation" \
      org.label-schema.license="BSD-3-Clause" \
      org.label-schema.url="https://github.com/intel-secl/intel-secl"

# Set env variables
ENV SGX_INSTALL_DIR /opt/intel
ENV SKC_LIBRARY_BIN_NAME skc_library_*.bin

# Copy binaries and SGX local repo
COPY dist/image/bin/libp11.so* /usr/lib/x86_64-linux-gnu/
COPY dist/image/bin/pkcs11.so /usr/lib/x86_64-linux-gnu/engines-1.1/

COPY dist/image/bin/${SKC_LIBRARY_BIN_NAME} $PWD
COPY dist/image/entrypoint.sh $PWD
RUN chmod 700 /entrypoint.sh
COPY dist/image/credential_agent.sh $PWD
RUN chmod 700 /credential_agent.sh
COPY dist/image/sgxssl ${SGX_INSTALL_DIR}/sgxssl
COPY dist/image/cryptoapitoolkit ${SGX_INSTALL_DIR}/cryptoapitoolkit

#Install dependencies and SGX components
RUN apt-get -yqq update
RUN apt-get -yqq install wget gnupg1 gnupg2 apt-utils curl p11-kit
RUN echo 'deb [arch=amd64] https://download.01.org/intel-sgx/sgx_repo/ubuntu/ focal main' | tee /etc/apt/sources.list.d/intel-sgx.list
RUN wget -qO - https://download.01.org/intel-sgx/sgx_repo/ubuntu/intel-sgx-deb.key | apt-key add -
RUN DEBIAN_FRONTEND=noninteractive TZ=Etc/UTC  apt-get -yqq install opensc makeself libgda-5.0-bin libgda-5.0-dev libcurl4-openssl-dev libssl-dev \
    libjsoncpp1 libjsoncpp-dev libcppunit-dev libcrypto++6 libglib2.0-dev jq cron nginx

RUN apt-get -yqq update
RUN DEBIAN_FRONTEND=noninteractive apt-get -yqq install libsgx-launch libsgx-uae-service libsgx-urts libsgx-ae-qve libsgx-dcap-ql libsgx-dcap-ql-dev \
    libsgx-dcap-default-qpl-dev libsgx-dcap-default-qpl

RUN groupadd intel && \
    usermod -G intel www-data && \
    ln -sf /usr/lib/x86_64-linux-gnu/libp11.so.3.4.3 /usr/lib/x86_64-linux-gnu/libp11.so

RUN apt-get clean && apt-get autoclean

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



## SKC Virtualization Flow

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

### Configuration for Key Transfer On SGX Virtualization Setup

???+ note 
    Below mentioned OpenSSL and NGINX configuration updates are provided as patches (nginx.patch and openssl.patch) as part of skc_library deployment script.

#### OpenSSL

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

#### Nginx

Update nginx configuration file `/etc/nginx/nginx.conf` with below changes

ssl_engine pkcs11;

Update the location of certificate with the loaction where it was copied into the skc_library machine. 

ssl_certificate "add absolute path of crt file";

Update the KeyID with the KeyID received when RSA key was generated in KBS

ssl_certificate_key "engine:pkcs11:pkcs11:token=KMS;id=164b41ae-be61-4c7c-a027-4a2ab1e5e4c4;object=RSAKEY;type=private;pin-value=1234";

#### SKC Configuration

* Create `keys.txt` in /root folder. This provides key preloading functionality in skc_library.

???+ note 
    Any number of keys can be added in keys.txt. Each PKCS11 URL should contain different Key ID which need to be transferred from KBS along with respective object tag for each key id specified

* Sample PKCS11 url is as below

  ```
  pkcs11:token=KMS;id=164b41ae-be61-4c7c-a027-4a2ab1e5e4c4;object=RSAKEY;type=private;pin-value=1234;
  ```

???+ note 
    Last PKCS11 url entry in `keys.txt` should match with the one in `nginx.conf`

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

#### Key-transfer flow validation

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

#### Note on Key Transfer Policy

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

#### Note on SKC Library Deployment

SKC Library Deployment needs to performed with root privilege

Each container instance of workload gets its own private SKC Client Library config information

The SKC Client Library TLS client certificate private key is stored in the configuration directories and can be read only with elevated root privileges
keys.txt (set of PKCS11 URIs for the keys to be securely provisioned into an SGX enclave) can only be modified with elevated privileges


#### Extracting SGX Enclave values for Key Transfer Policy

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



## Setup Task Flow

Setup tasks flows have been updated to have K8s native flow to be more agnostic to K8s workflows. Following would be the flow for setup task

- User would create a new `configMap`  object with the environment variables specific to the setup task. The Setup task variables would be documented in the Product Guide
- Users can provide variables for more than one setup task
- If variables involve BEARER_TOKEN then user need to delete the corresponding secret using `kubectl delete secret <service_name>-secret -n isecl` , update the new token inside secrets.txt file and re-create the secret using `kubectl create secret generic <service_name>-secret --from-file=secrets.txt --namespace=isecl`, else update the variables in configMap.yml
- Users need to add `SETUP_TASK: "<setup task name>/<comma separated setup task name>"` in the same `configMap`
- Provide a unique name to the new `configMap`
- Provide the same name in the `deployment.yaml` under `configMapRef` section
- Deploy the specific service again with  ` kubectl kustomize . | kubectl  apply -f -`

## Configuration Update Flow

Configuration Update flows have been updated to have K8s native flow to be more agnostic to K8s workflows using `configMap` only. Following would be the flow for configuration update

- User would create a new `configMap`  object using existing one and update the new values. The list of config variables would be documented in the Product Guide
- Users can update variables for more than one
- Users need to add `SETUP_TASK: "update-service-config"` in the same `configMap`
- Provide a unique name to the new `configMap`
- Provide the same name in the `deployment.yaml` under `configMapRef` section
- Deploy the specific service again with  ` kubectl  kustomize . | kubectl  apply -f -`

???+ note 
    Incase of agents, setup tasks or configuration updates done through above flows will be applied for all the agents running on different BMs. In order to run setup task or update configuration for individual agents, then user need to perform `kubectl exec -it <pod_name> /bin/bash` into a particular agent pod and run the specific setup task.

## Cleanup workflows

### Single-node

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
```shell
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

### Multi-node

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
```shell

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
