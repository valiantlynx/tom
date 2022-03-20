# OpenTom

**OpenTom** is a tiny, open source Linux distribution for TomTom™ devices.  
*This repository was tested to be working on **June 2020**.*

## Getting started
you should install the Remote - Containers vscode extention to run the container.
the container is a ubuntu focal linux os. thats where you run the commands.

I PUT THE FILE IN A DOCKER CONTAINER. ITS NOW A VSCODE DOCKER ENVIRONMENT. THUS U CAN SKIP STEP 1 AND 2. START AT set the ROOT...
Before proceeding, it is recommended to backup the original contents of your GPS's internal storage.

- Register the i386 architecture by executing `sudo dpkg --add-architecture i386` (or the equivalent for your system)
- Install the following dependencies: `autoconf chrpath fluid imagemagick libglib2.0-dev libtool subversion xsltproc gawk dh-autoreconf pkg-config libglib2.0-dev libc6:i386 libncurses5:i386 libstdc++6:i386 cmake`
- Set the `ROOT` envvar in `get_cross_env.sh`
- Enter the root terminal (`sudo su` in Ubuntu)
- Execute `source get_cross_env.sh`
- Run `make` to start the initial OpenTom build\*
  - This may take a while, as it will download and compile every system component
- Copy `build/ttsystem` (boot image) to the root folder of the GPS storage
- Copy the contents of `opentom_dist/` to a (new) folder called `opentom` on the storage

*\*If an error like the following one appears during the compilation process...*
> Invalid configuration \`x86_64-unknown-linux-gnu': machine \`x86_64-unknown' not recognized  
> make: *** [Makefile:288: arm-sysroot/usr/include/jpeglib.h] Error 2  

*...you can continue compiling by executing `linux32 make`*

## How to build extra applications

- Run `make extra` and copy the files as described above
- For `dosbox dune2 gnuboy linapple` and scummvm games: take them from the Internet and copy them into opentom/share subdirectories.
- For **coolreader**: Run `sudo updatedb` in case the default font is not found.
- For **Navit**\*, you need the TomTom gltt (see below, that read raw GPS data from `/dev/gps` and send it to `/var/run/gpspipe`).

*Note: currently, `sprsht freecell navit` are disabled as they are not compiling correctly.*

### On the TomTom side

- The IP that the GPS uses is usually `192.168.1.10` (if it does not work, try `192.168.1.200`)
- When you boot your TomTom with OpenTom, you can directly use Telnet to login in as `root` from USB (it is recommended to connect the device to Linux for maximum compatibility)
- Use the built-in FTP server to update your files
- `strace` and `gdb` are ready to be used to debug your programs

# Untested guides

The following sections of this documentation have not been tested by me (**@raulbalanza**), so they are not guaranteed to be working as of June 2020.

## How to modify `ttsystem`

- For kernel: `cd kernel; make menuconfig`
- For busybox: `cd build/busybox*; make menuconfig`
- For initramfs: do your changes and `touch initramfs/etc/rc`
- Then: return to `$ROOT` and run `make ttsystem`

## How to add some new applications

- Just extract you source into `$ROOT/src` (for libraries) or `applications/src` (or `build` if no patches should be applied)
- Run `./configure --prefix=$ARM_APPROOT --host=$T_ARCH` (adapt accordingly in case the project is not based on Autoconf)
- Copy the final executable into `$(TOMDIST)/bin`
- Run `make verif_dist` (inside `$ROOT` directory) to update used shared libs in `opentom_dist`
- Copy the files to your TomTom device
- If it works as you wish, make a patch (with `make patch-<my_app_dir_name>`) and update `applications/Makefile`

## Creating Nano-X test platform on your system

- Create `$ROOT/i386` directory
- Extract, configure and install: microwin, nxlib (with libNX11), SDL, Fltk, ... (with `LDFLAGS=-L/usr/local/lib --prefix=/usr/local`)
- Try NetBeans to perform you developments?

## How to free some memory (~3-4Mo on 32!)

Use an ext2 partion on your SDcard to replace (and free) initramfs with busybox pivot_root/chroot:

- Use `fdisk` to create two partitions on your SD card: partion1=vfat(TomTom), partition2=ext2(10Mo?)
- Copy `ttsystem` into SD.part1 and verify it boots, if not try to copy gns and program directory from original TomTom and others...
- When it boots:
- Verify that busybox include chroot and pivot_root,
- Copy the unmodified `initramfs/*` into linux SD partition, with `/var/*` linked to `/tmp`
- Copy the `configs/etc_rc_ext2` to `SD.part2/etc/rc`
- Verify that kernel include ext2 filesystem support
- Copy `configs/etc_rc_file.pivot_root_ext2` to `initramfs/etc/rc`
- `make ttsystem`
- Then on partition 1 copy `build/ttsystem` and the `opentom_dist` directory on SD.part1
- If something goes wrong, try `configs/kernel_config.console_ext2` to activate kernel FrameBuffer console


## How to install gltt from TomTom `ttsystem` file (for Navit)

- Copy your TomTom™ `ttsystem` file into `$ROOT/src` (e.g. `cp /mnt/TOMTOM/ttsystem $ROOT/src/ttsystem.tomtom`)
- `cd $ROOT/src`
- `ttimgextract ttsystem.tomtom `
- `mkdir -p ttsystem.tomtom.initramfs`
- `cd ttsystem.tomtom.initramfs`
- `gunzip -c ../ttsystem.tomtom.0 | sudo cpio -i`
- Now the boot ramdisk of your TomTom is extracted to `$ROOT/src/ttsystem.tomtom.initramfs` and the TomTom kernel is in `$ROOT/src/ttsystem.1`
- Then: `cp $ROOT/src/ttsystem.tomtom.initramfs/bin/gltt $TOMDIST/bin`

## To-Do

- Fix espeak => portaudio => OSS, that currently don't work
- Patch `spreadsheet` to be adapted to TomTom screen

## Support

- Checkout the creators' documentation in `docs/`
- Take a look at the shell scripts, Makefiles and patches
- Check the [original repository](https://github.com/george-hopkins/opentom) for the latest patches
- Send an email in case of a problem to any of the collaborators (mainly to the original creator).

## Collaborators

- Clément Gerardin (opentom@free.fr): project creator and supporter
- Raúl Balanzá (contact@raulbalanza.me): project sources update and verification
