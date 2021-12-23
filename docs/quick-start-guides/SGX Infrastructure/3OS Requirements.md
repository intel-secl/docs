# OS Requirements

Ensure that you have one of the following required operating systems:

* Ubuntu* 18.04/20.04 LTS
* Red Hat Enterprise Linux Server release 8.2 64bits
* Red Hat Enterprise Linux Server release 8.4 64bits for Stack based deployment

Date and time should be synced across all systems. If a system is configured to read the RTC time in the local time zone, then use RTC in UTC by running `timedatectl set-local-rtc 0` command on all deployment systems.

## User Access

* All services need to be built & deployed as `root` user.
???+ note 
    When using Ansible role for deployment, Ansible needs to be able to talk to remote machines as root user for successful deployment
* All Intel® SecL-DC service & agent ports should be allowed in firewall rules. 

## Proxy Settings

If system is behind a proxy, configure system to use proxy. 
Sample configuration
```
export http_proxy=http://<proxy-url>:<proxy-port>
export https_proxy=http://<proxy-url>:<proxy-port>
export no_proxy=0.0.0.0,127.0.0.1,localhost,<Deployment System IP>, <SGX Compute Node IP>, <KBS system Hostname>
```