# IMAPdedup
*A duplicate email message remover*

**Please note that these instructions have changed since version 1.1.**

IMAPdedup is a command-line utility that looks for duplicate messages in a set of IMAP mailboxes and tidies up all but the first copy of any duplicates found.

To be more exact, it *marks* the second and later occurrences of a message as 'deleted' on the server.   Exactly what that does in your environment will depend on your mail server and your mail client.

Some mail clients will let you still view such messages *in situ*, so you can take a look at what's happened before 'compacting' the mailbox.  Sometimes deleted messages appear in a 'Trash' folder.  Sometimes they are hidden and can be displayed and un-deleted if wanted, until they are purged.

Whatever your system does, you will usually have the option to see what has been deleted, and to recover it if needed, using your email program, after running this script.  (If your server purges the deleted messages automatically, you may be able to prevent this with the `--no-close` option.)  There is also a 'dry-run' option so you can check what might happen before doing anything scary.

## How it works

By default, IMAPdedup will simply look for messages with a duplicate Message-ID header.  This is a string generated by email systems that should normally be unique for any given message, so unless you've got some rather unusual mailboxes, it's a pretty safe choice.  (Note that GMail, for example, *does* sometimes have some unusual mailboxes.)

If you have messages *without* a Message-ID header, or you don't trust it, there's an option (-c) to use a checksum of the To, From, Subject, Date, Cc & Bcc fields instead.

And if you want to add the Message-ID, if it exists, into this checksum, add the '-m' option as well. I'd recommend this in general, because some (foolish) automated systems can send you multiple messages within a single second, with different contents but the same headers. (e.g. "Subject: Your review has just been published!")

## Installation

You can now install IMAPdedup using pip:

    pip install imapdedup

This will install the script, and you can then just run it from the command line as `imapdedup`.  In the examples below, we'll assume you've done that.

**Alternatives**

Some environments may require you to use `pip3` instead of `pip`. And some may not let you install packages globally. In that case, you can use the `--user` option to install it in your home directory:

    pip install --user imapdedup

IMAPdedup doesn't *need* any installation process, however, and doesn't have any external dependencies.  You only need Python 3 and can then just run the imapdedup.py file in `src/imapdedup`. In this case, the command you would run would be something like:

    python3 imapdedup.py ...

If you might want to modify the script yourself, I recommend using [uv](https://docs.astral.sh/uv/). You can then simply get a copy of the source and use: 

    uv run imapdedup ....

which will create a virtualenv for you and keep it up to date without you having to think much about it.


## Trying it out

If you want to experiment, create a new folder on your mail server, and copy some messages into it from your inbox.  Then copy some of them in a second time.  And maybe a third. That should give you a safe place to play!

## Simple use

You can list the full syntax by running:

    imapdedup -h

but the key options are described below.  You will of course need the address of your IMAP email server, and your username on that server.

Try starting with something harmless like:

    imapdedup -s imap.myisp.com -u myuserid -x -l

which prompts you for your password and then lists the mailboxes on the server. You can then use the mailbox names it returns when running other commands. (The `-x` option specifies that the connection should use SSL, which is generally the case nowadays. If this doesn't work, you can leave it out, but you should probably also complain to your email provider because they aren't providing sufficient security! A future version of the script will make this the default.)

It's worth trying getting this list at least once because different mail servers structure their folders differently. One of mine thinks of all the folders as being 'within' the inbox, for example, so they're called things like 'INBOX.Drafts','INBOX.Sent', and those are the names I need to use when talking to the server.

Once you know your folder names, you can run something like

    imapdedup -s imap.myisp.com -u myuserid -x -n INBOX.Test

and the script will tell you what it would do to your *INBOX/Test* folder.  If your folder name contains spaces, you'll need to put it in quotes:

    imapdedup -s imap.myisp.com -u myuserid -x -n "My Important Messages"

The `-n` option tells IMAPdedup that this is a 'dry run': it stops it from *actually making* any changes; it's a good idea to run with this first unless you like living dangerously.  When you're ready, leave that out, and it will go ahead and mark your duplicate messages as deleted.

The process can take some time on large folders or slow connections, so you may want to add the `-v` option to give you more information on how it's progressing.

The `-y` option will copy messages to the specified mailbox before deleting them.  This will normally have the combined effect of moving any duplicates to another folder.

The `-t` option will, instead of marking messages for deletion, attempt to tag them with the specified custom tag.  Note that not all IMAP servers will allow the creation of custom tags, and not all mail programs will allow you to view them.  Still, this can be a useful option if your software supports it!

You can specify multiple folders to work on, and it work through them in order and will delete or tag, in the later folders, duplicates of messages that it has found either in those folders or in earlier ones.   A warning: If you specify the same mailbox *twice*, it will look at it a second time, see a whole load of messages it has seen before and so delete them all as duplicates!  You have been warned, and that's why you should use the `-n` option first.  Don't ask me how I discovered this...

If you have a hierarchy of folders, you can search recursively within the children of a particular folder using the `-r` option and specifying the top-level folder.

You can use the `-R` option to reverse the order in which the folders are processed.  The main use of this is with recursive searches, if you want duplicates of messages found in child folders to be removed from the parents, rather than the other way around.

With the `-d` option, it's removes all marked messages from the server (Expunge).  This is a dangerous option, so use with caution!

## Specifying the password

If you don't wish to specify a password via a command-line argument, where it could be seen by other users of the system, and you don't want to type it in each time, you have three options:

* You can put it in an environment variable called IMAPDEDUP_PASSWORD, or
* You can specify it in a wrapper script as described below, or
* You can put it into your system's [keyring](https://pypi.org/project/keyring/).

For the keyring option you'll need to pip install [keyring](https://pypi.org/project/keyring/). You can then use `-K` to specify a system keyring name that will be used to get the password for the account specified via option `-u`.  If you just use `-K' without a value, the server name provided will be used as the keyring name.

## Authenticating as an admin user

Some IMAP servers - Zimbra's being one example - allow an admin user to login and then perform actions on behalf of another user.   If your server supports this via the 'AUTHENTICATE PLAIN' command, then the `-a` option allows you to specify the admin username.  More information on this mechanism can be found in [RFC 3501](https://datatracker.ietf.org/doc/html/rfc3501#section-6.2.2) and [RFC 2595](https://datatracker.ietf.org/doc/html/rfc2595#section-6).

## Use from an external script

We don't curently have a way of storing your options in a configuration file, but you can call imapdedup from your own script. Construct a list of arguments as you'd put them on the command line, and then process them using something like the following:

    #! /usr/bin/env python3

    import imapdedup

    options = [
        "-s", "imap.example.com", 
        "-u", "my_user_name",
        "-w", "my_password",
        "-x"
    ]

    mboxes = [
        'INBOX',
        'Some other mailbox',
    ]

    imapdedup.process(*imapdedup.get_arguments(options + mboxes))

If you are on a shared machine or filesystem and you are including sensitive information such as the password in this file, you may wish to set its permissions appropriately.

## Checking ALL of your mailboxes

Several people have asked for an option to check for duplicates across ALL of your mailboxes.  This isn't built-in for a couple of reasons.  The main one is that when you specify multiple folders on the command line, IMAPdedup will search them in order, deleting duplicate messages from the later ones if they have been found in the earlier ones.   If we added an 'All folders' option, we'd need a way to specify which ones came first: if you find duplicates in two or more mailboxes, which one(s) should be deleted?

So my proposed workaround if you're in this situation is to ask the program for a list of all your folders, put those in a file, edit that file as wanted (e.g. to change the order, or exclude particular folders), and then use `xargs` to pass all the folder names in the file to IMAPdedup.

Here's a quick demo. Imagine you normally run IMAPdedup like this:

    imapdedup -x -s servername -u username -w password ...

You can save your list of folders to a file called folders.txt like this:

    imapdedup -x -s servername -u username -w password -l > folders.txt

The standard unix `xargs` utility lets you read a list of words (or lines) from the standard input and run another command passing those words from the list as arguments. So you can do something like this (I've included the -n so it's a dry run and should be safe if you try it!):

    cat folders.txt | xargs imapdedup -x -s servername -u username -w password -n

xargs will basically run imapdedup with all the folder names in folders.txt passed as arguments as if you had typed them on the command line. *NOTE:* If your folder names have any spaces in them, you should open the text files in an editor and put quotes around each line that has them. Alternatively, you can use sed to do this for you:

    cat folders.txt | sed 's/^\(.*\)$/"\1"/' | xargs imapdedup -x -s servername -w pasword -u username -n

All of this does require you to be running on Linux or a Mac. Much harder to do anything like this on Windows, of course, though WSL might make it easier.


## Accessing the IMAP mailboxes via a local server

The -P option allows you to access the mailboxes via stdin/stdout to a subprocess, rather than over the network.
Dovecot can be run in this mode, for example:

    /usr/lib/dovecot/imap -o mail_location=maildir:~/.mbsync/mails

Typically you might wrap such a command in a script, and then specify the script as the argument of the -P option.


## Acknowledgements etc

For more information, please see [the page on Quentin's site](https://quentinsf.com/software/imapdedup).

This software is released under the terms of the GPL v2.  See the included LICENCE.TXT for details.

It comes with no warranties, express or implied; use at your own risk!

Many thanks to Liyu (Luke) Liu, Adam Horner, Michael Haggerty, 'GargaBou', Stefan Agner, Vincent Bernat, Jan Engels, Fabrizio Mele, Jonathan Vanasco, Kirill A. Korinsky, Thomas Bullinger, Stephen Schwetz and others for their contributions!

[Quentin Stafford-Fraser][1]

[1]:https://statusq.org

