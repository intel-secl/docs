# Upgrades

Upgrades for foundational security usecases is supported using helm upgrade mechanism. Upgrades are supported only for usecases
which are deployed using helm.

???+ note 
    Before performing any upgrade, Intel strongly recommends backing up the data mounted at NFS. 
    
## Upgrade Process

Upgrade process is done using helm upgrade command.

### Build

Intel assumes all services for any use-case are up and running before proceeding with the upgrade.

Push all the newer version of container images to registry. All oci images will be in k8s/container-images.

e.g

```
skopeo copy oci-archive:<oci-image-tar-name> docker://<registry-ip/hostname>:<registry-port>/<image-name>:<image-tag> --dest-tls-verify=false
```

------

#### Upgrade/Deploy

##### Add helm repo
```
helm repo add isecl-helm https://intel-secl.github.io/helm-charts
```

##### To search for helm repo with versions
```
helm search repo --versions
```

##### Download chart and values.yaml
  ```
  helm pull isecl-helm/<chart-name> && tar -xzf <chart-name>-$VERSION.tgz <chart-name>/values.yaml 
  e.g helm pull isecl-helm/Host-Attestation && tar -xzf Host-Attestation-$VERSION.tgz Host-Attestation/values.yaml  
  ```

##### Update the values.yaml, the values given in values.yaml should be same as that of given for currently deployed version. 
Charts with v5.1.0 has undergone changes from that of v5.0.0 and has changes in values.yaml file. The description provided in comments would help user to understand for setting appropriate values, the credentials for services given for v5.0.0 should be same for v5.1.0 as well. 

```yaml
versionUpgrade: true
currentVersion: "v5.0.0"
dbVersionUpgrade: false
image:
  dbVersionUpgradeImage: <Registry>/db-version-upgrade:v11-v14
```
 
Set the value for currentVersion under global section in values.yaml to the currently deployed version(if v5.0.0 is deployed currently then set the value as "v5.0.0")
 
Set the value for versionUpgrade to true. 
Set the value for dbVersionUpgrade to false, as there is no change in db image version.

##### Upgrade the charts with the below commands

```
   kubectl delete --all jobs -n <namespace> # Delete all the jobs, since jobs cannot be upgraded.
   kubectl scale deploy --all --replicas=0 -n <namespace> # Scale down all services, so that upgrade happens smoothly without any inconsistencies
   helm upgrade <release-name> isecl-helm/<chart-name> --version $VERSION -n <namespace> -f <chart-name>/values.yaml
   e.g helm upgrade host-attestation isecl-helm/Host-Attestation --version $VERSION  -n isecl -f Host-Attestation/values.yaml
```


For charts Trusted-Workload-Placement and Trusted-Workload-Placement-Cloud-Service-Provider, ISecl-Scheduler should be disconnected from K8s
base scheduler. This can be done by configuring in manifest of kube-scheduler as mentioned below, by commenting the *--config* option
```
  containers:
  - command:
    - kube-scheduler
    - --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
    - --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
    - --bind-address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --leader-elect=true
      #- --config=/opt/isecl-k8s-extensions/kube-scheduler-configuration.yml
```

Uncomment the *--config* option once upgrade is complete and all service pods are successfully running and jobs are completed.

### Rollback Services
Rollback is supported using helm rollback mechanism. The data at NFS will be automatically backed up using upgrade jobs during upgrade process, 
the backed up data will be stored in a versioned directory. The init containers at every pod will ensure that the versioned 
data directory mounted at NFS pointing corresponding PV to correct version. 

For rolling back to previous version, all jobs need to be deleted mandatorily since helm/k8s doesnt support rollback of jobs.
```
kubectl delete jobs --all -n <namespace>
```

Search for last successfully deployed helm chart for immediate previous version and find out the revision number with below command
```
helm history <release-name> -n <namespace>
e.g helm history Host-Attestation -n isecl
```

Rollback to previous revision
```
 helm rollback <release-name> <last successfully deployed revision number> -n <namespace>
 e.g helm rollback Host-Attestation 1 -n isecl
 kubectl scale deploy --all --replicas=0 -n isecl # Scale down and scale up services for reflecting the persistent volume claim to get volumes mounted to previous release version.
 kubectl scale deploy --all --replicas=1 -n isecl

```
