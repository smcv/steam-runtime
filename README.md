steam-runtime
=============

A binary compatible runtime environment for Steam applications on Linux.

Introduction
------------

This release of the steam-runtime SDK marks a change to a chroot environment used for building apps. A chroot environment is a standalone Linux environment rooted somewhere in your file system.

[https://en.wikipedia.org/wiki/Chroot](https://en.wikipedia.org/wiki/Chroot "")

All processes that run within the root run relative to that rooted environment. It is possible to install a differently versioned distribution within a root, than the native distribution. For example, it is possible to install an Ubuntu 12.04 chroot environment on an Ubuntu 14.04 system. Tools and utilities for building apps can be installed in the root using standard package management tools, since from the tool's perspective it is running in a native Linux environment. This makes it well suited for an SDK environment.

Steam-runtime Repository
------------------------

The Steam-runtime SDK relies on an APT repository that Valve has created that holds the packages contained within the steam-runtime. A single package, steamrt-dev, lists all the steam-runtime development packages (i.e. packages that contain headers and files required to build software with those libraries, and whose names end in -dev) as dependencies. Conceptually, a base chroot environment is created in the traditional way using debootstrap, steamrt-dev is then installed into this, and then a set of commonly used compilers and build tools are installed. It is expected that after this script sets the environment up, developers may want to install other packages / tools they may need into the chroot environment.
If any of these packages contain runtime dependencies, then you will have to make sure to satisfy these yourself, as only the runtime dependencies of the steamrt-dev packages are included in the steam-runtime. 

Installation
------------
All the software that makes up the Steam Runtime is available in both source and binary form in the Steam Runtime repository [https://repo.steampowered.com/steamrt](https://repo.steampowered.com/steamrt "")

Included in this repository are scripts for building local copies of the Steam Runtime for testing and scripts for building Linux chroot environments suitable for building applications.

Testing or shipping with the runtime
------------------------------------

Steam ships with a copy of the Steam Runtime and all Steam Applications
are launched within the runtime environment. For some scenarios, you
may want to test an application with a different build of the runtime.

### Downloading a Steam Runtime

Current and past versions of the Steam Runtime are available from
<https://repo.steampowered.com/steamrt-images-scout/snapshots/>.
Beta builds, newer than the one included with Steam, are sometimes
available from the same location. The versioned directory names correspond
to the `version.txt` found in official Steam Runtime builds, typically
`ubuntu12_32/steam-runtime/version.txt` in a Steam installation.
The file `steam-runtime.tar.xz` in each directory contains the Steam
Runtime. It unpacks into a directory named `steam-runtime/`.

Each directory also contains various other archive and metadata files,
and a `sources/` subdirectory with source code for all the packages that
went into this Steam Runtime release.

### Building your own Steam Runtime variant

For advanced use, you can use the **build-runtime.py** script to build
your own runtime. To get a Steam Runtime in a directory, run a command
like:

    ./build-runtime.py --output=$(pwd)/runtime

The resulting directory is similar to the `ubuntu12_32/steam-runtime`
directory in a Steam installation.

To get a Steam Runtime in a compressed tar archive for easy transfer to
other systems, similar to the official runtime deployed with the
Steam client, use a command like:

    ./build-runtime.py --archive=$(pwd)/steam-runtime.tar.xz

To output a tarball and metadata files with automatically-generated
names in a directory, specify the name of an existing directory, or a
directory to be created with a `/` suffix:

    ./build-runtime.py --archive=$(pwd)/archives/

or to force a particular basename to be used for the tar archive and all
associated metadata files, end with `.*`, which will usually need to be
quoted to protect it from shell interpretation:

    ./build-runtime.py --archive="$(pwd)/archives/steam-runtime.*"

The archive will unpack into a directory named `steam-runtime`.

The `--archive` and `--output` options can be combined, but at least one
is required.

Run `./build-runtime.py --help` for more options.

### Using a Steam Runtime

Once the runtime is downloaded (and unpacked into a directory, if you used
an archive), you can set up library pinning by running the **setup.sh** script,
then you can use the **run.sh** script to launch any program within that 
runtime environment.

For example, to get diagnostic information using the same tool used to get
what appears in Help -> System Information in Steam, if your runtime is in
'~/rttest', you could run: 
    
    ~/rttest/setup.sh
    ~/rttest/run.sh ~/rttest/usr/bin/steam-runtime-system-info

Or to launch Steam itself (and any Steam applications) within your runtime, 
set the STEAM_RUNTIME environment variable to point to your runtime directory;

    ~/.local/share/Steam$ STEAM_RUNTIME=~/rttest ./steam.sh
    Running Steam on ubuntu 14.04 64-bit 
    STEAM_RUNTIME has been set by the user to: /home/username/rttest
    
Building in the runtime
-----------------------

To prevent libraries from development and build machines 'leaking'
into your applications, you should build within a Steam Runtime chroot
environment or container.

To obtain one, first find an appropriate directory in
<https://repo.steampowered.com/steamrt-images-scout/snapshots/>.
The versioned directory names correspond to the
`version.txt` found in official Steam Runtime builds, typically
`ubuntu12_32/steam-runtime/version.txt` in a Steam installation: you
should usually choose a build environment whose version matches the
Steam Runtime bundled with the current Steam release, or a slightly
older version.

To build 64-bit software, download the files named
`com.valvesoftware.SteamRuntime.Sdk-amd64,i386-scout-sysroot.tar.gz` and
`com.valvesoftware.SteamRuntime.Sdk-amd64,i386-scout-sysroot.Dockerfile`. To
build legacy 32-bit software, instead download
`com.valvesoftware.SteamRuntime.Sdk-i386-scout-sysroot.tar.gz` and
`com.valvesoftware.SteamRuntime.Sdk-i386-scout-sysroot.Dockerfile`.

Each directory also contains various other archive and metadata files,
and a `sources/` subdirectory with source code for all the packages that
went into this Steam Runtime release.

### Using Docker

The recommended way to build for the Steam Runtime is in a Docker
container. Put the `-sysroot.tar.gz` and `-sysroot.Dockerfile` files
in an otherwise empty directory, `cd` into that directory, and import
them into Docker with a command like:

    sudo docker build \
    -f com.valvesoftware.SteamRuntime.Sdk-amd64,i386-scout-sysroot.Dockerfile \
    -t steamrt_scout_amd64:latest \
    .

or for a 32-bit environment,

    sudo docker build \
    -f com.valvesoftware.SteamRuntime.Sdk-i386-scout-sysroot.Dockerfile \
    -t steamrt_scout_i386:latest \
    .

Both containers can co-exist side by side. 32 bit steam-runtime libraries
are installed into the i386 root, and 64 bit steam-runtime libraries
are installed into the amd64 root. You can keep old versions of the
container around by tagging them with a version instead of `latest`,
for example `steamrt_scout_amd64:0.20191024.0`.

For historical reasons, it is also possible to run `setup_docker.sh`.
This will download an Ubuntu 12.04 container and convert it into a Steam
Runtime environment. The result does not match the official sysroot
tarball and is not guaranteed to match any specific/identifiable version
of the Steam Runtime, so this approach is not recommended.

### Using schroot

Alternatively, you can use Debian's schroot tool (this is likely to work
best on Debian or Ubuntu machines). `setup_chroot.sh` will create a
Steam Runtime chroot on your machine. This chroot environment contains
the same development libraries and tools as the Docker container. You will
need the 'schroot' tool installed, as well as root access through sudo.

For a 64-bit environment, use a command like:

    ./setup_chroot.sh --amd64 \
    --tarball ~/Downloads/com.valvesoftware.SteamRuntime.Sdk-amd64,i386-scout-sysroot.tar.gz

or for a 32-bit environment,

    ./setup_chroot.sh --i386 \
    --tarball ~/Downloads/com.valvesoftware.SteamRuntime.Sdk-i386-scout-sysroot.tar.gz

Both roots can co-exist side by side. 32 bit steam-runtime libraries are installed into the i386 root, and 64 bit steam-runtime libraries are installed into the amd64 root. 

Once setup-chroot.sh completes, you can use the **schroot** command to execute any build operations within the Steam Runtime environment.

    ~/src/mygame$ schroot --chroot steamrt_scout_i386 -- make -f mygame.mak

The root should be set up so that the path containing the build tree is the same inside as outside the root. If this path is not within the current user's home directory tree, it should be added to `/etc/schroot/default/fstab`

Then the next time the root is entered, this path will be available inside the root.

The setup script can be re-run to re-create the schroot environment.

For historical reasons, it is possible to run `setup_chroot.sh` without
using the `--tarball` option. This will download a minimal Ubuntu 12.04
environment and convert it into a Steam Runtime environment. The result
is not guaranteed to match the official sysroot tarballs, and whether
it succeeds is heavily dependent on the operating system on which you
are running the tool, so this approach is no longer recommended.

### Using a debugger in the build environment

To get the detached debug symbols that are required for `gdb` and
similar tools, you can download the matching
`com.valvesoftware.SteamRuntime.Sdk-amd64,i386-scout-debug.tar.gz`,
unpack it (preserving directory structure), and use its `files/`
directory as the schroot or container's `/usr/lib/debug`.

For example, with Docker, you might unpack the tarball in
`/tmp/scout-dbgsym-0.20191024.0` and use something like:

    sudo docker run \
    --rm \
    --init \
    -v /home:/home \
    -v /tmp/scout-dbgsym-0.20191024.0/files:/usr/lib/debug \
    -e HOME=/home/user \
    -u $(id -u):$(id -g) \
    -h $(hostname) \
    -v /tmp:/tmp \
    -it \
    steamrt_scout_amd64:latest \
    /dev/init -sg -- /bin/bash

or with schroot, you might create
`/var/chroots/steamrt_scout_amd64/usr/lib/debug/` and move the contents
of `files/` into it.

Default Tools
-------------

By default, a build environment is created that contains:

* gcc-4.6
* gcc-4.8 (default)
* gcc-5
* clang-3.4
* clang-3.6
* clang-3.8

Switching default compilers can be done by entering the chroot environment:

    ~$ schroot --chroot steamrt_scout_i386
    
    (steamrt_scout_i386):~$ # for gcc-4.6    
    (steamrt_scout_i386):~$ update-alternatives --auto gcc
    (steamrt_scout_i386):~$ update-alternatives --auto g++
    (steamrt_scout_i386):~$ update-alternatives --auto cpp-bin
    
    (steamrt_scout_i386):~$ # for gcc-4.8
    (steamrt_scout_i386):~$ update-alternatives --set gcc /usr/bin/gcc-4.8
    (steamrt_scout_i386):~$ update-alternatives --set g++ /usr/bin/g++-4.8
    (steamrt_scout_i386):~$ update-alternatives --set cpp-bin /usr/bin/cpp-4.8
    
    (steamrt_scout_i386):~$ # for clang-3.4
    (steamrt_scout_i386):~$ update-alternatives --set gcc /usr/bin/clang-3.4
    (steamrt_scout_i386):~$ update-alternatives --set g++ /usr/bin/clang++-3.4
    (steamrt_scout_i386):~$ update-alternatives --set cpp-bin /usr/bin/cpp-4.8
    
    (steamrt_scout_i386):~$ # for clang-3.6
    (steamrt_scout_i386):~$ update-alternatives --set gcc /usr/bin/clang-3.6
    (steamrt_scout_i386):~$ update-alternatives --set g++ /usr/bin/clang++-3.6
    (steamrt_scout_i386):~$ update-alternatives --set cpp-bin /usr/bin/cpp-4.8

Using detached debug symbols
----------------------------

Please see [doc/debug-symbols.md](doc/debug-symbols.md).
