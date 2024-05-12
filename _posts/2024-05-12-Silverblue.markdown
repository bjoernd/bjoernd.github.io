---
layout: default
title:  "Trying out Fedora SilverBlue"
date:   2024-05-12 20:42:12 +0200
categories: linux fedora
published: true
---
I recently decided to try out [Fedora Silverblue](https://fedoraproject.org/atomic-desktops/silverblue/) as my main Linux desktop OS. I had been a happy Fedora Core user for a couple of years and now wanted to learn what all the fuzz about Silverblue and friends is about.

## Why Silverblue?

Whatever level of expertise you're dabbling with Linux - you very likely have encountered situations where you misconfigured something and suddenly your system was broken. This can be a broken update or -- more likely in my case -- you just fatfingered a config file that you didn't really understand. Silverblue promises to keep your system safe:

- The root file system is read-only, meaning you cannot overwrite global system configurations. Everything you configure happens locally to your user.
- System updates are atomic, meaning you either install a new snapshot of a working system or you can rollback to the previous working system state.
- Besides that, you can still run everything you want in your local user context:
  - GUI applications are run via [FlatPak](https://www.flatpak.org)
  - For other environments, Fedora provides [Toolbox](https://docs.fedoraproject.org/en-US/fedora-silverblue/toolbox/), a container-based distribution variant that you can spawn and (mis)configure to whatever you like.

## Atomic System

As we'll see later, most of your work is going to happen in Flatpak-based applications or a Toolbox container anyway. Nevertheless, it turns out I did want a few more applications in my base system than were installed from start. Silverblue's atomic system images are managed using [`rpm-ostree`](https://coreos.github.io/rpm-ostree/), which in the context of the system image takes the place of `dnf`.

This is also where the atomic system needs some work from your end -- installing a package will only add it to your new base image. To actually make use of these packages, you'll have to reboot your system (remember, Silverblue is trying to keep your system in a working state and will switch over to the new image atomically.) There's a shortcut with `rpm-ostree --apply-live`, but I didn't use that so far.

So what did I install into the base image?
- [neovim](https://neovim.io) because I want a fancy editor at all times.
- [kitty](https://sw.kovidgoyal.net/kitty/) because I use the command line a lot and this is my go-to terminal emulator these days.
- [NVIDIA drivers and tools](https://docs.fedoraproject.org/en-US/fedora-silverblue/troubleshooting/#_using_nvidia_drivers) to make sure I get graphics acceleration for my gaming (tm) needs.

## Flatpak Applications

Generally, desktop applications are managed via Flatpaks in Silverblue. From a user perspective this doesn't feel much different - you open the 'Software' application, search for what you need, and install it. The fact that this is now no longer an RPM being installed into your base system, but a Flatpak installed in your user context is hidden. The look and feel of Fedora's GNOME desktop remains the same as for a regular Fedora Core installation.

I was able to install everything I usually need for day-to-day work:
- the [Vivaldi](https://vivaldi.com) web browser
- [Obsidian](https://obsidian.md) for note taking
- the [Steam](https://steamcommunity.com) client for gaming

Only installing PrusaSlicer was impossible via Flathub. They only [ship this as an AppImage](https://github.com/prusa3d/PrusaSlicer/releases) for Linux. Which of course, still works in Silverblue.

## Toolbox

Silverblue discourages you from installing everything into the base image. The way to go instead is to put your work environment into containers. Silverblue delivers `toolbox` to manage these containers.

Once created (`toolbox create`), you can at any time enter the container with `toolbox enter` and then use this environment as if it were a regular Fedora Core installation. You install packages with `dnf`, modify the whole system to your liking, and if you screw up, you just start from scratch with a new container. In fact, toolbox does not even restrict you to Fedora Core at a specific version. You can as well create containers mirroring other distributions (they support Arch Linux, Fedora, RHEL, and Ubuntu at this point) as well as any specific release of these distributions.

So far I have only created a single toolbox mirroring FC40 on my host, and it has worked well so far. I sense that support for different distros and versions opens up new opportunities (testing your new software on different distros right here, right now!). And likely there's also opportunity in creating toolboxes for different usage scenarios. Say I'd build one for developing software, one for fuzzing kernels, and one for stuff related to 3D printing. Then again, this might also become another container management bottleneck.

## Things I didn't get to work

- I was so far unable to convince the system to use Vivaldi as my default browser. I have the window open and everything works fine. But when one of the system applications tries to launch a browser, I always still end up in Silverblue's default Firefox browser. 
