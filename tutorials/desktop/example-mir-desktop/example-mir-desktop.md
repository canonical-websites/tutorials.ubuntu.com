---
id: egmde
summary: Learn to create a Wayland Desktop Environment using Mir
categories: desktop
tags: tutorial, mir
difficulty: 3
status: draft
feedback_url: https://github.com/canonical-websites/tutorials.ubuntu.com/issues
published: 2018-05-15
author: Alan Griffiths <alan@octopull.co.uk>

---

#  Egmde: A Wayland Desktop Environment Using Mir

## Introduction

[Mir](https://mirserver.io) is a library designed to facilitate writing GUI shells for a range of platform from IoT and phones to desktops.

This tutorial will take you through the development of an example desktop environment that supports Wayland applications.


### What you'll learn

You will learn how to employ the Mir API to produce a simple, but usable desktop environment: egmde.

### What you'll need

You can follow the steps in this tutorial on any supported release of Ubuntu from 17.10 or on Fedora from release 28. *Unfortunately, the toolkit support for Wayland in Ubuntu 16.04LTS isn't sufficiently up to date for everything to work.*

## Preparation

The code in this tutorial needs [Mir 0.31](https://community.ubuntu.com/t/mir-0-31-0-release/4637/6) or later. Mir 0.31 exists in Ubuntu 18.04 and Fedora 28 archives.

On Ubuntu, it is recommended to use the mir-team/release PPA which is available for all current releases of Ubuntu.

### On Ubuntu:

Add the mir-team/release PPA on Ubuntu:

```bash
sudo apt-add-repository ppa:mir-team/release
sudo apt update
```

Install some development tools and the Mir libraries. It is also useful to install the `weston` package as the example makes use of `weston-terminal` as a Wayland based terminal application and the Qt toolkit's Wayland support : `qtwayland5`.

```bash
sudo apt install g++ cmake
sudo apt install libmiral-dev mir-graphics-drivers-desktop 
sudo apt install weston qtwayland5
```

### On Fedora:

Install some development tools and the Mir libraries. It is also useful to install the `weston` package as the example makes use of `weston-terminal` as a Wayland based terminal application and the Qt toolkit's Wayland support : `qtwayland5`.

```bash
sudo dnf install gcc-c++ cmake
sudo dnf install mir-devel
sudo dnf install weston qt5-qtwayland
```

## egmde step 1: A minimally viable shell

To illustrate MirAL we're going to review code from a (very simple) window manager. It runs on desktops, tablets and phones and supports keyboard, mouse and touch input. It will support applications using the GTK and Qt toolkits, SDL applications and (using Xwayland X11 applications.

The full code for this example is available on github:

```bash
git clone https://github.com/AlanGriffiths/egmde.git
cd egmde
git checkout article-1
```

*Naturally, the code is likely to evolve, so you will find other branches, but this branch goes with this stage.*
 
 Assuming that you’ve MirAL installed as described above you can now build egmde as follows:

```bash
mkdir build
cd build
cmake ..
make
```

After this you can start a basic egmde based desktop. This will use VT4, so first switch to VT4 (Ctrl-Alt-F4) to sign in and switch back again. Then type:

```bash
./egmde-desktop
```

You should see a blank screen with a `weston-terminal` session. From this you can run commands and, in particular, start graphical applications. Perhaps `qtcreator` to examine the code?

## The example code

A lot of the functionality (default placement of windows, menus etc.) comes with Mir's `libmiral` library. For this exercise we have implemented one class and written a main function that injects it into MirAL. The main program looks like this:

```c++
using namespace miral;

int main(int argc, char const* argv[])
{
    MirRunner runner{argc, argv};

    return runner.run_with(
        {
            set_window_management_policy<egmde::WindowManagerPolicy>()
        });
}
```

Yes, you’ve guessed it: the egmde code is in `egmde::WindowManagerPolicy`. It looks like this:

```c++
class WindowManagerPolicy : public CanonicalWindowManagerPolicy
{
public:
    using CanonicalWindowManagerPolicy::CanonicalWindowManagerPolicy;

    bool handle_keyboard_event(MirKeyboardEvent const* event) override;

    bool handle_pointer_event(MirPointerEvent const* event) override;

    bool handle_touch_event(MirTouchEvent const* event) override;

    Rectangle confirm_placement_on_display(
        WindowInfo const& window_info, MirWindowState new_state, Rectangle const& new_placement) override;

    void handle_request_drag_and_drop(WindowInfo& window_info) override;

    void handle_request_move(WindowInfo& window_info, MirInputEvent const* input_event) override;

    void handle_request_resize(WindowInfo& window_info, MirInputEvent const* input_event, MirResizeEdge edge) override;

private:
...
};
```

These are the functions that it is necessary to implement for a minimal shell. I won't reproduce them all here, but this should give a flavor:

```c++
bool egmde::WindowManagerPolicy::handle_keyboard_event(MirKeyboardEvent const* event)
{
    auto const action = mir_keyboard_event_action(event);
    auto const shift_state = mir_keyboard_event_modifiers(event) & shift_states;

    if (action == mir_keyboard_action_down && shift_state == mir_input_event_modifier_alt)
    {
        switch (mir_keyboard_event_scan_code(event))
        {
            case KEY_F4:
                tools.ask_client_to_close(tools.active_window());
                return true;

            case KEY_TAB:
                tools.focus_next_application();
                return true;

            case KEY_GRAVE:
                tools.focus_next_within_application();
                return true;

            default:;
        }
    }

    return false;
}
```

### Keeping score

There's very little code needed to get this basic shell running:

```bash
$ wc -l *.h *.cpp *.sh
   88 egwindowmanager.h
   34 egmde.cpp
  420 egwindowmanager.cpp
   47 egmde-desktop.sh
  589 total
```

