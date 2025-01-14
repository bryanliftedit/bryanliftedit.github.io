---
title: Aruba AP-IAP Conversion
date: 2024-06-20 12:00:00 +/-TTTT
categories: [Hardware,Aruba]
tags: [hardware,aruba]     # TAG names should always be lowercase
description: Aruba campus to instant access point conversion -or- cost effective enterprise class wifi for the home.
---

## Background

For the past several years I've used Ubiquiti gear for my home WiFi.  It has worked, mostly, but I have run into instances where it seems to hiccup and require us to reconnect or reboot the AP itself.  I also had some coverage issues that really required a second AP.  Call it prosumer if you want, but I've long considered ditching this in lieu of the actual enterprise gear I manage professionally daily.  Cost here has always been an issue.

Enter decommissioned (but still very capable) Aruba campus APs.

Historically, Aruba has split their WiFi hardware models into a couple SKUs. AP for campus access points that require a mobility controller, and IAP for instant access points that have the capability of propping up their own virtual controller.  The only difference between the two was capability by way of software lock as it all ran on the exact same hardware.

Several years ago, they finally decided to streamline their SKUs and offer a single option per model that was capable of doing either.  With that, they enabled a command in later firmwares for non-instant models to convert them.  The caveat here is that the command could only be issued from a mobility controller...officially.

I picked up a pair of Aruba AP315 campus APs for around $30 shipped, and the procedure below details how to convert them without the use of a mobility controller.

## Conversion

#### Prepare Beforehand


* Configure a TFTP server on the same network containing the relevant Aruba firmware for your model.  **You will need to be on 8.6 or newer firmware.**

* Connect a serial console cable from your laptop to the access point.
    * Depending on the model, this may require a special cable.  Aruba does sell one, but I happened to have [these](https://www.amazon.com/IZOKEE-CP2102-Converter-Adapter-Downloader/dp/B07D6LLX19/ref=sr_1_5?crid=102PLNI4PRBN1&dib=eyJ2IjoiMSJ9.VcBZTY7hnkmhHf7Flvr9wP1b_Nhx37X0fRb6JG7yQtIXU2cQH5oBa3iz0CpqNh3gWe6CtqqYoCpAVq6ttb1lC8f02IwFfw6k0iUbvSLL42yMQbYD0CLTZg-r830pPyJZrnCC0BpLEJbPqZq9kBn-XsAFrEnQXiv6XnV8UuEbjzDd1VNVv-wAX4brxUvlFUyOAxhmFoxCgM-fVWntzR8iBK0z69Sa9SnmACbc1mzOwjA.RBvp_74J7fYm7IkTrXLOh9ukGmaZOzwfznjdJs0plDY&dib_tag=se&keywords=usb+uart&qid=1723004615&sprefix=usb+uart%2Caps%2C99&sr=8-5) on hand.  The pinout - large notch on top, from left to right is:  GND, TX, RX, 3.3v - 9600 baud

* SHA1 hash of the string ```US-<serial number from your device>```
    * Example:  US-CNJ1389 =  5aacc69894c36ecf89c28a567eaa6df1fb5b03f3
    * [SHA1 Hash Calc](https://passwordsgenerator.net/sha1-hash-generator/)

#### Procedure

* Stop the apboot by pressing any key.
* Write country code
    * For US: ```proginv system ccode CCODE-US-<sha1_hash>```
    * For RW: ```proginv system ccode CCODE-RW-<sha1_hash>```
* Set flash writable: ```invent -w```
* Connect to network
    * For DHCP: ```dhcp```
    * For static: ```setenv ipaddr <ip>```
* Define TFTP server: ```setenv serverip <tftp_server_ip>```
* Write firmware to slot 0: ```upgrade os 0 <firmware_file_from_tftp>```
* Write firmware to slot 1: ```upgrade os 1 <firmware_file_from_tftp>```
* ```factory_reset```
* **Save the config!**: ```saveenv```
* ```reset```

The AP should reboot and you can then shortly browse to its webpage where it will ask you to configure a virtual instant controller.