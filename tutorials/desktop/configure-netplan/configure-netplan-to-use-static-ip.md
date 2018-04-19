---
id: tutorial-configure-netplan-to-use-static-ip
summary: Re-configuring an Ubuntu 17.10 default Netplan DHCP configuration to use a static IP address.
categories: desktop
tags: tutorial,netplan,network,ubuntu,desktop
difficulty: 1
status: published
published: 2018-01-06
author: Daniel Lim Wee Soong <weesoong.lim@gmail.com>
feedback_url: https://github.com/canonical-websites/tutorials.ubuntu.com/issues

---

# Configure Netplan to use a static ip

## Overview
Duration: 1:00

Netplan is a YAML network configuration abstraction for various backends (NetworkManager, networkd).

It is a utility for easily configuring networking on a system. It can be used by writing a YAML description of the required network interfaces with what they should be configured to do. From this description it will generate the required configuration for a chosen renderer tool.

Netplan reads network configuration from `/etc/netplan/*.yaml` which are written by administrators, installers, cloud image instantiations, or other OS deployments. During early boot it then generates backend specific configuration files in `/run` to hand off control of devices to a particular networking daemon.

### Rationale of migrating to Netplan
Seasoned Ubuntu users may wonder why the migration from ifupdown to Netplan.

Netplan has been implemented to support simple, declarative representation of complex network configurations, as well as address some current limitations of ifupdown. At the same time, netplan provides a simple and elegant yaml configuration format with support for multiple backend providers.

Some of the shortcomings of ifupdown covered by netplan:
- ifupdown cannot represent all configs with a purely declarative syntax; therefore we can not parse the config
	* all netplan config is purely declarative
- ifupdown can only represent interfaces by name, so it is not portable across devices
	* netplan uses matching by name, MAC address, driver, etc.
- race conditions in complex configs
	* netplan has the context of hierarchy in the definition of the interfaces, such that this information is carried over to the renderer used and applied in the right ordering.

Given increasing demand for complex networking scenarios (large cloud uses often require complex layering of different features, such as bridges over bonds over VLANs, etc.), it has shown to be important to improve the ease of representing the network config.

### What you'll learn
- How to modify the netplan configuration file to use a static IP address.

### What you'll need

- Ubuntu 17.10
- Some very basic knowledge of command line use, know how to edit files.
- Basic networking knowledge about static IP addresses.

## Basic configuration
Duration: 3:00

To configure a static IP address for our machine, simply create a file in `/etc/netplan`, for example `/etc/netplan/config-0.yaml`

Then, we can add our configuration settings to the file. Here's a basic example,

```YAML
network:
    version: 2
    renderer: networkd
    ethernets:
        eth0:
            addresses:
                - 10.10.10.2/24
            dhcp4: no
```

### Network interface

This configuration file allows us to choose the network interface that we want to apply our settings to.

We can replace `eth0` with the network interface we want to enable DHCP on, and attempt to get a new IP address for it.

### IP Address

Similarly, under addresses, we can put a list of static IP addresses that we would want DHCP to attempt getting for us, ended with the netmask.

We can choose a subnet mask from the following list and add the prefix size to the end of the address.

```
Prefix size         | Subnet mask   
/24                 | 255.255.255.0 
/25                 | 255.255.255.128
/26                 | 255.255.255.192
/27                 | 255.255.255.224
/28                 | 255.255.255.240
/29                 | 255.255.255.248
/30                 | 255.255.255.252
```

Now, we are left with applying the changes.

## Apply changes
Duration: 2:00

It is very simple to apply the added settings, just run `sudo netplan apply`.

This will enable DHCP on the chosen interface, and immediately attempt to get a new IP address for it from the provided list of addresses.

### Use config file on next boot
We don't need to do anything for this. Netplan will apply settings from all configuration files under `/etc/netplan` every boot.

### Don't use config file on next boot
Simply remove the configuration for it. If the interface is not configured in a .yaml file in `/etc/netplan`, it will not be configured at boot.

## That's it!
Duration: 1:00

If you have any other queries regarding netplan, you can look for more documentation in this site or easily access the netplan documentation via:

```
man 5 netplan
```

Or, if you need some guidance with other parts of Ubuntu, take a look at the [community documentation][commdocs], and if you get stuck, help is always at hand:

* [Ask Ubuntu][askubuntu]
* [Ubuntu Forums][forums]
* [IRC-based support][ubuntuirc]

<!-- LINKS -->
[commdocs]: https://help.ubuntu.com/community/UsingTheTerminal
[askubuntu]: https://askubuntu.com/
[forums]: https://ubuntuforums.org/
[ubuntuirc]: https://wiki.ubuntu.com/IRC/ChannelList
