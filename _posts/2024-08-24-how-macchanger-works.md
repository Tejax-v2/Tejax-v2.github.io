---
title: "Magic of Macchanger: How MAC Address is changed"
date: 2024-09-24 +0530
categories: [How things work]
tags: [hardware,network] # TAG names should always be lowercase
---

## What is MAC Address

A MAC Address(Media Access Control) is a 48 bit long number uniquely representing a network card on your device.
It is used while transferring data over any network technology, like Ethernet, Wifi, Bluetooth, etc.
This number is hardcoded on all the network interface cards on your device
It is generally represented in the format XX:XX:XX:XX:XX:XX, where each X represents a hexadecimal digit.

For example, dc:21:48:56:9f:2a

This number is divided into two sections:
- Manufacture ID
  - First 3 bytes represent Manufacturer ID, for the company who manufactured that NIC(Network Interface Card) Card 
- Device ID
  - Remaining 3 bytes represent Device ID, which manufacturer has assigned to uniquely identify the card

For the given example,
<table>
  <thead>
    <tr>
      <th style="text-align: center;">Manufacturer ID</th>
      <th style="text-align: center;">Device ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align: center;">dc:21:48</td>
      <td style="text-align: center;">56:9f:2a</td>
    </tr>
  </tbody>
</table>


## How to use macchanger

Many tutorials can be found for this online, so I'm showing only one example

For linux:
- Update the packages and install macchanger: sudo apt update && sudo apt install macchanger -y
- Find your interface name (generally eth0 or wlan0): ifconfig
- To check your current mac address: macchanger -s *interface*
- Free up the network interface: sudo ifconfig *interface* down
- Set the new mac address to a random value: sudo macchanger -r *interface*
- Enable the network interface again: sudo ifconfig *interface* up
- Check the new mac address: macchanger -s *interface*

The mac address will have been changed.

> **NOTE**: 'sudo' is used before some commands because they require root privileges to run

## How macchanger works: The magical part

When the system boots up, the kernel is loaded into memory.
During this booting process, all the hardware checks are performed.
Network configurations are loaded into a Network Stack in the memory.
The hardcoded mac address is read from the NIC card and stored in the Network Stack.

The hardcoded MAC address is read only once, while booting. This is to reduce the redundant task of fetching MAC address from hardware everytime.
Once the system is loaded, whenever the device needs to communicate over the network, it uses this software-copy of MAC Address stored in the Network Stack Memory.

### What macchanger does

macchanger can never alter the hardcoded MAC address of network card.
It "spoofs" the mac address by changing the value of MAC address stored in the Network Stack Memory. So whenever the device communicates over the network, the modified value of mac address is used.
This creates an illusive picture that the MAC address of the device has been changed.

## Drawbacks

Whenever the mac address is changed using macchanger, the value is changed temporarily, since it is stored in the RAM.
So **whenever system is rebooted or network card is reset, the memory regains the original value of mac address.**

So, some people run macchanger in a startup script to change the mac address right after the system boots.

> **NOTE**: Network Card and NIC Card mean the same.