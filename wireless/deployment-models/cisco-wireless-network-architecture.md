# Cisco Wireless Network Architecture

In the [802.11 service sets lesson](https://networklessons.com/cisco/ccnp-encor-350-401/wireless-lan-802-11-service-sets), you learned how an AP provides a BSS for wireless clients. You also learned how we use an ESS with multiple APs to create a larger wireless network.

In this lesson, we’ll take a look at different Cisco wireless architectures we can use for enterprise networks.

## Autonomous AP Architecture

We use most wireless networks as an extension of the wired network. Wireless and wired clients are on the same LAN and can communicate with each other.

An autonomous AP has **all the required intelligence** to serve wireless clients and to connect to the wired network. The AP can offer one or more BSSes and connect VLANs to SSIDs.

Below is a picture of what this looks like on an enterprise network:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/11/cisco-architecture-autonomous-ap-scaled.png" alt=""><figcaption></figcaption></figure>

I highlighted the connections for one of the APs. Our autonomous AP is on the access layer and connected with a trunk. We have three VLANs:

* VLAN 10: Management
* VLAN 20: for SSID “WIFI-VLAN20”
* VLAN 30: for SSID “WIFI-VLAN30”

We need the 802.1Q trunk to the switch; this is how the AP has access to all required VLANs. The autonomous AP also has a management IP address; we need this to SSH into the AP so we can configure things like:

* RF parameters:
  * Channel
  * Transmit power
* VLANs
* SSIDs

We want to separate management and data traffic, which is why we use a separate management VLAN. Wireless users in the same VLAN can communicate directly with each other; this traffic doesn’t have to cross the wired network.

You configure each autonomous AP individually. This can be a pain; let me explain.

Let’s say you want the capability that wireless users can roam from one AP to another without losing their connection and DHCP assigned IP address. This is no problem, but it means you have to configure the same SSID and VLAN on all APs.

You also need to configure RF parameters like the channel you want to use and the transmit power. With multiple APs, you have to figure out which channels and what transmit power to use, so there is enough overlap and (almost) no interference between APs. You also need to ensure that when an AP fails, other APs can take over, so there is no coverage hole.

Requiring VLANs everywhere introduces an issue on the switched network; stretched VLANs. Take a look at the following picture:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/11/cisco-wireless-architecture-autonomous-ap-stretched-scaled.png" alt=""><figcaption></figcaption></figure>

When a wireless user wants to roam from one AP to an AP that is connected to the access layer of another distribution layer, you’ll have to cross the core layer. This means your VLAN has to span the entire network.

If you want to configure a new SSID, you have to configure it on all APs. You also need to configure a new VLAN on all APs and switches.

You could make your life a bit easier and use a management platform to configure the APs. You could do this with Cisco Prime Infrastructure. There is also no central point in the network to monitor wireless traffic and do things like policing, QoS, or intrusion detection.

Another term for the autonomous AP solution is the local MAC architecture.

As you see, there are some limitations and problems with the autonomous AP setup. It’s a fine solution for a small network, but when the network grows, it doesn’t scale very well.

## Split-MAC Architecture

Autonomous APs work alone; we configure them one by one. As explained, this decentralized architecture has some drawbacks:

* If we want roaming, we need to configure VLANs and SSIDs on all APs.
* When we want a new SSID, we also have to create a new VLAN on all switches.
* VLANs can span the entire wired network.
* We need to configure the RF parameters manually.
* There is no central point in the network to do things with our wireless traffic like policing, QoS, or intrusion detection.

To overcome these problems, we **move some functions from the AP to a central location, the Wireless LAN Controller (WLC)**:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/11/cisco-wireless-architecture-lightweight-ap-wlc-scaled.png" alt=""><figcaption></figcaption></figure>

With wireless networking, we have **real-time and management functions**. The AP should handle real-time functions, but everything that is not delay-sensitive can do from a central location. We separate the following management and real-time functions of the AP:

* Management functions:
  * Client authentication
  * Security management
  * Association and reassociation (roaming)
  * Quality of Service (QoS)
* Real-time functions:
  * Transmission of 802.11 frames
  * MAC management
  * Encryption

Since these functions are not real-time, we can move them to a central point, the WLC. We take away some of the intelligence of the AP, which is why we call them lightweight APs (LAP). We move this intelligence to the WLC.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/11/cisco-architecture-wlc-ap-realtime-management-functions.png" alt=""><figcaption></figcaption></figure>

The WLC controls many APs in the network. The **lightweight AP requires a WLC;** it can’t function on its own. Splitting functions between the AP and WLC is what we call the split-MAC architecture.

One exception is the FlexConnect architecture. This AP connects to a WLC but can also work standalone.

When a lightweight AP boots, it uses discovery mechanisms to search and connect to a WLC. Before the AP and WLC connect, they have to authenticate each other. We do this with pre-installed X.509 certificates on the AP and WLC. This prevents someone from adding an unauthorized AP to your network.

### CAPWAP

The AP and WLC connect with a tunneling protocol, the **Control And Provisioning of Wireless Access Points** **(CAPWAP)** tunneling protocol. CAPWAP encapsulates all data between the lightweight AP and the WLC.

CAPWAP is a standard, defined in RFC 5415, 5416, 5417, and 5418. It’s based on the Lightweight Access Point Protocol (LWAPP), a legacy Cisco proprietary solution.

There are two tunnel types:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/11/cisco-wireless-architecture-capwap-control-data-tunnel.png" alt=""><figcaption></figcaption></figure>

* **CAPWAP control messages**: contain information about WLAN management. We use this to configure and manage the AP. These control messages are encrypted.
* **CAPWAP data messages**: encapsulate packets to and from wireless clients who are associated with the AP. These messages are unencrypted by default

Each uses a different UDP port.

Tunneled traffic can be **switched or routed.** Using a tunnel means the lightweight APs and WLC don’t have to be in the same VLAN. This is useful since APs are typically on the access layer, and the WLC is in a central location (core layer or data center attached to the core).

Because of the CAPWAP tunnel, the AP and WLC are not only physically separated but also **logically separated**. This requires some more explanation. Take a look at the following picture:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/11/wireless-architecture-capwap-tunnel-vlan-1.png" alt=""><figcaption></figcaption></figure>

The tunnel is between the IP address of the lightweight AP and the WLC. As long as they can reach each other, they can establish the tunnel. The lightweight AP connects to an access mode switchport in VLAN 11. The lightweight AP offers two SSIDs:

* WIFI-VLAN20: Uses VLAN 20
* WIFI-VLAN30: Uses VLAN 30

How can the lightweight AP offer an SSID that uses VLAN 20 or 30 while it’s connected to a switchport in VLAN 11? That’s what the CAPWAP tunnel is for; it tunnels VLAN 20 and 30 traffic from the lightweight AP to the WLC.

This means that the WLC requires access to all VLANs, so it has a trunk to the switch. The lightweight AP can be in any VLAN, as long as it can reach the WLC.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/11/cisco-wireless-wlc-multiple-capwap-tunnels.png" alt=""><figcaption></figcaption></figure>

Now with the WLC that performs management functions for all lightweight APs, we can do things we couldn’t do before with the autonomous APs. The WLC has access to RF information from all APs. Some of the things we can do are:

* **RF monitoring**: The WLC scans channels and monitors RF usage of all RPs. It uses this data to select the best channels, transmit power to use and to detect rogue APs.
* **Wireless Intrusion Protection System (WIPS)**: The WLC can monitor wireless client data to detect and prevent unauthorized network access
* **Security Management**: The WLC can authenticate wireless clients against an external (RADIUS) server and force clients to use an IP address from a trusted DHCP server.
* **Dynamic client load balancing**: When two APs are close to each other, the WLC can associate clients with the least used AP. This helps to distribute the load between APs.
* **Client roaming**: Wireless clients can roam between APs without any noticeable delay.
* **Dynamic channel assignment**: The WLC automatically chooses the best RF channel for each AP.
* **Transmit power optimization**: The WLC automatically sets the transmit power for each AP based on the required coverage area.
* **Self-healing wireless coverage**: When an AP radio dies, the WLC can increase the transmit power of surrounding APs to get rid of a coverage hole.

## Cloud-Based AP Architecture

We talked about local MAC architecture with autonomous APs and some of their shortcomings. We also talked about split-MAC architecture with lightweight APs and WLCs, and how it solves some of the local MAC architecture issues.

Lightweight APs require a WLC; without one, they can’t function. Because of the WLC, AP management is easy, but we still need to manage the WLC. Because the WLC is a critical device, you probably want a second one for redundancy reasons. Redundancy is excellent but also makes your configuration more complicated.

An alternative is a cloud-based AP architecture. With this architecture, the controller function is pushed into the cloud. Cisco offers this with its Meraki products.

Meraki offers cloud-based wireless, switched, and security products. When you power an AP, it will automatically connect to the cloud and configure itself. Through the Meraki cloud dashboard, you can do things like:

* Configure APs.
* Push code upgrades to APs.
* Monitor wireless performance.
* Generate reports.

Like a WLC, the cloud instructs each AP what channel and transmit power to use. It also collects RF parameters like interference, usage, rogue APs, etc.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/11/cisco-architecture-wireless-cloud-meraki-scaled.png" alt=""><figcaption></figcaption></figure>

Similar to split-MAC architecture, we have two traffic paths:

* **Control plane**: Configure, manage, and monitor APs.
* **Data plane**: Traffic to and from wireless clients.

We only use the cloud for control plane traffic. Data plane traffic remains on the local network and is not forwarded to the cloud. This means that each AP requires a trunk to the switch, similar to autonomous AP architecture.

## Conclusion

In this lesson, you learned about the different wireless architectures:

* Autonomous AP Architecture:
  * All intelligence is inside the AP.
  * We require a trunk from the switch to the AP.
  * Autonomous APs work standalone. We configure them one by one.
  * If you want to roam, you need to configure the VLANs you use for your SSIDs on all switches.
  * Stretched VLAN issues.
  * No central point for management, so it’s difficult to make central decisions for things like policing, RF management, etc.
* Split-MAC Architecture:
  * We move some intelligence from the AP to the WLC:
    * Real-time functions remain on the lightweight AP.
    * Management functions are on the WLC.
  * CAPWAP:
    * Tunnels all traffic from the lightweight AP to the WLC.
    * It uses a control and data tunnel, each with a different UDP port.
    * We can route or switch CAPWAP traffic.
      * This means the lightweight AP and WLC are both physically and logically separated.
    * The lightweight AP connects to a switch in access mode.
    * The WLC connects to a switch in trunk mode.
    * The WLC has access to all RF information so it can make decisions for RF management, security, monitoring, etc.
* Cloud-Based AP Architecture:
  * We move the management functions to the cloud.
  * Data remains local in the network, so we need a trunk from the switch to the AP.

I hope you enjoyed this lesson. Please leave a comment if you have any questions!
