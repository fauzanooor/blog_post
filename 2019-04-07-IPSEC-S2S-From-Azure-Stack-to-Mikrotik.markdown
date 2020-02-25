---
layout: post
title:  "IPSEC S2S - From Azure Stack to Mikrotik"
date:   2019-04-07
---
Requirements (As far I know ya hehe)
====================================

-   Any mikrotik device or appliance with ROS v6.43.2 or later

    -   If you are using v6.42.x, I really recommended to upgrade,
        because in my experience, this version have a lot of problems
        with IPSec tunnel.

-   Azure Stack version 1808 or later

How to - Azure Stack Site
=========================

1.  Create Subnet Gateway in the Virtual Network, and here’s the example
    parameters :

    -   Address space : 10.0.0.0/23

    -   Default subnet (that used by VM(s) : 10.0.0.0/24

    -   Gateway subnet : 10.0.1.0/27

![GatewaySubnet](https://github.com/fauzanooor/fauzanooor.github.io/raw/master/images/GatewaySubnet.PNG)

1.  Create Virtual Network Gateway (VNG), and for the type I think it’s
    no problem at all if you choose basic or higher, but in this
    tutorial I choose basic. (Also ignore the warning that said
    provisioning VNG up to 45 minutes. Because creating VNG in
    AzureStack is really really really faster than AzurePublic haha).

    -   FYI, Public IP is will not appear yet, until you create the
        connection between VNG and Local Network Gateway (LNG)

2.  Create Local Network Gateway, and here’s the example parameters :

    -   IP address (Public IP that used by Mikrotik VPN) :
        xxx.xxx.xxx.188

    -   Address space (local address that used by local network behind
        Mikrotik) : 192.168.100.0/24

![LNG](https://github.com/fauzanooor/fauzanooor.github.io/raw/master/images/LNG.PNG)

1.  Create connection, and here’s the example parameters:

    -   Virtual Network Gateway : choose your VNG that created in the
        step 2
