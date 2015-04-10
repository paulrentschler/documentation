# Plone Development Cheatsheet

The commands on this cheatsheet are entered into the Windows Command Prompt
which is run by clicking the Start button and searching for either
"Command Prompt" or "cmd.exe".

Since the command prompt will be needed often, it's recommended that once you
find it in the Start Menu, you right-click on it and select either
"Pin to Taskbar" or "Pin to Start Menu" so that it's easily accessed in the
future.


## Initially building the Plone virtual machine (VM)

To initially build the Plone VM where all the development work will be done,
you first need to create a directory on your hard drive where the VM will
live and then use Git to pull down the VM files and Vagrant to build the VM.

    cd Documents
    mkdir eeq-plone
    cd eeq-plone
    git clone https://github.com/paulrentschler/plonedev.vagrant.git .
    git checkout equity-4.3
    vagrant up

This will take some time, but it will accomplish the following things:

1. Create a new Ubuntu Linux virtual machine
1. Use the Plone Unified Installer to install Plone 4.3.3
1. Clone the equity.buildout Git repository which contains a custom Buildout
    equity.cfg and versions.cfg files
1. Clone the equity.theme Git repository which contains the new Diazo-based
    Plone theme product
1. Run Buildout with the custom equity.cfg and versions.cfg files to install
    the various custom products and the equity.theme product

**Note:** Plone VM will be **running** at this point and needs to be shutdown
before turning off your computer.


## Working with the Plone VM

### Shutting down the Plone VM

From the eeq-plone directory, run the command:

    vagrant halt

This will power down the Plone VM.


### Restarting the Plone VM

From the eeq-plone directory, run the command:

    vagrant up

This will power up the Plone VM so it's ready to use.


### Destroying the Plone VM and starting over

**WARNING:** This will delete the VM and all files associated with the Plone VM,
if you have not backed up files and/or committed and pushed your code changes
to the appropriate Git repository **ALL CODE WILL BE LOST**!

From the eeq-plone directory, run the command:

    vagrant destroy

Once complete, you can then delete the Plone VM directory by typing:

    del *
    cd ..
    rmdir eeq-plone

or you can delete the eeq-plone directory using Windows Explorer.

To rebuild the Plone VM, follow the _Initially building the Plone virtual
machine (VM)_ steps above.


## Working with Plone in the VM

The Plone VM has been carefully created to allow you to do all of your work
from the Windows host machine without ever having to interact with the Linux
operating system that is ultimately running the Plone installation. All the
commands below are still run at the Windows Command Prompt. Editing of files
is done using your favorite Windows-based text editor such as Notepad,
NotePad++, or Sublime Text.


### Starting Plone in foreground mode

To get Plone running so that you can view it in the browser and see errors
in the Command Prompt window, run the following command:

    plonectl fg

When you see the text "INFO Zope Ready to handle requests", the Plone site is
up and running. At this point, open your favorite web browser on your Windows
computer and navigate to:

    http://localhost:8080

You will be prompted to create a new Plone site or can navigate to any
existing Plone sites.


### Creating a new Plone site

The first time you go to http://localhost:8080 you will be prompted to create
a new Plone site since none currently exists. To create a new Plone site
compatible with the future Educational Equity site, do the following:

1. Click the "Create a new Plone site" button
1. Enter "admin" for the User Name and Password and click "Ok"
1. Enter whatever you want for Path identifier and Title.
1. Select the following minimum Add-ons (you can select more):
    * Dexterity Content Types
    * Dexterity-based Plone Default Types
    * Diazo theme support
    * PloneFormGen
    * equity.theme
1. Click "Create Plone Site"

The Plone site will be built and you will be taken to the front page of the
new site when it's finished.


### Stopping Plone in foreground mode

Once you start Plone in foreground mode you will not be able to type an more
commands into the Command Prompt window until you stop Plone. To stop Plone,
do the following:

1. In the Command Prompt window press CTRL-C
1. Enter "Y" and press Enter
1. Run the following command:

        kill_plone

Plone is now no longer running and you can type commands in the Command Prompt
window again.


### Running Buildout

If you need to install a new product and thus re-run Buildout, use the
following command:

    buildout -c equity.cfg

<br><br>

## The daily quick reference

### Start the Plone VM

    vagrant up

### Start Plone

    plonectl fg

### Stop Plone

1. CTRL-C
1. "Y" and enter
1. run:

        kill_plone

### Stop the Plone VM

    vagrant halt
