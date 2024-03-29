In this page, we'll discuss how to successfully build all the packages you need to get your image started. What packages, which USE flags, and at what versions are entirely up to you. If you need some inspiration, we'll show you specifically how you can use our documentation as a starting point.

Variables
=========
Scattered throughout this page, we use some variables to quickly refer to directories set up in previous pages of this tutorial. We use the $NZ variable to refer to the directory where our compiled environment is stored. Our default location is this, but you can change it to whatever you want:
	export NZ="/usr/local/neuvoo"

We assume that you are using $CHOST="armv7a-unknown-linux-gnueabi". If not, then simply use your own $CHOST and emerge will be in the format of emerge-CHOST. Do NOT set CHOST in your environment. CHOST in our examples below are simply placeholders.

We also assume that you are installing your ROOT of your image somewhere other than /usr/$CHOST. We are using $NZ as our ROOT. This is entirely up to you.

Building
========
Lets start building our image!
	ROOT=$NZ USE="build" emerge-${CHOST} -1 --nodeps baselayout
	ROOT=$NZ emerge-${CHOST} system

Now you should have a cross-compiled stage3-like system in $NZ. Build any additional packages, like dhcpcd, that you may want.

Kernel
======
The following commands should have INSTALL_MOD_PATH="/some/location" added the beginning to change where the /lib directory is created for modules to be installed. You can install this into either the compiled environment in $NZ or save it in a tarball along with the kernel image.

First, install a source tree
	ROOT=$NZ emerge-${CHOST} gentoo-sources

Configure the kernel. We begin with the OMAP3 Beagle configuration and then tweak it from there. You can find additional configuration options under arch/<arch, like arm>/configs
	make ARCH="arm" CROSS_COMPILE="${CHOST}-" omap3_beagle_defconfig
	make ARCH="arm" CROSS_COMPILE="${CHOST}-" menuconfig

Build the kernel (everything)
	make ARCH="arm" CROSS_COMPILE="${CHOST}-"
or just do a
	make ARCH="arm" CROSS_COMPILE="${CHOST}-" uImage
	make ARCH="arm" CROSS_COMPILE="${CHOST}-" modules
then install the modules to INSTALL_MOD_PATH (root of your image)
	make ARCH="arm" INSTALL_MOD_PATH="/some/location" CROSS_COMPILE="${CHOST}-" modules_install

U-Boot
------
Install u-boot-tools. You'll need these to build a uImage.
	emerge -av u-boot-tools

Build the uImage.
	make ARCH="arm" CROSS_COMPILE="${CHOST}-" uImage

You'll find uImage in arch/<arch of your choice>/boot/uImage.

Common Problems
===============
Most, if not all, of the issues can be categorized into a particular type of problem. We've provided some pointers on how to solve each problem.

However, there are undoubtedly many more specific issues out there. Not all software are currently set up to cross-compile correctly, so you will undoubtedly run into more than one trouble packages along the way. If you run into something that is not discussed here, please visit our bug tracker.

cross-emerge <package> won't compile due to ...missing keywords
---------------------------------------------------------------
You may think you are smart by running
	ACCEPT_KEYWORDS="arm ~arm" emerge-${CHOST} <package>
or
	echo "<package> ~arm" >> $SYSROOT/etc/package.keywords
but it still won't work. Well, you're not dumb. The problem is that portage will still check against the cross-compiler environment even if $ROOT points to the compiled environment.

Why? "Stable" versions of portage currently have a [bug](http://bugs.gentoo.org/222895) in which portage will want that package unmasked and installed in the cross-compiler environment, as well as unmasked in the toolchain environment and installed in the compiled environment.

Versions of portage (such as portage-2.2_rc30) that have the --rootdeps option no longer have this issue. For everyone else, you'll want to make sure your cross-compiler environment is a chroot, and then unmask in the cross-compiler and toolchain environment the same version you're trying to install into the compiled environment.

*.so: file not recognized: File format not recognized
-----------------------------------------------------
This is problem is usualy caused by one of two things: A, a *.la file is pointing to the build system target, or B, autotools is calling on build system *.so libraries directly.  To fix this, first try using dev-util/lafilefixer:
	emerge lafilefixer
	lafilefixer /

If that doesn't work, you may have to temporarily move the offending *.so symlinks.  As always, make sure you keep backups.  Also, don't move the *.so.1 symlinks or other such numbered files, as those are usually needed at runtime by many build-system binaries.  In some cases you may need to move them, however, so pay attention to your log files.

Worst-Case Scenario
-------------------
We mentioned we'd only write about common problems you will run into. You should be feeling shivers run down your spine when you realize we wrote a section titled "Worst-Case Scenario".

Some packages simply will not cross-compile. At all. Period. As of this writing, gcc is one of those packages. How in the world, then, are you going to work around packages like gcc, which are required by packages essential for system operation, like emerge?

The answer isn't pretty, but it works. Download a binary package that meets your requirements as best as possible, emerge it, and continue with your building. Later, when you're running the image natively, rebuild that package. In a sentence, use our binary packages to bootstrap, and then recompile natively later to get a cleaner version.

There's another ugly answer, and that's to compile the required software by hand, install it by hand, and then list the package name with its version in $ROOT/etc/portage/package.provided. This is very difficult and can get very messy, so use it as a last resort. (For example: since you were the one who copied the files into place, portage will not know what package those files go with. Thus, if you try to emerge that package later, portage will complain of file collisions.)

Starting Where Official Pre-built Images Left Off
=================================================
You're already doing everything from scratch and would like to have one less thing to worry about, like which packages and USE flags to use. Or, you like a variant, but you want to change some fundamental components. You can use one of the pre-built images as a guide. The entire reason there's so much documentation for a variant and release, is so that people can create their own variant with similar features and configurations!

So, what are the parts to pay attention to?

An optional section, but perhaps the most useful, is "Build Instructions". This describes what exactly was executed to create the image that's available for download. Using these instructions may feel like cheating, but it may be good to try it just to see where you end up, and try and tweak it from there.

The "Package Listing" section is another very useful section. You can see what packages were installed, and with what USE flags.

Lastly, the "Configuration Changes and Other Customizations" section will be very handy in determining what changes you want to make beyond simply installing packages, especially in the /etc directory.

If you make any changes that you think should be included in the official builds, do not hesitate to let us know!