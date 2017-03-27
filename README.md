# rhel-base-images
# steps to create base images for rhel

Reference: http://cloudgeekz.com/625/howto-create-a-docker-image-for-rhel.html

RHEL docker images are not available from Docker Hub. It’s available from Redhat docker registry and the catalog can be accessed here. However there could be cases where you might want to create a RHEL docker image from scratch. The following steps will guide you to create a RHEL docker image from scratch. While these steps are for creating RHEL 7.1 LE docker image on PowerPC servers, the same can be applied for creating RHEL docker image on Intel servers as well.

The steps below have been performed on an Ubuntu 15.10 OS running as a PowerKVM guest. However you can run the same steps on RHEL or Fedora as well.

## Pre-requisites

1. Access to RHEL package repository. You can use a local repository created from RHEL installation ISO as described below

```
    # mkdir -p /mnt/rhel7-repo
    # mount -o loop RHEL-LE-7.1-20150219.1-Server-ppc64le-dvd1.iso /mnt/rhel7-repo/
```
    
2. Install `yum` package. This is not required on a RHEL or Fedora system as it’ll be there by default.

```
    # apt-get install yum
```

## Steps to create a RHEL docker image

1. Create a new RPM root directory

```
    # mkdir /rhel7-root
    # export rpm_root=/rhel7-root
```

2. Initialize the new RPM root directory

```
    # rpm --root ${rpm_root} --initdb
```

    The rest of the install instructions will use this new RPM root as the install destination for a minimalistic RHEL OS.
    
3. Install the redhat-release-server rpm package for RHEL 7.1 LE

```
    # rpm --root ${rpm_root} -ivh /mnt/rhel7-repo/Packages/redhat-release-server-7.1-1.ael7b.ppc64le.rpm
```

4. Configure yum repositories as required.
   For example if you want to use only the local repository, then do the following

```
    # rm -f ${rpm_root}/etc/yum.repos.d/*.repo
    # cat >${rpm_root}/etc/yum.repos.d/rhel71le.repo<<EOF
    [rhel71le]
    baseurl=file:///mnt/rhel7-repo
    enabled=1
    EOF
```

5. If you encounter ‘No such file or directory’ error, then you’ll need to create the /etc/yum.repos.d directory.
   
   Import GPG keys

```
    # rpm --root ${rpm_root} --import  /mnt/rhel7-repo/RPM-GPG-KEY-redhat-*
```

6. Install minimalistic RHEL OS

```
    # yum -y --installroot=${rpm_root} install yum
```

    This will install a minimalistic RHEL OS under /rhel7-root
7. Additional Customization
    Chroot to the new RHEL installation and perform any additional customizations if required
```
    # chroot ${rpm_root} /bin/bash
    bash-4.2# cat /etc/redhat-release 
    Red Hat Enterprise Linux Server release 7.1 (Maipo)
```
   
8. Convert this RHEL installation to a docker image

```
    # tar -C ${rpm_root}/ -c . | docker import - rhel7le
    9a6dac402e2bf561f833f644337f3061c6ff70ec906472d5c008755ca9ed7f1d
    # docker images
    REPOSITORY            TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
    rhel7le               latest              9a6dac402e2b        11 seconds ago      360.6 MB
```

    Use the new RHEL docker image

```
    # docker run --hostname='rhel7-container' rhel7le uname -a
    Linux rhel7-container 4.2.0-10-generic #12-Ubuntu SMP Tue Sep 15 19:46:04 UTC 2015 ppc64le ppc64le ppc64le GNU/Linux
```

Once you have created the RHEL docker image, you can push it to your local registry for sharing. Hope this will be of some help to you in your docker journey.
