# EXPERIMENT with rhel8/dotnet-21

## Executed on an entitled RHEL8 machine

```
podman run --name=rhel8-dotnet-21 -it --user=root docker://registry.redhat.io/rhel8/dotnet-21 /bin/bash
```

### Need to enable EPEL8 to install mono and mono-based components

```
dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
```

```
dnf config-manager --set-enabled epel
```

### Need to import the Fedora EPEL8 gpgkey before installing packages

```
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-8
```

### Install support packages

```
dnf install -y git unzip wget gcc-c++ cyrus-sasl-devel mono-winfx mono-wcf mono-devel swig cpio
```

### Clone repo

```
mkdir git-repo
cd git-repo
git clone https://github.com/worsco/dotnet-client.git
cd dotnet-client
```

Switch to rhel8 branch

```
git checkout rhel8
```

### Build it
```
./build.sh
```

### Test it
