# Endorsement Certificate Pre-Registration

Added in the 4.2 release, this feature adds an additional pre-registration step for Trust Agent hosts.  When this feature is enabled, Trust Agent provisioning and registration will be denied unless the Endorsement Certificate (EC) from the TPM of the Trust Agent host is known to the Verification Service.

This feature is intended for deployments where physical control of compute resources is limited, as in some 5G wireless, Edge, or IoT device deployments.  By pre-registering the Endorsement Certificate of the TPM before shipping the equipment to its physical destination, the HVS can be restricted to allow provisioning and registration only with known TPMs.  This mitigates the possibility of a malicious host being registered from a remote location where physical security may be weaker.

To enable this feature, set the following options to "true" in the Helm congifuration for the HVS (either the HVS individual service values.yaml, or the values.yaml for any use case deployment):

```
  config:
    requireEKCertForHostProvision: true # If set to true enable Endorsement Certificate Pre-registration (Allowed values: `true`\`false`)
    verifyQuoteForHostRegistration: true # If set to true enforce Endorsement Certificate Pre-registration (Allowed values: `true`\`false`)
```
---
**NOTE**

When this feature is enabled, Trust Agent provisioning will fail for any hosts whose TPM EC has not been pre-registered.  For the Trust Agent Kubernetes daemonset, this means the Trust Agent pods will fail to deploy.

If deploying Intel SecL as individual service deployments, it is recommended that the ECs be registered to the HVS before deploying the Trust Agent service.

The Trust Agent daemonset will continually re-attempt provisioning after failure, so the EC can also be registered after the entire application (or entire use case deployment) has been deployed.  After the EC is registered, simply delete the Trust Agent pod running on the worker node whose EC was just registered.  This will trigger a re-deployment of the pod from the daemonset, and the Trust Agent should deploy successfully.

This feature is primarily intended for worker nodes added to the cluster after the Intel SecL applciation has been fully deployed.  In this case, the TPM EC should be registerd before adding the node to the cluster.  Once the node is added, the Trust Agent will deploy successfully as expected.

---

To register a TPM Endorsement Certificate with the HVS, the EC and its issuer need to be retrieved from the TPM, and then provided as part of an API call to the HVS.  **These steps must be followed for each individual host to be attested, as each TPM Endorsement Certificate is unique.**

## Extract the Endorsement Certificate and Issuer from a TPM

The example below will retrieve the TPM Endorsement Certificate in base64 encoding from a RHEL server:

```
yum install tpm2-tools
tpm2_nvread -P hex:<owner secret> -x 0x1c00002 -a 0x40000001 -f ekcert.der or tpm2_nvread -P hex:<owner secret> -C 0x40000001 -o ekcert.der  0x1c00002
openssl x509 -inform der -in ekcert.der | base64 | tr -d " \t\n\r"
```

Next, retrieve the Issuer of the certificate:

```
openssl x509 -inform der -in ekcert.der --text | grep -Po 'CN =\K.*'
```

---
**NOTE**

The TPM "owner secret" will either be null or a 40 character hex string.  Note that if the TPM ownership secret is set to a value, the Trust Agent must be configured to use that same TPM owner secret when it is deployed.  This means either:

- Setting the TPM owner secret to run the above commands and then clearing TPM ownership to use the null secret for the Trust Agent
  **OR** 
- Configuring an identical owner secret for all Trust Agent hosts and using that secret both for these commands and for the Trust Agent deployment.

---

## Register the Endorsement Certificate to the HVS

Using the base64-encoded Endorsement Certificate and Issuer, use the following API to register the EC with the HVS:

```
POST https://<HVS IP or hostname>:<HVS port>/hvs/v2/tpm-endorsements
{
    "certificate": "Mjo2ZTowYjo3NDo0YzoKICAgICAgICAgM2M6ZDY6Yjc6NTU6M2M6Mjc6Y2U6OTU6YTY6ODQ6MjU6MmU6MTc6ODM6YjI6OTU6NzY6OGI6CiAgICAgICAgIGQzOjFkOjg0OjE1OjNlOmQ5OmI1Ojk1OjA0OjJkOjRlOmY5OjkzOjVmOjMxOjM3OjQwOmJhOgogICAgICAgICBjOTo1ZDo0OTo5OTowMjpiODo5NzoyNToxZjplYTpkMzpmNTpjYzowMzpkOTpiZTo1MDo5YjoKICAgICAgICAgNzk6NDQ6NmE6Yzg6M2I6YmU6MDc6MjQ6ZmI6YTA6YTE6N2Q6Y2U6NWM6YjM6MmE6NWY6OTE6CiAgICAgICAgIGJhOjgwOmRjOjk3OjhlOmI1OjM5OjJkOmQ3OjcyOjE0OjJjOjM4OmYyOjM1OmU0OjBhOmY5OgogICAgICAgICA5MjpkNjoxNjoyMTpiNjpiZTo0ZjpmODozYjo1MTpkYTpkNjpiMzo1NzoyZDpkNTphMjoxYjoKICAgICAgICAgOTc6N2U6NGE6ZGQ6ZGM6NmQ6ODE6OTM6NDM6YzI6MzI6ZGE6MGM6ZWM6MDY6NzY6Zjg6ZjM6CiAgICAgICAgIGNhOjc5OjdhOmNkOjQ0OjZmOjgxOmUwOmUyOjQ1OjQxOmFkOjgwOmZkOmQ4OmM4OjQ4OjdiOgogICAgICAgICBiODplODo5NzoyNTo2YToyNTo5OTplNjpjMjpmOTpiMDo1YzpjMTpkNzo1Mzo5ZToyNDplOToKICAgICAgICAgNzM6YzY6Nzk6MWU6Mjc6NmQ6ZDI6MmM6NWQ6ZTc6MmQ6ZjI6OGU6ODA6NzY6NTM6ZjI6NDM6CiAgICAgICAgIDJiOmI1OjNhOmY1OjNmOmFhOjVjOjA0OjUxOmFmOjZiOmM0OjEwOmQ2OmRlOmQ3OjFkOmE0OgogICAgICAgICBiNTo5NDo2MTowZDo0Yjo3NDpkZDo4Nzo5MjpjYTo1NDozNjpmODowYjo3ZDpkNTo0NTowNjoKICAgICAgICAgYzk6MWI6ZGE6NDk6MDQ6NGM6ZGU6YmE6MGI6Mzc6Zjk6ZGU6ZTQ6YjY6OGU6MWI6Mjc6ZmI6CiAgICAgICAgIDdiOjA3OmM0OmVjCi0tLS0tQkVHSU4gQ0VSVElGSUNBVEUtLS0tLQpNSUlFbkRDQ0E0U2dBd0lCQWdJRWI2QjNnakFOQmdrcWhraUc5dzBCQVFzRkFEQ0JnekVMTUFrR0ExVUVCaE1DClJFVXhJVEFmQmdOVkJBb01HRWx1Wm1sdVpXOXVJRlJsWTJodWIyeHZaMmxsY3lCQlJ6RWFNQmdHQTFVRUN3d1IKVDFCVVNVZEJLRlJOS1NCVVVFMHlMakF4TlRBekJnTlZCQU1NTEVsdVptbHVaVzl1SUU5UVZFbEhRU2hVVFNrZwpVbE5CSUUxaGJuVm1ZV04wZFhKcGJtY2dRMEVnTURBM01CN123VEUxTVRJeU1qRXpNRFkwTkZvWERUTXdNVEl5Ck1qRXpNRFkwTkZvd0FEQ0NBU0l3RFFZSktvWklodmNOQVFFQkJRQURnZ0VQQURDQ0FRb0NnZ0VCQUtYMTh2bmwKaFpqNWlvQnBKL3pwbmlneCtJSlNmS3FSRXE0dkdrRzk4Q1h0bXJJbTZNaWFkN09XQUpNdnZCbzhHWnArMENXTgpqbzZaWG1BbnNaeEUvd2RuUTBZdjA4WFdjOCtQblV1M1BaMFBoK3Q3RXRlaGUxS1dWazl2YllVZUF5TFVuc1FBCkg2cStIRDgzTXQ3bHk0WjBFRDlvUFU1UHpoSGVOSGhkTTM3VWNURVk5MkprTU85eGplODhVTEJnNEJ5RTdIdGgKcU5iUzlsTFZadG0xVk9mOVVUR2Fwem5sdnEyaGM2MlBuZWFWNllZbmdvM1N6ODN0d3o2dy8xVDkvZFV5TnliNgo0THBrUUtPTHcxdTNMYm9icndtT3NBREtBOE56UTdtbEtDd0VDRHZmZmFwRHlVV1hrMzVVc3V0Vkk5cTI1di9yCmUwY29CTnd6b2dJWUxKOENBd0VBQWFPQ0FaZ3dnZ0dVTUZzR0NDc0dBUVVGQndFQkJFOHdUVE123mdnckJnRUYKQlFjd0FvWS9hSFIwY0RvdkwzQnJhUzVwYm1acGJtVnZiaTVqYjIwdlQzQjBhV2RoVW5OaFRXWnlRMEV3TURjdgpUM0IwYVdkaFVuTmhUV1p5UTBFd01EY3VZM0owTUE0R0ExVWREd0VCL3dRRUF3SUFJREJZQmdOVkhSRUJBZjhFClRqQk1wRW93U0RFV01CUUdCV2VCQlFJQkRBdHBaRG8wT1RRMk5UZ3dNREVhTUJnR0JXZUJCUUlDREE5VFRFSWcKT1RZM01DQlVVRTB5TGpBeEVqQVFCZ1ZuZ1FVQ0F3d0hhV1E2TURjeU9EQU1CZ05WSFJNQkFmOEVBakFBTUZBRwpBMVVkSHdSSk1FY3dSYUJEb0VHR1AyaDBkSEE2THk5d2Eya3VhVzVtYVc1bGIyNHVZMjl0TDA5d2RHbG5ZVkp6CllVMW1ja05CTURBM0wwOXdkR2xuWVZKellVMW1ja05CTURBM0xtTnliREFWQmdOVkhTQUVEakFNTUFvR0NDcUMKRkFCRUFSUUJNQjhHQTFVZEl3UVlNQmFBRkp4OTlha2NQVW03NXplTlNyb1MvNDU0b3RkY01CQUdBMVVkSlFRSgpNQWNHQldlQkJ123JNQ0VHQTFVZENRUWFNQmd3RmdZRlo0RUZBaEF4RFRBTERBTXlMakFDQVFBQ0FYUXdEUVlKCktvWklodmNOQVFFTEJRQURnZ0VCQUJBREEzdEhsNnB4Z3FhR2lFYVNiZ3QwVER6V3QxVThKODZWcG9RbExoZUQKc3BWMmk5TWRoQlUrMmJXVkJDMU8rWk5mTVRkQXVzbGRTWmtDdUpjbEgrclQ5Y3dEMmI1UW0zbEVhc2c3dmdjaworNkNoZmM1Y3N5cGZrYnFBM0plT3RUa3QxM0lVTERqeU5lUUsrWkxXRmlHMnZrLzRPMUhhMXJOWExkV2lHNWQrClN0M2NiWUdUUThJeTJnenNCbmI0ODhwNWVzMUViNEhnNGtWQnJZRDkyTWhJZTdqb2x5VnFKWm5td3Ztd1hNSFgKVTU0azZYUEdlUjRuYmRJc1hlY3Q4bzZBZGxQeVF5dTFPdlUvcWx3RVVhOXJ4QkRXM3RjZHBMV1VZUTFMZE4ySAprc3BVTnZnTGZkVkZCc2tiMmtrRVRONjZDemY1M3VTMmpoc24rM3NIeE93PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==",
    "comment": "Sample EC registration",
    "hardware_uuid": "12912393-1231-1231-123e-12363512363b",
    "issuer": "C = DE, O = Sample, OU = Sample TPM2.0, CN = Sample",
    "revoked": false
}
```



