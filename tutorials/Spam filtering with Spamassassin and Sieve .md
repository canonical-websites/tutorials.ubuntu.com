---
id: Spam filtering with spamassassin and Sieve
summary: This tutorial will help you filter spam with spamassassin and Sieve
categories: community
tags: Beginner,spam,spamassassin,Sieve
difficulty:2
status: published
published: 2018-1-6
author: Nishant Parhi nishantparhi@gmail.com
---

# Spam filtering with spamassassin and Sieve
Duration: 55:00 

## Overview

SpamAssassin is a computer program used for e-mail spam filtering.SpamAssassin uses a 
variety of spam-detection techniques, including DNS-based and 
fuzzy-checksum-based spam detection, Bayesian filtering, external programs, blacklists
and online databases.

##Prerequisites
Before installing Spamassassin, you need to install and setup a mail transfer agent
such as Postfix on your virtual private server.

## Installation of Spamassassin
Duration: 5:00

    apt-get to install Spamassassin spamc
Once Spamassassin is installed, there are a few steps that has to be taken to 
make it fully functional.
To run Spamassassin you need to create a new user on your VPS.

## First add the group spams
Duration 10:00

    groupadd spamd
Add the user spamd with the home directory /var/log/spamassassin
    useradd -g spamd -s /bin/false -d /var/log/spamassassin spamd
then create the directory /var/log/spamassassin
    mkdir /var/log/spamassassin
and change the ownership of the directory to spams
    chown spamd:spamd /var/log/spamassassin
## Set up Spamassassin
Duration: 10:00 

Setting Up Spamassassin
Open the spamassassin config file using:
    nano /etc/default/spamassassin
To enable Spamassassin find the line
    ENABLED=0
    and change it to
    ENABLED=1
To enable automatic rule updates in order to get the latest spam filtering rules 
find the line
    CRON=0
    and change it to
    CRON=1

## Create a variable named SAHOME with the Spamassassin home directory
Duration: 5:00
 
    SAHOME="/var/log/spamassassin/"
Find and change the OPTIONS variable to
    OPTIONS="--create-prefs --max-children 2 --username spamd \
    -H {SAHOME} -s {SAHOME}spamd.log"
This specifies the username Spamassassin will run under as spamd, as well as
add the home directory, create the log file, and limit the child processes that 
Spamassassin can run.

## Start the Spamassassin daemon
Duration: 5:00

service spamassassin start
Now, let's config Postfix.
Configuring Postfix
The emails still do not go through Spamassasin. To do that, open Postfix config file using:
    nano /etc/postfix/master.cf
Find the the line

    smtp      inet  n       -       -       -       -       smtpd
and add the following
    -o content_filter=spamassassin
Now, Postfix will pipe the mail through Spamassassin.

## Setup after-queue content filter
Duration: 10:00

    spamassassin unix -     n       n       -       -       pipe
            user=spamd argv=/usr/bin/spamc -f -e  
            /usr/sbin/sendmail -oi -f {sender} {recipient}
For the changes to take effect restart postfix:
    service postfix restart
Now postfix will use spamassassin as a spam filter.
Configuring Spamassassin on your VPS

To get the maximum use of Spamassassin you have to create rules
nano /etc/spamassassin/local.cf
To activate a rules uncomment line remove the # symbol.

To add a spam header to spam mail uncomment or add the line:

    rewrite_header Subject [***** SPAM _SCORE_ *****]
Spamassassin gives a score to each mail after running different tests on it. The following line will mark the mail as spam if the score is more than the value specified in the rule.

    required_score           3.0
To use bayes theorem to check mails, uncomment or add the line:

    use_bayes               1
To enable bayes auto learning, uncomment or add the line:

    bayes_auto_learn        1
After adding the above details, save the file and restart spam assassin.

    service spamassassin restart
Testing
To see if Spamassassin is working, you can check the spamassassin log file using:

    nano /var/log/spamassassin/spamd.log
or send the email from an external server and check the mail headers.

## Sieve mail filtering setup
Duration: 10:00

Email filtering e.g. showing trash can to newsletters who do not honor unsubscribe links automatically. This is like Gmail filters.
Global spam filtering e.g. moving spam from inbox to junk folder (automatically)
Installing packages for sieve and managesieve
We are using dovecot pigeonhole project for sieve support.

## Instalation of sieve

    apt-get install dovecot-sieve dovecot-managesieved
Please note that, installing sieve without spamassassin won’t filter spam messages automatically.

Sieve-Dovecot Configuration
Enable sieve plugin support for dovecot-lmtp
    vim /etc/dovecot/conf.d/20-lmtp.conf
Add following:

protocol lmtp {
  postmaster_address = admin@example.com
  mail_plugins = mail_plugins sieve
}
Edit sieve dovecot-pluign configuration
vim /etc/dovecot/conf.d/90-sieve.conf
Add following:

plugin {
   sieve = ~/.dovecot.sieve
   sieve_global_path = /var/lib/dovecot/sieve/default.sieve
   sieve_dir = ~/sieve
   sieve_global_dir = /var/lib/dovecot/sieve/
}
Restart Dovecot
Restart dovecot for changes to take effect:

    service dovecot restart
You can see managesieve service running on port number 4190 by using telnet command:

    telnet example.com 4190
Will output something like:

Trying 162.243.12.140...
Connected to test3.rtcamp.com.
Escape character is '^]'.
"IMPLEMENTATION" "Dovecot Pigeonhole"
"SIEVE" "fileinto reject envelope encoded-character vacation subaddress comparator-i;ascii-numeric relational regex imap4flags copy include variables body enotify environment mailbox date ihave"
"NOTIFY" "mailto"
"SASL" "PLAIN LOGIN"
"STARTTLS"
"VERSION" "1.0"
OK "Dovecot ready."
You can find sieve commands here which you can run inside telnet shell to manage sieve filters. Or you can use following method to manipulate global sieve rules directly.

Global Sieve Rules
You can use sieve to implement/enforce server-wise rules/organization policies.

Create global sieve rules file
    mkdir /var/lib/dovecot/sieve/
Then create/open global sieve rules files:

    vim /var/lib/dovecot/sieve/default.sieve
Following example rules moves spam emails from Inbox to Junk folder automatically. X-Spam-Flag is added by spamassassin and amavis.

require "fileinto";
    if header :contains "X-Spam-Flag" "YES" {
      fileinto "Junk";
    }
Change Ownership:

    chown -R vmail:vmail /var/lib/dovecot
Compile sieve rules:

    sievec /var/lib/dovecot/sieve/default.sieve
At this point you have sieve interpreter and managesieve service running.

Enabling sieve plugin in roundcube
Roundcube has 2 plugins for sieve:managesieve andsieverules.

Both provides similar functionality but I like support for extended mail-headers in UI provided by sieverules. So we will be using sieverules plugin.

Enable sieverules plugin in roundcube config
Open

    vim /etc/roundcube/main.inc.php
Add sieverules to list of roundcube plugins

    rcmail_config['plugins'] = array('sieverules');
Configure sieverules plugin to use correct managesieve service port
Open

    vim /etc/roundcube/plugins/sieverules/config.inc.php
Add/Update following line:

    rcmail_config['sieverules_port'] = 4190;





