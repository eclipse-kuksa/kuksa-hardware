# Kuksa Dongle Setup Manual

This documentation provides information about setting up the Kuksa dongle.

## Table of Contents
1. [Hardware Pre-requisites](#hardware-pre-requisites)
2. [OS Installation](#installation-of-os)  
 2.1 [Raspbian OS](#raspbian-os)  
 2.2 [Ubuntu OS](#ubuntu-os)
3. [OBD STN2120 Configuration](#obd-stn2120-configuration)
4. [Configure WiFi Access Point](#configure-wireless-access-point-interface)  
 4.1 [Hostapd Configuration](#hostapd-configuration)  
 4.2 [Dnsmasq Configuration](#dnsmasq-configuration)  
 4.3 [Wireless Interface and Static IP](#wireless-interface-and-static-ip)
5. [Troubleshooting](#troubleshooting)

## Hardware Pre-requisites

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

Once you have installed any Debian based operating system (e.g., Raspbian OS, or Ubuntu), you have to configure the ELM327 serial bus to read/write to CAN transceiver. On the Dongle, you have to establish a connection to the CAN transceiver (e.g., using the "screen" command line tool - use "sudo apt get install screen" if not installed). Via "dmesg" you can check for the CAN transceiver device (typically */dev/ttyAMA0*).
Once connected, you need to configure the ELM327 baud rate. You can check the entire [STN2120 Chip Manual](https://www.scantool.net/scantool/downloads/98/stn11xx21xx_frpm-c.pdf) for further configuration options.

```sh
# connect to the ELM327 serial bus using 'screen'
kuksa@pi:~$ sudo screen /dev/ttyAMA0 9600
> STSBR 2000000     # configure the baud rate to 2 MBit/s in RAM
> "OK"              # prompts OK
> <CTRL + K>        # exit screen or CTRL + ALT + DEL to reboot the Dongle
# connect again to persist the config changes
kuksa@pi:~$ sudo screen /dev/ttyAMA0 2000000
> STWBR             # persist baud rate in ROM -> there will be NO response
> <CTRL + K>        # exit screen or CTRL + ALT + DEL to reboot the Dongle
```

You are DONE. Your CAN transceiver is configured. In order to test your CAN communication, you can create a Python script to test the CAN communication (e.g., https://python-obd.readthedocs.io/en/latest/). Please check the python-obd manual to check the properties to set to connect to the serial bus and configure the new baud rate.

```python
# pip3 install obd
# python3 python-obdtest.py
import obd

connection = obd.OBD(portstr="/dev/ttyAMA0", baudrate=2000000) # create connection

cmd = obd.commands.SPEED # select an OBD command (sensor)

response = connection.query(cmd) # send the command, and parse the response

print(response.value) # returns unit-bearing values thanks to Pint
print(response.value.to("mph")) # user-friendly unit conversions
```

We suggest to use the FreeMatics OBD-II Emulator Mk2: https://freematics.com/products/freematics-obd-emulator-mk2/  to send CAN messages to the Kuksa Dongle which you should receive on the Python script.

## Configure Wireless Access Point Interface

In order to be able to access in-vehicle data in a real deployment of the Dongle, wireless connectivity is required.
The simplest way to achieve such functionality is to configure a wireless network interface to act as an access point using WiFi.

#### Why do you need a WiFi USB Dongles and which one?

As the Raspberry Pi Compute Module 3 does not provide a WiFi card onboard, you have to order a **mini** WiFi USB dongle (small form factor to fit into the Kuksa Dongle housing). Not all dongles work out of the box with Ubuntu due to missing support of the chip set in the Linux kernel. We tested a couple of Dongles (in 2020/2021) and had good experience with WiFi Dongles equipped with a **Ralink RT5370** chip (e.g., [EDUP Nano Adapter EP at Amazon](https://www.amazon.de/-/en/gp/product/B00BXWU7X4/ref=ppx_yo_dt_b_asin_title_o00_s00?ie=UTF8&psc=1)).

### Pre-requisites

To configure a access point on the Kuksa Dongle, we need three features: (i) tool to switch a Wifi PHY/MAC into access point mode - here using **hostapd**, (ii) a tool to act as a DHCP server and route traffic on this interface accordingly - here using **dnsmasq**, and (iii) configuring a wireless network interface in you network manager (e.g., using netplan or systemd-networkd). In this documentation we are using **systemd-networkd** which is more stable and simple to configure the network interface compared to netplan.

Install the tools via command line:

```sh
ubuntu@ubuntu:~$ sudo apt install hostapd dnsmasq
```

**Attention: When you have set up a wireless interface previously to operate as a client (cf. [Configure Network Interfaces in Ubuntu](#configure-network-interfaces-in-ubuntu)), you have to ensure that you are configuring the right interface to act as a WiFi access point. In case it should be the previously configured *wlan0* interface, you have to delete the lines configured */etc/netplan/50-cloud-init.yaml* file accordingly. If you want to configure an additional one, make sure you are configuring the right interface in the upcoming subsections.**

### Hostapd Configuration

Hostapd allows us to switch a WiFi card or WiFi USB dongle into the access point mode and to configure access point related information such as the SSID or the authentication password. After installing the tool, you have to provide a configuration for your access point. Write the following lines in the configuration file:

```sh
ubuntu@ubuntu:~$ sudo nano /etc/hostapd/hostapd.conf
interface=<INTERFACE>
driver=nl80211
ssid=<YOUR_SSID>
hw_mode=g
channel=7
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=<YOUR_PASSWORD>
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```
where *<INTERFACE>* is the network interface to switch into access point mode (e.g., wlan0), *<YOUR_SSID>* is the name of your wireless network (e.g., KuksaWifi), and *<YOUR_PASSWORD>* is the password used for authentication (make sure that your passphrase is at least 8 characters long).

You can check your configuration file using the following command:

```sh
ubuntu@ubuntu:~$ sudo hostapd -dd /etc/hostapd/hostapd.conf
```

After your configuration has been validated, we have to tell the hostapd systemd service which configuration file to be used (default is none - so change the line starting with *#DAEMON_CONF*) as well as enable the service at boot phase of the system.

```sh
ubuntu@ubuntu:~$ sudo nano /etc/default/hostapd
...
DAEMON_CONF="/etc/hostapd/hostapd.conf"
...
ubuntu@ubuntu:~$ sudo systemctl unmask hostapd
ubuntu@ubuntu:~$ sudo systemctl enable hostapd
ubuntu@ubuntu:~$ sudo systemctl start hostapd
```

When rebooting your system, you see your configured access network in the list of available WiFi networks (e.g., Laptop or SmartPhone). However, your device gets no IP address assigned so far - this is handled by *dnsmasq* (see below).

### Dnsmasq Configuration

Dnsmasq sets up a DHCP server and a router so devices connecting to the Kuksa Dongle can get an IP address assigned to it.

First, we recommend to make a backup of the default configuration file. After that, provide the inputs of your configuration and reload and enable the tool using systemctl.

By providing the inputs, you can un-comment the predefined entries in the file according to your setup. For a simple setup, the following lines are sufficient:

```sh
ubuntu@ubuntu:~$ sudo cp /etc/dnsmasq.conf /etc/dnsmasq.conf.BACKUP   # make a backup of the default conf
ubuntu@ubuntu:~$ sudo nano /etc/dnsmasq.conf
...
# to manage only WLAN0 interface - make sure the name of the interface is correct
interface=wlan0
# to assign addresses between 192.168.4.2 and 192.168.4.20 with netmask 255.255.255.0 and lease time 24 hours
dhcp-range=192.168.4.2,192.168.4.20,255.255.255.0,24h
...
```
On system startup, dnsmasq will not wait for the configured network interface (e.g., wlan0) to initialize and will fail with error "<INTERFACE> not found".

We need to configure the systemd service to launch after the network is ready. So you have to modify dnsmasq.service file by adding the right target under the *[Unit]* section.

```sh
ubuntu@ubuntu:~$ sudo nano /lib/systemd/system/dnsmasq.service
...
[Unit]
...
After=network-online.target
Wants=network-online.target
...
ubuntu@ubuntu:~$ sudo systemctl reload dnsmasq	# reload dnsmasq config
ubuntu@ubuntu:~$ sudo systemctl enable dnsmasq
```

### Wireless Interface and Static IP

Ubuntu uses by default cloud-init for the initial setup. But the configuration is not that simple/intuitive compared to systemd-networkd. So we are using this network management to configure the wireless network interface. A proper documentation of systemd-networkd can be found on the [Systemd-Networkd ArchLinux Website](https://wiki.archlinux.org/index.php/systemd-networkd).

In order to connect to a wireless network with **systemd-networkd**, a wireless adapter configured is required.

```sh
ubuntu@ubuntu:~$ sudo nano /etc/systemd/network/25-wireless.network
[Match]
Name=wlan0					      # the name of your interface you want to configure

[Network]
DHCP=no						        # 'no' as managed by dnsmasq
Address=192.168.4.1/24		# assign static IP address, link to dnsmasq

ubuntu@ubuntu:~$ sudo systemctl reload dnsmasq	# reload dnsmasq config
ubuntu@ubuntu:~$ sudo systemctl enable dnsmasq
```

After the creating the configuration, you have to reload the service and maybe also enable it to be executed after a reboot of the system.

You are done and should be able to connect another device to the Kuksa Dongle.

After rebooting your system, it could be possible that systemd-networkd is waiting for the network to be initialized, resulting in longer bootup time (message something like "a start job is waiting until the network is configured"). Different proposals to overcome the limitation are described on a [StackOverflow Thread](https://askubuntu.com/questions/972215/a-start-job-is-running-for-wait-for-network-to-be-configured-ubuntu-server-17-1).

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
