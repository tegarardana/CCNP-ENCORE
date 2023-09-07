# Multicast PIM Auto-RP

Multicast PIM sparse mode requires an RP (Rendezvous Point) as a meeting point in the network for all multicast traffic. In our [first PIM sparse mode lesson](https://networklessons.com/cisco/ccnp-encor-350-401/multicast-pim-sparse-mode), we manually configured the RP on all routers.

For a small network, this is no problem. On large multicast networks, it’s not a good idea. First of all, it takes time to manually configure each router, but it’s also prone to errors. When your RP fails, there’s no automatic way to recover from it. You’ll have to configure all routers to use another router as the RP.

Luckily for us, there are two discovery protocols that we can use to automatically find an RP on the network:

* Auto-RP
* PIMv2 bootstrap router (BSR)

This lesson is about Auto-RP so we’ll cover BSR in another lesson. Auto-RP is a Cisco proprietary protocol that uses two different roles:

* **Candidate RP**
* **Mapping Agent**

The candidate RP is a router that announces itself that it wants to be an RP for the network. It does so by sending **RP announcement** packets to the **224.0.1.39 multicast address**.

The mapping agent listens to the RP announcement packets from our RP candidates and makes a list of all possible RPs. It will then elect an RP and informs the rest of the network with **RP mapping** packets that are sent to **multicast address 224.0.1.40.**

Let’s look at an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/04/multicast-pim-auto-rp-example.png" alt=""><figcaption></figcaption></figure>

Above we have a small network with 6 routers. R1 and R5 are advertising themselves as rendezvous points in the network. These two routers will send their RP announcements packets to 224.0.1.39 on all directly connected interfaces where PIM is enabled. One of our routers will become the mapping agent, take a look below:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/04/multicast-pim-auto-rp-mapping-advertised.png" alt=""><figcaption></figcaption></figure>

Above you can see that R3 is the mapping agent in our network. It has received the RP announcement packets from R1 and R5, and has elected R5 as the RP that will be advertised to the network. When two candidate RPs want to become RP for the same group(s) then the mapping agent will prefer the RP with the **highest IP address**.

R3 will advertise RP mapping packets that contain the RP address of R5 and the groups that it will serve. These are sent on all directly connected interfaces where PIM is enabled.

So far so good, there is one problem though. Above you can see that R3 sends these packets to R1, R4 and R5. These routers have now learned which RP they can use.

What about R2 and R6? Take a look below:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/04/multicast-pim-auto-rp-mapping-sparse-mode.png" alt=""><figcaption></figcaption></figure>

R2 and R6 didn’t receive the RP mapping packets. Why?

Keep in mind we are using PIM sparse mode here, multicast traffic is only forwarded **when a router requests it** with a PIM join message. This PIM join message is sent to the RP.

* R1 and R4 are not going to forward multicast packets to 224.0.1.40 to R2 unless R2 requests it.
* R5 and R4 are not going to forward multicast packets to 224.0.1.40 to R6 unless R6 requests it.

PIM dense mode works the other way around. Multicast traffic is **flooded** everywhere and then **pruned** with PIM prune messages.

It’s a classical chicken and egg problem. When R2/R6 want to receive traffic from 224.0.1.40, they’ll have to send a PIM join to the RP address for 224.0.1.40. However, they have no idea what the RP address is…

How do we solve it? There are two options:

* **PIM Sparse-Dense mode**
* **PIM Auto-RP Listener**

[PIM sparse-dense](https://networklessons.com/cisco/ccnp-encor-350-401/multicast-pim-sparse-dense-mode) mode is an extension to PIM sparse mode. It allows our multicast routers to use dense mode for multicast groups that don’t have an RP configured. When your router knows the RP, it will use sparse mode. If it doesn’t know the RP…it will use dense mode. This allows traffic to 224.0.1.40 to be flooded throughout the network so that R2 and R6 can also learn the RP address.

The second option, auto-rp listener is a bit similar. If you enable this then the router will use dense mode only for the 224.0.1.39 and 224.0.1.40 addresses.

In the example above, R2 and R6 were unable to learn the RP address since they didn’t receive the RP mapping packets to 224.0.1.40. The same problem can occur with the mapping agent. Since R3 is directly connected to R1 and R5, it was able to receive the RP announce packets to 224.0.1.39. If R3 was not directly connected to the RPs, then it would not have received these packets.

## Configuration

Let’s take a look at Auto RP in action. To demonstrate this, I will use the following topology:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/04/multicast-auto-rp-topology.png" alt=""><figcaption></figcaption></figure>

Above we have 4 routers. Our goal is that all routers should learn the RP address. R1 is the RP, R2 is the mapping agent. OSPF has been pre-configured so that all networks are reachable.

First, we need to enable multicast routing on all routers:

<pre><code>R1, R2, R3 &#x26; R4
<strong>(config)#ip multicast-routing
</strong></code></pre>

Let’s enable PIM sparse mode on all interfaces:

<pre><code><strong>R1(config)#interface GigabitEthernet 0/1
</strong><strong>R1(config-if)#ip pim sparse-mode
</strong></code></pre>

<pre><code><strong>R2(config)#interface GigabitEthernet 0/1
</strong><strong>R2(config-if)#ip pim sparse-mode 
</strong>
<strong>R2(config)#interface GigabitEthernet 0/2
</strong><strong>R2(config-if)#ip pim sparse-mode 
</strong></code></pre>

<pre><code><strong>R3(config)#interface GigabitEthernet 0/1
</strong><strong>R3(config-if)#ip pim sparse-mode 
</strong>
<strong>R3(config)#interface GigabitEthernet 0/2
</strong><strong>R3(config-if)#ip pim sparse-mode 
</strong></code></pre>

<pre><code><strong>R4(config)#interface GigabitEthernet 0/1
</strong><strong>R4(config-if)#ip pim sparse-mode
</strong></code></pre>

Before we configure the RP and mapping agent, let’s take a look at the multicast routing tables:

<pre><code><strong>R1#show ip mroute | begin 224
</strong>(*, 224.0.1.40), 00:03:01/00:02:07, RP 0.0.0.0, flags: DCL
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/1, Forward/Sparse, 00:02:59/00:00:08
</code></pre>

<pre><code><strong>R2#show ip mroute | begin 224
</strong>(*, 224.0.1.40), 00:02:50/00:02:07, RP 0.0.0.0, flags: DCL
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/1, Forward/Sparse, 00:02:48/00:02:07
</code></pre>

<pre><code><strong>R3#show ip mroute | begin 224
</strong>(*, 224.0.1.40), 00:02:09/00:02:59, RP 0.0.0.0, flags: DCL
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/1, Forward/Sparse, 00:02:07/00:00:55
</code></pre>

<pre><code><strong>R4#show ip mroute | begin 224
</strong>(*, 224.0.1.40), 00:02:04/00:02:59, RP 0.0.0.0, flags: DCL
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/1, Forward/Sparse, 00:02:02/00:02:59
</code></pre>

Each of these routers is listening to the 224.0.1.40 address. This is the address where the mapping agent will send its RP mapping packets to.

Let’s configure R1 to announce itself as the RP. I will create a new loopback interface for this that will be advertised in OSPF:

<pre><code><strong>R1(config)#interface loopback 0
</strong><strong>R1(config-if)#ip address 1.1.1.1 255.255.255.255
</strong><strong>R1(config-if)#ip pim sparse-mode 
</strong></code></pre>

<pre><code><strong>R1(config)#router ospf 1
</strong><strong>R1(config-router)#network 1.1.1.1 0.0.0.0 area 0
</strong></code></pre>

Now we can announce it’s candidacy with the “ip pim send-rp-announce” command:

<pre><code><strong>R1(config)#ip pim send-rp-announce loopback 0 ?
</strong>  scope  RP announcement scope
</code></pre>

Besides the interface we can also configure the scope by setting a TTL:

<pre><code><strong>R1(config)#ip pim send-rp-announce loopback 0 scope ?
</strong>  &#x3C;1-255>  TTL of the RP announce packet
</code></pre>

This is a small network so it doesn’t matter much what I use. Let’s pick 10:

<pre><code><strong>R1(config)#ip pim send-rp-announce loopback 0 scope 10 ?
</strong>  group-list  Group access-list
  interval    RP announcement interval
</code></pre>

The other thing we could configure is a group-list. By default, the RP will announce itself as an RP for the entire 224.0.0.0/4 range. If you only want to become RP for certain multicast groups, you can add an access-list here.

<pre><code><strong>R1(config)#ip pim send-rp-announce loopback 0 scope 10
</strong></code></pre>

R1 is now advertising itself as an RP. Let’s take a look at its multicast routing table:

<pre><code><strong>R1#show ip mroute | begin 224
</strong>(*, 224.0.1.39), 00:06:13/stopped, RP 0.0.0.0, flags: DP
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list: Null

(1.1.1.1, 224.0.1.39), 00:06:13/00:02:46, flags: PT
  Incoming interface: Loopback0, RPF nbr 0.0.0.0
  Outgoing interface list: Null

(*, 224.0.1.40), 00:15:11/00:02:53, RP 0.0.0.0, flags: DPL
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list: Null
</code></pre>

Above you see an SPT entry for the 224.0.1.39 multicast group address. It doesn’t show any outgoing interfaces but R1 is now flooding its RP announce packets on its interfaces:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/04/multicast-auto-rp-topology-rp-announcement.png" alt=""><figcaption></figcaption></figure>

Here is an example of what this packet looks like:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/04/multicast-pim-auto-rp-announcement-capture.png" alt=""><figcaption></figcaption></figure>

Above you can see the source (1.1.1.1) and destination (224.0.1.39). R1 is announcing itself as the RP for the entire 224.0.0.0/4 range.

Want to take a look at this packet yourself?

[Multicast PIM Auto RP Announcement Capture](https://www.cloudshark.org/captures/588a1bd4f607)

Although R1 is advertising itself as the RP, nothing will happen for now. We need a mapping agent that stores the information from RP and advertises it to the 224.0.1.40 address. We can verify this by looking at R2 again:

<pre><code><strong>R2#show ip mroute | begin 224
</strong>(*, 224.0.1.40), 00:19:14/00:02:36, RP 0.0.0.0, flags: DCL
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/1, Forward/Sparse, 00:19:12/00:02:36
</code></pre>

Above you can see that nothing is going on. Let’s advertise R2 as the mapping agent. Like R1, I will use a loopback interface for this:

<pre><code><strong>R2(config)#interface loopback 0
</strong><strong>R2(config-if)#ip address 2.2.2.2 255.255.255.255
</strong><strong>R2(config-if)#ip pim sparse-mode
</strong></code></pre>

<pre><code><strong>R2(config)#router ospf 1
</strong><strong>R2(config-router)#network 2.2.2.2 0.0.0.0 area 0
</strong></code></pre>

Now we can use the “ip pim send-rp-discovery” command to advertise R2 as a mapping agent:

<pre><code><strong>R2(config)#ip pim send-rp-discovery loopback 0 ?
</strong>  scope  Scope of the RP discovery packets
</code></pre>

We have to set a TTL for these packets. Let’s pick 10:

<pre><code><strong>R2(config)#ip pim send-rp-discovery loopback 0 scope 10
</strong></code></pre>

R2 will now send RP mapping packets on its directly connected interfaces:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/04/multicast-auto-rp-topology-ma-announcement.png" alt=""><figcaption></figcaption></figure>

Here’s what the packet looks like in wireshark:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/04/multicast-pim-auto-rp-mapping-capture.png" alt=""><figcaption></figcaption></figure>

Above you can see that the packet is sourced from 2.2.2.2 and destined to 224.0.1.40. Our mapping agent is advertising R1 as the RP for all multicast groups.

Want to take a look at this packet yourself?

[Multicast PIM Auto RP Mapping Capture](https://www.cloudshark.org/captures/b928cd82e567)

We now have an RP and a mapping agent. Let’s see what our routers think of it:

<pre><code><strong>R1#show ip pim rp mapping 
</strong>PIM Group-to-RP Mappings
This system is an RP (Auto-RP)

Group(s) 224.0.0.0/4
  RP 1.1.1.1 (?), v2v1
    Info source: 2.2.2.2 (?), elected via Auto-RP
         Uptime: 00:01:06, expires: 00:02:54
</code></pre>

R1 knows that it is the RP and that it has been elected by the mapping agent as the active RP. Let’s check R2:

<pre><code><strong>R2#show ip pim rp mapping 
</strong>PIM Group-to-RP Mappings
This system is an RP-mapping agent (Loopback0)

Group(s) 224.0.0.0/4
  RP 1.1.1.1 (?), v2v1
    Info source: 1.1.1.1 (?), elected via Auto-RP
         Uptime: 00:01:16, expires: 00:02:43
</code></pre>

R2 knows that it is the mapping agent and that R1 is elected as the RP. Let’s check R3:

<pre><code><strong>R3#show ip pim rp mapping
</strong>PIM Group-to-RP Mappings

Group(s) 224.0.0.0/4
  RP 1.1.1.1 (?), v2v1
    Info source: 2.2.2.2 (?), elected via Auto-RP
         Uptime: 00:02:54, expires: 00:02:01
</code></pre>

R3 is directly connected to the mapping agent so it has received the RP mapping packets. It has learned that R1 is the RP in this network.

Last but not least, let’s check R4:

<pre><code><strong>R4#show ip pim rp mapping
</strong>PIM Group-to-RP Mappings
</code></pre>

R4 doesn’t know anything…

This happened because R4 is behind R3, it’s not receiving any of the RP mapping packets from our mapping agent. Before we fix this, let’s take a closer look at the multicast routing tables:

<pre><code><strong>R1#show ip mroute | begin 224
</strong>(*, 224.0.1.39), 00:19:29/stopped, RP 0.0.0.0, flags: DP
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list: Null

(1.1.1.1, 224.0.1.39), 00:19:29/00:02:30, flags: PT
  Incoming interface: Loopback0, RPF nbr 0.0.0.0
  Outgoing interface list: Null

(*, 224.0.1.40), 00:28:27/stopped, RP 0.0.0.0, flags: DPL
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list: Null

(2.2.2.2, 224.0.1.40), 00:04:29/00:02:29, flags: PLT
  Incoming interface: GigabitEthernet0/1, RPF nbr 192.168.12.2
  Outgoing interface list: Null
</code></pre>

Above you can see that R1 is receiving packets from the mapping agent. Let’s check R2:

<pre><code><strong>R2#show ip mroute | begin 224
</strong>(*, 224.0.1.39), 00:05:27/stopped, RP 0.0.0.0, flags: DCL
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    Loopback0, Forward/Sparse, 00:05:27/00:02:04
    GigabitEthernet0/1, Forward/Sparse, 00:05:27/00:02:36

(1.1.1.1, 224.0.1.39), 00:04:28/00:02:30, flags: LT
  Incoming interface: GigabitEthernet0/1, RPF nbr 192.168.12.1
  Outgoing interface list:
    Loopback0, Forward/Sparse, 00:04:28/00:02:04

(*, 224.0.1.40), 00:28:13/stopped, RP 0.0.0.0, flags: DCL
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    Loopback0, Forward/Sparse, 00:05:27/00:02:02
    GigabitEthernet0/1, Forward/Sparse, 00:28:11/00:02:41

(2.2.2.2, 224.0.1.40), 00:04:28/00:02:29, flags: LT
  Incoming interface: Loopback0, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/1, Forward/Sparse, 00:04:28/00:02:41
</code></pre>

Above you can see that R2 is receiving the RP announcement traffic from R1. Let’s check R3:

<pre><code><strong>R3#show ip mroute | begin 224
</strong>(*, 224.0.1.39), 00:05:28/00:01:57, RP 0.0.0.0, flags: DC
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/2, Forward/Sparse, 00:05:28/00:01:57

(*, 224.0.1.40), 00:27:34/stopped, RP 0.0.0.0, flags: DPL
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list: Null

(2.2.2.2, 224.0.1.40), 00:04:29/00:02:29, flags: PLT
  Incoming interface: GigabitEthernet0/2, RPF nbr 192.168.23.2
  Outgoing interface list: Null
</code></pre>

Above you can see that R3 is receiving traffic from the mapping agent, we can also verify that it’s not forwarding this traffic towards R4. This is normal since we are using PIM sparse mode. It would only be forwarded when requested by a downstream router.

Here’s R4:

<pre><code><strong>R4#show ip mroute | begin 224
</strong>(*, 224.0.1.40), 00:27:27/00:02:29, RP 0.0.0.0, flags: DCL
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/1, Forward/Sparse, 00:27:25/00:02:29
</code></pre>

R4 is not receiving anything.

To fix this issue, we need to make sure that traffic for the 224.0.1.40 address gets flooded. We can use PIM sparse-dense mode or PIM auto-rp listener for this.

I’m going to use the second option:

<pre><code><strong>R3(config)#ip pim autorp listener
</strong></code></pre>

This command has a very confusing name. What it really does is making sure that multicast traffic that is sent to 224.0.1.39 and 224.0.1.40 gets flooded with dense mode. When we enable it on R3, it will flood the packets to R4:\


<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/04/multicast-auto-rp-topology-rp-announcement-r3.png" alt=""><figcaption></figcaption></figure>

This will allow R4 to receive the information from the mapping agent and to learn the RP address. Let’s verify this:

<pre><code><strong>R4#show ip pim rp mapping
</strong>PIM Group-to-RP Mappings

Group(s) 224.0.0.0/4
  RP 1.1.1.1 (?), v2v1
    Info source: 2.2.2.2 (?), elected via Auto-RP
         Uptime: 00:02:19, expires: 00:02:37
</code></pre>

Excellent, R4 has learned the RP address. Let’s check its multicast routing table again:

<pre><code><strong>R4#show ip mroute | begin 224
</strong>(*, 224.0.1.40), 00:47:53/stopped, RP 0.0.0.0, flags: DCL
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/1, Forward/Sparse, 00:47:51/00:02:01

(2.2.2.2, 224.0.1.40), 00:03:05/00:02:53, flags: PLTX
  Incoming interface: GigabitEthernet0/1, RPF nbr 192.168.34.3
  Outgoing interface list: Null
</code></pre>

Above you can see the incoming traffic. Mission accomplished, all routers have now learned the RP address.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
ip multicast-routing 
ip cef
!
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
 ip pim sparse-mode
!
interface GigabitEthernet0/1
 ip address 192.168.12.1 255.255.255.0
 ip pim sparse-mode
!
router ospf 1
 network 1.1.1.1 0.0.0.0 area 0
 network 192.168.12.0 0.0.0.255 area 0
!
ip pim send-rp-announce Loopback0 scope 10
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
interface Loopback0
 ip address 2.2.2.2 255.255.255.255
 ip pim sparse-mode
!
interface GigabitEthernet0/1
 ip address 192.168.12.2 255.255.255.0
 ip pim sparse-mode
!
interface GigabitEthernet0/2
 ip address 192.168.23.2 255.255.255.0
 ip pim sparse-mode
!
router ospf 1
 network 2.2.2.2 0.0.0.0 area 0
 network 192.168.12.0 0.0.0.255 area 0
 network 192.168.23.0 0.0.0.255 area 0
!
ip pim send-rp-discovery Loopback0 scope 10
!
end
```
{% endtab %}

{% tab title="R3" %}
```
hostname R3
!
ip multicast-routing 
ip cef
!
interface GigabitEthernet0/1
 ip address 192.168.34.3 255.255.255.0
 ip pim sparse-mode
!
interface GigabitEthernet0/2
 ip address 192.168.23.3 255.255.255.0
 ip pim sparse-mode
!
router ospf 1
 network 192.168.23.0 0.0.0.255 area 0
 network 192.168.34.0 0.0.0.255 area 0
!
ip pim autorp listener
!
end
```
{% endtab %}

{% tab title="R4" %}
```
hostname R4
!
ip multicast-routing 
ip cef
!
interface GigabitEthernet0/1
 ip address 192.168.34.4 255.255.255.0
 ip pim sparse-mode
!
router ospf 1
 network 192.168.34.0 0.0.0.255 area 0
!
end
```
{% endtab %}
{% endtabs %}

## Conclusion

Here’s what we have learned about Auto-RP in this lesson:

* Candidate RPs are routers that advertises themselves that they want to be the RP for the network. They send packets to the 224.0.1.39 address.
* Mapping agents are routers that listen to the 224.0.1.39 address and build a list of all possible RPs in the network. They elect an RP and advertise this to the rest of the work by using the 224.0.1.40 address.
* The problem with Auto-RP is that some routers might be unable to receive packets from the mapping agent, and some mapping agents might be unable to hear all RPs. This is because PIM sparse mode doesn’t flood traffic on all interfaces. To fix this, you can use pim sparse-dense mode or ip pim auto-rp listener. Both will ensure that traffic to 224.0.1.39 and 224.0.1.40 gets flooded throughout the network.

In another lesson, we’ll take a look at BSR (Bootstrap) which is an alternative to Auto-RP. I hope this lesson has been useful for you!
