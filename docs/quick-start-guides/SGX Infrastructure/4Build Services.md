# Build Services

The below steps needs to be carried out on the Build and Deployment VM

## Pre-requisites

* Install basic utilities for getting started

On Red Hat Enterprise Linux 8.2/8.4

Enable the Following package repositories

  * `rhel-8-for-x86_64-appstream-rpms`
  * `rhel-8-for-x86_64-baseos-rpms`
  * `codeready-builder-for-rhel-8-for-x86_64-rpms`

```shell
dnf install git wget tar python3 gcc gcc-c++ zip tar make yum-utils curl openssl-devel

# RHEL-8.2
	dnf install skopeo -y
# RHEL-8.4
	dnf install skopeo -y --nobest
  
dnf install https://download-ib01.fedoraproject.org/pub/epel/8/Everything/x86_64/Packages/m/makeself-2.4.2-1.el8.noarch.rpm
ln -s /usr/bin/python3 /usr/bin/python
ln -s /usr/bin/pip3 /usr/bin/pip
export PATH=/usr/local/bin:$PATH
```
On Ubuntu

```shell
apt-get install -y software-properties-common git gcc zip wget make python3 python3-yaml python3-pip tar lsof jq nginx curl libssl-dev
ln -s /usr/bin/python3 /usr/bin/python
ln -s /usr/bin/pip3 /usr/bin/pip

sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
apt-get update

# Ubuntu-18.04
echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_18.04/ /" | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_18.04/Release.key | sudo apt-key add -
apt-get update
apt-get -y upgrade
apt-get -y install skopeo

# Ubuntu-20.04
echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_20.04/ /" | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_20.04/Release.key | sudo apt-key add -
apt-get update
apt-get -y upgrade
apt-get -y install skopeo

apt-get install makeself
export PATH=/usr/local/bin:$PATH
```

* Install Repo Tool

```shell
tmpdir=$(mktemp -d)
git clone https://gerrit.googlesource.com/git-repo $tmpdir
install -m 755 $tmpdir/repo /usr/local/bin
rm -rf $tmpdir
```

* Golang Installation

```shell
wget https://dl.google.com/go/go1.16.7.linux-amd64.tar.gz
tar -xzf go1.16.7.linux-amd64.tar.gz
mv go /usr/local
export GOROOT=/usr/local/go
export PATH=$GOROOT/bin:$PATH
rm -rf go1.16.7.linux-amd64.tar.gz
```

* Install, Enable and start the Docker daemon
  ```shell
  On RHEL 8.2:
    dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
    dnf install -y docker-ce-20.10.8 docker-ce-cli-20.10.8

  On RHEL 8.4:
    dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
    dnf install -y docker-ce-20.10.9 docker-ce-cli-20.10.9

  On Ubuntu 18.04:
    wget https://download.docker.com/linux/ubuntu/dists/bionic/pool/stable/amd64/containerd.io_1.4.11-1_amd64.deb
    dpkg -i containerd.io_1.4.11-1_amd64.deb
    wget "https://download.docker.com/linux/ubuntu/dists/bionic/pool/stable/amd64/docker-ce-cli_20.10.8~3-0~ubuntu-bionic_amd64.deb"
    dpkg -i docker-ce-cli_20.10.8~3-0~ubuntu-bionic_amd64.deb
    wget "https://download.docker.com/linux/ubuntu/dists/bionic/pool/stable/amd64/docker-ce_20.10.8~3-0~ubuntu-bionic_amd64.deb"
    dpkg -i docker-ce_20.10.8~3-0~ubuntu-bionic_amd64.deb

  On Ubuntu 20.04:
    wget https://download.docker.com/linux/ubuntu/dists/focal/pool/stable/amd64/containerd.io_1.4.11-1_amd64.deb
    dpkg -i containerd.io_1.4.11-1_amd64.deb
    wget "https://download.docker.com/linux/ubuntu/dists/focal/pool/stable/amd64/docker-ce-cli_20.10.8~3-0~ubuntu-focal_amd64.deb"
    dpkg -i docker-ce-cli_20.10.8~3-0~ubuntu-focal_amd64.deb
    wget "https://download.docker.com/linux/ubuntu/dists/focal/pool/stable/amd64/docker-ce_20.10.8~3-0~ubuntu-focal_amd64.deb"
    dpkg -i docker-ce_20.10.8~3-0~ubuntu-focal_amd64.deb

  systemctl enable docker
  systemctl start docker
  ```

* Ignore the below steps if not running behind a proxy
  ```shell
  mkdir -p /etc/systemd/system/docker.service.d
  touch /etc/systemd/system/docker.service.d/proxy.conf
  
  #Add the below lines in proxy.conf and set proxy server details if proxy is used
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

* Pulling Source Code

```shell
mkdir -p /root/workspace && cd /root/workspace
repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v4.1.1 -m manifest/skc.xml
repo sync
```

## Building SGX Attestation Usecase
For buiding SGX Attestation usecase only - Distribution Based Deployment
```shell
make attest
```

For buiding SGX Attestation usecase only - Stack Based Deployment
```shell
make attest_stacks
```

## Building Orchestration Usecase
For buiding Orchestration usecase - Distribution Based Deployment
```shell
make orchestrator
```

For buiding Orchestration usecase - Stack Based Deployment
```shell
make orchestrator_stacks
```

## Building Secure Key Caching Usecase
For buiding Secure Key Caching usecase  - Distribution Based Deployment
```shell
make skc
```

For buiding Secure Key Caching usecase - Stack Based Deployment
```shell
make skc_stacks
```

## Building Sample Application
For buiding Sample Application for SGX Attestation - Distribution Based Deployment
```shell
make app
```

For buiding Sample Application for SGX Attestation - Stack Based Deployment
```shell
make app_stacks
```

## Building All Usecases - Distribution Based Deployment
```shell
make
```

## Building All Usecases - Stack Based Deployment
```shell
make sgx-stack
```

* Built Binaries
```shell
cd /root/workspace/binaries
```
Copy the binaries directory generated in the build system to the /root/ directory on the deployment system
