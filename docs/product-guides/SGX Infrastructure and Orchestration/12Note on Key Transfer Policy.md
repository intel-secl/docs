# Note on Key Transfer Policy

Key transfer policy is used to enforce a set of policies which need to be compiled with before the secret can be securely provisioned onto a sgx enclave

A typical Key Transfer Policy would look as below
```
        {
                "attestation_type": ["SGX"],
                "sgx": {
                        "attributes": {
                                "mrsigner": ["cd171c56941c6ce49690b455f691d9c8a04c2e43e0a4d30f752fa5285c7ee57f"],
                                "isvprodid": [0],
                                "mrenclave": ["da2923001b1ca93a8f95fad258cf883eeb3b4c798e788851515b04e656bae19a"],
                                "isvsvn": 1,
                                "client_permissions": ["nginx", "USA"],
                                "enforce_tcb_upto_date": false
                                }
                },
                "cert_issuer": ["CMS TLS Client CA"]
        }
```

**mrenclave** establishes the signing identity provided by an authority who has signed the sgx enclave. in other words the owner of the enclave

**mrsigner** represents the cryptographic hash of the enclave log (enclave code, data)

**enforce_tcb_upto_date** - If set to true, Key Broker service will provision the key only of the platform generating the quote conforms to the latest Trusted Computing Base

**client_permissions** - Special permission embedded into the skc_library client TLS certificate which can enforce additional restrictons on who can get access to the key,
    In above example: the key is provisioned only to the nginx workload and platform which is tagged with value for ex: USA
