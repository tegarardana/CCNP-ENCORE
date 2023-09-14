# First Hop Redundancy Protocol

## Introduction to Gateway Redundancy

In this lesson we’ll take a look at different protocols for **gateway redundancy.** So what is gateway redundancy and why do we need it? Let’s start with an example!

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/gateway-redundancy-scenario.png" alt=""><figcaption></figcaption></figure>

The network in the picture above is fairly simple. I have one computer connected to a switch. In the middle you’ll find two multilayer switches (SW1 and SW2) that both have an IP address that could be used as the default gateway for the computer. Behind SW1 and SW2 there’s a router that is connected to the Internet.

Which gateway should we configure on the computer? SW1 or SW2? You can only configure a one gateway after all…

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/gateway-redundancy-crashed-switch.png" alt=""><figcaption></figcaption></figure>

If we pick SW1 and it crashes, the computer won’t be able to get out of its own subnet because it only knows about **one** default gateway. To solve this problem we will create a **virtual gateway**:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/virtual-gateway.png" alt=""><figcaption></figcaption></figure>

Between SW1 and SW2 we’ll create a virtual gateway with its own IP address, in my example this is 192.168.1.3.

The computer will use 192.168.1.3 as its default gateway. One of the switches will be the active gateway and in case it fails the other one will take over.

There are three different protocols than can create a virtual gateway:

* [**HSRP (Hot Standby Routing Protocol)**](https://networklessons.com/cisco/ccnp-encor-350-401/hsrp-hot-standby-routing-protocol)
* [**VRRP (Virtual Router Redundancy Protocol)**](https://networklessons.com/cisco/ccnp-encor-350-401/vrrp-virtual-router-redundancy-protocol)
* [**GLBP (Gateway Load Balancing Protocol)**](https://networklessons.com/cisco/ccnp-encor-350-401/glbp-gateway-load-balancing-protocol)

In the next lessons I will explain each of these protocols and show you how to configure them. For now, I hope this lesson has helped to understand why we need a virtual gateway in the network.
