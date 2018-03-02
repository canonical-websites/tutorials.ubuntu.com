---
id: tutorial-email-server
summary: Learn how to set up postfix and dovecot in order to host an imap or pop3 email server.
categories: server
tags: tutorial, email, dovecot, postfix, server, guide, imap, pop3
difficulty: 4
status: published
feedback_url: https://github.com/canonical-websites/tutorials.ubuntu.com/issues
published: 2017-12-17
author: Varun Patel <varun-patel@live.com>

---

# Install an email server using postfix and dovecot
## Overview
Duration: 3:00

In this tutorial, we will go over the installation and configuration of Postfix and Dovecot to create either an imap or pop3 email server.
We will begin by installing all the programs necessary: MySQL, Postfix and Dovecot. We will then configure each of these.

### What You'll Learn:

* How to install `mail-stack-delivery`
* How to install and configure MySQL
* How to install and configure Postfix
* How to install and configure Dovecot

### What You'll Need

* A computer running Ubuntu 16.04 Xenial Xerus or above
* Updated apt package lists (you can do this in terminal using `sudo apt-get update`)
* An intermediate understanding of how email and email servers work
* To be comfortable using terminal
* A stable internet connection.

Survey
: How will you use this tutorial?
 - Only read through it
 - Read it and complete the exercises
: What is your current level of experience?
 - Novice
 - Intermediate
 - Proficient

## Installing the Required programs
Duration: 5:00

Lets start with MySQL:
```bash
sudo apt-get install mysql-server
```
When propmted to enter a password, ensure that you do so.
Now we need to set up the FQDN we will be using.
```bash
sudo nano /etc/hosts
```
And edit this file with the following
```bash
127.0.0.1	localhost.localdomain	  localhost
127.0.1.1       hostname.yourdomain.tld	  hostname
yourip  	hostname.yourdomain.tld	  hostname
```
Replace the following:
* *HOSTNAME* --> your hostname
* *YOURDOMAIN* --> your domain
* *TLD* --> your top level domain
* *YOURIP* --> your ip address

There are six packages we need to install, they can be installed by:
```bash
sudo apt-get install mail-stack-delivery
```

We can now configure MySQL click next to continue.

## Configuring MySQL
Duration: 7:00

We need to create three MySQL Tables:
* Domains
* Users
* Aliases

First create the mailserver database by entering root and creating a database:
```bash
sudo -i
mysqladmin -p create mailserver
sudo -i -u $USER
```
Replace $USER with your username.

Now we can enter the MySQL root user
```bash
mysql -u root -p
```
If this is successful your prompt should look like `mysql >`

We need to create a mailserver user and grant permissions to it:
```bash
GRANT SELECT ON mailserver.* TO 'usermail'@'127.0.0.1' IDENTIFIED BY 'mailpassword';
FLUSH PRIVILEGES;
```

### Creating the Database

Now we must tell MySQL to use the database `mailserver`
```bash
USE mailserver;
```

Now we can create a domains table:
```bash
CREATE TABLE `virtual_domains` (
`id`  INT NOT NULL AUTO_INCREMENT,
`name` VARCHAR(50) NOT NULL,
PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

Another one for users:
```bash
CREATE TABLE `virtual_users` (
`id` INT NOT NULL AUTO_INCREMENT,
`domain_id` INT NOT NULL,
`password` VARCHAR(106) NOT NULL,
`email` VARCHAR(120) NOT NULL,
PRIMARY KEY (`id`),
UNIQUE KEY `email` (`email`),
FOREIGN KEY (domain_id) REFERENCES virtual_domains(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

And one last one for the aliases:
```bash
CREATE TABLE `virtual_aliases` (
`id` INT NOT NULL AUTO_INCREMENT,
`domain_id` INT NOT NULL,
`source` varchar(100) NOT NULL,
`destination` varchar(100) NOT NULL,
PRIMARY KEY (`id`),
FOREIGN KEY (domain_id) REFERENCES virtual_domains(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

### Populating the Database

First we can populate the domains table:
```bash
INSERT INTO `mailserver`.`virtual_domains`
(`id` ,`name`)
VALUES
('1', 'yourdomain.tld'),
('2', 'hostname.yourdomain.tld');
```
Be sure to replace example.tld and hostname.example.tld with your actual FQDN.

Now for the users table:
```bash
INSERT INTO `mailserver`.`virtual_users`
(`id`, `domain_id`, `password` , `email`)
VALUES
('1', '1', ENCRYPT('secure_password_1', CONCAT('$6$', SUBSTRING(SHA(RAND()), -16))), 'email1@yourdomain.tld'),
('2', '1', ENCRYPT('secure_password_2', CONCAT('$6$', SUBSTRING(SHA(RAND()), -16))), 'email2@yourdomain.tld');
```

Ensure you replace the following with your own values:
* *secure_password_1* & *secure_password_2*
* *email1@yourdomain.tld* & *email2@yourdomain.tld*

Finally we can populate the aliases table:
```bash
INSERT INTO `mailserver`.`virtual_aliases`
(`id`, `domain_id`, `source`, `destination`)
VALUES
('1', '1', 'alias@example.com', 'email1@yourdomain.tld');
```

Make sure to change the values to fit your server.

Now type exit to leave the MySQL Root prompt

## Configuring Postfix
Duration: 15:00

We are now able to configure postfix to handle smtp connections.

### Main Configuration

First, we must edit the main configuation file:
```bash
sudo nano /etc/postfix/main.cf
```
Now find the line: `mydestination = example.com, hostname.example.com, localhost.example.com, localhost` and change it to `mydestination = localhost`

We need to add the following lines as well:
```bash
myhostname = hostname.example.com
# change these values ^^^
virtual_transport = lmtp:unix:private/dovecot-lmtp
virtual_mailbox_domains = mysql:/etc/postfix/mysql-virtual-mailbox-domains.cf
virtual_mailbox_maps = mysql:/etc/postfix/mysql-virtual-mailbox-maps.cf
virtual_alias_maps = mysql:/etc/postfix/mysql-virtual-alias-maps.cf
```

Lastly we can open port 587 to allow a secure connection:
```bash
sudo nano /etc/postfix/master.cf
```
And paste the following:
```bash
submission inet n       -       -       -       -       smtpd
-o syslog_name=postfix/submission
-o smtpd_tls_security_level=encrypt
```

We now need to create and edit the files we referenced in the last 3 lines

### SQL Connection Configuration

First up is the domains table connection:
```bash
sudo nano /etc/postfix/mysql-virutal-mailbox-domains.cf
```
We can add the following lines:
```bash
user = usermail
password = mailpassword
hosts = 127.0.0.1
dbname = mailserver
query = SELECT 1 FROM virtual_domains WHERE name='%s'
```

Next we edit the users table connection:
```bash
sudo nano /etc/postfix/mysql-virutal-mailbox-maps.cf
```
We can add the following lines:
```bash
user = usermail
password = mailpassword
hosts = 127.0.0.1
dbname = mailserver
query = SELECT 1 FROM virtual_users WHERE email='%s'
```

And finally we can edit the aliases table connection:
```bash
sudo nano /etc/postfix/mysql-virutal-alias-maps.cf
```
We can add the following lines:
```bash
user = usermail
password = mailpassword
hosts = 127.0.0.1
dbname = mailserver
query = SELECT 1 FROM virtual_aliases WHERE source='%s'
```
To finish this, we restart the postfix service: `sudo service postfix restart`

## Configuring Dovecot
Duration: 20:00

### Main configuration

We can begin by opening dovecot.conf in an editor:
```bash
sudo nano /etc/dovecot/dovecot.conf
```
Ensure the line `!include conf.d/*.conf` is present.
Now find the line `!include_try /usr/share/dovecot/protocols.d/*.protocol`, immediately below this, add the following:
```bash
protocols = imap lmtp
```
If you would like to enable the pop3 protocol (if you choose to) it can be included here.

Next we will edit the mail configuration file:
```bash
sudo nano /etc/dovecot/conf.d/10-mail.conf
```
Find the line for `mail_location` and set it to `maildir:/var/mail/vhosts/%d/%n`
Next, find the line `mail_priveliged_group` and set it to `mail`

### Permissions and Authentication

Now we are going to deal with permissions:
First we need to make a folder for each domain registered in the mysql table.
```bash
sudo mkdir -p /var/mail/vhosts/yourdomain.tld
sudo mkdir -p /var/mail/vhosts/hostname.yourdomain.tld
sudo groupadd -g 5000 vmail
sudo useradd -g vmail -u 5000 vmail -d /var/mail
sudo chown -R vmail:vmail /var/mail
```

Now we need to edit the authorisation files, begin with:
```bash
sudo nano /etc/dovecot/conf.d/10-auth.conf
```
Uncomment plaintext authentication and add the line `disable_plaintext_auth = yes`
Now modify the `auth_mechanisms` parameter to be `plain login`

It is now time for us to ecit the sql auth files:
```bash
sudo nano /etc/dovecot/conf.d/auth-sql.conf.ext

#enter the following

passdb {
  driver = sql
  args = /etc/dovecot/dovecot-sql.conf.ext
}
userdb {
  driver = static
  args = uid=vmail gid=vmail home=/var/mail/vhosts/%d/%n
}
```
Now we edit the file we just referenced:
```bash
sudo nano /etc/dovecot/dovecot-sql.conf.ext

# uncomment and change the following lines

driver = mysql
connect = host=127.0.0.1 dbname=mailserver user=usermail password=mailpassword
default_pass_scheme = SHA512-CRYPT
password_query = SELECT email as user, password FROM virtual_users WHERE email='%u';
```
Now change the ownership of the dovecot folder:
```bash
sudo chown -R vmail:dovecot /etc/dovecot
sudo chmod -R o-rwx /etc/dovecot
```
Finally we are at the last file `sudo nano /etc/dovecot/conf.d/10-master.conf`, modify the following:
```bash
##Uncomment inet_listener_imap and modify to port 0
service imap-login {
  inet_listener imap {
    port = 0
}

#Create LMTP socket and this configurations
service lmtp {
   unix_listener /var/spool/postfix/private/dovecot-lmtp {
	   mode = 0600
	   user = postfix
	   group = postfix
   }
  #inet_listener lmtp {
    # Avoid making LMTP visible for the entire internet
    #address =
    #port =
  #}
}



service auth {

  unix_listener /var/spool/postfix/private/auth {
  mode = 0666
  user = postfix
  group = postfix
  }

  unix_listener auth-userdb {
  mode = 0600
  user = vmail
  #group =
  }

  #unix_listener /var/spool/postfix/private/auth {
  # mode = 0666
  #}

  user = dovecot
}


service auth-worker {
  # Auth worker process is run as root by default, so that it can access
  # /etc/shadow. If this isn't necessary, the user should be changed to
  # $default_internal_user.
  user = vmail
}
```
Now we restart Dovecot:
```bash
sudo service dovecot restart
```

## You're Done!
Duration: 2:00

We can now connect to this server using any common mail client using port 993 for imap, port 995 for pop3 and port 587 or 25 for smtp
You may want to look into [spamassassin for mail filtering](https://tutorials.ubuntu.com/tutorial/install-spamassassin)

###You now know how to:

* Prepare an environment to install an email server
* Install `mail-stack-delivery`
* Create a MySQL Database for email
* Configure Postfix
* Configure Dovecot

###What's Next?

* Modify your external internet connection to forward ports 993, 995, 587 and 25
* Ensure your network has a static IP address
* Set up a spam filtering service like [spamassassin](https://tutorials.ubuntu.com/tutorial/install-spamassassin)
* Set up email clients

###I Need Help

* Double Check that the port is available
* Ensure the configuration files are correct
* Make sure you typed the commands properly
* Try using sudo (if you arent already)
* Make sure the information you entered is the same throughout the installation
* Check your dns for A and MX entries
* Make sure the same passwords were used throughout
* Ask a question on [Ask Ubuntu](https://askubuntu.com/questions/ask)
