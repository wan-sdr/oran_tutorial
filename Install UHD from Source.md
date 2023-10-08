# Build and Install UHD from Source Code

## Table of contents

- [Abstract](#abstract)
- [Quick Start](#quick-start)
- [Devices](#devices)
- [Dependencies](#dependencies)
- [Build and Install from Source](#build-and-install-from-source)
- [Post Installation Configuration](#post-installation-configuration)
- [Additional Resources](#additional-resources)

## Abstract

This Application Note provides a comprehensive guide for building and installing the open-source USRP Hardware Driver (
UHD) from source code. This guide addresses the use case shown in [Tested Configuration](#tested-configuration). For
other use cases, such as using package manager, other Linux distributions, Mac, Windows, Docker, WSL, and ARM
architecture, see [Additional Resources](#additional-resources).

### Tested Configuration

| Component        | Version          |
|:-----------------|:-----------------|
| PC Architecture  | x86              |
| Operating System | Linux            |
| Distribution     | Ubuntu 22.04 LTS |
| UHD Tag          | v4.5.0.0         |
| Virtualization   | N/A              |

## Quick Start

This section summarizes the step-by-step actions for a quick start experience, see the rest of this guide for more
details.

**1. Install Ubuntu Desktop**

Follow the instructions provided by Ubuntu:
https://ubuntu.com/tutorials/install-ubuntu-desktop#1-overview

**2. Install dependencies**

Run these commands:

```shell
# update package list
sudo apt-get update
# install Linux utilities, including build toolchain
sudo apt-get -y install build-essential ccache clang clang-format-14 cmake cmake-curses-gui cpufrequtils curl ethtool git inetutils-tools nano wget
# install UHD dependencies
sudo apt-get -y install doxygen dpdk libboost-all-dev libdpdk-dev libgps-dev libudev-dev libusb-1.0-0-dev python3-dev python3-docutils python3-mako python3-numpy python3-pip python3-requests python3-ruamel.yaml
```

**3. Get the latest source code**

Run these commands:

```shell
# create a directory to clone the UHD repository into
cd $HOME && mkdir workarea && cd workarea
# clone the UHD repository and checkout the latest tagged release
git clone https://github.com/EttusResearch/uhd.git uhd && cd uhd && git checkout v4.5.0.0
```

**4. Build and install**

To set a custom install prefix, see the detailed [Build and Install from Source](#build-and-install-from-source)
section.
Otherwise, to build and install with the default options, run these commands:

```shell
# create a directory for the build output
cd $HOME/workarea/uhd/host && mkdir build && cd build
# use CMake to configure the build with default options
cmake ..
# build the source code, using 8 threads
make -j8
# install into default prefix, /usr/local
sudo make install
# update dynamic linker with most recent shared libraries
sudo ldconfig
```

**5. Device specific configuration**

For USB devices, run these commands:

```shell
#  add UHD-specific rules to the system's udev rules
cd $HOME/workarea/uhd/host/utils
sudo cp uhd-usrp.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules
sudo udevadm trigger
```

For Ethernet devices, follow these instructions to configure static IP on the host
PC: https://linuxconfig.org/how-to-configure-static-ip-address-on-ubuntu-22-04-jammy-jellyfish-desktop-server

This table gives an example of the host network configuration for the USRP X310's SFP0 port with factory settings.
``Address`` and ``MTU`` depend on your device's specific settings,
see [Post Installation Configuration](#post-installation-configuration)

| Field       | Value         |
|-------------|---------------|
| IPv4 Method | Manual        |
| Address     | 192.168.10.1  |
| Subnet Mask | 255.255.255.0 |
| MTU         | 1500          |

Run these commands:

```shell
# send a ping from the host PC to the USRP IP to verify network connection
ping 192.168.10.2
```

**6. Verify device operation**

Run these commands:

```shell
# download FPGA images for the installed version of UHD
sudo uhd_images_downloader
# find all USRP devices connected to host
uhd_find_devices
# probe USRP device properties
uhd_usrp_probe
```

**For more information on the steps above, troubleshooting, performance tuning, UHD utilities, UHD application
development, and other open source projects that use UHD, see the rest of this guide and the
[Additional Resources](#additional-resources).**

## Devices

This guide only applies to USRP devices that operate by streaming IQ samples to an external host PC (x86 architecture)
via Ethernet or
USB. This mode of operation is
called **host (or network) mode**. Some USRP devices can operate in **embedded mode** where the IQ samples are
processed directly on the device's embedded ARM CPU, without streaming to an external host PC. Installing UHD on the
embedded CPU requires cross-compilation for the ARM architecture, which is not covered in this guide. The following
table groups each USRP device by operation mode:

| Operation Mode    | USRP Devices                                                         |
|-------------------|----------------------------------------------------------------------|
| Host/Network Only | X310, X300, N210, N200, B210, B200, B200mini, B200mini-i, B205mini-i |
| Embedded Only     | E310, E312, E313                                                     |
| Both              | X410, X440, N321, N320, N310, N300, E320                             |

**NOTE:** The embedded only devices can technically operate network mode, with very limited throughput, which we
generally do not recommend.

## Dependencies

The dependencies described in this section should work for Ubuntu 20.04 LTS and Ubuntu 22.04 LTS with UHD v4.0.0.0 or
later. For other older version of
Ubuntu, or other Linux distributions, please see [Additional Resources](#additional-resources).

All UHD dependencies are open source software that is available through the Ubuntu package manager ``apt``.
Typically, you do not need to build the dependencies themselves
from source, unless you are combining an older version of UHD and OS, or modifying UHD.

To install dependencies, run these commands:

```shell
# update package list
sudo apt-get update
# install Linux utilities, including build toolchain
sudo apt-get -y install autoconf automake build-essential ccache clang cmake cpufrequtils clang-format-14 curl ethtool git inetutils-tools libudev-dev nano wget
# install UHD dependencies
sudo apt-get -y install doxygen dpdk libboost-all-dev libdpdk-dev libgps-dev libncurses6 libusb-1.0-0-dev python3-dev python3-docutils python3-mako python3-numpy python3-requests python3-ruamel.yaml python3-setuptools
```

#### Pro Tip: Search Tool for Ubuntu Packages

Find any package available for specific version of Ubuntu, using this site: https://packages.ubuntu.com/. For
a given package, it tells you what files will be installed and where.

#### Pro Tip: Turn On/Off Optional Packages

The web tool also tells you the other packages
related to this package. Related packages are group by "depends", "recommends", and "suggests". By default,
``apt`` installs "depends" and "recommends", but not "suggests".

To change these settings:

```shell
# turn off recommended packages for <package-name>
sudo apt-get install --no-install-recommends <package-name>
#turn on suggested packages for <package-name>
sudo apt-get -o APT::Install-Suggests="true" install <package-name>
```

#### Pro Tip: Get UHD Dependencies

You can also use``apt depends`` to see the dependencies of any package. The benefit of this method is that you can
look up
dependencies for packages from PPAs, not just from the standard Ubuntu repositories. Ettus Research maintains a PPA of
prebuilt UHD packages. You can look up the dependencies of these packages and compare them to the ones given in the
earlier part of this section.

To look up UHD dependencies:

```shell
# add ettus research ppa
sudo add-apt-repository ppa:ettusresearch/uhd
# update apt package list
sudo apt-get update
# show dependencies of uhd-host
sudo apt-get depends uhd-host
#show dependencies of libuhd4.5.0
sudo apt-get depends libuhd4.5.0
```

Output of ```sudo apt-get depends uhd-host```

```
uhd-host
  Depends: libuhd4.5.0 (= 4.5.0.0-0ubuntu1~jammy1)
  Depends: python3
  Depends: python3-mako
  Depends: python3-numpy
  Depends: python3-requests
  Depends: python3-ruamel.yaml
  Depends: python3-setuptools
  Depends: libboost-filesystem1.74.0 (>= 1.74.0)
  Depends: libboost-program-options1.74.0 (>= 1.74.0)
  Depends: libboost-test1.74.0 (>= 1.74.0)
  Depends: libboost-thread1.74.0 (>= 1.74.0)
  Depends: libc6 (>= 2.34)
  Depends: libgcc-s1 (>= 4.0)
  Depends: libncurses6 (>= 6)
  Depends: libstdc++6 (>= 12)
  Depends: libtinfo6 (>= 6)
  Recommends: curl
  Recommends: procps
  Suggests: gnuradio
```

Output of ```sudo apt-get depends libuhd4.5.0```

```
libuhd4.5.0
  Depends: python3
  Depends: adduser
  Depends: libboost-chrono1.74.0 (>= 1.74.0)
  Depends: libboost-filesystem1.74.0 (>= 1.74.0)
  Depends: libboost-serialization1.74.0 (>= 1.74.0)
  Depends: libboost-thread1.74.0 (>= 1.74.0)
  Depends: libc6 (>= 2.34)
  Depends: libgcc-s1 (>= 3.3.1)
  Depends: libpython3.10 (>= 3.10.0)
  Depends: libstdc++6 (>= 12)
  Depends: libusb-1.0-0 (>= 2:1.0.22)
  Suggests: gnuradio
  Replaces: <libuhd003>
```

## Build and Install from Source

### UHD Versions

It’s important to know the different versions of the source code:

* Tagged releases are stable releases
* Master branch has the latest features
* Maintenance branches are for bug fixes on a major and API version after its release

The UHD version numbering scheme is MAJOR.API.ABI.PATCH,
see [UHD User Manual](https://files.ettus.com/manual/page_semver.html).

**NOTE:** Tag names currently have the format ``v4.5.0.0``. Before version 3.11.0.1, the format was
``release_003_011_000_001``.

To clone the UHD repository and check out the latest release::

```shell
# create a directory to clone the UHD repository into
cd $HOME && mkdir workarea && cd workarea
# clone the UHD repository and checkout the latest tagged release
git clone https://github.com/EttusResearch/uhd.git uhd && cd uhd && git checkout v4.5.0.0
```

To get a full list of tags:
```shell
git tag -l
```

To checkout the master or a maintenance branch:
```shell
# Example for UHD master branch:
git checkout master
# Example for UHD maintenance branch:
git checkout UHD-4.5
```

### Build and Install with System Prefix
CMake is the tool for configuring build options. An important CMake flag is ``-DCMAKE_INSTALL_PREFIX``. The default 
value is the system prefix. It tells Make to install UHD to the standard location ``/usr/local``
where the operating 
system will be able to find it in the standard ``$PATH`` and other environment variables. 

You can only install one concurrent version of UHD using the system prefix. To install multiple concurrent versions, see 
[Build and Install with Custom Prefix](#build-and-install-with-custom-prefix).

To build and install UHD:
```shell
# create a directory for the build output
cd $HOME/workarea/uhd/host && mkdir build && cd build
# use CMake to configure the build with default options
cmake ..
# build the source code, using 8 threads
make -j8
# optionally, run unit tests
make test
# install into default prefix, /usr/local
sudo make install
# update dynamic linker with most recent shared libraries
sudo ldconfig
```

**NOTE:** If some test cases fail in the optional ``make test`` step, it does not mean that the build failed. The test 
cases or the units under test might have a bug. Contact technical support in this case. 

#### Pro Tip: View All CMake Flags
It generally takes some experience with UHD to know all the important CMake Flags. The ``CMakeLists.txt`` files in 
the source code will give a lot of insight about build options. However, a simpler way to see all the available 
options is to use the ``cmake-curses-gui`` tool. 

To see all CMake flags:
```shell
# install cmake-curses-gui package
sudo apt-get update && sudo apt-get install cmake-curses-gui
# go to the build directory
cd $HOME/workarea/uhd/host/build
# use ccmake to configure the build instead
# when prompted, press c to configure, which reveals all the available flags
ccmake ..
```
### Build and Install with Custom Prefix


## Post Installation Configuration

## Additional Resources


