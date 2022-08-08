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

## Setup container runtime and Kubernetes

#### Install cri-o container runtime

* The supported version for CRIO is 1.23

* To setup crio runtime for RHEL, follow https://www.linuxtechi.com/how-to-install-kubernetes-cluster-rhel/
* To setup crio runtime for Ubuntu, follow https://computingforgeeks.com/deploy-kubernetes-cluster-on-ubuntu-with-kubeadm/

#### Install Kubernetes

* The supported version for Kubernetes is 1.23

* To setup k8 cluster for RHEL, follow https://phoenixnap.com/kb/how-to-install-kubernetes-on-centos.
* To setup k8 cluster for Ubuntu, follow https://phoenixnap.com/kb/install-kubernetes-on-ubuntu

#### Run the prerequisite script in Worker Node:
Clone the utils repo and follow below steps.

```
cd utils/build/skc-tools/common_attest/tee/build_scripts/
chmod +x install_prereqs.sh
./install_prereqs.sh
```

## Deployment Steps

Configure the values.yaml answer file.  Each values.yaml file will contain "<user input>" prompts and a comment describing the information required.

## Use Case charts Deployment

Pull the Helm charts from the Helm repository and then deploy.

Set the release version
```
export VERSION=v5.0.0
```

### Update TEE-Attestation values.yaml file

```shell
cms:
  image:
    name: <user input> # Certificate Management Service image name<br> (**REQUIRED**)

aas:
  image:
    name: <user input> # Authentication & Authorization Service image name<br> (**REQUIRED**)
  config:
    dbMaxConnections: 200 # Determines the maximum number of concurrent connections to the database server. Default is 200
    dbSharedBuffers: 2GB # Determines how much memory is dedicated to PostgreSQL to use for caching data. Default is 2GB
  secret:
    dbUsername:  # DB Username for AAS DB
    dbPassword:  # DB Password for AAS DB

tcs:
  image:
    name: <user input> # TEE caching service image name<br> (**REQUIRED**)
  secret:
    dbUsername:  # DB Username for TCS DB
    dbPassword:  # DB Password for TCS DB
    intelPcsApiKey: <user input> # Intel PCS API Subscription Key
    installAdminUsername:  # Admin Username for TCS
    installAdminPassword:  # Admin Password for TCS
  config:
    intelPcsUrl: <user input> # Intel PCS URL
    retryCount: <user input> # Retries attempted in case PCS is not responding. Retry count should be in range of 1 to 3.
    waitTime: <user input> # Time interval between each retry in seconds. Wait time should be less than or equal to 2 seconds.
    refreshInterval: <user input> # Automatic refresh time of platform collateral. RefreshInterval should be in range of 24 to 720 hours.
    dbMaxConnections: 200 # Determines the maximum number of concurrent connections to the database server. Default is 200
    dbSharedBuffers: 2GB # Determines how much memory is dedicated to PostgreSQL to use for caching data. Default is 2GB
    proxy:
      proxyEnabled: true #checking if proxy is enabled
      httpProxy:http://proxy-us.intel.com:911 #http proxy url
      httpsProxy:http://proxy-us.intel.com:911 #https proxy url
      noProxy:cms.isecl.svc.cluster.local,aas.isecl.svc.cluster.local #https proxy url
      allProxy: #https proxy url

qvs:
  image:
    name: <user input> # Quote Verification image name<br> (**REQUIRED**)
  secret:
    installAdminUsername:  # Admin user name for QVS
    installAdminPassword:  # Admin Password for QVS

fda:
  image:
    name: <user input> # Feature Discovery Agent image name<br> (**REQUIRED**)
  config:
    refreshInterval: <user input> # Refresh Interval should be in range of 60 to 240 seconds
    retryCount: <user input> # Retry count should be in range of 1 to 3
    validitySeconds: <user input> # Validity of custom token in seconds (Note: Value needs to be provided in quotes)
  secret:
    cccAdminUsername:  #ccc admin token username
    cccAdminPassword:  #ccc admin token password
  nodeLabel:
    sgxTxtEnabled: <user input> # The node label for SGX-ENABLED and TXT-ENABLED hosts<br> (**REQUIRED IF NODE IS SGX or TXT ENABLED**)

aps:
  image:
    name: <user input> # Attestation policy service image name<br> (**REQUIRED**)
  secret:
    dbUsername:  # DB Username for APS DB
    dbPassword:  # DB Password for APS DB
    installAdminUsername:  # Admin Username for APS
    installAdminPassword:  # Admin Password for APS
    serviceUsername:
    servicePassword:
  config:
    dbMaxConnections: 200 # Determines the maximum number of concurrent connections to the database server. Default is 200
    dbSharedBuffers: 2GB # Determines how much memory is dedicated to PostgreSQL to use for caching data. Default is 2GB

kbs:
  image:
    name: <user input> # Key Broker Service image name<br> (**REQUIRED**)

  config:
    kmip:
      serverIp: <user input> # The KMIP server IP
      serverHostname: <user input> # The KMIP server IP/hostname. Provide same value which is provided during KMIP certificate generation.
      serverPort: <user input> # The KMIP server port
    tee: true
  secret:
    installAdminUsername:  # Install Admin Username for KBS
    installAdminPassword:  # Install Admin Password for KBS
    cccUsername:  # Custom Claims Creator Username
    cccPassword:  # Custom Claims Creator Password

 customToken:
    validitySeconds: "31536000" # Custom Token validity in seconds (Default: "31536000")

global-admin-generator:
  enable: false # Set this to true for generating global admin user account
  secret:
    globalAdminUsername:  # Global admin user
    globalAdminPassword:  # Global admin password
  services_list: # Services list for global admin token generation. Accepted values HVS, WLS, KBS, APS, FDS, TA, QVS, TCS
    - APS
    - TCS
    - KBS
    - QVS

global:
  controlPlaneHostname: <user input> # K8s control plane IP/Hostname<br> (**REQUIRED**)
  controlPlaneLabel: microk8s.io/cluster # K8s control plane label<br> (**REQUIRED**)<br> Example: `node-role.kubernetes.io/master` in case of `kubeadm`/`microk8s.io/cluster` in case of `microk8s`

  image:
    pullPolicy: Always # The pull policy for pulling from container registry (Allowed values: `Always`/`IfNotPresent`)
    imagePullSecret:  # The image pull secret for authenticating with image registry, can be left empty if image registry does not require authentication
    initName: <user input> # The image name of init container
    aasManagerName: <user input> # The image name of aas-manager image name

  config:
    dbhostSSLPodRange: 10.1.0.0/8 # PostgreSQL DB Host Address(IP address/subnet-mask). IP range varies for different k8s network plugins(Ex: Flannel - 10.1.0.0/8 (default), Calico - 192.168.0.0/16).

  storage:
    nfs:
      server: <user input> # The NFS Server IP/Hostname<br> (**REQUIRED**)
      path: /mnt/nfs_share/  # The path for storing persistent data on NFS

  service:
    aas: 30444 # The service port for Authentication Authorization Service
    cms: 30445 # The service port for Certificate Management Service
    kbs: 30448 # The service port for Key broker service
    tcs: 30502 # The service port for Authentication Authorization Service
    qvs: 30501 # The service port for TCS Service
    aps: 30503 # The service port for Attestation policy service

  aas:
    secret:
      adminUsername:  # Service Username for AAS
      adminPassword:  # Service Password for AAS

  qvs:
    isSandBoxEnv: "no" # If SGX node which runs client (SKC client/ SGX Agent) is production grade then mark it "no", else mark it "yes".

  ingress:
    enable: false # Accept true or false to notify ingress rules are enable or disabled

  proxyEnabled: true # If proxy is enabled, then set to true
  httpProxy:http://proxy-us.intel.com:911/ # set HTTP Proxy value
  httpsProxy:http://proxy-us.intel.com:911/  # set HTTP Proxy value
  noProxy:0.0.0.0,localhost,127.0.0.0/8,10.0.0.0/8,svc,.svc.cluster.local # Append .svc,.svc.cluster.local, to no_proxy
  allProxy: # set all proxy value
```
### TEE-Attestation:

```
helm pull isecl-helm/TEE-Attestation --version $VERSION && tar -xzf TEE-Attestation-$VERSION.tgz TEE-Attestation/values.yaml
helm install tee-attastation isecl-helm/tee-attestation --version $VERSION -f tee-attestation/values.yaml --create-namespace -n <namespace>
```

Follow [Deploy SKC Library](./50%20Deploying%20Intel%20SecL%20Use%20Cases%20Using%20Helm.md#deploy-skc-library) to deploy SKC Library

### Update TEE-Orchestration values.yaml file

```shell
cms:
  image:
    name: <user input> # Certificate Management Service image name<br> (**REQUIRED**)

aas:
  image:
    name: <user input> # Authentication & Authorization Service image name<br> (**REQUIRED**)
  secret:
    dbUsername:  # DB Username for AAS DB
    dbPassword:  # DB Password for AAS DB
  config:
    dbMaxConnections: 200 # Determines the maximum number of concurrent connections to the database server. Default is 200
    dbSharedBuffers: 2GB # Determines how much memory is dedicated to PostgreSQL to use for caching data. Default is 2GB

tcs:
  image:
    name: <user input> # TEE caching service image name<br> (**REQUIRED**)
  secret:
    dbUsername:  # DB Username for TCS DB
    dbPassword:  # DB Password for TCS DB
    intelPcsApiKey: <user input> # Intel PCS API Subscription Key
    installAdminUsername:  # Admin Username for TCS
    installAdminPassword:  # Admin Password for TCS
  config:
    intelPcsUrl: <user input> # Intel PCS URL
    retryCount: <user input> # Retries attempted in case PCS is not responding. Retry count should be in range of 1 to 3.
    waitTime: <user input> # Time interval between each retry in seconds. Wait time should be less than or equal to 2 seconds.
    refreshInterval: <user input> # Automatic refresh time of platform collateral. RefreshInterval should be in range of 24 to 720 hours.
    dbMaxConnections: 200 # Determines the maximum number of concurrent connections to the database server. Default is 200
    dbSharedBuffers: 2GB # Determines how much memory is dedicated to PostgreSQL to use for caching data. Default is 2GB
    proxy:  
      proxyEnabled: true #checking if proxy is enabled
      httpProxy:http://proxy-us.intel.com:911 #http proxy url
      httpsProxy:http://proxy-us.intel.com:911 #https proxy url
      noProxy:cms.isecl.svc.cluster.local,aas.isecl.svc.cluster.local #https proxy url
      allProxy: #https proxy url
fds:
  image:
    name: <user input> # Feature Discovery Service image name<br> (**REQUIRED**)
  secret:
    dbUsername: # DB Username for FDS DB
    dbPassword: # DB Password for FDS DB
    installAdminUsername:  # Admin Username for FDS
    installAdminPassword:  # Admin Password for FDS
  config:
    dbMaxConnections: 200 # Determines the maximum number of concurrent connections to the database server. Default is 200
    dbSharedBuffers: 2GB # Determines how much memory is dedicated to PostgreSQL to use for caching data. Default is 2GB

fda:
  image:
    name: <user input> # Feature Discovery Agent image name<br> (**REQUIRED**)
  config:
    refreshInterval: <user input> # Refresh Interval should be in range of 60 to 240
    retryCount: <user input> # Retry count should be in range of 1 to 3
    validitySeconds: <user input> # Validity of custom token in seconds (Note: Value needs to be provided in quotes)
    cccAdminUsername:  #ccc admin token username
    cccAdminPassword:  #ccc admin token password
  nodeLabel:
    sgxTxtEnabled: <user input>  # The node label for SGX-ENABLED and TXT-ENABLED hosts<br> (**REQUIRED IF NODE IS SGX or TXT ENABLED**)

isecl-controller:
  image:
    name: <user input> # ISecL Controller Service image name<br> (**REQUIRED**)
  nodeTainting:
    taintRegisteredNodes: false # If set to true, taints the node which are joined to the k8s cluster. (Allowed values: `true`\`false`)
    taintRebootedNodes: false # If set to true, taints the node which are rebooted in the k8s cluster. (Allowed values: `true`\`false`)
    taintUntrustedNode: false # If set to true, taints the node which has trust tag set to false in node labels. (Allowed values: `true`\`false`)

ihub:
  image:
    name: <user input> # Integration Hub Service image name<br> (**REQUIRED**)
  k8sApiServerPort: 6443
  dependentServices:
    fds: fds
  secret:
    installAdminUsername: # Install Admin Username for IHub
    installAdminPassword: # Install Admin Password for IHub
    serviceUsername:  # Service Username for IHub
    servicePassword:  # Service Password for IHub

isecl-scheduler:
  image:
    name: <user input> # ISecL Scheduler image name<br> (**REQUIRED**)

global-admin-generator:
  enable: false # Set this to true for generating global admin user account
  secret:
    globalAdminUsername:  # Global admin user
    globalAdminPassword:  # Global admin password
  services_list: # Services list for global admin token generation. Accepted values HVS, WLS, WLA, KBS, APS, FDS, TA, QVS, TCS
    - APS
    - FDS
    - TCS

global:
  config:
    dbhostSSLPodRange: 10.1.0.0/8 # PostgreSQL DB Host Address(IP address/subnet-mask). IP range varies for different k8s network plugins(Ex: Flannel - 10.1.0.0/8 (default), Calico - 192.168.0.0/16).

  controlPlaneHostname: <user input> # K8s control plane IP/Hostname<br> (**REQUIRED**)
  controlPlaneLabel: <user input> #K8s control plane label<br> (**REQUIRED**)<br> Example: `node-role.kubernetes.io/master` (k8s 1.23) or node-role.kubernetes.io/control-plane (k8s 1.24) in case of `kubeadm`/`microk8s.io/cluster` in case of `microk8s

  image:
    pullPolicy: Always # The pull policy for pulling from container registry (Allowed values: `Always`/`IfNotPresent`)
    imagePullSecret:  # The image pull secret for authenticating with image registry, can be left empty if image registry does not require authentication
    initName: <user input> # The image name of init container
    aasManagerName: <user input> # The image name of aas-manager image name

  storage:
    nfs:
      server: <user input> # The NFS Server IP/Hostname<br> (**REQUIRED**)
      path: /mnt/nfs_share/  # The path for storing persistent data on NFS

  fdsUrl: <user input> # Fds Base Url, Do not include "/" at the end. e.g for ingress https://fds.isecl.com/fds/v1 , for nodeport  https://<control-plane-hostname/control-plane-IP>:30500/fds/v1
  cmsUrl: <user input> # CMS Base Url, Do not include "/" at the end. e.g for ingress https://cms.isecl.com/cms/v1 , for nodeport https://<control-plane-hostname/control-plane-IP>:30445/cms/v1
  aasUrl: <user input> # Authservice Base Url, Do not include "/" at the end. e.g for ingress https://aas.isecl.com/aas/v1 , for nodeport https://<control-plane-hostname/control-plane-IP>:30444/aas/v1

  service:
    aas: 30444 # The service port for Authentication Authorization Service
    cms: 30445 # The service port for Certificate Management Service
    fds: 30500 # The service port for Feature Discovery Service
    qvs: 30501 # The service port for Authentication Authorization Service
    tcs: 30502 # The service port for TCS Service
    isecl-scheduler: 30888 #The service port for Isecl scheduler

  aas:
    secret:
      adminUsername:  # Service Username for AAS
      adminPassword:  # Service Password for AAS

  qvs:
    isSandBoxEnv: "no" # If SGX node which runs client (SKC client/ SGX Agent) is production grade then mark it "no", else mark it "yes".

  ingress:
    enable: false # Accept true or false to notify ingress rules are enable or disabled

  proxyEnabled: true # If proxy is enabled, then set to true
  httpProxy:http://proxy-us.intel.com:911/ # set HTTP Proxy value
  httpsProxy:http://proxy-us.intel.com:911/  # set HTTP Proxy value
  noProxy:0.0.0.0,localhost,127.0.0.0/8,10.0.0.0/8,svc,.svc.cluster.local # Append .svc,.svc.cluster.local, to no_proxy
  allProxy: # set all proxy value
```

### TEE Orchestration

```
helm pull isecl-helm/TEE-Orchestration --version $VERSION && tar -xzf TEE-Orchestration-$VERSION.tgz TEE-Orchestration/values.yaml
helm install tee-orchestration isecl-helm/TEE-Orchestration --version $VERSION -f TEE-Orchestration/values.yaml --create-namespace -n <namespace>
```

### TEE Attestation Orchestration
Update TEE-Attestation-Orchestration values.yaml.
```
helm pull isecl-helm/TEE-Attestation-Orchestration --version $VERSION && tar -xzf TEE-Attestation-Orchestration-$VERSION.tgz TEE-Attestation-Orchestration/values.yaml
helm install tee-attestation-orchestration isecl-helm/TEE-Attestation-Orchestration --version $VERSION -f TEE-Attestation-Orchestration/values.yaml --create-namespace -n <namespace>
```

Follow [Deploy SKC Library](./50%20Deploying%20Intel%20SecL%20Use%20Cases%20Using%20Helm.md#deploy-skc-library) to deploy SKC Library

### Validate POD launch
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
         - key: <node label of workernode>
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

### Isecl-Scheduler Configuration
* Create a file called kube-scheduler-configuration.yml
```yaml
---
apiVersion: kubescheduler.config.k8s.io/v1beta2
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: "/etc/kubernetes/scheduler.conf"
profiles:
  - plugins:
      filter:
        enabled:
          - name: "NodePorts"
          - name: "NodeResourcesFit"
          - name: "VolumeBinding"
          - name: "NodeAffinity"
          - name: "NodeName"
      score:
        enabled:
          - name: "NodeResourcesBalancedAllocation"
            weight: 1
extenders:
  - urlPrefix: "https://127.0.0.1:30888/"
    filterVerb: "filter"
    weight: 5
    enableHTTPS: true
```

* Make directory /opt/isecl-k8s-extensions and copy kube-scheduler-configuration.yaml file into this newly created directory

* Add kube-scheduler-configuration.yml under kube-scheduler section /etc/kubernetes/manifests/kube-scheduler.yaml as mentioned below
```console
	spec:
          containers:
	  - command:
            - kube-scheduler
            - --config=/opt/isecl-k8s-extensions/kube-scheduler-configuration.yml
```

* Add mount path for isecl extended scheduler under container section /etc/kubernetes/manifests/kube-scheduler.yaml as mentioned below
```console
	containers:
		- mountPath: /opt/isecl-k8s-extensions
		name: extendedsched
		readOnly: true
```

Pod should be in running state and launched on the host as per values in pod.yml. Validate by running below command on sgx host:
```
	docker ps
```

### Isecl-Scheduler Configuration
* Create a file called kube-scheduler-configuration.yml
```yaml
---
apiVersion: kubescheduler.config.k8s.io/v1beta2
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: "/etc/kubernetes/scheduler.conf"
profiles:
  - plugins:
      filter:
        enabled:
          - name: "NodePorts"
          - name: "NodeResourcesFit"
          - name: "VolumeBinding"
          - name: "NodeAffinity"
          - name: "NodeName"
      score:
        enabled:
          - name: "NodeResourcesBalancedAllocation"
            weight: 1
extenders:
  - urlPrefix: "https://127.0.0.1:30888/"
    filterVerb: "filter"
    weight: 5
    enableHTTPS: true
```

* Make directory /opt/isecl-k8s-extensions and copy kube-scheduler-configuration.yaml file into this newly created directory

* Add kube-scheduler-configuration.yml under kube-scheduler section /etc/kubernetes/manifests/kube-scheduler.yaml as mentioned below
```console
	spec:
          containers:
	  - command:
            - kube-scheduler
            - --config=/opt/isecl-k8s-extensions/kube-scheduler-configuration.yml
```

* Add mount path for isecl extended scheduler under container section /etc/kubernetes/manifests/kube-scheduler.yaml as mentioned below
```console
	containers:
		- mountPath: /opt/isecl-k8s-extensions
		name: extendedsched
		readOnly: true
```

* Add volume path for isecl extended scheduler under volumes section /etc/kubernetes/manifests/kube-scheduler.yaml as mentioned below
```console
	spec:
	volumes:
	- hostPath:
		path: /opt/isecl-k8s-extensions
		type: ""
		name: extendedsched
```

* Restart Kubelet which restart all the k8s services including kube base scheduler
```console
	systemctl restart kubelet
```

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
repo init -u https://github.com/intel-secl/build-manifest.git -m manifest/skc.xml -b refs/tags/v5.0.0
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

#### Deploy TAA

Copy taa.env and sgx-agent bin file from k8s directory to a directory in SGX compute node.
* Update taa.env
	* Update CMS_BASE_URL as https://<controlplane-ip>:30445/cms/v1
	* Update CMS_TLS_CERT_SHA384 

* Install TAA

```
./sgx-agent-v5.0.0.bin
```

#### To uninstall TAA

```
taa uninstall --purge
```

#### Deploy Attestation Policy Service(APC)

Copy apc.env and apc-v5.0.0.bin from k8s directory to a directory in Master Node.
* Update apc.env
	* Update CMS_TLS_CERT_SHA384
	* Update CMS_BASE_URL as https://<controlplane-ip>:30445/cms/v1/
	* Update AAS_BASE_URL as https://<controlplane-ip>:30444/aas/v1
	* Update the bearer token 

* Install APC

```
./apc-v5.0.0.bin
```

#### To uninstall APC

```
apc uninstall --purge
```

### Attestation Validation Steps

#### Validate Attestation flow in Passport mode:
```
    1. Provision KBS with a key, key transfer policy and CSP CMS root-ca:
        a) Create a key transfer policy with SGX attributes. Refer KBS swagger docs for API spec.
        b) Create a key and associate it with key transfer policy id from above step. Refer KBS swagger docs for API spec.
            Take note of keys transfer_link as it is needed for fetching key in next steps.
        c) Copy CSP CMS root-ca to KBS trusted-ca directory and restart KBS./etc/kbs/certs/trustedca -- change file permission chown kbs:kbs file name

    2. Provision TAA with attestation token:
        a) Invoke TAA `fetch-token` command to request attestation token from APS.

           taa fetch-token -a <APS base url> -t <custom token to access APS attestation-token API>

    3. Validate key transfer using attestation token acquired in above step.
        a) Invoke TAA `fetch-key` command to request key from KBS.

        taa fetch-key -k <KBS key-transfer url> -m passport -t <sgx auth token for KBS>
```

#### Validate Attestation flow in Background-Check mode:
```
    1. Provision APS with a SGX attestation policy and Enterprise CMS root-ca:
        a) Create an attestation policy for an SGX enclave. Invoke APC `create-sgx-enclave-policy` command.
        apc create-sgx-enclave-policy -l <policy_label> -s <mrsigner_value> -m <mrenclave_value> -p <product_id> -n <isv_svn>`
pem file path: /etc/apc/certs/policy-sign/policy-signing-cert.pem
        b) Upload APC Policy-Signing certificate to APS. Refer APS swagger docs for API spec.
        c) Copy Enterprise CMS root-ca to APS trusted-ca directory and restart APS.
        d) Upload SGX enclave policy to APS. Refer APS swagger docs for API spec.

    2. Provision KBS with a key, key transfer policy and CSP CMS root-ca:
        a) Create a key transfer policy with SGX enclave policy id acquired in above step. Refer KBS swagger docs for API spec.
        b) Create a key and associate it with key transfer policy id from above step. Refer KBS swagger docs for API spec.
            Take note of keys transfer_link as it is needed for fetching key in next steps.
        c) Copy CSP CMS root-ca to KBS trusted-ca directory and restart KBS.

    3. Validate key transfer from TAA.
        a) Invoke TAA `fetch-key` command to request key from KBS.

           taa fetch-key -k <KBS key-transfer url> -m background-check -t <sgx auth token for KBS>
```
