---
id: tutorial-vpn-client
summary: In this tutorial you will learn how to install openvpn and use it as a client for a vpn server.
categories: server
tags: vpn, OpenVPN, client, guide, contribute, secure, security
difficulty: 2
status: published
feedback_url: https://github.com/canonical-websites/tutorials.ubuntu.com/issues
published: 2017-12-11
author: Varun Patel <varun-patel@live.com>

---
#Set Up a VPN Client Using OpenVPN

##Overview
Duration: 3:00

In this tutorial, you will learn how to take a `.ovpn` configuration file and use it to connect to a vpn server, some basic skills using terminal are recommended.

We will begin by installing OpenVPN in terminal using `apt`, adjust some DNS settings to prevent DNS leak and finally, load the `.ovpn` configuration file.

###What you'll learn

* How to prepare your environment for the installation of OpenVPN
* How to install OpenVPN through apt
* How to edit DNS settings to prevent dns leak

###What you'll need

* A computer running Ubuntu 16.04 Xenial Xerus or above
* Updated apt package lists (you can do this through terminal by typing `sudo apt-get update`)
* A basic understanding of how VPNs work
* A stable internet connection
* A `.ovpn` configuration file

You may want to check the tutorial: [Install A VPN Server](http://tutorials.ubuntu.com/tutorials/tutorial-vpn-server) to host your own VPN server

Survey
: How will you use this tutorial?
 - Only read through it
 - Read it and complete the exercises
: What is your current level of experience?
 - Novice
 - Intermediate
 - Proficient

##Installing OpenVPN
Duration: 1:00

The first step is to install OpenVPN, this is as simple as typing the following into terminal:
```bash
sudo apt-get updated
sudo apt-get install openvpn
```

Once this is complete, OpenVPN is installed,

##Adjusting your DNS
Duration: 3:00

In order to do this, we must edit the `.ovpn` configuration file. This prevents DNS leak.
```bash
sudo nano client1.ovpn
```
Find the following lines in the file and uncomment them.

```bash
script-security 2
up /etc/openvpn/update-resolv-conf
down /etc/openvpn/update-resolv-conf
```
Save and exit nano.

##Add the file to OpenVPN
Duration: 1:00

This is a very simple process, only a single line in terminal:
```bash
sudo openvpn --config client1.ovpn
```

##We're Done
Duration: 2:00

You are now connected to a vpn serve.

###Now you know how to:

* Connect to a VPN Server using an OpenVPN configuration file
* Prevent DNS leak by changing the DNS

### Next steps

* Take a look at hosting your own VPN server using the [Setting Up a VPN Server](http://tutorials.ubuntu.com/tutorials/tutorial-vpn-server) tutorial
