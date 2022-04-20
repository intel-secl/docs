# Deploying Intel SecL Use Cases Using Helm

Intel SecL-DC provides combined Helm charts to deploy an entire use case using a single configuration file, rather than deploying each individual service.  **This is the recommended method for deployment.**

Available use case deployments include:

- Host Attestation (This enables Platform Integrity ASttestation)
- Trusted Workload Placement (This deploys the Host Attestation use case and adds integration with Kubernetes to place worklaods on trusted workers)
  - The Trusted Workload Placement deployment can optionally be divided into separate deployments for the Cloud Service Provider and the Control Plane
    - The Control Plane deployment includes the CMS, AAS, HVS, and optionally supports NATS
    - The Cloud Service Provider deployment includes the Trust Agent, Integration Hub, Admission-Controller, Intel SecL Controller, and Intel SecL Scheduler

Additional use case deployments may be made available in future releases.

## Deployment Steps

Download the values.yaml deployment answer file for the desired usecase:

---
**NOTE***
Only download the values.yaml file for the specific deployment needed.  For example, the host-attestation values.yaml is all that is required to deploy the host-attestation use case.
---

```bash
curl -fsSL -o values.yaml https://raw.githubusercontent.com/intel-secl/helm-charts/v4.2.0-Beta/usecases/host-attestation/values.yaml
curl -fsSL -o values.yaml https://raw.githubusercontent.com/intel-secl/helm-charts/v4.2.0-Beta/usecases/trusted-workload-placement/values.yaml
curl -fsSL -o values.yaml https://raw.githubusercontent.com/intel-secl/helm-charts/v4.2.0-Beta/usecases/twp-control-plane/values.yaml
curl -fsSL -o values.yaml https://raw.githubusercontent.com/intel-secl/helm-charts/v4.2.0-Beta/usecases/twp-cloud-service-provider/values.yaml
```

Configure the values.yaml answer file.  Each values.yaml file will contain "<user input>" prompts and a comment describing the information required.  Below is a sample of the Trusted Workload Placement values.yaml:

```yaml
---
cms:
  image:
    name: <user input> # Certificate Management Service image name (**REQUIRED**)

aas:
  image:
    name: <user input> # Authentication & Authorization Service image name (**REQUIRED**)
  secret:
    dbUsername:  <user input> # DB Username for AAS DB
    dbPassword:  <user input> # DB Password for AAS DB

aas-manager:
  image:
    name: <user input> # Authentication & Authorization Manager image name (**REQUIRED**)
  secret:
    superAdminUsername: <user input> # Username for the service installation user (**REQUIRED**)
    superAdminPassword: <user input> # Password for the service installation user (**REQUIRED**)
    globalAdminUsername: <user input> # Username for the Administrator user (**REQUIRED**)
    globalAdminPassword: <user input> # Password for the Administrator user (**REQUIRED**)

hvs:
  image:
    name: <user input> # Host Verification Service image name<br> (**REQUIRED**)
  config:
    requireEKCertForHostProvision: false # If set to true enable Endorsement Certificate Pre-registration (Allowed values: `true`\`false`)
    verifyQuoteForHostRegistration: false # If set to true enforce Endorsement Certificate Pre-registration (Allowed values: `true`\`false`)
  secret:
    dbUsername:  # Database Username for HVS DB
    dbPassword:  # Database Password for HVS DB

trustagent:
  image:
    name: <user input> # Trust Agent image name (**REQUIRED**)

  nodeLabel:
    txt: TXT-ENABLED # The node label for TXT-ENABLED hosts<br> (**REQUIRED IF NODE IS TXT ENABLED**)
    suefi: "SUEFI-ENABLED" # The node label for SUEFI-ENABLED hosts (**REQUIRED IF NODE IS SUEFI ENABLED**)

  config:
    tpmOwnerSecret:  # The TPM owner secret if TPM is already owned. Recommended to leave blank for a null secret unless the TPM is already owned with a different secret

  hostAliasEnabled: false # Set this to true for using host aliases.  Also add corresponding entries for ip and hostnames. hostalias is required when ingress is deployed and pods are not able to resolve the domain names
  aliases:
    hostAliases:
      - ip: ""
        hostnames:
          - ""
          - ""

nats:
  clientPort: 30222

nats-init:
  image:
    name: <user input> # The image name of nats-init container, required for NATS only

isecl-controller:
  image:
    name: <user input> # ISecL Controller Service image name (**REQUIRED**)
  nodeTainting:
    taintRegisteredNodes: false # If set to true, taints worker nodes when joined to the cluster until they are attested as Trusted. (Allowed values: `true`\`false`)
    taintRebootedNodes: false # If set to true, taints worker nodes when rebooted until they are attested as Trusted. (Allowed values: `true`\`false`)
    taintUntrustedNode: false # If set to true, taints worker nodes when their trust status is false. (Allowed values: `true`\`false`)

ihub:
  image:
    name: <user input> # Integration Hub Service image name (**REQUIRED**)
  k8sApiServerPort: 6443

isecl-scheduler:
  image:
    name: <user input> # ISecL Scheduler image name (**REQUIRED**)

admission-controller:
  caBundle: <user input> #CA Bundle is used for signing new TLS certificates. value can be obtained by running kubectl config view --raw --minify --flatten -o jsonpath='{.clusters[].cluster.certificate-authority-data}'

global:
  controlPlaneHostname: <user input> # K8s control plane IP/Hostname (**REQUIRED**)
  controlPlaneLabel: node-role.kubernetes.io/master #K8s control plane label (**REQUIRED**)<br> Example: `node-role.kubernetes.io/master` in case of `kubeadm`/`microk8s.io/cluster` in case of `microk8s`

  image:
    pullPolicy: Always # The pull policy for pulling from container registry (Allowed values: `Always`/`IfNotPresent`)
    imagePullSecret:  # The image pull secret for authenticating with image registry, can be left empty if image registry does not require authentication
    initName: <user input> # The image name of init container

  config:
    dbhostSSLPodRange: 10.1.0.0/8 # PostgreSQL DB Host Address(IP address/subnet-mask). IP range varies for different k8s network plugins(Ex: Flannel - 10.1.0.0/8 (default), Calico - 192.168.0.0/16).
    nats:
      enabled: false # Enable/Disable NATS mode<br> (Allowed values: `true`\`false`)
      servers: nats://<user input>:30222   # NATS Server IP/Hostname (**REQUIRED IF ENABLED**)
      serviceMode: <user input> # The model for TA<br> (Allowed values: `outbound`)<br> (**REQUIRED IF ENABLED**)

  hvsUrl: <user input> # Hvs Base Url, Do not include "/" at the end. e.g for ingress https://hvs.isecl.com/hvs/v2 , for nodeport  https://<control-plane-hostname or control-plane-IP>:30443/hvs/v2
  cmsUrl: <user input> # CMS Base Url, Do not include "/" at the end. e.g for ingress https://cms.isecl.com/cms/v2 , for nodeport https://<control-plane-hostname or control-plane-IP>:30445/cms/v1
  aasUrl: <user input> # Authservice Base Url, Do not include "/" at the end. e.g for ingress https://aas.isecl.com/aas/v1 , for nodeport https://<control-plane-hostname or control-plane-IP>:30444/aas/v1

  storage:
    nfs:
      server: <user input> # The NFS Server IP/Hostname<br> (**REQUIRED**)
      path: /mnt/nfs_share  # The path for storing persistent data on NFS

  service:
    cms: 30445 # The service port for Certificate Management Service
    aas: 30444 # The service port for Authentication Authorization Service
    hvs: 30443 # The service port for SGX Host Verification Service
    ta: 31443 # The service port for Trust Agent

  ingress:
    enable: false # Accept true or false to notify ingress rules are enable or disabled

  aas:
    secret:
      adminUsername:  # Admin Username for AAS
      adminPassword:  # Admin Password for AAS

  hvs:
    secret:
      serviceUsername:  # Admin Username for HVS
      servicePassword:  # Admin Password for HVS

  ihub:
    secret:
      serviceUsername:  # Admin Username for IHub
      servicePassword:  # Admin Password for IHub
```

## Use Case charts Deployment

Pull the Helm charts from the Helm repository and then deploy.  

### Host-Attestation:

```
helm pull isecl-helm/Host-Attestation
helm install host-attastation isecl-helm/Host-Attestation -f values.yaml --create-namespace -n <namespace>
```

### Trusted Workload Placement

```
helm pull isecl-helm/Trusted-Workload-Placement
helm install host-attastation isecl-helm/Trusted-Workload-Placement -f values.yaml --create-namespace -n <namespace>
```

### Trusted Workload Placement - Cloud Service Provider Components Only

```
helm pull isecl-helm/twp-cloud-service-provider
helm install host-attastation isecl-helm/twp-cloud-service-provider -f values.yaml --create-namespace -n <namespace>
```

### Trusted Workload Placement - Control Plane Components Only

```
helm pull isecl-helm/twp-control-plane
helm install host-attastation isecl-helm/twp-control-plane -f values.yaml --create-namespace -n <namespace>
```

### Workload Confidentiality

```
helm pull isecl-helm/workload-confidentiality
helm install host-attastation isecl-helm/workload-confidentiality -f values.yaml --create-namespace -n <namespace>
```
