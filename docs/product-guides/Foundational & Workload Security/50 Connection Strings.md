# Connection Strings

Connection Strings define a remote API resource endpoint that will be
used to communicate with the registered host for retrieving TPM quotes
and other host information. Connection Strings differ based on the type
of host.

## Trust Agent

The Trust Agent connection string connects directly to the Trust Agent
on a given host. The Verification Service will use a service account
with the needed Trust Agent permissions to connect to the Trust Agent.
In previous Intel® SecL versions, each Trust Agent had its own unique
user access controls. Starting in the 1.6 release, all authentication
has been centralized with the new Authentication and Authorization
Service, eliminating the need for credentials to be provided for
connection strings connecting to Trust Agent resources.

By default, the Trust Agent uses "HTTP" mode, which uses the following connection string:

`intel:https://<HostNameOrIp>:31443`

The Trust Agent can also be used in NATS mode, which uses a slightly different connection string:

```
intel:nats://<unique host identifier, configured at Trust Agent installation>
```

The unique host identifier is a unique ID used by NATS to differentiate services when passing messages.  Any unique string is acceptable, but good examples can be the host's FQDN or hardware UUID.

Note: Make sure no two hosts are configured with same host identifier.

## VMware ESXi

### Importing VMware TLS Certificates

Before connecting to vCenter to register hosts or clusters, the vCenter TLS certificate needs to be imported to the Verification Service.  This must be done for each vCenter server that the Verification Service will connect to, for importing Flavors or registering hosts.

* Download the root CA certs from vCenter:

   ```shell
   wget --no-proxy "*" https://<vCenter IP or hostname>/certs/download.zip --no-check-certificate
   ```

   This downloads all the root CA certificates for you into `download.zip` file.

   ```shell
   unzip download.zip
   ```

   All of the certificates will be stored under `<pwd>/certs/`. Certs will be in `PEM` format.



* Upload the certificates to the HVS

   ```http
   POST https://%3CIP%3E:30443/hvs/v2/ca-certificates

   {
       "name": "<cert name>",
       "type": "root",
       "certificate": "MIIELTCCAxW..."
   }
   ```

???+ note 
    Please make sure that the certificate does not contain any other characters other than the base64 characters like that of `\n` or `-----BEGIN CERTIFICATE-----` etc.



* After upload is successful, restart the service by deleting the pod.  The Kubernetes deployment will automatically launch a new pod, which will reflect the updated settings.

```bash
kubectl delete pod -n <namespace> <hvspodname>
```


### Registering a VMware ESXi Host

The VMware ESXi connection string is actually directed to vCenter, not
the actual ESXi host. Many ESXi hosts managed by the same vCenter server
will use the same connection string. The username and password specified
are vCenter credentials, and the vCenter "Validate Session" privilege is
required for access.

```shell
vmware:https://<vCenterHostNameOrIp>:443/sdk;h=<hostname of ESXi host>;u=<username>;p=<password>
```
