# Build

## Pre-requisites

The below steps need to be done on `RHEL 8.3`/`Ubuntu-18.04`/`Ubuntu 20.04` Build machine (VM/Physical Node)

### Development Tools and Utilities

```shell
# RedHat Enterprise Linux 8.3
dnf install -y git wget tar python3 gcc gcc-c++ zip make yum-utils openssl-devel
dnf install -y https://dl.fedoraproject.org/pub/fedora/linux/releases/32/Everything/x86_64/os/Packages/m/makeself-2.4.0-5.fc32.noarch.rpm
ln -s /usr/bin/python3 /usr/bin/python
ln -s /usr/bin/pip3 /usr/bin/pip

# Ubuntu-18.04/Ubuntu-20.04
apt update
apt remove -y gcc gcc-7
apt install -y python3-problem-report git wget tar python3 gcc-8 make makeself openssl libssl-dev libgpg-error-dev
cp /usr/bin/gcc-8 /usr/bin/gcc
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
wget https://dl.google.com/go/go1.16.7.linux-amd64.tar.gz
tar -xzf go1.16.7.linux-amd64.tar.gz
sudo mv go /usr/local
export GOROOT=/usr/local/go
export PATH=$GOROOT/bin:$PATH
rm -rf go1.16.7.linux-amd64.tar.gz
```

### Docker

```shell
# RedHat Enterprise Linux-8.3
dnf module enable -y container-tools
dnf install -y yum-utils
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
dnf install -y docker-ce-20.10.8 docker-ce-cli-20.10.8

systemctl enable docker
systemctl start docker

# Ubuntu-18.04/Ubuntu-20.04
apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
    
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

apt-get update

# Ubuntu 18.04
apt-get -y install docker-ce=5:20.10.8~3-0~ubuntu-bionic docker-ce-cli=5:20.10.8~3-0~ubuntu-bionic containerd.io

# Ubuntu 20.04 
apt-get -y install docker-ce=5:20.10.8~3-0~ubuntu-focal docker-ce-cli=5:20.10.8~3-0~ubuntu-focal containerd.io

systemctl enable docker
systemctl start docker
```

Apply the below steps **only if** running behind a proxy

```shell
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

## Build OCI Container images and K8s Manifests

### Foundational Security

* Sync the repos

  ```shell
  mkdir -p /root/intel-secl/build/fs && cd /root/intel-secl/build/fs
  repo init -u https://github.com/intel-secl/build-manifest.git -m manifest/fs.xml -b refs/tags/v4.1.0-Beta
  repo sync
  ```

* Run the `pre-requisites` setup script

  ```shell
  cd utils/build/foundational-security/
  chmod +x fs-prereq.sh
  ./fs-prereq.sh -s
  ```

* Install skopeo

  ```shell
  # RHEL 8.x
  dnf install -y skopeo

  # Ubuntu 18.04
  echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_18.04/ /" | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
  curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_18.04/Release.key | sudo 
  
  # Ubuntu 20.04
  echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_20.04/ /" | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
  curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_20.04/Release.key | sudo 

  # Ubuntu-18.04/Ubuntu-20.04
  apt-key add -
  apt-get update
  apt-get -y upgrade
  apt-get -y install skopeo
  ```
  
* Build

  ```shell
  cd /root/intel-secl/build/fs/
  
  #Single node cluster with microk8s
  make k8s-aio
  
  #Multi node cluster with kubeadm
  make k8s
  ```

* Built Container images,K8s manifests and deployment scripts

  ```shell
  /root/intel-secl/build/fs/k8s/
  ```

### Workload Security

#### Container Confidentiality with CRIO Runtime

* Sync the repos

  ```shell
  mkdir -p /root/intel-secl/build/cc-crio && cd /root/intel-secl/build/cc-crio
  repo init -u https://github.com/intel-secl/build-manifest.git -m manifest/cc-crio.xml -b refs/tags/v4.1.0-Beta
  repo sync
  ```

* Run the `pre-requisites` script

  ```shell
  cd utils/build/workload-security
  chmod +x ws-prereq.sh
  ./ws-prereq.sh -c
  ```
  
* Build

  ```shell
  cd /root/intel-secl/build/cc-crio
  
  #Single node cluster with microk8s
  make k8s-aio
  
  #Multi node cluster with kubeadm
  make k8s
  ```

* Built Container images,K8s manifests and deployment scripts

  ```shell
  /root/intel-secl/build/cc-crio/k8s/
  ```
