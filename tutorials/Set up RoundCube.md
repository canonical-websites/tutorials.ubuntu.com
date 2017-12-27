---
id: Set up RoundCube in ubuntu
summary: This tutorial will help you set up RoundCube
categories: community
tags: Beginner,RoundCube,setup
difficulty:2
status: published
published: 2017-12-25
author: Nishant Parhi nishantparhi@gmail.com
---

# Set up RoundCube
Duration: 30:00

## Overview

Roundcube is a web-based IMAP email client that offers a user interface similar to Google’s Gmail or Hotmail. 
It is a server-side application written in PHP designed to access an email server or service. 
Email users interact with Roundcube over the internet using a web browser.

## Install LAMP Stack Packages 
Duration: 3:00(depends on internet speed)

Install the lamp-server metapackage, which installs Apache, MySQL, and PHP as dependencies:
    sudo apt-get install lamp-server
During the installation process, you will be asked to choose a password for the root MySQL user.
Secure your new MySQL installation:
    sudo mysql_secure_installation
Specify your Linode’s time zone in the /etc/php5/apache2/php.ini PHP configuration file. If your server is not using UTC, replace it with your local timezone listed on PHP.net:
    sudo sed -i -e "s/^;date\.timezone =.*$/date\.timezone = 'UTC'/" /etc/php5/apache2/php.ini

## Create an Apache Virtual Host with SSL
Duration: 3:00

We will create a new virtual host for Roundcube in this section. This makes a new webroot for Roundcube, separating it from any other webroots on your Linode.
Position your Linode’s shell prompt within the /etc/apache2/sites-available directory:
    cd /etc/apache2/sites-available
Download a copy of our apache2-roundcube.sample.conf virtual host configuration file. Replace instances of webmail.example.com with the desired domain or subdomain of your installation.
    sudo wget https://linode.com/docs/assets/apache2-roundcube.sample.conf
Transfer the file’s ownership to root:
    sudo chown root:root apache2-roundcube.sample.conf
Next, change the file’s access permissions:
    sudo chmod 644 apache2-roundcube.sample.conf

## 3rd step
Duration: 2:00

Once you have your SSL certificate, edit the following options in apache2-roundcube.sample.conf to match your desired configuration:
  ServerAdmin: administrative email address for your Linode (e.x. admin@example.com or webmaster@example.com)
  ServerName: full domain name of the virtual host (e.x. webmail.example.com)
  ErrorLog (optional): path to the custom error log file (e.x. /var/log/apache2/webmail.example.com/error.log; uncomment by removing #)
  CustomLog (optional): path to the custom access log file (e.x. /var/log/apache2/webmail.example.com/access.log; again, uncomment by removing #)
  SSLCertificateFile: path to the SSL certificate information (.crt) file
  SSLCertificateKeyFile: path to the SSL certificate private key (.key) file 

## Rename your configuration
Duration: 1:00

Rename your configuration file to match its full domain name:
    sudo mv apache2-roundcube.sample.conf webmail.example.com.conf
Lastly, disable the default Apache virtual host unless you plan to use it.
    sudo a2dissite 000-default.conf default-ssl.conf

## Create a MySQL Database and User
Duration: 5:00

Log into the MySQL command prompt as the root user:
    mysql -u root -p
Once logged in and the mysql> prompt is shown, create a new MySQL database called roundcubemail:
    CREATE DATABASE roundcubemail;
Create a new MySQL user called roundcube and assign it a strong password:
    CREATE USER 'roundcube'@'localhost' IDENTIFIED BY 'example_password';
Grant the new roundcube user full access to Roundcube’s database roundcubemail:
    GRANT ALL PRIVILEGES ON roundcubemail.* TO 'roundcube'@'localhost';
Flush the MySQL privilege tables to reload them:
    FLUSH PRIVILEGES;
Log out of the MySQL command prompt and return to a regular Linux shell prompt:
    exit

## Final Preparation for RoundCube
Duration: 3:00

Install and enable the packages php-pear, php5-intl, and php5-mcrypt:
    sudo apt-get install php-pear php5-intl php5-mcrypt && sudo php5enmod intl mcrypt
Enable the Apache modules deflate, expires, headers, rewrite, and ssl:
    sudo a2enmod deflate expires headers rewrite ssl
Additionally, install the PHP PEAR packages Auth_SASL, Net_SMTP, Net_IDNA2-0.1.1, Mail_mime, and Mail_mimeDecode:
    sudo pear install Auth_SASL Net_SMTP Net_IDNA2-0.1.1 Mail_mime Mail_mimeDecode

## Download and Install RoundCube
Duration: 5:00

Make sure your Linode’s shell prompt is operating inside your user’s home directory. The ~/Downloads folder is preferable, but ~/ is also acceptable.
    cd ~/Downloads
Download Roundcube. At the time of this writing, the current stable version is 1.1.4, so it will be used for the rest of this guide.
    wget http://downloads.sourceforge.net/project/roundcubemail/roundcubemail/1.1.4/roundcubemail-1.1.4.tar.gz
Decompress and copy Roundcube to the /var/www directory. Again, replace any occurrences of 1.1.4 in the filename with a newer version number if necessary:
    sudo tar -zxvf roundcubemail-1.1.4.tar.gz -C /var/www
Eliminate the version number from Roundcube’s directory name. This will make updating easier later:
    sudo mv /var/www/roundcubemail-1.1.4 /var/www/roundcube
Transfer ownership of the /var/www/roundcube directory to the www-data user. This will allow Roundcube to save its own configuration file, instead of you having to download it and then manually upload it to your Linode:
    sudo chown -R www-data:www-data /var/www/roundcube
Lastly, you should enable Roundcube’s automatic cache-cleaning shell script:
    echo '0 0 * * * root bash /var/www/roundcube/bin/cleandb.sh >> /dev/null' | sudo tee --append /etc/crontab
This utilizes a cron job to run the cleandb.sh shell script included with Roundcube once per day at midnight. Read our Scheduling Tasks with Cron guide to learn about Cron.

## Enable RoundCube's Apache Virtual Host
Duration: 2:00

Enable the webmail.example.com virtual host you just wrote in the Creating an Apache Virtual Host with SSL section:
    sudo a2ensite webmail.example.com.conf
Restart Apache to apply all configuration changes and enable your new virtual host:
    sudo service apache2 restart
The output should be * Restarting web server apache2 ... [ OK ]. If an error is given, use the error messages to troubleshoot your configuration. Missing files, incorrect permissions and typos are common causes for Apache not properly restarting.

## Configure RoundCube
Duration: 4:00

Launch your favorite web browser and navigate to https://webmail.example.com/installer. Again, make sure to replace webmail.example.com with your chosen domain name.
Begin configuring Roundcube. 
The first step of Roundcube’s graphical configuration is an environment check. 
Click on the NEXT button at the bottom of the page to continue.
Specify your Roundcube configuration options. 
The list of options below will get you a proper, working configuration, but you can adjust any unmentioned options as you see fit.
  General configuration > product_name: Name of your email service.
  General configuration > support_url: Where should your users go if they need help? A URL to a web-based contact form or an email address should be used. (e.g. http://example.com/support or mailto:support@example.com)
  General configuration > skin_logo: Replaces the default Roundcube logo with an image of your choice. The image must be located within the /var/www/roundcube directory and be linked relatively (e.g. skins/larry/logo.png). Recommended image resolution is 177px by 49px.
  Database setup > db_dsnw > Database password: Password for the roundcube MySQL user you created earlier.
  IMAP Settings > default_host: Hostname of your IMAP server. Use ssl://localhost to access the local server (i.e. your server) using OpenSSL.
  IMAP Settings > default_port: TCP port for incoming IMAP connections to your server. Use port 993 to ensure OpenSSL is used.
  IMAP Settings > username_domain: What domain name should Roundcube assume all users are part of? This allows users to only have to type in their email username (e.g. somebody) instead of their full email address (e.g. somebody@example.com).
  SMTP Settings > smtp_server: Hostname of your SMTP server. Use ssl://localhost to access the local server (i.e. your server) using OpenSSL.
  SMTP Settings > smtp_port: TCP port for incoming SMTP connections to your server. Use port 587 to ensure OpenSSL is used.
  SMTP Settings > smtp_user/smtp_pass: Click and check the Use the current IMAP username and password for SMTP authentication checkbox so that users can send mail without re-typing their user credentials.
  Display settings & user prefs > language: Allows you to select a default RFC1766-compliant locale for Roundcube. For a full listing of the supported language codes, run cat /usr/share/i18n/SUPPORTED on your Linode.
  Display settings & user prefs > draft_autosave: Most users will expect their drafts to be saved almost instantaneously while they type them. While Roundcube does not offer instantaneous draft saving as an option, it can save a user’s draft every minute. Select 1 min from the dropdown menu.
  Click on the CREATE CONFIG button toward the bottom of the page to save your new configuration. You should see a confirmation message on the corresponding page saying: The config file was saved successfully into RCMAIL_CONFIG_DIR directory of your Roundcube installation.
  Complete the configuration by clicking CONTINUE.

## Verify your Roundcube Installation
duration: 1:00

Navigate to https://webmail.example.com and log in using your email account’s username and password. If your configuration is functional, Roundcube will allow you to receive, read and send emails from inside and outside of your domain name.


Keep RoundCube Updated

Compare the Stable > Complete package version listed on Roundcube’s download page to the version currently installed on your Linode.
If a newer version is available, replace any occurrences of 1.1.4 with the newest version in the command below. This will download Roundcube to your ~/Downloads directory:
    cd ~/Downloads && wget http://downloads.sourceforge.net/project/roundcubemail/roundcubemail/1.1.4/roundcubemail-1.1.4.tar.gz
Extract and unzip the tarball (.tar.gz file) to ~/Downloads:
    tar -zxvf roundcubemail-1.1.4.tar.gz
Begin updating Roundcube by executing the /var/www/roundcube/bin/installto.sh PHP script. If you did not install Roundcube in the /var/www/roundcube directory, replace the trailing directory with that of Roundcube’s on your server:
    cd roundcubemail-1.1.4
    sudo php bin/installto.sh /var/www/roundcube
    Confirm the update by pressing Y and then ENTER. A successful upgrade will print something similar to this:
    Upgrading from 1.1.4. Do you want to continue? (y/N)
    y
    Copying files to target location...sending incremental file list
    ...
    Running update script at target...
    Executing database schema update.
    This instance of Roundcube is up-to-date.
All done means the update was successful; unless you don’t see this message, proceed to step six.
Delete the Roundcube directory and gzipped tarball from ~/Downloads:
    cd ~/Downloads && rm -rfd roundcubemail-1.1.4 roundcubemail-1.1.4.tar.gz

## Conclusion

Now that you have installed Roundcube, you have your very own free, web-based email client similar to Google’s Gmail or Microsoft’s Hotmail. Users can access their email by navigating to https://webmail.example.com.
From here, you can install plugins to add additional functionality and customize the theme to match your organization’s color scheme.




