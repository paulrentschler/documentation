# Tivoli - Creating Debian install files

How to convert the RPM install files provided for the Tivoli linux client into
Debian install files for use with Debian or Ubuntu.


Ubuntu is not a linux distribution that is supported by IBM for their Tivoli
backup client. This information was pulled together from internet sources
[here](http://www.rocko.me/?p=82) and [here](http://open-systems.ufl.edu/ubuntu_client).
Additional assistance with the newer 6.3.x versions was obtained from
[here](https://kb.berkeley.edu/page.php?id=27401).


## Install prerequisites

Install alien, which allows you to install RPM packages on a Debian/Ubuntu
machine, and some other supporting packages:

    sudo aptitude install alien


## Deconstruct the RPMs

Get the [Tivoli linux client software](http://www-01.ibm.com/support/docview.wss?uid=swg21239415)
from IBM and untar it.

    cd /usr/local/src
    mkdir tivoli
    sudo chown myusername:myusername tivoli
    cd tivoli
    cp ~/6.3.2.0-TIV-TSMBAC-LinuxX86.tar ./
    tar -xvf 6.3.2.0-TIV-TSMBAC-LinuxX86.tar

We will now use alien to unpack the RPM file so we can repackage it as a
Debian .deb package:

    sudo alien -g TIVsm-API64.x86_64.rpm
    sudo alien -g TIVsm-BA.x86_64.rpm
    sudo alien -g gskcrypt64-8.0.14.14.linux.x86_64.rpm
    sudo alien -g gskssl64-8.0.14.14.linux.x86_64.rpm


## Fix the RPM vs DEB incompatibilities

Before we repackage it, we need to edit the control files as they are not
properly formatted.

### TIMsv-API64

Edit the file:

    sudo vi TIVsm-API64-6.3.2/debian/control

It should look like this:

    Source: tivsm-api
    Section: alien
    Priority: extra
    Maintainer: Paul Rentschler <paul@rentschler.ws>
    Package: tivsm-api
    Architecture: amd64
    Depends:
    Description: the API IBM Tivoli Storage Manager API
    Version: 6.3.2

### TIVsm-BA

Edit the file:

    sudo vi TIVsm-BA-6.3.2/debian/control

It should look like this:

    Source: tivsm-ba
    Section: alien
    Priority: extra
    Maintainer: Paul Rentschler <paul@rentschler.ws>
    Package: tivsm-ba
    Architecture: amd64
    Depends:
    Description: the Backup Archive Client IBM Tivoli Storage Manager Client
    Version: 6.3.2

### GSKcrypt64

Edit the file:

    sudo vi gskcrypt64-8.0/debian/control

It should look like this:

    Source: gskcrypt64
    Section: alien
    Priority: extra
    Maintainer: Paul Rentschler <paul@rentschler.ws>
    Package: gskcrypt64
    Architecture: amd64
    Depends:
    Description: IBM GSKit Cryptography Runtime
    Version: 8.0

### GSKssl64

Edit the file:

    sudo vi gskssl64-8.0/debian/control

It should look like this:

    Source: gskssl64
    Section: alien
    Priority: extra
    Maintainer: Paul Rentschler <paul@rentschler.ws>
    Package: gskssl64
    Architecture: amd64
    Depends:
    Description: IBM GSKit SSL Runtime With Acme Toolkit
    Version: 8.0


### Fix the permissions

The _postinst_ script must be executable:

    sudo chmod 755 TIVsm-API64-6.3.2/debian/postinst
    sudo chmod 755 TIVsm-BA-6.3.2/debian/postinst
    sudo chmod 755 gskcrypt64-8.0/debian/postinst
    sudo chmod 755 gskssl64-8.0/debian/postinst


## Create the Debian install files

Rename the "debian" directories as they are expected to be uppercase:

    sudo mv TIVsm-API64-6.3.2/debian TIVsm-API64-6.3.2/DEBIAN
    sudo mv TIVsm-BA-6.3.2/debian TIVsm-BA-6.3.2/DEBIAN
    sudo mv gskcrypt64-8.0/debian gskcrypt64-8.0/DEBIAN
    sudo mv gskssl64-8.0/debian gskssl64-8.0/DEBIAN

Create the debian packages:

    sudo dpkg -b TIVsm-API64-6.3.2
    sudo dpkg -b TIVsm-BA-6.3.2
    sudo dpkg -b gskcrypt64-8.0
    sudo dpkg -b gskssl64-8.0

These commands will result in the following two messages indicating that the
Debian packages were created:

    dpkg-deb: building package `tivsm-api64' in `TIVsm-API64-6.3.2.deb'.
    dpkg-deb: building package `tivsm-ba' in `TIVsm-BA-6.3.2.deb'.
    dpkg-deb: building package `gskcrypt64' in `gskcrypt64-8.0.deb'.
    dpkg-deb: building package `gskssl64' in `gskssl64-8.0.deb'.

You now have Tivoli backup client installation files that will work with
Debian packaging. See "Ubuntu - Tivoli backup client installation.md" for
instructions on how to install these client files.
