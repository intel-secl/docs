# Binary Upgrades

???+ note 
    Before performing any upgrade, Intel strongly recommends backing up the database for the SHVS, SCS, and AAS.  See Postgres documentation for detailed options for backing up databases.  Below is a sample method for backing up an entire database server:

```
Backup to tar file:
pg_dump --dbname <database_name> --username=<database username> -F t > <database_backup_file>.tar
Restore from tar file:
pg_restore --dbname=<database_name> --username=<database username><database_backup_file>.tar
```

Some upgrades may involve changes to database content, and a backup will ensure that data is not lost in the case of an error during the upgrade process.

## Backward Compatibility

In general Intel SecL services are made to be backward-compatible within a given major release (for example, the 3.6 SHVS should be compatible with the 3.5 SGX Agent) in an upgrade priority order (see below).  Major version upgrades may require coordinated upgrades across all services.

## Upgrade Order

Upgrades should be performed in the following order to prevent misconfiguration or any service unavailability:

1) CMS, AAS

2) SCS, SHVS

3) SQVS, KBS, SGX Agent

Upgrading in this order will make each service unavailable only for the duration of the upgrade for that service.  

## Upgrade Process

### Binary Installations

For services installed directly (not deployed as containers), the upgrade process simply requires executing the new-version installer on the same machine where the old-version is running.  The installer will re-use the same configuration elements detected in the existing version's config file.  No additional answer file is required.
