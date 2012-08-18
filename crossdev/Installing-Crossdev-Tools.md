This page will tell you how to properly install and setup crossdev in your environment for cross-compiling.

Prerequisites
=============
* It is recommended that you follow the [[Hand-craft an Image/Create Cross-Compiler Environment]] Tutorial prior to continuing, though not required. 

**WARNING**: The way Gentoo crossdev works now, you will be installing software in both your native environment and cross-compiling environment, which may break your native environment. We recommend you set up and use a chrooted environment for image development!

Setting up Portage
==================
Change "x86" in the below commands to whichever architecture you chose when downloading your stage3 tarball.

	mkdir -p /etc/portage/package.keywords/
	echo "sys-devel/crossdev ~x86" >> /etc/portage/package.keywords/crossdev
	emerge -av crossdev
	echo "PORTDIR_OVERLAY=\"/usr/local/portage\"" >> /etc/make.conf
	mkdir -p /usr/local/portage

Compile crossdev
================

Build Toolchain
---------------
Next you'll want to install your cross-compiler. Gentoo has a really nice tool called crossdev that will run through all the required steps for you.

We show "armv7a-unknown-linux-gnueabi" as an example toolchain here. Please look to the crossdev handbook for information on how to select a different toolchain.

If you just run...
	crossdev -t armv7a-unknown-linux-gnueabi
...it will use all the latest, unstable software. You should probably specify specific, more stable versions, like so:
	crossdev --g <gcc version> --l <(g)libc version> --b <binutils version> --k <kernel headers version> -t armv7a-unknown-linux-gnueabi
If you are trying to cross-compile for an existing pre-built image, you must match the toolchain versions that the image was built with.

You might also consider throwing in "-P -v" before the -t option, so you can see how exactly crossdev is configuring each package:
	crossdev --g <gcc version> --l <(g)libc version> --b <binutils version> --k <kernel headers version></nowiki> '''-P -v''' <nowiki>-t armv7a-unknown-linux-gnueabi

Once crossdev is done building, you might consider packaging up the binaries so you don't have to rebuild them in the future:
	quickpkg --include-unmodified-config=y cross-armv7a-unknown-linux-gnueabi/gcc
	quickpkg --include-unmodified-config=y cross-armv7a-unknown-linux-gnueabi/glibc
	quickpkg --include-unmodified-config=y cross-armv7a-unknown-linux-gnueabi/binutils
	quickpkg --include-unmodified-config=y cross-armv7a-unknown-linux-gnueabi/linux-headers

This command will initialize the toolchain environment with some basic settings:
	emerge-wrapper --init

Switch to Cross-Compiler Toolchain
----------------------------------
You do not have to switch your active toolchains to use the cross-compiled toolchain.  cross-emerge will do that for you.  This is only useful if you have multiple cross-compiled toolchains to select from.
	gcc-config -l
	binutils-config -l

	gcc-config $CTARGET-<gcc version>
	binutils-config $CTARGET-<binutils version>

Customizing Environment for Crossdev
====================================
Now that you have installed your toolchain it is now time to establish the proper environment variables and configs for your building process to work correctly.

Setting Up Toolchain Environment
--------------------------------
Now that you have your variables set you now need to tell crossdev how you want to build your target environment.

We like to organize our package.* rules into directories, as shown below, so that the rules remain organized.  In the below example 
$CTARGET="armv7a-softfloat-linux-gnueabi". Replace $CTARGET with your own toolchain.

While we use CTARGET as a placemarker here, it is for example purposes only. Do not set CTARGET in your environment.

	mkdir -p /usr/$CTARGET/etc/portage/package.keywords
	mkdir -p /usr/$CTARGET/etc/portage/package.unmask
	mkdir -p /usr/$CTARGET/etc/portage/package.mask
	mkdir -p /usr/$CTARGET/etc/portage/package.use

The following is an excerpt from the make.conf used by us with the required variables set. It is not intended to be copy-pasted, but use it as a reference for which variables are needed. Crossdev provides a default make.conf which is almost sane.

	ARCH="arm"
	CHOST="armv7a-softfloat-linux-gnueabi"
	CFLAGS="-march=armv7-a -mtune=cortex-a8 -mfpu=neon -mfloat-abi=softfp -fomit-frame-pointer -Os -pipe"
	ROOT="/usr/${CHOST}"
	CXXFLAGS="${CFLAGS}"
	LDFLAGS="-L${ROOT}usr/lib -L${ROOT}lib"
	E_MACHINE=EM_ARM
	ACCEPT_KEYWORDS="arm"

All CFLAGS listed are not required, but they provide optimizations that nearly everyone will include. For more complete make.conf files that can be copied around, please see our [Pre-built Images](../images.md).

Establish /usr/$CTARGET/etc/make.profile
		ln -s /usr/portage/profiles/path/to/your/profile etc/make.profile

emerge-${CTARGET}
=================
Congratulations you have now built your chrooted crossdev environment and you should now be able to start cross-compiling your own packages by using the following command...
		emerge-${CTARGET} <package-name>

Further Reading
===============
You're pretty much on your own from here, in terms of how to create the image. You have your toolbox; now you need to use it. In the [Building Packages](building-packages.md) page, we'll discuss how to use these tools properly, and what to do if you run into different kinds of problems.

If you're trying to duplicate what we've done with our images to some degree, and just tweak it in different ways, you'll want to browse the [Images](../images.md) page. In [Building Packages](building-packages.md), we will also show you how to replicate our work using the documentation we've given.