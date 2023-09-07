# Multicast PIM Sparse Mode

In this lesson we’ll take a look at PIM sparse mode which works about the opposite of how [PIM dense mode](https://networklessons.com/cisco/ccnp-encor-350-401/multicast-pim-dense-mode) works. With PIM dense mode, we flood multicast traffic everywhere and then we prune it.

With PIM sparse mode we don’t forward any multicast traffic unless someone **requests it**.

This introduces one problem…how does a router know where to find the source of multicast traffic? Let me show you a picture to explain this:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/02/multicast-pim-sparse-without-rp.png" alt=""><figcaption></figcaption></figure>

Above we have a video server that is streaming multicast traffic to 239.1.1.1. At the bottom there’s R6 who received an IGMP membership from a directly connected host who would like to receive this multicast traffic.

How is R6 supposed to know that the video server at 192.168.1.1 is sending multicast traffic for this group? With PIM dense mode, this was simple. Multicast traffic was flooded everywhere which allowed R6 to learn the source address.

To solve this issue, PIM sparse mode uses a **RP (Rendezvous Point)** in the network. Here’s how it works:

* Each router that receives multicast traffic from a source will forward it to the RP.
* Each router that wants to receive multicast traffic will go to the RP.

The RP is like a “meeting point” for multicast traffic. Here’s an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/02/multicast-pim-sparse-example-topology.png" alt=""><figcaption></figcaption></figure>

Above you can see that R5 has become the RP in our network. R1 receives multicast traffic from the video server and forwards it to the RP. At the bottom we have R6 that is receiving this multicast traffic from the RP and it is forwarded to the host.

This does introduce another problem though…how does R6 know where to find the RP? There are a couple of different options for this:

* We can manually configure the IP address of the RP on each router.
* There are some protocols that can automatically discover the RP in our network. We’ll talk about this later.

Let’s add some more detail to this story. Let’s imagine that we are using the topology above but _at the moment, nobody is interested in the multicast traffic_. Here’s what happens:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/02/pim-multicast-register-for-rp.png" alt=""><figcaption></figcaption></figure>

Our video server is forwarding multicast traffic on its interface which is received by R1. Since we are using PIM sparse mode, this router will have to forward the multicast traffic to the RP. Instead of forwarding everything, R1 will only send the **first multicast packet**. This packet is encapsulated in a **PIM register** message and forwarded to the RP. Once the RP receives the PIM register message there are two options:

* When nobody is interested in the multicast traffic then the RP will **reject** the PIM register message.
* When there is at least one receiver, the RP **accepts** the RP register message.

For now, let’s say that we don’t have any receivers:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/02/pim-multicast-register-stop-from-rp.png" alt=""><figcaption></figcaption></figure>

Our RP will respond to R1 with a **PIM register stop** message. This will inform R1 to stop forwarding any multicast traffic for now. R1 will remain silent for now but not for long:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/02/pim-multicast-register-suppression-timer.png" alt=""><figcaption></figcaption></figure>

When we receive the PIM register stop packet, R1 will start a **suppression timer**. By default this timer is **60 seconds** and when the timer is almost expired, R1 will send another packet:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/02/pim-multicast-register-null-for-rp.png" alt=""><figcaption></figcaption></figure>

R1 will send another PIM register message but this one doesn’t carry the encapsulated multicast packet. It’s a simple request to ask the RP if it is interested now. This packet is called the **PIM register null** packet. When we still don’t have any receivers, the RP will send another PIM register stop message. When we do have receivers, the RP will not send a PIM register stop message and R1 will start forwarding the multicast traffic.

Now you know what happens when a source starts sending multicast traffic without any receivers. Let’s see what happens when we _do_ have receivers shall we? Take a look at the picture below:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/02/multicast-pim-sparse-pim-join.png" alt=""><figcaption></figcaption></figure>

The host that is connected to R6 would like to receive multicast traffic so it sends an IGMP membership report for the multicast group it wants. R6 now has to figure out how to get to the RP and request it to start forwarding the multicast traffic. It will check its unicast routing table for the IP address of the RP and sends a PIM join message on the interface that is used to reach the RP. In this case, that means the PIM join is forwarded towards R4.

When R4 receives the PIM join, it has to request the RP to start forwarding multicast traffic so it will also send a PIM join. It will check its unicast routing table, finds the interface that is used to reach the RP and sends a PIM join message towards the RP.

To summarize this, PIM sparse routers will send a PIM join message when:

* The router has received an IGMP membership report from a host on a directly connected interface.
* The router has received a PIM join from a downstream router.

When the RP receives the PIM join, it will start forwarding the multicast traffic.

This concept of joining the RP is called the **RPT (Root Path Tree)** or **shared distribution tree**. The RP is the root of our tree which decides where to forward multicast traffic to. Each multicast group might have different sources and receivers so we might have different RPTs in our network.

The end result will look like this:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/02/multicast-pim-sparse-example-topology.png" alt=""><figcaption></figcaption></figure>

Multicast traffic is now flowing from R1 towards the RP, down to R4, R6 and to our receiver (H1).

Are we done now? Not quite…

If you look closely at the picture above then you might have noticed that R6 has multiple paths towards the source. Right now our multicast traffic is flowing like this:

* R1 > R5 > R4 > R6

It’s possible however that this is not the most optimal path. The path from R1 > R2 > R6 has one less router than the current path so if all interfaces are equal, this path is probably better.

Once we are receiving multicast traffic through the RP, it’s possible to switch to the **SPT (Source Path Tree)**.

Here’s what happens next:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/02/multicast-pim-sparse-rpt-to-spt-switch.png" alt=""><figcaption></figcaption></figure>

When R6 has received the multicast traffic through the RP, it also learned the **source address** of this traffic. R6 can now decide it wants to use the SPT instead of the RPT to receive this traffic. When it wants this, it will send PIM join messages towards the source. R1 and R2 will start forwarding multicast traffic towards R6, using the best path:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/02/multicast-pim-sparse-prune-for-rp.png" alt=""><figcaption></figcaption></figure>

Since R6 is now receiving multicast traffic through R2 and R1, it doesn’t need it from the RP anymore. It will send PIM prune messages towards the RP, informing any routers in between that they can stop forwarding multicast traffic for this group.

You should now have a general idea of how PIM sparse mode works. Let’s configure a small network so you can see this in action!

## Configuration

We will use the following topology for this example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/02/multicast-pim-sparse-lab-topology.png" alt=""><figcaption></figcaption></figure>

R1, R2, R3 and R4 are our multicast routers that we will configure for PIM sparse mode. S1 will be the source that sends multicast traffic. H3 and H4 are two receivers.

Our routers are running OSPF so that we have full connectivity.

{% tabs %}
{% tab title="Configuration" %}
Want to follow me with this example? Here you will find the startup configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
interface GigabitEthernet0/1
 ip address 192.168.12.1 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/2
 ip address 192.168.14.1 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/3
 ip address 192.168.1.254 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
router ospf 1
 network 192.168.1.0 0.0.0.255 area 0
 network 192.168.12.0 0.0.0.255 area 0
 network 192.168.14.0 0.0.0.255 area 0
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
interface GigabitEthernet0/1
 ip address 192.168.12.2 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/2
 ip address 192.168.24.2 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/3
 ip address 192.168.23.2 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
router ospf 1
 network 192.168.12.0 0.0.0.255 area 0
 network 192.168.23.0 0.0.0.255 area 0
 network 192.168.24.0 0.0.0.255 area 0
!
end
```
{% endtab %}

{% tab title="R3" %}
```
hostname R3
!
interface GigabitEthernet0/1
 ip address 192.168.23.3 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/2
 ip address 192.168.3.254 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
router ospf 1
 network 192.168.3.0 0.0.0.255 area 0
 network 192.168.23.0 0.0.0.255 area 0
!
end
```
{% endtab %}

{% tab title="R4" %}
```
hostname R4
!
interface GigabitEthernet0/1
 ip address 192.168.14.4 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/2
 ip address 192.168.24.4 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/3
 ip address 192.168.4.254 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
router ospf 1
 network 192.168.4.0 0.0.0.255 area 0
 network 192.168.14.0 0.0.0.255 area 0
 network 192.168.24.0 0.0.0.255 area 0
!
end
```
{% endtab %}

{% tab title="H3" %}
```
hostname H3
!
interface GigabitEthernet0/1
 ip address 192.168.3.3 255.255.255.0
 no ip route-cache
 duplex auto
 speed auto
 media-type rj45
!
ip default-gateway 192.168.3.254
!
end
```
{% endtab %}

{% tab title="H4" %}
```
hostname H4
!
interface GigabitEthernet0/1
 ip address 192.168.4.4 255.255.255.0
 no ip route-cache
 duplex auto
 speed auto
 media-type rj45
!
ip default-gateway 192.168.4.254
!
end
```
{% endtab %}

{% tab title="S1" %}
```
hostname S1
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

Now we can focus on the multicast configuration.

### PIM Sparse Mode

Multicast routing is **disabled by default** on Cisco IOS routers so we have to enable it:

<pre><code>R1, R2, R3 &#x26; R4
<strong>(config)#ip multicast-routing
</strong></code></pre>

Now we can configure PIM. Let’s start with R1:

<pre><code><strong>R1(config)#interface GigabitEthernet 0/1
</strong><strong>R1(config-if)#ip pim sparse-mode
</strong></code></pre>

As soon as I enable PIM sparse mode on the interface our router will start sending PIM hello packets. Here’s what these look like in wireshark:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/02/multicast-pim-sparse-hello-packet.png" alt=""><figcaption></figcaption></figure>

Above you can see the type of the packet (hello) and it also shows the hold time. For whatever reason, wireshark (version 2.0.1) is showing me a value of 1 but the default is 105 seconds. If you want to take a look at this packet yourself, here’s the capture file if you want to take a look:

[PIM Sparse Mode Hello Packet](https://www.cloudshark.org/captures/64a4a26f412f)

It shows the correct holdtime of 105 seconds.

Let’s enable PIM sparse mode on R2 as well:

<pre><code><strong>R2(config)#interface GigabitEthernet 0/1
</strong><strong>R2(config-if)#ip pim sparse-mode 
</strong></code></pre>

As soon as we do this, R1 and R2 will become neighbors since they receive each others PIM hello packets.If they keep receiving them before the holddown timer is expired then they will keep seeing each other as PIM neighbors.:

```
R1#
%PIM-5-NBRCHG: neighbor 192.168.12.2 UP on interface GigabitEthernet0/1
```

Let’s enable all the other interfaces. You need to enable PIM on all interfaces that will send or receive multicast traffic. **Including the interfaces that connects to sources and receivers**:

<pre><code><strong>R1(config)#interface GigabitEthernet 0/2
</strong><strong>R1(config-if)#ip pim sparse-mode 
</strong>
<strong>R1(config)#interface GigabitEthernet 0/3
</strong><strong>R1(config-if)#ip pim sparse-mode
</strong></code></pre>

<pre><code><strong>R2(config)#interface GigabitEthernet 0/2
</strong><strong>R2(config-if)#ip pim sparse-mode
</strong>
<strong>R2(config)#interface GigabitEthernet 0/3
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
</strong>
<strong>R4(config)#interface GigabitEthernet 0/2
</strong><strong>R4(config-if)#ip pim sparse-mode 
</strong>
<strong>R4(config)#interface GigabitEthernet 0/3
</strong><strong>R4(config-if)#ip pim sparse-mode 
</strong></code></pre>

That takes care of the interfaces…

### PIM Sparse Neighbors

Let’s make sure all routers have become neighbors:

<pre><code><strong>R1#show ip pim neighbor 
</strong>PIM Neighbor Table
Mode: B - Bidir Capable, DR - Designated Router, N - Default DR Priority,
      P - Proxy Capable, S - State Refresh Capable, G - GenID Capable,
      L - DR Load-balancing Capable
Neighbor          Interface                Uptime/Expires    Ver   DR
Address                                                            Prio/Mode
192.168.12.2      GigabitEthernet0/1       00:00:17/00:01:27 v2    1 / DR S P G
192.168.14.4      GigabitEthernet0/2       00:00:15/00:01:29 v2    1 / DR S P G
</code></pre>

<pre><code><strong>R2#show ip pim neighbor 
</strong>PIM Neighbor Table
Mode: B - Bidir Capable, DR - Designated Router, N - Default DR Priority,
      P - Proxy Capable, S - State Refresh Capable, G - GenID Capable,
      L - DR Load-balancing Capable
Neighbor          Interface                Uptime/Expires    Ver   DR
Address                                                            Prio/Mode
192.168.12.1      GigabitEthernet0/1       00:00:40/00:01:36 v2    1 / S P G
192.168.24.4      GigabitEthernet0/2       00:00:38/00:01:35 v2    1 / DR S P G
192.168.23.3      GigabitEthernet0/3       00:00:37/00:01:36 v2    1 / DR S P G
</code></pre>

<pre><code><strong>R3#show ip pim neighbor 
</strong>PIM Neighbor Table
Mode: B - Bidir Capable, DR - Designated Router, N - Default DR Priority,
      P - Proxy Capable, S - State Refresh Capable, G - GenID Capable,
      L - DR Load-balancing Capable
Neighbor          Interface                Uptime/Expires    Ver   DR
Address                                                            Prio/Mode
192.168.23.2      GigabitEthernet0/1       00:00:55/00:01:22 v2    1 / S P G
</code></pre>

<pre><code><strong>R4#show ip pim neighbor 
</strong>PIM Neighbor Table
Mode: B - Bidir Capable, DR - Designated Router, N - Default DR Priority,
      P - Proxy Capable, S - State Refresh Capable, G - GenID Capable,
      L - DR Load-balancing Capable
Neighbor          Interface                Uptime/Expires    Ver   DR
Address                                                            Prio/Mode
192.168.14.1      GigabitEthernet0/1       00:01:11/00:01:33 v2    1 / S P G
192.168.24.2      GigabitEthernet0/2       00:01:11/00:01:35 v2    1 / S P G
</code></pre>

Above we can see that all routers have become PIM neighbors. The default version on Cisco IOS is PIM version 2. The expiry time is the holddown timer that is counting down. Each time when the router receives a PIM hello packet, it will reset itself to 00:01:45 (105 seconds).

### Multicast Routing

Nothing is going on at the moment, let’s see if there is anything in the multicast routing tables:

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

(*, 224.0.1.40), 00:01:35/00:02:29, RP 0.0.0.0, flags: DCL
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/2, Forward/Sparse, 00:01:34/00:01:25
    GigabitEthernet0/1, Forward/Sparse, 00:01:35/00:01:23
</code></pre>

Above we see the output of R1. At this moment we are not transmitting/receiving any multicast traffic but we still see something in the multicast routing tables. There’s an entry for 224.0.1.40. This multicast group address is used for Auto RP discovery, a protocol that dynamically learns RPs on the network.

The multicast routing table shows a lot of different flags, we’ll discuss some of them in this lesson. To save some virtual real estate pixel space I won’t show them over and over again when we look at the multicast routing tables.

The other router multicast routing tables are similar:

<pre><code><strong>R2#show ip mroute 
</strong>
(*, 224.0.1.40), 00:02:00/00:02:10, RP 0.0.0.0, flags: DCL
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/1, Forward/Sparse, 00:02:00/00:01:59
</code></pre>

<pre><code><strong>R3#show ip mroute 
</strong>
(*, 224.0.1.40), 00:02:16/00:02:47, RP 0.0.0.0, flags: DCL
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/1, Forward/Sparse, 00:02:16/00:02:47
</code></pre>

<pre><code><strong>R4#show ip mroute 
</strong>
(*, 224.0.1.40), 00:02:36/00:02:35, RP 0.0.0.0, flags: DCL
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/1, Forward/Sparse, 00:02:36/00:02:35
</code></pre>

When you are studying multicast it can be annoying to see the autoRP group address, especially in debug messages. Since IOS version 15.0(1)M it’s finally possible to disable auto RP. We don’t need it in this lesson so let’s disable it:

<pre><code>R1, R2, R3 &#x26; R4
<strong>(config)#no ip pim autorp
</strong></code></pre>

### Rendezvous Point

In this lesson we are going to keep it simple, instead of learning the RP dynamically we are going to configure it manually. Right now we don’t have a RP so it’s time configure one. R2 will be our RP and we’ll use one of its IP addresses as the RP address. Physical interfaces can go down so it’s better to use a loopback interface for the RP address:

<pre><code><strong>R2(config)#interface Loopback 0
</strong><strong>R2(config-if)#ip address 2.2.2.2 255.255.255.255
</strong>
<strong>R2(config)#router ospf 1
</strong><strong>R2(config-router)#network 2.2.2.2 0.0.0.0 area 0
</strong></code></pre>

The only thing left to do is to tell each router (including R2) that R2 will be the RP in our network:

<pre><code>R1, R2, R3 &#x26; R4:
<strong>(config)#ip pim rp-address 2.2.2.2
</strong></code></pre>

All of our routers now know that R2 will be the RP in this network. Configuring a static RP address is quick and simple but there are some disadvantages:

* We don’t have any redundancy. When R2 fails, we have to configure another RP manually.
* If you have a lot of multicast-enabled routers then it’s a tedious task to configure all your routers.

In other lessons I’ll show you how to use AutoRP and BSR (Bootstrap) to automatically learn RPs.

Once you have configured the RP you might notice some messages on your consoles:

```
R1#
%LINEPROTO-5-UPDOWN: Line protocol on Interface Tunnel0, changed state to up
```

```
R2#
%LINEPROTO-5-UPDOWN: Line protocol on Interface Tunnel0, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface Tunnel1, changed state to up
```

```
R3#
%LINEPROTO-5-UPDOWN: Line protocol on Interface Tunnel0, changed state to up
```

```
R4#
%LINEPROTO-5-UPDOWN: Line protocol on Interface Tunnel0, changed state to up
```

R1, R3 and R4 each create a tunnel0 interface. R2 also creates a tunnel1 interface.

The first tunnel interface is used to encapsulate the first multicast packet in a PIM register message and forward it to the RP. The RP has one additional tunnel interface which is used for decapsulation of the PIM register packet.

These tunnel interfaces won’t show up in your configuration but we can take a look at them with the following command:

<pre><code><strong>R1#show ip pim tunnel 
</strong>Tunnel0 
  Type       : PIM Encap
  RP         : 2.2.2.2
  Source     : 192.168.12.1
  State      : UP
  Last event : Created (00:00:22)
</code></pre>

<pre><code><strong>R2#show ip pim tunnel 
</strong>Tunnel0 
  Type       : PIM Encap
  RP         : 2.2.2.2*
  Source     : 192.168.12.2
  State      : UP
  Last event : Created (00:00:36)
Tunnel1* 
  Type       : PIM Decap
  RP         : 2.2.2.2*
  Source     : -
  State      : UP
  Last event : Created (00:00:36)
</code></pre>

<pre><code><strong>R3#show ip pim tunnel
</strong>Tunnel0 
  Type       : PIM Encap
  RP         : 2.2.2.2
  Source     : 192.168.23.3
  State      : UP
  Last event : Created (00:02:41)
</code></pre>

<pre><code><strong>R4#show ip pim tunnel
</strong>Tunnel0 
  Type       : PIM Encap
  RP         : 2.2.2.2
  Source     : 192.168.24.4
  State      : UP
  Last event : Created (00:03:07)
</code></pre>

Everything is now in place.

### PIM Register

First we will take a look at the PIM register packet. We can see it when we send multicast traffic from a source. Before we do this, let’s enable a debug so you can see what is going on behind the scenes:

<pre><code>R1 &#x26; R2
<strong>#debug ip pim 
</strong>PIM debugging is on
</code></pre>

Let’s generate some traffic:

<pre><code><strong>S1#ping 239.1.1.1 repeat 1000
</strong>Type escape sequence to abort.
Sending 1000, 100-byte ICMP Echos to 239.1.1.1, timeout is 2 seconds:
.....
</code></pre>

Here’s what R1 will tell us:

```
R1#
PIM(0): Check RP 2.2.2.2 into the (*, 239.1.1.1) entry
PIM(0): Building Triggered (*,G) Join / (S,G,RP-bit) Prune message for 239.1.1.1
PIM(0): Adding register encap tunnel (Tunnel0) as forwarding interface of (192.168.1.1, 239.1.1.1).
PIM(0): Received v2 Register-Stop on GigabitEthernet0/1 from 2.2.2.2
PIM(0): for source 192.168.1.1, group 239.1.1.1
PIM(0): Removing register encap tunnel (Tunnel0) as forwarding interface of (192.168.1.1, 239.1.1.1).
PIM(0): Clear Registering flag to 2.2.2.2 for (192.168.1.1/32, 239.1.1.1)
```

R1 encapsulates the first multicast packet that it receives from our source and forwards it with a PIM register message towards the RP. A bit later, it receives a PIM register stop from the RP who asks R1 to stop forwarding anything. Here’s what the PIM register packet looks like:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/02/multicast-pim-sparse-register.png" alt=""><figcaption></figcaption></figure>

Above you can see that the ICMP packet has been encapsulated in a PIM register packet.

Here’s what it looks like from R2’s perspective:

```
R2#
PIM(0): Received v2 Register on GigabitEthernet0/1 from 192.168.12.1
     for 192.168.1.1, group 239.1.1.1
PIM(0): Check RP 2.2.2.2 into the (*, 239.1.1.1) entry
PIM(0): Adding register decap tunnel (Tunnel1) as accepting interface of (*, 239.1.1.1).
PIM(0): Adding register decap tunnel (Tunnel1) as accepting interface of (192.168.1.1, 239.1.1.1).
PIM(0): Send v2 Register-Stop to 192.168.12.1 for 192.168.1.1, group 239.1.1.1
```

Above you can see that R2 receives the PIM register and replies with the PIM register stop. Here’s what this packet looks like:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/02/multicast-pim-sparse-register-stop.png" alt=""><figcaption></figcaption></figure>

When R1’s suppression timer is almost expired, it will send a PIM register null packet to check if the RP is still not interested:

```
R1#
PIM(0): Send v2 Data-header Register to 2.2.2.2 for 192.168.1.1, group 239.1.1.1
PIM(0): Received v2 Register-Stop on GigabitEthernet0/1 from 2.2.2.2
PIM(0):   for source 192.168.1.1, group 239.1.1.1
PIM(0): Clear Registering flag to 2.2.2.2 for (192.168.1.1/32, 239.1.1.1)
```

R2 quickly replies, telling R1 to shut up. Here’s what the PIM register null packet looks like:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/02/multicast-pim-sparse-register-null.png" alt=""><figcaption></figcaption></figure>

Here’s the debug from R2:

```
R2#
PIM(0): Received v2 Register on GigabitEthernet0/1 from 192.168.12.1
     (Data-header) for 192.168.1.1, group 239.1.1.1
PIM(0): Send v2 Register-Stop to 192.168.12.1 for 192.168.1.1, group 239.1.1.1
```

Do you want to take a look at these packets yourself? You can find them below:

[PIM Sparse Mode Register Packets](https://www.cloudshark.org/captures/611435f7d00d)

### Joining the RPT (Root Path Tree)

A multicast network without receivers is no fun so let’s add some. We will let H3 join a new multicast group: 239.3.3.3. Before we do, let’s enable a debug on R1, R2 and R3:

<pre><code>R1, R2 &#x26; R3
<strong>#debug ip pim 
</strong>PIM debugging is on
</code></pre>

Let’s join that group:

<pre><code><strong>H3(config)#interface GigabitEthernet 0/1
</strong><strong>H3(config-if)#ip igmp join-group 239.3.3.3
</strong></code></pre>

R3 will now join the RPT for 239.3.3.3 since it receives the IGMP membership report from H3. Here’s what it looks like:

<pre><code>R3#
<strong>PIM(0): Check RP 2.2.2.2 into the (*, 239.3.3.3) entry
</strong>PIM(0): Building Triggered (*,G) Join / (S,G,RP-bit) Prune message for 239.3.3.3
PIM(0): Upstream mode for (*, 239.3.3.3) changed from 0 to 1
PIM(0): Insert (*,239.3.3.3) join in nbr 192.168.23.2's queue
PIM(0): Building Join/Prune packet for nbr 192.168.23.2
PIM(0):  Adding v2 (2.2.2.2/32, 239.3.3.3), WC-bit, RPT-bit, S-bit Join
PIM(0): Send v2 join/prune to 192.168.23.2 (GigabitEthernet0/1)
</code></pre>

Above you can see the (\*,G) entry which tells us we are joining the SPT. The \* is a wildcard that indicates “any source”. The G is the group, in our case 239.3.3.3.

Here’s what this PIM join packet looks like:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/02/multicast-pim-sparse-join-wireshark-capture.png" alt=""><figcaption></figcaption></figure>

You can download it here if you want:

[PIM Sparse join packet](https://www.cloudshark.org/captures/7e56b654d43a)

Here’s what happens when our RP receives the PIM join:

```
R2#
PIM(0): Received v2 Join/Prune on GigabitEthernet0/3 from 192.168.23.3, to us
PIM(0): Join-list: (*, 239.3.3.3), RPT-bit set, WC-bit set, S-bit set
PIM(0): Check RP 2.2.2.2 into the (*, 239.3.3.3) entry
PIM(0): Adding register decap tunnel (Tunnel1) as accepting interface of (*, 239.3.3.3).
PIM(0): Add GigabitEthernet0/3/192.168.23.3 to (*, 239.3.3.3), Forward state, by PIM *G Join
```

Above you can see that R2 accepts the PIM join and puts its GigabitEthernet0/3 interface in forwarding mode.

Let’s see what the multicast routing tables look like now:

<pre><code><strong>R2#show ip mroute 239.3.3.3
</strong>
(*, 239.3.3.3), 00:06:06/00:03:18, RP 2.2.2.2, flags: S
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/3, Forward/Sparse, 00:06:06/00:03:18
</code></pre>

The output above is interesting. We have a RPT entry for (\*, 239.3.3.3) and the RP is 2.2.2.2. The “S” flag means we are using PIM sparse mode. Nobody is sending traffic to this group at the moment so the incoming interface is null. The outgoing interface is in forwarding mode.

Let’s check R3:

<pre><code><strong>R3#show ip mroute 239.3.3.3
</strong>
(*, 239.3.3.3), 00:06:10/00:02:31, RP 2.2.2.2, flags: SJC
  Incoming interface: GigabitEthernet0/1, RPF nbr 192.168.23.2
  Outgoing interface list:
    GigabitEthernet0/2, Forward/Sparse, 00:06:10/00:02:31
</code></pre>

R3 also has an entry for the RPT. There are three flags:

* S: we are using sparse mode.
* J: this is the SPT-bit. When possible, R3 will try to join the SPT for this multicast group. We’ll see this in a minute…
* C: this means we have a member of this multicast group that is directly connected to us.

Let’s see what happens when we send traffic to this multicast group:

<pre><code><strong>S1#ping 239.3.3.3 repeat 1000
</strong>Type escape sequence to abort.
Sending 1000, 100-byte ICMP Echos to 239.3.3.3, timeout is 2 seconds:
.
Reply to request 1 from 192.168.3.3, 32 ms
</code></pre>

Our receiver is replying. Here’s what the debugs tell us:

```
R1#
PIM(0): Check RP 2.2.2.2 into the (*, 239.3.3.3) entry
PIM(0): Building Triggered (*,G) Join / (S,G,RP-bit) Prune message for 239.3.3.3
PIM(0): Adding register encap tunnel (Tunnel0) as forwarding interface of (192.168.1.1, 239.3.3.3).
PIM(0): Received v2 Join/Prune on GigabitEthernet0/1 from 192.168.12.2, to us
PIM(0): Join-list: (192.168.1.1/32, 239.3.3.3), S-bit set
PIM(0): Add GigabitEthernet0/1/192.168.12.2 to (192.168.1.1, 239.3.3.3), Forward state, by PIM SG Join
```

R1 receives a PIM join from the RP and decides to put its GigabitEthernet0/1 interface into forwarding mode. Here’s what we see on the RP:

```
R2#
PIM(0): Received v2 Register on GigabitEthernet0/1 from 192.168.12.1
     for 192.168.1.1, group 239.3.3.3
PIM(0): Adding register decap tunnel (Tunnel1) as accepting interface of (192.168.1.1, 239.3.3.3).
PIM(0): Insert (192.168.1.1,239.3.3.3) join in nbr 192.168.12.1's queue
PIM(0): Building Join/Prune packet for nbr 192.168.12.1
PIM(0):  Adding v2 (192.168.1.1/32, 239.3.3.3), S-bit Join
PIM(0): Send v2 join/prune to 192.168.12.1 (GigabitEthernet0/1)
PIM(0): Received v2 Join/Prune on GigabitEthernet0/3 from 192.168.23.3, to us
PIM(0): Join-list: (192.168.1.1/32, 239.3.3.3), S-bit set
PIM(0): Update GigabitEthernet0/3/192.168.23.3 to (192.168.1.1, 239.3.3.3), Forward state, by PIM SG Join
PIM(0): Received v2 Register on GigabitEthernet0/1 from 192.168.12.1
     for 192.168.1.1, group 239.3.3.3
PIM(0): Removing register decap tunnel (Tunnel1) as accepting interface of (192.168.1.1, 239.3.3.3).
PIM(0): Installing GigabitEthernet0/1 as accepting interface for (192.168.1.1, 239.3.3.3).
PIM(0): Insert (192.168.1.1,239.3.3.3) join in nbr 192.168.12.1's queue
PIM(0): Building Join/Prune packet for nbr 192.168.12.1
PIM(0):  Adding v2 (192.168.1.1/32, 239.3.3.3), S-bit Join
PIM(0): Send v2 join/prune to 192.168.12.1 (GigabitEthernet0/1)
PIM(0): Received v2 Register on GigabitEthernet0/1 from 192.168.12.1
     for 192.168.1.1, group 239.3.3.3
PIM(0): Send v2 Register-Stop to 192.168.12.1 for 192.168.1.1, group 239.3.3.3
PIM(0): Received v2 Join/Prune on GigabitEthernet0/3 from 192.168.23.3, to us
PIM(0): Join-list: (*, 239.3.3.3), RPT-bit set, WC-bit set, S-bit set
PIM(0): Update GigabitEthernet0/3/192.168.23.3 to (*, 239.3.3.3), Forward state, by PIM *G Join
PIM(0): Update GigabitEthernet0/3/192.168.23.3 to (192.168.1.1, 239.3.3.3), Forward state, by PIM *G Join
PIM(0): Building Periodic (*,G) Join / (S,G,RP-bit) Prune message for 239.3.3.3
PIM(0): Received v2 Join/Prune on GigabitEthernet0/3 from 192.168.23.3, to us
PIM(0): Join-list: (192.168.1.1/32, 239.3.3.3), S-bit set
PIM(0): Update GigabitEthernet0/3/192.168.23.3 to (192.168.1.1, 239.3.3.3), Forward state, by PIM SG Join
```

R2 produces a lot of debug information. We can see that it receives a PIM register with an encapsulated multicast packet that it accepts. It now learns the source (192.168.1.1) of the 239.3.3.3 multicast group so the router will join the SPT.

Once it has joined the SPT, it no longer requires the PIM register packet. We can also see that it has received a PIM join from R3 and that its GigabitEthernet 0/3 interface goes into forwarding mode.

Here’s the output of R3:

```
R3#
PIM(0): Insert (192.168.1.1,239.3.3.3) join in nbr 192.168.23.2's queue
PIM(0): Building Join/Prune packet for nbr 192.168.23.2
PIM(0):  Adding v2 (192.168.1.1/32, 239.3.3.3), S-bit Join
PIM(0): Send v2 join/prune to 192.168.23.2 (GigabitEthernet0/1)
```

R3 already joined the RPT for 239.3.3.3 but since it now has learned the source, it will join the SPT (note the S-bit Join).

Let’s check the multicast routing tables:

<pre><code><strong>R1#show ip mroute 239.3.3.3
</strong>
(*, 239.3.3.3), 00:05:01/stopped, RP 2.2.2.2, flags: SPF
  Incoming interface: GigabitEthernet0/1, RPF nbr 192.168.12.2
  Outgoing interface list: Null

(192.168.1.1, 239.3.3.3), 00:05:01/00:02:59, flags: FT
  Incoming interface: GigabitEthernet0/3, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/1, Forward/Sparse, 00:05:01/00:02:36
</code></pre>

Above we can see that R1 is receiving traffic from 192.168.1.1 which is forwarded on its GigabitEthernet0/1 interface. The P-bit for the RPT (\*, 239.3.3.3) entry indicates that it has been pruned. The T-bit for the SPT (192.168.1.1,239.3.3.3) entry tells us that it has joined the SPT for this group.

Let’s check the RP:

<pre><code><strong>R2#show ip mroute 239.3.3.3
</strong>
(*, 239.3.3.3), 00:20:25/00:02:42, RP 2.2.2.2, flags: S
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/3, Forward/Sparse, 00:11:13/00:02:42

(192.168.1.1, 239.3.3.3), 00:05:19/00:02:38, flags: T
  Incoming interface: GigabitEthernet0/1, RPF nbr 192.168.12.1
  Outgoing interface list:
    GigabitEthernet0/3, Forward/Sparse, 00:05:19/00:03:02
</code></pre>

Above we see that the RP has joined the SPT and that it is forwarding on its GigabitEthernet0/3 interface. Last but not least, let’s check R3:

<pre><code><strong>R3#show ip mroute 239.3.3.3
</strong>
(*, 239.3.3.3), 00:20:56/stopped, RP 2.2.2.2, flags: SJC
  Incoming interface: GigabitEthernet0/1, RPF nbr 192.168.23.2
  Outgoing interface list:
    GigabitEthernet0/2, Forward/Sparse, 00:11:42/00:02:05

(192.168.1.1, 239.3.3.3), 00:05:46/00:02:46, flags: JT
  Incoming interface: GigabitEthernet0/1, RPF nbr 192.168.23.2
  Outgoing interface list:
    GigabitEthernet0/2, Forward/Sparse, 00:05:46/00:02:05
</code></pre>

Above we see that R3 has joined the SPT for (192.168.1.1,239.3.3.3) and that it is forwarding traffic on its GigabitEthernet0/2 interface.

The SPT requires more memory than the RPT, after all we just added another entry in our multicast routing table. On the other hand, there is less delay since the SPT is the shortest path to the source. When we use the RPT, traffic has to go through the RP first.

Since R3 is sitting behind the RP, we could disable SPT switchover. Cisco IOS router will switch to the SPT whenever the traffic exceeds 0 kbps…in other words: immediately. You can change this value or you can configure the router always to use the RPT:

<pre><code><strong>R3(config)#ip pim spt-threshold infinity
</strong></code></pre>

It will now use the RPT instead:

<pre><code><strong>R3#show ip mroute 239.3.3.3
</strong>
(*, 239.3.3.3), 00:00:12/00:02:47, RP 2.2.2.2, flags: SC
  Incoming interface: GigabitEthernet0/1, RPF nbr 192.168.23.2
  Outgoing interface list:
    GigabitEthernet0/2, Forward/Sparse, 00:00:12/00:02:47
</code></pre>

Seeing the difference between the SPT and RPT might be easier when we check R4. It’s shortest path is directly to R1, not through the RP. Let’s take a look. We’ll configure H4 to join the 239.4.4.4 group address:

<pre><code><strong>H4(config)#interface GigabitEthernet0/1
</strong><strong>H4(config-if)#ip igmp join-group 239.4.4.4
</strong></code></pre>

And we will send some traffic to this group address:

<pre><code><strong>S1#ping 239.4.4.4 repeat 1000
</strong>Type escape sequence to abort.
Sending 1000, 100-byte ICMP Echos to 239.4.4.4, timeout is 2 seconds:

Reply to request 1 from 192.168.4.4, 23 ms
</code></pre>

Now take a look at its multicast routing table:

<pre><code><strong>R4#show ip mroute 239.4.4.4
</strong>
(*, 239.4.4.4), 00:01:06/stopped, RP 2.2.2.2, flags: SJC
  Incoming interface: GigabitEthernet0/2, RPF nbr 192.168.24.2
  Outgoing interface list:
    GigabitEthernet0/3, Forward/Sparse, 00:01:06/00:02:40

(192.168.1.1, 239.4.4.4), 00:00:38/00:02:21, flags: JT
  Incoming interface: GigabitEthernet0/1, RPF nbr 192.168.14.1
  Outgoing interface list:
    GigabitEthernet0/3, Forward/Sparse, 00:00:38/00:02:40
</code></pre>

Above you can see that R4 has joined the RPT for 239.4.4.4 but it has joined the SPT for traffic that is sourced from 192.168.1.1 and destined to 239.4.4.4. Here’s what the RP looks like:

<pre><code><strong>R2#show ip mroute 239.4.4.4
</strong>
(*, 239.4.4.4), 00:02:05/00:03:22, RP 2.2.2.2, flags: S
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/2, Forward/Sparse, 00:02:05/00:03:22

(192.168.1.1, 239.4.4.4), 00:01:38/00:01:43, flags: PT
  Incoming interface: GigabitEthernet0/1, RPF nbr 192.168.12.1
  Outgoing interface list: Null
</code></pre>

Notice that R2 has pruned traffic from 192.168.1.1 to 239.4.4.4. This traffic is no longer required since R4 has switched to SPT and is no longer using the RP.

When R4 made the switch from RPT to SPT, it has sent a PIM prune message to the RP. I haven’t showed you this packet before so take a look below:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/02/multicast-pim-sparse-spt.png" alt=""><figcaption></figcaption></figure>

Above you can see the PIM prune packet that R4 has sent towards the RP. It tells R2 to stop sending traffic from 192.168.1.1 to 239.4.4.4. Here’s the capture if you want to see it for yourself:

[Multicast PIM Sparse Prune Packet](https://www.cloudshark.org/captures/8804ca52f5a7)

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
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
 ip igmp join-group 239.3.3.3
 duplex auto
 speed auto
 media-type rj45
!
ip default-gateway 192.168.3.254
!
end
```
{% endtab %}

{% tab title="H4" %}
```
hostname H4
!
no ip routing
!
interface GigabitEthernet0/1
 ip address 192.168.4.4 255.255.255.0
 no ip route-cache
 ip igmp join-group 239.4.4.4
 duplex auto
 speed auto
 media-type rj45
!
ip default-gateway 192.168.4.254
!
end
```
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
ip multicast-routing 
!
interface GigabitEthernet0/1
 ip address 192.168.12.1 255.255.255.0
 ip pim sparse-mode
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/2
 ip address 192.168.14.1 255.255.255.0
 ip pim sparse-mode
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/3
 ip address 192.168.1.254 255.255.255.0
 ip pim sparse-mode
 duplex auto
 speed auto
 media-type rj45
!
router ospf 1
 network 192.168.1.0 0.0.0.255 area 0
 network 192.168.12.0 0.0.0.255 area 0
 network 192.168.14.0 0.0.0.255 area 0
!
ip pim rp-address 2.2.2.2
no ip pim autorp
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
ip multicast-routing 
!
interface Loopback0
 ip address 2.2.2.2 255.255.255.255
!
interface GigabitEthernet0/1
 ip address 192.168.12.2 255.255.255.0
 ip pim sparse-mode
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/2
 ip address 192.168.24.2 255.255.255.0
 ip pim sparse-mode
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/3
 ip address 192.168.23.2 255.255.255.0
 ip pim sparse-mode
 duplex auto
 speed auto
 media-type rj45
!
router ospf 1
 network 2.2.2.2 0.0.0.0 area 0
 network 192.168.12.0 0.0.0.255 area 0
 network 192.168.23.0 0.0.0.255 area 0
 network 192.168.24.0 0.0.0.255 area 0
!
ip pim rp-address 2.2.2.2
no ip pim autorp
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
 ip address 192.168.23.3 255.255.255.0
 ip pim sparse-mode
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/2
 ip address 192.168.3.254 255.255.255.0
 ip pim sparse-mode
 duplex auto
 speed auto
 media-type rj45
!
router ospf 1
 network 192.168.3.0 0.0.0.255 area 0
 network 192.168.23.0 0.0.0.255 area 0
!
ip pim rp-address 2.2.2.2
no ip pim autorp
ip pim spt-threshold infinity
!
end
```
{% endtab %}

{% tab title="R4" %}
```
hostname R4
!
ip multicast-routing 
!
interface GigabitEthernet0/1
 ip address 192.168.14.4 255.255.255.0
 ip pim sparse-mode
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/2
 ip address 192.168.24.4 255.255.255.0
 ip pim sparse-mode
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/3
 ip address 192.168.4.254 255.255.255.0
 ip pim sparse-mode
 duplex auto
 speed auto
 media-type rj45
!
router ospf 1
 network 192.168.4.0 0.0.0.255 area 0
 network 192.168.14.0 0.0.0.255 area 0
 network 192.168.24.0 0.0.0.255 area 0
!
ip pim rp-address 2.2.2.2
no ip pim autorp
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

## Conclusion

That’s the end of our PIM sparse mode story. What have we learned?

* PIM sparse mode uses a RP (Rendezvous Point) as a “central point” for our multicast traffic.
* Routers will use PIM register packets to register sources with the RP. The first multicast packet is encapsulated and forwarded to the RP.
* When the RP is not interested in traffic from a certain group then it will send a PIM register stop packet.
* The router that sent the PIM register will start a suppression timer (60 seconds) and will send a PIM register null packet a few seconds before the suppression timer expires.
* Routers with receivers will join the RPT (Root Path Tree) for each group that they want to receive.
* Once routers with receivers get a multicast packet from the RP, they will switch from the RPT to the SPT when traffic exceeds 0 kbps (in other words: immediately).

I hope you enjoyed this lesson! If you have any questions, feel free to leave a comment.
