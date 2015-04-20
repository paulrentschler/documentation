# Using the TSM Client Command Line Interface for Backup & Restore

This document provides an introduction to using the TSM Command Line Interface
(CLI) to manually back up and restore files on the local machine. The screen
shots and descriptions may refer to older TSM clients, but the syntax should
be generic to all platforms and versions.

**BIG Thanks** to the University of Oxford IT Group as
[their document](https://help.it.ox.ac.uk/hfs/help/dsmc) serves as the basis
for this document.


## Starting the TSM Command Line client

Run "dsmc" as root from the shell prompt

    sudo dsmc

You should see a prompt like:

    IBM Tivoli Storage Manager
    Command Line Backup-Archive Client Interface
      Client Version 6, Release 4, Level 1.7
      Client date/time: 13-03-2014 15:01:20
    (c) Copyright by IBM Corporation and other(s) 1990, 2014. All Rights Reserved.

    Node Name: TEST-UBUNTU-OUCS
    Session established with server OX_HFS_B1: AIX
      Server Version 6, Release 3, Level 4.200
      Server date/time: 13-03-2014 15:01:15  Last access: 13-03-2014 13:01:04

    tsm>


## Accessing Help

Online help for TSM commands, options and error messages is available by typing
"help" at the "tsm>" prompt. The result will be similar to:

    1.0 New for IBM Tivoli Storage Manager Version 6.4
    2.0 Using commands
      2.1 Start and end a client command session
        2.1.1 Process commands in batch mode
        2.1.2 Process commands in interactive mode
      2.2 Enter client command names, options, and parameters
        2.2.1 Command name
        2.2.2 Options
        2.2.3 Parameters
        2.2.4 File specification syntax
      2.3 Wildcard characters
      2.4 Client commands reference
      2.5 Archive
      2.6 Archive FastBack

    Enter 'q' to exit help, 't' to display the table of contents,
    press enter or 'd' to scroll down, 'u' to scroll up or
    enter a help topic section number, message number, option name,
    command name, or command and subcommand:

Note that commands and options may be abbreviated to a short form as indicated
by capitalisation of words in the syntax entry for a command. Thus, for
example, "query filespace" can be abbreviated to "q fi".

Options and commands can also be included on the original command line so,
using the above example, you can run "sudo dsmc q fi" to just run a query of
the current partitions backed up. Obviously, more complex queries and commands
can be similarly run in the same manner.


## Querying the server

The following query commands illustrate typical command syntax and output.

### Querying your scheduled backup slot

To query your scheduled backup slot, enter:

    q sched

(which is short for "query schedule"). The output should look similar to:

    tsm> q sched

        Schedule Name: WEEKLY_ITSERV
          Description: ITSERV weekly incremental backup
       Schedule Style: Classic
               Action: Incremental
              Options:
              Objects:
             Priority: 5
       Next Execution: 149 Hours and 35 Minutes
             Duration: 15 Minutes
               Period: 1 Week
          Day of Week: Wednesday
                Month:
         Day of Month:
        Week of Month:
               Expire: Never


### Querying what files are included/excluded for backup

To query your list of included/excluded files/directories, enter:

    q inclexcl

The output should look similar to:

    tsm> q inclexcl
    *** FILE INCLUDE/EXCLUDE ***
    Mode Function  Pattern (match from top down)  Source File
    ---- --------- ------------------------------ -----------------
    Excl Filespace /var/run                       /opt/tivoli/tsm/client/ba/bin/incl.excl
    Excl Filespace /tmp                           /opt/tivoli/tsm/client/ba/bin/incl.excl
    Excl Directory /.../.opera/.../cache4         /opt/tivoli/tsm/client/ba/bin/incl.excl
    Excl Directory /.../.mozilla/.../Cache        /opt/tivoli/tsm/client/ba/bin/incl.excl
    Excl Directory /.../.netscape/.../cache       /opt/tivoli/tsm/client/ba/bin/incl.excl
    Excl Directory /var/tmp                       /opt/tivoli/tsm/client/ba/bin/incl.excl
    Excl All       /.../dsmsched.log              /opt/tivoli/tsm/client/ba/bin/incl.excl
    Excl All       /.../core                      /opt/tivoli/tsm/client/ba/bin/incl.excl
    Excl All       /.../a.out                     /opt/tivoli/tsm/client/ba/bin/incl.excl
    No DFS include/exclude statements defined.

Note that the _include/exclude_ directives are listed at the partition level
first, then the directory/folder level and finally at the file level. The order
they are displayed above is the order in which these directives are applied by
TSM. You will note that the order of the directives at any one level is the
opposite of the order in which they appear in the options file. That is, TSM
reads the directives listed in options file from the bottom up.


### Querying what partitions have been backed up

To query which partitions have been backed up, enter:

    q fi

The output should look similar to:

    tsm> q fi
    #       Last Incr Date      Type    File Space Name
    ---     --------------      ----    ---------------
      1   02-05-2013 02:13:13   EXT4    /
      2   25-07-2014 12:26:09   EXT3    /home


### Querying what files have been backed up

The syntax for querying what files you have backed up involves giving a file
specification. Also, if an incorrect file specification is given it may appear
that you have no backups. Consequently, several working examples are displayed
below.

If you give just a path to a directory/folder you will only get the folder
returned as the output:

    tsm> q ba /home/ians/projects
       Size      Backup Date                Mgmt Class           A/I File
       ----      -----------                ----------           --- ----
       512  B  24-04-2012 02:52:09          STANDARD             A  /home/ians/projects

If you just add a trailing * (star) as a wildcard in the above query, TSM will
only return those files and directories backed up **immediately below** the
directory path given in the query.

    tsm> q ba /home/ians/projects/*
       Size      Backup Date        Mgmt Class A/I File
       ----      -----------        ---------- --- ----
        512  12-09-2011 19:57:09    STANDARD    A  /home/ians/projects/hfs0106
      1,024  08-12-2011 02:46:53    STANDARD    A  /home/ians/projects/hsm41perf
        512  12-09-2011 19:57:09    STANDARD    A  /home/ians/projects/hsm41test
        512  24-04-2012 00:22:56    STANDARD    A  /home/ians/projects/hsm42upg

If you want to query all the current files and directories backed up under a
directory **and all its subdirectories** you need to add the "-subdir=yes"
option like so:

    tsm> q ba /home/ians/projects/* -subdir=yes
       Size      Backup Date        Mgmt Class A/I File
       ----      -----------        ---------- --- ----
        512  12-09-2011 19:57:09    STANDARD    A  /home/ians/projects/hfs0106
      1,024  08-12-2011 02:46:53    STANDARD    A  /home/ians/projects/hsm41perf
        512  12-09-2011 19:57:09    STANDARD    A  /home/ians/projects/hsm41test
        512  24-04-2012 00:22:56    STANDARD    A  /home/ians/projects/hsm42upg
      1,024  12-09-2011 19:57:09    STANDARD    A  /home/ians/projects/hfs0106/test
      1,024  12-09-2011 19:57:09    STANDARD    A  /home/ians/projects/hfs0106/test/test2
     12,048  04-12-2011 02:01:29    STANDARD    A  /home/ians/projects/hsm41perf/tables
     50,326  30-04-2012 01:35:26    STANDARD    A  /home/ians/projects/hsm42upg/PMR70023
     50,326  27-04-2012 00:28:15    STANDARD    A  /home/ians/projects/hsm42upg/PMR70099
     11,013  24-04-2012 00:22:56    STANDARD    A  /home/ians/projects/hsm42upg/md5check

Note that file specifications with spaces in them will need to be quoted.

By default only the current versions of files are listed. In order to query
both current (i.e., active) and previous (i.e., inactive) versions of files,
add the "-inactive" option to the query:

    tsm> q ba /home/ians/projects/* -subdir=yes -inactive
       Size      Backup Date        Mgmt Class A/I File
       ----      -----------        ---------- --- ----
        512  12-09-2011 19:57:09    STANDARD    A  /home/ians/projects/hfs0106
      1,024  08-12-2011 02:46:53    STANDARD    A  /home/ians/projects/hsm41perf
        512  12-09-2011 19:57:09    STANDARD    A  /home/ians/projects/hsm41test
        512  24-04-2012 00:22:56    STANDARD    A  /home/ians/projects/hsm42upg
      1,024  12-09-2011 19:57:09    STANDARD    A  /home/ians/projects/hfs0106/test
      1,024  12-09-2011 19:57:09    STANDARD    A  /home/ians/projects/hfs0106/test/test2
     12,048  04-12-2011 02:01:29    STANDARD    A  /home/ians/projects/hsm41perf/tables
      8,448  03-12-2011 01:31:18    STANDARD    I  /home/ians/projects/hsm41perf/tables
     50,326  30-04-2012 01:35:26    STANDARD    A  /home/ians/projects/hsm42upg/PMR70023
     50,326  27-04-2012 00:28:15    STANDARD    A  /home/ians/projects/hsm42upg/PMR70099
     11,013  24-04-2012 00:22:56    STANDARD    A  /home/ians/projects/hsm42upg/md5check
     11,013  23-04-2012 17:10:08    STANDARD    I  /home/ians/projects/hsm42upg/md5check

Note how the previous versions of files are marked by an "I" (for _Inactive_)
in the A/I column.

Be aware of potential confusion due to how TSM stores files in nested file
spaces. This can arise in the following situation: A user backs-up a file
"myconf.txt" on the "/usr" partition in the "/usr/local/etc" directory.
Subsequently, a new disk partition is mounted at "/usr/local", or it is defined
as a virtualmountpoint. Running the command:

    tsm> q ba /usr/local/etc/*

will <strong>not</strong> list the "myconf.txt" file. This is because TSM
always looks for a file in the filespace (partition) with the longest name that
matches the file specification you include in the command. In the above
example, the file was not backed up under the "/usr/local" filespace but under
the "/usr" filespace. To tell TSM to look for a file in latter filespace you
must specify the filespace explicitly using braces, like so:

    tsm> q ba {/usr}/local/etc/*


## Backing up your data

### Backing up local disks

The basic syntax for backing up local disk volumes is

    dsmc backup-type disk volume(s)

where "backup-type" is one of "incremental" or "selective". We
recommend incremental backups only; selective backups cause data to be sent
even if it already exists on the HFS. By default, if the "disk volume" is
omitted, TSM will backup those volumes specified by the "Domain" option in the
"dsm.opt" options file. If "Domain" is set to "All-Local", then to backup all
local volumes enter:

    tsm> incr

where "incr" is an abbreviation for "incremental".

To incrementally back up specific volumes enter:

    tsm> incr /  /usr  /usr/local  /home

To run an incremental _by date_ backup of the above, add the " -incrbydate"
option, as in:

    tsm> incr /  /usr  /usr/local  /home  -incrbydate

To back up entire disk volumes _irrespective_ of whether files have changed
since the last backup, use the "selective" command with a wildcard and
"-subdir=yes" as below:

    tsm> sel /*  /usr/*   /home/*  -su=yes


### 5.2. Backing up selected files

The basic syntax for backing up selected files is similar to that for backing
up disk partitions. Be aware, however, that **you cannot use wildcards in directory/folder names**:

    tsm> incr /home/ians/projects/hsm*/* -su=yes

    ANS1071E Invalid domain name entered: '/home/ians/projects/hsm*/*'

    tsm> sel /home/ians/projects/hsm*/* -su=yes
    Selective Backup function invoked.

    ANS1081E Invalid search file specification '/home/ians/projects/hsm*/*' entered

You can, however, enter several file specifications on the command line, like so:

    tsm> incr /home/ians/projects/hsm41test/*  /home/ians/projects/hsm41perf/* -su=yes


## Restoring your data</h2>

The basic syntax for restoring your data is

    dsmc restore source-file destination-file

If the "destination-file" is omitted then TSM will restore the file(s) to
their original location. Be aware that, as with backup,
**you cannot use wildcards in directory/folder names**. By default, TSM will
restore the most current _active_ version of a file.


### Restoring selected files

    tsm> rest /home/ians/myfile.txt  /home/ians/restore/
    tsm> rest /home/ians/myfile.txt  /home/ians/restore/myoldfile.txt

Note from the first example of each restore above that in order to specify a
directory as a destination, you need a trailing / (slash) at the end of the
destination-filespec. Otherwise TSM may overwrite a file of the same name. The
second example demonstrates a filename in the destination-filespec.

Restores of single files cannot be restarted if interrupted. In this case you
will need to restore the file afresh.


### Restoring multiple files and directories

    tsm> rest /home/ians/projects/hsm41test/*  /home/ians/projects/restore/ -su=yes

Note that in order to restore a full directory _and_ the contents of all its
sub-directories you need the "-su=yes" option. It is always good practice to
terminate the destination-filespec with a trailing / (slash) if the element in
the destination-filespec is a directory.

As this restore is wild-carded, it can be restarted if interrupted due to user
input (Ctrl-C), server error or communications error. Restartable restores can
be queried via "q rest" and will restart _at the point of interruption_.


### Restoring entire partitions

Essentially, the syntax is the same as in 'Restoring multiple files and
directories' above. However, the obvious caveats are to ensure enough space in
the destination partition and to allow enough time.

    tsm> rest /home/*  /tmp/restore/ -su=yes

As with 'Restoring multiple files and directories' above, this restore is
wild-carded and thus can be restarted if interrupted.


### Restoring old and/or deleted files

TSM does not, by default, list or restore old and deleted _inactive_ versions
of files and directories. If you need to restore such a file, you need the
"-inactive -pick" options. The "-pick" option, while not strictly necessary,
causes TSM to display a list of files from which to pick. Issuing a restore
as below will display the following pick window:

    tsm> rest /home/ians/projects/*  /tmp/restore/ -su=yes  -inactive -pick

    TSM Scrollable PICK Window - Restore

         #    Backup Date/Time        File Size A/I  File
       --------------------------------------------------------------------------------------------------
       170. | 12-09-2011 19:57:09        650  B  A   /home/ians/projects/hsm41test/inclexcl.test
       171. | 12-09-2011 19:57:09       2.74 KB  A   /home/ians/projects/hsm41test/inittab.ORIG
       172. | 12-09-2011 19:57:09       2.74 KB  A   /home/ians/projects/hsm41test/inittab.TEST
       173. | 12-09-2011 19:57:09       1.13 KB  A   /home/ians/projects/hsm41test/md5.out
       174. | 30-04-2012 01:35:26        512  B  A   /home/ians/projects/hsm42125upg/PMR70023
       175. | 26-04-2012 01:02:08        512  B  I   /home/ians/projects/hsm42125upg/PMR70023
       176. | 27-04-2012 00:28:15        512  B  A   /home/ians/projects/hsm42125upg/PMR70099
       177. | 24-04-2012 19:17:34        512  B  I   /home/ians/projects/hsm42125upg/PMR70099
       178. | 24-04-2012 00:22:56       1.35 KB  A   /home/ians/projects/hsm42125upg/dsm.opt
       179. | 24-04-2012 00:22:56       4.17 KB  A   /home/ians/projects/hsm42125upg/dsm.sys
       180. | 24-04-2012 00:22:56       1.13 KB  A   /home/ians/projects/hsm42125upg/dsmmigfstab
       181. | 24-04-2012 00:22:56       7.30 KB  A   /home/ians/projects/hsm42125upg/filesystems
       182. | 24-04-2012 00:22:56       1.25 KB  A   /home/ians/projects/hsm42125upg/inclexcl
       183. | 24-04-2012 00:22:56        198  B  A   /home/ians/projects/hsm42125upg/inclexcl.dce
       184. | 24-04-2012 00:22:56        291  B  A   /home/ians/projects/hsm42125upg/inclexcl.ox_sys
       185. | 24-04-2012 00:22:56        650  B  A   /home/ians/projects/hsm42125upg/inclexcl.test
       186. | 24-04-2012 00:22:56        670  B  A   /home/ians/projects/hsm42125upg/inetd.conf
       187. | 24-04-2012 00:22:56       2.71 KB  A   /home/ians/projects/hsm42125upg/inittab
       188. | 24-04-2012 00:22:56       1.00 KB  A   /home/ians/projects/hsm42125upg/md5check
       189. | 24-04-2012 00:22:56      79.23 KB  A   /home/ians/projects/hsm42125upg/mkreport.020423.out
       190. | 24-04-2012 00:22:56       4.27 KB  A   /home/ians/projects/hsm42125upg/ssamap.020423.out
       191. | 26-04-2012 01:02:08      12.78 MB  A   /home/ians/projects/hsm42125upg/PMR70023/70023.tar
       192. | 25-04-2012 16:33:36      12.78 MB  I   /home/ians/projects/hsm42125upg/PMR70023/70023.tar
            0---------10--------20--------30--------40--------50--------60--------70--------80--------90--
    <U>=Up  <D>=Down  <T>=Top  <B>=Bottom  <R#>=Right  <L#>=Left
    <G#>=Goto Line #  <#>=Toggle Entry  <+>=Select All  <->=Deselect All
    <#:#+>=Select A Range <#:#->=Deselect A Range  <O>=Ok  <C>=Cancel
    pick>


You are now in the pick interface and can select individual files to restore
via the number to the left, scroll up or down via "U" and "D" as described at
the bottom of each listing of files.

Remember to issue the destination-filespec with the original restore command if
you want to prevent overwriting current versions of files with older versions.


## Restoring your data to another machine

In certain circumstances, it may be necessary to restore some, or all, of your
data onto a machine other than the original from which it was backed up.
Ideally the machine platform should be identical to that of the original
machine.

Where this is not possible or practical please note that restores are
only possible for partition types that the operating system supports. Thus a
restore of an NTFS partition to a Windows 9x machine with just FAT support may
succeed but the file permissions will be lost. Please do not attempt
cross-platform restores  (i.e., by trying to restore files onto a Windows machine
that have previously been backed up with a non-Windows one). Using TSM for
Windows to try to access backups sent by other OS platforms can cause those
backups to become inaccessible from the host system.

To restore your data to another machine you will need the TSM software
installed on the target machine. Entries in "dsm.sys" and/or "dsm.opt" will
need to be edited if the node that you are restoring from does not reside on
the same HFS server as the one that you are restoring to.

To access files from another machine you should then start the TSM client like so:

    dsmc -virtualnodename=DEAD.MACHINE

where DEAD.MACHINE should be substituted for the nodename of the machine to be
restored. You will then be prompted for the TSM password for this machine.

Querying and restoring the filestore is then as in the previous section,
'Restoring your data'. You will probably want to restore to a different
destination to the original files to prevent overwriting files on the local
machine, like so:

    tsm> rest /home/* /scratch/     -su=yes


## Authorizing another machine to restore your files

If you are responsible for a number of TSM client machines, you can protect
against the loss of one machine by authorizing a different machine(s) to
restore backup versions of your files. The basic syntax for this is

    set access backup filespec node username

Thus to grant the root user on machine ANOTHER.NODE access to restore the
/home filespace, you would issue the following:

    tsm> set acc backup /home/*/* another.node root

The current access list can be queried and deleted using the ">query acc" and
"delete acc" commands.

Once access has been granted from another machine, you can query and restore
files from that machine to your local machine using the "-fromnode" option,
like so:

    tsm> q files -fromnode=ANOTHER.NODE
    tsm> rest -fromnode=ANOTHER.NODE /home/*/*  /home/restore/
