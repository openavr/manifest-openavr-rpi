Repo Manifests for building Raspberry Pi Dev Board Example
==========================================================

This repository provides the glue for fetching all of the git repositories that
are needed for building an image for the Raspberry Pi board.

Getting Started
---------------

1.  Install Google's Repo command.

    Download the Repo script:

        $ curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > repo
        $ chmod a+x repo
        $ sudo mv repo /usr/local/bin/

    **NOTE**: If you are using Ubuntu-18.04 or newer, you can install repo with:

        $ sudo apt-get install repo

2.  Initialize a Repo client.

    Create an empty directory to hold your working files:

        $ cd ${HOME}/${WORK_DIR}
        $ mkdir yocto-rpi
        $ cd yocto-rpi

    **NOTE**: Use whatever you want for `${WORK_DIR}` and/or `yocto-rpi`, just
    adjust what follows accordingly.

    Tell Repo where to find the manifest:

        $ repo init -u git://github.com/openavr/manifest-openavr-rpi -b master

    A successful initialization will end with a message stating that Repo is
    initialized in your working directory. Your client directory should now
    contain a `.repo` directory where files such as the manifest will be kept.

3.  Fetch all the repositories:

        $ repo sync

4.  Initialize the Yocto/OpenEmbedded Environment:

        $ source ./setup-env

    This copies default configuration information into the `build/conf`
    directory and sets up some environment variables for OpenEmbedded.  You may
    wish to edit the configuration options at this point.

5.  Build an image.

    This process downloads several gigabytes of source code and then proceeds to
    do an awful lot of compilation so make sure you have plenty of space (100GB
    minimum).

        $ bitbake core-image-openavr

    If everything goes well, you should have a compressed root filesystem
    tarball as well as kernel and bootloader binaries available in your
    *work/deploy* directory.  If you run into problems, the most likely
    candidate is missing packages.  Check out
    http://www.yoctoproject.org/docs/current/yocto-project-qs/yocto-project-qs.html#resources
    for the list of required packagaes for operating system. Also, take
    a look to be sure your operating system is supported:
    https://wiki.yoctoproject.org/wiki/Distribution_Support

    **TIP**: You can have bitbake pre-download (or fetch) all of the source
    files needed for an image before starting to compile anything with the
    following:

        $ bitbake -k core-image-openavr --runall=fetch

    **TIP**: The first time you do a bitbake build, use the `-k` option to force
    the build to continue doing as much as possible after an error so that you can
    go away and do other work while the initial build is running (it can take a
    while). For example:

        $ bitbake -k core-image-openavr

6.  Build an SDK for cross compiling target applications on an x86 machine.

    Run:

        $ bitbake -c populate_sdk core-image-openavr

    When this completes the sdk is in `./tmp/deploy/sdk/` as an `.sh` file
    you copy to the machine you want to cross compile on and run the file.
    It will default to installing the sdk in `/usr/local`, and you can ask it to
    install anywhere you have write access to.

Installation to SD Card
-----------------------

Writing the wic file to SD card with balena-Etcher doesn't seem to work.

TODO: Need to figure out why Etcher doesn't work.

Need to use `bmaptool`:

    $ apt install bmap-tools
    $ sudo bmaptool copy tmp/deploy/images/raspberrypi4/*.wic.bz2 /dev/sdX

Staying Up to Date
------------------

To pick up the latest changes for all source repositories, run:

    $ repo sync -d

Setup the Yocto/OpenEmbedded environment:

    $ source setup-env

If you forget to setup the environment prior to running bitbake, your system
will complain that it can't find bitbake on the path.  Don't try to install
bitbake using a package manager, just run the command above as bitbake will
be installed by repo.

You can then rebuild as before:

    $ bitbake core-image-openavr

Starting Fresh
--------------

So it is borked.  You're not really sure why.  But it doesn't work any more.

There are several degrees of *starting fresh* (from least to most dire).

1. clean a package:

        $ bitbake <package-name> -c clean

2. clean a package (removes sstate):

        $ bitbake <package-name> -c cleansstate

3. clean a package (removes sstate and downloads):

        $ bitbake <package-name> -c cleanall

References
----------

* https://layers.openembedded.org
* https://www.yoctoproject.org/documentation
* https://source.android.com/setup/using-repo
