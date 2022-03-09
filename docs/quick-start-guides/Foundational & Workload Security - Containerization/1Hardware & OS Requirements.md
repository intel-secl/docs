# Hardware & OS Requirements

## Physical Server requirements

* Intel® SecL-DC  supports and uses a variety of Intel security features, but there are some key requirements to consider before beginning an installation. Most important among these is the Root of Trust configuration. This involves deciding what combination of TXT, Boot Guard, tboot, and UEFI Secure Boot to enable on platforms that will be attested using Intel® SecL.

???+ note 
    At least one "Static Root of Trust" mechanism must be used (TXT and/or BtG). For Legacy BIOS systems, tboot must be used. For UEFI mode systems, UEFI SecureBoot must be used* Use the chart below for a guide to acceptable configuration options. Only dTPM is supported on Intel® SecL-DC platform hardware. 

  ![hardware-options](./images/trusted-boot-options.PNG)

## Machines

* Build Machine

* K8s control-plane Node Setup on CSP (VMs/Physical Nodes + TXT/SUEFI enabled Physical Nodes)

* K8s control-plane Node Setup on Enterprise (VMs/Physical Nodes)

## OS Requirements

* `RHEL 8.4`/`Ubuntu 20.04` for build

* `RHEL 8.4`/`Ubuntu 20.04` for K8s cluster deployments

???+ note 
    Foundational & Workload Security solution is built, installed and tested with root privileges. Please ensure that all the following instructions are executed with root privileges

## Container Runtime

* Docker-20.10.8
* CRIO-1.21.1

## K8s Distributions

* Single Node: 

  A single-node deployment uses `microk8s` to deploy the entire control plane in a pod on a single hardware machine. This is best for POC or demo environments, but can also be used when integrating Intel SecL with another application that runs on a virtual machine

* Multi Node: 

  A multi-node deployment is a more typical Kubernetes architecture, where the Intel SecL management plane is simply deployed as a Pod, with the Intel SecL agents (the WLA and the TA, depending on use case) deployed as a DaemonSet. `kubeadm` is the supported multi node distribution as of today.

## Storage

* `hostPath` in case of single-node `microk8s` for all services and agents
* `NFS` in case of multi-node `kubeadm` for services and `hostPath` for Trust Agent and Workload Agent

