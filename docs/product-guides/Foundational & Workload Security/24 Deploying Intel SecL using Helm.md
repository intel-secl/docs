# Deploying Intel SecL-DC

Beginning in the 4.2 release, Intel SecL-DC is deployed using Helm charts onto a Kubernetes platform.  Deployment using binary installers has been deprecated.

## Prerequisites for Deployment

- A Kubernetes cluster (version 1.22.2 or higher) must be available.  Hosts to be attested should be worker nodes in the Kubernetes cluster.  Ensure that kubectl is configured to work from the system where you will execute the deployment.  For default installations, a multi-node cluster (at least one controller node and at least one worker node) is recommended.  A single-node installation (all Kubernetes services and all workloads running on a single node, as in microk8s) will require manual adjustments to node labels and pod tolerations.  Worker nodes to be attested must meet the hardware security technology requirements for the use cases to be enabled.

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
```shell script
cat /etc/exports
/mnt/nfs_share/isecl/        control-plane.com(rw,sync,no_all_squash,root_squash)
/mnt/nfs_share/isecl/        node1.com(rw,sync,no_all_squash,root_squash)
/mnt/nfs_share/isecl/        node2.com(rw,sync,no_all_squash,root_squash)
```

*Note:* Even if a new node is joined to the cluster, the IP/FQDN name should be added to /etc/exportfs file at system where NFS server is running

After making the changes in `/etc/exports` file run the below command
```shell script
exportfs -arv
``` 

---
**NOTE**
Persistent storage will not be deleted if the Helm deployment is uninstalled.  Be sure to delete the created folders on the NFS server (/mnt/nfs_share/ by default) as well as the local storage used by the Trust Agent (/etc/trustagent and /var/log/trustagent on each worker node) and Workload Agent (/etc/workload-agent and /var/log/workload-agent on each worker node) if a "fresh" installation is needed.  Rerun the setup-nfs.sh script after deleting the shared folders to recreate the empty folder structure.
---

---
**NOTE**
If the Endorsement Certificate (EC) Pre-Registration feature is enabled for the HVS:

```
  requireEKCertForHostProvision: true # If set to true, worker node EK certificate should be registered in HVS DB, for AIK provisioning step of TA. (Allowed values: `true`\`false`)
  verifyQuoteForHostRegistration: true # If set to true, when the worker node is being registered to HVS, quote verification will be done. Default value is false. (Allowed values: `true`\`false`)
```

The worker node TPM Endorsement Certificates (ECs) must be registered with the HVS before the Trust Agent daemonsets will fully deploy.  This is because a necessary step during Trust Agent provisioning will be denied access until the HVS has the EC registered in its database.  See the Endorsement Certificate Pre-Registration section for more details about this feature.

The Trust Agent daemonset will continue to reattempt deployment on all worker nodes, so the deployment will self-recover after the EC registration happens.  Trust Agent deployment failures should be expected if the Trust Agent is deployed before this step is completed.

---

