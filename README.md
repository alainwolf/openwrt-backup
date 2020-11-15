# OpenWrt Backup

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
* Avoids writing to the flash storage.


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

    $ mkdir -p ~/.ssh
    $ dropbearkey -t rsa -f ~/.ssh/id_rsa
    $ dropbearkey -f ~/.ssh/id_rsa -y > ~/.ssh/id_rsa.pub
    $ chmod 0700 ~/.ssh
    $ chmod 0600 ~/.ssh/id_rsa

Turris Omnia routers use OpenSSH where which includes `ssh-keygen`:

    $ mkdir -p ~/.ssh
    $ ssh-keygen

The following command will print out the public key:

    $ cat ~/.ssh/id_rsa.pub

Add the contents of the public key file `/root/.ssh/id_rsa.pub` on the router to
the users `~/.ssh/authorized_keys` file on on remote server.


### OpenPGP Keys

The router needs the public PGP key of its owner to encrypt the backup archives.

The following command, used on your router, will store your key in its public
keyring:

    $ PGPKEY=0x0123456789ABCDEF
    $ gpg --keyserver hkps://keys.openpgp.org --search-keys $PGPKEY

The router then needs ultimate trust to your:

    $ gpg --edit-key $PGPKEY
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


### Installation

I suggest saving the script in your /root directory on the router. So it will be
included in the backups. Be sure to make it executable:

    $ chmod 0764 /root/openwrt-backup


### Configuration

Copy the provided sample configuration file:

    $ cp /root/openwrt-backup.conf.sample /root/openwrt-backup.conf

Fill out the variables according to your needs.


### Cron-Job

Use the `crontab -e` command to add a line like the following to have backups created automatically on schedule:

    #min hour mday month wday cmd
    00   02   *    *     *    /root/openwrt-backup


### Motivation

While this might seem a bit overstreched for the mere backup of a home router, which usually runs quietly and doesn't even change much ...

I could be done with 3 command-lines dropped into a crontab.

I wrote this mostly as a exercise to get a grip on some issues which I kept bumping against over an over.

I run shell scripts on many different systems, on some of them (especially the embedded ones) there is no bash available.

I use most of these automated (e.g. cronjobs) and interactive. Since a long time I wanted them to be able to handle both properly including ...

   * Display messages only if started interactively
   * Log to syslog, but only if automated
   * Separate error output from non-error to handle accordingly.
   * Log with the right tags and priorities in syslog.
   * Get mail reports, also when cron does not provide it (busybox).
   * Get mail reports only on failures (cron just sends you everything).
   * Let programs still use their I/Os despite all the perma-redirecting.
   * Properly handle error conditions with traps.
   * Handle errors without the ERR signal (dash).
   * All while still remaining portable.

What this script is actually doing, after I have mastered all these goals, is not so important ;-)

So after a lot of StackExchange reading and re-trying, here is what I learned:

   * Move (almost) everything into functions.
   * Use nested subshells for output redirection.
   * Wrap (almost) every function into these subshells, each separately.
   * Don't assume. Send trap/process signals yourself.
   * Who needs ERR, when there are 30+ other signals to choose from?

