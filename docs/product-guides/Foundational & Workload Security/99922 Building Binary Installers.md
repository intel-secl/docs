## Building Binary Installers

Intel® Security Libraries is distributed as open source code, and must be compiled into installation binaries before installation.

Instructions and sample scripts for building the Intel® SecL-DC components can be found here:

https://github.com/intel-secl/build-manifest

After the components have been built, the installation binaries can be found in the directories created by the build scripts.

```shell
<servicename>/out/<servicename>.bin
```

In addition, the build script will produce some sample database creation scripts that can be used during installation to configure database requirements (instructions are given in the installation sections):

create_db: `authservice/out/create_db.sh`

install_pgdb: `authservice/out/install_pgdb.sh`

In addition, sample Ansible roles to automatically build and deploy a testbed environment are provided:

https://github.com/intel-secl/utils/tree/v4.0.1/develop/tools/ansible-role

Also provided are sample API calls organized by workflows for Postman:

https://github.com/intel-secl/utils/tree/v4.0.1/develop/tools/api-collections