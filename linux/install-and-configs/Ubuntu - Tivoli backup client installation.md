# Ubuntu - Tivoli backup client installation

How to install and configure the Tivoli backup client on an Ubuntu Linux machine.


Ubuntu is not a linux distribution that is supported by IBM for their Tivoli
backup client. This information was pulled together from internet sources
[here](http://www.rocko.me/?p=82) and [here](http://open-systems.ufl.edu/ubuntu_client)
and the server logs from my colleague, Mike Hand's, Ubuntu installation.


## Create the Debian install files

You will need to convert the RPM install files into Debian install files; the
process of which is described in "Tivoli - creating debian install files.md".


## Installation

With the Debian packages created, now we can install the Tivoli client.

    sudo dpkg -i TIVsm-API-6.1.3.deb
    sudo dpkg -i TIVsm-BA-6.1.3.deb

The client is now installed in the _/opt/tivoli_ directory.

We must tell linux where to find the Tivoli library files:

    sudo vi /etc/ld.so.conf.d/tivoli.conf

This file should contain:

    /opt/tivoli/tsm/client/api/bin

Then update the database:

    sudo ldconfig


## Test the installation

Executing:

    sudo dsmc incremental

Should result in:

    ANS0102W Unable to open the message repository /opt/tivoli/tsm/client/ba/bin/EN_US/dsmclientV3.cat.
    The American English repository will be used instead.


## Configuring the client to use ITS's backup server

Create the _dsm.opt_ file:

    cd /opt/tivoli/tsm/client/ba/bin
    sudo cp dsm.opt.smp dsm.opt
    sudo vi dsm.opt

The file should contain the following:

    ************************************************************************
    * IBM Tivoli Storage Manager                                           *
    ************************************************************************
    * This file contains an option you can use to specify the TSM
    * server to contact if more than one is defined in your client
    * system options file (dsm.sys).  Copy dsm.opt.smp to dsm.opt.
    * If you enter a server name for the option below, remove the
    * leading asterisk (*).
    ************************************************************************

    Servername        backup.aset.psu.edu
    COMPRESSALWAYS    NO
    ARCHSYMLINKASFILE NO
    DOMAIN            ALL-LOCAL

Then create the _dsm.sys_ file:

    sudo cp dsm.sys.smp dsm.sys
    sudo vi dsm.sys

The file should contain the following:

    ************************************************************************
    * IBM Tivoli Storage Manager                                           *
    ************************************************************************
    * This file contains the minimum options required to get started
    * using TSM.  Copy dsm.sys.smp to dsm.sys. In the dsm.sys file,
    * enter the appropriate values for each option listed below and
    * remove the leading asterisk (*) for each one.
    *
    * If your client node communicates with multiple TSM servers, be
    * sure to add a stanza, beginning with the SERVERNAME option, for
    * each additional server.
    ************************************************************************

    Servername      backup.aset.psu.edu
    USERS           jsiegle
    SCHEDLOGNAME    "/var/log/dsmsched.log"
    ERRORLOGNAME    "/var/log/dsmerror.log"
    INCLEXCL        /opt/tivoli/tsm/client/ba/bin/tsm.backuplist
    TCPWINDOWSIZE   1024
    TCPBUFFSIZE     512
    COMPRESSION     YES
    PASSWORDDIR     /opt/tivoli/tsm/client/ba/bin/
    PASSWORDACCESS  GENERATE
    NODENAME        TEST.HUCK.PSU.EDU
    COMMmethod TCPip
    TCPPort 1500
    TCPServeraddress saverestore.its.psu.edu

Create the backup list of include/exclude files:

    sudo vi tsm.backuplist

The file should contain roughly this list to backup the /backup and
/usr/local/scripts directories:

    EXCLUDE.DIR /bin/.../*
    EXCLUDE.DIR /boot/.../*
    EXCLUDE.DIR /dev/.../*
    EXCLUDE.DIR /etc/.../*
    EXCLUDE.DIR /home/.../*
    EXCLUDE.DIR /lib/.../*
    EXCLUDE.DIR /lost+found/.../*
    EXCLUDE.DIR /media/.../*
    EXCLUDE.DIR /mnt/.../*
    EXCLUDE.DIR /opt/.../*
    EXCLUDE.DIR /proc/.../*
    EXCLUDE.DIR /root/.../*
    EXCLUDE.DIR /sbin/.../*
    EXCLUDE.DIR /selinux/.../*
    EXCLUDE.DIR /srv/.../*
    EXCLUDE.DIR /sys/.../*
    EXCLUDE.DIR /tpm/.../*
    EXCLUDE.DIR /usr/.../*
    EXCLUDE.DIR /var/.../*
    EXCLUDE /bin/*
    EXCLUDE /boot/*
    EXCLUDE /dev/*
    EXCLUDE /etc/*
    EXCLUDE /home/*
    EXCLUDE /lib/*
    EXCLUDE /lost+found/*
    EXCLUDE /media/*
    EXCLUDE /mnt/*
    EXCLUDE /opt/*
    EXCLUDE /proc/*
    EXCLUDE /root/*
    EXCLUDE /sbin/*
    EXCLUDE /selinux/*
    EXCLUDE /srv/*
    EXCLUDE /sys/*
    EXCLUDE /tpm/*
    EXCLUDE /usr/*
    EXCLUDE /var/*
    EXCLUDE /*
    INCLUDE /backup/ -subdir=yes
    INCLUDE /usr/local/scripts/ -subdir=yes
