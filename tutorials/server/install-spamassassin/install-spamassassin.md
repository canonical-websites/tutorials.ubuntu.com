---
id: tutorial-spam-filtering
summary: Learn how to add spam filtering and antivirus capabilities to a postfix mail server. This tutorial assumes that you already have a working mail server.
categories: server
tags: tutorial, spam, Spamassassin, ClamAV, mail, server, postfix, email, filter, gci
difficulty: 4
status: published
feedback_url: https://github.com/canonical-websites/tutorials.ubuntu.com/issues
published: 2018-01-13
author: Varun Patel <varun-patel@live.com>

---

# Add spam filtering to a mail server

## Overview
Duration: 2:00

In this tutorial, we will go over how to install Spamassassin and ClamAV to an existing mailserver installation, these provide spam filtering services to the postfix installation as well as an antivirus to prevent malicious emails.

### What You'll Learn

* How to install Spamassasin and ClamAV
* How to install Amavis
* How to configure Spamassasin, ClamAV and Amavis
* How to test a spam filter system

### What You'll Need

* A computer running Ubuntu 16.04 Xenial Xerus or above
* Updated apt package lists (you can do this in terminal using `sudo apt-get update`)
* A basic understanding of how mail servers and spam filters wotk
* A working mail server installation using postfix
* To be comfortable using terminal
* A stable internet connection

Survey
: How will you use this tutorial?
- Only read through it
- Read it and complete the exercises
: What is your current level of experience?
- Novice
- Intermediate
- Proficient

## Installing Programs
Duration:1:00

We will install six programs using apt
This is very simple:
```bash
sudo apt install amavisd-new spamassassin clamav-daemon opendkim postfix-policyd-spf-python pyzor razor arj cabextract cpio lhasa nomarch pax rar unrar unzip zip
```

We now have all the programs necessary to run a spam filtering service.

## Configuration
Duration: 10:00

We can begin by allowing Amavis (the spam filter) to access ClamAV (the antivirus) and vice versa;
```bash
sudo adduser clamav amavis
sudo adduser amavis clamav
```

### Spamassassin Configuration

Now we need to enable Spamassassin, this is done in `/etc/default/spamassassin`
```bash
sudo nano /etc/default/spamassassin
```
***change the following***
```bash
ENABLED=0
```
***to***
```bash
ENABLED=1
```
Spamassassin now needs a restart
```bash
sudo service spamassassin restart
```
### Amavis Configuration

Now we can enable Amavisd and intergrate it with Spamassassin and ClamAV
```bash
sudo nano /etc/amavis/conf.d/15-content_filter_mode
```
We need to uncomment the lines that follow by removing the `#`
```bash
@bypass_virus_checks_maps = (
   \%bypass_virus_checks, \@bypass_virus_checks_acl, \$bypass_virus_checks_re);
 ```
 and
 ```bash
 @bypass_spam_checks_maps = (
   \%bypass_spam_checks, \@bypass_spam_checks_acl, \$bypass_spam_checks_re);
```
We now need to set the filter to discard unwanted emails rather than bounce them as well as tighten the spam filter parameters. since this is a large file it is easiest to remove it completely and create a new one with the instructions we need:
```bash
sudo rm /etc/amavis/conf.d/20-debian_defaults
sudo nano /etc/amavis/conf.d/20-debian_defaults
```
copy and paste the following into the new config file:
```bash
use strict;
$QUARANTINEDIR = "$MYHOME/virusmails";
$quarantine_subdir_levels = 1; # enable quarantine dir hashing
$log_recip_templ = undef;    # disable by-recipient level-0 log entries
$DO_SYSLOG = 1;              # log via syslogd (preferred)
$syslog_ident = 'amavis';    # syslog ident tag, prepended to all messages
$syslog_facility = 'mail';
$syslog_priority = 'debug';  # switch to info to drop debug output, etc
$enable_db = 1;              # enable use of BerkeleyDB/libdb (SNMP and nanny)
$enable_global_cache = 1;    # enable use
$inet_socket_port = 10024;   # default listening socket
$sa_spam_subject_tag = '***SPAM*** ';
$sa_tag_level_deflt = -999; # add spam info headers if at, or above that level
$sa_tag2_level_deflt = 6.0; # add 'spam detected' headers at that level
$sa_kill_level_deflt = 21.0; # triggers spam evasive actions
$sa_dsn_cutoff_level = 4; # spam level beyond which a DSN is not sent
$sa_mail_body_size_limit = 200*1024; # don't waste time on SA if mail is larger
$sa_local_tests_only = 0;    # only tests which do not require internet access?
$MAXLEVELS = 14;
$MAXFILES = 1500;
$MIN_EXPANSION_QUOTA =      100*1024;  # bytes
$MAX_EXPANSION_QUOTA = 300*1024*1024;  # bytes
$final_virus_destiny      = D_DISCARD;  # (data not lost, see virus quarantine)
$final_banned_destiny     = D_BOUNCE;   # D_REJECT when front-end MTA
$final_spam_destiny       = D_DISCARD;
$final_bad_header_destiny = D_PASS;     # False-positive prone (for spam)
$enable_dkim_verification = 0; #disabled to prevent warning
$virus_admin = "postmaster\@$mydomain"; # due to D_DISCARD default
$X_HEADER_LINE = "Debian $myproduct_name at $mydomain";
@viruses_that_fake_sender_maps = (new_RE(
  [qr'\bEICAR\b'i => 0],            # av test pattern name
  [qr/.*/ => 1],  # true for everything else
));
@keep_decoded_original_maps = (new_RE(
  qr'^MAIL-UNDECIPHERABLE$', # recheck full mail if it contains undecipherables
  qr'^(ASCII(?! cpio)|text|uuencoded|xxencoded|binhex)'i,
));
$banned_filename_re = new_RE(
  qr'\.[^./]*\.(exe|vbs|pif|scr|bat|cmd|com|cpl|dll)\.?$'i,
  qr'\{[0-9a-f]{8}(-[0-9a-f]{4}){3}-[0-9a-f]{12}\}?$'i, # Windows Class ID CLSID, strict
  qr'^application/x-msdownload$'i,                  # block these MIME types
  qr'^application/x-msdos-program$'i,
  qr'^application/hta$'i,
  qr'.\.(exe|vbs|pif|scr|bat|cmd|com|cpl)$'i, # banned extension - basic
);
@score_sender_maps = ({ # a by-recipient hash lookup table,
  '.' => [  # the _first_ matching sender determines the score boost
   new_RE(  # regexp-type lookup table, just happens to be all soft-blacklist
    [qr'^(bulkmail|offers|cheapbenefits|earnmoney|foryou)@'i         => 5.0],
    [qr'^(greatcasino|investments|lose_weight_today|market\.alert)@'i=> 5.0],
    [qr'^(money2you|MyGreenCard|new\.tld\.registry|opt-out|opt-in)@'i=> 5.0],
    [qr'^(optin|saveonlsmoking2002k|specialoffer|specialoffers)@'i   => 5.0],
    [qr'^(stockalert|stopsnoring|wantsome|workathome|yesitsfree)@'i  => 5.0],
    [qr'^(your_friend|greatoffers)@'i                                => 5.0],
    [qr'^(inkjetplanet|marketopt|MakeMoney)\d*@'i                    => 5.0],
   ),
  ],  # end of site-wide tables
});
1;  # ensure a defined return
```
Amavis now needs a restart:
```bash
sudo service amavis restart
```

### Postfix Configuration

We need to enable the content filter in postfix;
```bash
sudo postconf -e 'content_filter = smtp-amavis:[127.0.0.1]:10024'
```

Now we can add some lines to the postfix master.cf;
```bash
sudo nano /etc/postfix/main.cf
```
add these lines to the bottom of `/etc/postfix/main.cf`
```bash
smtp-amavis     unix    -       -       -       -       2       smtp
        -o smtp_data_done_timeout=1200
        -o smtp_send_xforward_command=yes
        -o disable_dns_lookups=yes
        -o max_use=20
127.0.0.1:10025 inet    n       -       -       -       -       smtpd
        -o content_filter=
        -o local_recipient_maps=
        -o relay_recipient_maps=
        -o smtpd_restriction_classes=
        -o smtpd_delay_reject=no
        -o smtpd_client_restrictions=permit_mynetworks,reject
        -o smtpd_helo_restrictions=
        -o smtpd_sender_restrictions=
        -o smtpd_recipient_restrictions=permit_mynetworks,reject
        -o smtpd_data_restrictions=reject_unauth_pipelining
        -o smtpd_end_of_data_restrictions=
        -o mynetworks=127.0.0.0/8
        -o smtpd_error_sleep_time=0
        -o smtpd_soft_error_limit=1001
        -o smtpd_hard_error_limit=1000
        -o smtpd_client_connection_count_limit=0
        -o smtpd_client_connection_rate_limit=0
        -o receive_override_options=no_header_body_checks,no_unknown_recipient_checks,no_milters
```
postfix now needs to be restarted
```bash
sudo service postfix restart
```

## Testing the configuration

We can test the configuration with the following command:
```bash
telnet localhost 10024
```
you should see the following
```bash
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
220 [127.0.0.1] ESMTP amavisd-new service ready
```

## You're Done!
Duration: 2:00

We are now actively filtering spam and viruses using Amavis, Spamassassin and ClamAV.

###You now know how to:

* Prepare an environment to install a spam filter
* Install Amavis, Spamassassin and ClamAV
* Configure Amavis, Spamassassin and ClamAV

###What's Next?

* send some spam emails to test the configuration
* change `/etc/amavis/conf.d/20-debian_defaults` to filter what you want filtered

###I Need Help

* Double check that everything started properly (sudo service x restart)
* Check your router's port forwarding configuration (external connection)
* Ensure the configuration files are correct
* Make sure you typed the commands properly
* Try using sudo (if you aren't already) i.e. `sudo` + `command`
* Ask a question on [Ask Ubuntu](https://askubuntu.com/questions/ask)
