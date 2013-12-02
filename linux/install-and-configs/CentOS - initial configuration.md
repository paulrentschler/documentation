# CentOS: initial configuration

Documentation for initial configuration of a CentOS virtual server as part of the Standard Linux VM build

Community ENTerprise Operating System (CentOS)


## VM creation and CentOS installation

The provisioning of a new VM and base installation of CentOS are not covered by this document. [Generic CentOS installation instructions](http://www.tecmint.com/centos-6-4-installation-guide-with-screenshots/) are available online.

CentOS releases can be downloaded from the [Penn State mirror site](http://mirror.aset.psu.edu/pub/linux/distributions/centos/)

_Note: This document assumes you did a "minimal" installation of CentOS which is the default._

_Note: This document assumes you are logged in as the root user._


## Ensure the LAN connections work

For some reason the network connection(s) are not enabled when CentOS is initially installed.

### Adapter 1: NAT connection

The default network adaptor added by VirtualBox is a NAT connection with a DHCP assigned IP address. This works great for ensuring that the VM has access to the greater internet, it just needs to be enabled.

Edit /etc/sysconfig/network-scripts/ifcfg-eth0 and change the ONBOOT entry to yes so it looks like this:

    ONBOOT=yes


### Adapter 2: Host-only

If you need to be able to access the VM from the host computer system and in most cases you will; you will need to define a second network adapter that uses the Host-only Adapter connection with a statically assigned IP address.

Edit /etc/sysconfig/network-scripts/ifcfg-eth1 and change the ONBOOT entry as well as adding the static IP lines which look like this:

    BOOTPROTO=none
    IPADDR=192.168.236.10
    NETMASK=255.255.255.0

In the above IP address, we use the private IP space that starts with 192.168.0.0, the third octet (236) is defined by the VirtualBox install and you assign a machine specific fourth octet (10). Based on the network mask provided, only the fourth octet needs to be unique among the VMs on your host machine.


### Adapter 3: Bridged

If you will need to provide access to this VM from other machines on the network (outside of the host computer); you will need to define a third network adapter that uses the Bridge connection with an IP address assigned just like your host machine receive's it's IP address.


### Restart the network services

Restart the network services to have the changes take effect using the following command:

    service network restart



## Secure SSH access

The SSH server is installed by default CentOS installation but it runs on port 22 which is the standard port for SSH traffic. It is a good idea to move the SSH server to a different port to deter brute force SSH attacks.

Edit the configuration file and make the following changes:

    vi /etc/ssh/sshd_config

* Change _#Port 22_ to _Port 1855_
* Change _#PermitRootLogin yes_ to _PermitRootLogin no_

Save the configuration file and then use this command to restart the ssh server:

    service sshd restart

Verify that the ssh server is running and listening on port 1855 using this command:

    netstat -an | more

_Note: the readout should show that the ssh server is listening on Port 1855 and nothing is listening on Port 22._

The last thing we need to do is change the IP Tables rules to allow connections on port 1855 instead of port 22. Edit the /etc/sysconfig/iptables file and change this line:

    -A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT

To look like this:

    -A INPUT -m state --state NEW -m tcp -p tcp --dport 1855 -j ACCEPT

All we changed was the --dport value from 22 to 1855. Now restart the IP Tables with the following command:

    service iptables restart



## Add support for building packages

Enter the following command:

    yum groupinstall "Development Tools"



## Install some typical tools

Install wget:

    yum install wget

Install man:

    yum install man



## Install any updates

It's important to keep all of the software installed up-to-date, to do this:

    yum update



## Synchronize clock to internet time server

Install the NTP time server using this command:

    yum install ntp

Edit the configuration file:

    vi /etc/ntp.conf

Add _clock.psu.edu_ to the NTP server pool section, which should then look like this:

    # Use servers from the pool.ntp.org project.
    # Please consider joining the pool (http://www.pool.ntp.org/join.html).
    server clock.psu.edu
    server 0.centos.pool.ntp.org
    server 1.centos.pool.ntp.org
    server 2.centos.pool.ntp.org

Restart the time server with the following command:

    service ntpd restart



## Disable SELinux

SELinux is an excellent security tool but it tends to create a lot more headaches that it solves problems. Depending on the type of server you are running and your concern for security, disabling it might be the best option.

**Note: disabling SELinux reduces the security of the machine!**

Edit the SELinux configuration file using:

    sudo vi /etc/sysconfig/selinux

Change the configuration file so it looks like so:

    # This file controls the state of SELinux on the system.
    # SELINUX= can take one of these three values:
    #     enforcing - SELinux security policy is enforced.
    #     permissive - SELinux prints warnings instead of enforcing.
    #     disabled - No SELinux policy is loaded.
    #SELINUX=enforcing
    SELINUX=disabled
    # SELINUXTYPE= can take one of these two values:
    #     targeted - Targeted processes are protected,
    #     mls - Multi Level Security protection.
    SELINUXTYPE=targeted

Save the changes and the server will need to be rebooted before SELinux is disabled. You can wait to reboot the server until the end of this document.



## Assign sudo rights to the sudo group

Ubuntu has this great concept of creating a group (sudo) that has full sudo rights. This way all that is necessary to grant a user full sudo rights is to add them to the group rather than having to maintain the list of users in the sudoers file.

Edit the /etc/sudoers file and look for this line:

    root    ALL=(ALL)       ALL

Just below that line, add the following lines:

    ## Allow members of the 'sudo' group to run all commands
    %sudo   ALL=(ALL)       ALL



## Lock down the root account

Being able to log in as the root user is dangerous. It invites password sharing which means that the actions performed as the root user cannot be attributed to a single individual; thus it not recommended. Instead, use of the **sudo** command should be utilized by users who all have individual accounts and never share passwords. CentOS does not install this way; in fact the only account created by the install process is the root user. We're going to fix that.

### Creating a non-root account

Log into the new CentOS installation as the root user.

Create a group for assigning full sudo usage with the following command:

    groupadd --system sudo

Create a non-root user for yourself and set the password with the following commands:

    useradd -m -G adm,sudo -s /bin/bash -c "John Doe" jxd123
    passwd jxd123

Now log out as root and log back into the system using your new non-root user account.


### Verifying the non-root account has sudo permissions

Lets make sure your non-root user account has sudo permissions by entering the following command:

    cat /etc/shadow

You should get a permission denied message, then enter the following command:

    sudo !!

And you should see the contents of the /etc/shadow file. If you do, your sudo permissions are working and we can move on.

**Tip:** the "sudo !!" command is extremely handly as it executes the previous command "cat /etc/shadow" in our case but prepends it with the sudo command. Extremely useful in this situation where you execute a command but it turns out you need to execute it with sudo instead.


### Disabling the root account

First thing we want to do is create a long, complicated password for the root account. Generate one using the following command:

    date +%s | sha256sum | base64 | head -c 32 ; echo

Copy the resulting string and use it as the new password when you change root's password using the following command:

    sudo passwd root

Now edit the /etc/passwd file and change root's shell from /bin/bash to /sbin/nologin. The line should look like this:

    root:x:0:0:root:/root:/sbin/nologin



**The root user is now disabled, log in using individual, non-root accounts and sudo commands that need greater permissions.**
