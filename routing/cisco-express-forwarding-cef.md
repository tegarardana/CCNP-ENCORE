# Cisco Express Forwarding (CEF)

Perhaps you have heard about the term “wirespeed” before. It’s something the marketing department likes to use when it comes to selling networking equipment. It means that packets can be forwarded without any noticeable delay. Oh btw, for the remaining of this lesson the words “multilayer switch” and “router” are the same thing. Everything that I explain about the multilayer switches from now on also applies to routers.Let’s take a look at the difference between layer 2 and multilayer switches from the switch’s perspective:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/layer-2-vs-multilayer-switch.png" alt=""><figcaption></figcaption></figure>

You know that layer 2 switches only will switch Ethernet frames within a VLAN, and if we want we can filter traffic based on layer 2 (for example with port-security). The multilayer switch can do the same but is also able to route between VLANS and filter on layer 3 or 4 using access-lists.

Forwarding on layer 2 is based on the destination MAC address. Our switch learns the source MAC addresses on incoming frames and it builds the MAC address table. Whenever an Ethernet frame enters one of our interfaces, we’ll check the MAC address table to find the destination MAC address and we’ll send it out the correct interface.

Forwarding on layer 3 is based on the destination IP address. Forwarding happens when the switch receives an IP packet where the source IP address is in a **different subnet** than the destination IP address.

When our multilayer switch receives an IP packet with its **own MAC address as the destination** in the Ethernet header there are two possibilities:

* If the destination IP address is an address that is configured on the multilayer switch then the IP packet was destined for this switch.
* If the destination IP address is an address that is not configured on the multilayer switch then we have to act as a gateway and “route” the packet. This means we’ll have to do a lookup in the routing table to check for the **longest match**. Also we have to check if the IP packet is allowed if you configured an ACL.

Back in the days…switching was done at **hardware speed** while routing was done in **software**. Nowadays both switching and routing is done at hardware speed. In the remaining of this lesson you’ll learn why.

Let’s take a look at the difference between handling Ethernet Frames and IP Packets:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/mac-address-table-switch-headers.png" alt=""><figcaption></figcaption></figure>

The life of a layer 2 switch is simple:

1. The switch will verify the checksum of the Ethernet frame to make it sure it’s not corrupted or altered.
2. The switch receives an Ethernet frame and adds the source MAC address to the MAC address table.
3. The switch forwards the Ethernet frame to the correct interface if it knows the destination MAC address. If not, it will be flooded.

There is **no alteration** of the Ethernet frame!

Now let’s see what we have to do when we receive an IP packet on a multilayer switch:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/multilayer-switch-packet-forwarding.png" alt=""><figcaption></figcaption></figure>

In the example above H1 is sending an IP packet towards H2. Note that they are in different subnets so we will have to route it. When our multilayer switch receives the IP packet this is what will happen:

1. The switch will verify the checksum of the Ethernet frame to make it sure it’s not corrupted or altered.
2. The switch will verify the checksum of the IP packet to make it sure it’s not corrupted or altered.

The multilayer switch will check the routing table, notices that 192.168.20.0/24 is directly connected and the following will happen:

1. Check the ARP table to see if there’s a layer 2 to 3 mapping for H2. If there is no mapping the multilayer switch will send an ARP request.
2. The destination MAC address changes from FFF (Multilayer switch Fa0/1 ) to BBB (H2).
3. The source MAC address changes from AAA (H1) to GGG (Multilayer switch Fa0/2).
4. The TTL (time to live) field in the IP packet is decreased by 1 and because of this the IP header checksum will be recalculated.
5. The Ethernet frame checksum must be recalculated.
6. The Ethernet frame carrying the IP packet will be sent out of the interface towards H2.

As you can see there are quite some steps involved if we want to route IP packets.

When we look at multilayer switches there is a “separation of duties”. We have to build a table for the MAC addresses, fill a routing table, ARP requests, check if an IP packet matches an access-list etc and we need to forward our IP packets. These tasks are divided between the “**control plane**” and the “**data plane**”. Let me give you an illustration:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/control-vs-data-plane.png" alt=""><figcaption></figcaption></figure>

The control plane is responsible for exchanging routing information using routing protocols, building a routing table and ARP table.The data plane is responsible for the actual forwarding of IP packets. The routing table isn’t very suitable for fast forwarding because we have to deal with **recursive routing**. What is recursive routing? Let me give you an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/three-cisco-routers-in-a-row.png" alt=""><figcaption></figcaption></figure>

In the example above I have three routers. R3 has a loopback interface that we want to reach from R1. I will use static routes for reachability:

<pre><code><strong>R1(config)#ip route 3.3.3.0 255.255.255.0 192.168.23.3
</strong><strong>R1(config)#ip route 192.168.23.0 255.255.255.0 192.168.12.2
</strong></code></pre>

The first static route is to reach the loopback0 interface of R3 and points to the FastEthernet0/0 interface of R3. The second static route is required to reach network 192.168.23.0/24.

<pre><code><strong>R1#show ip route         
</strong>Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 -IS-IS level-2
       ia - IS-IS inter area, * - candidate default, per-user static route
       o - ODR, P - periodic downloaded static route

Gateway of last resort is not set

C    192.168.12.0/24 is directly connected, FastEthernet0/0
     3.0.0.0/24 is subnetted, 1 subnets
S       3.3.3.0 [1/0] via 192.168.23.3
S    192.168.23.0/24 [1/0] via 192.168.12.2
</code></pre>

Whenever R1 wants to reach 3.3.3.0/24 we have to do 3 lookups:

* The first lookup is to check the entry for 3.3.3.0 /24. It’s there and the next hop IP address is 192.168.23.3
* The second lookup is for 192.168.23.3. There’s an entry and the next hop IP address is 192.168.12.2.
* The third and last lookup is for 192.168.12.2. There’s an entry and it is directly connected.

R1 has to check the routing table 3 times before it knows where to send its traffic. Doesn’t sound very efficient right? Doing multiple lookups to reach a certain network is called **recursive routing**.

Most of the time all incoming and outgoing IP packets will be processed and forwarded by the data plane but there are some exceptions, first let me show you this picture:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/data-plane-forwarding-packets.png" alt=""><figcaption></figcaption></figure>

Most of the IP packets can be forwarded by the data plane. However there are some “special” IP packets that can’t be forwarded by the data plane immediately and they are sent to the control plane, here are some examples:

* IP packets that are destined for one of the IP addresses of the multilayer switch.
* Routing protocol traffic like OSPF, EIGRP or BGP.
* IP packets that have some of the options set in the IP header.
* IP packets with an expired TTL.

The control plane can forward outgoing IP packets to the data plane or use its own forwarding mechanism to determine the outgoing interface and the next hop IP address. An example of this is local policy based routing. If you have never heard about policy based routing, don’t worry…it’s covered in CCNP ROUTE.

Our multilayer switch has many more steps to take than the layer 2 switches so theoretically it should be slower right?

One reason that multilayer switches are able to forward frames and packets at wirespeed is because of special hardware called ASICs in the dataplane.

Information like MAC addresses, the routing table or access-lists are stored into these ASICs. The tables are stored in **content-addressable memory (CAM)** and **ternary content addressable memory (TCAM)**.

* The CAM table is used to store layer 2 information like:
  * The source MAC address.
  * The interface where we learned the MAC address on.
  * To which VLAN the MAC address belongs.

Table lookups are fast! Whenever the switch receives an Ethernet frame it will use a hashing algorithm to create a “key” for the destination MAC address + VLAN and it will compare this hash to the already hashed information in the CAM table. This way it is able to quickly lookup information in the CAM table.

* The TCAM table is used to store “higher layer” information like:
  * Access-lists.
  * Quality of service information.
  * Routing table.
* The TCAM table can match on 3 different values:
  * 0 = must be 0.
  * 1 = must be 1.
  * X = 0 or 1 both acceptable.
* Longest match will return a hit.
* Useful for a lookup where we don’t need an exact match. (routing table or ACLs for example).

Because there are 3 values we call it _ternary_.

So why are there 2 types of tables?

When we look for a MAC address we always require an **exact match**. We require the exact MAC address if we want to forward an Ethernet frame. The MAC address table is stored in a CAM table.

Whenever we need to match an IP packet against the routing table or an access-list we **don’t always need an exact match**. For example an IP packet with destination address 192.168.20.44 will match:

* 192.168.20.44 /32
* 192.168.20.0 /24
* 192.168.0.0 /16

Information like the routing table are stored in a TCAM table for this reason. We can decide whether all or some bits have to match.

Here’s an example of a TCAM table:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/tcam-table.png" alt=""><figcaption></figcaption></figure>

If we want to match IP address 192.168.10.22 the multilayer switch will first see if there’s a “most specific match”. There is nothing that matches 192.168.10.22 /32 so we’ll continue if there is anything else that matches. In this case there is an entry that matches 192.168.10.0 /24. The example above applies to routing table lookups, access-lists but also quality of service, VLAN access-lists and more.

Now you know all the steps a multilayer switch has to take when it has to forward ip packets, the control/data plane and that we use different tables stored in special hardware called ASICs. Let’s take a closer look at the actual ‘forwarding’ of IP packets.

There are different **switching methods** to forward IP packets. Here are the different switching options:

* **Process switching**:
  * All packets are examined by the CPU and all forwarding decisions are made in software…very slow!
* **Fast switching** (also known as **route caching**):
  * The first packet in a flow is examined by the CPU; the forwarding decision is cached in hardware for the next packets in the same flow. This is a faster method.
* **(CEF) Cisco Express Forwarding** (also known as **topology based switching):**
  * Forwarding table created in hardware beforehand. All packets will be switched using hardware. This is the fastest method but there are some limitations. Multilayer switches and routers use CEF.

When using **process switching** the router will remove the header for each Ethernet frame, look for the destination IP address in the routing table for each IP packet and then forward the Ethernet frame with the rewritten MAC addresses and CRC to the outgoing interface. Everything is done in software so this is very CPU-intensive.

**Fast switching** is more efficient because it will lookup the first IP packet but it will store the forwarding decision in the fast switching cache. When the routers receive Ethernet frames carrying IP packets in the same flow it can use the information in the cache to forward them to the correct outgoing interface.

The default for routers is **CEF (Cisco Express Forwarding)**. Let’s take a closer look at CEF:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/cef-forwarding-information-table.png" alt=""><figcaption></figcaption></figure>

The multilayer switch will use the information from tables that are built by the (control plane) **to build hardware tables**. It will use the routing table to build **the FIB (Forwarding Information Base)** and the ARP table to build the **adjacency table**. This is the fastest switching method because we now have all the layer 2 and 3 information required to forward IP packets in hardware.

{% hint style="info" %}
I should mention that some “lower end” routers don’t have dedicated hardware for forwarding, they store these tables in software.
{% endhint %}

Are you following me so far? Let’s take a look at the forwarding information table and the adjacency table on some routers. If you want to follow me along you can take a look at your multilayer switch OR use routers in GNS3:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/three-cisco-routers-in-a-row.png" alt=""><figcaption></figcaption></figure>

I’ll use the same topology that I showed you earlier. 3 routers and R3 has a loopback0 interface.

I’ll use static routes to have full connectivity:

<pre><code><strong>R1(config)#ip route 3.3.3.0 255.255.255.0 192.168.23.3
</strong><strong>R1(config)#ip route 192.168.23.0 255.255.255.0 192.168.12.2
</strong></code></pre>

<pre><code><strong>R2(config)#ip route 3.3.3.0 255.255.255.0 192.168.23.3
</strong></code></pre>

<pre><code><strong>R3(config)#ip route 192.168.12.0 255.255.255.0 192.168.23.2
</strong></code></pre>

These are the static routes that I’ll use.\
Now let me show you the routing and FIB table:

<pre><code><strong>R1#show ip route 
</strong>Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route

Gateway of last resort is not set

C    192.168.12.0/24 is directly connected, FastEthernet0/0
     3.0.0.0/24 is subnetted, 1 subnets
S       3.3.3.0 [1/0] via 192.168.23.3
S    192.168.23.0/24 [1/0] via 192.168.12.2
</code></pre>

<pre><code><strong>R1#show ip cef 
</strong>Prefix              Next Hop             Interface
0.0.0.0/0           drop                 Null0 (default route handler entry)
0.0.0.0/32          receive
3.3.3.0/24          192.168.12.2         FastEthernet0/0
192.168.12.0/24     attached             FastEthernet0/0
192.168.12.0/32     receive
192.168.12.1/32     receive
192.168.12.2/32     192.168.12.2         FastEthernet0/0
192.168.12.255/32   receive
192.168.23.0/24     192.168.12.2         FastEthernet0/0
224.0.0.0/4         drop
224.0.0.0/24        receive
255.255.255.255/32  receive
</code></pre>

**Show ip cef** reveals the FIB table to us.

You can see there’s quite some stuff in the FIB table, let me explain some of the entries:

* 0.0.0.0/0 is for the null0 interface. When we receive IP packets that match this rule then it will be dropped.
* 0.0.0.0/32 is for all-zero broadcasts. Forget about this one since we don’t use it anymore.
* 3.3.3.0/24 is the entry for the loopback0 interface of R3. Note that the next hop is 192.168.12.2 and NOT 192.168.23.3 as in the routing table!
* 192.168.12.0/24 is our directly connected network.
* 192.168.12.0/32 is reserved for the exact network address.
* 192.168.12.1/32 is the IP address on interface FastEthernet 0/0.
* 192.168.12.2/32 is the IP address on R2’s FastEthernet 0/0 interface.
* 192.168.12.255/32 is the broadcast address for network 192.168.12.0/24.
* 224.0.0.0/4 matches all multicast traffic. It will be dropped if multicast support is disabled globally.
* 224.0.0.0/24 matches all multicast traffic that is reserved for local network control traffic (for example OSPF, EIGRP).
* 255.255.255.255/32 is the broadcast address for a subnet.

Let’s take a detailed look at the entry for network 3.3.3.0 /24:

<pre><code><strong>R1#show ip cef 3.3.3.0 
</strong>3.3.3.0/24, version 8, epoch 0, cached adjacency 192.168.12.2
0 packets, 0 bytes
  via 192.168.23.3, 0 dependencies, recursive
    next hop 192.168.12.2, FastEthernet0/0 via 192.168.23.0/24
    valid cached adjacency
</code></pre>

The version number tells us how often this CEF entry was updated since the table was generated. We can see that in order to reach 3.3.3.0/24 we need to go to 192.168.23.3 and that a recursive lookup is required. The next hop is 192.168.12.2. It also says that it’s a **valid cached adjacency**. There are a number of different adjacencies:

* **Null adjacency**: used to send packets to the null0 interface.
* **Drop adjacency**: you’ll see this for packets that can’t be forwarded because of encapsulation errors, routes that cannot be resolved or protocols that are not supported.
* **Discard adjacency**: this is for packets that have to be discarded because of an access-list or other policy.
* **Punt adjacency**: used for packets that can’t be forwarded by CEF. They will be “punted” to the next switching method (fast switching and process switching).
* **Glean adjacency**:  used for directly connected routes. It’s used to tell the router that it should check the ARP table since it can reach the device directly.

Packets that are not forwarded by CEF are handled by the CPU. If you have many of those packets then you might see performance issues.

You can see how many packets have been handled by the CPU:

<pre><code><strong>R1#show cef not-cef-switched 
</strong>CEF Packets passed on to next switching layer
Slot  No_adj No_encap Unsupp'ted Redirect  Receive  Options   Access     Frag
RP         0       0           0        0       17        0        0        0
</code></pre>

You can use the **show cef not-cef-switched** command to verify this; the number of packets are listed per reason:

* **No\_adj**: adjacency is incomplete.
* **No\_encap**: ARP information is incomplete.
* **Unsupp’ted**: packet has features that are not supported.
* **Redirect**: ICMP redirect.
* **Receive**: These are the packets that were destined for an IP address configured on a layer 3 interface, packets that are meant for our router.
* **Options:** There are IP options in the header of the packet.
* **Access:** access-list evaluation failure.
* **Frag:** packet fragmention error.\


We can also take a look at the adjacency table that stores the layer 2 information for each entry:

<pre><code><strong>R1#show adjacency summary 
</strong>Adjacency Table has 1 adjacency
  Table epoch: 0 (1 entry at this epoch)

  Interface                 Adjacency Count
  FastEthernet0/0           1
</code></pre>

You can use the show adjacency summary command to take a quick look how many adjacencies we have. An adjacency is a mapping from layer 2 to 3 and comes from the ARP table.

<pre><code><strong>R1#show adjacency 
</strong>Protocol Interface                 Address
IP       FastEthernet0/0           192.168.12.2(9)
</code></pre>

R1 only has a single interface that is connected to R2. You can see the entry for 192.168.12.2 which is the FastEthernet 0/0 interface of R2. Let’s zoom in on this entry:

<pre><code><strong>R1#show adjacency detail 
</strong>Protocol Interface                 Address
IP       FastEthernet0/0           192.168.12.2(9)
                                   0 packets, 0 bytes
                                   CC011D800000CC001D8000000800
                                   ARP        03:55:00  
                                   Epoch: 0
</code></pre>

We can see there’s an entry for 192.168.12.2 and it says:

CC011D800000CC001D8000000800

What does this number mean? It’s the MAC addresses that we require and the Ethertype…let me break it down for you:

* CC011D800000 is the MAC address of R2’s FastEthernet0/0 interface.

<pre><code><strong>R2#show interfaces fastEthernet 0/0
</strong>FastEthernet0/0 is up, line protocol is up 
  Hardware is AmdFE, address is cc01.1d80.0000 (bia cc01.1d80.0000)
</code></pre>

• CC001D800000 is the MAC address of R1’s FastEthernet0/0 interface.

<pre><code><strong>R1#show interfaces fastEthernet 0/0
</strong>FastEthernet0/0 is up, line protocol is up 
  Hardware is AmdFE, address is cc00.1d80.0000 (bia cc00.1d80.0000)
</code></pre>

* 0800 is the Ethertype. 0x800 stands for IPv4.

Thanks to the FIB and adjacency table we have all the layer 2 and 3 information that we require to rewrite and forward packets. Keep in mind before actually forwarding the packet we first have to rewrite the header information:

* Source MAC address.
* Destination MAC address.
* Ethernet frame checksum.
* IP Packet TTL.
* IP Packet Checksum.

Once this is done we can forward the packet. Now you have an idea what CEF is about and how packets are dealt with.

Every now and then students ask me what the difference is between routers and switches since a multilayer switch can route, and a router can do switching if you want.

The difference is getting smaller but switches normally only use Ethernet. If you buy a Cisco Catalyst 3560 or 3750 you’ll only have Ethernet interfaces. They have ASICs so switching of frames can be done at wire speed. Routers on the other hand have other interfaces like serial links, wireless and they can be upgraded with modules for VPN, VoIP etc. You can’t configure stuff like NAT/PAT on a (small) switch. The line is getting thinner however…

I hope this lesson has helped you to understand CEF!
