# Kuksa Dongle Setup Manual

This documentation provides information about setting up the Kuksa dongle.

## Table of Contents
1. [Pre-requisits](#Pre-requisits)
2. [OS Installation](#installation-of-os)  
 2.1 [Raspbian OS](#raspbian-os)  
 2.2 [Ubuntu OS](#ubuntu-os)
3. [OBD STN2120 Configuration](#obd-stn2120-configuration)
4. [Troubleshooting](#troubleshooting)


## Pre-requisits

##### Mandatory

The Kuksa Dongle is constructed to host and power a Raspberry Pi Compute Module 3 or 3+. Such compute module is required for the dongle and needs to be available.
Check official Raspberry Pi website for further information on the [Raspberry Pi Compute Module 3](https://www.raspberrypi.org/products/compute-module-3/).  
There are actually several versions you can order.
We can recommend to use a **Compute Module Lite** as it allows to mount a SD Card
with an associated operating system. This is much more convenient than flashing
the OS onto the board. In order to install any operating system, a SD card with
sufficient space (e.g., 32GB) is required.

* RaspberryPi Compute Module 3
* SD card (32 GB)

##### Optional

In order to be able to send and receive messages via the OBD interface, a OBD-II emulator like the [FreeMatics OBD-II Emulator Mk2](https://freematics.com/products/freematics-obd-emulator-mk2/) is recommended.


## Installation of OS

This guide provides information to install either the Raspbian OS (32bit system), as well as an Ubuntu Server (headless, 64bit system). If you plan to use the [Kuksa.val](https://github.com/eclipse/kuksa.val) Docker containers, we recommend to install the Ubuntu Server (64bit) system.

### Raspbian OS

Most convenient way to setup an operating system for Raspberry Pi is the Raspbian OS.
We tested the Dongle with [Raspbian LITE OS](https://www.raspberrypi.org/software/operating-systems/) in kernel version 5.10.
Download it from the Raspberry Pi website and install the image on a SD card (e.g., using the [Balena Etcher](https://www.balena.io/etcher/) tool).

When you boot the OS the first time, you need to disable the serial console (otherwise you will face a kernel panic). This is due to the fact, that the port is used by the OBD-II protocol decoder.
You have to mount */boot partition* of the SD card (e.g., Windows or Linux machine), and edit the **cmdline.txt** by deleting the **console=serial0,115200** command. Change it from

```sh
# before the changes
dwc_otg.lpm_enable=0 console=serial0,115200 console=tty1 root=PARTUUID=<PARTION_UUID_VALUE> rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait quiet splash plymouth.ignore-serial-consoles
```
to

```sh
# after the changes
dwc_otg.lpm_enable=0 console=tty1 root=PARTUUID=<PARTION_UUID_VALUE> rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait quiet splash plymouth.ignore-serial-consoles
```
After the changes you can plug the SD card into the dongle, power it up and log into it using the default user:password credentials of Raspian ("*pi:raspberry*").

### Ubuntu OS

The Kuksa Dongle uses a Raspberry Pi Compute Module 3 B+ (w/o internal storage).
This allows us to use a Raspberry Pi compliant operating system on a SD card.
We are using an ARM supported [Ubuntu Server](https://ubuntu.com/download/raspberry-pi/thank-you?version=20.04.2&architecture=server-arm64+raspi) (64bit system - ARM compliant - headless).

In this version of documentation, Ubuntu 20.10 is used -- Problems are faced when using Ubuntu 20.04 LTS.

Download it from the Ubuntu website and install the image on a SD card (e.g., using the [Balena Etcher](https://www.balena.io/etcher/) tool).

When you boot the OS the first time, you need to disable the serial console (otherwise you will face a kernel panic) as well as some labling of network interfaces.

You have to mount */boot partition* of the SD card (e.g., Windows or Linux machine), and edit the **cmdline.txt** by deleting the **console=serial0,115200** command. Change it from

```sh
# before the changes
dwc_otg.lpm_enable=0 console=serial0,115200 console=tty1 root=LABEL=writable rootfstype=ext4 elevator=deadline rootwait fixrtc quiet splash
```
to

```sh
# after the changes
dwc_otg.lpm_enable=0 console=tty1 root=LABEL=writable rootfstype=ext4 elevator=deadline rootwait fixrtc net.ifnames=0
```
After the changes you can plug the SD card into the dongle, power it up and log in with the default Ubuntu credentials (**ubuntu:ubuntu**). Ubuntu will ask you immediately to provide a new password of your choice.

#### Configure Network Interfaces in Ubuntu

In Ubuntu 20.10, **Netplan** is used to configure network interfaces. In order to configure wired and wireless network interfaces, you have to configure the **50-cloud-init.yaml** file, for example:

```sh
ubuntu@ubuntu:~$ sudo nano /etc/netplan/50-cloud-init.yaml
...
...
# This file is generated from information provided by the datasource.  Changes
# to it will not persist across an instance reboot.  To disable cloud-init's
# network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    renderer: networkd
    ethernets:
        eth0:
            dhcp4: true
            optional: true
            #set-name: eth0
    version: 2
    wifis:
        wlan0:
            optional: true
            access-points:
                "<SSID>":
                    password: "<PASSWD>"
            dhcp4: true
```

Where **<SSID>** is the identification name of your Wifi network and **<PASSWD>** is the password to authenticate the node to your Wifi network. Once the network has been configured, you have to restart the network manager or reboot your system to get the changes effective.


## OBD STN2120 Configuration

The board is designed to use the [STN2120 Chip](https://www.obdsol.com/solutions/chips/stn2120/) for OBD-II connectivity to send and receive messages form a Controller Area Network (CAN). The chip is ELM327 compatible, but bigger, faster and meaner!. You have to configure the baud rate of the CAN transceiver to the serial console in order to receive messages correctly.

Once you have installed any Debian based operating system (e.g., Raspbian OS, or Ubuntu), you have to configure the ELM327 serial bus to read/write to CAN transceiver. On the Dongle, you have to establish a connection to the CAN transceiver (e.g., using the "screen" command line tool - use "sudo apt get install screen" if not installed). Via "dmesg" you can check for the CAN transceiver device. In our setup, the following command worked:

```sh
# connect to the ELM327 serial bus using 'screen'
kuksa@pi:~$ sudo screen /dev/ttyAMA0 9600 81N
```
Once connected, you need to configure the ELM327 baud rate. You can check the entire [STN2120 Chip Manual](https://www.scantool.net/scantool/downloads/98/stn11xx21xx_frpm-c.pdf) for further configuration options.

```sh
# connect to the ELM327 serial bus using 'screen'
kuksa@pi:~$ sudo sceen /dev/ttyAMA0 9600 81N
> STSBR 2000000     # configure the baud rate to 2 MBit/s
> "OK"              # prompts OK
> <CTRL + K>        # exit screen
# connect again to persist the config changes
kuksa@pi:~$ sudo sceen /dev/ttyAMA0 2000000 81N
> STWBR             # persist baud rate
> <CTRL + K>        # exit screen -> there will not be an OK
```

You are DONE. Your CAN transceiver is configured. In order to test your CAN communication, you can create a Python script to test the CAN communication (e.g., https://python-obd.readthedocs.io/en/latest/). Please check the python-obd manual to check the properties to set to connect to the serial bus and configure the new baud rate.

```python
# python-obdtest.py
import obd

connection = obd.OBD(portstr="/dev/ttyAMA0", baudrate=2000000) # create connection

cmd = obd.commands.SPEED # select an OBD command (sensor)

response = connection.query(cmd) # send the command, and parse the response

print(response.value) # returns unit-bearing values thanks to Pint
print(response.value.to("mph")) # user-friendly unit conversions
```

We suggest to use the FreeMatics OBD-II Emulator Mk2: https://freematics.com/products/freematics-obd-emulator-mk2/  to send CAN messages to the Kuksa Dongle which you should receive on the Python script.

## Troubleshooting

#### Failed to start OpenBSD Secure Shell Server
If you see, that sshd service can not be started correctly and ubuntu boots with error message: `Failed to start OpenBSD Secure Shell Server`, run the following command to generate missing key will fix the issue:

```sh
kuksa@pi:~$ sudo ssh-keygen -A
```

#### Enable Docker Daemon Autostart
If you want to use the docker images of the [Kuksa.val](https://github.com/eclipse/kuksa.val) and you do not want to start the containers every time, you can enable the Docker daemon to automatically start after installing docker on the Kuksa Dongle.

```sh
kuksa@pi:~$ sudo systemctl enable docker
```
