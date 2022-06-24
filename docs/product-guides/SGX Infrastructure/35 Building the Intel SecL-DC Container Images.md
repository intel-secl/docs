# Building the Intel SecL-DC Container Images

The Intel SecL-DC container images must be built from source and uploaded to a container registry.  Currently these images are not made available on any public container registry.

## Build Server Prerequisites
The Intel SecL container images must be built on Red Hat Enterprise Linux 8.4

### Development Tools and Utilities

#### RedHat Enterprise Linux 8.4

```
dnf install -y git wget tar python3 gcc gcc-c++ zip make yum-utils openssl-devel
dnf install -y https://download-ib01.fedoraproject.org/pub/epel/8/Everything/x86_64/Packages/m/makeself-2.4.2-1.el8.noarch.rpm
ln -s /usr/bin/python3 /usr/bin/python
ln -s /usr/bin/pip3 /usr/bin/pip
```

### Repo Tool

```
tmpdir=$(mktemp -d)
git clone https://gerrit.googlesource.com/git-repo $tmpdir
install -m 755 $tmpdir/repo /usr/local/bin
rm -rf $tmpdir
```

### Golang

```
wget https://dl.google.com/go/go1.18.linux-amd64.tar.gz
tar -xzf go1.18.linux-amd64.tar.gz
sudo mv go /usr/local
export GOROOT=/usr/local/go
export PATH=$GOROOT/bin:$PATH
rm -rf go1.18.linux-amd64.tar.gz
```

### Docker

```
dnf module enable -y container-tools
dnf install -y yum-utils
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
dnf install -y docker-ce-20.10.8 docker-ce-cli-20.10.8

systemctl enable docker
systemctl start docker
```

---
**NOTE**

Apply the following step only if you require a proxy:

```
mkdir -p /etc/systemd/system/docker.service.d
touch /etc/systemd/system/docker.service.d/proxy.conf
```

Add the below lines in /etc/systemd/system/docker.service.d/proxy.conf:

```
[Service]
Environment="HTTP_PROXY=<http_proxy>"
Environment="HTTPS_PROXY=<https_proxy>"
Environment="NO_PROXY=<no_proxy>"
```

Restart Docker:

```
systemctl daemon-reload
systemctl restart docker
```

---

### Skopeo

Installing skopeo in build environment
```
dnf install -y skopeo
```

## Sync the Repos


Sync the Repo:

```
mkdir -p /root/intel-secl/build/secure-key-caching && cd /root/intel-secl/build/secure-key-caching
repo init -u https://github.com/intel-secl/build-manifest.git -m manifest/skc.xml -b refs/tags/v5.0.0
repo sync
```

Run the additional prerequisite script:

```
cd utils/build/skc-tools/common_attest/tee/build_scripts/
chmod +x tee-build-prereqs.sh
./tee-build-prereqs.sh
```

## Building the Container Images

```
cd /root/intel-secl/build/secure-key-caching/
make k8s_skc
```

The resulting container images will be placed in the following folder:

```
/root/intel-secl/build/secure-key-caching/k8s/
```


---
**NOTE**

Build steps are same for single node and multi node deployment.

---

## Uploading the Container Images to a Registry

Once built, the images must be uploaded to a container registry using the appropriate version tags.  

| Intel SecL-DC Release Version | Required Container Version Tag |
| ----------------------------- | ------------------------------ |
| 5.0                           | v5.0.0                         |

The specific command to upload the images to the registry may differ based on the registry used.  The sample below assumes a Docker registry.

```
skopeo copy oci-archive:<oci-image-tar-name> docker://registry.server.com:<registry-port>/<image-name>:<image-tag>
```

---
**NOTE**
The container names used when uploading the images must match the container names specified in values.yaml when deploying with Helm.
---
