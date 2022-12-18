# Note on SKC Library Deployment

SKC Library Deployment needs to performed with root privilege

The SKC Client Library TLS client certificate private key is stored in the configuration directories and can be read only with elevated root privileges
keys.txt (set of PKCS11 URIs for the keys to be securely provisioned into an SGX enclave) can only be modified with elevated privileges

## Extracting SGX Enclave values for Key Transfer Policy

Values that are specific to the enclave such as sgx_enclave_issuer_anyof, sgx_enclave_measurement_anyof and sgx_enclave_issuer_product_id_anyof can be retrived using `sgx_sign` utility that is available as part of Intel SGX SDK.

Run `sgx_sign` utility on the signed enclave (This command should be run on the build system).
```
    /opt/intel/sgxsdk/bin/x64/sgx_sign dump -enclave <path to the signed enclave> -dumpfile info.txt
```

- For `sgx_enclave_issuer_anyof`, in info.txt, search for "mrsigner->value" . E.g mrsigner->value :
  ```
  mrsigner->value: "0x83 0xd7 0x19 0xe7 0x7d 0xea 0xca 0x14 0x70 0xf6 0xba 0xf6 0x2a 0x4d 0x77 0x43 0x03 0xc8 0x99 0xdb 0x69 0x02 0x0f 0x9c 0x70 0xee 0x1d 0xfc 0x08 0xc7 0xce 0x9e"
  ```
  Remove the whitespace and 0x characters from the above string and add it to the policy file. E.g :
  ```
  "sgx_enclave_issuer_anyof":["83d719e77deaca1470f6baf62a4d774303c899db69020f9c70ee1dfc08c7ce9e"]
  ```
- For `sgx_enclave_measurement_anyof`, in info.txt, search for metadata->enclave_css.body.enclave_hash.m . E.g metadata->enclave_css.body.enclave_hash.m :
  ```
  metadata->enclave_css.body.enclave_hash.m:
  0xad 0x46 0x74 0x9e 0xd4 0x1e 0xba 0xa2 0x32 0x72 0x52 0x04 0x1e 0xe7 0x46 0xd3
  0x79 0x1a 0x9f 0x24 0x31 0x83 0x0f 0xee 0x08 0x83 0xf7 0x99 0x3c 0xaf 0x31 0x6a
  ```
  Remove the whitespace and 0x characters from the above string and add it to the policy file. E.g :
  ```
  "sgx_enclave_measurement_anyof":["ad46749ed41ebaa2327252041ee746d3791a9f2431830fee0883f7993caf316a"]
  ```
Please note that the SGX Enclave measurement value will depend on the toolchain used to build and link the SGX enclave. Hence the SGX Enclave measurement value would differ across OS flavours.
For more details please refer https://github.com/intel/linux-sgx/tree/master/linux/reproducibility
