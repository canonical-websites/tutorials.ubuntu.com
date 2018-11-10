---
id: tutorial-charm-development-part2
summary: Second part of the charm development tutorials
categories: cloud, development
tags: tutorial,juju,charm,development
difficulty: 4
status: unpublished
published: 2018-11-10
author: Erik Lönroth <erik.lonroth@gmail.com>
feedback_url: https://github.com/canonical-websites/tutorials.ubuntu.com/issues
---

# Juju Charm development (part 2)
Duration: 0:05

## Building on part 1
Duration: 0:05

In the last part, you learned how to build and deploy your first charm.
If you havent done that part, go ahead and do that before doing this tutorial.

## What you will learn
Duration: 02:00

In this development tutorial, we will cover:

* Learn about the "layer-index".
* Learn how to add a 'mysql' database interface to charms.
* Write a charm that writes a text-file using a template.
* Deploy the example charm and relate to a mysql charm.
* Use 'juju ssh' to login to the juju unit and display the contents of the text-file.

* Preparing and setup of a basic workbench
* Creating the example charm with *charm tools*
* Understanding the anatomy of a charm (files and directories)
* Validating and building the charm
* Adding functionality via a secondary layer (layer:apt)
* Deploying the example charm with Juju


## The 'layer-index'
Duration: 2:00

Lets examine the inludes tag in ** layer.yaml **
```yaml
includes: 
  - 'layer:basic'
  - 'layer:apt'
```

You learned from the first part of the tutorial that the 'includes' tag in 'layer.yaml' is similar to an "include" statement in python. All charms must include the 'layer:base'.

The build system will add in code from the 'include' tag with our own.

Alot of already existing juju functionality can be found in the [layer-index].
Take a look and see if you find some interesting features you like to explore later.

As you see, we have also already added the [layer:apt].

Since we can include both 'layers' and 'interfaces' lets see how that is done.

## Add the interface:mysql
Duration: 10:00

To add the mysql interface, we have to:

* Modify 'layer.yaml'
* Modify 'metadata.yaml'
* Create a template inside a templates directory.

Lets do that.

Modify the **~/charms/layers/layer-example/layer.yaml** to look like this:

```yaml
includes: 
  - 'layer:basic'
  - 'layer:apt'
  - 'interface:mysql'
options:
  apt:
    packages:
     - hello 
```

Modify the **~/charms/layers/layer-example/metadata.yaml** to look like this:

```yaml
name: example
summary: A very basic example charm
maintainer: Your Name <your.name@mail.com>
description: |
  This is a charm I built as part of my beginner charming tutorial.
tags:
  - misc
  - tutorials
requires:
  database:
    interface: mysql
```


## Create the templates directory:
Duration: 05:00

The 'templates' directory is where templates go for charms. Lets create it.

```bash
cd ~/charms/layers/layer-example/
mkdir templates 
```

Modify the **~/charms/layers/layer-example/templates/text-file.tmpl** to look like this:

```no-highlight
DB_HOST = '{{ my_database.host() }}'
DB_NAME = '{{ my_database.database() }}'
DB_USER = '{{ my_database.user() }}'
DB_PASSWORD = '{{ my_database.password() }}'
```

The file above is in a "Jinja2" template format. Jinja2 templates allows us to perform very advanced template rendering with little effort.


positive
: You are getting good at this!

## Rewrite the layer_example.py
Duration: 05:00

** ~/charms/layers/layer-example/reactive/layer_example.py **
```python
from charms.reactive import set_flag, when, when_not
from charmhelpers.core.hookenv import application_version_set, status_set
from charmhelpers.fetch import get_upstream_version
import subprocess as sp
from charmhelpers.core.templating import render


@when_not('example.installed')
def install_example():
    set_flag('example.installed')


@when('apt.installed.hello')
def set_message_hello():
    # Set the upstream version of hello for juju status.
    application_version_set(get_upstream_version('hello'))

    # Run hello and get the message
    message = sp.check_output('hello', stderr=sp.STDOUT)

    # Set the maintenance status with a message
    status_set('maintenance', message )

    # Signal that we know the version of hello
    set_flag('hello.version.set')


@when('database.available')
def write_text_file(mysql):
    render(source='text-file.tmpl',
           target='/root/text-file.txt',
           owner='root',
           perms=0o775,
           context={
               'my_database': mysql,
           })
    status_set('active', 'Ready: File rendered.')


@when_not('database.connected')
def missing_mysql():
    status_set('blocked', 'Please add relation to MySQL')


@when('database.connected')
@when_not('database.available')
def waiting_mysql(mysql):
    status_set('waiting', 'Waiting for MySQL')
```

What happens here is:

* We install the 'hello' package, just as we did in the first part of the tutorial.
* The mysql interface raises a flag 'database.available' when the relation is joined, mysql has created a database, a username and password for us. 
* The database mysql object with the information we need is passed to us from the juju interface.
* We pass on the mysql object to Jinja2 render() that renders the template.
* We finish our work by setting the 'active' status.

## What is the difference from tutorial part 1?
Duration: 10:00

Look at the code in the function *set_message_hello()*. In the previous part of this tutorial, we set the 'active' status when the hello package was installed which was all we wanted our charm to do at that point.

However, in this part of the tutorial, we set status to 'maintenance' instead so we can do more work without signaling that we are 'active'.
```python
    # Set the maintenance status with a message
    status_set('maintenance', message )
```

Status is set to 'active' in the function *write_text_file(mysql)* when the template is rendered which is the goal for this charm.

```python
@when('database.available')
def write_text_file(mysql):
    render(source='text-file.tmpl',
           target='/root/text-file.txt',
           owner='root',
           perms=0o775,
           context={
               'my_database': mysql,
           })
        status_set('active', 'Ready: File rendered.')
```

We are ready to rebuild.

## Proof, Build, Deploy, Relate
Duration: 05:00

We are done and can run through the build, deployment and try relate our charm to the mysql charm.

```bash
cd ~/charms/layers
charm proof layer-example
charm build layer-example
juju deploy ../trusty/example
juju deploy cs:mysql
juju relate example mysql

```

Note: We deploy our local charm by referencing with a path (../trusty/example). mysql is on the other hand is deployed from the Charm Store (cs:mysql). "cs:mysql" tells juju to download the charm from over the network instead of uploading it from our local directory repository.

## Check the result with 'juju ssh'
Duration: 03:00

Lets look at the file we created inside the example/0 unit. We will do that using "juju ssh".

```bash
juju ssh example/0 sudo cat /root/text-file.txt

```
juju ssh executes a command on the remote machine and if everything goes well, you should see the contents of the rendered file, something like below:

```no-highlight
DB_HOST = '10.170.55.25'
DB_NAME = 'example'
DB_USER = 'Dae7EGh9Zei0nee'
DB_PASSWORD = 'aiRei1siePhewah'
```

positive:
: Congratulations, you have completed the second part of the tutorial!

## Next lesson: 
Duration: 00:10

In the next tutorial you will learn how to publish your charm to the charmstore so you can use it from anywhere around the world, in any cloud!

Move on to the next part of the tutorial series in [part 3/3]

## Author
[Erik Lönroth](http://eriklonroth.wordpress.com)
