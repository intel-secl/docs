# Container Upgrades

Container upgrades will be supported only on multi node deployments and will be based on recreate strategy from v3.6 to v4.0. All services except KBS can be upgraded just by updating the image name and tag to newer version in respective deployment.yml files.

## Backup and roll back applicable to all services

1.	Take back up of data in NFS mount for all services from NFS server system <NFS-Mount-Path>/isecl
2.	If upgrade is not successful, then update deployment.yml files with older images(v3.6) for 4.0 upgrade path and restore the backed up NFS data at same mount path.
3.	Bring up individual components with ./isecl-bootstrap.sh up <service>.

???+ note 
    If in case service fails to start or gets crashed, then copy all the backed up data as per folder structure like how was it earlier. Bring up DB instance pointing to backed up data and bring up component deployment by pointing to older version of container image.

## Backward Compatibility

In general Intel SecL services are made to be backward-compatible within a given major release (for example, the  SHVS should be compatible with the 3.5 SGX Agent) in an upgrade priority order (see below). Major version upgrades may require coordinated upgrades across all services.

## Upgrade Order

Upgrades should be performed in the following order to prevent misconfiguration or any service unavailability:

1.	CMS, AAS
2.	SCS, SHVS
3.	SQVS, KBS, SGX Agent
4.	ISECL-CONTROLLER, IHUB, ISECL-SCHEDULER

Upgrading in this order will make each service unavailable only for the duration of the upgrade for that service.

## Upgrade Process

### Container Installations

Assuming all services for any use-case are up and running.

Push all the newer version of container images to registry. All oci images will be in k8s/container-images.

e.g
```
skopeo copy oci-archive:<oci-image-tar-name> docker://<registry-ip/hostname>:<registry-port>/<image-name>:<image-tag> --dest-tls-verify=false
```

#### Individual services upgrade except KBS

1.	Update the image name in existing deployment.yml of respective service.
2.	Redeploy by running the below command, By doing "kubectl apply -f" on deployment.yml with change in image name will terminate service with older image version and bring up service with new image version.

```
cd /k8s/manifests/<service-manifests-folder> &&  kubectl kustomize . | kubectl apply -f -
e.g cd /k8s/manifests/cms && kubectl kustomize . | kubectl apply -f -
```

#### KBS Upgrade

1.	Update the image name with new image name/image tag in existing deployment.yml.
2.	Copy upgrade-job.yml from builds k8s/manifests/kbs/upgrade-job.yml into control plane node k8s/manifests/kbs/
```
scp <build-vm>:<build-dir>/k8s/manifests/kbs/upgrade-job.yml <control-plane-vm>:<existing dir>/k8s/manifests/kbs/upgrade-job.yml
```
3.	Add the following variables in kbs/configMap.yml
```
KMIP_HOSTNAME: <kmip fqdn or ip>
```
Run the command
```
kubectl apply -f kbs/configMap.yml --namespace=isecl
```
4.	(Optional) Add the following variables in kbs/secrets.yml if required KMIP_USERNAME and KMIP_PASSWORD Run the following commands
```
kubectl delete secret -n isecl kbs-secret
kubectl create secret generic kbs-secret --from-file=kbs/secrets.txt --namespace=isecl
```
5.	Update "image-name" and "image-tag" and existing version of deployed kbs in "current deployed version" in k8s/manifests/kbs/upgrade-job.yml
```
      containers:
        - name: kbs
          image: <image-name>:<image-tag>
          imagePullPolicy: Always
          securityContext:
            runAsUser: 1001
            runAsGroup: 1001
          env:
            - name: CONFIG_DIR
              value: "/etc/kbs"
            - name: COMPONENT_VERSION
              value: <current deployed version>
```
6.	Run the upgrade job,
```
cd k8s/manifests
kubectl delete deployment -n isecl kbs-deployment
kubectl apply -f kbs/upgrade-job.yml
```
7.	Check the status of kbs-upgrade job for completion.
```
kubectl get jobs -n isecl
kubectl logs -n isecl kbs-upgrade-<pod id>
```
8.	Update the image name in kbs/deployment.yml to newer version and deploy the latest kbs
```
kubectl apply -f kbs/deployment.yml or cd kbs && kubectl kustomize . | kubectl apply -f -
```
