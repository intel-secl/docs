# Hardware & OS Requirements

## Machines

* Ubuntu 18.04/20.04 Build Machine

* RHEL 8.4 Build Machine for stack based deployment

* K8s control-plane Node Setup on CSP (VMs/Physical Nodes + SGX enabled Physical Nodes)

* K8s control-plane Setup on Enterprise (VMs/Physical Nodes)

## OS Requirements

* Ubuntu 18.04/20.04 for build

* RHEL 8.4 for build for stack based deployment

* RHEL 8.2 or Ubuntu 18.04/20.04 for K8s cluster deployments

* RHEL 8.4 for stack based K8s cluster deployments 

> Note 
    SKC Solution is built, installed and tested with root privileges. Please ensure that all the following instructions are executed with root privileges

## Container Runtime

* Docker

## K8s Distributions

* Single Node: `microk8s`
* Multi Node: `kubeadm`

## Storage

* `hostPath` in case of single-node `microk8s` for all services and agents
* `NFS` in case of multi-node `kubeadm` for services and `hostPath` for sgx_agent and skc_library

