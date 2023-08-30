# Multicast IP Address

One of the differences between unicast and multicast IP addresses is that unicast IP addresses represent a single network device while multicast IP addresses represent a group of receives. IANA has reserved the class D range to use for multicast. The first 4 bits in the first octet are **1110** in binary, meaning we have the 224.0.0.0 through 239.255.255.255 range for IP multicast addresses.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/03/multicast-class-d-range.png" alt=""><figcaption></figcaption></figure>

Some of the addresses are reserved, however, and we can’t use them for our own applications.

The 224.0.0.0 – 224.0.0.255 range has been reserved by IANA to use for network protocols. All multicast IP packets in this range are **not forwarded** by routers between subnets. Let me give you an overview of reserved link-local multicast addresses. I’m sure you recognize some of the protocols:

<table><thead><tr><th width="213">Address</th><th width="176">Usage</th></tr></thead><tbody><tr><td>224.0.0.1</td><td>All Hosts</td></tr><tr><td>224.0.0.2</td><td>All Multicast Routers</td></tr><tr><td>224.0.0.3</td><td>Unassigned</td></tr><tr><td>224.0.0.4</td><td>DVMRP Routers</td></tr><tr><td>224.0.0.5</td><td>OSPF Routers</td></tr><tr><td>224.0.0.6</td><td>OSPF DR/BDR Router</td></tr><tr><td>224.0.0.7</td><td>ST Routers</td></tr><tr><td>224.0.0.8</td><td>ST Hosts</td></tr><tr><td>224.0.0.9</td><td>RIPv2 Routers</td></tr><tr><td>224.0.0.10</td><td>EIGRP Routers</td></tr><tr><td>224.0.0.11</td><td>Mobile Agents</td></tr><tr><td>224.0.0.12</td><td>DHCP Server / Relay</td></tr><tr><td>224.0.0.13</td><td>All PIM Routers</td></tr><tr><td>224.0.0.14</td><td>RSVP Encapsulation</td></tr><tr><td>224.0.0.15</td><td>All CBT Routers</td></tr><tr><td>224.0.0.16</td><td>Designated SBM</td></tr><tr><td>224.0.0.17</td><td>All SBMS</td></tr><tr><td>224.0.0.18</td><td>VRRP</td></tr><tr><td>224.0.0.19 – 255</td><td>Unassigned</td></tr></tbody></table>

You probably recognized OSPF (224.0.0.5 and 224.0.0.6), RIPv2 (224.0.0.9), and EIGRP (224.0.0.10).  Once you dive more into multicast, you will also encounter PIM (Protocol Independent Multicast) with 224.0.0.13.

IANA also reserved the 224.0.1.0 /24 range for certain applications. Everything in the 224.0.1.0 /24 range can be routed, however, unlike the 224.0.0.0 /24 range. Here’s an overview:

<table><thead><tr><th width="213">Address</th><th width="176">Usage</th></tr></thead><tbody><tr><td>224.0.1.0</td><td>VMTP Managers Group</td></tr><tr><td>224.0.1.1</td><td>NTP</td></tr><tr><td>224.0.1.2</td><td>SGI-Dogfight</td></tr><tr><td>224.0.1.3</td><td>Rwhod</td></tr><tr><td>224.0.1.6</td><td>NSS</td></tr><tr><td>224.0.1.8</td><td>SUN NIS+</td></tr><tr><td>224.0.1.20</td><td>Any Private Experiment</td></tr><tr><td>224.0.1.21</td><td>DVMRP on MOSPF</td></tr><tr><td>224.0.1.32</td><td>Mtrace</td></tr><tr><td>224.0.1.33</td><td>RSVP-encap-1</td></tr><tr><td>224.0.1.34</td><td>RSVP-encap-2</td></tr><tr><td>224.0.1.39</td><td>Cisco-RP-Announce</td></tr><tr><td>224.0.1.40</td><td>Cisco-RP-Discovery</td></tr><tr><td>224.0.1.52</td><td>Mbone-VCR-Directory</td></tr><tr><td>224.0.1.78</td><td>Tibco Multicast 1</td></tr><tr><td>224.0.1.79</td><td>Tibco Multicast 2</td></tr><tr><td>224.0.1.80 – 224.0.1.255</td><td>Unassigned</td></tr></tbody></table>

Many of these applications are never used. If you work with Cisco multicast, you will see 224.0.1.39 and 224.0.1.40 when configuring an RP (Rendezvous Point).

Make sure you don’t use the 224.0.0.0 /24 and 224.0.1.0 /24 range, and you will be safe. Like private and public IP addresses for unicast, IANA has reserved a range of IP addresses that we can use for multicast on our local networks. This is the 239.0.0.0 /8 range. Everything between 239.0.0.0 – 239.255.255.255 is **safe to use** on your own networks.
