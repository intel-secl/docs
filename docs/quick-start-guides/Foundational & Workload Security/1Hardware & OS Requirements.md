# Hardware & OS Requirements

## Physical Server requirements

* Intel® SecL-DC  supports and uses a variety of Intel security features, but there are some key requirements to consider before beginning an installation. Most important among these is the Root of Trust configuration. This involves deciding what combination of TXT, Boot Guard, tboot, and UEFI Secure Boot to enable on platforms that will be attested using Intel® SecL.

???+ note 
    At least one "Static Root of Trust" mechanism must be used (TXT and/or BtG). For Legacy BIOS systems, tboot must be used. For UEFI mode systems, UEFI SecureBoot must be used* Use the chart below for a guide to acceptable configuration options. Only dTPM is supported on Intel® SecL-DC platform hardware. 

  ![hardware-options](./images/trusted-boot-options.PNG)

## OS Requirements

* `RHEL 8.3` OS
* `rhel-8-for-x86_64-baseos-rpms` and `rhel-8-for-x86_64-appstream-rpms` repositories need to be enabled on build machine and remote machines
* Date and time should be in sync across the machines


## User Access

* The services need to be built & installed as `root` user. Ensure root privileges are present for the user to work with Intel® SecL-DC.
  
???+ note 
    When using Ansible role for deployment, Ansible needs to be able to talk to remote machines as root user for successful deployment

* All Intel® SecL-DC service & agent ports should be allowed in firewall rules. 

