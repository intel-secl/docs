# Deploying SKC Use Cases Using Helm

???+ note
SGX Attestation Infrastructure components need to be deployed in order to realize the SKC use case.

As indicated in the Features section, SKC provides 3 features essentially:

-   SGX Attestation Support: this is the feature that CSPs provide to tenants who need to run SGX workloads that require attestation.
-   SGX Support  in Orchestrators: this feature allows to discover SGX support in physical servers and related information:
  -   SGX supported.

  -   SGX enabled.

  -   Size of RAM reserved for SGX. It's called Enclave Page Cache (EPC).

  -   Flexible Launch Control enabled.
-   Key Protection: this is the feature used by tenants using a CSP to run workloads with key protection requirements.

The high-level architectures of these features are presented in the next sub-sections.

##  Key Protection

Key Protection leverages the SGX Attestation support and optionally, the SGX support in orchestrators.

Key Protection is implemented by the SKC Client -- a set of libraries - which must be linked with a tenant workload, like Nginx, deployed in a CSP environment and the Key Broker Service (KBS) deployed in the tenant's enterprise environment. The SKC Client retrieves the keys needed by the workload from KBS after proving that the key can be protected in an SGX enclave as shown in the diagram below.

![](images/image-20200727163116765.png)

## Deployment Steps

Configure the values.yaml answer file.  Each values.yaml file will contain "<user input>" prompts and a comment describing the information required.

## Use Case charts Deployment

Pull the Helm charts from the Helm repository and then deploy.

Set the release version
```
export VERSION=v5.0.0
```

### TEE-Attestation:

```
helm pull isecl-helm/TEE-Attestation --version $VERSION && tar -xzf TEE-Attestation-$VERSION.tgz TEE-Attestation/values.yaml
helm install tee-attastation isecl-helm/tee-attestation --version $VERSION -f tee-attestation/values.yaml --create-namespace -n <namespace>
```

Follow [Deploy SKC Library](./50%20Deploying%20Intel%20SecL%20Use%20Cases%20Using%20Helm.md#deploy-skc-library) to deploy SKC Library


### TEE Attestation Orchestration

```
helm pull isecl-helm/TEE-Attestation-Orchestration --version $VERSION && tar -xzf TEE-Attestation-Orchestration-$VERSION.tgz TEE-Attestation-Orchestration/values.yaml
helm install tee-attestation-orchestration isecl-helm/TEE-Attestation-Orchestration --version $VERSION -f TEE-Attestation-Orchestration/values.yaml --create-namespace -n <namespace>
```

Follow [Deploy SKC Library](./50%20Deploying%20Intel%20SecL%20Use%20Cases%20Using%20Helm.md#deploy-skc-library) to deploy SKC Library

### TEE Orchestration

```
helm pull isecl-helm/TEE-Orchestration --version $VERSION && tar -xzf TEE-Orchestration-$VERSION.tgz TEE-Orchestration/values.yaml
helm install tee-orchestration isecl-helm/TEE-Orchestration --version $VERSION -f TEE-Orchestration/values.yaml --create-namespace -n <namespace>
```

Follow [Deploy SKC Library](./50%20Deploying%20Intel%20SecL%20Use%20Cases%20Using%20Helm.md#deploy-skc-library) to deploy SKC Library

### Deploy SKC Library

Please follow the below command to deploy SKC Library in Kubernetes cluster.

#### Install Repo Tool

```
tmpdir=$(mktemp -d)
git clone https://gerrit.googlesource.com/git-repo $tmpdir
install -m 755 $tmpdir/repo /usr/local/bin
rm -rf $tmpdir

```

#### Perform repo init & repo sync and run skc_library_k8s_deploy target 

```
mkdir -p /root/intel-secl/build/skc-k8s && cd /root/intel-secl/build/skc-k8s
repo init -u https://github.com/intel-innersource/applications.security.isecl.tools.build-manifest.git -b v5.0/develop -m manifest/skc.xml
repo sync
make skc_library_k8s
```


#### Update SKC Library bootstrap env file and deploy SKC Library 

Update isecel-skc-k8s.env which is inside k8s/manifests directory.

```
# Kubernetes Distribution microk8s or kubeadm
K8S_DISTRIBUTION=<kubeadm|microk8s> 
# Control plane ip addresss
K8S_CONTROL_PLANE_IP=<control-plane-ip> 
# Control plane hostname
K8S_CONTROL_PLANE_HOSTNAME=<control-plane-hostname>  
# AAS admin user name
AAS_ADMIN_USERNAME=<aas-admin-username> 
# AAS admin password
AAS_ADMIN_PASSWORD=<aas-admin-password>
# ccc admin username
CCC_ADMIN_USERNAME=<ccc-admin-username> 
# ccc admin password
CCC_ADMIN_PASSWORD=<ccc-admin-password> 
# Default AAS Node port
AAS_PORT=30444 
# Default CMS Node port
CMS_PORT=30445 
# Default KBS Node port
KBS_PORT=30448 
# SKC Library image name
SKC_LIBRARY_IMAGE_NAME=<skc-library-hostname> 
# SKC Library image tag
SKC_LIBRARY_IMAGE_TAG=v5.0.0 
# KMIP IP Address
KMIP_IP=<kmip-ip> 
# KMIP SSL Client Cert path, sample cert path /etc/pykmip/client_certificate.pem
KMIP_CLIENT_CERT=<kmip-client-tls-cert-path> 
# KMIP SSL Client Key path, sample key path /etc/pykmip/client_key.pem
KMIP_CLIENT_KEY=<kmip-client-tls-key-path> 
# KMIP Root CA Cert path, sample key path /etc/pykmip/root_certificate.pem
KMIP_CLIENT_ROOTCA=<kmip-root-ca-cert-path> 
# Attestation mode passport or background 
MODE=<passport|background> 
#SKC Library image pull secret 
SKC_LIBRARY_IMAGE_PULL_SECRET=<pull-secret> 
#Get CTK Enclave measurement,prod id, isv svn values refer PG SGX Infrastructure/Note on SKC Library Deployment Appendix section
CTK_ENCLAVE_MEASUREMENT=<measurement> 
#SGX Node prod id
PROD_ID=<prod-id> 
#SGX Node isv-svn
ISV_SVN=<isv-svn> 
#SGX Node TCB Update value
TCB_UPDATE=false 
#SKC Library npm/sgx module debug enable
SKC_DEBUG=false 
# SGX enabled node label
NODE_LABEL=<SGX-node-label> 
# APS/KBS token validity in seconds
TOKEN_VALIDITY_SECS=43200


cd k8s/manifests
./skc-bootstrap.sh install
```

#### To uninstall SKC Library

```
./skc-bootstrap.sh uninstall
```
