# EXPERIMENT with rhel8/dotnet-21

## pod entitled build (can be performed on RHEL7/RHEL8/FC host)

### You need an entitlement for a RHEL8 system

1. Create a System in https://access.redhat.com/
2. Click on "Subscriptions"
3. Click on "Systems"
4. Click on "New"
   * System Type: [x] Virutal SYstem
   * Name: rhel8-container
   * Architecture: x86_64
   * Number of vCPUs: 2
   * Red Hat Enterprise Linux Version: 8
5. Click on "CREATE"
6. Attach a subscription to that system
   * My example: Employee SKU
7. "Download Certificates" link is now available
   * Download the entitlement file (.zip file)
8. Expand the contents of the .zip
9. Inside is a file named `consumer_export.zip`
   * Export the `/export/entitlement_certificates/*.pem` file into `$HOME/Development/rhel8-entitlements/`

```
mkdir -p $HOME/Development/rhel8-entitlements/
cd $HOME/Development/rhel8-entitlements/
```
10. You need to make a copy of the *.pem file as *-key.pem
    * We will be downloading an RHEL8 container image, you need to log into registry.redhat.io

```
podman login registry.redhat.io
```

* The pod will be run as root, and also needs "--privileged" (when SELinux is enabled) so that we can mount the directory that contains the entitlement within the pod.

```
podman run --privileged -it --user=root --name=rhel8-dotnet-21 \
-v $HOME/Development/rhel8-entitlements/:/etc/pki/entitlement \
registry.redhat.io/rhel8/dotnet-21 \
/bin/bash
```

* You are now "root" inside of the pod.

* Disable the mounting of the host's certs
If you don't remove the /etc/rhsm-host mount, you will not be able to use the entitlement as the entitlement will conflict with the ca provided by the host.
  
```
rm /etc/rhsm-host
```

Install EPEL repo

```
dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
```

Enable EPEL repo

```
dnf config-manager --set-enabled epel
```

Enable RHEL8 repos

```
dnf config-manager --set-enabled rhel-8-for-x86_64-baseos-rpms
dnf config-manager --set-enabled rhel-8-for-x86_64-appstream-rpms
```

Import the EPEL8 gpg signing key

```
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-8
```

Install necessary RPMs to build

```
dnf install -y git unzip wget gcc-c++ cyrus-sasl-devel mono-winfx mono-wcf mono-devel swig cpio
```

Clone repo that has code fixes

```
git clone https://github.com/worsco/dotnet-client
cd dotnet-client
```

Switch to the branch for RHEL8

```
git checkout rhel8
```

Build the app and create the .nupgk artifact

```
./build.sh
./build.sh QuickPack
```
