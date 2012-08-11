== Usage ==
Farmerge should usually be executed by '''root'''.

Usage can be seen by running "./farmerge.sh --help" or by running --help on any module ("./farmerge.sh --device <device> -- <module> --help").

A typical usage is given below as an example:

"Let's update the repository:"
{{Command|<nowiki>./farmerge.sh --device awesomedevice -- emerge --cpv world</nowiki>}}
"Whoops, someone's got a stale lock in there from a crash a while ago. I'm going to clear that lock:"
{{Command|<nowiki>./farmerge.sh --device awesomedevice -- mirror-lock --mirror-action unlock</nowiki>}}
"Blasted, now there are masked packages! Time to edit /etc/portage inside the chroot:"
{{Command|<nowiki>./farmerge.sh --device awesomedevice -- emerge --emerge-action shell</nowiki>}}
"Alright, now that world is done, let's see what's still not up-to-date in system:"
{{Command|<nowiki>./farmerge.sh --device awesomedevice -- emerge --cpv system</nowiki>}}

== Setting up ==
Get the latest farmerge git from the Neuvoo software repository:
{{Command|<nowiki>git clone git://gitorious.org/neuvoo/software.git software</nowiki>}}
You can keep your copy of farmerge up-to-date by running "git pull" periodically. Be careful not to update while farmerge is running, or farmerge will behave strangely.

In the software/farmerge directory, there is a conf/global.config.default. Copy this to conf/global.config:
{{Command|<nowiki>cp conf/global.config.default conf/global.config</nowiki>}}

Now open conf/global.config. Here's where setting up gets interesting. This configuration file is sourced as a bash file, so you can make the logic as complex or as 
simple as you want. This configuration file must exist, but it can also be empty. If it is empty, you will be required to provide these configuration values at the 
command-line.

The following variables are easy to set up:
* BUILD_DIR needs to point to the directory that contains the target chroot you want to mess with. Use the pre-configured DEVICE variable, the value for which you always 
provide on command-line, to change where that points.
* TIMEZONE_PATH points to the timezone data file in /usr/share/zoneinfo/ for the current machine.

Now we need to configure the ssh paths and URIs. Let's say you are running a server which has a domain called ezpkg.com, and your packages are located under 
ezpkg.com/binaries/. When you are SSH'ing into the server, the files are located at /var/www/ezpkg.com/htdocs/binaries/ on a standard Apache installation. Now, on the 
same server, you also have a "stage", where binaries are temporarily stored until they are all uploaded and ready to go live. They are located in /var/tmp/binaries/.

With that example in mind, the following configuration may make more sense:
* STAGING_SSH_PATH points to the path where the staged binaries are located. In our example, that is /var/tmp/binaries/${DEVICE}.
* STAGING_SSH_URI is the SSH URI used to connect to the server that houses the staged binaries. In our example, that is "user@ezpkg.com".
* MIRROR_SSH_PATH points to the path where the live binaries are located. In our example, that is /var/www/ezpkg.com/htdocs/binaries/${DEVICE}.
* MIRROR_SSH_URI is the SSH URI used to connect to the server that houses the live binaries. In our example, that is "user@ezpkg.com".
* LOCK_SSH_PATH points to the path where the lock files for each binary repository is stored. That could be /var/tmp/binaries/${DEVICE}/.lock/ in our example. 
{{Note|Farmerge's sync options are already set up to ignore directories called ".staging" and ".lock". Use this to your advantage if you want to store staged binaries 
and/or lock files inside of the live binary folders.}}
* LOCK_SSH_URI is the SSH URI used to connect to the server that houses the lock files. In our example, that is "user@ezpkg.com", but it does not have to be.
{{Warning|Presently, STAGING_SSH_URI and MIRROR_SSH_URI must both point to the same machines. The configuration has a similar warning, and provides a configuration line 
that already makes STAGING_SSH_URI the same as MIRROR_SSH_URI.}}

{{Note|If you view usage of any module, you will note that every module's usage can also be filled into a configuration file.}}

=== Mirror Setup ===
There are some one-time steps that must be done to bootstrap a mirror.

To get the mirror started, upload either an empty directory or an already-built binary repo to each MIRROR_SSH_PATH.

Farmerge expects every MIRROR_SSH_PATH mirror to contain a ".portage" directory and a ".world" file.
* .portage overwrites the /etc/portage directory in the target chroot.
* .world overwrites /var/lib/portage/world in the target chroot.
* There is one more file, .timestamp, but that will be initialized properly when farmerge syncs up to the mirror for the first time.

Every chroot is initialized with a stage tarball. Copy the /etc/portage and /var/lib/portage/world out of the tarball that will be used to start farmerge nodes for that 
particular binary repo and put it into the mirror.

=== Farmerge Node Setup ===
Every node needs the same base configuration for global.config we discussed above and the same stage tarball (which really should be the one used to initialize the 
mirror). Farmerge provides a tool to extract a stage tarball properly. You can configure further if you wish:
{{Command|<nowiki>./farmerge.sh --device awesomedevice -- create-chroot --tarball ./awesomedevice-stage5-1970.tar.bz2</nowiki>}}
This will create a directory at the BUILD_DIR you specified in global.config and extract the tarball. If "stage5" is not in the name, farmerge will complain. This is 
mostly for Neuvoo's use, since we require all binary repos be built based on stage5 tarballs. Since most projects don't have stage5's, you can turn this complaint off by 
adding '--tarball-is-stage5 1', or by adding it to the configuration:
{{File|conf/global.config|<pre>
export TARBALL_IS_STAGE5="1"
</pre>}}

Farmerge will handle mounting /dev/pts, /dev, /proc, and /sys. The portage tree is not auto-mounted. You can bind mount this if you wish. Keep in mind that farmerge will 
attempt to sync the tree on nearly every run to maintain consistency with other farmerge build nodes.

== Neuvoo Setup ==
To see how we use farmerge for our purposes specifically, see [[Binary Repositories#Neuvoo Developers]].

== Nitty-Gritty Details ==

=== Modules ===
Each module represents a unit of functionality, or a responsibility, farmerge is expected to perform. More modules can be added very easily if desired. An example module 
called _hello-world is provided for this purpose.

Modules are called with a set of arguments, which may receive defaults via an exported environment. (See module usage for specific variable names to be used.)

Execution of a module ought to be done via the _exec-module module.

=== Libraries ===
Libraries are units of functionality that more than one module wants to perform. For example, argument processing is tedious, so it has been placed in its own library 
called "variable-args".

Libraries are always sourced, and use return codes rather than exit codes to indicate a failure. See per-library documentation included with the source code for specific 
usage information.

=== Invisible Modules ===
Invisible modules are modules that are prefixed with an underscore. They are intended for use only by modules, which is precisely the criteria for a library, but since 
it may be desirable for a user to be able to access this module and its configuration, it remains halfway between a module and a library: an invisible module.
