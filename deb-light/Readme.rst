MythTV Light Debian & Ubuntu Packaging (.deb)
=============================================
This is an alternative package for Debian & Ubuntu systems.
It is also used for the "MythTV Light" packages for Raspberry Pi.

Differences
-----------
MythTV Light creates one package for MythTV and one package
for the plugins, rather than multiple packages as generated by
the main package setup. If you are building MythTV for your
own backends and frontends you can copy a single file to each
frontend and backend machine and install it.

MythTV Light creates a package of only the MythTV software. It
does not install a database or setup any services, automatic
backups, etc.

MythTV Light includes a suite of tools to support building and
testing of multiple versions of MythTV on one computer. You can
also test the built software before creating a package.

make vs cmake
-------------
MythTV historically was built using a Makefile in the mythtv and mythplugins
directories. There is now a cmake option, which will be preferred in future.
The deb-light build process caters for both methods and defaults to cmake.
Note the cmake option has not been tested on Raspberry Pi, so if you
encounter problems, use the make option.

Setup for building
------------------
Set up your system this way.

- Clone the github MythTV/mythtv repository into a directory of your
  choice. The packages will be placed in the directory above the
  clone. For example if you
  clone into $HOME/project, git will create $HOME/project/mythtv and the
  packages will be placed in $HOME/project. Other files and directories
  will also be created in $HOME/project.
- Place the contents of github MythTV/packaging/deb-light repository into
  any directory.
- You can put the deb-light directory on your
  path to avoid entering it every time.
- Make sure you have the build dependencies installed. You can use the
  ansible repository to install them. See github MythTV/ansible.
- If you are using cmake (which is the default), make sure you install the
  cmake prerequisites. See README.CMake.md in  the mythtv repository.

Options
.......
Options can be set in the $HOME/.buildrc file. Create the file if it does not
exist. Specify the build method (make or cmake) by adding a line as follows.
The options are make or cmake. The default is cmake:

    BUILD_METHOD=cmake

make Options
^^^^^^^^^^^^

If you want to override configuration options, you can supply them by
adding a MYTHTV_CONFIG_OPT_EXTRA line to $HOME/.buildrc.

  MYTHTV_CONFIG_OPT_EXTRA="--enable-opengl-video --enable-opengl"

cmake Options
^^^^^^^^^^^^^

Add options applicable to the cmake command, for example:

  MYTHTV_CONFIG_OPT_EXTRA="-DENABLE_<FEATURE>=OFF"

Change the preset. Default is qt5. For other options run cmake --list-presets
in the top level mythtv directory:

    BUILD_PRESET=qt5

Change the build type. Default is Debug. For other options see README.CMake.md
in the mythtv repository:

    BUILD_TYPE=Debug

Building the packages
---------------------
Follow these steps to build the packages.

mythtv-light
............

- To build the mythtv-light package, run the following. <dir1> is the place
  you cloned the mythtv repository and <dir2> is the place the contents
  of packaging/deb-light are located::

    cd <dir1>/mythtv/mythtv
    /<dir2>/build_package.sh

- The first time you run this it will prompt for a build destination,
  defaulting to $HOME/proj/build.
- Once the build is complete, the package will be in <dir1>. Also build
  logs and a directory containing the package data will be there. This
  directory is needed for the plugins build.

mythplugins-light
.................

- First build the mythtv-light package as described above.
- Run the same way::

    cd <dir1>/mythtv/mythplugins
    /<dir2>/build_package.sh

- The package will be in <dir1>.

Developing and Testing
======================

The build_package.sh script runs 4 scripts. If you are developing or testing
you can run them individually. This allows testing of multiple versions
of MythTV on one system at the same time.

config.sh
---------
With cmake builds, the config script does not run MythTV configure. It
configures cmake for a build,

This script works from the mythtv or the mythplugins directory. It first
cleans all old build data. It does not interfere with any source code
changes or patches you may have made. It then runs configure with the
normal options.

Before running config on mythplugins make sure you have already run
config build and install on mythtv because the plugins need that.

config on mythplugins clears out mythtv build data as well as mythplugins
build data. It also makes a copy of the mythtv install to run the build
against.

build.sh
--------
After running config.sh, build.sh performs the make against the source.
If building with cmake (the default), it does the MythTV configure, build, and install as described below.

install
-------
The install into the build directory is done by the build.sh script
when using cmake. Whan using make, it is done by running install.sh.

The script installs the build into the directory selected for test builds
the first time you run. It creates a hierarchy of directories for
each environment and branch, so that you can have several branches and
environments installed at once. It creates a separate directory for patched
versions from github clean versions so that you can test a before and after
scenario.

For mythtv it creates a directory called mythtv with subdirectories for
each branch. For plugins it creates a directory called mythplugins with
subdirectories. The mythplugins subdirectories contain both the mythtv and
the plugins so that you can test mythtv with the plugins.

Versions that have git differences (dirty), are put in a directory with "-tst"
appended to the name. If you want a different extension applied, for example
if you are working on multiple builds, add a "BUILD_DIRTY" line
to $HOME/.buildrc, for example::

  BUILD_DIRTY=xxx

When using cmake build (the default), install.sh does nothing. With cmake,
the -tst or DIRTY option does not work. All builds are put into the same
directory, whether dirty or not. It is not practical to use separate
directories as the dirtectory is set up in config, and you do not want to
run config very often as it causes the build to take a long time. If you 
want to save the commit build, copy or move it to another directory.

package.sh
----------
This script creates a package from the installed build. It will check that
the source version matches the built version.

test.sh
-------
This script uses up a mini environment in which you can test each version
you have built::

  test.sh <shortname> command

For example::

  test.sh mdm mythbackend
  test.sh mdmt gdb mythfrontend

The shortname comes from file $HOME/.buildnames. Open that file and you
will see lines like this::

  shortname=xa4mtvmr longname=xenial-amd64/mythtv/master

You can edit the file and change them to something easier to remember.
For example I use mdm for Mythtv Development Master. So edit the file and
change it to something like this::

  shortname=mdm longname=xenial-amd64/mythtv/master

Then you have to set up a .mythtv directory for each test version. It is
named as $HOME/.mythtv-<shortname>, for example $HOME/.mythtv-mdm. This
will contain details of your test database for that version.

Patched versions have "-tst" in their directory name and I add a t to the
shortname::

  shortname=mdmt longname=xenial-amd64/mythtv/master-tst

If you use the same .mythtv directory for .mythtv-mdm and .mythtv-mdmt
you can create the one as a link to the other::

  ln -s $HOME/.mythtv-mdm $HOME/.mythtv-mdmt

You can create multiple short names for the same version to test different
scenarios::

  shortname=mdmt longname=xenial-amd64/mythtv/master-tst
  shortname=mdmt1 longname=xenial-amd64/mythtv/master-tst

Then create $HOME/.mythtv-mdmt1 directory as a copy of $HOME/.mythtv-mdmt.
Edit $HOME/.mythtv-mdmt1/config.xml and insert a LocalHostName that is different
from your system id::

  <LocalHostName>test1</LocalHostName>

By running as follows you can have two setups, such as different themes,
screen settings or playback profiles::

  test.sh mdmt mythfrontend
  test.sh mdmt1 mythfrontend

