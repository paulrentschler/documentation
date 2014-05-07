# Creating new user accounts

Documentation for how to create new user accounts on linux servers.

## Ubuntu

### Administrator accounts

The following command will create an account and grant the user full administrative and sudo rights to the server. (Use with caution!)

    sudo useradd -m -G adm,sudo -s /bin/bash -c "John Doe" jxd123
    sudo passwd jxd123

Replace _"John Doe"_ and _jxd123_ as appropriate.


### Regular user accounts

The following command will create a regular user account.

    sudo useradd -m -s /bin/bash -c "John Doe" jxd123
    sudo passwd jxd123

Replace _"John Doe"_ and _jxd123_ as appropriate.
