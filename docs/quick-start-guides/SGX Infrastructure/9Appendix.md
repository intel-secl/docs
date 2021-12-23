# Appendix

##  SGX Attestation flow

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

## Creating RSA Keys in Key Broker Service

**Steps to run KMIP Server**

???+ note 
    Below mentioned steps are provided as script (install_pykmip.sh and pykmip.service) as part of kbs_script folder which will install KMIP Server as daemon.

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

## Configuration for NGINX testing

???+ note 
    Below mentioned OpenSSL and NGINX configuration updates are provided as patches (nginx.patch and openssl.patch) as part of skc_library deployment script. Patch can be applied with default nginx and openssl file. In case nginx/openssl contains any external changes then refer manual step.

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

## KBS key-transfer flow validation

On SGX Compute node, Execute below commands for KBS key-transfer:


???+ note 
    Before initiating key transfer make sure, PYKMIP server is running.

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

## Note on Key Transfer Policy

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

???+ note 
    sgx_enclave_issuer_anyof and sgx_enclave_issuer_product_id are mandatory fields which need to be specified in a Key Transfer policy

## Note on SKC Library Deployment

SKC Library Deployment (Binary as well as container) needs to performed with root privilege

For binary deployment of SKC client Library, only one instance of Workload can use SKC Client Library. The config information for SKC client library is bound to the workload.
In future, Multiple workloads might be supported For container deployment, since configmaps are used, each container instance of workload gets its own private SKC Client Library config information

The SKC Client Library TLS client certificate private key is stored in the configuration directories and can be read only with elevated root privileges
keys.txt (set of PKCS11 URIs for the keys to be securely provisioned into an SGX enclave) can only be modified with elevated privileges

## Deploy SKC Library as a Container 
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

## Extracting SGX Enclave values for Key Transfer Policy

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
