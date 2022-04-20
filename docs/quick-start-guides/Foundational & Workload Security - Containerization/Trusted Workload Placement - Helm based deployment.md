# Deployment

## Pre-requisites

* Ensure a docker registry is running locally or remotely. 

* Push all container images to docker registry. Example below

  ```shell
  # Without TLS enabled
  skopeo copy oci-archive:<oci-image-tar-name> docker://<registry-ip/hostname>:<registry-port>/<image-name>:<image-tag> --dest-tls-verify=false
  
  # With TLS enabled
  skopeo copy oci-archive:<oci-image-tar-name> docker://<registry-ip/hostname>:<registry-port>/image-name>:<image-tag>
  ```

* Non Managed Kubernetes Cluster up and running

* On each worker node with `TXT/BTG` enabled and registered to K8s control-plane, the following pre-req needs to be done on `RHEL-8.4`/`Ubuntu-20.04` systems

  * `Tboot-1.10.5` or later to be installed for servers using TXT and not using Secure Boot. [Tboot installation Details](../../product-guides/Foundational%&%Workload%Security/../Foundational%20&%20Workload%20Security/23%20Hardware%20Security%20Technology%20Considerations.md)

  * Only for `Ubuntu-20.04`, run the following commands

 ```shell
     $ modprobe msr
 ```

* The K8s cluster admin configure the existing bare metal worker nodes or register fresh bare metal worker nodes with labels. For example, a label like `node.type: SGX-SUEFI-ENABLED` can be used by the cluster admin to distinguish the baremetal worker node and the same label can be used in ISECL Agent pod configuration to schedule on all worker nodes marked with the label.

  ```shell
  #Label node , below example is for TXT & SUEFI enabled host
  kubectl label node <node-name> node.type=SUEFI-ENABLED
  
  #Label node , below example is for SGX & TXT enabled host
  kubectl label node <node-name> node.type=TXT-ENABLED
  ```
  
  * Helm 3 installed
   ```shell
   curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
   chmod 700 get_helm.sh
   ./get_helm.sh
   ```

  * Add the isecl-helm charts in helm chart repository
  ```shell
   helm repo add isecl-helm https://intel-secl.github.io/helm-charts/
   helm repo update
   ```

  * To find list of avaliable charts
   ```shell
   helm search repo
   ```

  * NFS setup
  
    - NFS server setup
        ```shell 
         curl -fsSL -o setup-nfs.sh https://raw.githubusercontent.com/intel-secl/helm-charts/v4.2.0-Beta/setup-nfs.sh
         chmod +x setup-nfs.sh
        ./setup-nfs.sh /mnt/nfs_share 1001 <ip>
        ```
        
        Either each node IPs/Hostnames or node subnet range in cluster should be added in /etc/exports file in NFS server
        
    - All nodes in cluster needs nfs-client to be installed
    
        - For ubuntu ```apt install nfs-common```
        
        - For rhel ```dnf install nfs-utils```
      

### Commands to fetch EK certicate and Issuer on worker node

The below obtained EK certificate can be used to upload to HVS DB, for allow registration of specific nodes use case.
If a specific host has to be allowed to register to HVS, then, that host EK certificate should be uploaded to HVS using /hvs/tpm-endorsements API

For RHEL OS
```
yum install tpm2-tools
tpm2_nvread -P hex:<owner secret> -x 0x1c00002 -a 0x40000001 -f ekcert.der or tpm2_nvread -P hex:<owner secret> -C 0x40000001 -o ekcert.der  0x1c00002
openssl x509 -inform der -in ekcert.der | base64 | tr -d " \t\n\r"

To get certificate Issuer
openssl x509 -inform der -in ekcert.der --text | grep -Po 'CN =\K.*'
```

Note: Above "owner secret" is TPM owner secret of 40 character hex string

### Use Case Helm Charts 

#### Foundational Security Usecases

| Use case                                | Helm Charts                                        |
| --------------------------------------- | -------------------------------------------------- |
| Trusted Workload Placement - Containers | *cms*<br />*aas*<br />*hvs*<br />*isecl-controller*<br />*isecl-scheduler*<br />*admission-controller*<br />*ihub*<br />*ta* |


### Setting up for Helm deployment

Create a namespace or use the namespace used for helm deployment.
```kubectl create ns isecl```

##### Create Secrets for ISecL Scheduler TLS Key-pair
ISecl Scheduler runs as https service, therefore it needs TLS Keypair and tls certificate needs to be signed by K8s CA, inorder to have secure communication between K8s base scheduler and ISecl K8s Scheduler.
The creation of TLS keypair is a manual step, which has to be done prior deplolying the helm for Trusted Workload Placement usecase. 
Following are the steps involved in creating tls cert signed by K8s CA.
```shell
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

##### Create Secrets for Admission controller TLS Key-pair
Create admission-controller-certs secrets for admission controller deployment
```shell
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
```shell
kubectl config view --raw --minify --flatten -o jsonpath='{.clusters[].cluster.certificate-authority-data}'
```
Add the output base64 encoded string to value in caBundle sub field of admission-controller in usecase/trusted-workload-placement/values.yml in case of usecase deployment chart.

*Note*: CSR needs to be deleted if we want to regenerate admission-controller-certs secret with command `kubectl delete csr admission-controller.isecl` 
## Installing isecl-helm charts

* Individual service and jobs charts
* Usecase charts(Umbrella charts)

### Bearer-token and Users and role permission generation.

After Authentication and Authorization Service(AAS) pod is bootstrapped and ready, a number of roles and user accounts must be generated. 
Most of these accounts will be service users, used by the various Intel® SecL services to work together. 
Another set of users will be used for installation permissions, and a final administrative user will be created to provide the initial authentication interface for the actual human user. 
The administrative user can be used to create additional users with appropriately restricted roles based on organizational needs.

Creating these required users and roles is facilitated by a job called aas-manager that will accept credentials and roles and permission settings from a aas-manager json file and automate the process.
aas-manager job gets trigged automatically once aas pod is ready. 
This job generates a secret called bearer-token in a given namespace for given a set of values in superAdminUsername and superAdminPassword in values.yaml file, has necessary roles and permissions required for bootstraping all the services.
By default this token will be valid for two hours, therfore it is refresed every 5 minutes as part of cronjob. User need not worry to refresh the bearer token while performing any of the setup tasks.
This job can be deleted and rerun for generating bearer-token secret on demand if at all required.

User needs to update the appropriate values for credential in usecase chart values.yaml file or values.yaml file in case of deploying individual charts.

The job will automatically generate the following users:

Verification Service User

Integration Hub Service User

Global Admin User

Installation User

These user accounts will be used during installation of several of the Intel® SecL-DC applications. In general, whenever credentials are required by an installation answer file.

The Global Admin user account has all roles for all services. This is a default administrator account that can be used to perform any task, including creating any other users. 
In general this account is useful for POC installations, but in production it should be used only to create user accounts with more restrictive roles. 
The administrator credentials should be protected and not shared.

### Individual helm chart deployment (using service/job charts)

The helm chart support Nodeports for services, to support ingress model. 

#### Update `values.yaml` for Use Case chart deployments
* The images are built on the build machine and images are pushed to a registry tagged with `release_version`(e.g:v4.2.0) as version for each image
* The NFS server and setup either using sample script or by the user itself
* The K8s non-managed cluster is up and running
* Helm 3 is installed

The `values.yaml` file in each of the charts is used for defining all the values required for an individual chart deployment. Most of the values are already defined
and yet there are few values needs to be defined by the user, these are marked by placeholder with the name \<user input\>.  
e.g 
```yaml
image:
  name: <user input> # The image name with which AAS-MANAGER image is pushed to registry

controlPlaneHostname: <user input> # K8s control plane IP/Hostname<br> (**REQUIRED**)

storage:
  nfs:
    server: <user input> # The NFS Server IP/Hostname
```

Note: The values.yaml file can be left as is for jobs mentioned below:

cleanup-secrets

aasdb-cert-generator

hvsdb-cert-generator

aas-manager

nats

By default Nodeport is supported for all ISecl services deployed on K8s, ingress can be enabled by setting the *enable* to true under ingress in values.yaml of
individual services

#### Deployment steps along with the sequence to be followed

Services which has database deployment associated with it needs db ssl certificates to be generated as secrets, this is done by deploying \<service\>db-cert-generator job.

Download the values.yaml for each of the services.

```shell
curl -fsSL -o cleanup-secrets.yaml https://raw.githubusercontent.com/intel-secl/helm-charts/v4.2.0-Beta/jobs/cleanup-secrets/values.yaml
curl -fsSL -o cms.yaml https://raw.githubusercontent.com/intel-secl/helm-charts/v4.2.0-Beta/services/cms/values.yaml
curl -fsSL -o aasdb-cert-generator.yaml https://raw.githubusercontent.com/intel-secl/helm-charts/v4.2.0-Beta/jobs/aasdb-cert-generator/values.yaml
curl -fsSL -o aas.yaml https://raw.githubusercontent.com/intel-secl/helm-charts/v4.2.0-Beta/services/aas/values.yaml
curl -fsSL -o aas-manager.yaml https://raw.githubusercontent.com/intel-secl/helm-charts/v4.2.0-Beta/jobs/aas-manager/values.yaml
curl -fsSL -o hvsdb-cert-generator.yaml https://raw.githubusercontent.com/intel-secl/helm-charts/v4.2.0-Beta/jobs/hvsdb-cert-generator/values.yaml
curl -fsSL -o hvs.yaml https://raw.githubusercontent.com/intel-secl/helm-charts/v4.2.0-Beta/services/hvs/values.yaml
curl -fsSL -o trustagent.yaml https://raw.githubusercontent.com/intel-secl/helm-charts/v4.2.0-Beta/services/ta/values.yaml
curl -fsSL -o isecl-controller.yaml https://raw.githubusercontent.com/intel-secl/helm-charts/v4.2.0-Beta/services/isecl-controller/values.yaml
curl -fsSL -o ihub.yaml https://raw.githubusercontent.com/intel-secl/helm-charts/v4.2.0-Beta/services/ihub/values.yaml
curl -fsSL -o isecl-scheduler.yaml https://raw.githubusercontent.com/intel-secl/helm-charts/v4.2.0-Beta/services/isecl-scheduler/values.yaml
curl -fsSL -o admission-controller.yaml https://raw.githubusercontent.com/intel-secl/helm-charts/v4.2.0-Beta/services/admission-controller/values.yaml
```

Update all the downloaded values.yaml with appropriate values.

Following are the steps need to be run for deploying individual charts.
```shell
helm pull isecl-helm/cleanup-secrets
helm install cleanup-secrets -f cleanup-secrets.yaml isecl-helm/cleanup-secrets -n isecl --create-namespace
helm pull isecl-helm/cms
helm install cms isecl-helm/cms -n isecl -f cms.yaml
helm pull isecl-helm/aasdb-cert-generator
helm install aasdb-cert-generator isecl-helm/aasdb-cert-generator -f aasdb-cert-generator.yaml  -n isecl
helm pull isecl-helm/aas
helm install aas isecl-helm/aas -n isecl -f aas.yaml
helm pull isecl-helm/aas-manager
helm install aas-manager isecl-helm/aas-manager -n isecl -f aas-manager.yaml
helm pull isecl-helm/hvsdb-cert-generator
helm install hvsdb-cert-generator isecl-helm/hvsdb-cert-generator -f hvsdb-cert-generator.yaml -n isecl
helm pull isecl-helm/hvs
helm install hvs isecl-helm/hvs -n isecl -f hvs.yaml
helm pull isecl-helm/trustagent 
helm install trustagent isecl-helm/trustagent -n isecl -f trustagent.yaml
helm pull isecl-helm/isecl-controller
helm install isecl-controller isecl-helm/isecl-controller -n isecl -f isecl-controller.yaml
helm pull isecl-helm/ihub
helm install ihub isecl-helm/ihub -n isecl -f ihub.yaml
helm pull isecl-helm/isecl-scheduler
helm install isecl-scheduler isecl-helm/isecl-scheduler -n isecl -f isecl-scheduler.yaml
helm pull isecl-helm/admission-controller
helm install isecl-scheduler isecl-helm/admission-controller -n isecl -f admission-controller.yaml
```

To uninstall a chart
```shell
helm uninstall <release-name> -n isecl
```

To list all the helm chart deployments 
```shell
helm list -A
```

Cleanup steps that needs to be done for a fresh deployment
* Uninstall all the chart deployments
* cleanup the secrets for isecl-scheduler-certs and admission-controller-certs. ```kubectl delete secret -n <namespace> isecl-scheduler-certs admission-controller-certs```
* Cleanup the data at NFS mount and TA nodes.
* Remove all objects(secrets, rbac, clusterrole, service account) related namespace related to deployment ```kubectl delete ns <namespace>```. 

**Note**: 
    
    Before redeploying any of the chart please check the pv and pvc of corresponding deployments are removed. Suppose
    if you want to redeploy aas, make sure that aas-logs-pv, aas-logs-pvc, aas-config-pv, aas-config-pvc, aas-db-pv, aas-db-pvc, aas-base-pvc are removed successfully.
    Command: ```kubectl get pvc -A``` && ```kubectl get pv -A```
    
    helm uninstall command wont remove secrets by itself, one has to manually delete secrets or use cleanup-secrets to cleanup all the secrets.
    
### Usecase based chart deployment (using umbrella charts)

Download the values.yaml file for Trusted Workload Placement usecase chart
```shell
curl -fsSL -o values.yaml https://raw.githubusercontent.com/intel-secl/helm-charts/v4.2.0-Beta/usecases/trusted-workload-placement/values.yaml
```

#### Update `values.yaml` for Use Case chart deployments

Some assumptions before updating the `values.yaml` are as follows:
* The images are built on the build machine and images are pushed to a registry tagged with `release_version`(e.g:v4.2.0) as version for each image
* The NFS server and setup either using sample script or by the user itself
* The K8s non-managed cluster is up and running
* Helm 3 is installed

The helm chart support Nodeports for services, to support ingress model. 
Enable the ingress by setting the value ingress enabled to true in values.yaml file

Update the ```hvsUrl, cmsUrl and aasUrl``` under global section according to the conifgured model.
e.g For ingress. hvsUrl: https://hvs.isecl.com/hvs/v2
    For Nodeport, hvsUrl: https://<controlplane-hosntam/IP>:30443/hvs/v2

#### Use Case charts Deployment

```shell
helm pull isecl-helm/Trusted-Workload-Placement
helm install <helm release name> isecl-helm/Trusted-Workload-Placement -f values.yaml --create-namespace -n <namespace>
```

> **Note:** If using a seprarate .kubeconfig file, ensure to provide the path using `--kubeconfig <.kubeconfig path>`

#### Configure kube-scheduler to establish communication with isecl-scheduler after successful deployment.

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

## Setup task workflow.

Check this document for available setup tasks for each of the services  [Setup tasks](setup-tasks.md) 
1. Edit the configmap of respective service where we want to run setup task. e.g ```kubectl edit cm cms -n isecl```
2. Add or Update all the variables required for setup tasks refer [here](setup-tasks.md)  for more details
3. Add *SETUP_TASK* variable in config map with one or more setup task names e.g ```SETUP_TASK: "download-ca-cert,download-tls-cert"```
4. Save the configmap
5. Some of the sensitive variables such as credentials, db credentials, tpm-owner-secret can be updated in secrets with the command 

    ```kubectl get secret -n <namespace> <secret-name> -o json | jq --arg val "$(echo <value> > | base64)" '.data["<variable-name>"]=$val' | kubectl apply -f -```
    
    e.g For updating the AAS_ADMIN_USERNAME in aas-credentials 
    
    ```kubectl get secret -n isecl aas-credentials -o json | jq --arg val "$(echo aaspassword | base64)" '.data["AAS_ADMIN_USERNAME"]=$val' | kubectl apply -f -``` 
   
6. Restart the pod by deleting it ```kubectl delete pod -n <namespace> <podname>```
7. Reset the configmap by removing SETUP_TASK variable 

      ```kubectl patch configmap -n <namespace> <configmap name> --type=json -p='[{"op": "remove", "path": "/data/SETUP_TASK"}]'```
        
        e.g For clearing SETUP_TASK variable in cms configmap
        
      ```kubectl patch configmap -n isecl cms --type=json -p='[{"op": "remove", "path": "/data/SETUP_TASK"}]'```
        
    
