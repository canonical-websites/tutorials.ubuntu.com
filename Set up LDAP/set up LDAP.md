Install LDAP
The OpenLDAP server is in Ubuntu's default repositories under the package "slapd", so we can install it easily with apt-get. We will also install some additional utilities:
    sudo apt-get update
    sudo apt-get install slapd ldap-utils

RECONFIGURE SLAPD
When the installation is complete, we actually need to reconfigure the LDAP package. Type the following to bring up the package configuration tool:
sudo dpkg-reconfigure slapd
(You will be asked a series of questions about how you'd like to configure the software)
 

Install PHPldapadmin
We will be administering LDAP through a web interface called PHPldapadmin. This is also available in Ubuntu's default repositories.
Install it with this command:
sudo apt-get install phpldapadmin


Configure PHPldapadmin
We need to configure some values within the web interface configuration files before trying it out.
Open the configuration file with root privileges:
sudo nano /etc/phpldapadmin/config.php

Log Into the Web Interface
You can access by going to your domain name or IP address followed by "/phpldapadmin" in your web browser:
domain_name_or_IP_address/phpldapadmin
