This will soon be replaced by updated Gentoo Embedded documentation. This page remains because it is a useful resource for cross-compiling a stage3-like system.

This is our first tutorial, and perhaps the most fundamental to our project. It describes how to get the necessary tools installed and prepared, how to build the software, and then package that software into a working, native Gentoo image for ARM devices.

Prerequisites
=============
More than anything else, this is like installing a fresh Gentoo system. The only difference is now you don't have a Handbook to tell you in what order to build your system in, or provide any recommendations in terms of USE flags and packages.

* Thus, it is important that you've already had experience installing Gentoo.
* You should already have an idea of what you want in terms of packages and USE flags.
* You should be able to understand the variety of error messages that emerge might spit out. Debugging compilation errors doesn't have to be a piece of cake, but you should be able to find out what the errors mean and how to resolve them (whether that be patching or reporting a bug) on your own.

Table of Contents
=================
It is recommended you follow the instructions in the following pages, in this order:
* [Create Cross-Compiler Environment](Create-Cross-Compiler-Environment.md)
* [Installing Crossdev Tools](Installing-Crossdev-Tools.md)
* [Building Packages](Building-Packages.md)
* [Packaging and Releasing](Packaging-and-Releasing.md)