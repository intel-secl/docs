# Deploying Individual Services Using Helm

Note:

```
Typically attributes that do not specifically call out for user input will not need to be changed unless there is a specific conflict from the Kubernetes environment.

When deploying individual services, all of the values.yaml files must be configured individually.
```

Set deployment version
```
export VERSION=v5.0.0
```
Following are the steps need to be run for deploying individual charts for host attestation usecase
```shell script
helm pull isecl-helm/cleanup-secrets --version $VERSION && tar -xzf cleanup-secrets-$VERSION.tgz cleanup-secrets/values.yaml 
helm install cleanup-secrets isecl-helm/cleanup-secrets --version $VERSION -f cleanup-secrets/values.yaml -n isecl

helm pull isecl-helm/cms --version $VERSION && tar -xzf cms-$VERSION.tgz cms/values.yaml 
helm install cms isecl-helm/cms --version $VERSION -f cms/values.yaml -n isecl

helm pull isecl-helm/aasdb-cert-generator --version $VERSION && tar -xzf aasdb-cert-generator-$VERSION.tgz aasdb-cert-generator/values.yaml 
helm install aasdb-cert-generator isecl-helm/aasdb-cert-generator --version $VERSION -f aasdb-cert-generator/values.yaml -n isecl

helm pull isecl-helm/aas --version $VERSION && tar -xzf aas-$VERSION.tgz aas/values.yaml 
helm install aas isecl-helm/aas --version $VERSION -f aas/values.yaml -n isecl

helm pull isecl-helm/hvsdb-cert-generator --version $VERSION && tar -xzf hvsdb-cert-generator-$VERSION.tgz hvsdb-cert-generator/values.yaml 
helm install hvsdb-cert-generator isecl-helm/hvsdb-cert-generator --version $VERSION -f hvsdb-cert-generator/values.yaml -n isecl

helm pull isecl-helm/hvs --version $VERSION && tar -xzf hvs-$VERSION.tgz hvs/values.yaml 
helm install hvs isecl-helm/hvs --version $VERSION -f hvs/values.yaml -n isecl

helm pull isecl-helm/trustagent --version $VERSION && tar -xzf trustagent-$VERSION.tgz trustagent/values.yaml 
helm install trustagent isecl-helm/trustagent --version $VERSION -f trustagent/values.yaml -n isecl
```

Following are the steps need to be run for deploying individual charts for workload security usecase
```shell script
helm pull isecl-helm/cleanup-secrets --version $VERSION && tar -xzf cleanup-secrets-$VERSION.tgz cleanup-secrets/values.yaml 
helm install cleanup-secrets isecl-helm/cleanup-secrets --version $VERSION -f cleanup-secrets/values.yaml -n isecl

helm pull isecl-helm/cms --version $VERSION && tar -xzf cms-$VERSION.tgz cms/values.yaml 
helm install cms isecl-helm/cms --version $VERSION -f cms/values.yaml -n isecl

helm pull isecl-helm/aasdb-cert-generator --version $VERSION && tar -xzf aasdb-cert-generator-$VERSION.tgz aasdb-cert-generator/values.yaml 
helm install aasdb-cert-generator isecl-helm/aasdb-cert-generator --version $VERSION -f aasdb-cert-generator/values.yaml -n isecl

helm pull isecl-helm/aas --version $VERSION && tar -xzf aas-$VERSION.tgz aas/values.yaml 
helm install aas isecl-helm/aas --version $VERSION -f aas/values.yaml -n isecl

helm pull isecl-helm/hvsdb-cert-generator --version $VERSION && tar -xzf hvsdb-cert-generator-$VERSION.tgz hvsdb-cert-generator/values.yaml 
helm install hvsdb-cert-generator isecl-helm/hvsdb-cert-generator --version $VERSION -f hvsdb-cert-generator/values.yaml -n isecl

helm pull isecl-helm/hvs --version $VERSION && tar -xzf hvs-$VERSION.tgz hvs/values.yaml 
helm install hvs isecl-helm/hvs --version $VERSION -f hvs/values.yaml -n isecl

helm pull isecl-helm/trustagent --version $VERSION && tar -xzf trustagent-$VERSION.tgz trustagent/values.yaml 
helm install trustagent isecl-helm/trustagent --version $VERSION -f trustagent/values.yaml -n isecl

helm pull isecl-helm/isecl-controller --version $VERSION && tar -xzf isecl-controller-$VERSION.tgz isecl-controller/values.yaml 
helm install isecl-controller isecl-helm/isecl-controller --version $VERSION -f isecl-controller/values.yaml -n isecl

helm pull isecl-helm/ihub --version $VERSION && tar -xzf ihub-$VERSION.tgz ihub/values.yaml 
helm install ihub isecl-helm/ihub --version $VERSION -f ihub/values.yaml -n isecl

helm pull isecl-helm/isecl-scheduler --version $VERSION && tar -xzf isecl-scheduler-$VERSION.tgz isecl-scheduler/values.yaml 
helm install isecl-scheduler isecl-helm/isecl-scheduler --version $VERSION -f isecl-scheduler/values.yaml -n isecl

helm pull isecl-helm/kbs --version $VERSION && tar -xzf kbs-$VERSION.tgz kbs/values.yaml 
helm install kbs isecl-helm/kbs --version $VERSION -f kbs/values.yaml -n isecl

helm pull isecl-helm/wls --version $VERSION && tar -xzf wls-$VERSION.tgz wls/values.yaml 
helm install wls isecl-helm/wls --version $VERSION -f wls/values.yaml -n isecl

helm pull isecl-helm/workload-agent --version $VERSION && tar -xzf workload-agent-$VERSION.tgz workload-agent/values.yaml 
helm install workload-agent isecl-helm/workload-agent --version $VERSION -f workload-agent/values.yaml -n isecl

helm pull isecl-helm/wpm --version $VERSION && tar -xzf wpm-$VERSION.tgz wpm/values.yaml 
helm install wpm isecl-helm/wpm --version $VERSION -f wpm/values.yaml -n isecl
```

The Intel SecL-DC services can individually be deployed using Helm, instead of automatically deploying a specific use case.

Below is a list of the Helm charts available on the Intel SecL Helm repository for deploying individual services (not entire use cases):

|NAME                               |                   CHART VERSION  | APP VERSION  |   DESCRIPTION                                         |
|-----------------------------------|----------------------------------|--------------|-------------------------------------------------------|
|isecl-helm/aas                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Authentica... |
|isecl-helm/admission-controller    |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Admision C... |
|isecl-helm/cms                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Certificat... |
|isecl-helm/hvs                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Host Verif... |
|isecl-helm/ihub                    |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Integratio... |
|isecl-helm/isecl-controller        |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Custom Con... |
|isecl-helm/isecl-scheduler         |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Extended S... |
|isecl-helm/nats                    |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing NATS server            |
|isecl-helm/trustagent              |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Trust Agent   |
|isecl-helm/kbs                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC key broker service |
|isecl-helm/wls                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Workload Service |
|isecl-helm/wla                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Workload Agent |

Below is a list of Helm charts used to run specific jobs needed during deployment:

|NAME                               |                   CHART VERSION  | APP VERSION  |   DESCRIPTION                                         |
|-----------------------------------|----------------------------------|--------------|-------------------------------------------------------|
|isecl-helm/aasdb-cert-generator    |                    5.0.0         |  v5.0.0      |    A Helm chart for creating aasdb certificates       |
|isecl-helm/hvsdb-cert-generator    |                    5.0.0         |  v5.0.0      |    A Helm chart for creating hvsdb certificates       |
|isecl-helm/cleanup-secrets         |                    5.0.0         |  v5.0.0      |    A Helm chart for cleaning up secrets               |

Below is a list of charts that needs to be deployed for Trusted Workload Placement

|NAME                               |                   CHART VERSION  | APP VERSION  |   DESCRIPTION                                         |
|-----------------------------------|----------------------------------|--------------|-------------------------------------------------------|
|isecl-helm/aas                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Authentica... |
|isecl-helm/aasdb-cert-generator    |                    5.0.0         |  v5.0.0      |    A Helm chart for creating aasdb certificates       |
|isecl-helm/cms                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Certificat... |
|isecl-helm/hvsdb-cert-generator    |                    5.0.0         |  v5.0.0      |    A Helm chart for creating hvsdb certificates      |
|isecl-helm/hvs                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Host Verif... |
|isecl-helm/ihub                    |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Integratio... |
|isecl-helm/isecl-controller        |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Custom Con... |
|isecl-helm/isecl-scheduler         |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Extended S... |
|isecl-helm/nats                    |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing NATS server            |
|isecl-helm/trustagent              |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Trust Agent   |


Below are the charts that needs to be deployed for Trusted Cloud Service Provider

|NAME                               |                   CHART VERSION  | APP VERSION  |   DESCRIPTION                                         |
|-----------------------------------|----------------------------------|--------------|-------------------------------------------------------|
|isecl-helm/ihub                    |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Integratio... |
|isecl-helm/isecl-controller        |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Custom Con... |
|isecl-helm/isecl-scheduler         |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Extended S... |
|isecl-helm/trustagent              |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Trust Agent   |
|isecl-helm/admission-controller    |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Admision C... |

Below are the charts that needs to be deployed for TWP Control Plane

|NAME                               |                   CHART VERSION  | APP VERSION  |   DESCRIPTION                                         |
|-----------------------------------|----------------------------------|--------------|-------------------------------------------------------|
|isecl-helm/aas                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Authentica... |
|isecl-helm/aasdb-cert-generator    |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Authentica... |
|isecl-helm/cms                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Certificat... |
|isecl-helm/hvsdb-cert-generator    |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Host Verif... |
|isecl-helm/hvs                     |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Host Verif... |
|isecl-helm/nats                    |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing NATS server            | 

Intel SecL-DC can optionally utilize a NATS server to manage connectivity between the Host Verification Service and any number of deployed Trust Agent hosts.  This acts as an alternative to communication via REST APIs - in NATS mode, a connection is established with the NATS server, and messages are sent and received over that connection.  Using NATS mode requires configuration changes in the values.yaml files for the HVS and Trust Agent charts, as well as deployment of NATS itself:

|NAME                               |                   CHART VERSION  | APP VERSION  |   DESCRIPTION                                         |
|-----------------------------------|----------------------------------|--------------|-------------------------------------------------------|
|isecl-helm/nats                    |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing NATS server            |

NATS deployments also require a setup job:

|NAME                               |                   CHART VERSION  | APP VERSION  |   DESCRIPTION                                         |
|-----------------------------------|----------------------------------|--------------|-------------------------------------------------------|
|isecl-helm/nats-init               |                    5.0.0         |  v5.0.0      |    A Helm chart for creating TLS certificates and ... |

Intel SecL-DC can optionally integrate with Kubernetes to control the placement of workloads based on the attestation status of worker nodes.  Trusted Workload Placement requires the following additional services:

|NAME                               |                   CHART VERSION  | APP VERSION  |   DESCRIPTION                                         |
|-----------------------------------|----------------------------------|--------------|-------------------------------------------------------|
|isecl-helm/admission-controller    |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Admision C... |
|isecl-helm/isecl-controller        |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Custom Con... |
|isecl-helm/isecl-scheduler         |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Extended S... |
|isecl-helm/ihub                    |                    5.0.0         |  v5.0.0      |    A Helm chart for Installing ISecL-DC Integratio... |

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
helm pull isecl-helm/nats && tar -xzf nats-$VERSION.tgz nats/values.yaml 
helm install nats isecl-helm/nats -f nats/values.yaml -n isecl
```



