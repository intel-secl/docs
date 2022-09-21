# Scalability and Sizing

## Configuration Maximums

### Registered Hosts

The Intel® SecL Verification Service can support a maximum of 2000
registered hosts with a single Verification Service instance with
default settings and the specified minimum required hardware.

The Verification Service can support up to 100,000 registered hosts, but will require a minimum of 16 vCPUs and 16GB RAM.  The service is primarily CPU-limited.

### HDD Space

The HDD space recommendations below represent expected log and database
growth using default settings. Altering the database or log rotation
settings, or the SAML expiration setting, may change the amount of disk
space required. For default settings, 100 GB of disk space is
recommended.



## Database Rotation Settings
--------------------------

The Intel® SecL Verification Service database will automatically rotate
the audit log table after one million records, and will retain up to ten
total rotations. These settings are user-configurable if a longer
retention period is needed.

**number-rotated** - defines the maximum number of
rotations before the oldest rotation is deleted to make space for a new
rotation. Default value is `10`

**max-row-count** – defines the maximum number of
rows in the audit log table before a rotation will occur. Default is
`10,000`

???+ note 
        - Rotation settings can be updated in the `config.yaml` file located in the nfs server path `<nfs_path mentioned in values.yaml>/isecl/hvs/config/`

        - After updation is completed, restart the service by deleting the pod.  The Kubernetes deployment will automatically launch a new pod, which will reflect the updated settings.

            ```
            kubectl delete pod -n <namespace> <hvspodname>
            ```
