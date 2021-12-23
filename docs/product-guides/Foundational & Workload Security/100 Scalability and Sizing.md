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

**mtwilson.audit.log.num.rotations** - defines the maximum number of
rotations before the oldest rotation is deleted to make space for a new
rotation.

**mtwilson.audit.log.max.row.count** – defines the maximum number of
rows in the audit log table before a rotation will occur.



## Log Rotation
------------

The Intel® SecL services (the Verification Service, Trust Agent, and
Integration Hub) use Logrotate to rotate logs automatically during a
daily cron job.

By default, logs are rotated once per month or when they exceed 1 GB in
size, whichever comes first, and 12 total rotations will be retained.

