# Build and Install UHD on Windows Subsystem Linux

## Table of contents

- [Abstract](#abstract)
- [Quick Start](#quick-start)
- [Post Installation Configuration](#post-installation-configuration)
    - [Configure USB Devices](#configure-usb-devices)
    - [Configure Ethernet Devices](#configure-ethernet-devices)
    - [Verify Device Operation](#verify-device-operation)
- [Additional Resources](#additional-resources)

## Abstract

## Quick Start

## Post Installation Configuration

### Configure USB Devices

**1. Follow WSL Documentation**

Unlike a bare metal installation of Linux, accessing USB devices from inside the WSL guest VM requires some
additional configuration in the Windows host OS. The official WSL documentation provides a comprehensive guide for 
connecting USB devices:
https://learn.microsoft.com/en-us/windows/wsl/connect-usb

After the USRP device is attached to WSL, ``usbipd`` shows its status as 
"attached" in the list of devices available on the Windows host OS. 

Example output of the ``usbipd wsl list`` command from a Windows Power Shell terminal in the Windows host OS:
```shell
BUSID  VID:PID    DEVICE                                                        STATE
...
5-1    2500:0022  Ettus Research B205mini                                       Attached - OAI-Ubuntu-22.04
...
```

Likewise, ``lsusb`` also finds the device inside the WSL guest VM.

Example output of the ``lsusb`` command in a Linux Bash terminal inside the WSL guest VM:

```shell
...
Bus 001 Device 004: ID 2500:0022 Ettus Research LLC USRP B205-mini
...
```
**NOTE:** the ``busid`` in the VM is different from the one in the host OS

**2. Add ``udev`` Rule inside WSL**

From this point, you can treat USRP from inside WSL as if it were connected to bare metal Linux. A key difference 
between a WSL VM and another virtualization technology called Docker is that WSL virtualizes the ``systemd`` 
daemon, see this WSL guide: https://learn.microsoft.com/en-us/windows/wsl/systemd. 

As such, ``udev``, a component of 
``systemd``, is available in WSL. As with bare metal Linux, you need to add a ``udev`` rule so that non-root users 
may access the device.

Run these commands in the WSL terminal:
```shell
#  add UHD-specific rules to the system's udev rules
cd <your-install-prefix>/uhd/host/utils
sudo cp uhd-usrp.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules
sudo udevadm trigger
```
**3. Understand the Effect of Loading FPGA Images**

The FPGA image does not persist on the USRP B Series devices. It is loaded automatically by UHD every time you run 
an application. 

Therefore, when the device is connected to the host PC, it does not have an FPGA image initially. The ``usbipd`` 
process running the Windows host OS, assigns a ``busid`` to the device in this image-less state. If you disconnect 
at this point then reconnect to the same USB port, the ``usbipd`` process will assign the same ``busid``.

Connect the USRP, then list the USB devices in Windows Power Shell:
```shell
usbipd wsl list
```
Example output:
```shell
BUSID  VID:PID    DEVICE                                                        STATE
...
5-1    2500:0022  Ettus Research B205mini                                       Not attached
...
```
Disconnect then reconnect the USRP, then list the USB devices again:
```shell
usbipd wsl list
```
Example output:
```shell
BUSID  VID:PID    DEVICE                                                        STATE
...
5-1    2500:0022  Ettus Research B205mini                                       Not attached
...
```

When you run an application that uses UHD, the driver will first load the FPGA image. The USRP momentarily 
disconnects from the host PC, then reconnects in its new imaged state. The FPGA image will persist as long as the 
USRP stays powered on (i.e. USB cable stays connected). Therefore, loading a new FPGA image, 
or disconnecting the USB cable, affects the ``busid`` assignment by ``usbipd``. The overall effect on device 
operation inside WSL is explained in more detailed in [Verify Device Operation](#verify-device-operation).


### Configure Ethernet Devices

### Verify Device Operation

## Additional Resources

