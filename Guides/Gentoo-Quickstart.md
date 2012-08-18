This page is a work in progress!

Why Gentoo
==========
We highly recommend Gentoo for these reasons:

* Gentoo is technically not a distribution. It's a meta-distribution, flexible by design, which means you get to build your own distribution. You pick what software you want, you install it, you set it up, and you configure it. Do the words "pulseaudio" and "networkmanager" make you twitch? Don't bother with them! If you don't like our pre-built images, make your own! All the tools are there for you to use, and we're here to help you use them!
* Gentoo isn't always the fastest, even if every binary is optimized for your system, but it's certainly the cleanest. Like we said before, nothing gets on your system without your consent, unlike most binary distributions. On every stable release, we detail exactly how the image was built, and what was installed. No more hidden surprises or unnecessary daemons running around.
* Bugs are easier to find and fix with Gentoo. The source code the developers themselves wrote is downloaded straight to your very own system and compiled before your eyes. Even if you don't know any code, you can still patch up a buggy package with a quick fix you found on Google and try it without waiting for a binary package guru to do the work for you. In addition, the logs generated while the source code was compiling are detailed enough to report almost any bug with very little back-and-forth dialog with developers.
* Gentoo practices "rolling" updates, which means you get the latest of any software, after they've been tested for at least 30 days by other users. What does "rolling updates" mean? Well, big new versions of most distributions are released every six months or so, and require users to perform massive upgrades. This is because binaries (in any operating system) are inherently brittle. They need to co-exist with specific versions of other software. Gentoo has the advantage of source code, where everything is built according to what's installed on ''your'' system. There's no such thing as "Gentoo v1." There's just "Gentoo," and you get to pick what versions of any software are installed, which is usually the latest stable.
* It's an excellent learning experience. It's pretty much guaranteed that you will know something you didn't know before by the time you've configured your desktop environment of choice, whether that be a CLI or GUI. Learning might not be on the top of your list of things to do, but you'll realize that, because Gentoo is largely vanilla, the lessons you learn here can be applied elsewhere. The time you put into Gentoo really pays off.

Why Not Gentoo
--------------
Here are the negatives:

* Having everything compiled on your system provides many advantages. However, compiling from source requires extra software most binary distribution users never have to deal with, such as a complete toolchain, which includes a compiler like gcc, a linker like ld, and other utilities like make and autotools. Compiling from source not only requires extra storage space, but quite a bit of RAM too. This is not a big deal, generally, since most packages are fine with "just" a few MB's, but some of the bigger ones like OpenOffice can require as much as 512MB of RAM!
* With power comes responsibility. Your system will break, and since you are the ruler of your domain, it's your responsibility to fix those wheels when they stop turning. Thankfully, most, if not all, issues can often be resolved within a matter of minutes of very simple Google research. Because there's always a Gentoo user running unstable versions, there'll usually be someone who has run into your bug before. Entire debates occur in the gentoo-dev mailing list about how best to work around a known bump in the road coming ahead, and new utilities are sometimes written specifically for those. Developers are also very responsive to bug reports, especially since the entire system of stabilization in Gentoo revolves around them. In short, you will run into problems, and you must be ready to deal with them. For some, even with the help, it's still too much.

But, Wait...
------------
We're going to do our best to eliminate those negatives, particularly where they're exaggerated by the embedded hardware. Here's how we're hoping to take Gentoo up another level:

* Rather than doing any compiling, users can pull packages from a binary repository. We're looking into how to allow for multiple USE flags at a time. We'll keep you posted! Until then, we're going to try to set reasonable defaults.
* We're planning a feature where, if a package fails to compile, the system will list all known bugs associated with the failed package. Potentially, there could be an opt-in unofficial community feedback system, where people can post their failures (semi-)automatically, and then community members can help "resolve" them. If the community agrees it is a bug, it can be forwarded to the bugs.gentoo.org system.

Why Not Gentoo, Still
---------------------
Gentoo will never be totally painless, no matter what you do. Since everything is in a human's control, the human will always be required to think and solve. For some, a "Just Works" distribution like Ubuntu, which guarantees stability and polish at a heavy price to flexibility and size, meets their needs better.

For the rest of you who are interested, read on!

Gentoo Fundamentals
===================
We're paraphrasing a lot of what's in the [Gentoo Handbook](http://www.gentoo.org/doc/en/handbook/). Read it for more in-depth information on a generic Gentoo installation.

Portage
-------
At its core, Gentoo is about building, and assisting those who are building, an operating system to each individual's needs. Building a distribution by hand is a daunting task, so a package manager, called portage, was written to assist users.

Here's how it works: if, say, you want to install your favorite instant messenger, all you do is run this in a terminal:
	emerge pidgin

Portage will check to be sure pidgin's dependencies are installed with the features pidgin needs, download pidgin's source code from either a Gentoo mirror or Pidgin's own website, patch it with any known fixes, compile the source code, verify pidgin won't overwrite any other package's files, and then install pidgin. After installation, portage might give you information about pidgin, and any little quirks to be aware of.

A feature portage is known for is the USE flag. By turning one on or off, you are enabling or disabling a feature in a package. Using the pidgin example again, you can turn on and off support for the Gadu-Gadu protocol by enabling or disabling the gadu USE flag. USE flags can be defined for the entire system or for individual packages. Some, like gadu, are only used by one or two packages, and make more sense to be defined at the package level. It's usually better to turn as little on as possible until you are sure your entire system needs, for example, Gadu-Gadu support, for whatever reason.

Portage's responsibility is not just limited to installing packages, but maintaining them as well. This little beauty of a command will update the software you specifically requested:
	emerge -uDNav world system
Let's explain that long string of arguments first:
* -u means "update". This tells portage to only install a package if there is an update. If you leave this out, it will reinstall a package, even if there is no update.
* -D means "deep". Portage will not only check for updates in world and system, but will scan updates for any packages depended on by world and system. This is what actually lets portage check for updates on all packages.
* -N means "newuse". If what happens when a USE flag is enabled at all changes for a package, this "newuse" argument will allow portage to reinstall the package and let the updated USE flag do its thing.
* -a means "ask". Without this, portage will assume you don't want to double-check the list of packages about to be installed. It's a good habit to check that list, because sometimes something catches your eye, like a USE flag getting turned off that you actually might want.
* -v means "verbose". This is totally optional, but since most Gentoo users like to observe their system, the extra bit of information that comes from the "verbose" argument is satisfying.

What does "world" and "system" mean? These are called package sets, and are responsible for maintaining a list of certain packages for later reference. Currently, there are only a few built-in sets, but future portage versions will allow users to define their own. The world and system sets are special. The world set maintains a list of packages you specifically asked to be installed. You can view and edit the list of packages in this set in the file /var/lib/portage/world. The system set is defined by a portage profile (which we will explain in a moment). Packages in the system set are necessary for your system to function. These are pre-defined by developers, and shouldn't be edited by users. Some of the system packages are the kernel, the compiler, and portage itself.

If you want to install a package but don't want it in the world set, simply use the "-1" argument.

So, essentially, that command says, "Install all packages in the world and system set, and install all packages required by those sets, but only if there are new versions or updated USE flags. Ask me first before installing, and tell me more about what's going on."

If you are curious about how else the "emerge" command can serve you, read the emerge man page.

Installation (Or, Where the Stage3 Comes From)
----------------------------------------------
This section is not essential, but hopefully it is at least interesting.

When you download a new version of some other Linux distribution, like Ubuntu or OpenSUSE, you are downloading what Gentoo folks call a stage4. When you download Gentoo for installation, you are downloading a stage3. What are these stages? Unfortunately, the authors of this wiki page don't know all the details, but they do understand the basics.

In order to build a new pre-built installation of Gentoo for you to download, Gentoo developers first start with a previous clean stage3 installation, and inside they build a new stage1, which is a very basic version of the stage3. They then use the stage1 and build a stage2 with a few more added things. Then, from the stage2, they go all out and build all the packages in the system set, and package the results as a stage3. Some people, like us, and most binary distributions like Ubuntu and OpenSUSE, take it a step further and add more packages to the stage3, such as a desktop environment like GNOME,  KDE, or LXDE. The result is then repackaged as a stage4.

When you decide to install Gentoo or Neuvoo, you actually have two options. You can download a stage3 or stage4, and build your system up from there, or you can build a stage3 or stage4 first. Since 99% of all Gentoo users will end up building the exact same stages, regardless of whether they want a lean installation or not, your best bet is to go with a pre-built stage first and see if you like it. However, it's good to know your options in case you decide they don't meet your requirements.

If you want to build your own stages, see [Create Image](Create-Image.md).

We're not going to cover Gentoo installation here. If you want to install Neuvoo, see the [Install Pre-Built Image](Install-Pre-Built-Image.md) page. If you want to install "vanilla" Gentoo, without any Neuvoo features, see [Gentoo's Handbook](http://www.gentoo.org/doc/en/handbook/).

Configuration
-------------
Gentoo configuration can be broken into two broad categories: portage configuration, found mostly in /etc/portage/, and system configuration, found in /etc/conf.d/.

TODO: explain each, one at a time.

Standard Gentoo Installation
============================

Gentoo Toolbox
==============

Install a Program
-----------------

Simple install.
	emerge samba

Simple Install: ask, be verbose. This is recommmended. This allows you to view more detailed package information and USE flags that will be used.
	emerge -av chromium

Simple Install: ask, be verbose, use remote binary only. This requires that you have PORT_BINHOST="/path/to/url/of/binrepo" set in /etc/make.conf
	emerge -avG chromium

Simple Install: ask, be verbose, use local binary if available, otherwise make binary. This requires that you have PKGDIR="/path/to/local/directory" set in /etc/make.conf
	emerge -avkb chromium

Advanced Install: specific, masked package, one time only.
	ACCEPT_KEYWORDS="~arm" emerge -av1 =sys-apps/baselayout-2.0.1

Advanced Install: no keywords, additional use flag.
	ACCEPT_KEYWORDS="**" USE="connection-sharing" emerge -av networkmanager

Uninstall a Program
-------------------
Simple uninstall.
	emerge --depclean mozilla-firefox

Simple uninstall: ask, be verbose. Recommended, this allows you to view more detailed package information.
	emerge --depclean av gnome

Advanced Uninstall: specific package.
	emerge --depclean -av =sys-apps/baselayout-2.0.1

Update Package Listing
----------------------
Resync Portage.
	emerge --sync

Resync Layman overlays.
	layman -S

System Updates
--------------
Simple: Update system packages.
	emerge -av @system

Update system and world packages.
	emerge -uDNav @world

Advanced: Update system and world packages, use remote binaries only.
	emerge -uDNavG @system @world