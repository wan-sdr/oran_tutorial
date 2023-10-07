# Build and Install UHD from Source

## Table of contents

- [Abstract](#abstract)
- [Quick Start](#quick-start)
- [Devices](#devices)
- [Dependencies](#dependencies)
- [Building and Installing from Source Code](#build-install)
- [Configuring USB](#configure-usb)
- [Configuring Ethernet](#configure-ethernet)
- [Verifying Operation](#verify-operation)
- [Thread Priority](#thread-priority)
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

This section summarizes the step-by-step actions described in detailed in the rest of this guide.
guide.

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

**3. Clone repository and checkout latest release**

Run these commands:

```shell
# create a directory to clone the UHD repository into
cd $HOME && mkdir workarea && cd workarea
# clone the UHD repository and checkout the latest tagged release
git clone https://github.com/EttusResearch/uhd.git uhd && cd uhd && git checkout v4.5.0.0
```

**4. Build and Install**

To set a custom install prefix, see the detailed [Building and Installing from Source Code](#build-install) section.
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
see [Post Installation Configuration](#post-installation)

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

## Dependencies

## Building and Installing from Source Code

## Configuring USB

## Configuring Ethernet

## Verifying Operation

## Thread Priority

## Additional Resources


