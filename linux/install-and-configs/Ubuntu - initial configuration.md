# Ubuntu: initial configuration

Documentation for initial configuration of an Ubuntu virtual server as part of the Standard Linux VM build


## VM creation and Ubuntu installation

The provisioning of a new VM and base installation of Ubuntu are not covered by this document. [Generic Ubuntu server installation instructions](https://help.ubuntu.com/community/Installation#Server_and_network_installations) are available online.

We normally only install long term support (LTS) versions of Ubuntu which are released on even years in April. The [most recent LTS releases](http://releases.ubuntu.com/) as of this document's writing were <em>Ubuntu 10.04 LTS Lucid Lynx</em> and <em>Ubuntu 12.04 LTS Precise Pangolin</em>. Ubuntu releases can be downloaded from the [Penn State mirror site](http://mirror.aset.psu.edu/pub/linux/distributions/ubuntu-releases/).

_Note: This document assumes you have installed no packages during the normal installation process._


## Installing SSH access

Install the SSH server with the following command:

    sudo aptitude install openssh-server

Edit the configuration file and make the following changes:

    sudo vi /etc/ssh/sshd_config

* Change _Port 22_ to _Port 73_
* Change _PermitRootLogin yes_ to _PermitRootLogin no_

Save the configuration file and then use this command to restart the ssh server:

    sudo service ssh restart

Verify that the ssh server is running and listening on port 73 using this command:

    netstat -an | more

_Note: the readout should show that the ssh server is listening on Port 73 and nothing is listening on Port 22._


## Configuring the IP address

_Note: If you configured a static IP address during the Ubuntu installation, you can skip this step_

Edit the network interfaces file using this command:

    sudo vi /etc/network/interfaces

Replace the following line:

    iface eth0 inet dhcp</pre>

With the lines below:

    iface eth0 inet static
        address 10.0.0.10
        netmask 255.255.255.0
        network 10.0.0.0
        broadcast 10.0.0.255
        gateway 10.0.0.1

_Note: the last number (octet) in "10.0.0.10" should be changed to an available IP address._

Configure the DNS resolution by entering:

    sudo vi /etc/resolvconf/resolv.conf.d/tail

Add the following to the file:

    nameserver 128.118.25.3
    nameserver 128.118.141.32

Restart the server using the command below for the changes to take effect

    sudo /sbin/shutdown -r now


## Add support for building packages

Enter the following command:

    sudo aptitude install build-essential


## Synchronize clock to internet time server

Install the NTP time server using this command:

    sudo aptitude install ntp

Edit the configuration file:

    sudo vi /etc/ntp.conf

Add _clock.psu.edu_ to the NTP server pool section, which should then look like this:

    # Use servers from the NTP Pool Project. Approved by Ubuntu Technical Board
    # on 2011-02-08 (LP: #104525). See <a href="http://www.pool.ntp.org/join.html">http://www.pool.ntp.org/join.html</a> for
    # more information.
    server clock.psu.edu
    server 0.ubuntu.pool.ntp.org
    server 1.ubuntu.pool.ntp.org
    server 2.ubuntu.pool.ntp.org
    server 3.ubuntu.pool.ntp.org

Restart the time server with the following command:

    sudo service ntp restart


## Set a minimum password length

Open the password configuration file:

    sudo vi /etc/pam.d/common-password

Find this line (in 12.04, line 25) and add the text _min=8_ to set the minimum password length:

    password   [success=1 default=ignore]   pam_unix.so obscure sha512 _min=8_
