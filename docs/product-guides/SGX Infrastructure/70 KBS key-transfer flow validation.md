# KBS key-transfer flow validation

On SGX compute node, Execute below commands for KBS key-transfer:


???+ note 
    Before initiating key transfer make sure, PYKMIP server is running.

```
    pkill nginx
```

Remove any existing pkcs11 token

```
    rm -rf /opt/intel/cryptoapitoolkit/tokens/*
```

Initiate Key tranfer from KBS

```
    systemctl restart nginx
```

Changing group ownership and permissions of pkcs11 token
```
    chown -R root:intel /opt/intel/cryptoapitoolkit/tokens/
    chmod -R 770 /opt/intel/cryptoapitoolkit/tokens/
```

Establish tls session with the nginx using the key transferred inside the enclave

```
    wget https://<Control Plane IP>:32443 --no-check-certificate
```
