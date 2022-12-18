## SKC Key Transfer Flow

Below steps to be followed post successful deployment with Single-Node/Multi-Node deployment

### Pre-requisites

* From cluster node, Clone the repos by using
```
   repo init -u https://github.com/intel-secl/build-manifest.git -m manifest/skc.xml -b refs/tags/v5.0.0
   repo sync
```  

* Do `make skc_library_k8s_deploy` and make sure new folder named k8s has been created.

* Do `cd k8s/manifests/` and update `isecl-skc-k8s.env` with appropriate values. Below are the list keys available in the file,
  * `K8S_DISTRIBUTION=` # K8s distribution microk8s/kubeadm
  * `K8S_CONTROL_PLANE_IP=` #Control plane IP
  * `K8S_CONTROL_PLANE_HOSTNAME=` #controlplane hostname
  * `AAS_ADMIN_USERNAME=` #AAS admin user name
  * `AAS_ADMIN_PASSWORD=` #AAS admin password
  * `CCC_ADMIN_USERNAME=` #enterprise AAS admin user name
  * `CCC_ADMIN_PASSWORD=` #enterprize AAS admin password
  * `AAS_PORT=`30444 #AAS default node port
  * `CMS_PORT=`30445 #CMS default node port
  * `KBS_PORT=`30448 #KBS default node port
  * `AAS_IP=` #AAS IP
  * `KBS_IP=` #KBS IP
  * `SCS_IP=` #SCS IP
  * `SCS_PORT=` 
  * `KBS_HOSTNAME=`
  * `CMS_IP=` #CMS IP
  * `CSP_SCS_IP=` #SCS IP on CSP
  * `CSP_SCS_PORT=` #SCS Port on CSP
  * `CSP_CMS_IP=` #CMS IP on CSP
  * `CSP_CMS_PORT=`30445 #CMS Port on CSP
  * `SKC_USER=` #SKC username
  * `SKC_USER_PASSWORD=` #SKC user password
  * `USE_SECURE_CERT=`false #set this to false by default
  * `SKC_LIBRARY_IMAGE_NAME=` #SKC Library oci/harbour image url
  * `SKC_LIBRARY_IMAGE_TAG=` #SKC Library oci/harbour image version
  * `SKC_LIBRARY_IMAGE_PULL_SECRET=` #SKC Library image pull secret
  * `CTK_ENCLAVE_MEASUREMENT=` #Get CTK Enclave measurement,prod id, isv svn values refer PG SGX Infrastructure/Note on SKC Library Deployment Appendix section
  * `CTK_SIGNER_MEASUREMENT=` #GET CTK MR Signer measurement
  * `PROD_ID=` #SGX Node prod id
  * `ISV_SVN=` #SGX Node isv-svn
  * `TCB_UPDATE=` #SGX Node TCB Update value
  * `SKC_DEBUG=`false #SKC Library npm/sgx module debug enable
  * `KMIP_IP=` #kmip server ip
  * `KMIP_CLIENT_CERT=`/etc/pykmip/client_certificate.pem #KMIP client certificate path
  * `KMIP_CLIENT_KEY=`/etc/pykmip/client_key.pem #KMIP client key path
  * `KMIP_CLIENT_ROOTCA=`/etc/pykmip/root_certificate.pem #KMIP root CA cert path
  * `MODE=` #Attestion mode passport/background.
  * `NODE_LABEL=` #node label of SGX host where SKC has to deploy

### Deploying SKC Library and Initiate key transfer flow

  * Do `./skc-bootstrap.sh install` to deploy SKC library and make sure deployment is successful
  * Once deployed, collect SKC library pod's log by running `kubectl get logs <skclib pod name> -n <namespace>`
  * Establish tls session with the nginx using the key transferred inside the enclave `wget https://<K8s control-plane IP>:30443 --no-check-certificate`

**_NOTE:_** 
Whenever running again `./skc-bootstrap.sh install`, make sure to delete `k8s` folder from repo root and follow steps from scratch.

### Uninstalling SKC Library
  
  * Do `./skc-bootstrap.sh uninstall` to clean up SKC library deployment

```
