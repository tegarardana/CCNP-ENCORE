# Infrastructure

## Introduction to Wireless LAN

Nowadays, there are about more wireless than wired devices. About 10 years ago, most offices only had desktops and some other network devices like printers. All of these were connected with wires. Today, a lot of users have a laptop, smartphone, and tablet. That’s three wireless devices for each user. Wireless speeds have increased significantly, getting close to wireless Gigabit.

IEEE uses 802.11 for all protocols that are related to wireless. Most of us have seen or heard about 802.11a, 802.11b, 802.11g, 802.11n and/or 802.11ac before. We also have the Wi-Fi Alliance that helps with the promotion of wireless networking. For example, IEEE has described authentication and encryption in their 802.11i standard. The Wi-Fi alliance has based WEP, WPA, and WPA2 on this standard. These names are easier to work with than referring to 802.11i.

### SOHO Wireless LAN

Your router at home probably has the same capabilities as the one below:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/12/soho-router.png" alt=""><figcaption></figcaption></figure>

It is connected to your ISP through cable or DSL, or perhaps fiber. It has some Ethernet ports to connect your computers and it has antennas for wireless users. In reality, these components are all built into one device:

* Ethernet Switch
* Wireless Access Point
* (Cable or DSL Modem)
* Router

If you take everything apart, it looks like this:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/12/soho-router-roles.png" alt=""><figcaption></figcaption></figure>

In small networks like this, the AP does everything by itself. We call this an **autonomous access point**. It uses 802.11 protocols to talk with the wireless clients and uses Ethernet on the LAN side.

### Enterprise Wireless LAN

When we look at large Enterprise networks, a single access point is not enough. Imagine a network with hundreds or thousands of users. When you walk around the office, you don’t want to get disconnected every time when your phone switches from one access point to another. You want to have a stable wireless connection, wherever you go. Switching seamlessly from one access point to another is called **roaming**.

A single access point also has limited bandwidth. If you have a meeting room with 100 users then a single access point might be unable to provide enough bandwidth for everyone.

Since we use wireless networking for our users, it has to be close to our users. That’s why you will find access points on the access layer of your network, just like your computers and printers:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/12/enterprise-network-wireless-access-points.png" alt=""><figcaption></figcaption></figure>

There’s still one issue. Let’s say you are connected to an access point and you start walking around the office, your phone will switch to another access point. How does this second access point know that you are already authenticated to the network? You could re-authenticate but that will break your connection…not a good idea. With autonomous access points, roaming could be difficult. To guarantee seamless roaming, the access points need to exchange authentication information of a client that roams from one access point to another.

Nowadays, most wireless networks use a wireless LAN controller:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/12/enterprise-network-wireless-controller.png" alt=""><figcaption></figcaption></figure>

All management tasks are moved from the access points to the wireless LAN controller. It takes care of authentication, roaming, creating new wireless networks, etc. The access points are only responsible for forwarding traffic, we call these **LWAPs (Light Weight Access Point).**

To achieve this, all traffic has to be sent from the access points to the wireless LAN controller. This is done by tunnels called **CAPWAP (Control And Provisioning of Wireless Access Points)**. The green dotted lines are the CAPWAP tunnels between the APs and WLC.

We now have one big wireless network. If you create a new wireless network (SSID) then it will be pushed to all access points. Roaming is also no problem since all traffic is forwarded to the WLC.

### Conclusion

You have now learned the basics of Wireless LANs:

* The SOHO router is a switch, router, access point and (optionally) modem all in one device.
* The access point is called an autonomous access point since it does everything. Wireless authentication, forwarding traffic between the LAN / Wireless LAN, etc.
* Enterprise wireless LANs use a WLC (Wireless LAN Controller) and LWAPs (Light-Weight Access Point).
* Management tasks like creating wireless networks, authentication and roaming is all done in the WLC. The LWAPs are responsible of forwarding wireless traffic to and from the LAN.
* The WLC and LWAPs allow us to create a big, scalable wireless network.
