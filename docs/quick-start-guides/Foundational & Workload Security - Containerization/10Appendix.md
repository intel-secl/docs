# Appendix

## Hardware feature detection
For checking whether a system is SUEFI enabled
```bootctl | grep "Secure Boot: enabled"```

For checking system installed with tboot
```txt-stat | grep "TXT measured launch: TRUE" ```

## Running behind Proxy

```shell
#Set proxy in ~/.bash_profile
export http_proxy=<proxy-url>
export https_proxy=<proxy-url>
export no_proxy=<ip_address/hostname>
```

## Git Config Sample (~/.gitconfig)

```ini
[user]
        name = <username>
        email = <email-id>
[color]
        ui = auto
 [push]
        default = matching 
```

## Rebuilding Repos

In order to rebuild repos, ensure the following steps are followed as a pre-requisite

```shell
# Clean all go-mod packages
rm -rf ~/go/pkg/mod/*

#Navigate to specific folder where repos are built, example below
cd /root/isecl/build/fs
rm -rf .repo *
#Rebuild as before
repo init ...
repo sync
make ...
```

## Setup Task Flow

Setup tasks flows have been updated to have K8s native flow to be more agnostic to K8s workflows. Following would be the flow for setup task

- User would create a new `configMap`  object with the environment variables specific to the setup task. The Setup task variables would be documented in the Product Guide
- Users can provide variables for more than one setup task
- Users need to add `SETUP_TASK: "<setup task name>/<comma separated setup task name>"` in the same `configMap`
- Provide a unique name to the new `configMap`
- Provide the same name in the `deployment.yaml` under `configMapRef` section
- Deploy the specific service again with  ` kubectl kustomize . | kubectl  apply -f -`

## Configuration Update Flow

Configuration Update flows have been updated to have K8s native flow to be more agnostic to K8s workflows using `configMap` only. Following would be the flow for configuration update

- User would create a new `configMap`  object using existing one and update the new values. The list of config variables would be documented in the Product Guide
- Users can update variables for more than one
- Users need to add `SETUP_TASK: "update-service-config"` in the same `configMap`
- Provide a unique name to the new `configMap`
- Provide the same name in the `deployment.yaml` under `configMapRef` section
- Deploy the specific service again with  ` kubectl  kustomize . | kubectl  apply -f -`

???+ note 
    Incase of agents, setup tasks or configuration updates done through above flows will be applied for all the agents running on different BMs. In order to run setup task or update configuration for individual agents, then user need to perform `kubectl exec -it <pod_name> /bin/bash` into a particular agent pod and run the specific setup task.

## Cleanup workflows

### Single-node

In order to cleanup and setup fresh again on single node without data, config from previous deployment

```shell
#Purge all data and pods,deploy,cm,secrets
./isecl-bootstrap.sh purge <all/usecase of choice>

#Purge all db data,pods,deploy,cm,secrets
./isecl-bootstrap-db-services.sh purge

#Comment/Remove the following lines from /var/snap/microk8s/current/args/kube-scheduler
--policy-config-file=/opt/isecl-k8s-extensions/scheduler-policy.json

#Restart kubelet
systemctl restart snap.microk8s.daemon-kubelet.service

#Setup fresh
./isecl-bootstrap-db-services.sh up
./isecl-bootstrap.sh up <all/usecase of choice>

#Reconfigure K8s-scheduler
vi /var/snap/microk8s/current/args/kube-scheduler

#Add the below line
--policy-config-file=/opt/isecl-k8s-extensions/scheduler-policy.json

#Restart kubelet
systemctl restart snap.microk8s.daemon-kubelet.service
```

In order to cleanup and setup fresh again on single node with data, config from previous deployment

```shell
#Down all data and pods,deploy,cm,secrets with deleting config,data,logs
./isecl-bootstrap.sh down <all/usecase of choice>

#Comment/Remove the following lines from /var/snap/microk8s/current/args/kube-scheduler
--policy-config-file=/opt/isecl-k8s-extensions/scheduler-policy.json

#Restart kubelet
systemctl restart snap.microk8s.daemon-kubelet.service

#Setup fresh
./isecl-bootstrap-db-services.sh up
./isecl-bootstrap.sh up <all/usecase of choice>

#Reconfigure K8s-scheduler
vi /var/snap/microk8s/current/args/kube-scheduler

#Add the below line
--policy-config-file=/opt/isecl-k8s-extensions/scheduler-policy.json

#Restart kubelet
systemctl restart snap.microk8s.daemon-kubelet.service
```

In order to clean single service and bring up again on single node without data, config from previous deployment
```shell
./isecl-bootstrap.sh down <service-name>
rm -rf /etc/<service-name>
rm -rf /var/log/<service-name>

#Only in case of KBS, perform one more step along with above 2 steps
rm -rf /opt/kbs
./isecl-bootstrap.sh up <service-name>
```

In order to redeploy single service again on single node with data, config from previous deployment
```shell
./isecl-bootstrap.sh down <service-name>
./isecl-bootstrap.sh up <service-name>
```

### Multi-node

In order to cleanup and setup fresh again on multi-node without data,config from previous deployment

```shell
#Purge all data and pods,deploy,cm,secrets
./isecl-bootstrap.sh down <all/usecase of choice>

#Delete 'scheduler-policy.json'
rm -rf /opt/isecl-k8s-extensions

#Comment/Remove '--policy-config-file=...' from /etc/kubernetes/manifests/kube-scheduler.yaml as below
 containers:
  - command:
    - kube-scheduler
    - --policy-config-file=/opt/isecl-k8s-extensions/scheduler-policy.json

#Comment/Remove '- mountPath: ...' from /etc/kubernetes/manifests/kube-scheduler.yaml as below
containers:
    volumeMounts:
    - mountPath: /opt/isecl-k8s-extensions/
      name: extendedsched
      readOnly: true
      
#Comment/Remove '-hostPath:...' from /etc/kubernetes/manifests/kube-scheduler.yaml as below
volumes:
  - hostPath:
      path: /opt/isecl-k8s-extensions/
      type:
    name: extendedsched

#Restart kubelet
systemctl restart kubelet

#Purge all db data,pods,deploy,cm,secrets
./isecl-bootstrap-db-services.sh purge

#Cleanup all data from NFS share --> User controlled
rm -rf /<nfs-mount-path>/isecl/<service-name>/config
rm -rf /<nfs-mount-path>/isecl/<service-name>/logs

Repeat the above steps for all service

#Only in case of KBS, perform one more step along with above 2 steps
rm -rf /<nfs-mount-path>/isecl/<service-name>/opt

#Cleanup data from each worker node --> User controlled
rm -rf /etc/workload-agent
rm -rf /var/log/workload-agent
rm -rf /opt/trustagent
rm -rf /var/log/trustagent

#Setup fresh
./isecl-bootstrap-db-services.sh up
./isecl-bootstrap.sh up <all/usecase of choice>

#Reconfigure K8s-scheduler
#Uncomment '--policy-config-file=...' from /etc/kubernetes/manifests/kube-scheduler.yaml as below
containers:
  - command:
    - kube-scheduler
    - --policy-config-file=/opt/isecl-k8s-extensions/scheduler-policy.json

#Uncomment '- mountPath: ...' from /etc/kubernetes/manifests/kube-scheduler.yaml as below
containers:
    volumeMounts:
    - mountPath: /opt/isecl-k8s-extensions/
      name: extendedsched
      readOnly: true

#Uncomment '-hostPath:...' from /etc/kubernetes/manifests/kube-scheduler.yaml as below
volumes:
  - hostPath:
      path: /opt/isecl-k8s-extensions/
      type:
    name: extendedsched

#Copy ihub_public_key.pem from <NFS-vol-base-path>/ihub/config/ to K8s control-plane and update IHUB_PUB_KEY_PATH in isecl-k8s.env

#Bootstrap isecl-scheduler
./isecl-bootstrap.sh up isecl-scheduler

#Restart kubelet
systemctl restart kubelet
```

In order to cleanup and setup fresh again on multi-node without removing data,config from previous deployment

```shell
#Down all pods,deploy,cm,secrets with removing persistent data
./isecl-bootstrap.sh down <all/usecase of choice>

#Delete 'scheduler-policy.json'
rm -rf /opt/isecl-k8s-extensions

#Comment/Remove '--policy-config-file=...' from /etc/kubernetes/manifests/kube-scheduler.yaml as below
 containers:
  - command:
    - kube-scheduler
    - --policy-config-file=/opt/isecl-k8s-extensions/scheduler-policy.json

#Comment/Remove '- mountPath: ...' from /etc/kubernetes/manifests/kube-scheduler.yaml as below
containers:
    volumeMounts:
    - mountPath: /opt/isecl-k8s-extensions/
      name: extendedsched
      readOnly: true
      
#Comment/Remove '-hostPath:...' from /etc/kubernetes/manifests/kube-scheduler.yaml as below
volumes:
  - hostPath:
      path: /opt/isecl-k8s-extensions/
      type:
    name: extendedsched
    
#Restart kubelet
systemctl restart kubelet

#Setup fresh
./isecl-bootstrap-db-services.sh up
./isecl-bootstrap.sh up <all/usecase of choice>

#Copy ihub_public_key.pem from <NFS-vol-base-path>/ihub/config/ to K8s control-plane and update IHUB_PUB_KEY_PATH in isecl-k8s.env

#Bootstrap isecl-scheduler
./isecl-bootstrap.sh up isecl-scheduler

#Reconfigure K8s-scheduler
#Uncomment '--policy-config-file=...' from /etc/kubernetes/manifests/kube-scheduler.yaml as below
containers:
  - command:
    - kube-scheduler
    - --policy-config-file=/opt/isecl-k8s-extensions/scheduler-policy.json

#Uncomment '- mountPath: ...' from /etc/kubernetes/manifests/kube-scheduler.yaml as below
containers:
    volumeMounts:
    - mountPath: /opt/isecl-k8s-extensions/
      name: extendedsched
      readOnly: true

#Uncomment '-hostPath:...' from /etc/kubernetes/manifests/kube-scheduler.yaml as below
volumes:
  - hostPath:
      path: /opt/isecl-k8s-extensions/
      type:
    name: extendedsched
    
#Restart kubelet
systemctl restart kubelet
```

In order to clean single service and bring up again on multi node without data, config from previous deployment
```./isecl-bootstrap.sh down <service-name>```
log into nfs system 

```shell
rm -rf /<nfs-mount-path>/isecl/<service-name>/config
rm -rf /<nfs-mount-path>/isecl/<service-name>/logs

#Only in case of KBS, perform one more step along with above 2 steps
rm -rf /<nfs-mount-path>/isecl/<service-name>/opt

#Cleanup data from each worker node incase of workload agent
rm -rf /etc/workload-agent
rm -rf /var/log/workload-agent

#Cleanup data from each worker node incase of trustagent
rm -rf /opt/trustagent
rm -rf /var/log/trustagent
```
log into K8s control-plane
```./isecl-bootstrap.sh up <service-name>```

In order to redeploy single service again on multi node with data, config from previous deployment
```shell
./isecl-bootstrap.sh down <service-name>
./isecl-bootstrap.sh up <service-name>
```