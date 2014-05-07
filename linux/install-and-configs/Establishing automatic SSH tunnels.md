# Establishing automatic SSH tunnels

Instructions on configuring automatically connected and authenticated SSH
tunnels between servers. Specifically used for tunneling MySQL traffic
securely between servers.

Adapted from initial documentation by: [Jon Pierce](http://jonpierce.net/).


## How to use this document

In this example, _woodstock_ is a web server using an SSH tunnel to execute
commands on _snoopy_, which is a MySQL server.

It is assumed that SSH is running on both servers and listening on port 2222
instead of the default port 22 as a non-standard port is a slightly more
secure way of running SSH. If your SSH server listens on a port other than
2222, be sure to change it to 22 or whatever port you use in the examples below.

The server you should execute the commands on appears as a subheading to the
list of commands like this:

### snoopy

1. List of commands
1. to execute on _snoopy_

## Prepare the MySQL server to accept SSH connections

### snoopy

1. Create a user account called "tunnel". This user account will execute
commands on the remote server's (_woodstock_) behalf.

        useradd -s /bin/false tunnel

1. Create a place to store the public encryption keys that will be used
for authentication instead of passwords

        mkdir -p /home/tunnel/.ssh
        touch /home/tunnel/.ssh/authorized_keys
        chown -R tunnel:tunnel /home/tunnel
        chmod -R 755 /home/tunnel
        chmod 644 /home/tunnel/.ssh/authorized_keys


## Setup the encryption keys for authentication

Since servers can't type passwords, we create and configure public/private
encryption keys to handle the authentication without passwords.

### woodstock

1. Create the keys. If the command asks for any values,
**use the defaults provided**!

        cd ~
        sudo ssh-keygen -t rsa -b 4096

1. Copy the public key to _snoopy_

        scp -P2222 ~/.ssh/id\_rsa.pub user1@snoopy:~/woodstock-id_rsa.pub


### snoopy

1. Append the newly uploaded key to the authorized_keys file and then
delete the original file

        sudo -i
        cat /home/user1/woodstock-id\_rsa.pub >> /home/tunnel/.ssh/authorized_keys
        rm /home/user1/woodstock-id_rsa.pub


## Test the encryption keys

### woodstock

1. Attempt to execute a remote command. You should not be prompted for a password.

        sudo ssh -v -p 2222 tunnel@snoopy logout


## Setup a persistent SSH tunnel that connects on startup

### woodstock (Ubuntu-based machine)

Use the autossh software to establish and keep established the ssh tunnel
to _snoopy_.

1. Install autossh

        aptitude install autossh

1. Create the startup script /etc/init.d/tunnel-snoopy
(replace "snoopy" as appropriate). [Startup script source](https://gist.github.com/atr000/643783)

        #! /bin/sh
        #
        # Author:   Andreas Olsson
        # Version:  @(#)autossh_tunnel.foo  0.1  27-Aug-2008  andreas@arrakis.se
        #
        # For each tunnel; make a uniquely named copy of this template.


        ## SETTINGS
        #
        # autossh monitoring port (unique)
        MPORT=27020
        # the ssh tunnel to setup
        TUNNEL="-L 3307:127.0.0.1:3306 -p 2222"
        # remote user
        RUSER="tunnel"
        # remote server
        RSERVER="snoopy"
        # You must use the real autossh binary, not a wrapper.
        DAEMON=/usr/lib/autossh/autossh
        #
        ## END SETTINGS

        PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

        NAME=`basename $0`
        PIDFILE=/var/run/${NAME}.pid
        SCRIPTNAME=/etc/init.d/${NAME}
        DESC="SSH Tunnel to Snoopy"

        test -x $DAEMON || exit 0

        export AUTOSSH_PORT=${MPORT}
        export AUTOSSH_PIDFILE=${PIDFILE}
        ASOPT=${TUNNEL}" -f -N "${RUSER}"@"${RSERVER}

        # Function that starts the daemon/service.
        d_start() {
            start-stop-daemon --start --quiet --pidfile $PIDFILE \
                --exec $DAEMON -- $ASOPT
            if [ $? -gt 0 ]; then
                echo -n " not started (or already running)"
            else
                sleep 1
                start-stop-daemon --stop --quiet --pidfile $PIDFILE \
                --test --exec $DAEMON > /dev/null || echo -n " not started"
            fi
        }

        # Function that stops the daemon/service.
        d_stop() {
            start-stop-daemon --stop --quiet --pidfile $PIDFILE \
                --exec $DAEMON \
                || echo -n " not running"
        }

        case "$1" in
          start)
            echo -n "Starting $DESC: $NAME"
            d_start
            echo "."
            ;;
          stop)
            echo -n "Stopping $DESC: $NAME"
            d_stop
            echo "."
            ;;
          restart)
            echo -n "Restarting $DESC: $NAME"
            d_stop
            sleep 1
            d_start
            echo "."
            ;;
          *)
            echo "Usage: $SCRIPTNAME {start|stop|restart}" >&2
            exit 3
            ;;
        esac

        exit 0

1. Configure the script to start and stop the daemon on bootup / shutdown

        cd /etc/init.d
        chmod +x tunnel-snoopy
        update-rc.d tunnel-snoopy defaults


### woodstock (Red Hat-based machine)

Use the autossh software to establish and keep established the ssh tunnel to
_snoopy_.

1. Install autossh

        yum install autossh

1. Create the startup script /etc/init.d/tunnel-snoopy
(replace "snoopy" as appropriate)

        #!/bin/bash
        #
        # autosshd This script starts and stops the autossh daemon
        #

        ## SETTINGS
        #
        # autossh monitoring port (unique)
        MPORT=27020
        # the ssh tunnel to setup
        TUNNEL="-L 3307:127.0.0.1:3306 -p 2222"
        # remote user
        RUSER="tunnel"
        # remote server
        RSERVER="snoopy"
        # You must use the real autossh binary, not a wrapper.
        DAEMON=/usr/bin/autossh
        #
        ## END SETTINGS
        EXECCOM=${DAEMON}" -M "${MPORT}" -f -N "${TUNNEL}" "${RUSER}"@"${RSERVER}

        # Source function library.
        . /etc/rc.d/init.d/functions

        # Source networking configuration.
        . /etc/sysconfig/network

        # Check that networking is up.
        [ ${NETWORKING} = "no" ] && exit 0

        export AUTOSSH_PIDFILE="/var/run/autossh.pid"

        # By default it's all good.
        RETVAL=0

        # Start function.
        start() {
            local name=$1
            echo -n $"Starting ${name}: "
            if [ -e "/var/lock/subsys/${name}" ]; then
                if [ -e "/var/run/${name}.pid" ] && [ -e /proc/`cat /var/run/${name}.pid` ]; then
                    echo -n $"already exists.";
                    failure $"already exists.";
                    echo
                    return 1
                fi
            fi
            daemon ${EXECCOM}
            RETVAL=$?
            [ $RETVAL -eq 0 ] && touch "/var/lock/subsys/${name}"
            echo
            return $RETVAL
        }

        # Stop function.
        stop() {
            local name=$1
            echo -n $"Stopping ${name}: "
            killproc -p "/var/run/${name}.pid" ${name}
            RETVAL=$?
            echo
            [ $RETVAL -eq 0 ] && rm -f "/var/lock/subsys/${name}"
            return $RETVAL
        }

        # See how we were called.
        case "$1" in
            start)
                start "autossh"
                ;;
            stop)
                stop "autossh"
                ;;
            restart)
                $0 stop
                sleep 3
                $0 start
                ;;
            status)
                status "autossh"
                RETVAL=$?
                ;;
            *)
                echo $"Usage: $0 {start|stop|restart|status}"
                exit 1
                ;;
        esac

        exit $RETVAL


1. Configure the script to start and stop the daemon on bootup / shutdown

        cd /etc/init.d
        chmod +x tunnel-snoopy
        sudo ln -s tunnel-snoopy ../rc0.d/K20tunnel-snoopy
        sudo ln -s tunnel-snoopy ../rc1.d/K20tunnel-snoopy
        sudo ln -s tunnel-snoopy ../rc2.d/S20tunnel-snoopy
        sudo ln -s tunnel-snoopy ../rc3.d/S20tunnel-snoopy
        sudo ln -s tunnel-snoopy ../rc4.d/S20tunnel-snoopy
        sudo ln -s tunnel-snoopy ../rc5.d/S20tunnel-snoopy
        sudo ln -s tunnel-snoopy ../rc6.d/K20tunnel-snoopy


## Manually starting the SSH tunnel

### woodstock

        autossh -M 27020 -f -N -L 3307:127.0.0.1:3306 -p2222 tunnel@snoopy


## Manually stop the SSH tunnel

### woodstock

1. Find the process id using

        ps -ax | grep ssh

1. Kill the process

        kill -9 {process id}
