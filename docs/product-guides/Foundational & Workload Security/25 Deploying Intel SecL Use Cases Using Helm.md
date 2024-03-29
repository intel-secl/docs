# Deploying Intel SecL Use Cases Using Helm

Intel SecL-DC provides combined Helm charts to deploy an entire use case using a single configuration file, rather than deploying each individual service.  **This is the recommended method for deployment.**

Available use case deployments include:

- Host Attestation (This enables Platform Integrity Attestation)
- Trusted Workload Placement (This deploys the Host Attestation use case and adds integration with Kubernetes to place worklaods on trusted workers)
  - The Trusted Workload Placement deployment can optionally be divided into separate deployments for the Cloud Service Provider and the Control Plane
    - The Control Plane deployment includes the CMS, AAS, HVS, and optionally supports NATS
    - The Cloud Service Provider deployment includes the Trust Agent, Integration Hub, Admission-Controller, Intel SecL Controller, and Intel SecL Scheduler
- Workload Security (this deploys Host Attestation and Trusted Workload Placement, and adds components to enable Workload Confidentiality)


## Deployment Steps

Configure the values.yaml answer file.  Each values.yaml file will contain "<user input>" prompts and a comment describing the information required.

## Use Case charts Deployment

Pull the Helm charts from the Helm repository and then deploy.

Set the release version
```
export VERSION=v5.1.0
```

### Host-Attestation:

```
helm pull isecl-helm/Host-Attestation --version $VERSION && tar -xzf Host-Attestation-$VERSION.tgz Host-Attestation/values.yaml
helm install host-attastation isecl-helm/Host-Attestation --version $VERSION -f Host-Attestation/values.yaml --create-namespace -n <namespace>
```

### Trusted Workload Placement

```
helm pull isecl-helm/Trusted-Workload-Placement --version $VERSION && tar -xzf Trusted-Workload-Placement-$VERSION.tgz Trusted-Workload-Placement/values.yaml
helm install trusted-workload-placement isecl-helm/Trusted-Workload-Placement --version $VERSION -f Trusted-Workload-Placement/values.yaml --create-namespace -n <namespace>
```

### Trusted Workload Placement - Cloud Service Provider Components Only

```
helm pull isecl-helm/Trusted-Workload-Placement-Cloud-Service-Provider --version $VERSION && tar -xzf Trusted-Workload-Placement-Cloud-Service-Provider-$VERSION.tgz Trusted-Workload-Placement-Cloud-Service-Provider/values.yaml
helm install twp-cloud-service-provider isecl-helm/Trusted-Workload-Placement-Cloud-Service-Provider --version $VERSION -f Trusted-Workload-Placement-Cloud-Service-Provider/values.yaml --create-namespace -n <namespace>
```

### Trusted Workload Placement - Control Plane Components Only

```
helm pull isecl-helm/Trusted-Workload-Placement-Control-Plane --version $VERSION && tar -xzf Trusted-Workload-Placement-Control-Plane-$VERSION.tgz Trusted-Workload-Placement-Control-Plane/values.yaml
helm install twp-control-plane isecl-helm/Twp-Control-Plane --version $VERSION -f Twp-Control-Plane/values.yaml --create-namespace -n <namespace>
```

### Workload Security 

```
helm pull isecl-helm/Workload-Security --version $VERSION && tar -xzf Workload-Security-$VERSION.tgz Workload-Security/values.yaml
helm install workload-security isecl-helm/Workload-Security --version $VERSION -f Workload-Security/values.yaml --create-namespace -n <namespace>
```

### Admission Controller
when a new worker node is joined/rebooted in the cluster, below steps can be done, to taint such nodes, by labelling it as untrusted, intiatially. Tainiting, doesn't allow scheduling of any pods on that worker node.

IHUB pulls the data from HVS, and pushes to ISECL Controller, based on the report status of the node, if the worker node is trusted, then that node will be untainted.

Node Joining or Node Rebooted When the worker node is being joined/rebooted in the k8s cluster, Untrusted:True NoExecute and NoSchedule taint would be added to the worker Node

To the K8s cluster, when a new worker node is being joined, such worker Nodes are tainted Set taintRebootedNodes to "true" in values.yml during helm deployment.

In the K8s cluster, if any of the worker node is Rebooted, such worker Nodes are tainted Set taintRegisteredNodes to "true" in values.yml during helm deployment.

???+ note By default, taintRebootedNodes and taintRegisteredNodes, will be false.

???+ note In no_proxy, add .svc,.svc.cluster.local, and then do kubeadm init

Upload image to registry The admission controller tar file that is present in k8s image folder should be uploaded to registry and update the image name in values.yml file during helm deployment.

# Usecase Workflows API Collections

The below allow to get started with workflows within Intel® SecL-DC for Foundational and Workload Security Usecases. More details available in [API Collections](https://github.com/intel-secl/utils/tree/v5.1/develop/tools/api-collections) repository

## Pre-requisites

* Postman client should be [downloaded](https://www.postman.com/downloads/) on supported platforms or on the web to get started with the usecase collections.

???+ note 
    The Postman API Network will always have the latest released version of the API Collections. For all releases, refer the github repository for [API Collections](https://github.com/intel-secl/utils/tree/v5.1/develop/tools/api-collections)

## Use Case Collections

| Use case               | Sub-Usecase                                   | API Collection     |
| ---------------------- | --------------------------------------------- | ------------------ |
| Foundational Security  | Host Attestation(RHEL)                              | ✔️                  |
|                        | Data Fencing  with Asset Tags(RHEL)                 | ✔️                  |
|                        | Trusted Workload Placement (Containers)  | ✔️ |
| Workload Security | Container Confidentiality with CRIO Runtime | ✔️                 

## Downloading API Collections

* Postman API Network for latest released: https://explore.postman.com/intelsecldc

  or 

* Github repo for all releases

  ```shell
  #Clone the github repo for api-collections
  git clone https://github.com/intel-secl/utils.git
  
  #Switch to specific release-version of choice
  cd utils/
  git checkout <release-version of choice>
  
  #Import Collections from
  cd tools/api-collections
  ```

???+ note 
    The postman-collections are also available when cloning the repos via build manifest under `utils/tools/api-collections`
