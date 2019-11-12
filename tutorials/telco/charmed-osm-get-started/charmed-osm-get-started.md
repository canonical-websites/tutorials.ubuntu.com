---
id: charmed-osm-get-started
summary: Getting started with Charmed OSM
categories: NFV
tags: OSM, MANO, NFV, MicroK8s, MicroStack, beginner
difficulty: 1
status: draft
published: 2019-11-09
author: Tytus Kurek <tytus.kurek@canonical.com>
feedback_url: https://bugs.launchpad.net/canonical-osm

---

# Getting started with Charmed OSM



## Introduction

Duration: 5:00

This tutorial provides an introduction to Charmed OSM. It uses Canonical's Development Stack for [OSM][osm_website] (Open Source MANO (Management and Orchestration)) which is a toolset that you can use to onboard and orchestrate real VNF (Virtual Network Function) workloads directly on your workstation. It consists of the following components:
* **MicroK8s** - for deploying local Kubernetes cluster that will host your OSM installation,
* **MicroStack** - for deploying local OpenStack cluster and using it as a VIM (Virtual Infrastructure Manager) for OSM,
* **Charmed OSM** - simple OSM installation fully aligned with the upstream project.
Before we get started let's try to better understand these components.

### What is MicroK8s?

[MicroK8s][microk8s_website] is a pure upstream Kubernetes installation that can run directly on your workstation. It is a Kubernetes in a [snap][microk8s_snap], which means that all Kubernetes services and supporting libraries are packaged together in a single image that can be installed on 42 Linux distribution, Windows and Mac OS. MicroK8s is fully compliant with the upstream Kubernetes, thus you can use it to host your OSM installation.

### What is MicroStack?

[MicroStack][microstack_website] is a pure upstream OpenStack installation that can also run directly on your workstation. Like MicroK8s, it is an OpenStack in a [snap][microstack_snap]. MicroStack includes all key OpenStack components: Keystone, Nova, Neutron, Glance, and is evolving extremely fast. You can use MicroStack as a VIM for OSM.

### What is Charmed OSM?

[Charmed OSM][charmed_osm_website] is a pure upstream OSM distribution that is based on [Juju charms][charms_website]. By using charms for OSM deployments telcos can benefit from a model-driven declarative approach and simplified operations. Although we use Charmed OSM for testing/development purposes in this tutorial, it is production-grade and can be successfully used in live environments.

### In this tutorial you will learn how to:

* Deploy Charmed OSM on top of MicroK8s,
* Deploy MicroStack and add it as a VIM for Charmed OSM,
* Onboard sample VNF workloads through Charmed OSM.

### You will need:

A workstation which meets the following minimum recommended system requirements:
* **Ubuntu 18.04 LTS** Operating System,
* **16 GB** of RAM,
* **4 CPUs**,
* **50 GB** of free storage space.

## Before you start

Duration: 5:00

If you are using Windows or Mac OS, install [multipass][multipass] first to launch an Ubuntu VM.

If you are using Ubuntu 18.04 LTS, which we highly recommend, you can move on. Otherwise, you may need to install [snapd][snapd] first.

Install Juju and OSM client from the snap store:

```bash
$ sudo snap install juju --classic
juju 2.6.10 from Canonical✓ installed
$ sudo snap install osmclient --edge
osmclient (edge) 0+git.44be24a from Adam Israel (adam) installed
```

Allows OSM client snap to access Juju configuration:

```bash
$ sudo snap connect osmclient:juju-client-observe
```

Bootstrap Juju controller on LXD that OSM will use to deploy proxy charms:

```bash
$ juju bootstrap localhost osm-lxd
Since Juju 2 is being run for the first time, downloading latest cloud information.
Fetching latest public cloud list...
Updated your list of public clouds with 2 cloud regions added:

    added cloud region:
        - google/asia-northeast2
        - google/europe-west6
Creating Juju controller "osm-lxd" on localhost/localhost
Looking for packaged Juju agent version 2.6.10 for amd64
To configure your system to better support LXD containers, please see: https://github.com/lxc/lxd/blob/master/doc/production-setup.md
Launching controller instance(s) on localhost/localhost...
 - juju-ba6acd-0 (arch=amd64)                 
Installing Juju agent on bootstrap instance
Fetching Juju GUI 2.15.0
Waiting for address
Attempting to connect to 10.25.201.34:22
Connected to 10.25.201.34
Running machine configuration script...
Bootstrap agent now started
Contacting Juju controller at 10.25.201.34 to verify accessibility...

Bootstrap complete, controller "osm-lxd" now is available
Controller machines are in the "controller" model
Initial model "default" added
```

positive
: This command takes a while to finish.

## Setting up MicroK8s

Duration: 3:00

To install MicroK8s execute:

```bash
$ sudo snap install microk8s --classic
microk8s v1.16.2 from Canonical✓ installed
```

Once completed, execute the following commands to set the required permissions and enable storage, and DNS plugins in MicroK8s:

```bash
$ sudo usermod -a -G microk8s $USER
$ newgrp microk8s
$ microk8s.status --wait-ready
microk8s is running
addons:
cilium: disabled
dashboard: disabled
dns: disabled
fluentd: disabled
gpu: disabled
helm: disabled
ingress: disabled
istio: disabled
jaeger: disabled
knative: disabled
linkerd: disabled
metrics-server: disabled
prometheus: disabled
rbac: disabled
registry: disabled
storage: disabled
$ microk8s.enable storage dns
Enabling default storage class
deployment.apps/hostpath-provisioner created
storageclass.storage.k8s.io/microk8s-hostpath created
serviceaccount/microk8s-hostpath created
clusterrole.rbac.authorization.k8s.io/microk8s-hostpath created
clusterrolebinding.rbac.authorization.k8s.io/microk8s-hostpath created
Storage will be available soon
Enabling DNS
Applying manifest
serviceaccount/coredns created
configmap/coredns created
deployment.apps/coredns created
service/kube-dns created
clusterrole.rbac.authorization.k8s.io/coredns created
clusterrolebinding.rbac.authorization.k8s.io/coredns created
Restarting kubelet
DNS is enabled
```

That’s it! Your local Kubernetes cluster is up and running. You can interact with it by using the `microk8s.kubectl` command, for example:

```bash
$ microk8s.kubectl cluster-info
Kubernetes master is running at https://127.0.0.1:16443
CoreDNS is running at https://127.0.0.1:16443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

## Setting up MicroStack

Duration: 10:00

To install MicroK8s execute:

```bash
$ sudo snap install microstack --classic --beta
microstack (beta) stein from Canonical✓ installed
```

Once completed, execute the following commands:

```bash
$ sudo microstack.init --auto
2019-11-10 20:54:14,841 - microstack_init - INFO - Setting up ipv4 forwarding...
2019-11-10 20:54:14,940 - microstack_init - INFO - Waiting for RabbitMQ to start ...
Waiting for 10.20.20.1:5672
2019-11-10 20:54:14,949 - microstack_init - INFO - RabbitMQ started!
2019-11-10 20:54:14,949 - microstack_init - INFO - Configuring RabbitMQ ...
2019-11-10 20:54:16,766 - microstack_init - INFO - RabbitMQ Configured!
2019-11-10 20:54:16,798 - microstack_init - INFO - Waiting for MySQL server to start ...
Waiting for 10.20.20.1:3306
2019-11-10 20:54:16,803 - microstack_init - INFO - Mysql server started! Creating databases ...
/snap/microstack/171/lib/python3.6/site-packages/pymysql/cursors.py:170: Warning: (1287, 'Using GRANT for creating new user is deprecated and will be removed in future release. Create new user with CREATE USER statement.')
  result = self._query(query)
2019-11-10 20:54:17,791 - microstack_init - INFO - Configuring Keystone Fernet Keys ...
2019-11-10 20:54:34,892 - microstack_init - INFO - Bootstrapping Keystone ...
2019-11-10 20:54:40,374 - microstack_init - INFO - Creating service project ...
2019-11-10 20:54:45,931 - microstack_init - INFO - Keystone configured!
2019-11-10 20:54:45,967 - microstack_init - INFO - Configuring nova ...
/snap/microstack/171/lib/python3.6/site-packages/pymysql/cursors.py:170: Warning: (1287, "Using GRANT statement to modify existing user's properties other than privileges is deprecated and will be removed in future release. Use ALTER USER statement for this operation.")
  result = self._query(query)
Waiting for 10.20.20.1:8774
2019-11-10 20:56:26,974 - microstack_init - INFO - Creating default flavors...
2019-11-10 20:56:52,934 - microstack_init - INFO - Configuring Neutron
Waiting for 10.20.20.1:9696
2019-11-10 20:58:15,223 - microstack_init - INFO - Configuring Glance ...
Waiting for 10.20.20.1:9292
2019-11-10 20:58:50,777 - microstack_init - INFO - Fetching cirros image ...
100% [........................................................................] 12716032 / 127160322019-11-10 20:58:57,486 - microstack_init - INFO - Creating microstack keypair (~/.ssh/id_microstack)
2019-11-10 20:59:00,262 - microstack_init - INFO - Creating security group rules ...
2019-11-10 20:59:10,779 - microstack_init - INFO - restarting libvirt and virtlogd ...
2019-11-10 20:59:10,928 - microstack_init - INFO - Complete. Marked microstack as initialized!
```

That’s it! Your local OpenStack cluster is up and running. You can interact with it by using the `microstack.openstack` command.

In order to configure MicroStack, so that it could be used as a VIM for Charmed OSM, add Ubuntu image:

```bash
$ wget https://cloud-images.ubuntu.com/releases/16.04/release/ubuntu-16.04-server-cloudimg-amd64-disk1.img
--2019-11-12 17:25:01--  https://cloud-images.ubuntu.com/releases/16.04/release/ubuntu-16.04-server-cloudimg-amd64-disk1.img
Resolving cloud-images.ubuntu.com (cloud-images.ubuntu.com)... 91.189.88.89, 2001:67c:1560:8001::8001
Connecting to cloud-images.ubuntu.com (cloud-images.ubuntu.com)|91.189.88.89|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://cloud-images.ubuntu.com/releases/xenial/release/ubuntu-16.04-server-cloudimg-amd64-disk1.img [following]
--2019-11-12 17:25:02--  https://cloud-images.ubuntu.com/releases/xenial/release/ubuntu-16.04-server-cloudimg-amd64-disk1.img
Reusing existing connection to cloud-images.ubuntu.com:443.
HTTP request sent, awaiting response... 200 OK
Length: 296878080 (283M) [application/octet-stream]
Saving to: ‘ubuntu-16.04-server-cloudimg-amd64-disk1.img.1’

ubuntu-16.04-server-cloudimg-amd64-disk1.img.1     100%[================================================================================================================>] 283.12M  24.2MB/s    in 10s     

2019-11-12 17:25:12 (27.7 MB/s) - ‘ubuntu-16.04-server-cloudimg-amd64-disk1.img.1’ saved [296878080/296878080]

$ microstack.openstack image create \
                     --public \
                     --disk-format qcow2 \
                     --container-format bare \
                     --file ubuntu-16.04-server-cloudimg-amd64-disk1.img \
                     ubuntu1604
+------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field            | Value                                                                                                                                                                                      |
+------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| checksum         | b6515bf62f9c3132db1afd9e0f8ad3ea                                                                                                                                                           |
| container_format | bare                                                                                                                                                                                       |
| created_at       | 2019-11-12T17:26:27Z                                                                                                                                                                       |
| disk_format      | qcow2                                                                                                                                                                                      |
| file             | /v2/images/360204d3-adcf-49ec-abb3-51816836faab/file                                                                                                                                       |
| id               | 360204d3-adcf-49ec-abb3-51816836faab                                                                                                                                                       |
| min_disk         | 0                                                                                                                                                                                          |
| min_ram          | 0                                                                                                                                                                                          |
| name             | ubuntu1604                                                                                                                                                                                 |
| owner            | 249ddaebc6924a63b44b24d28ae1aa9d                                                                                                                                                           |
| properties       | os_hash_algo='sha512', os_hash_value='7ea215276496dca550a318d07f582ed8eda5eee2e674884dce09504aaacf3fd2976b030c4618d575e413865e5cbd278c2e297a8f37b883b7b6946a9fd010b1e5', os_hidden='False' |
| protected        | False                                                                                                                                                                                      |
| schema           | /v2/schemas/image                                                                                                                                                                          |
| size             | 296878080                                                                                                                                                                                  |
| status           | active                                                                                                                                                                                     |
| tags             |                                                                                                                                                                                            |
| updated_at       | 2019-11-12T17:26:30Z                                                                                                                                                                       |
| virtual_size     | None                                                                                                                                                                                       |
| visibility       | public                                                                                                                                                                                     |
+------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```

## Setting up Charmed OSM

Duration: 18:00

In this step we'll first deploy Charmed OSM on MicroK8s and then use MicroStack as a VIM.

### Deploy Charmed OSM

Bootstrap another Juju controller on MicroK8s that will host your Charmed OSM installation:

```bash
$ juju bootstrap microk8s osm-k8s
Creating Juju controller "osm-k8s" on microk8s/localhost
Creating k8s resources for controller "controller-osm-k8s"
Downloading images
Starting controller pod
Bootstrap agent now started
Contacting Juju controller at 10.152.183.19 to verify accessibility...

Bootstrap complete, controller "osm-k8s" now is available in namespace "controller-osm-k8s"

Now you can run
	juju add-model <model-name>
to create a new model to deploy k8s workloads.
```

positive
: This command takes a while to finish.

Once finished, add a model for OSM deployments:

```bash
$ juju add-model osm
Added 'osm' model on microk8s/localhost with credential 'microk8s' for user 'admin'
```

Generate an overlay bundle:

```bash
osmclient.overlay
```

Deploy Charmed OSM:

```bash
$ juju deploy osm --overlay vca-overlay.yaml
Located bundle "cs:bundle/osm-21"
Resolving charm: cs:~charmed-osm/grafana-k8s-26
Resolving charm: cs:~charmed-osm/kafka-k8s-11
Resolving charm: cs:~charmed-osm/lcm-k8s-27
Resolving charm: cs:~charmed-osm/mariadb-k8s-23
Resolving charm: cs:~charmed-osm/mon-k8s-24
Resolving charm: cs:~charmed-osm/mongodb-k8s-21
Resolving charm: cs:~charmed-osm/nbi-k8s-30
Resolving charm: cs:~charmed-osm/pol-k8s-22
Resolving charm: cs:~charmed-osm/prometheus-k8s-27
Resolving charm: cs:~charmed-osm/ro-k8s-27
Resolving charm: cs:~charmed-osm/ui-k8s-35
Resolving charm: cs:~charmed-osm/zookeeper-k8s-30
Executing changes:
- upload charm cs:~charmed-osm/grafana-k8s-26 for series kubernetes
- deploy application grafana-k8s with 1 unit on kubernetes using cs:~charmed-osm/grafana-k8s-26
  added resource grafana-image
- set annotations for grafana-k8s
- upload charm cs:~charmed-osm/kafka-k8s-11 for series kubernetes
- deploy application kafka-k8s with 1 unit on kubernetes using cs:~charmed-osm/kafka-k8s-11
  added resource kafka-image
- set annotations for kafka-k8s
- upload charm cs:~charmed-osm/lcm-k8s-27 for series kubernetes
- deploy application lcm-k8s with 1 unit on kubernetes using cs:~charmed-osm/lcm-k8s-27
  added resource lcm-image
- set annotations for lcm-k8s
- upload charm cs:~charmed-osm/mariadb-k8s-23 for series kubernetes
- deploy application mariadb-k8s with 1 unit on kubernetes using cs:~charmed-osm/mariadb-k8s-23
  added resource galera-image
  added resource mariadb-image
- set annotations for mariadb-k8s
- upload charm cs:~charmed-osm/mon-k8s-24 for series kubernetes
- deploy application mon-k8s with 1 unit on kubernetes using cs:~charmed-osm/mon-k8s-24
  added resource mon-image
- set annotations for mon-k8s
- upload charm cs:~charmed-osm/mongodb-k8s-21 for series kubernetes
- deploy application mongodb-k8s with 1 unit on kubernetes using cs:~charmed-osm/mongodb-k8s-21
  added resource mongodb-image
  added resource mongodb-sidecar-image
- set annotations for mongodb-k8s
- upload charm cs:~charmed-osm/nbi-k8s-30 for series kubernetes
- deploy application nbi-k8s with 1 unit on kubernetes using cs:~charmed-osm/nbi-k8s-30
  added resource nbi-image
- set annotations for nbi-k8s
- upload charm cs:~charmed-osm/pol-k8s-22 for series kubernetes
- deploy application pol-k8s with 1 unit on kubernetes using cs:~charmed-osm/pol-k8s-22
  added resource pol-image
- set annotations for pol-k8s
- upload charm cs:~charmed-osm/prometheus-k8s-27 for series kubernetes
- deploy application prometheus-k8s with 1 unit on kubernetes using cs:~charmed-osm/prometheus-k8s-27
  added resource alpine-image
  added resource prometheus-image
- set annotations for prometheus-k8s
- upload charm cs:~charmed-osm/ro-k8s-27 for series kubernetes
- deploy application ro-k8s with 1 unit on kubernetes using cs:~charmed-osm/ro-k8s-27
  added resource ro-image
- set annotations for ro-k8s
- upload charm cs:~charmed-osm/ui-k8s-35 for series kubernetes
- deploy application ui-k8s with 1 unit on kubernetes using cs:~charmed-osm/ui-k8s-35
  added resource ui-image
- set annotations for ui-k8s
- upload charm cs:~charmed-osm/zookeeper-k8s-30 for series kubernetes
- deploy application zookeeper-k8s with 1 unit on kubernetes using cs:~charmed-osm/zookeeper-k8s-30
  added resource zookeeper-image
- set annotations for zookeeper-k8s
- add relation kafka-k8s:zookeeper - zookeeper-k8s:zookeeper
- add relation ro-k8s:mysql - mariadb-k8s:mysql
- add relation nbi-k8s:mongo - mongodb-k8s:mongo
- add relation nbi-k8s:kafka - kafka-k8s:kafka
- add relation lcm-k8s:kafka - kafka-k8s:kafka
- add relation lcm-k8s:mongo - mongodb-k8s:mongo
- add relation mon-k8s:kafka - kafka-k8s:kafka
- add relation mon-k8s:mongo - mongodb-k8s:mongo
- add relation pol-k8s:kafka - kafka-k8s:kafka
- add relation pol-k8s:mongo - mongodb-k8s:mongo
- add relation lcm-k8s:ro - ro-k8s:ro
- add relation prometheus-k8s:prometheus - mon-k8s:prometheus
- add relation grafana-k8s:prometheus - prometheus-k8s:prometheus
- add relation ui-k8s:mysql - mariadb-k8s:mysql
- add relation ui-k8s:nbi - nbi-k8s:nbi
Deploy of bundle completed.
```

positive
: This command takes a while to finish. Moreover, once it’s finished, Juju is still deploying Charmed OSM on MicroK8s.

You can check the actual status by running the `juju status` command. Once all applications turn to the "active" state, your Charmed OSM installation is completed:

```bash
$ juju status
Model  Controller  Cloud/Region        Version  SLA          Timestamp
osm    osm-k8s     microk8s/localhost  2.6.10   unsupported  21:07:57Z

App             Version  Status   Scale  Charm           Store       Rev  OS          Address         Notes
grafana-k8s              active       1  grafana-k8s     jujucharms   26  kubernetes  10.152.183.75   
kafka-k8s                active       1  kafka-k8s       jujucharms   11  kubernetes  10.152.183.214  
lcm-k8s                  active       1  lcm-k8s         jujucharms   27  kubernetes  10.152.183.119  
mariadb-k8s              active       1  mariadb-k8s     jujucharms   23  kubernetes  10.152.183.222  
mon-k8s                  active       1  mon-k8s         jujucharms   24  kubernetes  10.152.183.183  
mongodb-k8s              active       1  mongodb-k8s     jujucharms   21  kubernetes  10.152.183.191  
nbi-k8s                  active       1  nbi-k8s         jujucharms   30  kubernetes  10.152.183.209  
pol-k8s                  active       1  pol-k8s         jujucharms   22  kubernetes  10.152.183.195  
prometheus-k8s           active       1  prometheus-k8s  jujucharms   27  kubernetes  10.152.183.4    
ro-k8s                   active       1  ro-k8s          jujucharms   27  kubernetes  10.152.183.237  
ui-k8s                   waiting      1  ui-k8s          jujucharms   35  kubernetes  10.152.183.236  
zookeeper-k8s            active       1  zookeeper-k8s   jujucharms   30  kubernetes  10.152.183.133  

Unit               Workload  Agent  Address     Ports                       Message
grafana-k8s/0*     active    idle   10.1.50.25  3000/TCP                    ready
kafka-k8s/0*       active    idle   10.1.50.28  9092/TCP                    ready
lcm-k8s/0*         active    idle   10.1.50.30  9999/TCP                    ready
mariadb-k8s/0*     active    idle   10.1.50.16  3306/TCP                    ready
mon-k8s/0*         active    idle   10.1.50.31  8000/TCP                    ready
mongodb-k8s/0*     active    idle   10.1.50.22  27017/TCP                   ready
nbi-k8s/0*         active    idle   10.1.50.32  9999/TCP                    ready
pol-k8s/0*         active    idle   10.1.50.29  80/TCP                      ready
prometheus-k8s/0*  active    idle   10.1.50.23  9090/TCP                    ready
ro-k8s/0*          active    idle   10.1.50.27  9090/TCP                    ready
ui-k8s/0*          active    idle   10.1.50.33  80/TCP                      ready
zookeeper-k8s/0*   active    idle   10.1.50.26  2181/TCP,2888/TCP,3888/TCP  ready
```

You can then export the `OSM_HOSTNAME` environmental variable:

```bash
$ export OSM_HOSTNAME=`juju status nbi-k8s | grep kubernetes | awk '{print $8}'`
$ echo "export OSM_HOSTNAME=$OSM_HOSTNAME" >> ~/.bashrc
```

At this point, you can start interacting with your Charmed OSM installation by using the `osmclient.osm` command, for example:

```bash
$ osmclient.osm ns-list
+------------------+----+--------------------+---------------+-----------------+
| ns instance name | id | operational status | config status | detailed status |
+------------------+----+--------------------+---------------+-----------------+
+------------------+----+--------------------+---------------+-----------------+
```

You can also access Charmed OSM web GUI at the IP address returned by the following command:

```bash
$ juju status ui-k8s | grep kubernetes | awk '{print $8}'
10.152.183.236
```

### Add MicroStack as a VIM

In order to add MicroStack as a VIM for Charmed OSM, execute the following command:
```bash
$ osmclient.osm vim-create --name microstack \
      --user admin \
      --password keystone \
      --auth_url http://10.20.20.1:5000/v3 \
      --tenant admin \
      --account_type openstack \
      --config='{security_groups: default,
                 keypair: microstack,
                 project_name: admin,
                 user_domain_name: default,
                 region_name: microstack,
                 insecure: True,
                 availability_zone: nova,
                 version: 3,
                 use_floating_ip: true}'
c59b6823-65e5-4c64-9a36-86cea30ba1db
```

That’s it! You can start onboarding your VNFs.

## Onboarding VNFs

Duration: 15:00

At this point you have MicroK8s installed, Charmed OSM deployed on top of MicroK8s and MicroStack configured as a VIM. We will now deploy a basic network service to exercise the environment.

### Network Service

Download sample NSD (Network Service Descriptor) and VNFD (Virtual Network Function Descriptor):

```bash
$ wget http://bit.ly/basic_vnfd -O hackfest_basic_vnf.tar.gz
--2019-11-12 16:48:16--  http://bit.ly/basic_vnfd
Resolving bit.ly (bit.ly)... 67.199.248.11, 67.199.248.10
Connecting to bit.ly (bit.ly)|67.199.248.11|:80... connected.
HTTP request sent, awaiting response... 301 Moved Permanently
Location: http://osm-download.etsi.org/ftp/osm-5.0-five/6th-hackfest/packages/hackfest_basic_vnf.tar.gz [following]
--2019-11-12 16:48:16--  http://osm-download.etsi.org/ftp/osm-5.0-five/6th-hackfest/packages/hackfest_basic_vnf.tar.gz
Resolving osm-download.etsi.org (osm-download.etsi.org)... 195.238.226.47
Connecting to osm-download.etsi.org (osm-download.etsi.org)|195.238.226.47|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 18632 (18K) [application/x-gzip]
Saving to: ‘hackfest_basic_vnf.tar.gz’

hackfest_basic_vnf.tar.gz                          100%[================================================================================================================>]  18.20K  --.-KB/s    in 0.1s    

2019-11-12 16:48:16 (188 KB/s) - ‘hackfest_basic_vnf.tar.gz’ saved [18632/18632]

$ wget http://bit.ly/basic_nsd -O hackfest_basic_ns.tar.gz
--2019-11-12 16:49:00--  http://bit.ly/basic_nsd
Resolving bit.ly (bit.ly)... 67.199.248.10, 67.199.248.11
Connecting to bit.ly (bit.ly)|67.199.248.10|:80... connected.
HTTP request sent, awaiting response... 301 Moved Permanently
Location: http://osm-download.etsi.org/ftp/osm-5.0-five/6th-hackfest/packages/hackfest_basic_ns.tar.gz [following]
--2019-11-12 16:49:01--  http://osm-download.etsi.org/ftp/osm-5.0-five/6th-hackfest/packages/hackfest_basic_ns.tar.gz
Resolving osm-download.etsi.org (osm-download.etsi.org)... 195.238.226.47
Connecting to osm-download.etsi.org (osm-download.etsi.org)|195.238.226.47|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 18463 (18K) [application/x-gzip]
Saving to: ‘hackfest_basic_ns.tar.gz’

hackfest_basic_ns.tar.gz                           100%[================================================================================================================>]  18.03K  --.-KB/s    in 0.1s    

2019-11-12 16:49:01 (189 KB/s) - ‘hackfest_basic_ns.tar.gz’ saved [18463/18463]

```

Then, upload the downloaded packages to OSM:

```bash
$ osmclient.osm upload-package hackfest_basic_vnf.tar.gz
/snap/osmclient/37/lib/python2.7/site-packages/osmclient/common/utils.py:69: YAMLLoadWarning: calling yaml.load() without Loader=... is deprecated, as the default Loader is unsafe. Please read https://msg.pyyaml.org/load for full details.
  dict = yaml.load(tar.extractfile(yamlfile))
d913b8e4-2f92-4c3d-bba0-2a3ad567190d
$ osmclient.osm upload-package hackfest_basic_ns.tar.gz
/snap/osmclient/37/lib/python2.7/site-packages/osmclient/common/utils.py:69: YAMLLoadWarning: calling yaml.load() without Loader=... is deprecated, as the default Loader is unsafe. Please read https://msg.pyyaml.org/load for full details.
  dict = yaml.load(tar.extractfile(yamlfile))
1eac876d-4406-4a55-b99a-baee9a672eb6
```

You can list the existing NSDs and VNFDs with the following commands:

```bash
$ osmclient.osm nsd-list
+-------------------+--------------------------------------+
| nsd name          | id                                   |
+-------------------+--------------------------------------+
| hackfest_basic-ns | 1eac876d-4406-4a55-b99a-baee9a672eb6 |
+-------------------+--------------------------------------+
$ osmclient.osm vnfd-list
+--------------------+--------------------------------------+
| nfpkg name         | id                                   |
+--------------------+--------------------------------------+
| hackfest_basic-vnf | d913b8e4-2f92-4c3d-bba0-2a3ad567190d |
+--------------------+--------------------------------------+
```

Now, as the packages have been uploaded, you can create basic Network Service:

```bash
$ osmclient.osm ns-create --ns_name hackfest_basic_ns \
                          --nsd_name hackfest_basic-ns \
                          --vim_account microstack \
                          --config '{
                              vld: [ { name: mgmtnet, vim-network-name: test } ]
                            }'
/snap/osmclient/37/lib/python2.7/site-packages/osmclient/sol005/ns.py:164: YAMLLoadWarning: calling yaml.load() without Loader=... is deprecated, as the default Loader is unsafe. Please read https://msg.pyyaml.org/load for full details.
  ns_config = yaml.load(config)
```

The Network Service is being deployed. You can check its status by executing the `osmclient.osm ns-list` command. Wait until the config status turns to "configured".

```bash
+-------------------+--------------------------------------+--------------------+---------------+-----------------+
| ns instance name  | id                                   | operational status | config status | detailed status |
+-------------------+--------------------------------------+--------------------+---------------+-----------------+
| hackfest_basic_ns | acad055e-37f4-4b59-bb31-9a6c3b972be7 | running            | configured    | done            |
+-------------------+--------------------------------------+--------------------+---------------+-----------------+
```

You can now SSH to the machine that was launched by executing the following command:

```bash
$ IP=`microstack.openstack server list -c Networks -f value --name basic | awk '{ print $2 }'`
$ ssh ubuntu@$IP -i .ssh/id_microstack
The authenticity of host '10.20.20.24 (10.20.20.24)' can't be established.
ECDSA key fingerprint is SHA256:hJYfYoa70RxYBeWZjYBt6nAVu8vqILW1xbYtnJaDPAo.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.20.20.24' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-166-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

0 packages can be updated.
0 updates are security updates.



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

ubuntu@hackfest-basic-ns-1-hackfest-basic-vm-1:~$ 
```

### Network Service with Day-1/Day-2 configuration

The previous Network Service that we deployed was very basic. Now, we will introduce more complexity, by adding Day-1 and Day-2 configuration to the VNF being onboarded using Proxy Charms.

First, let's remove the existing Network Service:

```bash
$ osmclient.osm ns-delete acad055e-37f4-4b59-bb31-9a6c3b972be7
Deletion in progress
```

Then, download NS and VNF descriptors:

```bash
$ wget http://bit.ly/simplecharm_vnfd -O hackfest_simplecharm_vnf.tar.gz
--2019-11-12 16:51:24--  http://bit.ly/simplecharm_vnfd
Resolving bit.ly (bit.ly)... 67.199.248.10, 67.199.248.11
Connecting to bit.ly (bit.ly)|67.199.248.10|:80... connected.
HTTP request sent, awaiting response... 301 Moved Permanently
Location: https://osm-download.etsi.org/ftp/osm-6.0-six/7th-hackfest/packages/hackfest_simplecharm_vnf.tar.gz [following]
--2019-11-12 16:51:24--  https://osm-download.etsi.org/ftp/osm-6.0-six/7th-hackfest/packages/hackfest_simplecharm_vnf.tar.gz
Resolving osm-download.etsi.org (osm-download.etsi.org)... 195.238.226.47
Connecting to osm-download.etsi.org (osm-download.etsi.org)|195.238.226.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6427585 (6.1M) [application/x-gzip]
Saving to: ‘hackfest_simplecharm_vnf.tar.gz’

hackfest_simplecharm_vnf.tar.gz                    100%[================================================================================================================>]   6.13M  1.70MB/s    in 3.6s    

2019-11-12 16:51:28 (1.70 MB/s) - ‘hackfest_simplecharm_vnf.tar.gz’ saved [6427585/6427585]

$ wget http://bit.ly/simplecharm_nsd -O hackfest_simplecharm_ns.tar.gz
--2019-11-12 16:51:57--  http://bit.ly/simplecharm_nsd
Resolving bit.ly (bit.ly)... 67.199.248.10, 67.199.248.11
Connecting to bit.ly (bit.ly)|67.199.248.10|:80... connected.
HTTP request sent, awaiting response... 301 Moved Permanently
Location: https://osm-download.etsi.org/ftp/osm-6.0-six/7th-hackfest/packages/hackfest_simplecharm_ns.tar.gz [following]
--2019-11-12 16:51:57--  https://osm-download.etsi.org/ftp/osm-6.0-six/7th-hackfest/packages/hackfest_simplecharm_ns.tar.gz
Resolving osm-download.etsi.org (osm-download.etsi.org)... 195.238.226.47
Connecting to osm-download.etsi.org (osm-download.etsi.org)|195.238.226.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 18457 (18K) [application/x-gzip]
Saving to: ‘hackfest_simplecharm_ns.tar.gz’

hackfest_simplecharm_ns.tar.gz                     100%[================================================================================================================>]  18.02K  --.-KB/s    in 0.1s    

2019-11-12 16:51:58 (186 KB/s) - ‘hackfest_simplecharm_ns.tar.gz’ saved [18457/18457]

```

Upload the downloaded packages to OSM:

```bash
$ osmclient.osm upload-package hackfest_simplecharm_vnf.tar.gz
/snap/osmclient/37/lib/python2.7/site-packages/osmclient/common/utils.py:69: YAMLLoadWarning: calling yaml.load() without Loader=... is deprecated, as the default Loader is unsafe. Please read https://msg.pyyaml.org/load for full details.
  dict = yaml.load(tar.extractfile(yamlfile))
56c16434-ca44-49d7-b4c3-0b3787fe9ce1
$ osmclient.osm upload-package hackfest_simplecharm_ns.tar.gz
/snap/osmclient/37/lib/python2.7/site-packages/osmclient/common/utils.py:69: YAMLLoadWarning: calling yaml.load() without Loader=... is deprecated, as the default Loader is unsafe. Please read https://msg.pyyaml.org/load for full details.
  dict = yaml.load(tar.extractfile(yamlfile))
267c4095-11f1-446b-ae81-b61b9152a183
```

Create the Network Service:
```bash
$ osmclient.osm ns-create --ns_name hackfest_simplecharm_ns \
                          --nsd_name hackfest_simplecharm-ns \
                          --vim_account microstack \
                          --config '{
                              vld: [ { name: mgmtnet, vim-network-name: test } ]
                            }'
/snap/osmclient/37/lib/python2.7/site-packages/osmclient/sol005/ns.py:164: YAMLLoadWarning: calling yaml.load() without Loader=... is deprecated, as the default Loader is unsafe. Please read https://msg.pyyaml.org/load for full details.
  ns_config = yaml.load(config)
```

The Network Service is being deployed. Again, you can check its status by executing the `osmclient.osm` command. Wait until the config status turns to "configured".

positive
: It can take up to 5 minutes for the Network Service to be deployed.

```bash
$ osmclient.osm ns-list
+-------------------------+--------------------------------------+--------------------+---------------+-----------------+
| ns instance name        | id                                   | operational status | config status | detailed status |
+-------------------------+--------------------------------------+--------------------+---------------+-----------------+
| hackfest_simplecharm_ns | ba9d0007-45a7-4bc7-bb66-05c808cd31a5 | running            | configured    | done            |
+-------------------------+--------------------------------------+--------------------+---------------+-----------------+
```

The action available in this Network Service consists of creating a file inside the VNF at the path specified by the filename parameter.

Execute the action:

```bash
$ osmclient.osm ns-action --action_name touch \
                          --vnf_name 1 \
                          --params '{filename: /home/ubuntu/touched}' \
                          hackfest_simplecharm_ns
/snap/osmclient/37/lib/python2.7/site-packages/osmclient/scripts/osm.py:2808: YAMLLoadWarning: calling yaml.load() without Loader=... is deprecated, as the default Loader is unsafe. Please read https://msg.pyyaml.org/load for full details.
  op_data['primitive_params'] = yaml.load(params)
10b41672-e0bc-4cca-ba4e-4607d910ee1e
```

And check whether the file has been created:

```bash
$ IP=`microstack.openstack server list -c Networks -f value --name simplecharm | awk '{ print $2 }'`
$ ssh ubuntu@$IP -i .ssh/id_microstack ls
The authenticity of host '10.20.20.24 (10.20.20.24)' can't be established.
ECDSA key fingerprint is SHA256:PhIBx7jV8gTuGK5Hu/tqpEPdiTrg7zzGwHFa2CEbLT0.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.20.20.24' (ECDSA) to the list of known hosts.
first-touch
touched
```

## Enable/Disable the Development Stack

Duration: 2:00

We encourage you to disable the Development Stack when you are not using it, especially if you are running it on your workstation. This way, you do not waste your workstation’s resources. You can disable the Development Stack by running the following commands:

```bash
$ microk8s.disable dns
Disabling DNS
Reconfiguring kubelet
Removing DNS manifest
serviceaccount "coredns" deleted
configmap "coredns" deleted
deployment.apps "coredns" deleted
service "kube-dns" deleted
clusterrole.rbac.authorization.k8s.io "coredns" deleted
clusterrolebinding.rbac.authorization.k8s.io "coredns" deleted
DNS is disabled
$ sudo snap disable microstack
microstack disabled
$ sudo snap disable microk8s
microk8s disabled
```

You can always enable the Development Stack with the following commands:

```bash
$ sudo snap enable microstack; sleep 60
microstack enabled
$ sudo snap enable microk8s
microk8s enabled
$ microk8s.status --wait-ready
microk8s is running
addons:
cilium: disabled
dashboard: disabled
dns: enabled
fluentd: disabled
gpu: disabled
helm: disabled
ingress: disabled
istio: disabled
jaeger: disabled
knative: disabled
linkerd: disabled
metrics-server: disabled
prometheus: enabled
rbac: disabled
registry: disabled
storage: enabled
$ microk8s.enable dns
Enabling DNS
Applying manifest
serviceaccount/coredns created
configmap/coredns created
deployment.apps/coredns created
service/kube-dns created
clusterrole.rbac.authorization.k8s.io/coredns created
clusterrolebinding.rbac.authorization.k8s.io/coredns created
Restarting kubelet
DNS is enabled
```

In general, when Openstack compute nodes are restarted, the servers are in the shutoff state, so you will need to start them:

```bash
$ microstack.openstack server list
$ microstack.openstack server start <server ID>
```

## That’s all folks!

Duration: 2:00

Congratulations! You have made it!

* Deploy Charmed OSM on top of MicroK8s,
* Deploy MicroStack and add it as a VIM for Charmed OSM,
* Onboard sample VNF workloads through Charmed OSM.

In this tutorial you have learned how to deploy Charmed OSM on top of MicroK8s, add MicroStack as a VIM and onboard sample VNF workloads through Charmed OSM. You can now use your newly learned skills to accelerate your transition to NFV with OSM.

### Where to go from here?

Visit Charmed OSM [website][charmed_osm_website]
Read more about [Canonical's solutions for telcos][telco_website]
Explore [MicroK8s][microk8s_website] and [MicroStack][microstack_website]
Read upstream OSM [documentation][osm_documentation]
[Tell us][contact] your NFV story!

<!-- LINKS -->
[contact]: https://ubuntu.com/contact-us/form?product=generic-contact-us
[charmed_osm_website]: https://charmed-osm.com
[charms_website]: https://jaas.ai
[microk8s_snap]: https://snapcraft.io/microk8s
[microk8s_website]: https://microk8s.io
[microstack_snap]: https://snapcraft.io/microstack
[microstack_website]: https://microstack.run
[multipass]: https://multipass.run/#install
[osm_documentation]: https://osm.etsi.org/wikipub/index.php/Main_Page
[osm_website]: https://osm.etsi.org/
[snapd]: https://snapcraft.io/docs/installing-snapd
[telco_website]: https://ubuntu.com/telecommunications
