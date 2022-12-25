# Deploying Intel SecL-DC

Beginning in the 4.2 release, Intel SecL-DC is deployed using Helm charts onto a Kubernetes platform.  Deployment using binary installers has been deprecated.

## Prerequisites for Deployment

- A Kubernetes cluster (version 1.23.1 or higher) must be available.  Hosts to be attested should be worker nodes in the Kubernetes cluster.  Ensure that kubectl is configured to work from the system where you will execute the deployment.  For default installations, a multi-node cluster (at least one controller node and at least one worker node) is recommended.  A single-node installation (all Kubernetes services and all workloads running on a single node, as in microk8s) will require manual adjustments to node labels and pod tolerations.  Worker nodes to be attested must meet the hardware security technology requirements for the use cases to be enabled.

- Helm 3 must be installed on the system from which the deployment will be launched. 
  
  ```
  curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
  chmod 700 get_helm.sh
  ./get_helm.sh 
  ```

- An NFS server is required with at minimum 2Gi of storage space available (10Gi is recommended for deployments managing up to 10,000 hosts.  50gi is recommended for deployments managing up to 100,000 hosts).

- All Intel SecL-DC container images must be pushed to a container registry using the appropriate version tag

- All worker nodes to be attested must have a supported combination of hardware security technologies enabled (see previous section).
- All worker nodes to be attested must be labelled according to the security technologies enabled (specifically tboot vs Secure Boot)
  - Label any tboot-enabled worker nodes with the label "TXT-ENABLED"
    ```
    kubectl label nodes <node name> node.type=TXT-ENABLED
    ```

  - Label any Secure Boot-enabled worker nodes with "SUEFI-ENABLED"
    ```
    kubectl label nodes <node name> node.type=SUEFI-ENABLED
    ```

### Setting up for Helm deployment

Create a namespace or use the namespace used for helm deployment. 

  ```
  kubectl create ns isecl
  ```

???+ note 
    ISecL Scheduler and Admission Controller secrets are required for Trusted Workload Placement and Workload Security releated usecases

### Create Secrets for ISecL Scheduler TLS Key-pair
ISecl Scheduler runs as https service, therefore it needs TLS Keypair and tls certificate needs to be signed by K8s CA, inorder to have secure communication between K8s base scheduler and ISecl K8s Scheduler.
The creation of TLS keypair is a manual step, which has to be done prior deplolying the helm for Trusted Workload Placement usecase. 
Following are the steps involved in creating tls cert signed by K8s CA.

  ```
  mkdir -p /tmp/k8s-certs/tls-certs && cd /tmp/k8s-certs/tls-certs
  openssl req -new -days 365 -newkey rsa:4096 -addext "subjectAltName = DNS:<Controlplane hostname>" -nodes -text -out server.csr -keyout server.key -sha384 -subj "/CN=ISecl Scheduler TLS Certificate"

  cat <<EOF | kubectl apply -f -
  apiVersion: certificates.k8s.io/v1
  kind: CertificateSigningRequest
  metadata:
    name: isecl-scheduler.isecl
  spec:
    request: $(cat server.csr | base64 | tr -d '\n')
    signerName: kubernetes.io/kube-apiserver-client
    usages:
    - client auth
  EOF
 
  kubectl certificate approve isecl-scheduler.isecl
  kubectl get csr isecl-scheduler.isecl -o jsonpath='{.status.certificate}' \
    | base64 --decode > server.crt
  kubectl create secret tls isecl-scheduler-certs --cert=/tmp/k8s-certs/tls-certs/server.crt --key=/tmp/k8s-certs/tls-certs/server.key -n isecl
  ```

???+ note 
    CSR needs to be deleted if we want to regenerate isecl-scheduler-certs secret with command `kubectl delete csr isecl-scheduler.isecl`

### Create Secrets for Admission controller TLS Key-pair
Create admission-controller-certs secrets for admission controller deployment

  ```
  mkdir -p /tmp/adm-certs/tls-certs && cd /tmp/adm-certs/tls-certs
  openssl req -new -days 365 -newkey rsa:4096 -addext "subjectAltName = DNS:admission-controller.isecl.svc" -nodes -text -out server.csr -keyout server.key -sha384 -subj "/CN=system:node:<nodename>;/O=system:nodes"

  cat <<EOF | kubectl apply -f -
  apiVersion: certificates.k8s.io/v1
  kind: CertificateSigningRequest
  metadata:
    name: admission-controller.isecl
  spec:
    groups:
    - system:authenticated
    request: $(cat server.csr | base64 | tr -d '\n')
    signerName: kubernetes.io/kubelet-serving
    usages:
    - digital signature
    - key encipherment
    - server auth
  EOF

  kubectl certificate approve admission-controller.isecl
  kubectl get csr admission-controller.isecl -o jsonpath='{.status.certificate}' \
      | base64 --decode > server.crt
  kubectl create secret tls admission-controller-certs --cert=/tmp/adm-certs/tls-certs/server.crt --key=/tmp/adm-certs/tls-certs/server.key -n isecl

  ```

Generate CA Bundle
  ```
  kubectl config view --raw --minify --flatten -o jsonpath='{.clusters[].cluster.certificate-authority-data}'
  ```

Add the output base64 encoded string to value in caBundle sub field of admission-controller in usecase/trusted-workload-placement/values.yml in case of usecase deployment chart.

???+ note 
    CSR needs to be deleted if we want to regenerate admission-controller-certs secret with command `kubectl delete csr admission-controller.isecl` 


### Create Secrets for KBS KMIP Certificates

Copy KMIP Client Certificate, Client Key and Root Certificate from KMIP Server to Node where KMIP secrets needs to be generated.

  ```
  kubectl create secret generic kbs-kmip-certs -n isecl --from-file=client_certificate.pem=<KMIP Client Certificate Path> --from-file=client_key.pem=<KMIP Client Key Path> --from-file=root_certificate.pem=<KMIP Root Certificate Path>
  ``` 

???+ note 
    KBS KMIP Certificate is required for Workload Security usecase

## Retrieving Helm charts

The Intel SecL-DC Helm charts are hosted in a github Helm repo:

  ```
  helm repo add isecl-helm https://intel-secl.github.io/helm-charts/
  helm repo update
  ```

The list of available charts on the added repo can be shown using the "search repo" command:

  ``` 
  helm search repo --versions
  ```

## Persistent Storage

Several Intel SecL-DC services require persistent storage for databases, configuration settings, and logs.  By default the Helm deployments are configured to use NFS storage for these persistent volumes.

On the NFS server, execute the provided setup-nfs.sh script to set up the needed persistent volume folder structure and share the mount points.

  ```
  curl -fsSL -o setup-nfs.sh https://raw.githubusercontent.com/intel-secl/helm-charts/v4.2.0-Beta/setup-nfs.sh
  chmod +x setup-nfs.sh
  ./setup-nfs.sh /mnt/nfs_share 1001 <ip or hostname of the control plane>
  ```

All the pods running on nodes in cluster need access to NFS mount path via PVC e.g `/mnt/nfs_share`. 
This requires user to add IPs/FQDN names to be added to /etc/exports file on system where NFS server is installed
Add all the IPs or FQDN names of each of nodes in K8s cluster in /etc/exports file in system where NFS is server is installed and configured using above script.
e.g For 3 node cluster with FQDN names control-plane.com(control plane), node1.com and node2.com(nodes)
  ```
  cat /etc/exports
  /mnt/nfs_share/isecl/        control-plane.com(rw,sync,no_all_squash,root_squash)
  /mnt/nfs_share/isecl/        node1.com(rw,sync,no_all_squash,root_squash)
  /mnt/nfs_share/isecl/        node2.com(rw,sync,no_all_squash,root_squash)
  ```

???+ note 
    Even if a new node is joined to the cluster, the IP/FQDN name should be added to /etc/exportfs file at system where NFS server is running

After making the changes in `/etc/exports` file run the below command
  ```
  exportfs -arv
  ``` 

## Default Service and Agent Mount Paths

```yaml
#Certificate-Management-Service
Config: <NFS-vol-base-path>/isecl/cms/config
Logs: <NFS-vol-base-path>/isecl/cms/logs

#Authentication Authorization Service
Config: <NFS-vol-base-path>/isecl/aas/config
Logs: <NFS-vol-base-path>/isecl/aas/logs
Pg-data: <NFS-vol-base-path>/isecl/aas/db

#Host Attestation Service
Config: <NFS-vol-base-path>/isecl/hvs/config
Logs: <NFS-vol-base-path>/isecl/hvs/logs
Pg-data: <NFS-vol-base-path>/usr/local/kube/data/hvs

#Integration-Hub
Config: <NFS-vol-base-path>/isecl/ihub/config
Log: <NFS-vol-base-path>/isecl/ihub/logs

#Workload Service
Config: <NFS-vol-base-path>/isecl/wls/config
Logs: <NFS-vol-base-path>/isecl/wls/log
Pg-data: <NFS-vol-base-path>/usr/local/kube/data/wls

#Key-Broker-Service
Config: <NFS-vol-base-path>/isecl/kbs/config
Log: <NFS-vol-base-path>/isecl/kbs/logs
Opt: <NFS-vol-base-path>/isecl/kbs/kbs/opt

#Trust Agent:
Config: /etc/trustagent/v5.1.0/
Logs:  /var/log/trustagent/
tpmrm: /dev/tpmrm0
txt-stat: /usr/sbin/txt-stat
ta-hostname-path: /etc/hostname
ta-hosts-path: /etc/hosts

#Workload Agent:
Config: /etc/workload-agent/v5.1.0/
Logs: /var/log/workload-agent
```

???+ note 
    Persistent storage will not be deleted if the Helm deployment is uninstalled.  Be sure to delete the created folders on the NFS server (/mnt/nfs_share/ by default) as well as the local storage used by the Trust Agent (/etc/trustagent and /var/log/trustagent on each worker node) and Workload Agent (/etc/workload-agent and /var/log/workload-agent on each worker node) if a "fresh" installation is needed.  Rerun the setup-nfs.sh script after deleting the shared folders to recreate the empty folder structure.


???+ note 
    If the Endorsement Certificate (EC) Pre-Registration feature is enabled for the HVS:

```
  requireEKCertForHostProvision: true # If set to true, worker node EK certificate should be registered in HVS DB, for AIK provisioning step of TA. (Allowed values: `true`\`false`)
  verifyQuoteForHostRegistration: true # If set to true, when the worker node is being registered to HVS, quote verification will be done. Default value is false. (Allowed values: `true`\`false`)
```

The worker node TPM Endorsement Certificates (ECs) must be registered with the HVS before the Trust Agent daemonsets will fully deploy.  This is because a necessary step during Trust Agent provisioning will be denied access until the HVS has the EC registered in its database.  See the Endorsement Certificate Pre-Registration section for more details about this feature.

The Trust Agent daemonset will continue to reattempt deployment on all worker nodes, so the deployment will self-recover after the EC registration happens.  Trust Agent deployment failures should be expected if the Trust Agent is deployed before this step is completed.

---

# Default Service Ports

For both Single node and multi-node deployments following ports are used. All services exposing APIs will use the below ports

```yaml
CMS: 30445
AAS: 30444
HVS: 30443
WLS: 30447
IHUB: None
KBS: 30448
K8s-scheduler: 30888
K8s-controller: None
TA: 31443
WLA: None
```
