# Deploying Intel SecL Use Cases Using Helm

Intel SecL-DC provides combined Helm charts to deploy an entire use case using a single configuration file, rather than deploying each individual service.  **This is the recommended method for deployment.**

Available use case deployments include:

- Host Attestation (This enables Platform Integrity Attestation)
- Trusted Workload Placement (This deploys the Host Attestation use case and adds integration with Kubernetes to place worklaods on trusted workers)
  - The Trusted Workload Placement deployment can optionally be divided into separate deployments for the Cloud Service Provider and the Control Plane
    - The Control Plane deployment includes the CMS, AAS, HVS, and optionally supports NATS
    - The Cloud Service Provider deployment includes the Trust Agent, Integration Hub, Admission-Controller, Intel SecL Controller, and Intel SecL Scheduler

Additional use case deployments may be made available in future releases.

## Deployment Steps

Configure the values.yaml answer file.  Each values.yaml file will contain "<user input>" prompts and a comment describing the information required.

## Use Case charts Deployment

Pull the Helm charts from the Helm repository and then deploy.

Set the release version
```
VERSION=v5.0.0
```

### Host-Attestation:

```
helm pull isecl-helm/Host-Attestation --version $VERSION && tar -xzf Host-Attestation-$VERSION.tgz Host-Attestation/values.yaml
helm install host-attastation isecl-helm/Host-Attestation --version $VERSION -f Host-Attestation/values.yaml --create-namespace -n <namespace>
```

### Trusted Workload Placement

```
helm pull isecl-helm/Trusted-Workload-Placement --version $VERSION && tar -xzf Trusted-Workload-Placement-$VERSION.tgz Trusted-Workload-Placement/values.yaml
helm install trusted-workload-placement isecl-helm/Trusted-Workload-Placement --version $VERSION -f Trusted-Workload-Placement/values.yaml --create-namespace -n <namespace>
```

### Trusted Workload Placement - Cloud Service Provider Components Only

```
helm pull isecl-helm/Trusted-Workload-Placement-Cloud-Service-Provider --version $VERSION && tar -xzf Trusted-Workload-Placement-Cloud-Service-Provider-$VERSION.tgz Trusted-Workload-Placement-Cloud-Service-Provider/values.yaml
helm install twp-cloud-service-provider isecl-helm/Trusted-Workload-Placement-Cloud-Service-Provider --version $VERSION -f Trusted-Workload-Placement-Cloud-Service-Provider/values.yaml --create-namespace -n <namespace>
```

### Trusted Workload Placement - Control Plane Components Only

```
helm pull isecl-helm/Trusted-Workload-Placement-Control-Plane --version $VERSION && tar -xzf Trusted-Workload-Placement-Control-Plane-$VERSION.tgz Trusted-Workload-Placement-Control-Plane/values.yaml
helm install twp-control-plane isecl-helm/Twp-Control-Plane --version $VERSION -f Twp-Control-Plane/values.yaml --create-namespace -n <namespace>
```
