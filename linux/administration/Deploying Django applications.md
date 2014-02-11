# Deploying Django applications

This document talks about how to take a Django application and it's associated
modules and deploy it into a production environment on a linux server backed
by Apache.


## Prerequisites

### Installing VirtualEnv

**to do**


### MySQL development library

If the application uses MySQL as the database, the development libraries will
need to be installed using the following on RedHat/CentOS:

    sudo yum install mysql-devel


## Setup the VirtualEnv

If the Apache setup is being followed from the **Apache installation.md**
documentation, then the hosting structure of:

    /home/httpd/domain.tld/[hostname]/html

for the document root of each website should exist or something similar to this.

Following this model, create a directory for the Django project's virtual
environment along the lines of:

    /home/httpd/domain.tld/[hostname]/[django project]

Using the command:

    virtualenv --no-site-packages /home/httpd/domain.tld/[hostname]/[django project]

Activate the virtualenv:

    source /home/httpd/domain.tld/[hostname]/[django project]/bin/activate


## Install the application and modules

### Required python modules

Clone the Django application project from the revision control repository into
a temporary directory so the requirements.txt file can be used.

    cd ~
    mkdir tmp
    cd tmp
    git clone [django application repo path]

Install all the required packages into the virtualenv

    cd /home/httpd/domain.tld/[hostname]/[django project]
    ./bin/pip install -r ~/tmp/[django application]/requirements.txt


### Django application

Create a new Django project

    cd /home/httpd/domain.tld/[hostname]/[django project]
    django-admin.py startproject [django application]

Clone the Django application project into the virtualenv.

    cd [django application]
    rm -rf [django application]
    git clone [django application repo path]

If the new Django application directory as dashes in them, rename it to change
the dashes (-) to underscores (_).

Symlink the WSGI file

    ln -s [django application]/wsgi.py ./


### Django modules

Clone the Django modules into the project

    cd /home/httpd/domain.tld/[hostname]/[django project]/[django application]
    git clone [django module repo path]

If any of the new Django module directories have dashes in them, rename them
to change the dashes (-) to underscores (_).



## Configure the application and modules

### Create a local copy of settings.py

The settings.py file is not included in the repositories because it contains
confidential settings that are server specific. As such, a copy of the
distribution file must be made and settings changed.

    cd /home/httpd/domain.tld/[hostname]/[django project]/[django application]
    cd [django application]
    cp settings.dist settings.py
    vi settings.py


### Setup the database

Create a new database based on the name specified in the settings.py file and
grant the database user specified in the settings.py file full access to it.

Then create the necessary tables and perform any migrations (if using South)
with the following:

    cd /home/httpd/domain.tld/[hostname]/[django project]/[django application]
    python manage.py syncdb
    python manage.py migrate


### Rebuild the Elastic Search index (if appropriate)

If the application uses Elastic Search and the Haystack Django module, the
search index will need to be rebuilt if any data is propopulated into the
database. Rebuild the index with:

    cd /home/httpd/domain.tld/[hostname]/[django project]/[django application]
    python manage.py rebuild_index
