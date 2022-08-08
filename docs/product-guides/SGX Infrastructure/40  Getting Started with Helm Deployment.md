## Getting Started
Below steps guide in the process for installing isecl-helm charts on a kubernetes cluster.

### Supported Operating Systems

The Intel® Security Libraries Attestation Policy Creator supports:

Red Hat Enterprise Linux 8.4

### Recommended Hardware

-   2 vCPUs

-   RAM: 8 GB

-   100 GB

-   Additional memory and disk space may be required depending on the size of images to be encrypted

### Pre-requisites

* Non Managed Kubernetes Cluster up and running
* Helm 3 installed
  ```shell
  curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
  chmod 700 get_helm.sh
  ./get_helm.sh
  ```

* For building container images Refer here for [instructions](https://github.com/intel-secl/docs/blob/v4.2/develop/docs/quick-start-guides/Foundational%20%26%20Workload%20Security%20-%20Containerization/5Build.md)

* Setup NFS, Refer [instructions](https://github.com/intel-innersource/applications.security.isecl.engineering.helm-charts/blob/v5.0/develop/NFS-Setup.md) for setting up and configuring NFS Server

### Installing isecl-helm charts

* Add the isecl-helm charts in helm chart repository
```
helm repo add isecl-helm https://intel-secl.github.io/helm-charts
helm repo update
```

* To find list of avaliable charts
```shell script
helm search repo
```

### Support Details

| Kubernetes        | Details                                                      |
| ----------------- | ------------------------------------------------------------ |
| Cluster OS        | *RedHat Enterprise Linux 8.x* <br/>*Ubuntu 20.04*            |
| Distributions     | Any non-managed K8s cluster                                  |
| Versions          | v1.23                                                        |
| Storage           | NFS                                                          |
| Container Runtime | *docker*,*CRI-O*<br/> |

### Update `values.yaml` for Use Case chart deployments

* The images are built on the build machine and images are pushed to a registry tagged with `release_version`(e.g:v5.0.0) as version for each image
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

By default Nodeport is supported for all ISecl services deployed on K8s, ingress can be enabled by setting the *enable* to true under ingress in values.yaml of
usecase charts.

Note: The values.yaml file can be left as is for jobs.
