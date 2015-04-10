# Tivoli - Creating Debian install files

How to convert the RPM install files provided for the Tivoli linux client into
Debian install files for use with Debian or Ubuntu.


Ubuntu is not a linux distribution that is supported by IBM for their Tivoli
backup client. This information was pulled together from internet sources
[here](http://www.rocko.me/?p=82) and [here](http://open-systems.ufl.edu/ubuntu_client).


## Install prerequisites

Install alien, which allows you to install RPM packages on a Debian/Ubuntu
machine, and some other supporting packages:

    sudo aptitude install alien libstdc++6 ksh ia32-libs


## Deconstruct the RPMs

Get the [Tivoli linux client software](ftp://ftp.aset.psu.edu/pub/tivoli-storage-management/maintenance/client/v6r1/Linux/LinuxX86/v613/6.1.3.0-TIV-TSMBAC-LinuxX86.tar)
from Penn State's repository and untar it. Version 6.3.1.0 was the newest
version available.

    cd /usr/local/src
    mkdir tivoli
    cd tivoli
    wget ftp://ftp.aset.psu.edu/pub/tivoli-storage-management/maintenance/client/v6r1/Linux/LinuxX86/v613/6.1.3.0-TIV-TSMBAC-LinuxX86.tar
    tar -xvf 6.1.3.0-TIV-TSMBAC-LinuxX86.tar

We will now use alien to unpack the RPM file so we can repackage it as a
Debian .deb package:

    sudo alien -g TIVsm-API.i386.rpm TIVsm-BA.i386.rpm


## Fix the RPM vs DEB incompatibilities

Before we repackage it, we need to edit the control files as they are not
properly formatted.

### TIMsv-API

Edit the file:

    sudo vi TIVsm-API-6.1.3/debian/control

It should look like this:

    Source: tivsm-api
    Section: alien
    Priority: extra
    Maintainer: Paul Rentschler <par117@psu.edu>
    Package: tivsm-api
    Architecture: amd64
    Description: the API IBM Tivoli Storage Manager API
    Version: 6.1.3.0

### TIVsm-BA

Edit the file:

    sudo vi TIVsm-BA-6.1.3/debian/control

It should look like this:

    Source: tivsm-ba
    Section: alien
    Priority: extra
    Maintainer: Paul Rentschler <par117@psu.edu>
    Package: tivsm-ba
    Architecture: amd64
    Description: the Backup Archive Client IBM Tivoli Storage Manager Client
    Version: 6.1.3.0


## Create the Debian install files

Rename the "debian" directories as they are expected to be uppercase:

    sudo mv TIVsm-API-6.1.3/debian TIVsm-API-6.1.3/DEBIAN
    sudo mv TIVsm-BA-6.1.3/debian TIVsm-BA-6.1.3/DEBIAN

Create the debian packages:

    sudo dpkg -b TIVsm-API-6.1.3
    sudo dpkg -b TIVsm-BA-6.1.3

These commands will result in the following two messages indicating that the
Debian packages were created:

    dpkg-deb: building package `tivsm-api' in `TIVsm-API-6.1.3.deb'.
    dpkg-deb: building package `tivsm-ba' in `TIVsm-BA-6.1.3.deb'.

You now have Tivoli backup client installation files that will work with
Debian packaging.
