# Flavor Management
===================

## Flavor Format Definitions
-------------------------

A Flavor is a standardized set of expectations that determines what
platform measurements will be considered “trusted.” Flavors are
constructed in a specific format, containing a metadata section
describing the Flavor, and then various other sections depending on the
Flavor type or Flavor part.

### Meta

The first part of a Flavor is the `meta` section:

```json
"meta":{
    "vendor": "INTEL",
    "description": {
        "flavor_part": "PLATFORM",
        "bios_name": "Intel Corporation",
        "bios_version": "SE5C620.86B.00.01.0004.071220170215",
        "tpm_version": "2.0"
    }
}
```

This section defines the Flavor part and any versioning information.

???+ note 
    Even when the BIOS or OS version remains the same, the actual measurements in the measured boot process will be different between TPM 1.2 and TPM 2.0, and so the TPM version is captured here as well. The attributes in the Meta section are used by the Flavor matching engine when matching Flavors to Hosts.  Note that TPM 1.2 is supported only for VMware ESXi hosts.

### Hardware

The `hardware` section is unique to PLATFORM flavor parts:

```json
"hardware": {
    "processor_info": "54 06 05 00 FF FB EB BF",
    "processor_flags": "fpu vme de …",
    "feature": {
        "tpm": {
            "enabled": true,
            "pcr_banks": [
                "SHA1",
                "SHA256"
            ]
        },
        "txt": {
            "enabled": true
        }
    }
}
```

This part of the Flavor defines expected hardware attributes of the
host, and contains processor and TPM-related attributes.

### PCR banks (Algorithms)

TPMs can have one or more PCR banks enabled with different hash algorithms.  Intel SecL will always attempt to use the most secure algorithm available in the enabled PCR banks.

For example, if a given TPM has the following PCR banks enabled:

```
SHA1
SHA256
SHA384
```

The HVS will prefer the SHA384 PCR bank when creating flavors and performing attestations.  

The TPM vendor and version, platform OEM, and BIOS version and configuration each impact which PCR banks can potentially be enabled.  Some manufacturers will allow users to configure which banks are enabled/disabled in the BIOS.  Other manufacturers will enable only one PCR bank, and others will be permanently disabled.

Flavors will only utilize a single PCR bank, and when importing from a sample host the HVS will always prefer the strongest algorithm supported by the enabled TPM PCR banks.  In the above example, a flavor imported from that host would use the SHA384 bank for all hash values.  This means that all hosts that will be attested using this flavor must also have SHA384 banks enabled in their TPMs.

Typically, among otherwise-identical servers this will not be an issue.  However, in a mixed environment it can be possible to have an OS flavor, for example, that needs to apply for some hosts that have SHA384 banks enabled, and other servers that only have SHA256 enabled and do not support SHA384.

In this circumstance, multiple flavors for the same OS version would need to be created - one for SHA384, and another for SHA256.



### PCRs

The last section of a Flavor is the “PCRs” section, which contains the
actual expected measurements for any PCRs. This section will contain PCR
measurements for each applicable algorithm supported by the TPM (SHA1
only for TPM 1.2, SHA256 and SHA1 sections for TPM 2.0).

Some PCRs simply have a value and nothing else. Other PCRs, however,
contain different `event` measurements. This indicates that separate
individual platform or OS components are independently measured and
extended to the same PCR. PCRs with event measurements will contain an
`Event` array that lists, in the correct order, all of the events in the
measurement event log that are extended to this PCR. When the
Verification Service attests a host against a given Flavor, each
measurement event is compared to the Flavor value, and all of the events
are replayed to confirm that a replay of all of the measurement
extensions do in fact result in the hash seen in the PCR value. In this
way, the Verification Service can ensure that the measurement event log
contents are secure, and the individual measurements can be attested so
that the cause for an Untrusted attestation can easily be seen.

The full PCRs section is not shown here due to length; see the sample
Flavor sections for a full sample.

```JSON
"pcrs": {
    "SHA1": {
        "pcr_0": {
            "value": "d2ed125942726641a7260c4f92beb67d531a0def"
        },
        "pcr_17": {
            "value": "1ec12004b371e3afd43d04155abde7476a3794fa",
            "event": [
                {
                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                    "value": "2fb7d57dcc5455af9ac08d82bdf315dbcc59a044",
                    "label": "HASH_START",
                    "info": {
                        "ComponentName": "HASH_START",
                        "EventName": "OpenSource.EventName"
                    }
                },
                ...
```

### Sample PLATFORM Flavor

The `PLATFORM` Flavor part encompasses measurements that are unique to a
specific platform, including the server OEM, BIOS version, etc. A
PLATFORM Flavor can be `shared` across all hosts of the same model that
have the same BIOS version.

```json
{
    "flavor_collection": {
        "flavors": [
            {
                "meta": {
                    "vendor": "INTEL",
                    "description": {
                        "flavor_part": " PLATFORM",
                        "bios_name": "Intel Corporation",
                        "bios_version": "SE5C620.86B.00.01.0004.071220170215",
                        "tpm_version": "2.0"
                    }
                },
                "hardware": {
                    "processor_info": "54 06 05 00 FF FB EB BF",
                    "processor_flags": "fpu vme de …",
                    "feature": {
                        "tpm": {
                            "enabled": true,
                            "pcr_banks": [
                                "SHA1",
                                "SHA256"
                            ]
                        },
                        "txt": {
                            "enabled": true
                        }
                    }
                },
                "pcrs": {
                    "SHA1": {
                        "pcr_0": {
                            "value": "d2ed125942726641a7260c4f92beb67d531a0def"
                        },
                        "pcr_17": {
                            "value": "1ec12004b371e3afd43d04155abde7476a3794fa",
                            "event": [
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "2fb7d57dcc5455af9ac08d82bdf315dbcc59a044",
                                    "label": "HASH_START",
                                    "info": {
                                        "ComponentName": "HASH_START",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "ffb1806465d2de1b7531fd5a2a6effaad7c5a047",
                                    "label": "BIOSAC_REG_DATA",
                                    "info": {
                                        "ComponentName": "BIOSAC_REG_DATA",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "3c585604e87f855973731fea83e21fab9392d2fc",
                                    "label": "CPU_SCRTM_STAT",
                                    "info": {
                                        "ComponentName": "CPU_SCRTM_STAT",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "9069ca78e7450a285173431b3e52c5c25299e473",
                                    "label": "LCP_CONTROL_HASH",
                                    "info": {
                                        "ComponentName": "LCP_CONTROL_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "5ba93c9db0cff93f52b521d7420e43f6eda2784f",
                                    "label": "LCP_DETAILS_HASH",
                                    "info": {
                                        "ComponentName": "LCP_DETAILS_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "5ba93c9db0cff93f52b521d7420e43f6eda2784f",
                                    "label": "STM_HASH",
                                    "info": {
                                        "ComponentName": "STM_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "3c585604e87f855973731fea83e21fab9392d2fc",
                                    "label": "OSSINITDATA_CAP_HASH",
                                    "info": {
                                        "ComponentName": "OSSINITDATA_CAP_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "3d42560dcf165a5557b3156a21583f2c6dbef10e",
                                    "label": "MLE_HASH",
                                    "info": {
                                        "ComponentName": "MLE_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "274f929dbab8b98a7031bbcd9ea5613c2a28e5e6",
                                    "label": "NV_INFO_HASH",
                                    "info": {
                                        "ComponentName": "NV_INFO_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "ca96de412b4e8c062e570d3013d2fccb4b20250a",
                                    "label": "tb_policy",
                                    "info": {
                                        "ComponentName": "tb_policy",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "d123e2f2b30f1effa8d9522f667af0dac4f48cfb",
                                    "label": "vmlinuz",
                                    "info": {
                                        "ComponentName": "vmlinuz",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "f3742133e1a0deb48177a74ed225418e5cf73fd1",
                                    "label": "initrd",
                                    "info": {
                                        "ComponentName": "initrd",
                                        "EventName": "OpenSource.EventName"
                                    }
                                }
                            ]
                        }
                    },
                    "SHA256": {
                        "pcr_0": {
                            "value": "db83f0e8a1773c21164c17986037cdf8afc1bbdc1b815772c6da1befb1a7f8a3"
                        },
                        "pcr_17": {
                            "value": "50bd58407a1893056eacff493245cfe785f045b2c0e1cc3e6e9eb5812d8d91bd",
                            "event": [
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "9301981c093654d5aa3430ba05c880a52eb22b9e18248f5f93e1fe1dab1cb947",
                                    "label": "HASH_START",
                                    "info": {
                                        "ComponentName": "HASH_START",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "2785d1ed65f6b5d4b555dc24ce5e068a44ce8740fe77e01e15a10b1ff66cca90",
                                    "label": "BIOSAC_REG_DATA",
                                    "info": {
                                        "ComponentName": "BIOSAC_REG_DATA",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "67abdd721024f0ff4e0b3f4c2fc13bc5bad42d0b7851d456d88d203d15aaa450",
                                    "label": "CPU_SCRTM_STAT",
                                    "info": {
                                        "ComponentName": "CPU_SCRTM_STAT",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "df3f619804a92fdb4057192dc43dd748ea778adc52bc498ce80524c014b81119",
                                    "label": "LCP_CONTROL_HASH",
                                    "info": {
                                        "ComponentName": "LCP_CONTROL_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                }
                            ]
                        }
                    }
                }
```

### Sample OS Flavor

An OS Flavor encompasses all of the measurements unique to a given OS.
This includes the OS kernel and other measurements.

```json
{
    "flavor_collection": {
        "flavors": [
            {
                "meta": {
                    "vendor": "INTEL",
                    "description": {
                        "flavor_part": "OS",
                        "os_name": "RedHatEnterpriseServer",
                        "os_version": "7.3",
                        "vmm_name": "",
                        "vmm_version": "",
                        "tpm_version": "2.0"
                    }
                },
                "pcrs": {
                    "SHA1": {
                        "pcr_17": {
                            "value": "1ec12004b371e3afd43d04155abde7476a3794fa",
                            "event": [
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "2fb7d57dcc5455af9ac08d82bdf315dbcc59a044",
                                    "label": "HASH_START",
                                    "info": {
                                        "ComponentName": "HASH_START",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "ffb1806465d2de1b7531fd5a2a6effaad7c5a047",
                                    "label": "BIOSAC_REG_DATA",
                                    "info": {
                                        "ComponentName": "BIOSAC_REG_DATA",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "3c585604e87f855973731fea83e21fab9392d2fc",
                                    "label": "CPU_SCRTM_STAT",
                                    "info": {
                                        "ComponentName": "CPU_SCRTM_STAT",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "9069ca78e7450a285173431b3e52c5c25299e473",
                                    "label": "LCP_CONTROL_HASH",
                                    "info": {
                                        "ComponentName": "LCP_CONTROL_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "5ba93c9db0cff93f52b521d7420e43f6eda2784f",
                                    "label": "LCP_DETAILS_HASH",
                                    "info": {
                                        "ComponentName": "LCP_DETAILS_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "5ba93c9db0cff93f52b521d7420e43f6eda2784f",
                                    "label": "STM_HASH",
                                    "info": {
                                        "ComponentName": "STM_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "3c585604e87f855973731fea83e21fab9392d2fc",
                                    "label": "OSSINITDATA_CAP_HASH",
                                    "info": {
                                        "ComponentName": "OSSINITDATA_CAP_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "3d42560dcf165a5557b3156a21583f2c6dbef10e",
                                    "label": "MLE_HASH",
                                    "info": {
                                        "ComponentName": "MLE_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "274f929dbab8b98a7031bbcd9ea5613c2a28e5e6",
                                    "label": "NV_INFO_HASH",
                                    "info": {
                                        "ComponentName": "NV_INFO_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "ca96de412b4e8c062e570d3013d2fccb4b20250a",
                                    "label": "tb_policy",
                                    "info": {
                                        "ComponentName": "tb_policy",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "d123e2f2b30f1effa8d9522f667af0dac4f48cfb",
                                    "label": "vmlinuz",
                                    "info": {
                                        "ComponentName": "vmlinuz",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                    "value": "f3742133e1a0deb48177a74ed225418e5cf73fd1",
                                    "label": "initrd",
                                    "info": {
                                        "ComponentName": "initrd",
                                        "EventName": "OpenSource.EventName"
                                    }
                                }
                            ]
                        }
                    },
                    "SHA256": {
                        "pcr_17": {
                            "value": "50bd58407a1893056eacff493245cfe785f045b2c0e1cc3e6e9eb5812d8d91bd",
                            "event": [
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "9301981c093654d5aa3430ba05c880a52eb22b9e18248f5f93e1fe1dab1cb947",
                                    "label": "HASH_START",
                                    "info": {
                                        "ComponentName": "HASH_START",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "2785d1ed65f6b5d4b555dc24ce5e068a44ce8740fe77e01e15a10b1ff66cca90",
                                    "label": "BIOSAC_REG_DATA",
                                    "info": {
                                        "ComponentName": "BIOSAC_REG_DATA",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "67abdd721024f0ff4e0b3f4c2fc13bc5bad42d0b7851d456d88d203d15aaa450",
                                    "label": "CPU_SCRTM_STAT",
                                    "info": {
                                        "ComponentName": "CPU_SCRTM_STAT",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "df3f619804a92fdb4057192dc43dd748ea778adc52bc498ce80524c014b81119",
                                    "label": "LCP_CONTROL_HASH",
                                    "info": {
                                        "ComponentName": "LCP_CONTROL_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "6e340b9cffb37a989ca544e6bb780a2c78901d3fb33738768511a30617afa01d",
                                    "label": "LCP_DETAILS_HASH",
                                    "info": {
                                        "ComponentName": "LCP_DETAILS_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "6e340b9cffb37a989ca544e6bb780a2c78901d3fb33738768511a30617afa01d",
                                    "label": "STM_HASH",
                                    "info": {
                                        "ComponentName": "STM_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "67abdd721024f0ff4e0b3f4c2fc13bc5bad42d0b7851d456d88d203d15aaa450",
                                    "label": "OSSINITDATA_CAP_HASH",
                                    "info": {
                                        "ComponentName": "OSSINITDATA_CAP_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "26e1d98742f79c950dc637f8c067b0b72a1b0e8ff75db4e609c7e17321acf3f4",
                                    "label": "MLE_HASH",
                                    "info": {
                                        "ComponentName": "MLE_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "0f6e0c7a5944963d7081ea494ddff1e9afa689e148e39f684db06578869ea38b",
                                    "label": "NV_INFO_HASH",
                                    "info": {
                                        "ComponentName": "NV_INFO_HASH",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "27808f64e6383982cd3bcc10cfcb3457c0b65f465f779d89b668839eaf263a67",
                                    "label": "tb_policy",
                                    "info": {
                                        "ComponentName": "tb_policy",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "c89ad1d1e9adaa7ecfee2abce763b92472685f7d1b9f3799bf49974b66ed9638",
                                    "label": "vmlinuz",
                                    "info": {
                                        "ComponentName": "vmlinuz",
                                        "EventName": "OpenSource.EventName"
                                    }
                                },
                                {
                                    "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                    "value": "81b88e268e697ccf1790d41b9de748a8f395acfb47aa67c9845479d4e8456f77",
                                    "label": "initrd",
                                    "info": {
                                        "ComponentName": "initrd",
                                        "EventName": "OpenSource.EventName"
                                    }
                                }
                            ]
                        }
                    }
                }
            }
        ]
    },
    "flavorgroup_name": "automatic"
}
```

### Sample HOST\_UNIQUE Flavor

Host-Unique flavors define measurements for a specific host. This can be
either a single large flavor that incorporates all of the host
measurements into a single flavor document used only to attest a single
host, or can be a small subset of measurements that are specific to a
single host. For example, some VMWare module measurements will change
from one host to the next, while most others will be shared assuming the
same ESXi build is used. The full Flavor requirement for such a host
would include Host-Unique flavors to cover the measurements that are
unique to only this one host, and would still use a generic PLATFORM and
OS flavor for the other measurements that would be identical for other
similarly configured hosts.

???+ note 
    The HOST\_UNIQUE Flavors are unique to a specific host, and should always be imported directly from the specific host.

```json
{
    "flavors": [
        {
            "meta": {
                "id": "4d387cbd-f72b-4742-b4e5-c5b0ffed59e0",
                "vendor": "INTEL",
                "description": {
                    "flavor_part": "HOST_UNIQUE",
                    "source": "Purley11",
                    "bios_name": "Intel Corporation",
                    "bios_version": "SE5C620.86B.00.01.0004.071220170215",
                    "os_name": "RedHatEnterpriseServer",
                    "os_version": "7.4",
                    "tpm_version": "2.0",
                    "hardware_uuid": "00448C61-46F2-E711-906E-001560A04062"
                }
            },
            "pcrs": {
                "SHA256": {
                    "pcr_17": {
                        "value": "f9ef8c53ddfc8096d36eda5506436c52b4bfa2bd451a89aaa102f03181722176",
                        "event": [
                            {
                                "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                "value": "df3f619804a92fdb4057192dc43dd748ea778adc52bc498ce80524c014b81119",
                                "label": "LCP_CONTROL_HASH",
                                "info": {
                                    "ComponentName": "LCP_CONTROL_HASH",
                                    "EventName": "OpenSource.EventName"
                                }
                            },
                            {
                                "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                "value": "09f468dfc1d98a1fee86eb7297a56b0e097d57be66db4eae539061332da2e723",
                                "label": "initrd",
                                "info": {
                                    "ComponentName": "initrd",
                                    "EventName": "OpenSource.EventName"
                                }
                            }
                        ]
                    },
                    "pcr_18": {
                        "value": "c1f7bfdae5f270d9f13aa9620b8977951d6b759f1131fe9f9289317f3a56efa1",
                        "event": [
                            {
                                "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha256",
                                "value": "df3f619804a92fdb4057192dc43dd748ea778adc52bc498ce80524c014b81119",
                                "label": "LCP_CONTROL_HASH",
                                "info": {
                                    "ComponentName": "LCP_CONTROL_HASH",
                                    "EventName": "OpenSource.EventName"
                                }
                            }
                        ]
                    }
                },
                "SHA1": {
                    "pcr_17": {
                        "value": "48695f747a3d494710bd14d20cb0a93c78a485cc",
                        "event": [
                            {
                                "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                "value": "9069ca78e7450a285173431b3e52c5c25299e473",
                                "label": "LCP_CONTROL_HASH",
                                "info": {
                                    "ComponentName": "LCP_CONTROL_HASH",
                                    "EventName": "OpenSource.EventName"
                                }
                            },
                            {
                                "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                "value": "b1f8db372e396bb128280821b7e0ac54a5ec2791",
                                "label": "initrd",
                                "info": {
                                    "ComponentName": "initrd",
                                    "EventName": "OpenSource.EventName"
                                }
                            }
                        ]
                    },
                    "pcr_18": {
                        "value": "983ec7db975ed31e2c85ef8e375c038d6d307efb",
                        "event": [
                            {
                                "digest_type": "com.intel.mtwilson.lib.common.model.MeasurementSha1",
                                "value": "9069ca78e7450a285173431b3e52c5c25299e473",
                                "label": "LCP_CONTROL_HASH",
                                "info": {
                                    "ComponentName": "LCP_CONTROL_HASH",
                                    "EventName": "OpenSource.EventName"
                                }
                            }
                        ]
                    }
                }
            }
        }
    ]
}
```

### Sample ASSET\_TAG Flavor

Asset Tag flavor parts are unique to Asset Tag attestation. These
flavors verify that the Asset Tag data in the host’s TPM correctly
matches the most recently created, currently valid Asset Tag certificate
that has been deployed to that host.

```json
{
    "meta": {
        "id": "b3e0c056-5b6c-4b6b-95c4-de5f1473cac0",
        "description": {
            "flavor_part": "ASSET_TAG",
            "hardware_uuid": "<Hardware UUID of the server to be tagged>"
        }
    },
    "external": {
        "asset_tag": {
            "tag_certificate": {
                "encoded": "<Tag certificate in base64 encoded format>",
                "issuer": "CN=assetTagService",
                "serial_number": 1519153541461,
                "subject": "<Hardware UUID of the server to be tagged>",
                "not_before": "2018-02-20T11:05:41-0800",
                "not_after": "2019-02-20T11:05:41-0800",
                "fingerprint_sha384": "46001d8472e56de423aac7c55f061404d27d50e497dfc21a861ef1965d7ac1e44887aee918fb5805385a3cbdf820899d",
                "attribute": [
                    {
                        "attr_type": {
                            "id": "2.5.4.789.2"
                        },
                        "attribute_values": [
                            {
                                "objects": {}
                            }
                        ]
                    },
                    {
                        "attr_type": {
                            "id": "2.5.4.789.2"
                        },
                        "attribute_values": [
                            {
                                "objects": {}
                            }
                        ]
                    },
                    {
                        "attr_type": {
                            "id": "2.5.4.789.2"
                        },
                        "attribute_values": [
                            {
                                "objects": {}
                            }
                        ]
                    }
                ]
            }
        }
    }
}
```



## Flavor Templates

Added in the Intel SecL-DC 4.0 release, Flavor Templates expose the backend logic that determines which PCRs and event log measurements will be used for specific Flavor parts.  Where previously these rules were hardcoded on the backend, this new feature allows new templates to be added, and allows the customization or deletion of existing templates to suit specific business needs.

Flavor Templates are conditional rules that apply to a Flavor part cumulatively based on defined conditions.  For example a PLATFORM Flavor for a Linux host with Intel TXT enabled would normally include PCR0. If tboot is also enabled, elements from PCR 17 and 18 will be added to the PLATFORM flavor.  These are cumulative based on which conditions are true on a given host.

By default, Flavor Templates will come pre-populated in the HVS database to meet the same default behavior for previous releases.  

Flavor Templates can be added, removed, or edited to create customized rules.  For example, if there is a specific event log measurement that a user would like to add to an OS flavor, a new Flavor Template can be added for the OS Flavor part that defines a condition for applying the Template, along with the specific event log measurement that should be used when that condition is satisfied.

Flavor Templates are cumulative.  If a given host matches all of the conditions defined for Flavor Template A and Flavor Template B, both Templates will be applied when importing Flavors from that host.

### Sample Flavor Template

Below is a sample default Flavor Template used for RedHat Enterprise Linux servers with TPM2.0 and tboot enabled:

```json
{
   "id":"8798fb0c-2dfa-4464-8281-e650a30da7e6",
   "label":"default-linux-rhel-tpm20-tboot",
   "condition":[
      "//host_info/os_name//*[text()='RedHatEnterprise']",
      "//host_info/hardware_features/TPM/meta/tpm_version//*[text()='2.0']",
      "//host_info/tboot_installed//*[text()='true']"
   ],
   "flavor_parts":{
      "OS":{
         "meta":{
            "tpm_version":"2.0",
            "tboot_installed":true
         },
         "pcr_rules":[
            {
               "pcr":{
                  "bank":[
                     "SHA384",
                     "SHA256",
                     "SHA1"
                  ],
                  "index":17
               },
               "eventlog_includes":[
                  "vmlinuz"
               ]
            }
         ]
      },
      "PLATFORM":{
         "meta":{
            "tpm_version":"2.0",
            "tboot_installed":true
         },
         "pcr_rules":[
            {
               "pcr":{
                  "bank":[
                     "SHA384",
                     "SHA256",
                     "SHA1"
                  ],
                  "index":0
               },
               "pcr_matches":true
            },
            {
               "pcr":{
                  "bank":[
                     "SHA384",
                     "SHA256",
                     "SHA1"
                  ],
                  "index":17
               },
               "eventlog_equals":{
                  "excluding_tags":[
                     "LCP_CONTROL_HASH",
                     "initrd",
                     "vmlinuz"
                  ]
               }
            },
            {
               "pcr":{
                  "bank":[
                     "SHA384",
                     "SHA256",
                     "SHA1"
                  ],
                  "index":18
               },
               "eventlog_equals":{
                  "excluding_tags":[
                     "LCP_CONTROL_HASH",
                     "initrd",
                     "vmlinuz"
                  ]
               }
            }
         ]
      },
      "HOST_UNIQUE":{
         "meta":{
            "tpm_version":"2.0",
            "tboot_installed":true
         },
         "pcr_rules":[
            {
               "pcr":{
                  "bank":[
                     "SHA384",
                     "SHA256",
                     "SHA1"
                  ],
                  "index":17
               },
               "eventlog_includes":[
                  "LCP_CONTROL_HASH",
                  "initrd"
               ]
            },
            {
               "pcr":{
                  "bank":[
                     "SHA384",
                     "SHA256",
                     "SHA1"
                  ],
                  "index":18
               },
               "eventlog_includes":[
                  "LCP_CONTROL_HASH"
               ]
            }
         ]
      }
   }
}
```

### Flavor Template Definitions

A Flavor Template consists of several sections:

The "id" and "label" keys are unique identifiers.  The ID is generated automatically by the HVS when the Template is created; the label is user-specified and must be unique.

#### Conditions

The "condition" section contains a map of host-info elements to match when deciding to apply the Template.  For example:

```json
   "condition":[
      "//host_info/os_name//*[text()='RedHatEnterprise']",
      "//host_info/hardware_features/TPM/meta/tpm_version//*[text()='2.0']",
      "//host_info/tboot_installed//*[text()='true']"
   ]
```

This sample contains three conditions, each of which must be true for this Template to apply:

os_name: 'RedHatEnterprise'

tpm_version: 2.0

tboot_installed: true

This will apply for RedHat hosts with TPM2.0 and tboot enabled.  If a Flavor is imported from a VMware ESXi host, this template will not apply.  The condition paths directly refer to host-info elements collected from the host.  The full host-info details for a host can be viewed using the /hvs/v2/host-status API; below is a snippet of the host-info section (note that additional host-info elements may be added as new platform features are incorporated):

```json
               "host_info": {
                    "os_name": "RedHatEnterprise",
                    "os_version": "8.4",
                    "os_type": "linux",
                    "bios_version": "SE11111.111.11.11.1111.11111111111",
                    "vmm_name": "",
                    "vmm_version": "",
                    "processor_info": "54 06 05 00 FF FB EB BF",
                    "host_name": "hostname",
                    "bios_name": "Intel Corporation",
                    "hardware_uuid": "<UUID>",
                    "process_flags": "FPU VME DE PSE TSC MSR PAE MCE CX8 APIC SEP MTRR PGE MCA CMOV PAT PSE-36 CLFSH DS ACPI MMX FXSR SSE SSE2 SS HTT TM PBE",
                    "no_of_sockets": "72",
                    "tboot_installed": "false",
                    "is_docker_env": "false",
                    "hardware_features": {
                        "TXT": {
                            "enabled": "true"
                        },
                        "TPM": {
                            "enabled": "true",
                            "meta": {
                                "tpm_version": "2.0"
                            }
                        },
                        "CBNT": {
                            "enabled": "false",
                            "meta": {
                                "profile": "",
                                "msr": ""
                            }
                        },
                        "UEFI": {
                            "enabled": "false",
                            "meta": {
                                "secure_boot_enabled": true
                            }
                        }
                    },
                    "installed_components": [
                        "tagent"
                    ]
                }
```



#### Flavor Parts

This section of the template will define behaviors for each Flavor part.  Each different Flavor part is optional; if the new Template will only affect the OS Flavor part, only the OS Flavor part needs to be defined here.  Each Flavor part specified will have its own "meta" section where conditional host attributes will be defined.  These must match with host-info attributes; for example, in the sample above the OS part uses the following "meta" section elements:

```json
{
 "tpm_version":"2.0",
 "tboot_installed":true
}
```

These directly correspond to host-info elements from the hosts this Template will apply to.

#### PCR Rules

Each Flavor part section of a Flavor Template may contain 0 or more PCR rules that define PCRs to include.  Again using the OS Flavor part example from above, the default Template defines SHA1, SHA256, or SHA384 PCR banks; this tells the HVS to use the "best" available PCR bank algorithm, but that each of these algorithms is acceptable.  Alternatively, if the Template listed only the SHA384 PCR bank, the resulting Flavor would *require* the SHA384 PCR bank and would disregard any SHA256 or SHA1 banks, even if the SHA384 bank is unavailable and the SHA256 bank is enabled on the server.

The PCR Rules will also contain at least one PCR index, indicating which PCR the rule applies to.

##### pcr_matches

PCRs can require a direct PCR value match (when event logs are unnecessary and the final PCR hash is required to match a specific value), and/or can contain event log include/exclude/equals rules.

A direct PCR value match requirement is the easiest definitions, but should only be used when a specific PCR is known to always be the same on all hosts that the resulting Flavor will apply to:

```json
            {
               "pcr":{
                  "bank":[
                     "SHA384",
                     "SHA256",
                     "SHA1"
                  ],
                  "index":0
               },
               "pcr_matches":true
            }
```

This requires the value of PCR0 to exactly match, and will not examine specific event log details for this PCR index.

##### eventlog_includes

The following example shows how to require a specific event log entry to exist:

```json
            {
               "pcr":{
                  "bank":[
                     "SHA384",
                     "SHA256",
                     "SHA1"
                  ],
                  "index":17
               },
               "eventlog_includes":[
                  "vmlinuz"
               ]
            }
```

This rule will require the "vmlinuz" event log measurement to be present in the PCR17 event log.

##### excluding_tags

Specific event logs can also be excluded; in the below example, all events from PCR17 will be part of the resulting Flavor, but will exclude the LCP_CONTROL_HASH, initrd, and vmlinuz measurement events specifically.  This is often used when a specific PCR contains measurements that should apply to different Flavor parts; different rules need to be defined to ensure that the correct events are included in the right Flavor part, and events that will apply for different Flavor parts must be excluded.

```json
            {
               "pcr":{
                  "bank":[
                     "SHA384",
                     "SHA256",
                     "SHA1"
                  ],
                  "index":17
               },
               "eventlog_equals":{
                  "excluding_tags":[
                     "LCP_CONTROL_HASH",
                     "initrd",
                     "vmlinuz"
                  ]
               }
            }
```



## Flavor Matching
---------------

Flavors are matched to host by objects called `Flavor Groups` A Flavor
Group represents a set of rules to satisfy for a set of flavors to be
matched to a host for attestation. For example, a Flavor Group can
require that a `PLATFORM` Flavor and an `OS` Flavor be used for attestation.
Without this level of association, a host that matches measurements for
only a `PLATFORM` flavor, for example, can be attested as Trusted, even
though the OS Flavor would attest the host as Untrusted.

Flavor matching can be automatic (the default), or can explicitly
specify a host to which the Flavor Group must apply.

Automatic flavor matching allows for more ease in datacenter lifecycle
management with updates and patches that may cause the appropriate
flavors to change over time. Automatic flavor matching will trigger a
new matching action when a new flavor is added, when an existing flavor
is deleted, or when a host is initially attested as Untrusted. The
system will automatically attempt to find a new set of flavors that
match the Flavor Group rules that will attest the host as Trusted. For
example, if a host in your datacenter has recently had a BIOS update,
the next attestation will cause the host to appear Untrusted (because
the `PLATFORM` measurements will now differ). Using automatic flavor
matching, the Verification Service will automatically search for a new
`PLATFORM` flavor that matches the actual BIOS version and measurement
seen on the host. If a new BIOS version is successfully found, the
Verification Service will use the new version for attestation, and the
host will appear Trusted. If no matching `PLATFORM` flavor is found, the
host will appear Untrusted. When automatic flavor matching is used,
think of the various flavors in the Verification Service as a collection
of valid configurations, and an attested host matching any combination
of those configurations (within the confines of the Flavor Group
requirements for which flavor types must be present) will be attested as
Trusted.

Host-based flavor matching explicitly maps a specific host to a flavor.
Host-based attestation requires that a host saves its entire
configuration in a composite flavor document in the system, and then
later validates against this flavor to detect any changes. In this case,
if a host received a BIOS upgrade, the host will attest as Untrusted,
and no attempt will be made to re-match a new flavor. An administrator
will need to explicitly specify a new flavor to be used for that host.

### When Does Flavor Matching Happen?

Generally speaking, a new Flavor match operation is triggered whenever a
host is registered, whenever a host is attested and would be untrusted,
and whenever a Flavor is added to or removed from a Flavor group.

When a new host is registered, the Verification Service will retrieve
the Host Report and derive the platform information needed for Flavor
matching (BIOS version, server OEM, OS type and version, TPM version,
etc.). The Verification Service then searches through the Flavors in the
same Flavor group that the host is in, and finds any Flavors that match
the platform information.

If a Flavor is deleted, the Verification Service finds any hosts that
are currently associated with that Flavor, and attempts to match them to
alternative Flavors.

If a Flavor is added, the Verification Service looks for any hosts in
the same Flavor group that are not currently matched to a Flavor of the
appropriate Flavor part, and checks to see whether those hosts should be
mapped to the new Flavor.

If a new Report is generated for a host and would not result in a
Trusted attestation, the Verification Service will first repeat the
Flavor matching process to be sure that no matching Flavors exist in the
host’s Flavor group that would result in a Trusted attestation. If the
Service still finds no matching Flavors, the host will appear as
Untrusted.

### Flavor Matching Performance

Flavor matching causes affected hosts to be moved into the `QUEUE` state
while the host and Flavor are evaluated to determine whether the host
and Flavor should be linked. Hosts can remain in the QUEUE state for
varying amounts of time based on the extent of the Flavor match
required. This means that the trust status of a host will not be
actually updated to reflect a new Flavor until after the process
finishes, which may take a few seconds or minutes depending on the
number of registered hosts, Flavors in the same Flavorgroup, etc.

If a new host is registered, only that host will be added to the queue,
and other hosts will be unaffected. The Verification Service will look
for only the `HOST\_UNIQUE` flavor part applicable to that specific host,
and then will look at all PLATFORM and OS Flavors in the same
Flavorgroup has the host, using the Flavor metadata and host info to
narrow the results. The Service will match the new host to the most
similar Flavors, and then move the host to the `CONNECTED` state and
generate a new trust report.

When a new PLATFORM or OS Flavor is created, the Service will instead
add all hosts in the same Flavorgroup as the new Flavors to the queue.
Each host in the queue will then be re-evaluated against every `PLATFORM` and `OS` Flavor in the Flavorgroup to determine the closest match.

This means that adding a new Flavor can cause more hosts to each spend
more time in the QUEUE state, as compared to adding a new host. For this
reason, as a best practice for initial population of Flavors and hosts
for a new deployment, it is suggested that Flavors be created before
registering hosts. This is not a concern after the initial population of
Flavors and hosts.

### Flavor Groups

Flavor Groups represent a collection of one or more Flavors that are
possible matches for a collection of one or more hosts. Flavor Groups
link to both Flavors and hosts – a host in Flavor Group "ABC" will only
be matched to Flavors in Flavor Group "ABC"

### Default Flavor Group

By default the Verification Service includes a Flavor Group named
`automatic` and another named `unique` During host registration, the
`automatic` Flavor Group is used as a default selection if no other
Flavor Group is specified.

#### automatic

The automatic Flavor Group is used as the default Flavor Group for all
hosts and all Flavor parts. If no other Flavor Groups are specified when
creating Flavors or Hosts, all Hosts and Flavors will be added to this
group. This is useful for datacenters that want to manage a single set
of acceptable configurations for all hosts.

#### unique

The unique Flavor Group is used to contain `HOST\_UNIQUE` Flavors. This
Flavorgroup is used by the backend software and should not be managed
manually.

### Flavor Match Policies

Flavor Match Policies are used to define how the Flavor Match engine
will match Flavors to hosts for attestation for a given Flavor Group.
Each Flavor part can have defined Flavor Match Policies within a given
Flavor Group.

```json
{
    "PLATFORM": {
        "any_of",
        "required"
    },
    "OS": {
        "all_of",
        "required_if_defined"
    },
    "HOST_UNIQUE": {
        "latest",
        "required_if_defined"
    },
    "ASSET_TAG": {
        "latest",
        "required_if_defined"
    },
    "IMA": {
        "all_of",
        "required_if_defined"
    }
}
```

The sample Policy above would require that a `PLATFORM` Flavor part be
matched, but any `PLATFORM` Flavor part in the Flavor Group may be
matched. The `OS` Flavor Part will only be required if there is an `OS` Flavor part in the Flavor Group; if there are no `OS` Flavor parts in the
Group, the match will not be required. If more than one `OS` Flavor part
exists in the Group, all of those `OS` parts will be required to match for
a host to be Trusted.

#### Default Flavor Match Policy

The `automatic` Flavor Group, and any Flavor Group created without
explicitly defining a Flavor Match Policy, will be created using the
following Flavor Match Policy. This is the default behavior for Flavor
Matching:

```json
{
    "PLATFORM": {
        "any_of",
        "required"
    },
    "OS": {
        "any_of",
        "required"
    },
    "HOST_UNIQUE": {
        "latest",
        "required_if_defined"
    },
    "ASSET_TAG": {
        "latest",
        "required_if_defined"
    },
    "IMA": {
        "all_of",
        "required_if_defined"
    }
}
```

#### ANY\_OF

The `ANY_OF` Policy allows any Flavor of the specified Flavor part to be
matched. If the Flavor Group contains OS Flavor 1 and OS Flavor 2, a
host will be Trusted if it matches either OS Flavor 1 or OS Flavor 2.

#### ALL\_OF

The `ALL_OF` Policy requires all Flavors of the specified Flavor Part in
the Flavor Group to be matched. For example, if Flavor Group X contains
PLATFORM Flavor Part 1 and `PLATFORM` Flavor Part 2, a host in Flavor
Group X will need to match both `PLATFORM` Flavor 1 and `PLATFORM` Flavor 2
to attest as Trusted. If the host matches only one of the Flavors, or
neither of them, the host will be attested as Untrusted.

#### LATEST

The `LATEST` Policy requires that the most recently created Flavor of the
specified Flavor part be used when matching to a host. For example:

```json
"ASSET_TAG": {
    "latest",
    "required_if_defined"
}
```

`ASSET_TAG` Flavor parts by default use the above Policy. This means that
if Asset Tag Flavors are in the Flavor Group, the most recently created
Asset Tag Flavor will be used. If no Asset Tag Flavors are present in
the Flavor Group, then this Flavor part will be ignored.

#### REQUIRED

The `REQUIRED` Policy requires a Flavor of the specified part to be
matched. For example:

```json
"PLATFORM": {
    "any_of",
    "required"
}
```

This policy means that a `PLATFORM` Flavor part must be used; if the
Flavor Group contains no `PLATFORM` Flavor parts, hosts in this Flavor
Group will always count as Untrusted.

#### REQUIRED\_IF\_DEFINED

The `REQUIRED_IF_DEFINED` Policy requires that a Flavor part be used if
a Flavor of that part exists. If no Flavor part of this type exists in
the Flavor Group, the Flavor part will not be required.

```json
"ASSET_TAG": {
    "latest",
    "required_if_defined"
}
```

`ASSET_TAG` Flavor parts by default use the above Policy. This means that
if Asset Tag Flavors are in the Flavor Group, the most recently created
Asset Tag Flavor will be used. If no Asset Tag Flavors are present in
the Flavor Group, then this Flavor part will be ignored.

### Flavor Match Event Triggers

Several events will cause the background queue service to attempt to
re-match Flavors and hosts:

1.  Host registration

    This event is the first time a host will be attempted to be matched to
    appropriate Flavors in the same Flavor Group, and affects only the host
    that was added (other hosts will not be re-matched to Flavors when you
    add a new host).

2.  Flavor creation

    When a new Flavor is added to a Flavor Group, the queue system will
    repeat the Flavor match operation for all hosts in the same Flavor Group
    as the new Flavor.

3.  Flavor deletion

    When a Flavor is deleted, the queue system will repeat the Flavor match
    operation for all hosts in the same Flavor Group as the deleted Flavor.

4.  Creation of a new Attestation Report

    When a new Attestation Report is generated, if the host would attest as
    Untrusted with the currently-matched Flavors, the host being attested
    will be re-matched as part of the Report generation process. This
    ensures that Reports are always generated using the best possible Flavor
    matches available in the database.

### Sample Flavorgroup API Calls

#### Create a New Flavorgroup

```
POST https://<Verification Service IP or Hostname>:8443/hvs/v2/flavorgroups
Authorization: Bearer <token>

{
    "name": "firstTest",
    "flavor_match_policy_collection": {
        "flavor_match_policies": [
            {
                "flavor_part": " PLATFORM",
                "match_policy": {
                    "match_type": "ANY_OF",
                    "required": "REQUIRED"
                }
            }
        ]
    }
}
```

Response:

```json
{
    "id": "a0950923-596b-41f7-b9ad-09f525929ba1",
    "name": "firstTest",
    "flavor_match_policy_collection": {
        "flavor_match_policies": [
            {
                "flavor_part": " PLATFORM",
                "match_policy": {
                    "match_type": "ANY_OF",
                    "required": "REQUIRED"
                }
            }
        ]
    }
}
```

---
**NOTE**: 
By default only `automatic` flavor groups will be created and availabe as Default flavor group. Custom flavor groups required for user specific needs to be created by user before adding that particular flavor group for Host registration and Flavor creation.
---


## IMA Flavor

IMA flavor will be generated with the ima measurements present in the host manifest and this flavor can be created only once based on the request. IMA flavor will be associated with the host based on ALL_OF match type policy. This will be supported by the existing API call itself.

POST : https://{{hvs_ip}}:30443/hvs/v2/flavors

Body
```
{
  "connection_string":  "https://<host-ip>:31443;",
  "partial_flavor_types": ["PLATFORM","OS","HOST_UNIQUE","IMA","SOFTWARE"],
  "flavorgroup_names": []
}
```

IMA flavor will be created on default flavor group `automatic` based on the IMA Flavor template. This flavor will be stored in a flavors DB like other existing flavors.

# IMA Flavor template
  ```
	{  
		"label": "ima",  
		"condition" :[
			"//host_info/os_name//*[text()='RedHatEnterprise'] or //host_info/os_name//*[text()='Centos'] or //host_info/os_name//*[text()='Ubuntu']"
		], 
		"flavor_parts": {  
			"IMA": {  
				"pcr_rules": [{  
					"pcr": {  
						"index": 10,  
						"bank": ["SHA384", "SHA256", "SHA1"]  
					},  
					"pcr_matches": true  
				}]  
			}  
		}  
	}  
  ```
    
# Sample for IMA Flavor 

    ```
    {  
    "flavor":{  
        "meta":{  
        "id":"2c3bcb9d-b140-4732-8a12-0af281246fe8",  
        "description":{  
            "digest_algorithm":"SHA256",  
            "flavor_part":"IMA",  
            "label":"ima"  
        },  
        "vendor":"INTEL"  
        },    
        "pcrs": [{    
                    "pcr": {
            "index": 10,
            "bank":"SHA256"
                    }, 
        "measurement": "3f95ecbb0bb8e66e54d3f9e4dbae8fe57fed96f0",
        "pcr_matches":true
            }],  
        "ima_measurements":[{  
            "file":"/root/test",  
            "measurement":"12aacb9f9da0518937f3da6c0526b381787faa60a65b319208391f303b5fe7b7a"  
        },  
        {  
            "file":"/usr/share/vim/vim80/ftplugin/yamlfile.vim",  
            "measurement":"a0ca2d70982bf2db5a71a59ca7afd90aed7041ae703e2e711847c98aff0ac800"  
        }]    
    }  
    }    
    ```

As with other Flavor parts, hosts will be matched to Flavors in the same
Flavorgroup that the host is added to, and will not be matched to
Flavors in different Flavorgroups. Flavor matching will happen on the
same events as for other Flavor parts.

# Ima-Integrity rule

Current IMA logs will be taken from the hostmanifest file and the replay will be performed for all the IMA logs against the pcr10 cumulative hash recieved from the host.

# ImaEventLogEquals rule

IMA flavor will be compared against the host manifest data of ima-log. If it matches, then it will store ima-trust status as true.

# Attestation report generation

  - If the ima eventlog equals rule is true for all the measurements, we will provide only one entry in the report like below,
    ```
    "IMA": {
    	"trust": true,
    	"rules": [{
    		"rule": {
    			"rule_name": "rule.ImaEventLogEquals",
    			"markers": [
    				"IMA"
    			]
    		},
    		"flavor_id": "253e5fc3-11b0-4a38-a2a0-4a264aa40139",
    		"trusted": true
    	}]
    }  
    ``` 
    
  - If ima eventlog equals rule fails for any of the ima measurements, we will log those entries under faults in the reports as below.
    ```
    "IMA": {
        "trust": false,
        "rules": [{
            "rule": {
                "rule_name": "rule.ImaEventLogEquals",
                "markers": [
                    "IMA"
                ]
            },
            "flavor_id": "253e5fc3-11b0-4a38-a2a0-4a264aa40139",
            "faults": [{
                "fault_name": "fault.PcrValueMismatch",
                "description": "Host IMA log /usr/lib/systemd/libsystemd-shared-239.so with value 122e981e1d16de3269667f4e84bf8532462214f4f8a183e1c441bbfd3caecd10 does not match expected value 762e981e1d16de3269667f4e84bf84a0a0c84f4f8a183e13ac5ba1c441bbfd3c",
                "pcr_index": "10",
                "pcr_bank": "SHA256",
            }],
            "trusted": false
        }]
    }
    ```

  - If Hostmanifest contains unexpected/additional IMA log entries when compare to flavor measurements, PcrEventLogContainsUnexpectedEntries fault will be raised.
    ```
    "IMA": {
        "trust": false,
        "rules": [{
            "rule": {
                "rule_name": "rule.ImaEventLogEquals",
                "markers": [
                    "IMA"
                ]
            },
            "flavor_id": "253e5fc3-11b0-4a38-a2a0-4a264aa40139",
            "faults": [{
                "fault_name": "fault.PcrEventLogContainsUnexpectedEntries",
                "description": "Module manifest for PCR 10 of SHA256 value contains 1 unexpected entries",
                "unexpected_ima_entries": {
                    "pcr": {
                        "index": 10,
                        "bank": "SHA256"
                    },
                    "ima_measurements": [
                        {
                            "file": "/root/ima-cont-test/check1.txt",
                            "measurement": "73b5b283c458f698b3eafc6afbb01149dec48c541896bce2a74d79a0881d7864"
                        }
                    ],
                    "ima_template": "ima-ng"
                }
            }],
            "trusted": false
        }]
    }
    ```
  - If hostmanifest does not contain all the measurements from flavor, PcrEventLogMissingExpectedEntries fault will be raised in the report.

    ```
    "IMA": {
        "trust": false,
        "rules": [{
            "rule": {
                "rule_name": "rule.ImaEventLogEquals",
                "markers": [
                    "IMA"
                ]
            },
            "flavor_id": "253e5fc3-11b0-4a38-a2a0-4a264aa40139",
            "faults": [{
                "fault_name": "fault.PcrEventLogMissingExpectedEntries",
                "description": "Module manifest for PCR 10 of SHA256 value missing 2 expected entries",
                "unexpected_ima_entries": {
                    "pcr": {
                        "index": 10,
                        "bank": "SHA256"
                    },
                    "ima_measurements": [
                        {
                            "file": "/root/itest.txt",
                            "measurement": "73b5b283c458f698b3eafc6afbb01149dec48c541896bce2a74d79a0881d7864"
                        },
                        {
                            "file": "/root/icheck.txt",
                            "measurement": "fbf99c21c97b5134c22430e6bdd5f35a44e6d3a14855f13a8fdcce84eb52994a"
                        }
                    ],
                    "ima_template": "ima-ng"
                }
            }],
            "trusted": false
        }]
    }
    ```

# Sample for IMA measurement log integrity rule in report,

    ```
    "IMA": {
    "trust": true,
    "rules": [{
        "rule": {
            "rule_name": "rule.ImaMeasurementLogIntegrity",
            "markers": [
                "IMA"
            ],
            "expected_pcr": {
                "pcr": {
                    "index": 10,
                    "bank": "SHA256"
                },
                "measurement": "b6e1251e1d16de3269667f4e84bf84a0a0c84f4f8a183e13ac5ba1c441bbfd3c",
                "pcr_matches": true
            },
            "expected_imavalues": {
                "ima_measurements": [
                    {
                        "file": "boot_aggregate",
                        "measurement": "cacb533ca55a87d4e0c5954608102091ab42b99ae8fbbdbba7c93efad5e6a174"
                    },
                    {
                        "file": "/root/ima-cont-test/check1.txt",
                        "measurement": "b351e6014c0713eda511289d7af11a3c49f7162b471fb93adca869f37d6d4d15"
                    },
                    {
                        "file": "/root/ima-cont-test/check2.txt",
                        "measurement": "fc1ebc0a83cdea7e2e63063c81f935319207194b4425d3a88fd1f9c8de1184b7"
                    }
                ]
            }
        },
        "flavor_id": "12835fc3-11b0-4a38-a2a0-4a264aa40139",
        "trusted": true
    }]
    }
    ```

As part of tpm quote response, TA will send ima_log.

To include the ima logs in the tpm quote response, the following cofiguration parameters will be added by hvs in the tpm-quote request body.

    ```    
    `ima_measure_enabled` - it can be true or false under TA `config.yaml` located in `/etc/trustagent/v5.1.0/`. Only if it is true, 
    IMA logs will be sent by TA.
    ```

---
**NOTE**: 
IMA logs will be sent only when the ima enable option is set in the tpm quote request body and the same configuration is enabled in the TA as well.
---

## HVS Pagination
-------------------------

#   API Consumption

*   API consumers will be required to paginate through HVS data (ex. reports) if the amount of data in the database exceeds the limit or default limit (otherwise all records will be returned in a single query).
*   When starting pagination, API consumers will not provide a cursor value to their request (i.e. so that the first page is returned).
*   On subsequent page requests, API consumers will get the next page by providing the cursor returned in the previous call.
*   API consumers will terminate the collection of pages arbitrarily or when the cursor is less than zero (i.e. -1 will be returned by the controller to signal EOF).

#   Database changes

    Each of HVS' object tables will be update to include the “row_id” column (autoincrementing number id) which will be used as the pagination key.

    The following requirements/recommendations apply to all impacted database tables/queries.

    *   All impacted database tables must be updated to have auto-increment id field with unique and not null constraint applied.
    *   All impacted database queries must be updated to include limit construct. They should also be able to filter records based on provided "afterId" value.

#   API Changes

*   All the impacted end-points will support two additional query parameters, "limit" for specifying number of records to return, and "afterId" for specifying auto-increment id from which to return records.
*   If no "limit" parameter is passed, default limit will be used. And if no "afterId" parameter is passed, records from beginning will be returned.
*   All the impacted response models will support two additional json attributes, "next" link for fetching next page, and "prev" link for fetching previous page.
*   First page won't have "prev" link. And last page won't have "next" link.

---
**NOTE**: 
The default limit value is `1000` and default afterId value is `0`
---

# Impacted APIs

 The following "GET" (search) APIs are updated to handle pagination

| API	                            | Description                                                     |
|-----------------------------------|-----------------------------------------------------------------|
| GET /hosts                        | Get all the hosts registered on HVS                             |
| GET /reports 	                    | Get the latest report for all the hosts registered on HVS       |
| GET /host-status                  | Get the latest host-status for all the hosts registered on HVS  |
| GET /flavors                      | Get all the flavors created on HVS                              | 
| GET /flavorgroups                 | Get all the flavorgroups created on HVS                         |
| GET /exsi-cluster                 | Get all the exsi-clusters along with hostnames registered on HVS|
| GET /tpm-endorsements	            | Get all the active tpm-endorsement certificates uploaded to HVS |
