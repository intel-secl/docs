# Ubuntu System Tools and Utilities

## Ubuntu System Tools and utils

```
apt-get install -y software-properties-common git gcc zip wget make python3 python3-yaml python3-pip tar lsof jq nginx curl libssl-dev
ln -s /usr/bin/python3 /usr/bin/python
ln -s /usr/bin/pip3 /usr/bin/pip
add-apt-repository ppa:projectatomic/ppa
apt-get update
apt-get install skopeo
apt-get install makeself
export PATH=/usr/local/bin:$PATH
```

> Note 
    After skopeo installation, if /etc/containers/policy.json file not available then follow below steps:
```
mkdir /etc/containers
touch /etc/containers/policy.json 

#Add below contents in policy.json
{
    "default": [
        {
            "type": "insecureAcceptAnything"
        }
    ],
    "transports":
        {
            "docker-daemon":
                {
                    "": [{"type":"insecureAcceptAnything"}]
                }
        }
}
```

## Repo Tool

```
tmpdir=$(mktemp -d)
git clone https://gerrit.googlesource.com/git-repo $tmpdir
install -m 755 $tmpdir/repo /usr/local/bin
rm -rf $tmpdir
```

## Golang Installation

```
wget https://dl.google.com/go/go1.14.1.linux-amd64.tar.gz
tar -xzf go1.14.1.linux-amd64.tar.gz
sudo mv go /usr/local
export GOROOT=/usr/local/go
export PATH=$GOROOT/bin:$PATH
rm -rf go1.14.1.linux-amd64.tar.gz
```

***To Generate Swagger Documentation for components, Java Runtime needs to be installed***

```
apt-get install -y default-jre
```
