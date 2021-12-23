# Deployment

The below details would enable the deployment through Ansible Role for Intel® SecL-DC Foundational & Workload Security Usecases. However the services can still be installed manually using the Product Guide. More details on Ansible Role for Intel® SecL-DC in [Ansible-Role](https://github.com/intel-secl/utils/tree/v4.1/develop/tools/ansible-role) repository.


## Pre-requisites

* The Ansible Server is required to use this role to deploy Intel® SecL-DC services based on the supported deployment model. The Ansible server is recommended to be installed on the Build machine itself. 
* The role has been tested with Ansible Version `2.9.10`

### Installing Ansible

* Install Ansible on Build Machine

  ```shell
  pip3 install ansible==2.9.10
  ```

* Install `epel-release` repository and install `sshpass` for ansible to connect to remote hosts using SSH

  ```shell
  dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
  dnf install sshpass
  ```

* Create directory for ansible default configuration and hosts file

  ```shell
  mkdir -p /etc/ansible/
  touch /etc/ansible/ansible.cfg
  ```

* Copy the default `ansible.cfg` contents from https://raw.githubusercontent.com/ansible/ansible/v2.9.10/examples/ansible.cfg and paste it under `/etc/ansible/ansible.cfg`


## Download the Ansible Role

The role can be cloned locally from git and the contents can be copied to the roles folder used by your ansible server 

```shell
#Create directory for using ansible deployment
mkdir -p /root/intel-secl/deploy/

#Clone the repository
cd /root/intel-secl/deploy/ && git clone https://github.com/intel-secl/utils.git

#Checkout to specific release-version
cd utils/
git checkout <release-version of choice>
cd tools/ansible-role

#Update ansible.cfg roles_path to point to path(/root/intel-secl/deploy/utils/tools/)
```

## Usecase Setup Options

| Usecase                                                      | Variable                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Host Attestation                                             | `setup: host-attestation` in playbook or via `--extra-vars` as `setup=host-attestation` in CLI |
| Application Integrity                                        | `setup: application-integrity` in playbook or via `--extra-vars` as `setup=application-integrity` in CLI |
| Data Fencing & Asset Tags                                    | `setup: data-fencing` in playbook or via `--extra-vars` as `setup=data-fencing` in CLI |
| Trusted Workload Placement - VM            | `setup: trusted-workload-placement-vm` in playbook or via `--extra-vars` as `setup=trusted-workload-placement-vm` in CLI |
| Trusted Workload Placement - Containers                      | `setup: trusted-workload-placement-containers` in playbook or via `--extra-vars` as `setup=trusted-workload-placement-containers` in CLI |
| Launch Time Protection - VM Confidentiality                  | `setup: workload-conf-vm` in playbook or via `--extra-vars` as `setup=workload-conf-vm` in CLI |
| Launch Time Protection - Container Confidentiality with CRIO Runtime | `setup: workload-conf-containers-crio` in playbook or via `--extra-vars` as `setup=workload-conf-containers-crio`in CLI |

???+ note  
    Orchestrator installation is not bundled with the role and need to be done independently. Also, components dependent on the orchestrator like `isecl-k8s-extensions` and `integration-hub` are installed either partially or not installed

???+ note  
    `Key Broker Service` is not configured with KMIP compliant KMS when installing through ansible role  

## Update Ansible Inventory

In order to deploy Intel® SecL-DC binaries, the following inventory can be used and the required inventory vars as below need to be set. The below example inventory can be created under `/etc/ansible/hosts`

```
[CSP]
<machine1_ip/hostname>

[Enterprise]
<machine2_ip/hostname>

[Node]
<machine3_ip/hostname>

[CSP:vars]
isecl_role=csp
ansible_user=root
ansible_password=<password>

[Enterprise:vars]
isecl_role=enterprise
ansible_user=root
ansible_password=<password>

[Node:vars]
isecl_role=node
ansible_user=root
ansible_password=<password>
```

???+ note  
    Ansible requires `ssh` and `root` user access to remote machines. The following command can be used to ensure ansible can connect to remote machines with host key check. Ensure the existing keys of the machines are cleared to enable fresh keyscan.
  ```shell
  ssh-keyscan -H <ip_address/hostname> >> /root/.ssh/known_hosts
  ```

## Create and Run Playbook

The following are playbook and CLI example for deploying Intel® SecL-DC binaries based on the supported deployment models and usecases. The below example playbooks can be created as `site-bin-isecl.yml`

???+ note  
    If running behind a proxy, update the proxy variables under `vars/main.yml` and run as below

???+ note  
    Go through the `Additional Examples and Tips` section for specific workflow samples

**Option 1**

```yaml
- hosts: all
  gather_facts: yes
  any_errors_fatal: true
  vars:
    setup: <setup var from supported usecases>
    binaries_path: <path where built binaries are copied to>
  roles:   
  - ansible-role
  environment:
    http_proxy: "{{http_proxy}}"
    https_proxy: "{{https_proxy}}"
    no_proxy: "{{no_proxy}}"
```

and

```shell
ansible-playbook <playbook-name>
```

OR

**Option 2:**

```yaml
- hosts: all
  gather_facts: yes
  any_errors_fatal: true
  roles:   
  - ansible-role
  environment:
    http_proxy: "{{http_proxy}}"
    https_proxy: "{{https_proxy}}"
    no_proxy: "{{no_proxy}}"
```

and

```shell
ansible-playbook <playbook-name> \
--extra-vars setup=<setup var from supported usecases> \
--extra-vars binaries_path=<path where built binaries are copied to>
```

## Additional Examples & Tips

### TBoot Installation

Tboot needs to be built by the user from tboot source and the `tboot.gz` & `tboot-syms` files needs to be copied under the `binaries` folder. The supported version of Tboot as of 4.0 release is `tboot-1.10.1`.The options must then be provided during runtime in the playbook:

```shell
ansible-playbook <playbook-name> \
--extra-vars setup=<setup var from supported usecases> \
--extra-vars binaries_path=<path where built binaries are copied to> \
--extra-vars tboot_gz_file=<path where built binaries are copied to>/tboot.gz
--extra-vars tboot_syms_file=<path where built binaries are copied to>/tboot-syms
```

or 

Update the following in `vars/main.yml`

```yaml
# The TPM Storage Root Key(SRK) Password to be used if TPM is already owned
tboot_gz_file: "<binaries_path>/tboot.gz"
tboot_syms_file: "<binaries_path>/tboot-syms"
```

### TPM is already owned

If the Trusted Platform Module(TPM) is already owned, the owner secret(SRK) can be provided directly during runtime in the playbook:

```shell
ansible-playbook <playbook-name> \
--extra-vars setup=<setup var from supported usecases> \
--extra-vars binaries_path=<path where built binaries are copied to> \
--extra-vars tpm_secret=<tpm owner secret>
```

or

Update the following vars in `vars/main.yml`

```yaml
# The TPM Storage Root Key(SRK) Password to be used if TPM is already owned
tpm_owner_secret: <tpm_secret>
```

### UEFI SecureBoot enabled

If UEFI mode and UEFI SecureBoot feature is enabled, the following option can be used to during runtime in the playbook

```shell
ansible-playbook <playbook-name> \
--extra-vars setup=<setup var from supported usecases> \
--extra-vars binaries_path=<path where built binaries are copied to> \
--extra-vars uefi_secureboot=yes \
--extra-vars grub_file_path=<uefi mode grub file path>
```

or

Update the following vars in `vars/main.yml`

```yaml
# UEFI mode or UEFI SecureBoot mode
# ['no' - UEFI mode, 'yes' - UEFI SecureBoot mode]
uefi_secureboot: 'yes'

# The grub file path for UEFI Mode systems
# [/boot/efi/EFI/redhat/grub.cfg - UEFI Mode]
grub_file_path: /boot/efi/EFI/redhat/grub.cfg
```

### In case of Misconfigurations 

If any service installation fails due to any misconfiguration, just uninstall the specific service manually , fix the misconfiguration in ansible and rerun the playbook. The successfully installed services wont be reinstalled.

