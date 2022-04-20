# Build

## Pre-requisites

The below steps need to be done on Ubuntu 18.04/20.04(stack based deployment) Build machine (VM/Physical Node)

### Development Tools and Utilities

```shell
#RHEL 8.4(stack based deployment only)
dnf install -y git wget tar python3 gcc gcc-c++ zip tar make yum-utils openssl-devel
dnf install -y https://dl.fedoraproject.org/pub/fedora/linux/releases/32/Everything/x86_64/os/Packages/m/makeself-2.4.0-5.fc32.noarch.rpm
ln -s /usr/bin/python3 /usr/bin/python
ln -s /usr/bin/pip3 /usr/bin/pip


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

### Repo tool

```shell
tmpdir=$(mktemp -d)
git clone https://gerrit.googlesource.com/git-repo $tmpdir
install -m 755 $tmpdir/repo /usr/local/bin
rm -rf $tmpdir
```

### Golang

```shell
wget https://dl.google.com/go/go1.16.7.linux-amd64.tar.gz
tar -xzf go1.16.7.linux-amd64.tar.gz
sudo mv go /usr/local
export GOROOT=/usr/local/go
export PATH=$GOROOT/bin:$PATH
rm -rf go1.16.7.linux-amd64.tar.gz
```

### Docker

```shell
On RHEL 8.2:
    dnf module enable -y container-tools
    dnf install -y yum-utils
    dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
    dnf install -y docker-ce-20.10.8 docker-ce-cli-20.10.8

On RHEL 8.4:
    dnf module enable -y container-tools
    dnf install -y yum-utils
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

#Ignore the below steps if not running behind a proxy
mkdir -p /etc/systemd/system/docker.service.d
touch /etc/systemd/system/docker.service.d/proxy.conf

#Add the below lines in proxy.conf
[Service]
Environment="HTTP_PROXY=<http_proxy>"
Environment="HTTPS_PROXY=<https_proxy>"
Environment="NO_PROXY=<no_proxy>"

systemctl daemon-reload
systemctl restart docker
```

### Skopeo

For RHEL 8.4 OS	

```shell
dnf install -y skopeo --nobest
```

For Ubuntu 18.04 OS

```shell
echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_18.04/ /" | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_18.04/Release.key | sudo apt-key add -
apt-get update
apt-get -y upgrade
apt-get -y install skopeo
```

For Ubuntu 20.04 OS

```shell
echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_20.04/ /" | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_20.04/Release.key | sudo apt-key add -
apt-get update
apt-get -y upgrade
apt-get -y install skopeo
```

## Build OCI Container images and K8s Manifests

The build process for OCI containers images and K8s manifests for RHEL 8.2 & Ubuntu 18.04 deployments must be done on RHEL 8.2 machine only

The build process for OCI containers images and K8s manifests for RHEL 8.4 must be done on RHEL 8.4 machine only

### Single Node

* Sync the repos

  ```shell
  mkdir -p /root/intel-secl/build/skc-k8s-single-node && cd /root/intel-secl/build/skc-k8s-single-node
  repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v4.0.1 -m manifest/skc.xml
  repo sync
  ```

* Build - Distribution Based Deployment

  ```shell
  make k8s-aio
  ```

* Build - Stack Based Deployment

  ```shell
  make k8s-aio-stacks
  ```

* Built Container images,K8s manifests and deployment scripts

  ```shell
  /root/intel-secl/build/skc-k8s-single-node/k8s/
  ```

### Multi Node

* Sync the repos

  ```shell
  mkdir -p /root/intel-secl/build/skc-k8s-multi-node && cd /root/intel-secl/build/skc-k8s-multi-node
  repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v4.2.0-Beta -m manifest/skc.xml
  repo sync
  ```

* Build - Distribution Based Deployment

  ```shell
  make k8s
  ```
  
* Build - Stack Based Deployment

  ```shell
  make k8s-stacks
  ```

* Built Container images,K8s manifests and deployment scripts

  ```shell
  /root/intel-secl/build/skc-k8s-multi-node/k8s/
  ```

## Deploy Linux Stacks for Intel SGX
	
* To setup and deployment of the Linux* Stacks for Intel® SGX, follow https://download.01.org/intelsgxstack/2021-07-28/Getting_Started.pdf
