# Default Service and Agent Mount Paths

## Single Node Deployments

Single node Deployments use `hostPath` mounting pod(container) files directly on host. Following is the complete list of the files being mounted on host

```yaml
#Certificate-Management-Service
Config: /etc/cms
Logs: /var/log/cms

#Authentication Authorization Service
Config: /etc/authservice
Logs: /var/log/authservice
Pg-data: /usr/local/kube/data/authservice/pgdata

#SGX Host Attestation Service
Config: /etc/shvs
Logs: /var/log/shvs
Pg-data: /usr/local/kube/data/shvs

#SGX Caching Service
Config: /etc/scs
Logs: /var/log/scs
Pg-data: /usr/local/kube/data/scs

#Integration-Hub
Config: /etc/ihub
Log: /var/log/ihub

#SGX Quote Verificaton Service
Config: /etc/sqvs
Logs: /var/log/sqvs

#Key-Broker-Service
Config: /etc/kbs
Log: /var/log/kbs
Opt: /opt/kbs

#SGX Library:
Config: /root/lib/resources/sgx_default_qcnl.conf
openssl-config: /etc/pki/tls/openssl.cnf
pkcs11-config: /opt/skc/etc/pkcs11-apimodule.ini
kms-npm-config: /opt/skc/etc/kms_npm.ini
sgx-stm-config: /opt/skc/etc/sgx_stm.ini
kms-cert: /root/9e9db4b5-5893-40fe-b6c4-d54ec6609c55.crt
haproxy-hosts: /etc/hosts
nginx-config: /etc/nginx/nginx.conf
skc-lib-config: /root/lib/skc_library.conf
kms-key: /tmp/keys.txt
nginx-logs: /var/log/sqvs/

#SGX Agent:
Config: /etc/sgx-agent
Logs: /var/log/sgx-agent
EFI: /sys/firmware/efi/efivars
```

## Multi Node Deployments

Multi node Deployments use k8s persistent volume and persistent volume claims for mounting pod(container) files on NFS volumes for all services, agents will continue to use `hostPath`. Following is a sample list of the files being mounted on NFS base volumes

```yaml
#Certificate-Management-Service
Config: <NFS-vol-base-path>/isecl/cms/config
Logs: <NFS-vol-base-path>/isecl/cms/logs

#Authentication Authorization Service
Config: <NFS-vol-base-path>/isecl/aas/config
Logs: <NFS-vol-base-path>/isecl/aas/logs
Pg-data: <NFS-vol-base-path>/isecl/aas/db

#SGX Host Attestation Service
Config: <NFS-vol-base-path>/isecl/shvs/config
Logs: <NFS-vol-base-path>/isecl/shvs/logs
Pg-data: <NFS-vol-base-path>/usr/local/kube/data/shvs

#Integration-Hub
Config: <NFS-vol-base-path>/isecl/ihub/config
Log: <NFS-vol-base-path>/isecl/ihub/logs

#SGX Quote Verificaton Service
Config: <NFS-vol-base-path>/isecl/sqvs/config
Logs: <NFS-vol-base-path>/isecl/sqvs/log

#Key-Broker-Service
Config: <NFS-vol-base-path>/isecl/kbs/config
Log: <NFS-vol-base-path>/isecl/kbs/logs
Opt: <NFS-vol-base-path>/isecl/kbs/kbs/opt

#SGX Agent:
Config: /etc/sgx-agent
Logs: /var/log/sgx-agent
EFI: /sys/firmware/efi/efivars
```
