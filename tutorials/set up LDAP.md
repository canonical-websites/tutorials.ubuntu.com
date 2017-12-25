---
id: set up LDAP
summary: This tutorial will help u set up LDAP
categories: community
tags: beginner,LDAP,setup
difficulty:2
status: published
published: 2017-12-21
author: Nishant Parhi nishantparhi@gmail.com
---

#Set up LDAP
Total Duration: 14:00

##Overview
LDAP, or Lightweight Directory Access Protocol, is a protocol for managing related information from a centralized location through the use of a file and directory hierarchy.
It functions in a similar way to a relational database in certain ways, and can be used to organize and store any kind of information. LDAP is commonly used for centralized authentication.
In this guide, we will cover how to install and configure an OpenLDAP server on an Ubuntu 12.04 VPS. We will populate it with some users and groups. In a later tutorial, authentication using LDAP will be covered.


## First we need to install LDAP
Duration: 2:00(DEPENDING ON INTERNET CONNECTION)
 
The OpenLDAP server is in Ubuntu's default repositories under the package "slapd", so we can install it easily with apt-get. We will also install some additional utilities:
    sudo apt-get update
    sudo apt-get install slapd ldap-utils

## The second step is to reconfigure SLAPD
Duration: 2:00

When the installation is complete, we actually need to reconfigure the LDAP package. Type the following to bring up the package configuration tool:
    sudo dpkg-reconfigure slapd
(You will be asked a series of questions about how you'd like to configure the software)

## The third step is to install PHPldapadmin
Duration: 2:00(DEPENDING ON INTERNET CONNECTION)

We will be administering LDAP through a web interface called PHPldapadmin. This is also available in Ubuntu's default repositories.
Install it with this command:
    sudo apt-get install phpldapadmin

## The fourth step is to configure PHPldapadmin
duration: 6:00

We need to configure some values within the web interface configuration files before trying it out.
Open the configuration file with root privileges:
    sudo nano /etc/phpldapadmin/config.php
Change the red value to the way you will be referencing your server, either through domain name or IP address.
    $servers->setValue('server','host','domain_nam_or_IP_address');
For the next part, you will need to reflect the same value you gave when asked for the DNS domain name when we reconfigured "slapd".
You will have to convert it into a format that LDAP understands by separating each domain component. Domain components are anything that is separated by a dot.
These components are then given as values to the "dc" attribute.
For instance, if your DNS domain name entry was "imaginary.lalala.com", LDAP would need to see "dc=imaginary,dc=lalala,dc=com". Edit the following entry to reflect the name you selected (ours is "test.com" as you recall):
    $servers->setValue('server','base',array('dc=test,dc=com'));
The next value to modify will use the same domain components that you just set up in the last entry. Add these after the "cn=admin" in the entry below:
    $servers->setValue('login','bind_id','cn=admin,dc=test,dc=com');
Search for the following section about the "hidetemplatewarning" attribute. We want to uncomment this line and set the value to "true" to avoid some annoying warnings that are unimportant.
    $config->custom->appearance['hide_template_warning'] = true;

## The fifth step is to log into the web interface
Duration: 1:00

You can access by going to your domain name or IP address followed by "/phpldapadmin" in your web browser:
domain_name_or_IP_address/phpldapadmin

## Conclusion

Conclusion
we should now have a basic LDAP server set up with a few users and groups. we can expand this information and add all of the different organizational structures to replicate the structure of your business.
