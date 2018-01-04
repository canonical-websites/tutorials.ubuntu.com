---
id: Monitor systems using Nagios
summary: This tutorial will help you to monitor systems using Nagios
categories: community
tags: Beginner,Nagios,monitor systems
difficulty:3
status: published
published: 2018-1-1
author: Nishant Parhi nishantparhi@gmail.com
---

# Monitor systems using Nagios
Duration: 55:00 

## Overview

Nagios is the most popular, open source, powerful monitoring system for any kind of infrastructure.
It enables organizations to identify and resolve IT infrastructure problems before they affect critical business processes. 
Nagios has the capability of monitoring application, services, entire IT infrastructure.

## Prerequisite
Duration: 10:00

Let’s use following commands to install required packages for Nagios.

For Ubuntu 16.04 or Above:
 
    $ sudo apt update
    $ sudo apt install wget build-essential unzip openssl libssl-dev
    $ sudo apt install apache2 php7.0 apache2-mod-php7.0 php7.0-gd libgd-dev 

## Install Nagios Core Service
Duration: 15:00
After installing required dependencies and adding user accounts. Let’s start with Nagios core installation. Download latest Nagios core service from the official site.

    $ cd /opt/
    $ wget https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.3.4.tar.gz
    $ tar xzf nagios-4.3.4.tar.gz
    $ cd nagios-4.3.4
    $ sudo ./configure --with-command-group=nagcmd
    $ sudo make all
    $ sudo make install
    $ sudo make install-init
    $ sudo make install-config
    $ sudo make install-commandmode
Now copy event handlers scripts under libexec directory. These binaries provides multiple events triggers for your Nagios web interface.

   $ cp -R contrib/eventhandlers/ /usr/local/nagios/libexec/
   $ chown -R nagios:nagios /usr/local/nagios/libexec/eventhandlers
Now create nagios apache2 configuration file.
   $ sudo vi /etc/apache2/conf-available/nagios.conf

## Installing Nagios Plugins
Duration: 10:00
After installing and configuring Nagios core service, Download latest nagios-plugins source and install using following commands.

    $ cd /opt
    $ wget http://www.nagios-plugins.org/download/nagios-plugins-2.2.1.tar.gz
    $ tar xzf nagios-plugins-2.2.1.tar.gz
    $ cd nagios-plugins-2.2.1
Now compile and install Nagios plugins

    $ sudo ./configure --with-nagios-user=nagios --with-nagios-group=nagios --with-openssl
    $ sudo make
    $ sudo make install

## Verify Settings
Duration: 5:00
Use the nagios commands to verify Nagios installation and configuration file. After successfully verify start the Nagios core service.

    $ /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    $ service nagios start
Also configure Nagios to auto start on system boot.

## Access Nagios Web Interface
Access your nagios setup by access nagios server using hostname or ip address followed by 
/nagios.
[change domain name with ip]

 http://svr1.tecadmin.net/nagios/

    $ sudo systemctl enable nagios

## Nagios After login screen

Now you have successfully installed and configured Nagios Monitoring Server core service in your system. 
Now visit net article to monitor Linux host and Windows host using Nagios server.

## Now we need to monitor systems using NRPE
Duration: 15:00

Firstly we would require installing nrpe service on remote Linux system, which we need to monitor through Nagios server.

On CentOS/RHEL/Fedora

    yum install nrpe nagios-plugins*
On Debian/Ubuntu/LinuxMint

    $ sudo apt-get install nagios-nrpe-server nagios-plugins
Step 1.2 – Configure NRPE
After successfully installing NRPE service, Edit nrpe configuration file /etc/nagios/nrpe.cfg in your favorite editor and add your nagios service ip in allowed hosts.

    vim /etc/nagios/nrpe.cfg
allowed_hosts=127.0.0.1, 192.168.1.x
Where 192.168.1.100 is your Nagios server ip address.

After making above changes in nrpe configuration file, Lets restart NRPE service as per your system

 service nrpe restart       On CentOS/RHEL/Fedora 
    $ sudo /etc/init.d/nagios-nrpe-server restart  # On Debian/Ubuntu/LinuxMint
## Verify Connectivity from Nagios
Now run the below command from Nagios server to make sure your nagios is able to connect nrpe client on remote Linux system. Here 192.168.1.11 is your remote Linux system ip.

 /usr/local/nagios/libexec/check_nrpe -H 192.168.1.xy

## Add Linux Host in Nagios
Using NagiosQL3 web interface for managing configuration of Nagios server. Below steps is for CLI lovers. To add a host in your Nagios server from command line.
First create a configuration file /usr/local/nagios/etc/servers/MyLinuxHost001.cfg using below values. for example you Linux hosts ip is 192.168.1.11. We also need to define a service with host. So add a ping check service, which will continuously check that host is up or not.

## Now verify configuration files using following command. If there are no errors found in configuration, restart nagios service.

   nagios -v /usr/local/nagios/etc/nagios.cfg
   service nagios restart
## Check Host in Nagios Web Interface
Open your Nagios web interface and check for new Linux hosts added in Nagios core service. 
