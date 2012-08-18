The following guidelines define how information will fit into our wiki.

The guidelines will describe what content should fit in that kind of page, while any linked copy-and-paste template(s) explain how that content fits on the page.

Common Guidelines
=================
* Any page which is still a work in progress, or obsolete but kept for archival purposes, must say so at the top page.

Generic Page Types
==================
* A page is simply informing the user of information or facts. They rarely provide instructions.

* The HOWTO is a series of instructions.
** If it is extensive, it can be split into sub-pages.
** At the beginning, it should always include a description about its goals and the end result. It should always have prerequisites that alert the user to what they should already have for hardware, software, or knowledge before they continue. It and its subpages, if they exist, can be referred to from any other page without confusion.

Release Page
============
* Release pages describe a particular release of Gentoo for Pandora. It has the following details:
** Notes on what that release is all about
** Toolchain used
** Repository information
** A link to or a list of known bugs for this image
** Links to variants

* Each variant page is structured like a release page. The differences are:
** A list of packages installed, each with the name, version, and USE flags. Preferably, dependency information should be included.
** The make.conf file included with the variant.
** A list of changes made to the image beyond simply emerging the above packages, each with a patch file if applicable, and a reason if it is not immediately obvious.
** There should be no list of variants, since this IS a variant.
** This variant should also link back to the image(s) it was based off of, if any.