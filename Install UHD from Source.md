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

To set a custom install prefix, see the detailed [Build and Install from Source](#build-and-install-from-source) section.
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

| Operation Mode   | USRP Devices                                                         |
|------------------|----------------------------------------------------------------------|
| Host/Network Only | X310, X300, N210, N200, B210, B200, B200mini, B200mini-i, B205mini-i |
| Embedded Only    | E310, E312, E313                                                     |
| Both             | X410, X440, N321, N320, N310, N300, E320                             |

**NOTE:** The embedded only devices can technically operate network mode, with very limited throughput, which we 
generally do not recommend. 

## Dependencies

We tested this guide with the [configuration](#tested-configuration) above, but the dependencies described in this 
section should work for Ubuntu 20.04 LTS and Ubuntu 22.04 LTS with UHD v4.0.0.0 or later. For other older version of 
Ubuntu, or other Linux distributions, please see [Additional Resources](#additional-resources).

Dependencies are other software libraries that a software calls functions from. When building and 
installing from source code, the user must make sure these dependencies are present on the operating system and the 
toolchain must know where to find them. When installing software as a prebuilt package, the software installer or 
the OS package manager takes care of the dependencies for you.

In addition to software libraries that UHD needs to function, you also need to install several essential Linux 
utilities 
to perform the build from source process and to configure system settings after installation. These 
utilities include version control system, build automation tools, toolchains, etc.

All UHD dependencies and essential Linux utilities are open source software that is available as prebuilt packages 
installed using the standard Ubuntu package manager ``apt``. ``apt`` looks in a list of repositories to find these 
packages, the standard repository is ``Universe``. More recent versions of packages may be available in a third party 
repositories called a PPA, maintained by the original software developers or community members.  

You can get all packages needed to build, install and use UHD using ``apt`` and the ``Universe`` repository. 
Typically, do you need to build the dependencies themselves from source, or get them from a PPA, unless you are 
combining an older version of UHD and OS, or modifying UHD. 

To install dependencies, run these commands:

```shell
# update package list
sudo apt-get update
# install Linux utilities, including build toolchain
sudo apt-get -y install build-essential ccache clang clang-format-14 cmake cmake-curses-gui cpufrequtils curl ethtool git inetutils-tools nano wget
# install UHD dependencies
sudo apt-get -y install doxygen dpdk libboost-all-dev libdpdk-dev libgps-dev libudev-dev libusb-1.0-0-dev python3-dev python3-docutils python3-mako python3-numpy python3-pip python3-requests python3-ruamel.yaml
```

## Build and Install from Source





## Post Installation Configuration

## Additional Resources


