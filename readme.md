Check Site Backup Integrity: WordPress/PHP MySQL
================================================

If you've set up a backup solution for PHP/WordPress, you will probably be backing up site files as well as the site database. For a proper backup solution, you need to check that the backup copy is viable. This is not something that you should discover during an emergency recovery situation. Doing this manually takes too much time.

In our case, site files from the Apache root of a production server are backed up incrementally on a daily basis to a date-stamped directory. This contains a subdirectory `html` - which in turn contains a subdirectory for each site under the document root. It also contains a subdirectory `sql` which contains a collection of dumped databases for the sites in question. You should also back up important config files but that is beyond the scope of this exercise.

To test the integrity of backed up sites, one option is to build working copies of the sites on a virtual machine. To avoid the need to change URLs on the backup copies, the `/etc/hosts` file is amended on the guest VM. Obviously, the guest VM needs to run a server that broadly matches the original backed up server (in this case Apache), and the virtual hosts settings for the guest VM server need to be set up correctly (this is a one-time import from the backed-up config directory).

You don't necessarily need to use a VM - you could use any machine on the local network. The reason this is done on a VM/separate machine is so that the main host computer can access the *actual* live sites for maintenance purposes. The hosts file of the VM has been amended to point towards the local copies. This method also keeps seperation between backed up clones and ongoing development websites - which are two different things.

## Aim
Quickly move site files and databases for an entire backed-up production server into a local virtual machine. Import databases and set up working sites on the VM, thereby checking the integrity of multiple site backups.

Backup integrity is checked regularly, so this should be a simple process. Ideally, once the system has been setup it should be run by administrators rather than developers.

This is achieved in a two-step process:

1. Export files from the Host machine to the guest (run command in Host)
2. Import databases in the Guest (run command in Guest)

## Requirements
These BASH scripts have been tested on Ubuntu Xenial Xerus 16.04.

[Zenity](https://help.gnome.org/users/zenity/stable/intro.html.en) has been used to create user dialogues.

[VirtualBox](https://www.virtualbox.org/) is required for the Virtual Machine. In this case, the VM runs Ubuntu 16.04 Xenial Xerus desktop - desktop rather than server because it allows easy checking of the moved sites. To achieve this, the guest machine hosts file (`/etc/hosts`) must be set up properly to point at the local copies.

The virtual machine also runs Ubuntu 16.04. The database server is MariaDB, but the commands woudl work on a standard MySQL database server.

The sql backups directory includes the `performance_schema.sql`, `phpmyadmin.sql`, `mysql.sql` and log files from the original server. These aren't necessary to check backups, and if imported will probably mess up the VM MySQL configuration. Because of this, we exclude these files from the transfer - see the `sql-verification-exclude` file in this repo for example.

For the backed up sites on the guest machine to work properly, the MySQL users from the original server should be imported in a one-time operation.

## Move Files to the VM
This is achieved with the `move-backups` script. This script prompts the user to choose a directory to move. The script is tightly coupled to our requirements, but would be easy to amend.

The directory to be moved is a datestamped directory that contains the entire
`html` directory (i.e. document root) from a backed-up Apache server. It also contains backed up MySQL files (originally created by `mysqldump`) in a `sql` directory.

Usage:

- Add `move-backups` to `usr/local/bin` on the Host computer: `mv move-backups /usr/local/bin`
- Make executable: `chmod +x /usr/local/bin/move-backups`
- Run `move-backups` in a terminal and follow instructions

When prompted, you should select a directory that contains the backed-up `html`
directory from the Apache doc root - the directory that is normally located
at `/var/www/` in a standard Apache setup.

Note that the moved files won't do anything unless you also import the
associated databases on the guest machine.

## Import Databases
- Add `import-databases` to `usr/local/bin` on the Guest computer/VM: `mv import-databases /usr/local/bin`
- Make executable: `chmod +x /usr/local/bin/import-databases`
- Run `import-databases` in a terminal

## TODO
- Document how to add key-pair to avoid entering password during slash
- Auto create the staging directory for the sql files transfer
- Change ownership of moved HTML files
- Trigger the import-databases script from the host, so working copies are built with a single command
- Better feedback on the `import-databases` script
- Document how to import users from original server to the guest machine

Resources
---------
* [Closing the VM](https://www.virtualbox.org/manual/ch08.htmlvboxmanage-controlvm)
* [Get path of a filename](http://stackoverflow.com/a/10274182)
* [Starting the VM from a script](https://www.virtualbox.org/manual/ch08.htmlvboxmanage-startvm)
* [Remove trailing slash](http://stackoverflow.com/a/19485757)
* [VM Management](http://ubuntuforums.org/archive/index.php/t-1078689.html)
