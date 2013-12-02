# Apache installation

Documentation for installing and configuring Apache with PHP support as part of the Standard Linux VM build

[Apache](http://httpd.apache.org) is a secure, efficient, and extensible web server that conforms to the current HTTP protocol standards.

[PHP](http://php.net) is a widely-used general-purpose scripting language that is especially suited for web development and can be embedded into HTML or run from the command line.


## Installing the web server and PHP

### Ubuntu

Enter the following commands to install apache and php:

    sudo aptitude install apache2
    sudo aptitude install php5

Then use the following commands to install the commonly used php modules:

    sudo aptitude install php5-mysql php5-curl php5-gd php5-ldap php5-mcrypt


### RedHat / CentOS

Enter the following commands to install apache and php:

    sudo yum install httpd
    sudo yum install php

Install the EPEL repository:

    wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
    wget http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
    sudo rpm -Uvh remi-release-6*.rpm epel-release-6*.rpm
    rm epel-release-6.8.noarch.rpm remi-release-6.rpm
    sudo yum update

Then use the following commands to install the commonly used php modules:

    sudo yum install php-mysql php-gd php-ldap php5-mcrypt

Open a hole in the firewall to allow traffic to Apache by editing /etc/sysconfig/iptables and adding the following line:

    -A INPUT -m state --state NEW -m tcp -p tcp -dport 80 -j ACCEPT

It should be added near the bottom of the file in with the other INPUT statements so that the file looks something like this:

    # Firewall configuration written by system-config-firewall
    # Manual customization of this file is not recommended.
    *filter
    :INPUT ACCEPT [0:0]
    :FORWARD ACCEPT [0:0]
    :OUTPUT ACCEPT [0:0]
    -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
    -A INPUT -p icmp -j ACCEPT
    -A INPUT -i lo -j ACCEPT
    -A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
    -A INPUT -m state --state NEW -m tcp -p tcp --dport 1855 -j ACCEPT
    -A INPUT -j REJECT --reject-with icmp-host-prohibited
    -A FORWARD -j REJECT --reject-with icmp-host-prohibited
    COMMIT

Finally, start the web server using the following command:

    sudo service httpd start



## Testing the Apache/PHP installation

### Ubuntu

Enter the following to create the testme.php file for editing:

    sudo vi /var/www/testme.php


### RedHat / CentOS

Enter the following to create the testme.php file for editing:

    sudo vi /var/www/html/testme.php


In the file, enter:

    <?php echo phpinfo(); ?>

Access the following URL (replacing the IP address with the one for your server) in your browser of choice:

    http://10.0.0.15/testme.php

_Note: if everything is working properly, you should see a page displaying the information regarding your php installation._


## Setting up the directory structures

### Document root

Use the following commands to create a new directory structure for Apache's document root (make the appropriate substitutions for server name, domain name, and your user account):

    cd /home
    sudo mkdir -p httpd/domain.tld/[hostname]/html
    sudo chown -R par117:adm httpd


### Log directory

Use the following commands to create a new directory structure for Apache's log files (make appropriate substitutions for server name and domain name):

    cd /var/log
    sudo mkdir -p html/domain.tld
    sudo chown -R root:adm html
    sudo chmod 770 html



## Creating the virtual host(s)

Ubuntu's default configuration for Apache uses different files stored in the _/etc/apache2/sites-available_ directory to specify the configuration of each virtual host. These virtual hosts can then be enabled and disabled using the _a2ensite_ and _a2dissite_ commands respectively.

This method of virtual host management is far superior to RedHat/CentOS's system of defining all the virtual hosts in one file _/etc/httpd/conf.d/vhosts.conf_.

### RedHat / CentOS customizations

Before we can create the virtual host configurations, we need to implement the Ubuntu method of virtual host management on our RedHat/CentOS installation.

Edit /etc/httpd/conf/httpd.conf and update the end of the file to match the following:

    ### Section 3: Virtual Hosts
    #
    # VirtualHost: If you want to maintain multiple domains/hostnames on your
    # machine you can setup VirtualHost containers for them. Most configurations
    # use only name-based virtual hosts so the server doesn't need to worry about
    # IP addresses. This is indicated by the asterisks in the directives below.
    #
    # Please see the documentation at
    # <URL:http://httpd.apache.org/docs/2.2/vhosts/>
    # for further details before you try to setup virtual hosts.
    #
    # You may use the command line option '-S' to verify your virtual host
    # configuration.

    #
    # Use name-based virtual hosting.
    #
    #NameVirtualHost *:80
    #
    # NOTE: NameVirtualHost cannot be used without a port specifier
    # (e.g. :80) if mod_ssl is being used, due to the nature of the
    # SSL protocol.
    #

    #
    # VirtualHost example:
    # Almost any Apache directive may go into a VirtualHost container.
    # The first VirtualHost section is used for requests without a known
    # server name.
    #
    #<VirtualHost *:80>
    #    ServerAdmin webmaster@dummy-host.example.com
    #    DocumentRoot /www/docs/dummy-host.example.com
    #    ServerName dummy-host.example.com
    #    ErrorLog logs/dummy-host.example.com-error_log
    #    CustomLog logs/dummy-host.example.com-access_log common
    #</VirtualHost>


    #
    # Virtual host configurations have been broken up into separate files that are
    # stored in the /etc/httpd/sites-available directory. There is one file for
    # each virtual host and they are all prefixed with a 3-digit number so that
    # the order they are loaded in can be controled.
    #
    # Active virtual host configurations should be symlinked from the
    # /etc/httpd/sites-available directory to the /etc/httpd/sites-enabled
    # directory, while preserving the file names, where they will be processed.
    #
    #
    # Load virtual host config files from "/etc/httpd/sites-enabled".
    #
    Include sites-enabled/*

Note that everything is commented out except the last line which instructs Apache to read the configuration files from the sites-enabled directory which is where our active virtual host configuration files will be symlinked.

If the file /etc/httpd/conf.d/vhosts.conf exists, replace the existing contents with some documentation of how virtual hosts are now managed by editing the file and replacing any existing contents with the following:

    #
    # Virtual host configurations have been broken up into separate files that are
    # stored in the /etc/httpd/sites-available directory. There is one file for
    # each virtual host and they are all prefixed with a 3-digit number so that
    # the order they are loaded in can be controled.
    #
    # The main httpd.conf file is configured to read the virtual host configuration
    # files from the /etc/httpd/sites-enabled directory. Active virtual hosts
    # should be symlinked from the /etc/httpd/sites-available directory to the
    # /etc/httpd/sites-enabled directory while preserving the file names.
    #

Create the new directories using the following commands:

    cd /etc/httpd
    sudo mkdir sites-enabled sites-available

In order to make creating and deleting the symlinks between the sites-available and sites-enabled directories, Ubuntu has the commands a2ensite and a2dissite for enabling and disabling virtual hosts (or sites). This functionality can be duplicated using Paul Rentschler's [ApacheManage product](https://bitbucket.org/paulrentschler/apachemanage). Follow the installation instructions found in the readme.md file.



### Creating the virtual host configurations

Use the following commands to create regular and SSL virtual hosts for the www.domain.tld website (modify the server name and domain name information as appropriate for other websites):

    cd /etc/apache2/sites-available
    sudo vi 000-[hostname].domain.tld

In the file, enter the following:

    <VirtualHost *:80>
      ServerName [hostname].domain.tld
      ServerAdmin webteam@domain.tld
      DocumentRoot /home/httpd/domain.tld/[hostname]/html

      <IfModule mod_dir.c>
        DirectoryIndex index.php index.cgi index.html index.htm
      </IfModule>

    ###
    # Directory access and configuration
    ###
      <Directory />
        Options FollowSymLinks
        AllowOverride None
        Order allow,deny
        Allow from all
      </Directory>
      <Directory /home/httpd/domain.tld/[hostname]/html/>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
        Order allow,deny
        Allow from all
      </Directory>


    ###
    # Log configuration
    # Possible values include: debug, info, notice, warn, error, crit, alert, emerg.
    ###
      LogLevel warn
      CustomLog /var/log/apache2/domain.tld/[hostname]-access.log combined
      ErrorLog /var/log/apache2/domain.tld/[hostname]-error.log


    ###
    # Configure the general rewrite rules
    ###
      RewriteEngine On
    #  RewriteLog "/var/log/apache2/domain.tld/[hostname]-redirect.log"
    #  RewriteLogLevel 3


    ###
    # Plone configuration settings (uncomment if Plone is installed)
    ###
    #  RequestHeader unset X_REMOTE_USER
    #  RequestHeader unset X_REMOTE_REALM

    #  RewriteCond %{REQUEST_METHOD} ^TRACE
    #  RewriteRule .* - [F]

      # make sure the ZMI, editing, and login are done via SSL
    #  RewriteRule ^(.*/[manage|manage_main|edit])$      https://%{SERVER_NAME}$1              [NE,L]
    #  RewriteRule ^/login_form(.*)$                     https://%{SERVER_NAME}/login_form$1   [R=permanent,L]


      # redirect the author pages and the join form to FSD
    #  RewriteRule ^/author/(.*)$                        /people/$1                            [R=permanent,L]
    #  RewriteRule ^/join_form(.*)$                      /people                               [R=permanent,L]

      # disable the sendto_form
    #  RewriteRule ^/(.*)/sendto_form(.*)$               /$1                                   [R=permanent,L]


      # handle Outlook's malformed RSS feed requests
    #  RewriteRule ^(.*)/rss$ $1/RSS


    ###
    # Rewrite rules that exclude certain non-plone sites and then direct everything else to plone
    # The two rules combined normalize requests to avoid caching /page and /page/ separately
    #
    # This configuration assumes that Squid is installed in front of Plone and runs on port 3128
    #
    # The RewriteCond provides an example of how to exclude sending a URL to Plone
    ###
    #  RewriteCond %{REQUEST_URI}  !^/test.php$
      RewriteRule ^/(.*)/$ http://127.0.0.1:3128/VirtualHostBase/http/%{SERVER_NAME}:80/intranet/VirtualHostRoot/$1 [L,P]

    #  RewriteCond %{REQUEST_URI}  !^/test.php$
      RewriteRule ^/(.*)$ http://127.0.0.1:3128/VirtualHostBase/http/%{SERVER_NAME}:80/intranet/VirtualHostRoot/$1 [L,P]

    </VirtualHost>

Then create an SSL virtual host using:

    sudo vi 001-[hostname].domain.tld-ssl

In the file, enter the following:

    <VirtualHost *:443>
      ServerName         [hostname].domain.tld
      ServerAdmin        webteam@domain.tld
      DocumentRoot       /home/httpd/domain.tld/[hostname]/html
      UseCanonicalName   On

    ###
    # Configure CoSign
    ###
      CosignProtected                 On
      CosignHostname                  connect.webaccess.psu.edu
      CosignPort                      6664
      CosignValidReference            ^https?:\/\/.*\.psu\.edu(\/.*)?
      CosignValidationErrorRedirect   https://webaccess.psu.edu/validation_error.html
      CosignRedirect                  https://webaccess.psu.edu/
      CosignPostErrorRedirect         https://webaccess.psu.edu/post_error.html
      CosignService                   [hostname].domain.tld
      CosignCrypto                    /etc/apache2/ssl/[hostname].domain.tld.key /etc/apache2/ssl/[hostname].domain.tld.crt /etc/apache2/ssl
      CosignAllowPublicAccess         Off


    ###
    # Directory access and configuration
    ###
      <Directory />
        Options FollowSymLinks
        AllowOverride None
      </Directory>
      <Location />
        # Configure access restrictions
        AuthType Cosign
        Require valid-user

        Order allow,deny
        Allow from all
      </Location>

      <Location /cosign/valid>
        SetHandler          cosign
        CosignProtected     Off
        Allow from all
        Satisfy any
      </Location>

      <Directory /home/httpd/domain.tld/[hostname]/html/>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride None
        Order allow,deny
        Allow from all
      </Directory>

      <Files ~ "^([\._]ht|README$|VERSION$|COPYING$)">
        Order allow,deny
        Deny from all
      </Files>


    ###
    # Turn on proxying for sending requests to Plone
    #
    # We are using apache as a reverse proxy/gateway in front of Plone
    # and don't need ProxyRequests turned on but we do need the
    # <Proxy *> block and related statements.
    #
    # See: /etc/apache2/mods-enabled/proxy.conf for full details
    ###
      <IfModule mod_proxy.c>
        ProxyRequests Off
        <Proxy *>
            AddDefaultCharset off
            Order deny,allow
            Allow from all
        </Proxy>
        ProxyVia On
      </IfModule>


    ###
    # Log configuration
    # Possible values include: debug, info, notice, warn, error, crit, alert, emerg.
    ###
      LogLevel warn
      CustomLog /var/log/apache2/domain.tld/[hostname]-access.log combined
      CustomLog /var/log/apache2/domain.tld/[hostname]-ssl_log "%t %h %{SSL_PROTOCOL}x %{SSL_CIPHER}x \"%r\" %b"
      ErrorLog /var/log/apache2/domain.tld/[hostname]-error.log


    ###
    # Configure SSL
    ###
      SSLEngine On
      SSLCertificateFile /etc/apache2/ssl/[hostname].domain.tld.crt
      SSLCertificateKeyFile /etc/apache2/ssl/[hostname].domain.tld.key
      SSLCertificateChainFile /etc/apache2/ssl/[hostname].domain.tld.ca-bundle


    ###
    # Put CoSign-specified username and realm in headers
    ###
      RequestHeader set X_REMOTE_USER %{remoteUser}e
      RequestHeader set X_REMOTE_REALM %{REMOTE_REALM}e


    ###
    # Configure the general rewrite rules
    ###
      RewriteEngine On
    #  RewriteLog "/var/log/apache2/domain.tld/[hostname]-redirect.log"
    #  RewriteLogLevel 3


      # redirect the author pages and the join form to FSD
      RewriteRule ^/author/(.*)$                        /people/$1                            [R=permanent,L]
      RewriteRule ^/join_form(.*)$                      /people                               [R=permanent,L]

      # disable the sendto_form
      RewriteRule ^/(.*)/sendto_form(.*)$               /$1                                   [R=permanent,L]


    ###
    # Rewrite rules that exclude certain non-plone sites and then direct everything else to plone
    # The two rules combined normalize requests to avoid caching /page and /page/ separately
    #
    # This configuration forwards all requests directly to the Zope client running on port 8080
    #
    # The RewriteCond provides an example of how to exclude sending a URL to Plone
    ###
      RewriteCond %{REQUEST_URI}  !^/cosign/valid(.*)$
      RewriteRule ^/(.*)/$ http://127.0.0.1:8080/VirtualHostBase/https/%{SERVER_NAME}:443/intranet/VirtualHostRoot/$1 [L,P,E=remoteUser:%{LA-U:REMOTE_USER}]

      RewriteCond %{REQUEST_URI}  !^/cosign/valid(.*)$
      RewriteRule ^/(.*)$ http://127.0.0.1:8080/VirtualHostBase/https/%{SERVER_NAME}:443/intranet/VirtualHostRoot/$1 [L,P,E=remoteUser:%{LA-U:REMOTE_USER}]

    </VirtualHost>



## Enable/disable Apache modules

### Ubuntu

Enable the required Apache modules with the following commands:

    sudo a2enmod rewrite
    sudo a2enmod proxy
    sudo a2enmod ssl
    sudo a2enmod headers
    sudo a2enmod proxy_http

And enter the following command to disable the unneeded Apache module:

    sudo a2dismod status


### RedHat / CentOS

The rewrite, proxy, headers, and proxy_http modules are installed and activated as part of the default Apache installation. The SSL module needs to be installed using:

    sudo yum install mod-ssl



## Enable/disable Apache virtual hosts

Disable the default Apache virtual hosts with the following commands:

    sudo a2dissite default
    sudo a2dissite default-ssl

Then enter the following commands to enable the new virtual hosts:

    sudo a2ensite 000-[hostname].domain.tld
    sudo a2ensite 001-[hostname].domain.tld-ssl


### Ubuntu

Restart Apache with the following command to implement all of the changes:

    sudo service apache2 restart


### RedHat / CentOS

Restart Apache with the following command to implement all of the changes:

    sudo service httpd restart



## Configure Apache to auto-start on bootup

### Ubuntu

Ubuntu's default installation of Apache automatically configures Apache to start on bootup so no action is needed.


### RedHat / CentOS

RedHat/CentOS's default installation does not automatically configure Apache to start on bootup so to do that issue the following command:

    sudo chkconfig httpd on
