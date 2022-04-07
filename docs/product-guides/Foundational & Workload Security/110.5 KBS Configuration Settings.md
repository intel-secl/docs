# Key Broker Service

## Installation Answer File Options

## Configuration Options

## Command-Line Options

The Key Broker Service supports several command-line commands that can
be executed only as the Root user:

Usage:
        kbs <command> [arguments]

Available Commands:
        help|-h|--help         						  	Show this help message
        version|-v|--version   					 	 Show the version of current kbs build
        setup <task>           							 Run setup task
        start                  									   Start kbs
        status                 							    	 Show the status of kbs
        stop                   									  Stop kbs
        uninstall [--purge]    							 Uninstall kbs
                --purge            								all configuration and data files will be removed if this flag is set

Usage of kbs setup:
        kbs setup <task> [--help] [--force] [-f <answer-file>]
                --help                      show help message for setup task
                --force                     existing configuration will be overwritten if this flag is set
                -f|--file <answer-file>     the answer file with required arguments

Available Tasks for setup:
        all                                 Runs all setup tasks
        download-ca-cert                    Download CMS root CA certificate
        download-cert-tls                   Download CA certificate from CMS for tls
        create-default-key-transfer-policy  Create default key transfer policy for KBS
        update-service-config               Sets or Updates the Service configuration

## Directory Layout

The Verification Service installs by default with the following folders:

### /opt/kbs/bin

Contains KBS binaries

### /etc/kbs/

Contains KBS configuration files

### /var/log/kbs/

Contains KBS logs
