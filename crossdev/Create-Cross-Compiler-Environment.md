Setting up a cross-compile environment is a necessity. crossdev often requires some package versions to be installed on the host as what is going to be cross-compiled. 
This means that, if you are not operating within a chrooted system, you will be installing all kinds of possibly conflicting and unstable packages to your host system!

With a Gentoo installation inside the chroot environment, you can play around with crossdev to your heart's content without fear of breaking your own system. If anything does break, simply reinstall the chrooted environment.

This guide assumes that you will be using a chrooted environment, but you do not have to.  If you do not wish to use a chrooted environment simply skip this step.

Prerequisites
=============
* Minimal knowledge of chroot, though if you're geeky enough you'll pick this up pretty quickly.
* Any Linux distribution

Preparing the Cross-compiler Environment
========================================
You'll want to pick a directory somewhere to store things such as the chroot and Neuvoo toolkit, if you chose to use it.

For our examples, we'll use the variable $NV whenever we refer to this directory. Here's our recommended location:
	export NV="/usr/local/neuvoo"

Install Environment
===================
The chrooted cross-compile environment is pretty simple. It's just a Gentoo stage3 tarball, inside which you set up crossdev and portage like you would if it was installed to root.  Make sure you get the stage3 compatible with your current system!

First, you need to grab a stage3 tarball from a Gentoo mirror:
* Visit [the Gentoo mirrors page](http://www.gentoo.org/main/en/mirrors2.xml), and pick one.
* Navigate to the releases directory, pick your architecture, and then navigate through the current/stages directories.
* Select a stage3 tarball.

Next, you'll want a snapshot of portage.
* Visit the mirror you previously selected, or [http://www.gentoo.org/main/en/mirrors2.xml a new one].
* Navigate to the snapshots directory.
* Select portage-latest.tar.bz2

Now, create the directory that will house your cross-compile environment. We give a recommended location here:
	mkdir -p $NV/chroot

Extract the stage3 tarball you downloaded to this directory. Replace "x86" with your architecture, and "2008.0" with the version you selected.
	tar -xvjpf stage3-x86-2008.0.tar.bz2 -C $NV/chroot

Extract the portage snapshot into your cross-compile environment:
	tar -xvjf portage-latest.tar.bz2 -C $NV/chroot/usr

Lastly, copy resolv.conf, which contains DNS information. (The -L makes sure cp doesn't copy a symlink, but follows it instead.)
	cp -L /etc/resolv.conf $NV/chroot/etc/resolv.conf
If your DNS information ever changes, you'll want to copy it again.

You no longer need to keep the stage3 and portage snapshot tarballs. However, we recommend you at least keep the stage3 around, as they are a pain to download, and you might find yourself constantly recreating your cross-compile environment after making an environmentally hazardous mistake.

Entering Environment
====================
You will have to prep the cross-compile environment before enter it, to make sure the software is told everything it needs about your system.

First, mount some necessities:
	mount -t proc none $NV/chroot/proc
	mount --rbind /dev $NV/chroot/dev
	mount --rbind /sys $NV/chroot/sys

The cross-compile environment is now ready. Enter it like so:
* Root terminal:
	chroot $NV/chroot/ /bin/bash
* Using sudo:
	sudo -H chroot $NV/chroot/ /bin/bash

Then finally run:
	env-update && source /etc/profile

Consider prepending your chroot shells with "(chroot)" so you don't get confused which shells are in chroot and which aren't. This is a particularly good idea if you use su for root access, since it can be hard to distinguish which root shell is the chroot shell.
	export PS1="(chroot) $PS1"

Initialize Emerge
=================
Somehow the "latest" portage is never actually the latest. Run the following command within chroot to bring everything up to date:
	emerge --sync

If prompted to update the portage software, do so!
	emerge -1v portage

Update Environment
==================
In fact, while you're updating stuff, it's a really good idea to update your new chrooted environment to all the latest software before you do anything else:
	emerge -uNDav @world @system

Further Reading
===============
You've set up your cross-compiler environment, but now you need the actual cross-compiler. Instructions for building a cross-compiler can be found in the [Installing Crossdev Tools](installing-crossdev-tools.md) page.