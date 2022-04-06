# Build Services and packages

The below steps needs to be carried out on the Build and Deployment VM

## Pre-requisites

* The repos can be built only as `root` user

* RHEL 8.4 Machine for building repos

* Enable the following RHEL repos:

  * `rhel-8-for-x86_64-appstream-rpms`
  * `rhel-8-for-x86_64-baseos-rpms`

* Install basic utilities for getting started

  ```shell
  dnf install git wget tar python3 yum-utils
  ```

* Create symlink for python3

  ```shell
  ln -s /usr/bin/python3 /usr/bin/python
  ln -s /usr/bin/pip3 /usr/bin/pip
  ```

* Install repo tool

  ```shell
  tmpdir=$(mktemp -d)
  git clone https://gerrit.googlesource.com/git-repo $tmpdir
  install -m 755 $tmpdir/repo /usr/local/bin
  rm -rf $tmpdir
  ```

* Golang installation
  
```shell
  wget https://dl.google.com/go/go1.16.7.linux-amd64.tar.gz
  tar -xzf go1.16.7.linux-amd64.tar.gz
  sudo mv go /usr/local
  export GOROOT=/usr/local/go
  export PATH=$GOROOT/bin:$PATH
  rm -rf go1.16.7.linux-amd64.tar.gz
  ```
  
## Building

### Foundational Security Usecase

* Sync the repo

  ```shell
  mkdir -p /root/intel-secl/build/fs && cd /root/intel-secl/build/fs
  repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v5.0.0 -m manifest/fs.xml
  repo sync
  ```

* Run the `pre-requisites` setup script

  ```shell
  cd utils/build/foundational-security/
  chmod +x fs-prereq.sh
  ./fs-prereq.sh -s
  ```

* Build all repos

  ```shell
  cd /root/intel-secl/build/fs/
  make binaries
  ```

* Built Binaries

  ```shell
  /root/intel-secl/build/fs/binaries
  ```

### Workload Security Usecase

#### VM Confidentiality

* Sync the repo

  ```shell
  mkdir -p /root/intel-secl/build/vmc && cd /root/intel-secl/build/vmc
  repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v5.0.0 -m manifest/vmc.xml
  repo sync
  ```

* Run the pre-req script

  ```shell
  cd utils/build/workload-security
  chmod +x ws-prereq.sh
  ./ws-prereq.sh -v
  ```
  
* Build repo

  ```shell
  cd /root/intel-secl/build/vmc/
  make all
  ```

* Built Binaries
  ```shell
  /root/intel-secl/build/vmc/binaries/
  ```

#### Container Confidentiality with CRIO Runtime

* Sync the repo

  ```shell
  mkdir -p /root/intel-secl/build/cc-crio && cd /root/intel-secl/build/cc-crio
  repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v5.0.0 -m manifest/cc-crio.xml
  repo sync
  ```

* Run the `pre-requisites` script

  ```shell
  cd utils/build/workload-security
  chmod +x ws-prereq.sh
  ./ws-prereq.sh -c
  ```
  
* Enable and start the Docker daemon

  ```shell
  systemctl enable docker
  systemctl start docker
  ```

* Ignore the below steps if not running behind a proxy

  ```shell
  mkdir -p /etc/systemd/system/docker.service.d
  touch /etc/systemd/system/docker.service.d/proxy.conf
  
  #Add the below lines in proxy.conf
  [Service]
  Environment="HTTP_PROXY=<http_proxy>"
  Environment="HTTPS_PROXY=<https_proxy>"
  Environment="NO_PROXY=<no_proxy>"
  ```

  ```shell
  #Reload docker
  systemctl daemon-reload
  systemctl restart docker
  ```

* Download go dependencies

  ```shell
  cd /root/
  go get github.com/cpuguy83/go-md2man
  mv /root/go/bin/go-md2man /usr/bin/
  ```

* Build the repos

  ```shell
  cd /root/intel-secl/build/cc-crio
  make binaries
  ```
  
???+ note  
    The crio use case uses containerd that is bundled with `docker-ce-20.10.8` during build time. As of this release , the version being used is `containerd-1.4.4`. If the remote docker-ce repo gets updated for newer containerd version, then the version of containerd might be incompatible for building crio use case. It is recommended to use the version 1.4.4 in that case.
  
* Built binaries
  
  ```shell
  /root/intel-secl/build/cc-crio/binaries/
  ```
