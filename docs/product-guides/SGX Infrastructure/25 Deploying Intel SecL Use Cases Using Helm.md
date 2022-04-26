# Deploying SKC Use Cases Using Helm

???+ note
SGX Attestation Infrastructure components need to be deployed in order to realize the SKC use case.

As indicated in the Features section, SKC provides 3 features essentially:

-   SGX Attestation Support: this is the feature that CSPs provide to tenants who need to run SGX workloads that require attestation.
-   SGX Support  in Orchestrators: this feature allows to discover SGX support in physical servers and related information:
  -   SGX supported.

  -   SGX enabled.

  -   Size of RAM reserved for SGX. It's called Enclave Page Cache (EPC).

  -   Flexible Launch Control enabled.
-   Key Protection: this is the feature used by tenants using a CSP to run workloads with key protection requirements.

The high-level architectures of these features are presented in the next sub-sections.

##  Key Protection

Key Protection leverages the SGX Attestation support and optionally, the SGX support in orchestrators.

Key Protection is implemented by the SKC Client -- a set of libraries - which must be linked with a tenant workload, like Nginx, deployed in a CSP environment and the Key Broker Service (KBS) deployed in the tenant's enterprise environment. The SKC Client retrieves the keys needed by the workload from KBS after proving that the key can be protected in an SGX enclave as shown in the diagram below.

![](images/image-20200727163116765.png)

## Deployment Steps

Configure the values.yaml answer file.  Each values.yaml file will contain "<user input>" prompts and a comment describing the information required.

## Use Case charts Deployment

Pull the Helm charts from the Helm repository and then deploy.

Set the release version
```
export VERSION=v5.0.0
```

### TEE-Attestation:

```
helm pull isecl-helm/TEE-Attestation --version $VERSION && tar -xzf TEE-Attestation-$VERSION.tgz TEE-Attestation/values.yaml
helm install tee-attastation isecl-helm/tee-attestation --version $VERSION -f tee-attestation/values.yaml --create-namespace -n <namespace>
```

Follow the build instructions for SKC Library and update the isecl-skc-k8s.env and install SKC Library using below command

```
./skc-bootstrap.sh install
```


### TEE Attestation Orchestration

```
helm pull isecl-helm/TEE-Attestation-Orchestration --version $VERSION && tar -xzf TEE-Attestation-Orchestration-$VERSION.tgz TEE-Attestation-Orchestration/values.yaml
helm install tee-attestation-orchestration isecl-helm/TEE-Attestation-Orchestration --version $VERSION -f TEE-Attestation-Orchestration/values.yaml --create-namespace -n <namespace>
```

Follow the build instructions for SKC Library and update the isecl-skc-k8s.env and install SKC Library using below command

```
./skc-bootstrap.sh install
```

### TEE Orchestration

```
helm pull isecl-helm/TEE-Orchestration --version $VERSION && tar -xzf TEE-Orchestration-$VERSION.tgz TEE-Orchestration/values.yaml
helm install tee-orchestration isecl-helm/TEE-Orchestration --version $VERSION -f TEE-Orchestration/values.yaml --create-namespace -n <namespace>
```

Follow the build instructions for SKC Library and update the isecl-skc-k8s.env and install SKC Library using below command

```
./skc-bootstrap.sh install
```



