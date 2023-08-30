# Multicast PIM Dense Mode

In the previous lesson we discussed the [basics of multicast routing](https://networklessons.com/cisco/ccnp-encor-350-401/multicast-routing) and I explained some of the differences between dense and sparse mode multicast routing protocols. The only multicast routing protocol that is fully supported on Cisco IOS devices is **PIM (Protocol Independent Multicast)**.

The _independent_ part is a bit confusing…PIM **depends** 100% on the information in the **unicast routing table** but it doesn’t matter which unicast routing protocol you use to fill it. Other multicast routing protocols like DVMRP and MOSPF don’t use the unicast routing table but **build their own tables**.

PIM supports three different modes:

* PIM Dense mode
* PIM Sparse mode
* PIM Sparse-Dense mode

Here’s the key difference between sparse and dense mode:

* Dense mode: **we forward** multicast traffic on all interfaces until a downstream router **requests us to stop fowarding**.
* Sparse mode: **we don’t forward** multicast traffic on any interface until a downstream router **requests us to forward it**.

In this lesson, we’ll take a close look at PIM dense mode.

PIM dense mode is a **push method** where we use **source based trees**. What does this mean? Let’s look at an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/01/multicast-pim-dense-mode-flooding.png" alt=""><figcaption></figcaption></figure>

Above we see a video server sending a multicast packet towards R1. As soon as R1 receives this multicast packet it will create an entry in its multicast routing table where it stores the source address and multicast group address. It will then flood the traffic on all of its interfaces.

Other routers that receive this multicast packet will do the same thing, the multicast packet is flooded on all of their interfaces. This does cause some issues, one problem is that we will have multicast routing loops. For example you can see that the packet that R1 receives is forwarded to R2 > R4 > R5 and back to R1 (and the other way around). How do we deal with this? Take a look below:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/01/multicast-pim-dense-mode-pruning.png" alt=""><figcaption></figcaption></figure>

Each router that is not interested in the multicast traffic will send a **prune** to its upstream router, requesting it to stop forwarding it. The upstream router is the router where we **receive** multicast traffic from, a downstream router is a router where we **forward** multicast traffic to. The red arrows above are the prune messages that the routers will send. There are a couple of reasons why a router can send a prune message:

* When we don’t have any directly connected hosts that are interested in receiving the multicast traffic.
* When no downstream routers are interested in receiving the multicast traffic.
* When we receive traffic on a non-RPF interface.

**RPF (Reverse Path Forwarding)** is an additional check that helps to prevent multicast routing loops:

When a router receives a multicast packet it will check the **source IP address** of this packet in the unicast routing table. It will check the **next hop** and **outgoing interface** that we use to reach the source.

* When the multicast packet was received on the interface that we use to reach the source, the RPF check **succeeds**.
* When the multicast packet was received on another interface, it **fails** the RPF check and the packet is **discarded**.

The end result will look like this:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/01/multicast-pim-dense-mode-source-tree.png" alt=""><figcaption></figcaption></figure>

We now have a nice clean topology where multicast traffic is flooded from R1 to R2 > R6 > H1. This flood and prune behavior will occur **every three minutes**.

We call this topology the **source-based distribution tree** or **SPT (Shortest Path Tree)**. Sometimes it’s also called the **source tree:**

* The tree defines the path from the source to the receiver(s).
* The source is the root of our tree.
* The routers in between that are forwarding traffic are the nodes.
* The subnets with receivers are the branches and leaves of the tree.

Depending on the source and/or multicast groups that we use, you might have more than one source tree in your network. Once we look at the configuration, you’ll see that we use the \[S,G] notation to refer to a particular source tree:

* **S: the source address**
* **G: the multicast group address.**

Now you have an idea what PIM dense mode is about, let’s see what it looks like on some real routers shall we?

## Configuration

We will use the following topology for this demonstration:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/01/multicast-pim-dense-topology.png" alt=""><figcaption></figcaption></figure>

Above we have a video server on the left side which will be the source of our multicast traffic. H2 and H3 are two host devices who want to receive the multicast traffic. R1, R2 and R3 will be configured to use PIM dense mode.

### Unicast Routing

First we will configure a routing protocol on all routers, I’ll pick OSPF. All interfaces will be advertised:

<pre><code><strong>R1(config)#router ospf 1
</strong><strong>R1(config-router)#network 192.168.1.0 0.0.0.255 area 0
</strong><strong>R1(config-router)#network 192.168.12.0 0.0.0.255 area 0
</strong><strong>R1(config-router)#network 192.168.13.0 0.0.0.255 area 0
</strong></code></pre>

<pre><code><strong>R2(config)#router ospf 1
</strong><strong>R2(config-router)#network 192.168.2.0 0.0.0.255 area 0
</strong><strong>R2(config-router)#network 192.168.12.0 0.0.0.255 area 0
</strong><strong>R2(config-router)#network 192.168.23.0 0.0.0.255 area 0
</strong></code></pre>

<pre><code><strong>R3(config)#router ospf 1
</strong><strong>R3(config-router)#network 192.168.3.0 0.0.0.255 area 0
</strong><strong>R3(config-router)#network 192.168.13.0 0.0.0.255 area 0
</strong><strong>R3(config-router)#network 192.168.23.0 0.0.0.255 area 0
</strong></code></pre>

Now we can focus on the multicast configuration.

### PIM Dense Mode

Multicast routing is **disabled by default** on Cisco IOS routers so we have to enable it:

<pre><code>R1, R2 &#x26; R3
<strong>(config)#ip multicast-routing
</strong></code></pre>

Now we can configure PIM. Let’s start with R1:

<pre><code><strong>R1(config)#interface GigabitEthernet 0/1
</strong><strong>R1(config-if)#ip pim dense-mode
</strong></code></pre>

As soon as I enable PIM dense mode on the interface our router will start sending PIM hello packets. Here’s what these look like in wireshark:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/01/multicast-pim-dense-hello-packet.png" alt=""><figcaption></figcaption></figure>

Above you can see the type of the packet (hello) and it also shows the hold time. For whatever reason, wireshark (version 2.0.1) is showing me a value of 1 but the default is 105 seconds. If you want to take a look at this packet yourself, here’s the cloudshark capture:

[PIM Dense Mode Hello Packet](https://www.cloudshark.org/captures/0e6504bdc586)

It shows the correct holdtime of 105 seconds.

Let’s enable PIM dense on R2 as well:

<pre><code><strong>R2(config)#interface GigabitEthernet 0/1
</strong><strong>R2(config-if)#ip pim dense-mode 
</strong></code></pre>

As soon as we do this, R1 and R2 will become neighbors since they receive each others PIM hello packets.If they keep receiving them before the holddown timer is expired then they will keep seeing each other as PIM neighbors.:

```
R1#
%PIM-5-NBRCHG: neighbor 192.168.12.2 UP on interface GigabitEthernet0/1
```

Let’s enable all the other interfaces. You also need to enable PIM on interfaces that connect to host devices:

<pre><code><strong>R1(config)#interface GigabitEthernet 0/2
</strong><strong>R1(config-if)#ip pim dense-mode 
</strong>
<strong>R1(config)#interface GigabitEthernet 0/3
</strong><strong>R1(config-if)#ip pim dense-mode
</strong></code></pre>

<pre><code><strong>R2(config)#interface GigabitEthernet 0/2
</strong><strong>R2(config-if)#ip pim dense-mode
</strong>
<strong>R2(config)#interface GigabitEthernet 0/3
</strong><strong>R2(config-if)#ip pim dense-mode
</strong></code></pre>

<pre><code><strong>R3(config)#interface GigabitEthernet 0/1
</strong><strong>R3(config-if)#ip pim dense-mode 
</strong>
<strong>R3(config)#interface GigabitEthernet 0/2
</strong><strong>R3(config-if)#ip pim dense-mode
</strong>
<strong>R3(config)#interface GigabitEthernet 0/3
</strong><strong>R3(config-if)#ip pim dense-mode
</strong></code></pre>

### PIM Dense Neighbors

Let’s make sure all routers have become neighbors:

<pre><code><strong>R1#show ip pim neighbor 
</strong>PIM Neighbor Table
Mode: B - Bidir Capable, DR - Designated Router, N - Default DR Priority,
      P - Proxy Capable, S - State Refresh Capable, G - GenID Capable,
      L - DR Load-balancing Capable
Neighbor          Interface                Uptime/Expires    Ver   DR
Address                                                            Prio/Mode
192.168.12.2      GigabitEthernet0/1       00:05:59/00:01:37 v2    1 / DR S P G
192.168.13.3      GigabitEthernet0/2       00:00:48/00:01:26 v2    1 / DR S P G
</code></pre>

<pre><code><strong>R2#show ip pim neighbor 
</strong>PIM Neighbor Table
Mode: B - Bidir Capable, DR - Designated Router, N - Default DR Priority,
      P - Proxy Capable, S - State Refresh Capable, G - GenID Capable,
      L - DR Load-balancing Capable
Neighbor          Interface                Uptime/Expires    Ver   DR
Address                                                            Prio/Mode
192.168.12.1      GigabitEthernet0/1       00:06:18/00:01:22 v2    1 / S P G
192.168.23.3      GigabitEthernet0/2       00:01:00/00:01:43 v2    1 / DR S P G
</code></pre>

<pre><code><strong>R3#show ip pim neighbor 
</strong>PIM Neighbor Table
Mode: B - Bidir Capable, DR - Designated Router, N - Default DR Priority,
      P - Proxy Capable, S - State Refresh Capable, G - GenID Capable,
      L - DR Load-balancing Capable
Neighbor          Interface                Uptime/Expires    Ver   DR
Address                                                            Prio/Mode
192.168.13.1      GigabitEthernet0/1       00:01:14/00:01:29 v2    1 / S P G
192.168.23.2      GigabitEthernet0/2       00:01:09/00:01:33 v2    1 / S P G
</code></pre>

Above we can see that all routers have become PIM neighbors. The default version on Cisco IOS is version 2. The expiry times is the holddown timer that is counting down. Each time when the router receives a PIM hello packet, it will reset itself to 00:01:45 (105 seconds).

### Multicast Routing

Let’s take a look at the multicast routing tables:

<pre><code><strong>R1#show ip mroute 
</strong>IP Multicast Routing Table
Flags: D - Dense, S - Sparse, B - Bidir Group, s - SSM Group, C - Connected,
       L - Local, P - Pruned, R - RP-bit set, F - Register flag,
       T - SPT-bit set, J - Join SPT, M - MSDP created entry, E - Extranet,
       X - Proxy Join Timer Running, A - Candidate for MSDP Advertisement,
       U - URD, I - Received Source Specific Host Report, 
       Z - Multicast Tunnel, z - MDT-data group sender, 
       Y - Joined MDT-data group, y - Sending to MDT-data group, 
       G - Received BGP C-Mroute, g - Sent BGP C-Mroute, 
       N - Received BGP Shared-Tree Prune, n - BGP C-Mroute suppressed, 
       Q - Received BGP S-A Route, q - Sent BGP S-A Route, 
       V - RD &#x26; Vector, v - Vector, p - PIM Joins on route, 
       x - VxLAN group
Outgoing interface flags: H - Hardware switched, A - Assert winner, p - PIM Join
 Timers: Uptime/Expires
 Interface state: Interface, Next-Hop or VCD, State/Mode

(*, 224.0.1.40), 00:12:12/00:02:48, RP 0.0.0.0, flags: DCL
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/2, Forward/Dense, 00:03:06/stopped
    GigabitEthernet0/1, Forward/Dense, 00:12:12/stopped
</code></pre>

<pre><code><strong>R2#show ip mroute 
</strong>IP Multicast Routing Table
Flags: D - Dense, S - Sparse, B - Bidir Group, s - SSM Group, C - Connected,
       L - Local, P - Pruned, R - RP-bit set, F - Register flag,
       T - SPT-bit set, J - Join SPT, M - MSDP created entry, E - Extranet,
       X - Proxy Join Timer Running, A - Candidate for MSDP Advertisement,
       U - URD, I - Received Source Specific Host Report, 
       Z - Multicast Tunnel, z - MDT-data group sender, 
       Y - Joined MDT-data group, y - Sending to MDT-data group, 
       G - Received BGP C-Mroute, g - Sent BGP C-Mroute, 
       N - Received BGP Shared-Tree Prune, n - BGP C-Mroute suppressed, 
       Q - Received BGP S-A Route, q - Sent BGP S-A Route, 
       V - RD &#x26; Vector, v - Vector, p - PIM Joins on route, 
       x - VxLAN group
Outgoing interface flags: H - Hardware switched, A - Assert winner, p - PIM Join
 Timers: Uptime/Expires
 Interface state: Interface, Next-Hop or VCD, State/Mode

(*, 224.0.1.40), 00:08:33/00:02:35, RP 0.0.0.0, flags: DCL
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/2, Forward/Dense, 00:03:15/stopped
    GigabitEthernet0/1, Forward/Dense, 00:08:33/stopped
</code></pre>

<pre><code><strong>R3#show ip mroute 
</strong>IP Multicast Routing Table
Flags: D - Dense, S - Sparse, B - Bidir Group, s - SSM Group, C - Connected,
       L - Local, P - Pruned, R - RP-bit set, F - Register flag,
       T - SPT-bit set, J - Join SPT, M - MSDP created entry, E - Extranet,
       X - Proxy Join Timer Running, A - Candidate for MSDP Advertisement,
       U - URD, I - Received Source Specific Host Report, 
       Z - Multicast Tunnel, z - MDT-data group sender, 
       Y - Joined MDT-data group, y - Sending to MDT-data group, 
       G - Received BGP C-Mroute, g - Sent BGP C-Mroute, 
       N - Received BGP Shared-Tree Prune, n - BGP C-Mroute suppressed, 
       Q - Received BGP S-A Route, q - Sent BGP S-A Route, 
       V - RD &#x26; Vector, v - Vector, p - PIM Joins on route, 
       x - VxLAN group
Outgoing interface flags: H - Hardware switched, A - Assert winner, p - PIM Join
 Timers: Uptime/Expires
 Interface state: Interface, Next-Hop or VCD, State/Mode

(*, 224.0.1.40), 00:03:24/00:02:10, RP 0.0.0.0, flags: DCL
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/2, Forward/Dense, 00:03:20/stopped
    GigabitEthernet0/1, Forward/Dense, 00:03:24/stopped
</code></pre>

At this moment we don’t have any multicast traffic going on but we do see a multicast group in the tables, 224.0.1.40. This can be confusing if you are new to multicast, let me explain:

PIM sparse mode (which we will discuss in other lessons) uses a RP (Rendezvous Point) and Cisco routers support a protocol called “Auto RP Discovery” to automatically find the RP in the network. Auto RP uses this multicast group address, 224.0.1.40.

PIM dense mode doesn’t use RPs so there is no reason at all for our routers to listen to the 224.0.1.40 group address. For whatever reason, as soon as you enable PIM then autoRP is also enabled and the router will listen to this group address. You can ignore this entry completely when you are working with PIM dense mode.

If you take a close look then you can see that none of the routers are receiving or forwarding traffic for this group:

* The incoming interface is null.
* The outgoing interface list shows stopped.

You can see that the multicast routing table has a LOT of different flags. Don’t worry about these too much for now, we’ll discuss some of them in other lessons.

Let’s generate some multicast traffic ourselves. First i’ll have to configure one of the hosts to join a multicast group. Let’s configure H2 to join 239.1.1.1:

<pre><code><strong>H2(config)#interface GigabitEthernet 0/1
</strong><strong>H2(config-if)#ip igmp join-group 239.1.1.1
</strong></code></pre>

Let’s check how this influences the multicast routing table of R2:

<pre><code><strong>R2#show ip mroute 239.1.1.1
</strong>
(*, 239.1.1.1), 00:00:19/00:02:51, RP 0.0.0.0, flags: DC
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/3, Forward/Dense, 00:00:19/stopped
    GigabitEthernet0/2, Forward/Dense, 00:00:19/stopped
    GigabitEthernet0/1, Forward/Dense, 00:00:19/stopped
</code></pre>

Above we can see that R2 has created an \*, 239.1.1.1 entry for multicast group 239.1.1.1. At this moment however we are not receiving any traffic for this group so we don’t know what the source is and we can’t forward anything.

Let’s change that. The most simple method to generate multicast traffic is to send a ping from the multicast group address:

<pre><code><strong>S1#ping 239.1.1.1 repeat 2147483647
</strong>Type escape sequence to abort.
Sending 2147483647, 100-byte ICMP Echos to 239.1.1.1, timeout is 2 seconds:

Reply to request 0 from 192.168.2.2, 132 ms
Reply to request 1 from 192.168.2.2, 7 ms
Reply to request 2 from 192.168.2.2, 6 ms
</code></pre>

Above you can see that H2 is replying. Let’s check the multicast routing tables:

<pre><code><strong>R1#show ip mroute 239.1.1.1
</strong>
IP Multicast Routing Table
Flags: D - Dense, S - Sparse, B - Bidir Group, s - SSM Group, C - Connected,
       L - Local, P - Pruned, R - RP-bit set, F - Register flag,
       T - SPT-bit set, J - Join SPT, M - MSDP created entry, E - Extranet,
       X - Proxy Join Timer Running, A - Candidate for MSDP Advertisement,
       U - URD, I - Received Source Specific Host Report, 
       Z - Multicast Tunnel, z - MDT-data group sender, 
       Y - Joined MDT-data group, y - Sending to MDT-data group, 
       G - Received BGP C-Mroute, g - Sent BGP C-Mroute, 
       N - Received BGP Shared-Tree Prune, n - BGP C-Mroute suppressed, 
       Q - Received BGP S-A Route, q - Sent BGP S-A Route, 
       V - RD &#x26; Vector, v - Vector, p - PIM Joins on route, 
       x - VxLAN group
Outgoing interface flags: H - Hardware switched, A - Assert winner, p - PIM Join
 Timers: Uptime/Expires
 Interface state: Interface, Next-Hop or VCD, State/Mode

(*, 239.1.1.1), 00:01:16/stopped, RP 0.0.0.0, flags: D
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/2, Forward/Dense, 00:01:16/stopped
    GigabitEthernet0/1, Forward/Dense, 00:01:16/stopped

(192.168.1.1, 239.1.1.1), 00:01:16/00:01:43, flags: T
  Incoming interface: GigabitEthernet0/3, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/1, Forward/Dense, 00:01:16/stopped
    GigabitEthernet0/2, Prune/Dense, 00:01:15/00:01:43
</code></pre>

Let’s discuss what we see above:

* R1 is receiving multicast traffic for 239.1.1.1 from 192.168.1.1 on its Gi0/3 interface and has added this entry in its multicast routing table. This is the \[S,G] entry that I explained before.
* R1 has been forwarding this traffic for one minute and 16 seconds on the Gi0/1 interface.
* When we don’t forward any packets for one minute and 43 seconds then this entry will be removed from the multicast routing table. As long as we keep receiving packets, this timer will be resetted to three minutes.
* The T flags means that we are using the shortest path source tree.
* The RPF neighbor for R1 is 0.0.0.0, this value is empty since it’s a directly connected interface.
* R1 is forwarding traffic on its Gi0/1 interface that is connected to R2. It has been doing this for one minute and 16 seconds and there is no expiry. It will keep doing this until it gets pruned.
* R1 has stopped forwarding traffic on its Gi0/2 interface since it is pruned.

Since R3 is not interested in receiving this multicast traffic and it has sent a prune message to R1, asking it to stop forwarding this traffic. Here’s what this prune message looks like:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/01/multicast-pim-dense-prune-packet.png" alt=""><figcaption></figcaption></figure>

Above you can see that this packet was sent by R3 (192.168.13.3) and destined to 224.0.0.13 (PIM version 2).

PIM uses a single packet for join and prune messages, in this packet we can see that R3 requests a prune for 239.1.1.1 from 192.168.1.1. This packet is meant for 192.168.13.1.

In PIM version 1 an interface would be pruned for three minutes. After these three minutes, the traffic would be forwarded again until we send another prune message. This “flood and prune” behavior is pretty inefficient so in PIM version 2 use something called “state refresh”. Prune messages are sent periodically. Each time we receive a prune message, the prune timer is resetted to three minutes which prevents the “flood and prune” behavior.

Depending on your IOS version, it is possible that state refresh is disabled by default. You can enable it with the `ip pim state-refresh origination-interval`\
command.

An interface is pruned for **three minutes** and each time we send a prune message, this timer will be resetted. If we stop sending prune messages then the interface will go back into forwarding after three minutes.

Want to see this packet for yourself? Take a look here:

[PIM Dense Prune Message](https://www.cloudshark.org/captures/8804ca52f5a7)

Let’s check R2:

<pre><code><strong>R2#show ip mroute 239.1.1.1
</strong>
(*, 239.1.1.1), 00:03:47/stopped, RP 0.0.0.0, flags: DC
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/3, Forward/Dense, 00:03:47/stopped
    GigabitEthernet0/2, Forward/Dense, 00:03:47/stopped
    GigabitEthernet0/1, Forward/Dense, 00:03:47/stopped
          
(192.168.1.1, 239.1.1.1), 00:01:22/00:01:37, flags: T
  Incoming interface: GigabitEthernet0/1, RPF nbr 192.168.12.1
  Outgoing interface list:
    GigabitEthernet0/2, Prune/Dense, 00:01:22/00:01:37
    GigabitEthernet0/3, Forward/Dense, 00:01:22/stopped
</code></pre>

Let’s discuss what we have above:

* R2 has set the DC flags for this group. The D means we are using dense mode and the C means that we have a directly connected host that wants to receive this traffic (H2).
* R2 has been receiving traffic from 192.168.1.1 to 239.1.1.1 on its Gi0/1 interface. The RPF neighbor is 192.168.12.1 (R1).
* R2 has stopped forwarding traffic on its Gi0/2 interface towards R3.
* R2 is forwarding traffic on its Gi0/3 interface towards H2.

What about R3?

<pre><code><strong>R3#show ip mroute 239.1.1.1
</strong>
(*, 239.1.1.1), 00:01:27/stopped, RP 0.0.0.0, flags: D
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/2, Forward/Dense, 00:01:27/stopped
    GigabitEthernet0/1, Forward/Dense, 00:01:27/stopped

(192.168.1.1, 239.1.1.1), 00:01:27/00:01:32, flags: PT
  Incoming interface: GigabitEthernet0/1, RPF nbr 192.168.13.1
  Outgoing interface list:
    GigabitEthernet0/2, Prune/Dense, 00:01:27/00:01:32, A
</code></pre>

Let’s see:

* R3 is receiving traffic for 239.1.1.1 from 192.168.1.1 on its Gi0/1 interface that  connects to R1. The RPF neighbor is 192.168.13.1.
* The P flag means that this traffic has been pruned.
* R3 has pruned its Gi0/2 interface. The A flag stands for **assert winner**.

So the big question…what is an assert winner?

R2 and R3 are both connected to the 192.168.23.0/24 subnet and both of them are receiving traffic from R1.

Let’s imagine that we have a host on the 192.168.23.0/24 subnet that wants to receive multicast traffic. Which router will forward this traffic? R2 or R3? If both routers forward multicast traffic then the host would receive duplicate multicast packets.

To prevent this from happening, we hold an election to decide which router becomes the forwarder of multicast traffic, that’s the assert winner.

In this case it seems R3 has won the election for the 192.168.23.0/24 subnet, it will be responsible for forwarding traffic on this subnet. However since we don’t have any hosts in the 192.168.23.0/24 subnet, R2 has asked R3 to stop forwarding multicast traffic…as a result, it has been pruned.

If you want to learn the details of the assert election then you can check out the [PIM assert mechanism](https://networklessons.com/cisco/ccnp-encor-350-401/multicast-pim-assert-explained) lesson later.

### PIM Graft Message

So far, so good. Multicast packets are flowing from S1 to H2 and unnecessary interfaces have been pruned.

What if something changes in our network? For example let’s say that we want H3 to join the 239.1.1.1 multicast group. At the moment R3 has pruned its interfaces but as soon as H3 joins, R3 should receive multicast packets.

R3 could stop sending prune messages to R1…when R1 doesn’t receive any prune messages for three minutes then it will put the interface in forwarding again. This doesn’t sound very efficient though…

The “graft message” can be used to **unprune a pruned interface**. When we send the graft message to our upstream neighbor then the interface will be placed in **forwarding immediately**.

Let’s see how this works…to see it in action, we’ll enable PIM debugging on all routers:

<pre><code>R1, R2 &#x26; R3
<strong>#debug ip pim 
</strong>PIM debugging is on
</code></pre>

Now we will configure H3 to join the 239.1.1.1 multicast group. This will trigger R3 to ask R1 to forward multicast traffic for this group:

<pre><code><strong>H3(config)#interface GigabitEthernet 0/1
</strong><strong>H3(config-if)#ip igmp join-group 239.1.1.1
</strong></code></pre>

Here’s what you will see on R3 and R1:

```
R3#
PIM(0): Building Graft message for 239.1.1.1, GigabitEthernet0/3: no entries
PIM(0): Building Graft message for 239.1.1.1, GigabitEthernet0/2: no entries
PIM(0): Building Graft message for 239.1.1.1, GigabitEthernet0/1:
     192.168.1.1/32 count 1
PIM(0): Send v2 Graft to 192.168.13.1 (GigabitEthernet0/1)
PIM(0): Received v2 Graft-Ack on GigabitEthernet0/1 from 192.168.13.1
     Group 239.1.1.1:
     192.168.1.1/32
```

```
R1#
PIM(0): Join-list: (192.168.1.1/32, 239.1.1.1)
PIM(0): Add GigabitEthernet0/2/0.0.0.0 to (192.168.1.1, 239.1.1.1), Forward state, by PIM Graft
PIM(0): Send v2 Graft-Ack on GigabitEthernet0/2 to 192.168.13.3
```

Above you can see that R3 sends the graft message, asking R1 to go into forwarding again. When R1 receives it, it will put its Gi0/2 interface into forwarding immediately. It will also inform R3 by replying with a graft acknowledgment.

Here’s what these packets look like in wireshark. First the graft message:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/01/multicast-pim-dense-graft-packet.png" alt=""><figcaption></figcaption></figure>

Above you can see that this is a unicast packet from R3 to R1. In the graft message, we find the group address and source that we want to receive. Below you can see the graft acknowledgment:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/01/multicast-pim-dense-graft-ack-packet.png" alt=""><figcaption></figcaption></figure>

The acknowledgment is basically the same message but the type has been changed to graft-ack.

Want to take a look at these packets yourself? Check out the cloudshark capture here:

[Multicast PIM Dense Graft Packets](https://www.cloudshark.org/captures/dcd04a3ee9d6)

Let’s take another look at our multicast routing tables:

<pre><code><strong>R1#show ip mroute 239.1.1.1
</strong>
(*, 239.1.1.1), 00:26:48/stopped, RP 0.0.0.0, flags: D
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/2, Forward/Dense, 00:26:48/stopped
    GigabitEthernet0/1, Forward/Dense, 00:26:48/stopped

(192.168.1.1, 239.1.1.1), 00:26:48/00:01:18, flags: T
  Incoming interface: GigabitEthernet0/3, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/1, Forward/Dense, 00:26:48/stopped
    GigabitEthernet0/2, Forward/Dense, 00:08:04/stopped
</code></pre>

<pre><code><strong>R2#show ip mroute 239.1.1.1
</strong>
(*, 239.1.1.1), 00:19:51/stopped, RP 0.0.0.0, flags: DC
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/3, Forward/Dense, 00:19:51/stopped
    GigabitEthernet0/2, Forward/Dense, 00:19:51/stopped
    GigabitEthernet0/1, Forward/Dense, 00:19:51/stopped
          
(192.168.1.1, 239.1.1.1), 00:19:51/00:02:35, flags: T
  Incoming interface: GigabitEthernet0/1, RPF nbr 192.168.12.1
  Outgoing interface list:
    GigabitEthernet0/2, Prune/Dense, 00:09:20/00:02:33
    GigabitEthernet0/3, Forward/Dense, 00:19:51/stopped
</code></pre>

<pre><code><strong>R3#show ip mroute 239.1.1.1
</strong>
(*, 239.1.1.1), 00:27:36/stopped, RP 0.0.0.0, flags: DC
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/3, Forward/Dense, 00:08:51/stopped
    GigabitEthernet0/2, Forward/Dense, 00:27:36/stopped
    GigabitEthernet0/1, Forward/Dense, 00:27:36/stopped
          
(192.168.1.1, 239.1.1.1), 00:12:37/00:01:49, flags: T
  Incoming interface: GigabitEthernet0/1, RPF nbr 192.168.13.1
  Outgoing interface list:
    GigabitEthernet0/3, Forward/Dense, 00:08:51/stopped
    GigabitEthernet0/2, Prune/Dense, 00:09:48/00:02:02, A
</code></pre>

Above we can see that R1 has started forwarding traffic towards R3 and R3 is forwarding it towards H3.

### Link Failures

When a link fails and / or changes are made to the unicast routing table then it’s possible that the RPF interface will also change. When the RPF interface changes, it’s also possible that changes are made to outgoing interfaces.

Let’s find out how PIM Dense mode responds to changes in our topology. To demonstrate this, we’ll shut one of the interfaces on R1. This means R3 will have to receive multicast traffic through R2:

<pre><code><strong>R1(config)#interface GigabitEthernet 0/2
</strong><strong>R1(config)#shutdown
</strong></code></pre>

Here’s what R1 will report:

```
R1#
PIM(0): Neighbor 192.168.13.3 (GigabitEthernet0/2) timed out
PIM(0): Prune GigabitEthernet0/2/224.0.1.40 from (*, 224.0.1.40)
PIM(0): Prune GigabitEthernet0/2/239.1.1.1 from (*, 239.1.1.1)
PIM(0): Prune GigabitEthernet0/2/239.1.1.1 from (192.168.1.1/32, 239.1.1.1)
```

Since the interface is down, R1 will lose its neighbor (R3) and prunes all interfaces. Here’s R2:

```
R2#
PIM(0): Received v2 Assert on GigabitEthernet0/2 from 192.168.23.3
PIM(0): Assert metric to source 192.168.1.1 is [Inf/3]
PIM(0): We win, our metric [110/2]
PIM(0): Update GigabitEthernet0/2/192.168.23.3 to (192.168.1.1, 239.1.1.1), Forward state, by PIM Assert
PIM(0): Changed GigabitEthernet0/2 from Prune to Forward state
PIM(0): (192.168.1.1/32, 239.1.1.1) oif GigabitEthernet0/2 in Forward state
PIM(0): Send v2 Assert on GigabitEthernet0/2 for 239.1.1.1, source 192.168.1.1, metric [110/2]
PIM(0): Assert metric to source 192.168.1.1 is [110/2]
PIM(0): We win, our metric [110/2]
PIM(0): Schedule to prune GigabitEthernet0/2
PIM(0): (192.168.1.1/32, 239.1.1.1) oif GigabitEthernet0/2 in Forward state
PIM(0): Join-list: (192.168.1.1/32, 239.1.1.1)
PIM(0): Add GigabitEthernet0/2/0.0.0.0 to (192.168.1.1, 239.1.1.1), Forward state, by PIM Graft
PIM(0): Send v2 Graft-Ack on GigabitEthernet0/2 to 192.168.23.3
PIM(0): Received v2 Join/Prune on GigabitEthernet0/2 from 192.168.23.3, to us
PIM(0): Join-list: (192.168.1.1/32, 239.1.1.1)
PIM(0): Update GigabitEthernet0/2/192.168.23.3 to (192.168.1.1, 239.1.1.1), Forward state, by PIM SG Join
```

Above you can see that R2 wants to prune its Gi0/2 interface but receives a graft message from R3, requesting it to put its interface in forwarding. Here’s R3:

```
R3#
PIM(0): Send v2 Assert cancel on GigabitEthernet0/2 for 239.1.1.1, source 192.168.1.1, metric [Inf/3], RP-bit
PIM(0): Received v2 Assert on GigabitEthernet0/2 from 192.168.23.2
PIM(0): Assert metric to source 192.168.1.1 is [110/2]
PIM(0): Cached metric is RP-bit [Inf/-1]
PIM(0): Set join delay timer to 800 msec for (192.168.1.1/32, 239.1.1.1) on GigabitEthernet0/2
PIM(0): Building Graft message for 239.1.1.1, GigabitEthernet0/3: no entries
PIM(0): Building Graft message for 239.1.1.1, GigabitEthernet0/2:
192.168.1.1/32 count 1
PIM(0): Send v2 Graft to 192.168.23.2 (GigabitEthernet0/2)
PIM(0): Building Graft message for 239.1.1.1, GigabitEthernet0/1: no entries
PIM(0): Received v2 Graft-Ack on GigabitEthernet0/2 from 192.168.23.2
Group 239.1.1.1:
192.168.1.1/32
PIM(0): Insert (192.168.1.1,239.1.1.1) join in nbr 192.168.23.2's queue
PIM(0): Building Join/Prune packet for nbr 192.168.23.2
PIM(0): Adding v2 (192.168.1.1/32, 239.1.1.1) Join
PIM(0): Send v2 join/prune to 192.168.23.2 (GigabitEthernet0/2)
```

Above you can see R3 sending the graft message to R2. Let’s check the multicast routing tables:

<pre><code><strong>R2#show ip mroute 239.1.1.1
</strong>
(*, 239.1.1.1), 00:31:01/stopped, RP 0.0.0.0, flags: DC
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/3, Forward/Dense, 00:31:01/stopped
    GigabitEthernet0/2, Forward/Dense, 00:31:01/stopped
    GigabitEthernet0/1, Forward/Dense, 00:31:01/stopped
          
(192.168.1.1, 239.1.1.1), 00:31:01/00:01:31, flags: T
  Incoming interface: GigabitEthernet0/1, RPF nbr 192.168.12.1
  Outgoing interface list:
    GigabitEthernet0/2, Forward/Dense, 00:01:52/stopped, A
    GigabitEthernet0/3, Forward/Dense, 00:31:01/stopped
</code></pre>

<pre><code><strong>R3#show ip mroute 239.1.1.1
</strong>
(*, 239.1.1.1), 00:38:08/stopped, RP 0.0.0.0, flags: DC
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/3, Forward/Dense, 00:19:23/stopped
    GigabitEthernet0/2, Forward/Dense, 00:38:08/stopped

(192.168.1.1, 239.1.1.1), 00:23:09/00:01:14, flags: T
  Incoming interface: GigabitEthernet0/2, RPF nbr 192.168.23.2
  Outgoing interface list:
    GigabitEthernet0/3, Forward/Dense, 00:19:23/stopped
</code></pre>

Above you can see that R2 is now forwarding traffic towards R3. On R3 we can also see that 192.168.23.2 is now the RPF interface to reach 192.168.1.1.

## Conclusion

You have now learned the basics of PIM Dense mode where we build a source based tree. Each router will create an entry in its multicast routing table where we store the source and multicast group address. When the RPF check succeeds, we forward traffic on all our interfaces except the one where we received the traffic on.

Router that are not interested in the traffic will send prune messages,asking their upstream neighbor to stop forwarding.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="H2" %}
```
hostname H2
!
no ip routing
!
interface GigabitEthernet0/1
 ip address 192.168.2.2 255.255.255.0
 no ip route-cache
 ip igmp join-group 239.1.1.1
 duplex auto
 speed auto
 media-type rj45
!
ip default-gateway 192.168.2.254
!
end
```
{% endtab %}

{% tab title="H3" %}
```
hostname H3
!
no ip routing
!
interface GigabitEthernet0/1
 ip address 192.168.3.3 255.255.255.0
 no ip route-cache
 ip igmp join-group 239.1.1.1
 duplex auto
 speed auto
 media-type rj45
!
ip default-gateway 192.168.3.254
!
end
```
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
ip multicast-routing 
ip cef
!
interface GigabitEthernet0/1
 ip address 192.168.12.1 255.255.255.0
 ip pim dense-mode
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/2
 ip address 192.168.13.1 255.255.255.0
 ip pim dense-mode
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/3
 ip address 192.168.1.254 255.255.255.0
 ip pim dense-mode
 duplex auto
 speed auto
 media-type rj45
!
router ospf 1
 network 192.168.1.0 0.0.0.255 area 0
 network 192.168.12.0 0.0.0.255 area 0
 network 192.168.13.0 0.0.0.255 area 0
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
ip multicast-routing 
ip cef
!
interface GigabitEthernet0/1
 ip address 192.168.12.2 255.255.255.0
 ip pim dense-mode
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/2
 ip address 192.168.23.2 255.255.255.0
 ip pim dense-mode
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/3
 ip address 192.168.2.254 255.255.255.0
 ip pim dense-mode
 duplex auto
 speed auto
 media-type rj45
!
router ospf 1
 network 192.168.2.0 0.0.0.255 area 0
 network 192.168.12.0 0.0.0.255 area 0
 network 192.168.23.0 0.0.0.255 area 0
!
end
```
{% endtab %}

{% tab title="R3" %}
```
hostname R3
!
ip multicast-routing 
!
interface GigabitEthernet0/1
 ip address 192.168.13.3 255.255.255.0
 ip pim dense-mode
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/2
 ip address 192.168.23.3 255.255.255.0
 ip pim dense-mode
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/3
 ip address 192.168.3.254 255.255.255.0
 ip pim dense-mode
 duplex auto
 speed auto
 media-type rj45
!
router ospf 1
 network 192.168.3.0 0.0.0.255 area 0
 network 192.168.13.0 0.0.0.255 area 0
 network 192.168.23.0 0.0.0.255 area 0
!
end
```
{% endtab %}

{% tab title="S1" %}
```
hostname S1
!
no ip routing
!
interface GigabitEthernet0/0
 ip address 10.255.1.200 255.255.0.0
 no ip route-cache
 shutdown
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/1
 ip address 192.168.1.1 255.255.255.0
 no ip route-cache
 duplex auto
 speed auto
 media-type rj45
!
ip default-gateway 192.168.1.254
!
end
```
{% endtab %}
{% endtabs %}

I hope this lesson has been useful to understand PIM dense mode, if you have any questions feel free to leave a comment.
