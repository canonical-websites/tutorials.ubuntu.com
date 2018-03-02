---
id: tutorial-vpn-server
summary: Learn how to install and configure an OpenVPN server and create client configurations. This tutorial covers some more advanced topics like networking and encryption, some background is helpful.
categories: server
tags: VPN, OpenVPN, , server, guide, tutorial
difficulty: 4
status: published
published: 2017-12-07
feedback-url: https://github.com/canonical-websites/tutorials.ubuntu.com/issues
author: Varun Patel <varun-patel@live.com>

---
#Set up an VPN Server Using OpenVPN

##Overview
Duration: 2:30

In this tutorial, you will learn now to install a VPN server onto your existing Ubuntu installation using OpenVPN. This tutorial is recommended for users who are comfortable with using the terminal.

Over the course of this tutorial, we will install dependencies for OpenVPN, install OpenVPN using apt, set up encryption certificates, configure OpenVPN and learn how to run it and create client certificates.

###What you'll learn

* How to prepare your environment for the installation of OpenVPN
* How to install OpenVPN through apt
* How to create and use encryption certificates
* How to begin using the server end of OpenVPN

### What you'll need

* A computer running Ubuntu 16.04 Xenial Xerus or above
* Updated apt package lists ( you can do this through terminal by typing `sudo apt-get update`)
* An intermediate understanding of how VPNs work
* A basic understanding of how encryption certificates and CAs work
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

## Prerequisites
Duration: 1:30

### Installing an Internal Certificate Authority

In order to encrypt our VPN connection, we must be running a local certificate autority, this creates encryption keys. For the purposes of this tutorial we will use the `easy-rsa` package. This package can be installed using apt:
```bash
sudo apt-get install easy-rsa
```

### Installing OpenVPN

To host this VPN server, we will use OpenVPN. OpenVPN is a well tested and comprehensive tool. It can be installed using apt:
```bash
sudo apt-get install openvpn
```

## Configuring a Certificate Authority
Duration: 10:00

OpenVPN utilizes certificates in order to encrypt connections between the server and clients. In order to create trusted certificates, we will need to set up a certificate authority (CA).

To begin, we need to copy the easy-rsa template into our user's home directory with the `make-cadir` command:
```bash
make-cadir ~/ovpn-ca
```
Now we must enter the newly created directory
```bash
cd ovpn-ca
```
Because of the nature of Certificate Authorities, we need to include comprehensive information to make sure everything works smoothly. This information is located in the variables file. We need to edit this:
```bash
nano vars
```
To begin with you can copy the following:
```bash
. . .

export KEY_COUNTRY="COUNTRY_CODE"
export KEY_PROVINCE="PROVINCE_CODE"
export KEY_CITY="CITY_NAME"
export KEY_ORG="ORGANISATION_NAME"
export KEY_EMAIL="EMAIL"
export KEY_OU="ORGANISATIONAL_UNIT"
export KEY_NAME="vpn"

. . .
```
Now you need to change the following
* *COUNTRY_CODE* --> The two character code for the country you are in. i.e. The United Kingdon is GB, The United States of America is US and Canada is CA
* *PROVINCE_CODE* --> The code for the province/state/region you are in. i.e. California is CA, New York is NY, Ontario is ON
* *CITY_NAME* --> The name of the city you are in. i.e. London, New York, Tokyo, etc.
* *ORGANISATION_NAME* --> The name of your organisation, for personal purposes, write your name.
* *EMAIL* --> Put in your email.
* *ORGANISATIONAL_UNIT* --> This is for your orgaisational unit, for personal purposes write your name again.

You should end up with something like this:
```bash
. . .

export KEY_COUNTRY="GB"
export KEY_PROVINCE="LON"
export KEY_CITY="London"
export KEY_ORG="Ubuntu"
export KEY_EMAIL="webteam/run serve tutorials@canonical.com"
export KEY_OU="Tutorials"
export KEY_NAME="vpn"

. . .
```


Now save and close the file using `ctrl`+`x` and pressing enter twice

Now we are ready to build the certificate authority

##Building the Certificate Authority
Duration: 4:00

Using easy-rsa, we can build the certificate authority, begin by ensuring you are in the correct directory:
```bash
cd ~/ovpn-ca
```
Now we can source the vars file we made in the previous section:
```bash
source vars
```
Now we can make sure we are beginning with a clean install by using `./clean-all` and finally build the CA using `./build-ca`
We are asked a series of questions, we should answer in the same way as we did in the vars file.
Once this is done, we can proceed to creating certificates.

##Creating Certificates
Duration: 10:00

###Server End Certificates
We will now genereate the server-end certificate and key. This allows for the encrypted connection essential for a vpn. we can create the certificate and key by typing:
```bash
./build-key-server vpn
```
When prompted you can enter the same information as before, when asked for a challenge password you can leave it blank and after that say `'y'` and `enter`
We have now created a key, It is now time to create a couple more files begin with the following:
```bash
./build-dh
```
This will take a while
Now the last step:
```bash
openvpn --genkey --secret keys/ta.key
```
We are now ready to create client certificates.

###Client End Certificates
We will now generate certificates for clients, each client will need an individual certificate, the following steps can be repeated for each client.
Let's begin by creating a client named client1:
```bash 
cd ~/ovpn-ca
source vars
./build-key client1
```
Now we can follow the same process as the server key as for configuration.
We now have the framework for client encryption, this means we can move on to configuring OpenVPN.

##Configuring OpenVPN
Duration: 8:00

First, we must copy files to the OpenVPN directory:
```bash
cd ~/ovpn-ca/keys
sudo cp ca.crt vpn.crt vpn.key ta.key dh2048.pem /etc/openvpn
```

Now we can unzip and move a temporary configuration file to the openvpn directory and edit the file using:
```bash
unzip /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz | sudo tee /etc/openvpn/server.conf
sudo nano /etc/openvpn/server.conf
```
We now need to edit a couple of lines.
Find the following in your config file, and add the line `key-direction 0`:
```bash
# For extra security beyond that provided
# by SSL/TLS, create an "HMAC firewall"
# to help block DoS attacks and UDP port flooding.
#
# Generate with:
#   openvpn --genkey --secret ta.key
#
# The server and each client must have
# a copy of this key.
# The second parameter should be '0'
# on the server and '1' on the clients.
tls-auth ta.key 0 # This file is secret

key-direction 0

```
Right below, remove the `;` to uncomment the line `cipher AES-256-CBC`, below this add the line `auth SHA256`
This section should now look like the following
```bash
# Select a cryptographic cipher.
# This config item must be copied to
# the client config file as well.
# Note that 2.4 client/server will automatically
# negotiate AES-256-GCM in TLS mode.
# See also the ncp-cipher option in the manpage
cipher AES-256-CBC
auth SHA256
```
The last section we need to change is a few blocks down. Remove the semicolons before `user nobody` and `group nogroup` to uncomment these lines.
```bash
# It's a good idea to reduce the OpenVPN
# daemon's privileges after initialization.
#
# You can uncomment this out on
# non-Windows systems.
user nobody
group nogroup
```


##Adjust the Networking Configuration
Duration:10:00

###Forwarding Internet Connections
We need to allow our computer to forward internet connections, we can do so by editing the system control file:
```bash
sudo nano /etc/sysctl.conf
``` 
About a third of the way down, we find the lines:
```bash
#Uncomment the next line to enable packet forwarding for IPv4
#net.ipv4.ip_forward=1

# Uncomment the next line to enable packet forwarding for IPv6
#  Enabling this option disables Stateless Address Autoconfiguration
#  based on Router Advertisements for this host
#net.ipv6.conf.all.forwarding=1
```
Remove the `#`s before the `net.ipv4.ip_forward=1` and the `net.ipv6.conf.all.forwarding=1`, this will uncomment these lines and allow packet forwarding.
Now that this is edited, we need to reset the values for out current session, this is as simple as:
```bash
sudo sysctl -p
```

###Configuring the Firewall
We will begin by checking our current configuration and editing the files associated, this is as simple as typing the following:
```bash
ip route | grep default
sudo nano /etc/ufw/before.rules
```
We need to add a little code for OpenVPN, add the following below the first section
```bash
# START OPENVPN RULES
# NAT table rules
*nat
:POSTROUTING ACCEPT [0:0] 
# Allow traffic from OpenVPN client to wlp11s0 (change to the interface you discovered!)
-A POSTROUTING -s 10.8.0.0/8 -o wlp11s0 -j MASQUERADE
COMMIT
# END OPENVPN RULES
```
Now we need to tell ufw to forward packets, first open the file `/etc/default/ufw`
```bash
sudo nano /etc/default/ufw
```
Now find the line "`DEFAULT_FORWARD_POLICY="DROP"`" and change the DROP to ACCEPT. The line should now look like this:
```bash
# Set the default forward policy to ACCEPT, DROP or REJECT.  Please note that
# if you change this you will most likely want to adjust your rules
DEFAULT_FORWARD_POLICY="ACCEPT"
```

###Opening the OpenVPN Port

In order to allow the requests through the firewall, we need to modigy it. This can be done using:
```bash
sudo ufw allow 1194/udp
sudo ufw allow OpenSSH
Sudo ufw disable
sudo ufw enable
```
In the last two lines we restarted the firewall service.

##Starting the VPN Service
Duration 2:00

###Systemctl
We now need to start the service and tell the system to start it at boot, systemctl fulfills this function. Begin by ensuring everything has been set up properly:
```bash
sudo systemctl start openvpn@vpn
sudo systemctl status openvpn@vpn
sudo systemctl enable openvpn@vpn
```
Now we can move on to the client configuration infrastructure

##Client Configuration
Duration: 7:00

###Creating a Config Directory
We should begin by creating a client configuration directory.
```bash
sudo mkdir ~/client-configs/files
sudo chmod 700 ~/client-configs/files
```
We changed permissions on the last line because all client keys are stored in these config files, this information should not be accessible to regular users.

###Creating a Base Config
Like we did for the OpenVPN Config, we have a base config for the client ready, it just needs to be copied to our directory  and edited as follows
```bash
sudo cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf ~/client-configs/base.conf
sudo nano ~/client-configs/base.conf
```
Modify the following:
```bash
# The hostname/IP and port of the server.
# You can have multiple remote entries
# to load balance between the servers.
remote SERVER_IP 1194
```
* *SERVER_IP* --> your server's static ip address
```bash
# Downgrade privileges after initialization (non-Windows only)
user nobody
group nogroup
```
* Remove semicolons before `user nobody` and `group nogroup` to uncomment them
```bash
# SSL/TLS parms.
# See the server config file for more
# description.  It's best to use
# a separate .crt/.key file pair
# for each client.  A single ca
# file can be used for all clients.
#ca ca.crt
#cert client.crt
#key client.key
```
* Add a `#` to the start of each of the last three lines to comment them out
* We will be adding our own certificates to the file instead.
```bash
key-direction 1
```
* This line needs to be added at the end of the configuration file.

###Creating a Basic Terminal Script to Create Clients
We can now set up a basic shell script to generate clients. First we can create a shell file.
```bash
sudo nano ~/client-configs/make-config.sh
```
Now paste the following
```
#!/bin/bash
# First argument: Client identifier
KEY_DIR=~/ovpn-ca/keys
OUTPUT_DIR=~/client-configs/files
BASE_CONFIG=~/client-configs/base.conf

cat ${BASE_CONFIG} \
    <(echo -e '<ca>') \
    ${KEY_DIR}/ca.crt \
    <(echo -e '</ca>\n<cert>') \
    ${KEY_DIR}/${1}.crt \
    <(echo -e '</cert>\n<key>') \
    ${KEY_DIR}/${1}.key \
    <(echo -e '</key>\n<tls-auth>') \
    ${KEY_DIR}/ta.key \
    <(echo -e '</tls-auth>') \
    > ${OUTPUT_DIR}/${1}.ovpn
```
Ensure we can run it as an executable by changing its permissions
```bash
sudo chmod 700 ~/client-configs/make-config.sh
```
We have finished all the configuration on the server end, it is now time to relax and allow the client to do its work now.

##Creating a Client Configuration
Duration: 5:00

We now need to navigate into the client directory we just made and run the script:
```bash
cd ~/client-configs
sudo ./make-config.sh client1
```
Congratulations! You have completed the server-end tutorial on how to install a vpn. Be sure to read the client-end tutorial on how to use the .ovpn file generated in `~/client-configs/files` to connect to the server.

##You're Done
Duration: 1:00

You have finally finished the tutorial on how to create a vpn server.
OpenVPN is a widely used vpn program, your connection can be trusted.
Installing the .ovpn file you generated in the last step is explained in the client vpn installation tutorial.

###You now know how to:
* Prepare an environment to install a VPN Server using OpenVPN
* Install OpenVPN and easy-rsa
* Create encryption certificates using easy-rsa
* Configure OpenVPN
* Start the VPN service
* Prepare an openvpn client configuration

###What's Next?
* Modify your external internet connection to forward port 1194
* Ensure your network has a static IP address
* Get a URL to easily send traffic to your server (optional)
* Set up a client using the Install a VPN Client Tutorial on [Ubuntu Tutorials](https://tutorials.ubuntu.com)

###I Need Help
* Double Check that the port is available
* Ensure the configuration files are correct
* Make sure you typed the commands properly
* Try using sudo (if you arent already)
* Make sure the information you entered is the same throughout the installation
* Try using [this script](https://github.com/Nyr/openvpn-install)
* Ask a question on [Ask Ubuntu](https://askubuntu.com/questions/ask)
