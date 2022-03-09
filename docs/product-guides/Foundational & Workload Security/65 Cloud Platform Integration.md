## Cloud Platform Integration

Intel® SecL can be integrated with Cloud scheduler services (or potentially
other services) to provide additional security controls. For example, by
integrating Intel® SecL with the OpenStack scheduler service, the
OpenStack placement service can incorporate the Intel® SecL security
attributes into VM scheduling.

### The Integration Hub

The Integration Hub acts as the integration point between the Verification Service and a third party service. The primary purpose of the Hub is to collect and maintain up-to-date attestation information, and to “push” that information to the external service. The secondary purpose is to allow for multitenancy, the Verification Service does not allow for permissions to be applied for specific hosts, so a user with the “attestation” role can access all attestations for all hosts. By using separate Integration Hub instances for each Cloud environment (or "tenant"), the Hub will push attestations only for the associated hosts to a given tenant’s integration endpoints.

For example, Tenant A is using hosts 1-10 for an OpenStack environment. Tenant B is using hosts 11-15 for a Kubernetes environment. Two Hub instances must be configured, one managing tenant A's OpenStack cluster and a second instance managing Tenant B's Kubernetes environment.  Each integration Hub will automatically retrieve the list of hosts used by its configured orchestration endpoint, retrieve the attestation reports only for those hosts, and push the attestation attribute information to each configured endpoint. Neither tenant will have access to the Verification Service, and will not be able to see attestation or other host details regarding infrastructure used by other tenants.

Different integration endpoints can be added to the Integration Hub through a plugin architecture. By default, the Integration Hub includes plugins for OpenStack and Kubernetes (Kubernetes deployments require the additional installation of two Intel® SecL-DC Custom Resource Definitions on the Kube Control Plane).

![integration](./images/integration2.png)

### Integration with OpenStack

Starting in the Rocky release, OpenStack can now use “Traits” to provide qualitative data about Nova Compute hosts, and to establish Trait requirements for VM instances. The updated scheduler will place VMs
requiring a given Trait on Nova Compute nodes that meet the Trait requirements.

Intel SecL-DC uses the Integration Hub to continually push platform integrity and Asset Tag information to the OpenStack Traits resources.  This means the OpenStack scheduler natively supports workload scheduling incorporating Intel SecL-DC security attributes, including attestation report Trust status and Asset Tags. The OpenStack Placement Service will automatically attempt to place images with Trait requirements on compute nodes that have those Traits.

???+ note 
    This control only applies to instances launched using the OpenStack scheduler, and the Traits functions will not affect manually-launched instances where a specific Compute Node is defined (since this does not use the scheduler at all). Intel SecL-DC uses existing OpenStack interfaces and does not modify OpenStack code.  The datacenter owner or OpenStack administrator is responsible for the security of the OpenStack workload scheduling process in general, and Intel recommends following published OpenStack security best practices.

#### Prerequisites

-   Verification Service must be installed and running.

-   OpenStack\* Rocky (or later) Nova, Glance, Horizon, and Keystone services must
    be installed and running

-   The Integration Hub must be installed and running.

#### Setting Image Traits

Image Traits define the policy for which Traits are required for that
Image to be launched on a Nova Compute node. By setting these Traits to
“required,” the OpenStack scheduler will require these same Traits to be
present on a Nova Compute node in order to launch instances of the
image.

To set the Image Traits for Intel SecL-DC, a specific naming convention
is used. This naming convention will match the Traits that the
Integration Hub will automatically push to OpenStack. Two types of
Traits are currently supported – one Trait is used to require that the
Compute Node be Trusted in the Attestation Report, and the other Trait
is used to require specific Asset Tag key/value pairs.

To require a `Trusted` Attestation Report:

```
CUSTOM_ISECL_TRUSTED=required
```

The naming convention for Asset Tags is more flexible, and any number of
these Traits can be used simultaneously.

???+ note 
    All of the Traits must be present on the Compute Node for the scheduler to allow instances to land, so be sure not to set mutually exclusive Asset Tag values.

```
CUSTOM_ISECL_AT_TAG_<key>_<value>=required`
```

For example, to define a Trait that will require an Asset Tag where
`State = CA` use the following:

```
CUSTOM_ISECL_AT_TAG_STATE_CA= required
```

These Traits can be set using CLI commands for OpenStack Glance:

```shell
openstack image set --property trait:CUSTOM_ISECL_AT_STATE__CA=required <image_name>
```

```shell
openstack image set --property trait:CUSTOM_ISECL_TRUSTED=required <image_name>
```

To remove a Trait so that it is no longer required for an Image:

```shell
openstack image unset --property trait:CUSTOM_ISECL_AT_STATE__CA <image_name>
```

```shell
openstack image unset --property trait:CUSTOM_ISECL_TRUSTED <image_name>
```

#### Configuring the Integration Hub for Use with OpenStack

The Integration Hub must be configured with the API URLs and credentials for the OpenStack instance it will integrate with.  This can be done during installation using the "OPENSTACK_..." variables shown in the `ihub.env` answer file  sample (see the Installing the Integration Hub section).

However, this configuration can also be performed after installation using CLI commands:

```
ihub setup openstack --endpoint-url="http://openstack:5000/v3" --endpoint-user="username" --endpoint-pass="password"
```

Restart the Integration Hub after configuring the endpoint. Note that "endpoint name" should be replaced with any user-friendly name for the OpenStack instance you would prefer.

#### Scheduling Instances

Once Trait requirements are set for images and the Integration Hub is
configured to push attributes to OpenStack, instances can be launched in
OpenStack as normal. As long as the OpenStack Nova scheduler is used to
schedule the workloads, only compliant Compute Nodes will be scheduled
to run instances of controlled images.

???+ note 
    This control only applies to instances launched using the OpenStack scheduler, and the Traits functions will not affect manually-launched instances where a specific Compute Node is defined (since this does not use the scheduler at all). Intel SecL-DC uses existing OpenStack interfaces and does not modify OpenStack code.  The datacenter owner or OpenStack administrator is responsible for the security of the OpenStack workload scheduling process in general, and Intel recommends following published OpenStack security best practices.

### Integration with Kubernetes

Through the use of Custom Resource Definitions for the Kubernetes
Control Plane, Intel® Security Libraries can make Kubernetes aware of Intel®
SecL security attributes and make them available for pod orchestration.
In this way, a security-sensitive pod can be launched only on `Trusted`
physical worker nodes, or on physical worker nodes that match specified
Asset Tag values.

???+ note 
    This control only applies to pods launched using the Kubernetes scheduler, and these scheduling controls will not affect manually-launched instances where a specific worker node is defined (since this does not use the scheduler at all). Intel SecL-DC uses existing Kubernetes interfaces and does not modify Kubernetes code, using only the standard Custom Resource Definition mechanism to add this functionality to the Kubernetes Control Plane.  The datacenter owner or Kubernetes administrator is responsible for the security of the Kubernetes workload scheduling process in general, and Intel recommends following published Kubernetes security best practices.

#### Prerequisites

-   Verification Service must be installed and running.

-   Kubernetes Control Plane Node must be installed and running

-   The supported Kubernetes versions are `1.21.4`, `1.22.2` and the
    integration is validated with `1.21.4`, `1.22.2`

-   Kubernetes Worker Nodes must be configured as physical hosts and
    attached to the Control Plane Node


#### Installing the Intel® SecL Custom Resource Definitions

Intel® SecL uses Custom Resource Definitions to add the ability to base
orchestration decisions on Intel® SecL security attributes to
Kubernetes. These CRDs allow Kubernetes administrators to configure pods
to require specific security attributes so that the Kubernetes Control Plane
Node will schedule those pods only on Worker Nodes that match the
specified attributes.

Two CRDs are required for integration with Intel® SecL – an extension
for the Control Plane nodes, and a scheduler extension. A single installer will
deploy both of these CRDs. The extensions are deployed as a Kubernetes
deployment in the `isecl` namespace.

???+ note 
    Please refer detail steps given for  `3.15 Installing the Intel® SecL Kubernetes Extensions and Integration Hub` section.


#### Configuring Pods to Require Intel® SecL Attributes

1.  (Optional) Verify that the worker nodes have had their Intel® SecL
    security attributes populated:

    ```shell
    kubectl get nodes --show-labels
    ```

    The output should show the Trust staus and any Asset Tags applied to
    all of the registered Worker Nodes.

2.  Add the following to any Pod creation files:

    ```yaml
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: isecl.trusted
                    operator: In
                    values:
                      - "true"
                  - key: isecl.TAG_Country
                    operator: In
                    values:
                      - CA
                      - US
                  - key: isecl.TAG_Customer
                    operator: In
                    values:
                      - Coke
                      - Pepsi
                  - key: isecl.TAG_State
                    operator: In
                    values:
                      - CA
    ```


The `isecl.trusted` key defines the requirement for a Trusted host. Only
one of these keys should be used. The `isecl.TAG_` keys indicate Asset
Tags; if the workload should only launch on hosts with the `COUNTRY=US` Asset Tag, the pod should be launched with the matchExpression key
`isecl.TAG_COUNTRY` with the value `US`.

All of the matchExpression definitions must be true for a given worker
node to launch the pod – in the example above, the host must be
attested as Trusted with Asset Tags `Country=US` ,`Customer=Customer1` and `State=CA`. If the worker node has
additional Asset Tags beyond the ones required, the pod will still be
able to be launched on that node. However, if one of the specified
Tags is missing or has a different value, that worker node will not be
used for that pod.

#### Tainting Untrusted Worker Nodes

Optionally, the Intel® SecL Kubernetes controller can be configured to flag
worker nodes as `tainted` to prevent any pods from launching on them.
This restriction is applied regardless of whether the pod has a specific
trust policy – if a worker node is flagged as `tainted` no pods will be
launched on that worker.

This setting is disabled by default. To enable this setting:

1.  Edit the `isecl-controller.yaml` file under
    `/opt/isecl-k8s-extensions/yamls/isecl-controller.yaml` 
    
2. set the following options:

   ```
   # To taint nodes when a report indicates the host is Untrusted:
   TAINT_UNTRUSTED_NODES=true
   # To taint nodes when they are joined to the Kubernetes cluster until they are Trusted:
   TAINT_REGISTERED_NODES=true
   # To taint nodes when they are rebooted until they are Trusted:
   TAINT_REBOOTED_NODES=true
   ```

3. Run

   ```shell
   kubectl apply -f /opt/isecl-k8s-extensions/yamls/isecl-controller.yaml
   ```

If the TAINT_UNTRUSTED option is used, worker nodes that attest as untrusted will be `tainted` with the
NoExecute flag and unable to launch pods.  There may be a delay of up to 2 minutes before the taint is applied as the Integration Hub only retrieves new reports every 2 minutes.

If the TAINT_REGISTERED option is used, worker nodes will be tainted by default by the Intel SecL extension controller.  The taint will be removed when the Integration Hub sees a Trusted report and updates the controller.

Similarly, if the TAINT_REBOOTED option is used, worker nodes will be tainted by default when rebooted by the Intel SecL extension controller.  The taint will be removed when the Integration Hub sees a Trusted report and updates the controller.

If a worker was previously considered tainted and the untrusted state is resolved, the Intel® SecL CRDs will remove the tainted flag and the worker will be able to launch pods again.

???+ note
    Important: The taint rules can potentially result in all available worker nodes becoming tainted due to a mass-reboot, a mistake with Flavor configuration, etc.  This is especially relevant for small proof-of-concept deployments that may only use a limited number of worker nodes.  Tainted worker nodes will be evacuate any Intel SecL services, which prevents ISecL from being used to remediate the issue.  It is strongly recommended to set taint tolerations for each of the Intel SecL management plane services such that they can still run on tainted workers to prevent such a situation.  These settings are not required if the taint options are set to "false" (default).

The following taints are used by the Intel SecL extension controller:

```json
"taints": [
    {
      "effect": "NoSchedule",
      "key": "untrusted",
      "value": "true"
    },
    {
      "effect": "NoExecute",
      "key": "untrusted",
      "value": "true"
    }
```

Pods can be configured to ignore these taints using "tolerations:"

```yaml
tolerations:
- key: "untrusted"
operator: "Equal"
value: "true"
effect: "NoSchedule"
- key: "untrusted"
operator: "Equal"
value: "true"
effect: "NoExecute"
```

Note that some pods, including the Trust Agent, should use these tolerations even on untrusted workers.  The Trust Agent is particularly important because, without the Trust Agent running on the worker to generate a new trust report, the taints will not be cleared.