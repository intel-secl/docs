# Appendix

**Important Note:** SGX Attestation fails when SGX is enabled on a host booted using tboot

**Root Cause:** tboot requires the "noefi" kernel parameter to be passed during boot, in order to not use an unmeasured EFI runtime services. As a result, the kernel does not expose EFI variables to user-space. SGX Attestation requires these EFI variables to fetch Platform Manifest data.

**Workaround:**

The EFI variables required by SGX are only needed during the SGX provisioning/registration phase. Once this step is completed successfully, access to the EFI variables is no longer required. This means this issue can be worked around by installing the SGX agent without booting to tboot, then rebooting the system to tboot. SGX attestation will then work as expected while booted to tboot.

1. Enable SGX and TXT in the platform BIOS

2. Perform SGX Factory Reset and boot into the “plain” distribution kernel (without tboot or TCB)

3. Install tboot and ISecL components (SGX Agent, Trust Agent and Workload Agent)

4. The SGX Agent installation fetches the SGX Platform Manifest data and caches it

5. Reboot the system into the tboot kernel mode.

6. Verify that TXT measured launch was successful:

   `txt-stat |grep "TXT measured launch"`

7. The SGX and Platform Integrity Attestation use cases should now work as normal.


### SGX Attestation flow
```
To Deploy SampleApp:
  Copy sample_apps.tar, sample_apps.sha2 and sampleapps_untar.sh from binaries directory to a directory in SGX compute node and untar it using './sample_apps_untar.sh'
  Install Intel® SGX SDK for Linux*OS into /opt/intel/sgxsdk using './install_sgxsdk.sh'
  Install SGX dependencies using './deploy_sgx_dependencies.sh'
???+ note 
    Make sure to deploy SQVS with includetoken configuration as false.

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

???+ note 
    Below mentioned steps are provided as script (install_pykmip.sh and pykmip.service) as part of kbs_script folder which will install KMIP Server as daemon. Refer to ‘Install KMIP Server as daemon’ section.

```
1. Install python3 and vim-common
   For RHEL 8.2
   # dnf -y install python3-pip vim-common  
   For Ubuntu 18.04/20.04    
   # apt -y install python3-pip vim-common    
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

???+ note 
    This step is required only when PyKMIP script is used as a backend KMIP server.

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

???+ note 
    If KMIP_KEY_ID is not provided then RSA key register will be done with keystring.

Update sgx_enclave_measurement_anyof value in transfer_policy_request.json with enclave measurement value obtained using sgx_sign utility. Refer to "Extracting SGX Enclave values for Key Transfer Policy" section.

**Create RSA Key**

	Execute the command

	./run.sh reg

Copy the generated cert file to SGX Compute node where skc_library is deployed. Also make a note of the key id generated.

## Configuration for NGINX testing on RHEL 8.2

???+ note 
    Below mentioned OpenSSL and NGINX configuration updates are provided as patches (nginx.patch and openssl.patch) as part of skc_library deployment script. Patch can be applied with default nginx and openssl file. In case nginx/openssl contains any external changes then refer manual step.

**Apply Patch**
        Execute the command with nginx version - nginx 1.14.1 (Rhel) and openssl version- Openssl 1.1.1g (Rhel)

        patch -b /etc/nginx/nginx.conf < nginx.patch
        patch -b /etc/ssl/openssl.cnf < openssl.patch

**OpenSSL Configuration**

Update openssl configuration file /etc/pki/tls/openssl.cnf with below changes:

[openssl_def]
engines = engine_section

[engine_section]
pkcs11 = pkcs11_section

[pkcs11_section]
engine_id = pkcs11

dynamic_path =/usr/lib64/engines-1.1/pkcs11.so

MODULE_PATH =/opt/skc/lib/libpkcs11-api.so

init = 0

**Nginx Configuration**

Update nginx configuration file /etc/nginx/nginx.conf with below changes:

ssl_engine pkcs11;

Update the location of certificate with the location where it was copied into the SGX compute node.

ssl_certificate "add absolute path of crt file";

Update the fields(token, object and pin-value) with the values given in keys.txt for the KeyID corresponding to the certificate.

ssl_certificate_key "engine:pkcs11:pkcs11:token=KMS;object=RSAKEY;pin-value=1234";

**SKC Configuration**

 Create keys.txt in /root folder. This provides key preloading functionality in skc_library.

Any number of keys can be added in keys.txt. Each PKCS11 URL should contain different Key IDs which need to be transferred from KBS along with respective object tag for each key id specified

Token, object and pin-value given in PKCS11 url entry in keys.txt should match with the one in nginx.conf.

The keyID should match the keyID of RSA key created in KBS. File location should match on pkcs11-apimodule.ini;

	pkcs11:token=KMS;id=164b41ae-be61-4c7c-a027-4a2ab1e5e4c4;object=RSAKEY;type=private;pin-value=1234;

	Sample /opt/skc/etc/pkcs11-apimodule.ini file

	[core]
	preload_keys=/root/keys.txt
	keyagent_conf=/opt/skc/etc/key-agent.ini
	mode=SGX
	debug=false

	[SGX]
	module=/opt/intel/cryptoapitoolkit/lib/libp11sgx.so

## Configuration for NGINX testing for Ubuntu 18.04/20.04

???+ note 
    Below mentioned OpenSSL and NGINX configuration updates are provided as patches (nginx.patch and openssl.patch) as part of skc_library deployment script. Patch can be applied with default nginx and openssl file. In case nginx/openssl contains any external changes then refer manual step.

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
        debug=false


       [SGX]
        module=/opt/intel/cryptoapitoolkit/lib/libp11sgx.so
