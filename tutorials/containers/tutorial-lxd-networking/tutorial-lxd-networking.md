---
id: tutorial-lxd-networking
summary: Learn about the networking capabilities of LXD containers and how to use them to your benefit
categories: containers
tags: tutorial, LXD, containers, network, networking, bridged, gci
difficulty: 3
status: published
feedback_url: https://github.com/canonical-websites/tutorials.ubuntu.com/issues
published: 2018-01-16
author: Varun Patel <varun-patel@live.com>

---

# Networking in LXD containers

## Overview
Duration: 3:00

In this tutorial, we will cover the various networking capabilities of LXD.

### What You'll Learn

* How to set up "bridged" adapters
* How to set up ipv4 and ipv6
* How to set up DNS
* How to set up tunnels

### What You'll Need

* A computer running Ubuntu 16.04 Xenial Xerus or newer
* A working LXD installation ([tutorial](https://tutorials.ubuntu.com/tutorial/tutorial-setting-up-lxd-1604))
* A running container
* Some form of networking infrastructure

Survey
: How will you use this tutorial?
- Only read through it
- Read it and complete the exercises
: What is your current level of experience?
- Novice
- Intermediate
- Proficient

## Getting Started
Duration: 3:00

Let's begin by creating a test network called `testbr0`
```bash
lxc network create testbr0
```

We can now open the .yaml file to edit in terminal.
```bash
lxc network edit testbr0
```

In this file, we can add the configuration options we choose from the following four pages.
This is done after the `config:` line, you should already see some indented lines.

The following four pages contain lists of 'keys' that can help you get exactly what you want out of your networking configuraiton

## Bridging
Duration: 3:00

* bridge.driver: 'native' or 'openvswitch', this key is used to determine the driver used to manage network connections, the default is 'native'
* bridge.external_interfaces: this key is used to include unconfigured interfaces in the bridge
* bridge.mtu: this key is used to to set the bridge MTU, please note that the default varies depending on tunneling and FAN setups.
* bridge.mode: 'standard' or 'fan', this key is used to set the bridge to standard or fan to assign ip addresses, the default is 'standard'
* fan.overlay_subnet: 'x.x.x.x/x', this key is used to set the overlay subnet for FAN, the default is '240.0.0.0/8'
* fan.type: 'vxlan' or 'ipip', this key is used to set the tunneling mode for FAN, the default is 'vxlan'
* fan.underlay_subnet: 'x.x.x.x/x', this key is used to set the underlay subnet for FAN, the default is the gateway subnet

## IP Configruation
Duration: 6:00

### IPv4

* ipv4.address: (x.x.x.x/x), this key is used to set the ipv4 address of the bridge, the default is a random unused subnet
* ipv4.dhcp: 'true' or 'false', this key is used to set a boolean value which determines whether the bridge allocates addresses using DHCP or not, the default value is 'true'
* ipv4.dhcp.expiry: 'xh' (i.e. 1h, 2h, 24h), this key is used to set the exipry of DHCP leases, the default is 1 hour
* ipv4.dhcp.ranges: (x.x.x.x-x.x.x.x), this key is used to set a list of ip ranges to use for DHCP, the default is all addresses
* ipv4.firewall: 'true' or 'false', this key is used to set a boolean value which determines whether or not a firewall is used , the default is 'true'
* ipv4.nat: 'true' or 'false', this key is used to determine whether or not to use NAT, the default is true if a random ipv4 address is used otherwise this value is false
* ipv4.routes: (x.x.x.x/x, x.x.x.x/x...), this key is used to list ipv4 subnets to route into the bridge
* ipv4.routing: 'true' or 'false', this key is used to choose whether or not to route traffic in and out of the bridge, the default is 'true'

### IPv6

* ipv6.address: (x:x:x:x/x), this key is used to set the ipv6 address of the bridge, the default is a random unused subnet
* ipv6.dhcp: 'true' or 'false', this key is used to set a boolean value which determines whether or not to provide configuration over DHCP, the default value is 'true'
* ipv6.dhcp.expiry: 'xh' (i.e. 1h, 2h, 24h), this key is used to set the exipry of DHCP leases, the default is 1 hour
* ipv6.dhcp.ranges: (x:x:x:x-x:x:x:x), this key is used to set a list of ip ranges to use for DHCP, the default is all addresses
* ipv6.dhcp.stateful: 'true' or 'false', this key is used to set a boolean value which determines whether the bridge allocates addresses using DHCP or not, the default value is 'false'
* ipv6.firewall: 'true' or 'false', this key is used to set a boolean value which determines whether or not a firewall is used , the default is 'true'
* ipv6.nat: 'true' or 'false', this key is used to determine whether or not to use NAT, the default is true if a random ipv6 address is used otherwise this value is false
* ipv6.routes: (x:x:x:x/x, x:x:x:x/x...), this key is used to list ipv6 subnets to route into the bridge
* ipv6.routing: 'true' or 'false', this key is used to choose whether or not to route traffic in and out of the bridge, the default is 'true'

## DNS Setup
Druation: 1:00

* dns.domain: 'lxd' or 'www.example.com', this key is used to set the domain advertised to DHCP clients to use for DNS resolution, the default is 'lxd'
* dns.mode: 'none', 'managed' or 'dynamic', this key is used to set the DNS registration mode, the default is 'managed'
* raw.dnsmasq: 'strings', this key is used to add any dnsmasq configuration.

## Tunnels
Duration: 2:00

* tunnel.NAME.group: (x.x.x.x), this key is used to set the multicast address used by vlxan only if local and remote are not set, the default is '239.0.0.1'
* tunnel.NAME.id: 'x' (i.e. 0, 1, 2, 3, 4 ...), this key is used to set the tunnel ID used by a vxlan tunnel, this is an integer value, the default is '0'
* tunnel.NAME.interface: (string), this key is used to set the string value of a vxlan host interface
* tunnel.NAME.local: (x.x.x.x), this key is used to set the local addtess for the tunnel, this value is not necessary for a musticast vxlan setup
* tunnel.NAME.port: 'x' (i.e. 0, 1, 2, 3, 4 ...), this key is used to set the port for the vxlan tunnel, the default is '0'
* tunnel.NAME.protocol: 'vxlan' or 'gre', this key is used to set the tunnelling protocol to use
* tunnel.NAME.remote (x.x.x.x), this key is used to set the remote address for the tunnel, this value is not necessary for a multicast vxlan setup

## Connecting the bridge to a container
Duration: 1:00

Connecting a network to containers is easy, if you want to connect all containers to your newly created networking configuration, use the following command:
```bash
lxc network attach-profile testbr0 default <network_adapter_name>
```

if this is only for one container, use the following:
lxc network attach <container_name> default <network_adapter_name>

## You're Done!
Duration: 3:00

We are now running a custom networking setup for our LXD containers

###You now know how to:

* Create a netowrk on LXD
* Attach the network to a container
* Set up Security, DNS and Tunnels

###What's Next?

* Play around with the configuration options
* check out the [LXD docs](https://github.com/lxc/lxd/tree/master/doc) for more configuration options

###I Need Help

* Make sure you typed the commands properly
* Try using sudo (if you aren't already) i.e. `sudo` + `command`
* Ask a question on [Ask Ubuntu](https://askubuntu.com/questions/ask)
