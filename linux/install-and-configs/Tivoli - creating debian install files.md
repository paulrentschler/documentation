# Tivoli - Creating Debian install files

How to convert the RPM install files provided for the Tivoli linux client into
Debian install files for use with Debian or Ubuntu.


Ubuntu is not a linux distribution that is supported by IBM for their Tivoli
backup client. This information was pulled together from internet sources
[here](http://www.rocko.me/?p=82) and [here](http://open-systems.ufl.edu/ubuntu_client).


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

    sudo alien -g TIVsm-API64.x86_64.rpm TIVsm-BA.x86_64.rpm


## Fix the RPM vs DEB incompatibilities

Before we repackage it, we need to edit the control files as they are not
properly formatted.

### TIMsv-API

Edit the file:

    sudo vi TIVsm-API64-6.3.2/debian/control

It should look like this:

    Source: tivsm-api
    Section: alien
    Priority: extra
    Maintainer: Paul Rentschler <par117@psu.edu>
    Package: tivsm-api
    Architecture: amd64
    Description: the API IBM Tivoli Storage Manager API
    Version: 6.3.2.0

### TIVsm-BA

Edit the file:

    sudo vi TIVsm-BA-6.3.2/debian/control

It should look like this:

    Source: tivsm-ba
    Section: alien
    Priority: extra
    Maintainer: Paul Rentschler <par117@psu.edu>
    Package: tivsm-ba
    Architecture: amd64
    Description: the Backup Archive Client IBM Tivoli Storage Manager Client
    Version: 6.3.2.0


### Fix the permissions

The _postinst_ script must be executable:

    sudo chmod 755 TIVsm-API64-6.3.2/debian/postinst
    sudo chmod 755 TIVsm-BA-6.3.2/debian/postinst


## Create the Debian install files

Rename the "debian" directories as they are expected to be uppercase:

    sudo mv TIVsm-API64-6.3.2/debian TIVsm-API64-6.3.2/DEBIAN
    sudo mv TIVsm-BA-6.3.2/debian TIVsm-BA-6.3.2/DEBIAN

Create the debian packages:

    sudo dpkg -b TIVsm-API64-6.3.2
    sudo dpkg -b TIVsm-BA-6.3.2

These commands will result in the following two messages indicating that the
Debian packages were created:

    dpkg-deb: building package `tivsm-api64' in `TIVsm-API64-6.3.2.deb'.
    dpkg-deb: building package `tivsm-ba' in `TIVsm-BA-6.3.2.deb'.

You now have Tivoli backup client installation files that will work with
Debian packaging. See "Ubuntu - Tivoli backup client installation.md" for
instructions on how to install these client files.
