# Build

## Pre-requisites

The below steps need to be done on RHEL 8.2 Build machine (VM/Physical Node)

### Development Tools and Utilities

```shell
dnf install -y git wget tar python3 gcc gcc-c++ zip tar make yum-utils openssl-devel
dnf install -y https://dl.fedoraproject.org/pub/fedora/linux/releases/32/Everything/x86_64/os/Packages/m/makeself-2.4.0-5.fc32.noarch.rpm
ln -s /usr/bin/python3 /usr/bin/python
ln -s /usr/bin/pip3 /usr/bin/pip
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
wget https://dl.google.com/go/go1.14.4.linux-amd64.tar.gz
tar -xzf go1.14.4.linux-amd64.tar.gz
sudo mv go /usr/local
export GOROOT=/usr/local/go
export PATH=$GOROOT/bin:$PATH
rm -rf go1.14.4.linux-amd64.tar.gz
```

### Docker

```shell
dnf module enable -y container-tools
dnf install -y yum-utils
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
dnf install -y docker-ce-19.03.13 docker-ce-cli-19.03.13

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

```shell
dnf install -y skopeo
```

## Build OCI Container images and K8s Manifests

The build process for OCI containers images and K8s manifests for RHEL 8.2 & Ubuntu 18.04 deployments must be done on RHEL 8.2 machine only

### Single Node

* Sync the repos

  ```shell
  mkdir -p /root/intel-secl/build/skc-k8s-single-node && cd /root/intel-secl/build/skc-k8s-single-node
  repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v4.0.2 -m manifest/skc.xml
  repo sync
  ```

* Build

  ```shell
  make k8s-aio
  ```

* Built Container images,K8s manifests and deployment scripts

  ```shell
  /root/intel-secl/build/skc-k8s-single-node/k8s/
  ```

### Multi Node

* Sync the repos

  ```shell
  mkdir -p /root/intel-secl/build/skc-k8s-multi-node && cd /root/intel-secl/build/skc-k8s-multi-node
  repo init -u https://github.com/intel-secl/build-manifest.git -b refs/tags/v4.0.2 -m manifest/skc.xml
  repo sync
  ```

* Build

  ```shell
  make k8s
  ```

* Built Container images,K8s manifests and deployment scripts

  ```shell
  /root/intel-secl/build/skc-k8s-multi-node/k8s/
  ```
