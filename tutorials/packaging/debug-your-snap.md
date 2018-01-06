---
id: debug-your-snap
summary: We are going to walk though ways to debug a snap.
categories: packaging
status: published
tags: snapcraft, debug
difficulty: 2
published: 2018-01-08
feedback_url: https://github.com/canonical-websites/tutorials.ubuntu.com/issues
author: Daniel Lim Wee Soong <weesoong.lim@gmail.com>

---

# Debug your snap

## Overview
Duration: 1:00

Snaps are Linux packages you can install on a wide range of Linux distributions. They are installed from a centralized store and run in a confined sandbox. 

The trouble with sandboxed apps is that they can be harder to debug that non-sandboxed (traditionally packaged) apps. This tutorial will take readers through common steps to debug a malfunctioning snap.

### What you’ll learn

  - How to debug a snap without confinement.
  - How to debug a confined snap.

### What you’ll need

  - Any supported snap GNU/Linux distribution .
  - Some very basic knowledge of command line use, know how to edit files.
  - Knowledge of using snapcraft to package a snap.


Survey
: How will you use this tutorial?
 - Only read through it
 - Read it and complete the exercises
: What is your current level of experience?
 - Novice
 - Intermediate
 - Proficient


## Getting started
Duration: 1:00

Apart from snapcraft, all you need is `snappy-debug`.

### Installing dependencies

Now simply run:


```bash
$ sudo snap install snappy-debug
```

This will install snappy-debug which is very helpful in debugging confined snaps.

We’re all set now, let’s get cracking and let’s see how we can debug our snap easily!


## Debugging without confinement
Duration: 3:00

By default snaps run in a restrictive sandbox to ensure that they can only access the system and other snaps in controlled ways. The `confinement` key is used in snapcraft.yaml to specify whether or not the snap is expected to work correctly when the specified interfaces are connected and the snap is confined. 

We specify `strict` to indicate the snap works properly when confined or `devmode` to indicate it only works properly when unconfined. If confinement is unspecified, the snap is assumed to work correctly when confined since the snaps are expected to be developed for running in the sandbox. We're only allowed to upload snaps to the stable channel when strict confinement is used.

It is best to test our snap without confinement first. That will make sure any problem that arises is due to the app itself and not `snapcraft`.

### Ignore confinement

To do this, specify `confinement: devmode` in the snapcraft.yaml file like so:


```yaml
name: hello-world-service
confinement: devmode
apps:
  hello-service:
    command: command-hello-service.wrapper
    daemon: simple
    plugs:
    - network-bind
...
```

We should then specify `--devmode` when installing the snap for testing and development.

negative
: Important: You must always specify --devmode when installing in devmode regardless of how confinement is set in your yaml.

At this time, any errors that occur must be related to the app itself and not `snapcraft`.

Simple, isn't it. Let's see what can we do once we are sure the snap runs perfectly without confinement.


## Confined snaps
Duration: 3:00

Once the snap is working well without confinement, it is time to confine it.

### Confining the snap
Usually, snaps should be under the strict confinement, such that it is not able to access files from directories other than the one it is living in.

We can test the snap with confinement by installing it without specifying `--devmode`, and iterate until it is working properly under confinement. When satisfied we can indicate that it works under confinement with:

```yaml
name: hello-world-service
confinement: strict
apps:
  hello-service:
    command: command-hello-service
    daemon: simple
    plugs:
    - network-bind
...
```

But what if it doesn't work?

## Debugging confined snaps

When debugging our snap when it’s installed in `strict` mode (without `--devmode`), we can use the `grep "audit:" /var/log/syslog` command for sandbox denials. 

Alternatively, we can use the `snappy-debug` snap to assist us:

```bash
$ sudo snap install snappy-debug
$ sudo /snap/bin/snappy-debug.security scanlog hello-world-service
```

This will let us know if services like AppArmor or Seccomp is denying access to anything. At the same time, it will give us suggestions on how to prevent the error.

### Plugs
One type of suggestion would be to add plugs, like the following example:

```
Suggestions:
* add 'network-control' to 'plugs'
```

To fix this, we can add the relevant plug to the YAML file, for example
```yaml
name: hello-world-service
confinement: strict
apps:
  hello-service:
    command: command-hello-service
    daemon: simple
    plugs:
    - network-bind
    - network-control
...
```

### No permission to open file
Another suggestion would be to change the program to only use `$SNAP_DATA` or any snap related environment variables. This happens when the snap is trying to access a file or directory out of the one it is living in. 

If it is required for the snap to do so, we will need to install the snap with the `--classic` flag. Classic snaps are given the permission to access any file outside of the directory it is living in.

When satisfied, change the confinement type to `classic`, like the following,
```yaml
name: hello-world-service
confinement: classic
apps:
  hello-service:
    command: command-hello-service
    daemon: simple
    plugs:
    - network-bind
    - network-control
...
```

If so, why not just set our snap to use classic confinement and we would have been done earlier? One consideration is, due to security concerns, we need to post a request to allow a classic snap in the snapcraft forum, and have a valid reason for the request to be approved.

If it is not of utmost neccesity that the snap access files outside, it will have to be strictly confined. So, not all snaps can have classic confinement.

Well, that's it, this should be almost everything we need to know about debugging a confined snap.

## That’s all folks!
Duration: 1:00

### Easy, wasn’t it?

Congratulations! You made it!

By now you should be able to debug your snap when anything goes wrong.

### Next steps

  - You can have a look at “[build a nodejs service]” which is the logical follow up of that tutorial, bringing your some debugging techniques, more information on confinement and how to package some snap application as a service.
  - Learn some more advanced techniques on how to use your snap system looking for our others tutorials!
  - Join the snapcraft.io community on the [snapcraft forum].

### Further readings
  - [Snapcraft.io documentation] is a good place to start reading the whole snap and snapcraft documentation.
  - Documentation on [interfaces].
  - [Snapcraft syntax reference], covering various available options like the daemon ones.
  - Check how you can [contact us and the broader community].





[basic snap usage]: https://tutorials.ubuntu.com/tutorial/basic-snap-usage
[here1]: https://github.com/ubuntu/snap-tutorials-code/tree/master/create-your-first-snap/step3
[here2]: https://github.com/ubuntu/snap-tutorials-code/tree/master/create-your-first-snap/step4
[here3]: https://github.com/ubuntu/snap-tutorials-code/tree/master/create-your-first-snap/step5
[here4]: https://github.com/ubuntu/snap-tutorials-code/tree/master/create-your-first-snap/step6
[My Apps]: https://dashboard.snapcraft.io/dev/snaps/
[available here]: https://docs.snapcraft.io/build-snaps/builders
[this]: https://github.com/ubuntu/snap-tutorials-code/tree/master/create-your-first-snap/final
[build a nodejs service]: https://tutorials.ubuntu.com/tutorial/build-a-nodejs-service
[snapcraft forum]: https://forum.snapcraft.io/
[Snapcraft.io documentation]: http://snapcraft.io/docs/
[interfaces]: https://snapcraft.io/docs/core/interfaces
[Snapcraft syntax reference]: http://snapcraft.io/docs/build-snaps/syntax
[contact us and the broader community]: http://snapcraft.io/community/
