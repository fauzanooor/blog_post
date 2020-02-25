---
layout: post
title:  "IPSEC S2S From Azure Stack to Mikrotik"
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

    -   Local Network Gateway : choose your LNG that created in the step
        3

    -   Shared key (also known as Pre-Shared Key (PSK)) : 5z4ZNenz8Mkk

![Connection](https://github.com/fauzanooor/fauzanooor.github.io/raw/master/images/Connection.PNG)

1.  Just that, and let’s move on to Mikrotik Site

    -   Also you can check to public IP that used by VNG, it should be
        appear now. And usually azure stack will use xxx.xxx.xxx.**2**
        for all VNGs

How to - Mikrotik Site
======================

1.  Create IPSec policy, and here’s the example parameters :

    -   Src. address (Local network that used by Mikrotik site) :
        192.168.100.0/24

    -   Dst. address (Local network that used by Azure Stack site) :
        10.0.0.0/24

    -   IPsec protocols : esp

    -   SA src. address (Public IP that used by Mikrotik VPN) :
        xxx.xxx.xxx.188

    -   SA dst. address (Public IP that used by VNG) : xxx.xxx.xxx.2

    -   Proposal : default

![ipsec-policy-general](https://github.com/fauzanooor/fauzanooor.github.io/raw/master/images/ipsec-policy-general.PNG)
![ipsec-policy-action](https://github.com/fauzanooor/fauzanooor.github.io/raw/master/images/ipsec-policy-action.PNG)

1.  Create policy proposal, and here’s the parameters (should be same) :

    -   Auth. algorithm : SHA26

    -   Encr. algorithm : AES-256 GCM

    -   Lifetime : 07:30:00

    -   PFS Group : none

![proposal](https://github.com/fauzanooor/fauzanooor.github.io/raw/master/images/proposal.PNG)

1.  Create peer, and here’s the parameters (should be same) :

    -   Address (Public IP that used by Azure Stack’s VNG) :
        xxx.xxx.xxx.2

    -   Profile : default

    -   Auth. method : pre shared key

    -   Exchange mode : IKE2

    -   Secret (shared key) : 5z4ZNenz8Mkk

![peer](https://github.com/fauzanooor/fauzanooor.github.io/raw/master/images/peer.PNG)

1.  Create peer profile, and here’s the parameters (should be same) :

    -   Hash algorithm : SHA256

    -   Encryption algorithm : AES-256

    -   DH group : modp1024

    -   Proposal check : obey

    -   Lifetime : 08:00:00

    -   Lifebytes : 33553408

![peer-profile-2](https://github.com/fauzanooor/fauzanooor.github.io/raw/master/images/peer-profile-2.PNG)

1.  After that the VPN IPSec status it’s should be also *established*,
    and if we check from the Mikrotik’s logs, there is *peer authorized*
    log appear, like these screenshots below.

![established](https://github.com/fauzanooor/fauzanooor.github.io/raw/master/images/established.PNG)
![peer-authorized](https://github.com/fauzanooor/fauzanooor.github.io/raw/master/images/peer-authorized.PNG)
\* Also, the connection status on azure stack site it should be change
to *connected*

1.  And the final step is create NAT rule, and here’s the example
    parameters :

    -   Chain : srcnat

    -   Src. address (Local network that used by Mikrotik site) :
        192.168.100.0/24

    -   Dst. address (Local network that used by Azure Stack site) :
        10.0.0.0/24

    -   Action : accept

![NatRule1](https://github.com/fauzanooor/fauzanooor.github.io/raw/master/images/NatRule1.PNG)
![NatRule2-1](https://github.com/fauzanooor/fauzanooor.github.io/raw/master/images/NatRule2-1.PNG)

UPDATE!!
========

-   If you have an issue with you cannot access SSH or etc via this
    IPSec S2S, maybe issue came from the MTU, which is booth MTU value
    from Azure Stack VNG and VMs behind it are not same. And the
    solution is you can change the MTU on the VM(s) to 1350.

-   Or if you have many VM(s), you can setup mangle rule from Mikrotik.
    And here’s script
    ```ip firewall mangle add action=change-mss chain=forward new-mss=1350 out-interface=ether1 passthrough=yes protocol=tcp tcp-flags=syn tcp-mss=1351-65535```
    ```ip firewall mangle add action=change-mss chain=forward in-interface=ether1 new-mss=1350 passthrough=yes protocol=tcp tcp-flags=syn tcp-mss=1351-65535```

References that might/should be usefull
=======================================

-   <https://docs.microsoft.com/en-us/azure/azure-stack/azure-stack-vpn-gateway-settings>

-   <https://blogs.technet.microsoft.com/netgeeks/2017/07/11/creating-a-site-to-site-vpn-ipsec-ikev2-with-azure-and-mikrotik-routeros/>

-   <https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-about-vpn-devices>

-   <http://www.themightybinary.com/blog/mikrotik-mtu-and-tcp-mss/>


