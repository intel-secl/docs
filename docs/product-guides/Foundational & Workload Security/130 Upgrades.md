# Upgrades

???+ note 
    Before performing any upgrade, Intel strongly recommends backing up the database for the HVS, WLS, and AAS.  See Postgres documentation for detailed options for backing up databases.  Below is a sample method for backing up an entire database server:

```
Backup to tar file:
pg_dump --dbname <database_name> --username=<database username> -F -t > <database_backup_file>.tar
Restore from tar file:
pg_restore --dbname=<database_name> --username=<database username><database_backup_file>.tar
```

Some upgrades may involve changes to database content, and a backup will ensure that data is not lost in the case of an error during the upgrade process.

## Backward Compatibility

In general Intel SecL services are made to be backward-compatible within a given major release (for example, the 3.6 HVS should be compatible with the 3.5 Trust Agent) in an upgrade priority order (see below).  Major version upgrades may require coordinated upgrades across all services.

## Upgrade Order

Upgrades should be performed in the following order to prevent misconfiguration or any service unavailability:

1) CMS, AAS

2)  HVS

3) WLS, IHUB

4) KBS, Trust Agents, Workload Agents, WPM

Upgrading in this order will make each service unavailable only for the duration of the upgrade for that service.  

## Upgrade Process

### Binary Installations

For services installed directly (not deployed as containers), the upgrade process simply requires executing the new-version installer on the same machine where the old-version is running.  The installer will re-use the same configuration elements detected in the existing version's config file.  No additional answer file is required unless configuration settings will change.


### Container Deployments

Container upgrades will be based on recreate strategy. All services except(KBS and HVS) can be upgraded just by updating
the image name and tag to newer version in respective deployment.yml files.

Config and database schema changes and data migration will be done through K8s jobs, during the upgrade job, the service
should be down, otherwise may lead to inconsistent data or data corruption. For 4.0 there is change in database schema
for HVS and k8s job manifest will be provided along with other manifests for deployment.

Agent upgrade jobs, by default copy backup of configuration directories on all hosts at location /tmp/<agent_name>_backup,
this location can be updated in respective upgrade jobs. There is also a rollback job for each agent, which restores the
backed up configuration files back to original location.

##### Build

Intel assumes all services for any use-case are up and running before proceeding with the upgrade.

Push all the newer version of container images to registry. All oci images will be in k8s/container-images.

e.g

```shell
skopeo copy oci-archive:<oci-image-tar-name> docker://<registry-ip/hostname>:<registry-port>/<image-name>:<image-tag> --dest-tls-verify=false
```

------

##### Upgrade/Deploy

##### Backup Services

1. Take back up of data in NFS mount for all services from NFS server system \<NFS-Mount-Path>/isecl

   ```shell
   #Example
   cp -r <nfs-location>/isecl <nfs-location>/isecl-<version>-backup
   ```

##### Rollback Services

1. If upgrade is not successful, then update `deployment.yml` files with older images(v3.6) for 4.0 upgrade path and restore the backed up nfs data at same mount path
2. Bring up individual components with `./isecl-bootstrap.sh up <service>`

???+ note 
    If in case service fails to start or gets crashed, then copy all the backed up data as per folder structure like how was it earlier. Bring up db instance pointing to backed up data and bring up component deployment by pointing to older version of container image.

##### HVS Upgrade

1. Update the image name with new image name/image tag in existing `deployment.yml`

2. Copy `upgrade-job.yml` from builds `k8s/manifests/hvs/upgrade-job.yml` into control plane node `k8s/manifests/hvs/`

```
scp <build-vm>:<build-dir>/k8s/manifests/hvs/upgrade-job.yml <control-plane-vm>:<existing dir>/k8s/manifests/hvs/upgrade-job.yml
```

3. Update `<image-name>` and `<image-tag>` and existing version of deployed hvs in  `<current deployed version>` in `k8s/manifests/hvs/upgrade-job.yml`

```yaml
containers:
  - name: hvs
    image: '<image-name>:<image-tag>'
    imagePullPolicy: Always
    securityContext:
      runAsUser: 1001
      runAsGroup: 1001
    env:
      - name: CONFIG_DIR
        value: /etc/hvs
      - name: COMPONENT_VERSION
        value: <current deployed version>      
```

4. Run the upgrade job,

```shell
cd k8s/manifests
kubectl delete deployment hvs-deployment -n isecl
kubectl apply -f hvs/upgrade-job.yml
```

5. Check the status of hvs-upgrade job for completion.

```shell
kubectl get jobs -n isecl
```

and check for successful database and config upgrades in hvs-upgrade job pod logs

```shell
kubectl logs -n isecl hvs-upgrade-<podname>
```

Delete hvs-upgrade once upgrade is running.
```shell
kubectl delete jobs hvs-upgrade -n isecl
```

6. Redeploy the latest hvs

```shell
kubectl apply -f hvs/deployment.yml
```

???+ note 
    If hvs-upgrade job is failed or upgraded service deployment went into `CrashLoopBackOff`, restore the service with instruction given in Rollback Services section

**KBS Upgrade:**

1. Update the image name with new image name/image tag in existing `deployment.yml`

2. Copy `upgrade-job.yml` from builds `k8s/manifests/kbs/upgrade-job.yml` into control plane node k8s/manifests/kbs/

```
scp <build-vm>:<build-dir>/k8s/manifests/kbs/upgrade-job.yml <control-plane-vm>:<existing dir>/k8s/manifests/kbs/upgrade-job.yml
```

3. Add the following variables in `kbs/configMap.yml`

```yaml
KMIP_HOSTNAME: <kmip hostname>
```

Run the command

```shell
kubectl apply -f kbs/configMap.yml --namespace=isecl
```

4. (Optional) Add the following variables in `kbs/secrets.yml` if required
   `KMIP_USERNAME` and `KMIP_PASSWORD`. Run the following commands

```shell
kubectl delete secret -n isecl kbs-secret
kubectl create secret generic kbs-secret --from-file=kbs/secrets.txt --namespace=isecl
```

5. Update `<image-name>` and `<image-tag>` and existing version of deployed kbs in  `<current deployed version>` in `k8s/manifests/kbs/upgrade-job.yml`

```yaml
containers:
  - name: kbs
    image: '<image-name>:<image-tag>'
    imagePullPolicy: Always
    securityContext:
      runAsUser: 1001
      runAsGroup: 1001
    env:
      - name: CONFIG_DIR
        value: /etc/kbs
      - name: COMPONENT_VERSION
        value: <current deployed version>      
```

6. Run the upgrade job,

```shell
cd k8s/manifests
kubectl delete deployment -n isecl kbs-deployment
kubectl apply -f kbs/upgrade-job.yml
```

7. Check the status of kbs-upgrade job for completion.

```shell
kubectl get jobs -n isecl
kubectl logs -n isecl kbs-upgrade-<pod id>
```

Delete kbs-upgrade once upgrade is running.
```shell
kubectl delete jobs kbs-upgrade -n isecl
```

8. Update the image name in `kbs/deployment.yml` to newer version and deploy the latest kbs

```shell
kubectl apply -f kbs/deployment.yml
or
cd kbs && kubectl kustomize . | kubectl apply -f -
```

**Individual services upgrade**

1. Update the image name in existing `deployment.yml` of respective service.

2. Redeploy by running the below command, By doing `kubectl apply -f` on `deployment.yml` with change in image name will terminate service with older image version and bring up service with new image version

```shell
cd /k8s/manifests/<service-manifests-folder> &&  kubectl kustomize . | kubectl apply -f -
e.g cd /k8s/manifests/cms && kubectl kustomize . | kubectl apply -f -
```

**TA\WLA Upgrade:**

Copy following manifests and scripts from build vm to k8s control plane vm

```shell
scp <build-vm>:<build-dir>/k8s/manifests/<component>/<component>-upgrade.yml <control-plane-vm>:<existing dir>/k8s/manifests/<component>/
scp <build-vm>:<build-dir>/k8s/manifests/<component>/rollback.yml <control-plane-vm>:<existing dir>/k8s/manifests/<component>/
```

Addional step for TA:
```shell
scp <build-vm>:<build-dir>/k8s/manifests/ta/daemonset.yml <control-plane-vm>:<existing dir>/k8s/manifests/ta/daemonset.yml
scp <build-vm>:<build-dir>/k8s/manifests/ta/daemonset-suefi.yml <control-plane-vm>:<existing dir>/k8s/manifests/ta/daemonset-suefi.yml
```


Log in to the worker nodes with TXT+TPM2.0 enabled and perform the following steps:

1. Update tboot to 1.10.1

Log in to control plane node and perform following steps:

* Update the new image name and tag in `daemonset.yml` and `daemonset-suefi.yml` files

```yaml
containers:
  - image: <image-name>:<image-tag>
```

* Update `<image-name>` and `<image-tag>` and existing version of deployed component in `<component-exising-installed-version>` in `k8s/manifests/<component>/<component>-upgrade.yml`

```yaml
containers:
  - name: tagent
    image: '<image-name>:<image-tag>'
    imagePullPolicy: Always
    command:
      - /container_upgrade.sh
    securityContext:
      privileged: true
    env:
      - name: CONFIG_DIR
        value: <component-config-dir>
      - name: COMPONENT_VERSION
        value: <component-exising-installed-version>
```

* Upgrade job for TA\WLA will re provision all tasks. This might require new `BEARER_TOKEN` in existing ta-secret or wla-secret
   (Optional). Update new value for TPM_OWNER_SECRET in `<component>/secret.txt`. Run the following commands
```shell
 kubectl delete secret -n isecl <component>-secret
 #Post updation of BEARER_TOKEN & TPM_OWNER_SECRET in <component>/secrets.txt
 kubectl create secret generic <component>-secret --from-file=secrets.txt --namespace=isecl
```

*  Perform upgrade and redeploy trustagent or workload agent daemonsets
```shell
 cd k8s/manifests/
 kubectl apply -f <component>/<component>-upgrade.yml
```
* Wait for all <component>-upgrade daemonset come up in running state

* Delete <component>-upgrade daemonset once all <component>-upgrade daemonsets are running.

```shell
kubectl delete daemonset -n isecl <component>-upgrade
```

* Bring up newer version of <component>-daemonsets

```shell
kubectl apply -f <component>/daemonset.yml
```

Additional step for TA:
```shell
kubectl apply -f ta/daemonset-suefi.yml
```

##### Backup and Rollback in TA and WLA

The `<component>-upgrade.yml` is run once daemonset, while performing upgrade job, ta will copy the `/opt/trustagent` into backup directory `/tmp/trustagent_backup` on every host
and wla upgrade will copy the '/etc/workload-agent' into backup directory `/tmp/wlagent_backup`
Following steps will help us to achieve rollback

1. Restore the backed up directories from `/tmp/trustagent_backup` into `/opt/trustagent` in TA and in WLA from `/tmp/wlagent_backup` into `/etc/workload-agent`
```shell
cd k8s/manifests/
#For TA
kubectl apply -f <componenet>/rollback.yml
#For WLA
kubectl apply -f <componenet>/<component>-rollback.yml
```
2. Update the image name to previous version in `<component>/daemonset.yml` and `<component>/daemonset-suefi.yml`(if upgrade path is 4.1 -> 5.0, then update image tag to 4.1)
```shell
kubectl apply -f <component>/daemonset.yml
```

Additional step for TA:
```shell
kubectl apply -f ta/daemonset-suefi.yml
```
