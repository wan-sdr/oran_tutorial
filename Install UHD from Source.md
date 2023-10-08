# Build and Install UHD from Source Code

## Table of contents

- [Abstract](#abstract)
- [Quick Start](#quick-start)
    - [Option 1: Prebuilt Packages from PPA](#option-1-prebuilt-packages-from-ppa)
    - [Option 2: Build and Install from Source](#option-2-build-and-install-from-source)
- [Devices](#devices)
- [Dependencies](#dependencies)
- [Build and Install from Source](#build-and-install-from-source)
    - [UHD Versions](#uhd-versions)
    - [Build and Install with System Prefix](#build-and-install-with-system-prefix)
    - [Build and Install with Custom Prefix](#build-and-install-with-custom-prefix)
- [Post Installation Configuration](#post-installation-configuration)
    - [Configure USB Devices](#configure-usb-devices)
    - [Configure Ethernet Devices](#configure-ethernet-devices)
    - [Verify Device Operation](#verify-device-operation)
- [Additional Resources](#additional-resources)

## Abstract

This Application Note provides a comprehensive guide for building and installing the open-source USRP Hardware Driver (
UHD) from source code. This guide addresses the use case shown in [Tested Configuration](#tested-configuration). For
other use cases, such as other Linux distributions, Mac, Windows, Docker, WSL, and ARM
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

This section summarizes the step-by-step for a quick start experience, see the rest of this guide for more
details.

### Option 1: Prebuilt Packages from PPA

For the *quickest* start experience, install prebuilt UHD packages from the Ettus Research PPA. Instead, If you want
to learn the build and install from source process, skip to Option 2.

**1. Install Ubuntu Desktop**

Follow the instructions provided by Ubuntu:
https://ubuntu.com/tutorials/install-ubuntu-desktop#1-overview

**2. Install prebuilt UHD packages from PPA**

```shell
# add ettus research ppa
sudo add-apt-repository ppa:ettusresearch/uhd
# update apt package list
sudo apt-get update
# install the following UHD packages
sudo apt-get install libuhd-dev uhd-host uhd-doc
```

**3. Verify device operation**

Run these commands:

```shell
# download FPGA images for the installed version of UHD
sudo uhd_images_downloader
# find all USRP devices connected to host
uhd_find_devices
# probe USRP device properties
uhd_usrp_probe
```

### Option 2: Build and Install from Source

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

For USB devices, run these commands **with the USRP disconnected**:

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
see [Configure Ethernet Devices](#configure-ethernet-devices)

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

**7. Verify device operation**

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

To build and install UHD with system prefix:

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

The system prefix ``/usr/local`` is shown at the end of the example output of ``cmake ..``:

```shell
...
-- ######################################################
-- # UHD enabled components
-- ######################################################
--   * LibUHD
--   * LibUHD - C API
--   * LibUHD - Python API
--   * Examples
--   * Utils
--   * Tests
--   * USB
--   * B100
--   * B200
--   * USRP1
--   * USRP2
--   * X300
--   * MPMD
--   * SIM
--   * N300
--   * N320
--   * E320
--   * E300
--   * X400
--   * OctoClock
--   * DPDK
--   * Manual
--   * API/Doxygen
--   * Man Pages
--
-- ######################################################
-- # UHD disabled components
-- ######################################################
--
-- ******************************************************
-- * You are building a development branch of UHD.
-- * These branches are designed to provide early access
-- * to UHD and USRP features, but should be considered
-- * unstable and/or experimental!
-- ******************************************************
-- Building version: 4.5.0.HEAD-0-g471af98f
-- Using install prefix: /usr/local
-- Configuring done
-- Generating done
-- Build files have been written to: /home/orantestbed/workarea/uhd/host/build
```

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

``CMAKE_INSTALL_PREFIX`` has the default value ``/usr/local/`` as shown in the example output of ``ccmake ..``. You
can scroll through the list of flags and confirm that each ``ENABLE_<COMPONENT-NAME>`` value matches the "UHD Enabled
Components" list from the output of ``cmake ..``

```shell
 ...
 Boost_CHRONO_LIBRARY_RELEASE     /usr/lib/x86_64-linux-gnu/libboost_chrono.so.1.74.0
 Boost_DATE_TIME_LIBRARY_RELEAS   /usr/lib/x86_64-linux-gnu/libboost_date_time.so.1.74.0
 Boost_FILESYSTEM_LIBRARY_RELEA   /usr/lib/x86_64-linux-gnu/libboost_filesystem.so.1.74.0
 Boost_INCLUDE_DIR                /usr/include
 Boost_PROGRAM_OPTIONS_LIBRARY_   /usr/lib/x86_64-linux-gnu/libboost_program_options.so.1.74.0
 Boost_SERIALIZATION_LIBRARY_RE   /usr/lib/x86_64-linux-gnu/libboost_serialization.so.1.74.0
 Boost_SYSTEM_LIBRARY_RELEASE     /usr/lib/x86_64-linux-gnu/libboost_system.so.1.74.0
 Boost_THREAD_LIBRARY_RELEASE     /usr/lib/x86_64-linux-gnu/libboost_thread.so.1.74.0
 Boost_UNIT_TEST_FRAMEWORK_LIBR   /usr/lib/x86_64-linux-gnu/libboost_unit_test_framework.so.1.74
 CMAKE_BUILD_TYPE
 CMAKE_INSTALL_PREFIX             /usr/local
 DPDK_CONFIG_INCLUDE_DIR          /usr/include/x86_64-linux-gnu/dpdk
 DPDK_LIBRARY                     DPDK_LIBRARY-NOTFOUND
 DPDK_VERSION_INCLUDE_DIR         /usr/include/dpdk
 ENABLE_B100                      ON
 ENABLE_B200                      ON
 ENABLE_C_API                     ON
 ENABLE_DOXYGEN                   ON
 ENABLE_DOXYGEN_DOT               OFF
 ENABLE_DOXYGEN_FULL              OFF
 ENABLE_DOXYGEN_SHORTNAMES        OFF
 ENABLE_DPDK                      ON
 ENABLE_E300                      ON
 ENABLE_E320                      ON
 ENABLE_EXAMPLES                  ON
 ENABLE_LIBUHD                    ON
 ENABLE_MANUAL                    ON
 ENABLE_MAN_PAGES                 ON
 ENABLE_MAN_PAGE_COMPRESSION      ON
 ENABLE_MPMD                      ON
 ENABLE_N300                      ON
 ENABLE_N320                      ON
 ENABLE_OCTOCLOCK                 ON
 ENABLE_PYTHON_API                ON
 ENABLE_SIM                       ON
 ENABLE_STATIC_LIBS               OFF
 ENABLE_TESTS                     ON
 ENABLE_USB                       ON
 ENABLE_USRP1                     ON
 ENABLE_USRP2                     ON
 ENABLE_UTILS                     ON
 ENABLE_X300                      ON
...
```

### Build and Install with Custom Prefix

If you want to install multiple concurrent versions of UHD, use the CMake flag
``-DCMAKE_INSTALL_PREFIX=<your-custom-prefix>``. The custom prefix can be any directory, but it's good practice to
create an ``installs`` directory in ``$HOME/workarea`` that you created earlier. You need a **different custom prefix
for each UHD version**. In addition, you can also have one version of
UHD in the system prefix.

To build and install UHD with custom prefix:

```shell
# create a directory for all custom prefix installations
cd $HOME/workarea && mkdir installs
# create a directory for the build output
cd $HOME/workarea/uhd/host && mkdir build_custom_prefix && cd build_custom_prefix
# use CMake to configure the build with default options
cmake -DCMAKE_INSTALL_PREFIX=~/workarea/installs/uhdv4_5_0_0 ..
# build the source code, using 8 threads
make -j8
# optionally, run unit tests
make test
# install into default prefix, /usr/local
make install
# update dynamic linker with most recent shared libraries
sudo ldconfig
```

**NOTE:** ``sudo`` is not necesssary in the ``make install`` because the install files are not written to a
privileged directory like ``/usr/local``. In fact, adding ``sudo`` to this step, makes the custom prefix a
privileged directory, and non-root users cannot it.

The custom prefix ``~/workarea/installs/uhdv4_5_0_0``

is shown in example output of
``cmake -DCMAKE_INSTALL_PREFIX=~/workarea/installs/uhdv4_5_0_0 ..``

```shell
...
-- Building version: 4.5.0.HEAD-0-g471af98f
-- Using install prefix: /home/orantestbed/workarea/installs/uhdv4_5_0_0
-- Configuring done
-- Generating done
-- Build files have been written to: /home/orantestbed/workarea/uhd/host/build_custom_prefix
```

Look inside the custom prefix for various UHD program directories:

```shell
ls /home/orantestbed/workarea/installs/uhdv4_5_0_0
```

Output:

```shell
bin  include  lib  share
```

Look inside ``bin`` for the binary executables of various utilities:

```shell
ls /home/orantestbed/workarea/installs/uhdv4_5_0_0/bin
```

Output:

```shell
rfnoc_image_builder    uhd_cal_tx_dc_offset   uhd_find_devices       uhd_usrp_probe     usrpctl
uhd_adc_self_cal       uhd_cal_tx_iq_balance  uhd_image_loader       usrp2_card_burner
uhd_cal_rx_iq_balance  uhd_config_info        uhd_images_downloader  usrp_hwd.py
```

The operating system cannot find these UHD program files at the custom prefix location using the default values of
``$PATH`` and other
environmental variables. Important paths within the custom prefix need to be added to the relevant environment
variables.

Look for a UHD program file listed above:

```shell
which uhd_find_devices
```

``which`` returns the version installed in the system prefix, or no result if none exists, as shown in the
example
output:

```shell
/usr/local/bin/uhd_find_devices
```

The environment of the custom prefix need to be "activated" by updating the appropriate environment variables.

Create a ``bash`` shell
script to update the environment variables:

```shell
# go to the directory for the custom prefix
cd $HOME/workarea/installs/uhdv4_5_0_0
# create an empty file for the script
touch uhdv4_5_0_0.env
```

Copy the following code into the file using an editor like ``nano``:

```shell
#!/bin/bash

CUSTOMPREFIX=$HOME/workarea/installs/uhdv4_5_0_0
export PATH=$CUSTOMPREFIX/bin:$PATH
export LD_LOAD_LIBRARY=$CUSTOMPREFIX/lib:$LD_LOAD_LIBRARY
export LD_LIBRARY_PATH=$CUSTOMPREFIX/lib:$LD_LIBRARY_PATH
export PYTHONPATH=$CUSTOMPREFIX/lib/python3.10/site-packages:$PYTHONPATH
export PKG_CONFIG_PATH=$CUSTOMPREFIX/lib/pkgconfig:$PKG_CONFIG_PATH
export UHD_RFNOC_DIR=$CUSTOMPREFIX/share/uhd/rfnoc/
export UHD_IMAGES_DIR=$CUSTOMPREFIX/share/uhd/images
```

#### Pro Tip: How to Find ``PYTHONPATH``

``PYTHONPATH`` differs depending on the version of OS, Python, and UHD. To find it consistently:

```shell
find ~/workarea/installs/uhdv4_5_0_0/ -name uhd | grep "packages"
```

Output:

```shell
/home/orantestbed/workarea/installs/uhdv4_5_0_0/lib/python3.10/site-packages/uhd
```

To activate the environment for the custom prefix:

```shell
source uhdv4_5_0_0.env
```

Look for a UHD program file again:

```shell
which uhd_find_devices
```

The version in the custom prefix is found, but not the one in the system prefix, as shown in the example output:

```shell
/home/orantestbed/workarea/installs/uhdv4_5_0_0/bin/uhd_find_devices
```

The ``source`` command affects only the terminal it runs in. Therefore,
you can have multiple terminals, each with a different custom prefix environment activated. The effects do not
persist after the terminal is closed.

**NOTE:** ``source`` is a built-in command for ``bash`` shells only.

**NOTE:** Installing UHD to a custom prefix also affects how other software that depends on UHD will find it when they
are
built from source, see [Find UHD in Custom Prefix](#find-uhd-in-custom-prefix).

## Post Installation Configuration

### Configure USB Devices

On Linux, ``udev`` handles USB plug and unplug events. The following commands install a udev rule so that non-root users
may
access the device. This step is only necessary for devices that use USB to connect to the host computer, such as the
B200, B210, B200mini, B200mini-i, and B205mini-i. This setting should take effect immediately and does not require a
reboot or
logout/login. Be
sure that no USRP device is connected via USB when running these commands

```shell
#  add UHD-specific rules to the system's udev rules
cd $HOME/workarea/uhd/host/utils
sudo cp uhd-usrp.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules
sudo udevadm trigger
```

Verify USB connection using:

```shell
lsusb
```

You should see the USRP listed on the USB bus with a VID of ``2500`` and PID of ``0020``, ``0021``, ``0022``, for B200,
B210,
B200mini/B200mini-i/B205mini-i,
respectively.

```shell
...
Bus 001 Device 002: ID 2500:0022 Ettus Research LLC USRP B205-mini
...
```

### Configure Ethernet Devices

The network interface of the host PC needs to have a static IP address on the same subnet as the USRP IP address. You
should
configure the network interface using the graphical Network Manager. If you use the command line tool ``ifconfig``, the
Network Manager will probably overwrite these settings. Follow these instructions to configure static IP on the host
PC: https://linuxconfig.org/how-to-configure-static-ip-address-on-ubuntu-22-04-jammy-jellyfish-desktop-server

The table below gives an example of the host network configuration for the USRP X310's SFP0 port with factory settings.
The default IP of SFP0 is 192.168.10.2, so we can assign the host PC with the IP 192.168.10.1. The default
Ethernet PHY of SFP0 is 1 GbE, so MTU is set to 1500. If the Ethernet PHY is 10 GbE, then MTU is set to 9000.

| Field       | Value         |
|-------------|---------------|
| IPv4 Method | Manual        |
| Address     | 192.168.10.1  |
| Subnet Mask | 255.255.255.0 |
| MTU         | 1500          |

Verify network connection using ``ping``:

```shell
ping 192.168.10.2
```

Output:

```shell
PING 192.168.10.2 (192.168.10.2) 56(84) bytes of data.
64 bytes from 192.168.10.2: icmp_seq=1 ttl=63 time=2.20 ms
64 bytes from 192.168.10.2: icmp_seq=2 ttl=63 time=2.16 ms
64 bytes from 192.168.10.2: icmp_seq=3 ttl=63 time=2.14 ms
64 bytes from 192.168.10.2: icmp_seq=4 ttl=63 time=2.18 ms
64 bytes from 192.168.10.2: icmp_seq=5 ttl=63 time=1.90 ms
^C
--- 192.168.10.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4005ms
rtt min/avg/max/mdev = 1.903/2.118/2.202/0.109 ms
```

To learn more about default network settings of each device, see the [UHD Manual](https://files.ettus.com/manual/).

If you have any issues with connecting to the USRP, see [Troubleshooting](#troubleshooting) or 
contact
Technical Support.

### Verify Device Operation

The FPGA image version on the USRP must match the UHD version on the host PC.

Download FPGA images for the installed version of UHD:

```shell
sudo uhd_images_downloader
```

**NOTE:** ``sudo`` is **not needed** if you installed UHD with a custom prefix that is not in a privileged directory.

Output:

```shell
[INFO] Using base URL: https://files.ettus.com/binaries/cache/
[INFO] Images destination: /usr/local/share/uhd/images
[INFO] No inventory file found at /usr/local/share/uhd/images/inventory.json. Creating an empty one.
32358 kB / 32358 kB (100%) x4xx_x410_fpga_default-g5da42f3.zip
42578 kB / 42578 kB (100%) x4xx_x440_fpga_default-g5da42f3.zip
21587 kB / 21587 kB (100%) x3xx_x310_fpga_default-ga2a04e3.zip
19763 kB / 19763 kB (100%) x3xx_x300_fpga_default-ga2a04e3.zip
01121 kB / 01121 kB (100%) e3xx_e310_sg1_fpga_default-ga2a04e3.zip
01118 kB / 01118 kB (100%) e3xx_e310_sg3_fpga_default-ga2a04e3.zip
10196 kB / 10196 kB (100%) e3xx_e320_fpga_default-ga2a04e3.zip
20929 kB / 20929 kB (100%) n3xx_n310_fpga_default-g5da42f3.zip
14275 kB / 14275 kB (100%) n3xx_n300_fpga_default-g5da42f3.zip
27174 kB / 27174 kB (100%) n3xx_n320_fpga_default-g5da42f3.zip
00481 kB / 00481 kB (100%) b2xx_b200_fpga_default-g92c09f7.zip
00464 kB / 00464 kB (100%) b2xx_b200mini_fpga_default-g92c09f7.zip
00883 kB / 00883 kB (100%) b2xx_b210_fpga_default-g92c09f7.zip
00511 kB / 00511 kB (100%) b2xx_b205mini_fpga_default-g92c09f7.zip
00167 kB / 00167 kB (100%) b2xx_common_fw_default-g7f7d016.zip
00007 kB / 00007 kB (100%) usrp2_usrp2_fw_default-g6bea23d.zip
00450 kB / 00450 kB (100%) usrp2_usrp2_fpga_default-g6bea23d.zip
02415 kB / 02415 kB (100%) usrp2_n200_fpga_default-g6bea23d.zip
00009 kB / 00009 kB (100%) usrp2_n200_fw_default-g6bea23d.zip
02757 kB / 02757 kB (100%) usrp2_n210_fpga_default-g6bea23d.zip
00009 kB / 00009 kB (100%) usrp2_n210_fw_default-g6bea23d.zip
00319 kB / 00319 kB (100%) usrp1_usrp1_fpga_default-g6bea23d.zip
00522 kB / 00522 kB (100%) usrp1_b100_fpga_default-g6bea23d.zip
00006 kB / 00006 kB (100%) usrp1_b100_fw_default-g6bea23d.zip
00017 kB / 00017 kB (100%) octoclock_octoclock_fw_default-g14000041.zip
04839 kB / 04839 kB (100%) usb_common_windrv_default-g14000041.zip
[INFO] Images download complete.
```

Find all USRP devices connected to host PC:

```shell
uhd_find_devices
```

Example output for B205mini:

```shell
[INFO] [UHD] linux; GNU C++ version 11.4.0; Boost_107400; DPDK_21.11; UHD_4.5.0.HEAD-0-g471af98f
--------------------------------------------------
-- UHD Device 0
--------------------------------------------------
Device Address:
    serial: 316B7A0
    name: B205i
    product: B205mini
    type: b200
```

Probe USRP device properties

```shell
uhd_usrp_probe
```

Example output for B205mini:

```shell
[INFO] [UHD] linux; GNU C++ version 11.4.0; Boost_107400; DPDK_21.11; UHD_4.5.0.HEAD-0-g471af98f
[INFO] [B200] Detected Device: B205mini
[INFO] [B200] Loading FPGA image: /usr/local/share/uhd/images/usrp_b205mini_fpga.bin...
[INFO] [B200] Operating over USB 3.
[INFO] [B200] Initialize CODEC control...
[INFO] [B200] Initialize Radio control...
[INFO] [B200] Performing register loopback test...
[INFO] [B200] Register loopback test passed
[INFO] [B200] Setting master clock rate selection to 'automatic'.
[INFO] [B200] Asking for clock rate 16.000000 MHz...
[INFO] [B200] Actually got clock rate 16.000000 MHz.
  _____________________________________________________
 /
|       Device: B-Series Device
|     _____________________________________________________
|    /
|   |       Mboard: B205mini
|   |   serial: 316B7A0
|   |   name: B205i
|   |   product: 30522
|   |   revision: 3
|   |   FW Version: 8.0
|   |   FPGA Version: 7.0
|   |
|   |   Time sources:  none, internal, external
|   |   Clock sources: internal, external
|   |   Sensors: ref_locked
|   |     _____________________________________________________
|   |    /
|   |   |       RX DSP: 0
|   |   |
|   |   |   Freq range: -8.000 to 8.000 MHz
|   |     _____________________________________________________
|   |    /
|   |   |       RX Dboard: A
|   |   |     _____________________________________________________
|   |   |    /
|   |   |   |       RX Frontend: A
|   |   |   |   Name: FE-RX1
|   |   |   |   Antennas: TX/RX, RX2
|   |   |   |   Sensors: temp, rssi, lo_locked
|   |   |   |   Freq range: 50.000 to 6000.000 MHz
|   |   |   |   Gain range PGA: 0.0 to 76.0 step 1.0 dB
|   |   |   |   Bandwidth range: 200000.0 to 56000000.0 step 0.0 Hz
|   |   |   |   Connection Type: IQ
|   |   |   |   Uses LO offset: No
|   |   |     _____________________________________________________
|   |   |    /
|   |   |   |       RX Codec: A
|   |   |   |   Name: B205mini RX dual ADC
|   |   |   |   Gain Elements: None
|   |     _____________________________________________________
|   |    /
|   |   |       TX DSP: 0
|   |   |
|   |   |   Freq range: -8.000 to 8.000 MHz
|   |     _____________________________________________________
|   |    /
|   |   |       TX Dboard: A
|   |   |     _____________________________________________________
|   |   |    /
|   |   |   |       TX Frontend: A
|   |   |   |   Name: FE-TX1
|   |   |   |   Antennas: TX/RX
|   |   |   |   Sensors: temp, lo_locked
|   |   |   |   Freq range: 50.000 to 6000.000 MHz
|   |   |   |   Gain range PGA: 0.0 to 89.8 step 0.2 dB
|   |   |   |   Bandwidth range: 200000.0 to 56000000.0 step 0.0 Hz
|   |   |   |   Connection Type: IQ
|   |   |   |   Uses LO offset: No
|   |   |     _____________________________________________________
|   |   |    /
|   |   |   |       TX Codec: A
|   |   |   |   Name: B205mini TX dual DAC
|   |   |   |   Gain Elements: None
```

**At this point, the build and install from source process completed successfully and the USRP is operational. If you 
experienced any issues during this process, or if you are curious about various UHD warning messages, see 
[Troubleshooting](#troubleshooting) or contact Technical Support.** 

### SDR Application Development

For a complete open-source SDR toolchain, you also need an application that processes the IQ samples that UHD
streams between USRP and host PC. This could be an application that you develop using the UHD C/C++ or
Python API, or existing application framework such as GNU Radio, Open Air Interface, SRS RAN, etc.

* To develop with the UHD C/C++ API,
see [Getting Started with UHD and C++](https://kb.ettus.com/Getting_Started_with_UHD_and_C%2B%2B).

* To develop with the UHD Python API, see [UHD Python API](https://kb.ettus.com/UHD_Python_API).

* To build and install GNU Radio from source, on top of UHD, see instructions from GNU Radio:
https://wiki.gnuradio.org/index.php?title=LinuxInstall#From_Source

* To develop with OAI, see
the [OAI Reference Architecture](https://kb.ettus.com/OAI_Reference_Architecture_for_5G_and_6G_Research_with_USRP)

* We are  working on a similar reference architecture for SRS.

#### Find UHD in Custom Prefix
When building these applications from source code, they are typically configured to look for UHD in the system prefix.
If UHD is installed in a custom prefix, the build configuration needs these CMake
flags to find UHD as a dependency:
* ```-DCMAKE_INSTALL_PREFIX=<your-custom-prefix>``` 
* ```-DUHD_DIR=<your-custom-prefix>lib/cmake/uhd/```
* ```-DUHD_INCLUDE_DIRS=<your-custom-prefix>/include/```
* ```-DUHD_LIBRARIES=<your-custom-prefix>/lib/libuhd.so```.

How these flags are set depends on the application's build system. 

For UHD C/C++ API, in the [Compile and Install](https://kb.ettus.com/Getting_Started_with_UHD_and_C%2B%2B#Compile_and_Install) section of the guide, replace the ``cmake`` step with the 
following command:

```shell
cmake -DCMAKE_INSTALL_PREFIX=<your-custom-prefix> -DUHD_DIR=<your-custom-prefix>lib/cmake/uhd/ 
-DUHD_INCLUDE_DIRS=<your-custom-prefix>/include/ -DUHD_LIBRARIES=<your-custom-prefix>/lib/libuhd.so ..
```
For UHD Python API, it is enough just to update the ``PYTHONPATH`` variable with the custom prefix, since Python is 
not compiled, the interpreter just needs to know where the UHD Python package is for the ``import`` statement.

For GNU Radio, follow the same step as the UHD C/C++ use case.

However, OAI uses a build script that runs on top of ``cmake``, see the OAI guide linked above. 

## Additional Resources

### UHD Installation

* Overview of All UHD Installation Application Notes (Coming soon...)
* Build and Install UHD on Older Ubuntu and Other Linux Distributions (Coming soon...)
* Build and Install UHD from Source on Mac
* Build and Install UHD from Source on Windows
* Build and Install UHD on Embedded Devices
* Build and Install UHD from Source on an Offline System
* Using UHD with Windows Subsystem Linux (Coming soon...)
* Using UHD with Docker Containers (Coming soon...)

### Troubleshooting

* Verifying Device Operation
* Host Performance Tuning
* Device Discovery Issues
* Device Recovery

### Application Development

* Developing UHD Application
* GNU Radio Installation
* OAI Reference Architecture
* SRS Reference Architecture (Coming Soon...)


