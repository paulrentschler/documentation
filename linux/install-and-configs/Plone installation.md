# Plone installation

Documentation for installing Plone as part of a standard Linux VM build

[Plone](http://plone.org) is an enterprise grade, open source content management system (CMS) written in [Python](http://python.org) on the [Zope Application Framework](http://zope.org).

## Pre-install preparation

Install the required packages by running the following commands:

### Ubuntu

    sudo apt-get install build-essential
    sudo apt-get install libssl-dev
    sudo apt-get install libxml2-dev
    sudo apt-get install libxslt1-dev
    sudo apt-get install libbz2-dev
    sudo apt-get install libjpeg62-dev
    sudo apt-get install libreadline-gplv2-dev
    sudo apt-get install wv
    sudo apt-get install poppler-utils


### RedHat / CentOS

    sudo yum groupinstall "Development Tools"
    sudo yum install openssl-devel
    sudo yum install libxml2-devel
    sudo yum install libxslt-devel
    sudo yum install bzip2-devel
    sudo yum install openjpeg-devel
    sudo yum install readline-devel
    sudo yum install wv
    sudo yum install poppler-utils


**Note:** the above commands were shamelessly stolen from [plone.org](http://plone.org/documentation/manual/installing-plone/installing-on-linux-unix-bsd/debian-libraries)


### Install Python 2.7 on RedHat / CentOS 6

As of September 2013, version 4.3.2 is the newest and requires Python 2.7 which is not what is installed with RedHat / CentOS 6.

Install more dependencies needed for compiling python:

    sudo yum install sqlite-devel tk-devel

Download and unpack Python 2.7

    cd /usr/local/src
    wget http://python.org/ftp/python/2.7.3/Python-2.7.3.tar.bz2
    tar xvf Python-2.7.3.tar.bz2

Compile python

    cd Python-2.7.3
    ./configure --prefix=/usr/local
    sudo make
    sudo make altinstall

Python 2.7 is now installed in /usr/local/bin as python2.7.



## Installation

[Copy the link to the universal installer from plone.org](http://plone.org/products)

Then enter the following commands, substituting the link to the universal installer in the wget command, below:

    cd /usr/local/src
    sudo chown par117:adm .
    wget https://launchpad.net/plone/4.3/4.3.2/+download/Plone-4.3.2-UnifiedInstaller.tgz
    tar -zxvf Plone-4.3.2-UnifiedInstaller-20120604.tgz

Run the universal installer by entering the following commands:

    cd Plone-4.3.2-UnifiedInstaller/
    sudo ./install.sh zeo --target=/opt/plone-4.3.2 --with-python=/usr/local/bin/python2.7 --static-lxml=yes

When the installer has finished running, create a symlink to the current version of Plone by entering the following commands:

    cd /opt/
    sudo ln -s plone-4.3.2 current-plone



## Setup the start script and make it autostart on bootup

Copy the start script using the following command:

    sudo cp /opt/current-plone/zeocluster/bin/plonectl /etc/init.d

Then edit the start script changing any "_plone-4.3.2_" references to "_current-plone_" using the following commands:

    sudo vi /etc/init.d/plonectl
    :%s/plone-4.3.2/current-plone/g
    :wq

Set Plone to autostart on bootup by entering the following command:

    sudo update-rc.d plonectl defaults

Then start Plone:

    sudo service plonectl start
