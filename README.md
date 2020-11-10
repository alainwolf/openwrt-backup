# Openwrt Backup

A backup script for your OpenWrt (or Turris) router.

Creates a compressed archive of selected folders from the overlay partition,
encrypts it with PGP and uploads it to a remote storage location using
'rsync over ssh'.

### Features

* Logs to syslog, if non-interactive (e.g. run by cron);
* Safes only the changes since the last release install (overlay partition);
* OpenPGP encrypted backup archive;
* SSH secured passwordless file transfer;
* Sends out mail reports on failure (or always if you wish so).


### Required Software

The backup script uses the following software packages:

* gnupg - GNU Privacy Guard;
* gnupg-utils - Key management utilities;
* gzip - compression utility;
* rsync - copy files to and from remote machines;
* tar - utility to package a set of files in a archive file.

To install these, enter the following commands on your router:

    $ opkg install gnupg gnupg-utils gzip rsync tar ssmtp


### Remote user with SSH public key

Since the backup-archives are uploaded to a remote system, the router needs a
login with an SSH public key on that server.

To create an SSH key-pair on OpenWrt:

    $ cd /root
    $ mkdir -p .ssh
    $ dropbearkey -t rsa -f .ssh/id_rsa
    $ chmod 0700 /root/.ssh
    $ chmod 0600 /root/.ssh/id_rsa

The command will print out the public key and fingerprint when done.

Unlike OpenSSH, Dropbear will not create public key files along with your
private keys. To export the public key again:

    $ dropbearkey -f .ssh/id_rsa -y

Turris routers have OpenSSH where which includes `ssh-keygen`:

    $ cd /root
    $ mkdir -p .ssh
    $ ssh-keygen


### OpenPGP Keys

The router needs the public PGP key of its owner to encrypt the backup archives.

The following command, used on your router, will store your key in its public
keyring:

    $ PGPKEY=0x0123456789ABCDEF
    $ gpg --keyserver hkps://keys.openpgp.org --search-keys PGPKEY

The router then needs ultimate trust to your:

    $ gpg --edit-key PGPKEY
    gpg> trust
    1 = I don't know or won't say
    2 = I do NOT trust
    3 = I trust marginally
    4 = I trust fully
    5 = I trust ultimately
    m = back to the main menu

    Your decision? 5
    gpg> quit


### Mail Setup

To send you the mail reports, your router needs a mail sever, where he is
allowed to send mail from. Any OpenWrt package, which provides the `sendmail`
command will do. The `ssmtp` package seems to be a popular choice.
