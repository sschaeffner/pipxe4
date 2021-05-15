piPXE - iPXE for the Raspberry Pi 4
===================================

[![Build](https://github.com/sschaeffner/pipxe4/actions/workflows/build.yml/badge.svg)](https://github.com/sschaeffner/pipxe4/actions/workflows/build.yml)
[![Release](https://img.shields.io/github/v/release/sschaeffner/pipxe4)](https://github.com/sschaeffner/pipxe4/releases/latest)

piPXE is a build of the [iPXE] network boot firmware for the
[Raspberry Pi].

Quick start
-----------

1. Download [sdcard.img] and write it onto any blank micro SD card
using a tool such as `dd` or [Etcher].

2. Insert the micro SD card into your Raspberry Pi.

3. Power on your Raspberry Pi.

Within a few seconds you should see iPXE appear and begin booting from
the network:

![Screenshot](screenshot.png)

Building from source
--------------------

To build from source, clone this repository and run `make`.  This will
build all of the required components and eventually generate the SD
card image [sdcard.img].

You will need various build tools installed, including a
cross-compiling version of `gcc` for building AArch64 binaries.

Fedora build tools:

    sudo dnf install -y binutils gcc gcc-aarch64-linux-gnu \
                        git-core iasl libuuid-devel make \
                        mtools perl python subversion xz-devel

Ubuntu build tools:

    sudo apt install -y build-essential gcc-aarch64-linux-gnu \
                        git iasl lzma-dev mtools perl python \
                        subversion uuid-dev

How it works
------------

The SD card image contains:

* Broadcom [VC4 boot firmware]: `bootcode.bin` and related files
* [TianoCore EDK2] UEFI firmware built for the [RPi4] platform: `RPI_EFI.fd`
* [iPXE] built for the `arm64-efi` platform: `/efi/boot/bootaa64.efi`

The Raspberry Pi has a somewhat convoluted boot process in which the
VC4 GPU is responsible for loading the initial executable ARM CPU
code.  The flow of execution is approximately:

1. The GPU code in the onboard boot ROM loads `bootcode.bin` from the SD card.
2. The GPU executes `bootcode.bin` and loads `RPI_EFI.fd` from the SD card.
3. The GPU allows the CPU to start executing `RPI_EFI.fd`.
4. The CPU executes `RPI_EFI.fd` and loads `bootaa64.efi` from the SD card.
5. The CPU executes `bootaa64.efi` (i.e. iPXE) to boot from the network.

Build Settings
--------------

The build has some non-default options for edk2 configured. See `EFI_FLAGS` in [Makefile].

* `--pcd=PcdPlatformBootTimeOut=$(EFI_TIMEOUT)` sets the timeout for entering the menu
* `--pcd=gRaspberryPiTokenSpaceGuid.PcdRamMoreThan3GB=1` enables more than 3GB RAM
* `--pcd=gRaspberryPiTokenSpaceGuid.PcdRamLimitTo3GB=0` disables limit to 3GB RAM
* `--pcd=gRaspberryPiTokenSpaceGuid.PcdSystemTableMode=1` enables ACPI+DT System Table

Licence
-------

Every component is under an open source licence.  See the individual
subproject licensing terms for more details:

* <https://github.com/raspberrypi/firmware/blob/master/boot/LICENCE.broadcom>
* <https://github.com/tianocore/edk2/blob/master/Readme.md>
* <https://ipxe.org/licensing>

[iPXE]: https://ipxe.org
[Raspberry Pi]: https://www.raspberrypi.org
[sdcard.img]: https://github.com/sschaeffner/pipxe4/releases/latest/download/sdcard.img
[Makefile]: https://github.com/sschaeffner/pipxe4/blob/master/Makefile#L7
[Etcher]: https://www.balena.io/etcher
[VC4 boot firmware]: https://github.com/raspberrypi/firmware/tree/master/boot
[TianoCore EDK2]: https://github.com/tianocore/edk2
[RPi4]: https://github.com/tianocore/edk2-platforms/tree/master/Platform/RaspberryPi/RPi4
