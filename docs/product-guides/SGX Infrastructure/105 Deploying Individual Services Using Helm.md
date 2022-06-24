# Deploying Individual Services Using Helm

Note:

```
Typically attributes that do not specifically call out for user input will not need to be changed unless there is a specific conflict from the Kubernetes environment.

When deploying individual services, all of the values.yaml files must be configured individually.
```

Set deployment version
```
export VERSION=5.0.0
```
Following are the steps need to be run for deploying individual charts for tee attestation usecase
```shell script
helm pull isecl-helm/cleanup-secrets --version $VERSION && tar -xzf cleanup-secrets-$VERSION.tgz cleanup-secrets/values.yaml 
helm install cleanup-secrets isecl-helm/cleanup-secrets --version $VERSION -f cleanup-secrets/values.yaml -n isecl

helm pull isecl-helm/cms --version $VERSION && tar -xzf cms-$VERSION.tgz cms/values.yaml 
helm install cms isecl-helm/cms --version $VERSION -f cms/values.yaml -n isecl

helm pull isecl-helm/aasdb-cert-generator --version $VERSION && tar -xzf aasdb-cert-generator-$VERSION.tgz aasdb-cert-generator/values.yaml 
helm install aasdb-cert-generator isecl-helm/aasdb-cert-generator --version $VERSION -f aasdb-cert-generator/values.yaml -n isecl

helm pull isecl-helm/aas --version $VERSION && tar -xzf aas-$VERSION.tgz aas/values.yaml 
helm install aas isecl-helm/aas --version $VERSION -f aas/values.yaml -n isecl

helm pull isecl-helm/apsdb-cert-generator --version $VERSION && tar -xzf apsdb-cert-generator-$VERSION.tgz apsdb-cert-generator/values.yaml 
helm install apsdb-cert-generator isecl-helm/apsdb-cert-generator --version $VERSION -f apsdb-cert-generator/values.yaml -n isecl

helm pull isecl-helm/aps --version $VERSION && tar -xzf aps-$VERSION.tgz fds/values.yaml 
helm install aps isecl-helm/aps  --version $VERSION -f aps/values.yaml -n isecl

helm pull isecl-helm/tcs --version $VERSION && tar -xzf tcs-$VERSION.tgz tcs/values.yaml 
helm install tcs isecl-helm/tcs --version $VERSION -f tcs/values.yaml -n isecl

helm pull isecl-helm/fdsdb-cert-generator --version $VERSION && tar -xzf fdsdb-cert-generator-$VERSION.tgz fdsdb-cert-generator/values.yaml 
helm install fdsdb-cert-generator isecl-helm/fdsdb-cert-generator --version $VERSION -f fdsdb-cert-generator/values.yaml -n isecl

helm pull isecl-helm/fds --version $VERSION && tar -xzf fds-$VERSION.tgz fds/values.yaml 
helm install fds isecl-helm/fds --version $VERSION -f fds/values.yaml -n isecl

helm pull isecl-helm/fda --version $VERSION && tar -xzf fda-$VERSION.tgz fda/values.yaml 
helm install fda isecl-helm/fda --version $VERSION -f fda/values.yaml -n isecl

helm pull isecl-helm/qvs --version $VERSION && tar -xzf qvs-$VERSION.tgz qvs/values.yaml 
helm install qvs isecl-helm/qvs --version $VERSION -f qvs/values.yaml -n isecl

helm pull isecl-helm/kbs --version $VERSION && tar -xzf kbs-$VERSION.tgz kbs/values.yaml 
helm install kbs isecl-helm/kbs --version $VERSION -f kbs/values.yaml -n isecl
```


Below is a list of the Helm charts available on the SKC Helm repository for deploying individual services (not entire use cases):

|NAME                               |                   CHART VERSION  | APP VERSION  |   DESCRIPTION                                         |
|-----------------------------------|----------------------------------|--------------|-------------------------------------------------------|
|isecl-helm/aas                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Authentica... |
|isecl-helm/admission-controller    |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Admision C... |
|isecl-helm/cms                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Certificat... |
|isecl-helm/fds                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Feature Discover Service |
|isecl-helm/ihub                    |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Integratio... |
|isecl-helm/isecl-controller        |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Custom Con... |
|isecl-helm/isecl-scheduler         |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Extended S... |
|isecl-helm/nats                    |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing NATS server            |
|isecl-helm/fda                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Feature Discovery Agent   |
|isecl-helm/kbs                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC key broker service |
|isecl-helm/qvs                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Quote Verification Service |
|isecl-helm/tcs                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC TEE Caching Service |

Below is a list of Helm charts used to run specific jobs needed during deployment:

|NAME                               |                   CHART VERSION  | APP VERSION  |   DESCRIPTION                                         |
|-----------------------------------|----------------------------------|--------------|-------------------------------------------------------|
|isecl-helm/aasdb-cert-generator    |                    5.0.0         |  v5.0.0      |    A Helm chart for creating aasdb certificates       |
|isecl-helm/fdsdb-cert-generator    |                    5.0.0         |  v5.0.0      |    A Helm chart for creating fdsdb certificates       |
|isecl-helm/apsdb-cert-generator    |                    5.0.0         |  v5.0.0      |    A Helm chart for creating apsdb certificates       |
|isecl-helm/cleanup-secrets         |                    5.0.0         |  v5.0.0      |    A Helm chart for cleaning up secrets               |
|isecl-helm/aas-manager             |                    5.0.0         |  v5.0.0      |    A Helm chart for bootstrapping default users an... |


Below is a list of charts that needs to be deployed for TEE Attestation and Orchestration

|NAME                               |                   CHART VERSION  | APP VERSION  |   DESCRIPTION                                         |
|-----------------------------------|----------------------------------|--------------|-------------------------------------------------------|
|isecl-helm/aas                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Authentica... |
|isecl-helm/cms                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Certificat... |
|isecl-helm/fds                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Feature Discover Service |
|isecl-helm/ihub                    |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Integratio... |
|isecl-helm/isecl-controller        |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Custom Con... |
|isecl-helm/isecl-scheduler         |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Extended S... |
|isecl-helm/nats                    |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing NATS server            |
|isecl-helm/fda                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Feature Discovery Agent   |
|isecl-helm/kbs                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC key broker service |
|isecl-helm/qvs                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Quote Verification Service |
|isecl-helm/tcs                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC TEE Caching Service |


Below are the charts that needs to be deployed for TEE Orchestration

|NAME                               |                   CHART VERSION  | APP VERSION  |   DESCRIPTION                                         |
|-----------------------------------|----------------------------------|--------------|-------------------------------------------------------|
|isecl-helm/aas                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Authentica... |
|isecl-helm/cms                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Certificat... |
|isecl-helm/fds                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Feature Discover Service |
|isecl-helm/ihub                    |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Integratio... |
|isecl-helm/isecl-controller        |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Custom Con... |
|isecl-helm/isecl-scheduler         |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Extended S... |
|isecl-helm/nats                    |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing NATS server            |
|isecl-helm/fda                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Feature Discovery Agent   |
|isecl-helm/qvs                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Quote Verification Service |
|isecl-helm/tcs                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC TEE Caching Service |

Intel SecL-DC can optionally utilize a NATS server to manage connectivity between the Host Verification Service and any number of deployed Trust Agent hosts.  This acts as an alternative to communication via REST APIs - in NATS mode, a connection is established with the NATS server, and messages are sent and received over that connection.  Using NATS mode requires configuration changes in the values.yaml files for the HVS and Trust Agent charts, as well as deployment of NATS itself:

|NAME                               |                   CHART VERSION  | APP VERSION  |   DESCRIPTION                                         |
|-----------------------------------|----------------------------------|--------------|-------------------------------------------------------|
|isecl-helm/nats                    |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing NATS server            |

NATS deployments also require a setup job:

|NAME                               |                   CHART VERSION  | APP VERSION  |   DESCRIPTION                                         |
|-----------------------------------|----------------------------------|--------------|-------------------------------------------------------|
|isecl-helm/nats-init               |                    5.0.0         |  v5.0.0      |    A Helm chart for creating TLS certificates and ... |


## Configuration Changes Needed for NATS Mode

If NATS mode will be used, the NATS service answer file must also be downloaded and configured, and additional configuration elements must be added to the HVS and Trust Agent values.yaml files.

Download and configure the NATS configuration file:

```
tar -xzf nats-$VERSION.tgz nats/values.yaml
```

In the hvs.yaml and trustagent.yaml files, configure the NATS block in the existing config element:

```    
    nats:
      enabled: true # Enable/Disable NATS mode<br> (Allowed values: `true`\`false`)
      servers: nats://<user input>:30222   # NATS Server IP/Hostname<br> (**REQUIRED IF ENABLED**)
      serviceMode: outbound # The model for TA (Allowed values: `outbound`) (**REQUIRED IF ENABLED**)
```

Note that the NATS server IP/hostname is typically the same as the Kubernetes control plane.

The following example shows an installation for of NATS

## NATS Deployment Instruction

```
helm pull isecl-helm/nats --version $VERSION && tar -xzf nats-$VERSION.tgz nats/values.yaml 
helm install nats isecl-helm/nats --version $VERSION -f nats/values.yaml -n isecl
```



