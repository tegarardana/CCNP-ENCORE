# Wireless LAN 802.11 Service Sets

Like wired networks, wireless networks have different physical and logical topologies. The 802.11 standard describes different **service sets**.  A service set describes how a group of wireless devices communicate with each other.

Each service set uses the **Same Service Set Identifier (SSID)**. The SSID is the “friendly” name of the wireless network. It’s the wireless network name you see when you look at available wireless networks on your wireless device.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/11/wireless-ssids-300x251.jpg" alt=""><figcaption></figcaption></figure>

In this lesson, I’ll explain the different service sets, and we’ll take a look at some other common AP modes.

## IBSS

With an **Independent Basic Service Set (IBSS)**, two or more wireless devices connect **directly** without an access point (AP). We also call this an ad hoc network. One of the devices has to start and advertise an SSID, similar to what an AP would do. Other devices can then join the network.

An IBSS is not a popular solution. You could use this if you want to transfer files between two or more laptops, smartphones, or tablets without connecting to the wireless network that an AP provides.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/11/wireless-lan-topology-ibss.png" alt=""><figcaption></figcaption></figure>

## Infrastructure Mode

With infrastructure mode, we connect all wireless devices to a central device, the AP. All data goes through the AP. The 802.11 standard describes different service sets. Let’s take a look.

### Basic Service Set (BSS)

With a Basic Service Set (BSS), wireless clients connect to a wireless network **through an AP**. A BSS is what we use for most wireless networks. The idea behind a BSS is that the AP is responsible for the wireless network.

Each wireless client advertises its capabilities to the AP, and the AP grants or denies permission to join the network. The BSS uses a single channel for all communication. The AP and its wireless clients use the same channel to transmit and receive.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/11/wireless-lan-topology-bss-overview.png" alt=""><figcaption></figcaption></figure>

The SSID is the “nice” name of the wireless network, and it **doesn’t have to be unique**.

The AP also advertises the **Basic Service Set Identifier (BSSID).** This is the MAC address of the AP’s radio, a **unique address that identifies the AP**. All wireless clients have to connect to the AP. This means the AP’s signal range defines the size of the BSS. We call this the **Basic Service Area (BSA)**.

In the picture above, the BSA is a beautiful circle. This might be the case if you install your AP somewhere in the middle of a meadow with nothing around the AP. In a building, the BSA probably looks more like this:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/11/wireless-lan-topology-bss-coverage.png" alt=""><figcaption></figcaption></figure>

When a wireless device wants to join the BSS, it sends an **association request** to the AP. The AP either permits or denies the request. When the wireless device has joined the BSS, we call it a **wireless client** or **802.11 station (STA)**.

All traffic from a wireless client has to go through the AP even if it is destined for another wireless client.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/11/wireless-lan-topology-bss.png" alt=""><figcaption></figcaption></figure>

Everything has to go through the AP because the AP is our central point for management, and it limits the size of the BSS. The AP’s signal range defines the **boundary of the BSS**.

#### **Distribution System (DS)**

A BSS is a standalone network with a single AP. In the pictures above, there is no connection with a wired network.

Most wireless networks, however, are an extension of the wired network. An AP supports both wired and wireless connections. The 802.11 standard calls the upstream wired network the **distribution system (DS)**.

The AP bridges the wireless and wired L2 Ethernet frames, allowing traffic to flow from the wired to the wireless network and vice versa.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/11/wireless-lan-topology-distribution-system.png" alt=""><figcaption></figcaption></figure>

We can also do this with VLANs. The AP connects to the switch with an 802.1Q trunk. Each **SSID maps to a different VLAN**:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/11/802-11-bss-ds-vlan-trunk.png" alt=""><figcaption></figcaption></figure>

Each wireless network has a unique BSSID. The BSSID is based on the MAC address, so most vendors (including Cisco) increment the last digit of the MAC address to create a unique BSSID.

Even though we have multiple wireless networks, they all use the same underlying hardware, radios, and channels. If you have an AP with multiple radios, then it’s possible to assign wireless networks to different radios. For example, you could use one wireless network on the 2.4 GHz radio and another one on the 5 GHz radio.

### Extended Service Set (ESS)

A BSS uses a single AP. This might not be enough because of two reasons:

* **Coverage**: A single AP’s signal can’t cover an entire floor or building. You need multiple APs if you want wireless everywhere.
* **Bandwidth**: An AP uses a single channel, and wireless is half-duplex. The more active wireless clients you have, the lower your throughput will be. This also depends on the data rates you support. A wireless client that sits on the border of your BSA might still be able to reach the AP, but can only use low data rates. A wireless client that sits close to the AP can use high data rates. The distant wireless client will claim more “airtime,” reducing bandwidth for everyone.

To create a larger wireless network, we use multiple APs and connect all of them to the wired network. The APs work together to create a large wireless network that spans an entire floor or building. The user only sees a **single SSID,** so they won’t notice whether we use one or multiple APs. **Each AP uses a different BSSID,** so behind the scenes, the wireless client sees multiple APs it can connect to. We call this topology with multiple APs, an **Extended Service Set (ESS)**.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/11/802-11-service-set-ess-topology.png" alt=""><figcaption></figcaption></figure>

APs work together. For example, if you associate with one AP and you walk around the building, you won’t disconnect. The wireless client will automatically “jump” from one AP to another AP. We call this **roaming**. To make this a seamless experience, we need an overlap between APs.

Each AP offers its own BSS and uses a different channel to prevent interference between APs.

### Mesh Basic Service Set (MBSS)

If you want to provide a wireless network for a large area, like a city, then it’s not easy to connect each AP to a wired network.

Instead, you could build a mesh network, also known as a Mesh Basic Service Set (MBSS). With a mesh network, we **bridge wireless traffic from one AP to another**. Mesh APs usually have multiple radios. One radio is for backhaul traffic of the mesh network between APs; the other radio is to maintain a BSS for wireless clients on another channel.

At least one AP is connected to the wired network; we call this the Root AP (RAP). The other APs are Mesh APs (MAP) and are only connected through the wireless backhaul.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/11/wireless-lan-topologies-mesh-network-mss.png" alt=""><figcaption></figcaption></figure>

There are multiple paths for a MAP to reach the wired network through the RAP, so we need a protocol that finds the best loop-free path. Similar to how spanning-tree works for L2 or routing protocols for L3, there are different wireless solutions. IEEE has the 802.11s standard for mesh networks. Vendors sometimes also use proprietary solutions. For example, Cisco has the Adaptive Wireless Path Protocol (AWPP).

Cisco APs support both indoor and outdoor mesh networks.

## AP Modes

Thus far, we have only talked about service sets. Some APs also support different non-infrastructure modes. I’ll explain the most common AP modes below.

### Repeater

If you need to cover a large area with your wireless network, you usually create an ESS. An ESS, however, requires wired connections. If it’s impossible to connect your AP with a wire, you could configure an AP in repeater mode.

A wireless repeater receives a signal and retransmits it. This allows wireless devices that are not close enough to the AP to join the network.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/11/ap-modes-repeater.png" alt=""><figcaption></figcaption></figure>

There must be an overlap between the cell size of the AP and the repeater. For optimal performance, it should be  about 50%. If the repeater has a single radio, then it will receive and transmit on the same signal as the AP. In this case, the AP will also receive the retransmitted signal. Since wireless is half-duplex, adding a repeater will reduce your available throughput by about 50%.

To work around this, some repeaters have two or more radios. They receive on one channel (same as the AP) and retransmit on another.

### Workgroup Bridge

What if you have a wired device that needs to connect to a wireless network but doesn’t have a radio? For example, older printers, computers, or point of sale (PoS) systems. In this case, you can use a workgroup bridge (WGB). The WGB has a wired connection you connect to the wired device and a wireless connection, which it uses to act as a wireless client of a BSS.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/11/ap-modes-workgroup-bridge.png" alt=""><figcaption></figcaption></figure>

There are two types of WGBs:

* Universal workgroup bridge (uWGB): Universal WGB only supports a single wired client. This is based on the 802.11 standards.
* Workgroup Bridge (WGB): WGB (or Workgroup Bridge Mode) is a Cisco proprietary extension to the 802.11 standards and supports multiple wired clients.

### Outdoor Bridge

What if you want to connect two buildings, but there is no cable in between, and you don’t want to use a WAN? You could use an outdoor wireless bridge. You can configure two APs to create a wireless bridge between two LANs over a longer distance. Wireless bridges between two buildings, and even between two cities are possible.

There are two options:

* Point-to-point
* Point-to-multipoint

Let’s start with point-to-point:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/11/wireless-bridge-two-buildings.png" alt=""><figcaption></figcaption></figure>

We have two buildings, each with a LAN.  The APs are in bridge mode and use directional antennas that focus their signal in one direction, towards the AP on the other side.

{% hint style="info" %}
If you wonder what the maximum distance of a wireless bridge could be: The record for the longest wifi link is from [CISAR (Italian Center for Radio Activities)](http://www.cisar.it/images/2016/27HighCapacityLong.pdf). It’s 304 kilometers (188 miles).
{% endhint %}

If you want to bridge more than two LANs, you could use a point-to-multipoint bridge:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/11/wireless-bridge-three-buildings.png" alt=""><figcaption></figcaption></figure>

LAN 1 and 3 use APs with directional antennas. LAN 2 in the middle uses an omnidirectional antenna so that the signal is transmitted equally in both directions.

## Conclusion

You have learned about 802.11 service sets and some common non-infrastructure AP modes.

* A service set describes how a group of wireless devices communicate together.
* Each service set uses an SSID. This is the name of the wireless network that we show to users.
* There are different service sets:
  * IBSS: An ad-hoc network for direct communication between wireless devices without an AP.
  * BSS: Network with an AP. all communication goes through the AP.
    * DS: The wired network that we connect to the wireless network.
      * Each VLAN maps to a different SSID.
      * We use an 802.1Q trunk between the AP and switch.
  * ESS: Wireless network with multiple APs. Required often because of two reasons:
    * A single AP can’t cover an entire floor or building.
    * Wireless is half-duplex so available bandwidth depends on the number and location of wireless devices.
  * MBSS: A mesh network, useful when you can’t connect all APs to a wired network. The APs create a wireless backbone.
* Different AP modes:
  * Repeater: Repeats the signal from an AP, useful if you can’t use an ESS but should be avoided.
  * Workgroup Bridge: Allows you to connect a wired device as a client to a wireless network.
  * &#x20;Outdoor Bridge: useful if you want to connect remote sites to each other without using a WAN.

I hope you enjoyed this lesson! If you have any questions, please leave a comment.
