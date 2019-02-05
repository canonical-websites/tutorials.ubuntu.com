---
id: snap-a-website
summary: In this tutorial, you’ll learn how to create a desktop web app with Electron and package it as a snap.
categories: packaging
tags: snapcraft, snap, beginner, nodejs, electron, website
status: published
difficulty: 2
published: 2017-11-14
author: David Callé <david.calle@canonical.com>
feedback_url: https://github.com/canonical-websites/tutorials.ubuntu.com/issues

---

# Snap a website with Electron

## Overview
Duration: 1:00

Turning your website into a desktop integrated app is a relatively simple thing to do, but distributing it as such and making it noticeable in app stores is another story.

This tutorial will show you how to leverage Electron and snaps to create a website-based desktop app from scratch and publish it on a multi-million user store shared between many Linux distributions.

For this tutorial, the website we are going to package is an HTML5 game called [Castle Arena](http://castlearena.io).

![](https://assets.ubuntu.com/v1/7f7e704f-shot.png)

### What you'll learn

- How to create a website-based desktop app using Electron
- How to turn it into a snap package
- How to test it and share it with the world

### What you'll need

- Ubuntu Desktop 16.04 or above
- Some basic command-line knowledge

We are going to start by downloading some dev tools and set up the project.

## Electron setup
Duration: 3:00

Let's start by opening a terminal and installing some basic development tools, notably:

* **curl**: a command-line utility to download remote content
* **build-essential**: a collection of dependencies for building source code
* **libgconf2-4**: an Electron dependency, commonly used to store app configuration values
* **nodejs**: A JavaScript runtime, that will also provide the `npm` command we are going to use

```bash
sudo snap install node --channel=11/stable
```

Create a `castlearena` project directory and enter it:

```bash
mkdir castlearena
cd castlearena
```

Inside our project, let's create an `app` directory to host the Electron app and its dependencies, and enter it.

```bash
mkdir app
cd app
```

With the `npm` command, we can install *electron* and *electron-builder* inside the current directory. They will provide the framework and tooling to build our app binary.

```bash
npm install electron --save-dev --save-exact
npm install electron-builder --save-dev
```

When this is done we can move to the next step: creating the app.

## Creating the app
Duration: 10:00

We are going to start by creating a `main.js` file.

Open your favorite editor and save the following code as `main.js`

```js
const electron = require('electron');
const { shell, app, BrowserWindow } = electron;
const HOMEPAGE = 'http://castlearena.io'

let mainWindow;

app.on('ready', () => {
    window = new BrowserWindow({
        width: 1200,
        height: 900,
        webPreferences: {
          nodeIntegration: false
        }
    });
    window.loadURL(HOMEPAGE);

    window.on('closed', () => {
        window = null;
    });
});

```

That's pretty much all we need to display a website in a standalone window.

Positive
: Note that we have defined a `HOMEPAGE` variable, which contains the base URL of our app.
This could point to a local file as well.

Now, from your project directory (`castlearena/app/`), let's try the app:

```bash
./node_modules/.bin/electron main.js
```

You should now see a new window!

![](https://assets.ubuntu.com/v1/38b307f0-snap-a-website-app-1.png)

### Some refinements

Our app runs, which is a good start, but we are going to refine it a little bit.

#### Hiding the menu

Electron apps show a menu by default. We can either keep it intact, customize it or hide it.

Since our HTML5 game doesn't need any menu, we are going to hide it with the `setMenuBarVisibility` function:

```js
window.setMenuBarVisibility(false);
```

Let's add this line right above the `loadURL` function call, so our file looks like this:

```js
const electron = require('electron');
const { shell, app, BrowserWindow } = electron;
const HOMEPAGE = 'http://castlearena.io'

let mainWindow;

app.on('ready', () => {
    window = new BrowserWindow({
        width: 1200,
        height: 900,
        webPreferences: {
          nodeIntegration: false
        }
    });
    window.setMenuBarVisibility(false);
    window.loadURL(HOMEPAGE);

    window.on('closed', () => {
        window = null;
    });
});

```

#### Opening external URLs in the default browser

External URLs are URLs leading outside our app. Since our app is not a web browser with navigation (back or forward) buttons, let's deal with them by opening them in the default web browser.

When a URL is about to be loaded, Electron triggers a `will-navigate` event. We are going to listen to these events and compare the URL they provide with the domain of our app. If they don't match, we will open the URL in the default browser.

This snippet takes care of it:

```js
window.webContents.on('will-navigate', (ev, url) => {
    parts = url.split('/');
    if (parts[0] + '//' + parts[2] != HOMEPAGE) {
        ev.preventDefault();
        shell.openExternal(url);
    };
});
```

We are going to add it to our code, so it looks like this:

```js
const electron = require('electron');
const { shell, app, BrowserWindow } = electron;
const HOMEPAGE = 'http://castlearena.io'

let mainWindow;

app.on('ready', () => {
    window = new BrowserWindow({
        width: 1200,
        height: 900,
        webPreferences: {
          nodeIntegration: false
        }
    });
    window.setMenuBarVisibility(false);
    window.loadURL(HOMEPAGE);

    window.webContents.on('will-navigate', (ev, url) => {
        parts = url.split('/');
        if (parts[0] + '//' + parts[2] != HOMEPAGE) {
            ev.preventDefault();
            shell.openExternal(url);
        };
    });

    window.on('closed', () => {
        window = null;
    });
});
```

We are done with refinements, and we are almost done with the app. You can test it some more to see if it behaves as you would expect:

```bash
./node_modules/.bin/electron main.js
```

 Next, we are going to add some metadata to the app project and use electron-builder to generate a snap.

## Electron metadata (package.json)
Duration: 2:00

Now that we have a working app, we are going to turn it into an executable file using a dependency we installed earlier: electron-builder To build the executable, it requires a `package.json` file containing some basic metadata.

In your favorite editor, create a `package.json` file with the following content:

```json
{
  "name": "castlearena",
  "version": "0.1.0",
  "main": "./main.js",
  "scripts": {
    "start": "electron ."
  },
  "build": {
    "linux": {
      "target": ["snap"]
    }
  }
}
```

As you can see, all entries are self-explanatory. We have a name, a version, the location of our app, how to start the app using the `electron` command and instructions to build a snap package

Positive
: This is the simplest version of a `package.json` that works with electron-builder. You can extend it with "author", "description" and a lot of other fields, but since we are packaging this app as a snap, we have filled th minimum required to successfully build it.

Save the file as `package.json` in our Electron project (`castlearena/app`), alongside `main.js`.

We are done with the app! Now, let's have a look at our snap packaging.

### Application Icon

By default `electron-builder` will add a stock icon to the application when built. We should override that with an icon of our own design. Once installed, users will have the icon in their menu/launcher. Electron builder has different expectations for how icons are generated based on the build platform. This is covered in their [documentation](https://www.electron.build/icons). For Linux the minimum we need is a `build/icons/256x256.png`.

Create this directory by running, from the `app` directory of our project:

```bash
mkdir build/icons
```

Now, let's add an icon.

#### The icon

This one is a good match for our app, download it and save it as `256x256.png` in the `build/icons/` directory.

![](https://assets.ubuntu.com/v1/b2af323c-icon.png?w=256)

Positive
: A size of 256x256px is generally a safe bet for desktop icons to look good under most circumstances. However, Electron Builder supports smaller, and larger icons. See their [documentation](https://www.electron.build/icons) for more information.

In the next step, we are going to run `electron-builder`, install our new snap and test it.

## Building and testing the snap
Duration: 8:00

Now that we have created all the pieces: the app, the electron metadata (`package.json`), and the icon, it's time to build the snap.

The command we are going to use is `electron-builder`: it will build the application and package up everything needed into a snap in the `dist/` folder.

From the root of our project (`castlearena/app`), run:

```bash
./node_modules/.bin/electron-builder
```

When the process is over, you should see a new file at the root of your project:

`castlearena_0.1_amd64.snap`

**Congrats, you made a snap!**

### Time for some testing...

To install the snap on your system, run:

```bash
snap install --dangerous castlearena_0.1_amd64.snap
```

Positive
: **What does `--dangerous` mean?**
The `--dangerous` flag indicates that you are acknowledging that this snap could be unsafe to install, this is necessary when the snap hasn't been through store security checks. Since you are the developer of the snap, you know it's safe, but the snap command doesn't and would prevent the install if you omitted the flag.

Then, you can start the app with:

```bash
castlearena
```

If you followed each step, you should be greeted with a pretty fun game:

![victory](https://assets.ubuntu.com/v1/10195d75-Peek+2017-11-14+17-01.gif)

You should also search for the app in your desktop app list and check if the icon renders nicely! If something looks wrong, at this point you probably know where to fix it: the desktop file.

## What next?
Duration: 1:00

What a journey! If this is the first time you installed a snap or created a LXD container, you may have realised that most of the time spent in this tutorial was looking at downloads! When the tools have been used once, things get much faster, since most of them don't have to be installed or initialised anymore.

### Let's recap what we went through

  * You know how to create a basic Electron app
  * You know how to snap an Electron app in a clean environment

### How to share my snap with the world?

To release it in the Snap Store and make it available to others, privately or publicly, follow the [publishing steps](https://docs.snapcraft.io/build-snaps/publish).

### What if the snap doesn't work?

Here are some commonly used techniques to debug snaps:

* Inspect system logs for security denials:
  `tail -f /var/log/syslog | grep <snap>`
* Run your snap in `devmode` (developer mode), to replace AppArmor security denials by warnings:
  `snap install --devmode <snap package>` will install the snap in `devmode`
  Then inspect system logs for security warnings with `tail -f /var/log/syslog | grep <snap>`
* Mount a directory as a snap (with write access):
  `unsquashfs <snap package>` will unpack the snap into a `squashfs-root` drectory.
  `snap try squashfs-root` will mount the directory as if it was an installed snap.
  You can then run the snap and edit its content at the same time.
* Enter a shell with the same confinement as the snap to inspect its environment, paths, and see what it sees:
  `snap run --shell <snap>`

### Where to find support?

The [snapcraft forum](https://forum.snapcraft.io) is the best place to get all your questions answered and get in touch with the community.
