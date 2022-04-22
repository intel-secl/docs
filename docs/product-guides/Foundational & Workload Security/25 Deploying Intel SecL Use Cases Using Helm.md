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
helm pull isecl-helm/host-attestation && tar -xzf host-attestation-$VERSION.tgz host-attestation/values.yaml
helm install host-attastation isecl-helm/host-attestation -f host-attestation/values.yaml --create-namespace -n <namespace>
```

### Trusted Workload Placement

```
helm pull isecl-helm/trusted-workload-placement && tar -xzf trusted-workload-placement-$VERSION.tgz trusted-workload-placement/values.yaml
helm install trusted-workload-placement isecl-helm/trusted-workload-placement -f trusted-workload-placement/values.yaml --create-namespace -n <namespace>
```

### Trusted Workload Placement - Cloud Service Provider Components Only

```
helm pull isecl-helm/twp-cloud-service-provider && tar -xzf twp-cloud-service-provider-$VERSION.tgz twp-cloud-service-provider/values.yaml
helm install twp-cloud-service-provider isecl-helm/twp-cloud-service-provider -f twp-cloud-service-provider/values.yaml --create-namespace -n <namespace>
```

### Trusted Workload Placement - Control Plane Components Only

```
helm pull isecl-helm/twp-control-plane && tar -xzf twp-control-plane-$VERSION.tgz twp-control-plane/values.yaml
helm install twp-control-plane isecl-helm/twp-control-plane -f twp-control-plane/values.yaml --create-namespace -n <namespace>
```
