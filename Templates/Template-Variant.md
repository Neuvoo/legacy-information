A short description of the features this variant brings to the table.

If it was based off of another image, it should specify so here as well.

Status
======
A very simple description of where we are at. Could be "in early development", "testing images", "Adding finishing touches", and so on. If a release date is known, it could be included here.

Standards
=========
This image follows the following common Neuvoo development standards, which are:

* It boots!
* It is fully reproducible using documentation on this page.
* Never, ever edit make.conf for profile purposes
* All bugs are documented and reported upstream or in Gentoo's bugzilla prior to release.

Download
========
See [Install Pre-Built Image](Install-Pre-Built-Image.md) for instructions on installing this image. (This line can be replaced, as long as alternative instructions are provided.)

* version: [0.0.0-beagle](http://host.org/image.tar.bz2) / root password: rootpassword

Roadmap
=======
* A bullet point list of features could go here.
* A link to a list of bug reports targeted towards this release could also go here.

Specifications
==============
(At least the following specifications are required, but more can be added.)
* Toolchain: xxxx-xxxx-xxxx-xxxx
** cc: gcc-xxxx
** libc: glibc-xxxx
** binutils: binutils-xxxx
** headers: linux-headers-xxxx
* Binary repository: link here
* Bugs: link here
** ...and/or others listed here in separate bullet points if they aren't already mentioned.

Package Listing
===============
A big list of packages and their versions and USE flags here.

Included make.conf
==================
{{File|/etc/make.conf|<pre>
The contents of the make.conf included with the variant.
</pre>}}

Configuration Changes and Other Customizations
==============================================
A description or list of patches that properly describe any changes made beyond the defaults included with each installed package.

Build Instructions
==================
This section is optional. It describes exactly what steps or commands were used to build the variant.

Further Reading
===============
For information on how to install an image, see [Install Pre-Built Image](Install-Pre-Built-Image.md).

For information on building your own images, see [Create Image](Create-Image.md).

(Include other links here.)