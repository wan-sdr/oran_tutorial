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

| Component         | Version          |
|:------------------|:-----------------|
| PC Architecture   | x86              |
| Operating System  | Linux            |
| Distribution      | Ubuntu 22.04 LTS |
| UHD Tag           | v4.5.0.0         |
| Virtualization    | N/A              |

## Quick Start
This section summarizes the step-by-step actions given in the rest of this guide. Follow these steps for 
a quick start experience. If you want to learn the technical details behind these steps, please read the entire 
guide.

**1. Install Ubuntu Desktop**

Follow the [instructions](https://ubuntu.com/tutorials/install-ubuntu-desktop#1-overview) provided by Ubuntu.

**2. Install Dependencies** 

In a terminal, run these shell commands:
```shell
# update package list
sudo apt-get update

# install Linux utilities, including build toolchain
sudo apt-get -y install build-essential ccache clang clang-format-14 cmake cmake-curses-gui cpufrequtils curl ethtool git inetutils-tools nano wget

# install UHD dependencies
sudo apt-get -y install doxygen dpdk libboost-all-dev libdpdk-dev libgps-dev libudev-dev libusb-1.0-0-dev python3-dev python3-docutils python3-mako python3-numpy python3-pip python3-requests

```

## Devices

## Dependencies

## Building and Installing from Source Code

## Configuring USB

## Configuring Ethernet

## Verifying Operation

## Thread Priority

## Additional Resources


