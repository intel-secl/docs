# Workload Agent

## Installation Answer File Options

## Configuration Options

## Command-Line Options

Available Commands:

###  Help

wlagent help|-help|--help Show help message

###  setup

wlagent setup [task] Run setup task

#### Available Tasks for setup

##### SigningKey

Generate a TPM signing key

##### BindingKey

Generate a TPM binding key

##### RegisterSigningKey

Register a signing key with the host verification service

-   Environment variable BEARER_TOKEN=<token> for authenticating with
    Verification service

##### RegisterBindingKey

Register a binding key with the host verification service

-   Environment variable BEARER_TOKEN=<token> for authenticating with
    Verification service

### start

Start wlagent

### stop

Stop wlagent

### status

Reports the status of wlagent service

### uninstall

Uninstall wlagent

### uninstall --purge

Uninstalls workload agent and deletes the existing configuration
directory

### version

Reports the version of the workload agent

## Directory Layout

The Workload Agent installs by default to /opt/workload-agent with the
following folders.

### Bin

Contains scripts and executable binaries.
