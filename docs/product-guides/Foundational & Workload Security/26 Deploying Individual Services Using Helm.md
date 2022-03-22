# Deploying Individual Services Using Helm

The Intel SecL-DC services can individually be deployed using Helm, instead of automatically deploying a specific use case.  

Below is a list of the Helm charts available on the Intel SecL Helm repository for deploying individual services (not entire use cases):

|NAME                               |                   CHART VERSION  | APP VERSION  |   DESCRIPTION                                         |
|-----------------------------------|----------------------------------|--------------|-------------------------------------------------------|
|isecl-helm/aas                     |                    4.2.0         |  v4.2.0      |    A Helm chart for Installing ISecL-DC Authentica... |
|isecl-helm/admission-controller    |                    4.2.0         |  v4.2.0      |    A Helm chart for Installing ISecL-DC Admision C... |
|isecl-helm/cms                     |                    4.2.0         |  v4.2.0      |    A Helm chart for Installing ISecL-DC Certificat... |
|isecl-helm/hvs                     |                    4.2.0         |  v4.2.0      |    A Helm chart for Installing ISecL-DC Host Verif... |
|isecl-helm/ihub                    |                    4.2.0         |  v4.2.0      |    A Helm chart for Installing ISecL-DC Integratio... |
|isecl-helm/isecl-controller        |                    4.2.0         |  v4.2.0      |    A Helm chart for Installing ISecL-DC Custom Con... |
|isecl-helm/isecl-scheduler         |                    4.2.0         |  v4.2.0      |    A Helm chart for Installing ISecL-DC Extended S... |
|isecl-helm/nats                    |                    4.2.0         |  v4.2.0      |    A Helm chart for Installing NATS server            |
|isecl-helm/trustagent              |                    4.2.0         |  v4.2.0      |    A Helm chart for Installing ISecL-DC Trust Agent   |

Below is a list of Helm charts used to run specific jobs needed during deployment:

|NAME                               |                   CHART VERSION  | APP VERSION  |   DESCRIPTION                                         |
|-----------------------------------|----------------------------------|--------------|-------------------------------------------------------|
|isecl-helm/aasdb-cert-generator    |                    4.2.0         |  v4.2.0      |    A Helm chart for creating aasdb certificates       |
|isecl-helm/hvsdb-cert-generator    |                    4.2.0         |  v4.2.0      |    A Helm chart for creating hvsdb certificates       |
|isecl-helm/cleanup-secrets         |                    4.2.0         |  v4.2.0      |    A Helm chart for cleaning up secrets               |
|isecl-helm/aas-manager             |                    4.2.0         |  v4.2.0      |    A Helm chart for bootstrapping default users an... |


Note that not all services are needed for all use cases.

The following services are needed for all deployments:

|NAME                               |                   CHART VERSION  | APP VERSION  |   DESCRIPTION                                         |
|-----------------------------------|----------------------------------|--------------|-------------------------------------------------------|
|isecl-helm/aas                     |                    4.2.0         |  v4.2.0      |    A Helm chart for Installing ISecL-DC Authentica... |
|isecl-helm/cms                     |                    4.2.0         |  v4.2.0      |    A Helm chart for Installing ISecL-DC Certificat... |
|isecl-helm/hvs                     |                    4.2.0         |  v4.2.0      |    A Helm chart for Installing ISecL-DC Host Verif... |
|isecl-helm/trustagent              |                    4.2.0         |  v4.2.0      |    A Helm chart for Installing ISecL-DC Trust Agent   |

These services are the basis for any deployment, and will enable the Platform Integrity Attestation use case.

Intel SecL-DC can optionally utilize a NATS server to manage connectivity between the Host Verification Service and any number of deployed Trust Agent hosts.  This acts as an alternative to communication via REST APIs - in NATS mode, a connection is established with the NATS server, and messages are sent and received over that connection.  Using NATS mode requires configuration changes in the values.yaml files for the HVS and Trust Agent charts, as well as deployment of NATS itself:

|NAME                               |                   CHART VERSION  | APP VERSION  |   DESCRIPTION                                         |
|-----------------------------------|----------------------------------|--------------|-------------------------------------------------------|
|isecl-helm/nats                    |                    4.2.0         |  v4.2.0      |    A Helm chart for Installing NATS server            |

NATS deployments also require a setup job:

|NAME                               |                   CHART VERSION  | APP VERSION  |   DESCRIPTION                                         |
|-----------------------------------|----------------------------------|--------------|-------------------------------------------------------|
|isecl-helm/nats-init               |                    4.2.0         |  v4.2.0      |    A Helm chart for creating TLS certificates and ... |

Intel SecL-DC can optionally integrate with Kubernetes to control the placement of workloads based on the attestation status of worker nodes.  Trusted Workload Placement requires the following additional services:

|NAME                               |                   CHART VERSION  | APP VERSION  |   DESCRIPTION                                         |
|-----------------------------------|----------------------------------|--------------|-------------------------------------------------------|
|isecl-helm/admission-controller    |                    4.2.0         |  v4.2.0      |    A Helm chart for Installing ISecL-DC Admision C... |
|isecl-helm/isecl-controller        |                    4.2.0         |  v4.2.0      |    A Helm chart for Installing ISecL-DC Custom Con... |
|isecl-helm/isecl-scheduler         |                    4.2.0         |  v4.2.0      |    A Helm chart for Installing ISecL-DC Extended S... |

## Downloading the Deployment Answer Files

Each service and several setup jobs uses a values.yaml file for configuring the installation.  These can be downloaded directly from the Intel SecL Helm repo:

```
curl -fsSL -o cleanup-secrets.yaml https://raw.githubusercontent.com/intel-secl/helm-charts/v4.2.0-Beta/jobs/cleanup-secrets/values.yaml
curl -fsSL -o cms.yaml https://raw.githubusercontent.com/intel-secl/helm-charts/v4.2.0-Beta/services/cms/values.yaml
curl -fsSL -o aasdb-cert-generator.yaml https://raw.githubusercontent.com/intel-secl/helm-charts/v4.2.0-Beta/jobs/aasdb-cert-generator/values.yaml
curl -fsSL -o aas.yaml https://raw.githubusercontent.com/intel-secl/helm-charts/v4.2.0-Beta/services/aas/values.yaml
curl -fsSL -o aas-manager.yaml https://raw.githubusercontent.com/intel-secl/helm-charts/v4.2.0-Beta/jobs/aas-manager/values.yaml
curl -fsSL -o hvsdb-cert-generator.yaml https://raw.githubusercontent.com/intel-secl/helm-charts/v4.2.0-Beta/jobs/hvsdb-cert-generator/values.yaml
curl -fsSL -o hvs.yaml https://raw.githubusercontent.com/intel-secl/helm-charts/v4.2.0-Beta/services/hvs/values.yaml
curl -fsSL -o trustagent.yaml https://raw.githubusercontent.com/intel-secl/helm-charts/v4.2.0-Beta/services/ta/values.yaml
```

Note that the code sample above will rename each values.yaml file according to the service it configures for easier reference.

## Answer File Configuration

Each values.yaml file will contain placeholders calling out specific requirements for user input.  Below is a sample of the HVS values.yaml file.  Note each instance of **<user input>** and the description in the corresponding comment.

Most instances that require an IP address or hostname will use the IP or FQDN of the Kubernetes control plane.

```
# Default values for hvs

nameOverride: "" # The name for HVS chart<br> (Default: `.Chart.Name`)
controlPlaneHostname: <user input> # K8s control plane IP/Hostname (**REQUIRED**)

# Warning: Ensure that the naming is applied consistently for all dependent services when modifying nameOverride
dependentServices: # The dependent Service Name for deploying  HVS chart, default is the chart name and override is from nameOverride value.
  cms: cms
  aas: aas

config:
  envVarPrefix: HVS
  dbPort: 5432 # PostgreSQL DB port
  dbSSL: on # PostgreSQL DB SSL<br> (Allowed Values: `on`/`off`)
  dbSSLCert: /etc/postgresql/secrets/server.crt # PostgreSQL DB SSL Cert
  dbSSLKey: /etc/postgresql/secrets/server.key  # PostgreSQL DB SSL Key
  dbSSLCiphers: ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256 # PostgreSQL DB SSL Ciphers
  dbListenAddresses: "*" # PostgreSQL DB Listen Address
  dbName: hvsdb # HVS DB Name
  dbSSLMode: verify-full # PostgreSQL DB SSL Mode
  dbhostSSLPodRange: 10.1.0.0/8 # PostgreSQL DB Host Address(IP address/subnet-mask). IP range varies for different k8s network plugins(Ex: Flannel - 10.1.0.0/8 (default), Calico - 192.168.0.0/16).
  requireEKCertForHostProvision: <user input> # If set to true, worker node EK certificate should be registered in HVS DB, for AIK provisioning step of TA. (Allowed values: `true`\`false`)
  verifyQuoteForHostRegistration: <user input> # If set to true, when the worker node is being registered to HVS, quote verification will be done. Default value is false. (Allowed values: `true`\`false`)
  nats:
    enabled: false # Enable/Disable NATS mode<br> (Allowed values: `true`\`false`)
    servers: ""  # NATS Server IP/Hostname
    serviceMode: "" # The model for TA<br> (Allowed values: `outbound`)

# The values provided for serviceUsername and servicePassword here should be same as that of provided for aas.hvs.secret.serviceUsername and aas.hvs.secret.servicePassword in values.yaml file for aas-manager chart
secret:
  dbUsername:  # DB Username for HVS DB
  dbPassword:  # DB Password for HVS DB
  serviceUsername:  # Admin Username for HVS
  servicePassword:  # Admin Password for HVS

image:
  db:
    registry: dockerhub.io # The image registry where PostgreSQL image is pulled from
    name: postgres:11.7 # The image name of PostgreSQL
    pullPolicy: Always # The pull policy for pulling from container registry for PostgreSQL image
  svc:
    name: <user input> # The image name with which HVS image is pushed to registry<br> (**REQUIRED**)
    pullPolicy: Always # The pull policy for pulling from container registry for HVS<br> (Allowed values: `Always`/`IfNotPresent`)
    imagePullSecret:  # The image pull secret for authenticating with image registry, can be left empty if image registry does not require authentication

storage:
  nfs:
    server: <user input> # The NFS Server IP/Hostname<br> (**REQUIRED**)
    reclaimPolicy: Retain # The reclaim policy for NFS<br> (Allowed values: `Retain`/)
    accessModes: ReadWriteMany # The access modes for NFS<br> (Allowed values: `ReadWriteMany`)
    path: /mnt/nfs_share # The path for storing persistent data on NFS
    dbSize: 5Gi # The DB size for storing DB data for HVS in NFS path
    configSize: 10Mi # The configuration size for storing config for HVS in NFS path
    logsSize: 1Gi # The logs size for storing logs for HVS in NFS path
    baseSize: 6.1Gi # The base volume size (configSize + logSize + dbSize)

securityContext:
  hvsdbInit: # The fsGroup id for init containers for HVS DB
    fsGroup: 2000
  hvsdb: # The security content for HVS DB Service Pod
    runAsUser: 1001
    runAsGroup: 1001
  hvsInit: # The fsGroup id for init containers for HVS
    fsGroup: 1001
  hvs: # The security content for HVS Pod
    runAsUser: 1001
    runAsGroup: 1001
    capabilities:
      drop:
        - all
    allowPrivilegeEscalation: false

service:
  directoryName: hvs
  cms:
    containerPort: 8445 # The containerPort on which CMS can listen
  aas:
    containerPort: 8444 # The containerPort on which AAS can listen
    port: 30444 # The externally exposed NodePort on which AAS can listen to external traffic
  hvsdb:
    containerPort: 5432 # The containerPort on which HVS DB can listen
  hvs:
    containerPort: 8443 # The containerPort on which HVS can listen
    port: 30443 # The externally exposed NodePort on which HVS can listen to external traffic
  ingress:
    enable: false # Accept true or false to notify ingress rules are enable or disabled
```

Typically attributes that do not specifically call out for user input will not need to be changed unless there is a specific conflict from the Kubernetes environment.

When deploying individual services, all of the values.yaml files must be configured individually.

## Configuration Changes Needed for NATS Mode

If NATS mode will be used, the NATS service answer file must also be downloaded and configured, and additional configuration elements must be added to the HVS and Trust Agent values.yaml files.

Download and configure the NATS configuration file:

```
curl -fsSL -o nats.yaml https://raw.githubusercontent.com/intel-secl/helm-charts/v4.2.0-Beta/services/nats/values.yaml
```

In the hvs.yaml and trustagent.yaml files, configure the NATS block in the existing config element:

```    
    nats:
      enabled: true # Enable/Disable NATS mode<br> (Allowed values: `true`\`false`)
      servers: nats://<user input>:30222   # NATS Server IP/Hostname<br> (**REQUIRED IF ENABLED**)
      serviceMode: outbound # The model for TA (Allowed values: `outbound`) (**REQUIRED IF ENABLED**)
```

Note that the NATS server IP/hostname is typically the same as the Kubernetes control plane.

## Deployment

Once all of the values.yaml answer files have been configured, the Helm charts are ready to be deployed.  The services and their setup jobs must be run in the correct order to ensure that the appropriate secrets are generated and available when needed.

The following example shows a minimal installation for the Platform Integrity Attestation use case **without NATS**:

```
helm pull isecl-helm/cleanup-secrets
helm install cleanup-secrets -f cleanup-secrets.yaml isecl-helm/cleanup-secrets -n isecl --create-namespace
helm pull isecl-helm/cms
helm install cms isecl-helm/cms -n isecl -f cms.yaml
helm pull isecl-helm/aasdb-cert-generator
helm install aasdb-cert-generator isecl-helm/aasdb-cert-generator -f aasdb-cert-generator.yaml  -n isecl
helm pull isecl-helm/aas
helm install aas services/aas -n isecl -f aas.yaml
helm pull isecl-helm/aas-manager
helm install aas-manager jobs/aas-manager -n isecl -f aas-manager.yaml
helm pull isecl-helm/hvsdb-cert-generator
helm install hvsdb-cert-generator isecl-helm/hvsdb-cert-generator -f hvsdb-cert-generator.yaml -n isecl
helm pull isecl-helm/hvs
helm install hvs isecl-helm/hvs -n isecl -f hvs.yaml
helm pull isecl-helm/trustagent 
helm install trustagent isecl-helm/trustagent -n isecl -f trustagent.yaml
```

The following example shows an installation for the Platform Integrity Attestation use case **with NATS.**

```
helm pull isecl-helm/cleanup-secrets
helm install cleanup-secrets -f cleanup-secrets.yaml isecl-helm/cleanup-secrets -n isecl --create-namespace
helm pull isecl-helm/cms
helm install cms isecl-helm/cms -n isecl -f cms.yaml
helm pull isecl-helm/aasdb-cert-generator
helm install aasdb-cert-generator isecl-helm/aasdb-cert-generator -f aasdb-cert-generator.yaml  -n isecl
helm pull isecl-helm/aas
helm install aas services/aas -n isecl -f aas.yaml
helm pull isecl-helm/aas-manager
helm install aas-manager jobs/aas-manager -n isecl -f aas-manager.yaml
helm pull isecl-helm/nats
helm install nats isecl-helm/nats - nats.yaml -n isecl
helm pull isecl-helm/hvsdb-cert-generator
helm install hvsdb-cert-generator isecl-helm/hvsdb-cert-generator -f hvsdb-cert-generator.yaml -n isecl
helm pull isecl-helm/hvs
helm install hvs isecl-helm/hvs -n isecl -f hvs.yaml
helm pull isecl-helm/trustagent 
helm install trustagent isecl-helm/trustagent -n isecl -f trustagent.yaml
```
