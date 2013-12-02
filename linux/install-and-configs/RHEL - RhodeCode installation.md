# RhodeCode installation on RedHat EL6

## A. Preparations

1. Download & install Python:

		$ sudo yum install python python-devel

1. Install _pip_ which is a tool for installing and managing Python
	packages, such as those found in the Python Package Index (pypi):

		$ cd /tmp
		$ wget http://mirror-fpt-telecom.fpt.net/fedora/epel/6/i386/epel-release-6-8.noarch.rpm
		$ sudo rpm -ivh epel-release-6-8.noarch.rpm
		$ sudo yum install python-pip

	Please note that the name of the command is _pip-python_ under RHEL
	and friends. It's recommended that you add the following alias to
	your ~/.bashrc file:

		$ echo 'alias pip="/usr/bin/pip-python"' >> $HOME/.bashrc
		$ . $HOME/.bashrc

1. Install the basic development tools and Virtualenv which is a great 
    sandboxing system for Python libraries and its usage is highly
    recommended:

    	$ sudo yum groupinstall "Development Tools"
    	$ sudo pip install -i https://pypi.rhodecode.com/ --upgrade pip
		$ sudo pip install virtualenv

1. Create the RhodeCode Enterprise directory along with a directory for
    the repository files and a new Virtualenv sandbox for all RhodeCode
    Enterprise libraries:

		$ sudo mkdir -p /opt/rhodecode/repo
		$ sudo chown -R <your username> /opt/rhodecode
		$ virtualenv --no-site-packages /opt/rhodecode/venv

1. Activate the newly created Virtualenv:

		$ source /opt/rhodecode/venv/bin/activate

	Please keep in mind. Whenever you install or upgrade RhodeCode you
	always need to activate its Virtualenv so that all chagnes are done
	inside the sandbox. To exit a snadbox type 'deactivate'.


## B. Download & Install RhodeCode Enterprise

You will now download the latest stable release of RhodeCode Enterprise from
their PYPI server with the help of the Python Installer Pip.

1. If not already done, please activate your RhodeCode Enterprise
	Virtualenv sandbox:

		$ source /opt/rhodecode/venv/bin/activate

1. Download RhodeCode Enterprise and all its dependencies:

		$ sudo pip install https://rhodecode.com/dl/latest

This step will take some time. If an error like "The read operation timed out
while getting" occurs then please wait a few minutes and run the command again.
The error means that on eof the dependency servers is down or overloaded.


## C. Go Ahead

This finishes the downloading and installing of RhodeCode Enterprise and its
core dependencies. Now continue on to
[setting up your new installation](https://rhodecode.com/help/kb/installation-setup/basic-setup-ubuntu-linux)
