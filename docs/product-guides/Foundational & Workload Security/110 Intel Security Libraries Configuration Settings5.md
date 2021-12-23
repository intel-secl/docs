# Authentication and Authorization Service

## Installation Answer File Options

| Key                             | Sample Value                                 | Description                                                  |
| ------------------------------- | -------------------------------------------- | ------------------------------------------------------------ |
| CMS\_BASE\_URL                  | https://<cms IP or hostname\>/cms/v1/        | Required; Provides the URL for the CMS.                      |
| AAS\_NOSETUP                    | false                                        | Optional. Determines whether “setup” will be executed after installation. Typically this is set to “false” to install and perform setup in one action. The “true” option is intended for building the service as a container, where the installation would be part of the image build, and setup would be performed when the container starts for the first time to generate any persistent data. |
| AAS\_DB\_HOSTNAME               | localhost                                    | Required. Hostname or IP address of the AAS database         |
| AAS\_DB\_PORT                   | 5432                                         | Required. Database port number                               |
| AAS\_DB\_NAME                   | pgdb                                         | Required. Database name                                      |
| AAS\_DB\_USERNAME               | dbuser                                       | Required. Database username                                  |
| AAS\_DB\_PASSWORD               | dbpassword                                   | Required. Database password                                  |
| AAS\_DB\_SSLMODE                | verify-ca                                    | Defines the SSL mode for the connection to the database. If not specified, the database connection will not use certificate verification. If specified, certificate verification will be required for database connections. |
| AAS\_DB\_SSLCERTSRC             | /usr/local/pgsql/data/server.crt             | Optional, required if the`“AAS_DB_SSLMODE` is set to `verify-ca` Defines the location of the database SSL certificate. |
| AAS\_DB\_SSLCERT                | \<path\_to\_cert\_file\_on\_system\>         | Optional. The `AAS_DB_SSLCERTSRC` variable defines the source location of the database SSL certificate; this variable determines the local location. If the former option is used without specifying this option, the service will copy the SSL certificate to the default configuration directory. |
| AAS\_ADMIN\_USERNAME            | admin@aas                                    | Required. Defines a new AAS administrative user. This user will be able to create new users, new roles, and new role-user mappings. This user will have the `AAS:Administrator` role. |
| AAS\_ADMIN\_PASSWORD            | aasAdminPass                                 | Required. Password for the new AAS admin user.               |
| AAS\_JWT\_CERT\_SUBJECT         | "AAS JWT Signing Certificate"                | Optional. Defines the subject of the JWT signing certificate. |
| AAS\_JWT\_TOKEN\_DURATION\_MINS | 5                                            | Optional. Defines the amount of time in minutes that an issued token will be valid. |
| SAN\_LIST                       | 127.0.0.1,localhost,10.x.x.x                 | Comma-separated list of IP addresses and hostnames that will be valid connection points for the service. Requests sent to the service using an IP or hostname not in this list will be denied, even if it resolves to this service. |
| BEARER\_TOKEN                   | \<token\>                                    | Required. Token from the CMS generated during CMS setup that allows the AAS to perform initial setup tasks. |
| LOG\_LEVEL                      | Critical, error, warning, info, debug, trace | Optional. Defaults to INFO. Changes the log level used.      |

## Configuration Options

## Command-Line Options

Usage:
        authservice <command> [arguments]

Available Commands:
        -h|--help | help                 Show this help message
        setup <task>                     Run setup task
        start                            Start authservice
        status                           Show the status of authservice
        stop                             Stop authservice
        uninstall [--purge]              Uninstall authservice. --purge option needs to be applied to remove configuration and data files
        -v|--version | version           Show the version of authservice

Usage of authservice setup:
        authservice setup [task] [--help] [--force] [-f <answer-file>]
                --help                      show help message for setup task
                --force                     existing configuration will be overwritten if this flag is set
                -f|--file <answer-file>     the answer file with required arguments

        Available Tasks for setup:
                all                      Runs all setup tasks
                download-ca-cert         Download CMS root CA certificate
                download-cert-tls        Download CA certificate from CMS for tls
                database                 Setup authservice database
                admin                    Add authservice admin username and password to database and assign respective
                                         roles to the user
                jwt                      Create jwt signing key and jwt certificate signed by CMS
                update-service-config    Sets or Updates the Service configuration



## Directory Layout

The Verification Service installs by default to `/opt/authservice` with
the following folders.

### Bin

Contains executable scripts and binaries.

### dbscripts

Contains database scripts.

