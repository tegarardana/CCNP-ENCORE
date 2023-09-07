# Multicast PIM Designated Router (DR)

When we configure PIM on our routers, we will establish PIM neighbor adjacencies, and the PIM hello messages are also used to elect a designated router for each multi-access network. The DR is the router that will **forward the PIM join** message from the receiver to the RP (rendezvous point).

Since the DR is used to forward PIM join messages to the RP, it doesn’t do much good for multicast dense mode where we don’t have an RP. The only exception is when you use IGMPv1…in that case, the PIM DR will work as the IGMP query router because IGMPv1 doesn’t have a query router election.

{% hint style="danger" %}
Don’t mix up the PIM DR with the PIM forwarder! To decide which router will forward multicast traffic, we have the [PIM Assert mechanism](https://networklessons.com/cisco/ccnp-encor-350-401/multicast-pim-assert-explained).
{% endhint %}

Let’s take a look at the following topology to see  how the DR works:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/03/multicast-pim-dr.png" alt=""><figcaption></figcaption></figure>

Above, we see a small network with 4 routers. R1 is our RP, and R4 is the receiver. As you can see, R2, R3, and R4 are connected to the same multi-access network (switch). When R4 sends a PIM join message, both R2 and R3 would receive it and forward it to R1. This would mean that we have 2 multicast streams which results in duplicate packets and wasted bandwidth.

To avoid this problem, we will elect the DR. R2 or R3 will become the designated router, and only one of them will forward the PIM join message to our RP.

Let’s configure this small network and take a close look at how the DR works:

<pre><code><strong>R1(config)#ip multicast-routing
</strong><strong>R1(config)#ip pim rp-address 1.1.1.1
</strong>
<strong>R1(config)#interface loopback 0
</strong><strong>R1(config-if)#ip pim sparse-mode
</strong>
<strong>R1(config)#interface fastEthernet 0/0
</strong><strong>R1(config-if)#ip pim sparse-mode
</strong>
<strong>R1(config)#interface fastEthernet 0/1
</strong><strong>R1(config-if)#ip pim sparse-mode
</strong></code></pre>

We will use the loopback interface on R1 to advertise as the RP.

<pre><code><strong>R2(config)#ip multicast-routing
</strong><strong>R2(config)#ip pim rp-address 1.1.1.1 
</strong>
<strong>R2(config)#interface fastEthernet 0/0
</strong><strong>R2(config-if)#ip pim sparse-mode 
</strong>
<strong>R2(config)#interface fastEthernet 0/1
</strong><strong>R2(config-if)#ip pim sparse-mode
</strong></code></pre>

<pre><code><strong>R3(config)#ip multicast-routing 
</strong><strong>R3(config)#ip pim rp-address 1.1.1.1
</strong>
<strong>R3(config)#interface fastEthernet 0/0
</strong><strong>R3(config-if)#ip pim sparse-mode 
</strong>
<strong>R3(config)#interface fastEthernet 0/1
</strong><strong>R3(config-if)#ip pim sparse-mode
</strong></code></pre>

<pre><code><strong>R4(config)#ip multicast-routing 
</strong></code></pre>

R2 and R3 are configured for sparse mode and a static RP, R4 is only a receiver, so we don’t need PIM.

<pre><code><strong>R2#show ip pim neighbor 
</strong>PIM Neighbor Table
Mode: B - Bidir Capable, DR - Designated Router, N - Default DR Priority,
      S - State Refresh Capable
Neighbor          Interface                Uptime/Expires    Ver   DR
Address                                                            Prio/Mode
192.168.12.1      FastEthernet0/0          01:24:25/00:01:28 v2    1 / S
192.168.234.3     FastEthernet0/1          01:24:29/00:01:26 v2    1 / DR S
</code></pre>

R3 has been elected as the Designated router on this segment. Why? because by default, the **highest IP address** will determine who becomes the PIM DR. Now let’s enable a debug to see what the designated router really does for us:

<pre><code><strong>R2#debug ip pim
</strong>PIM debugging is on
</code></pre>

<pre><code><strong>R3#debug ip pim
</strong>PIM debugging is on
</code></pre>

We will use debug ip pim on R2 and R3. Now we will join a multicast group on R4:

<pre><code><strong>R4(config)#interface fastEthernet 0/0
</strong><strong>R4(config-if)#ip igmp join-group 239.1.1.1
</strong></code></pre>

R4 will join multicast group 239.1.1.1. Now let’s see what R2 and R3 think of this:

```
R2#
Check RP 1.1.1.1 into the (*, 239.1.1.1) entry
```

Above, you see that R2 doesn’t do much with it, it does add 1.1.1.1 as the RP for multicast group 239.1.1.1. You can see it here:

<pre><code><strong>R2#show ip mroute 239.1.1.1
</strong>IP Multicast Routing Table
Flags: D - Dense, S - Sparse, B - Bidir Group, s - SSM Group, C - Connected,
       L - Local, P - Pruned, R - RP-bit set, F - Register flag,
       T - SPT-bit set, J - Join SPT, M - MSDP created entry,
       X - Proxy Join Timer Running, A - Candidate for MSDP Advertisement,
       U - URD, I - Received Source Specific Host Report,
       Z - Multicast Tunnel, z - MDT-data group sender,
       Y - Joined MDT-data group, y - Sending to MDT-data group
Outgoing interface flags: H - Hardware switched, A - Assert winner
 Timers: Uptime/Expires
 Interface state: Interface, Next-Hop or VCD, State/Mode

<strong>(*, 239.1.1.1), 00:11:38/00:02:19, RP 1.1.1.1, flags: SP
</strong>  Incoming interface: FastEthernet0/0, RPF nbr 192.168.12.1
  Outgoing interface list: Null
</code></pre>

Above, you see that it created an entry for the 239.1.1.1 group address with 1.1.1.1 as the RP. Now let’s take a look at R3:

```
R3#
Check RP 1.1.1.1 into the (*, 239.1.1.1) entry
Building Triggered (*,G) Join / (S,G,RP-bit) Prune message for 239.1.1.1
Insert (*,239.1.1.1) join in nbr 192.168.13.1's queue
Building Join/Prune packet for nbr 192.168.13.1
Adding v2 (1.1.1.1/32, 239.1.1.1), WC-bit, RPT-bit, S-bit Join
Send v2 join/prune to 192.168.13.1 (FastEthernet0/0)
```

This is a more interesting output. First, it adds 1.1.1.1 as the RP for 239.1.1.1, but you can also see that it builds a PIM join message and forwards it to our RP. This is because R3 is our designated router.

By default, the **highest IP address** determines who will become the DR. This is because the **default priority is 1**. Let’s change the priority so that R2 becomes the DR:

<pre><code><strong>R2(config)#interface fa0/1
</strong><strong>R2(config-if)#ip pim dr-priority 100
</strong></code></pre>

The router with the highest priority becomes the DR. If you still have the debug enabled, this is what you see:

```
R2#
PIM(0): Changing DR for FastEthernet0/1, from 192.168.234.3 to 192.168.234.2 (this system)
%PIM-5-DRCHG: DR change from neighbor 192.168.234.3 to 192.168.234.2 on interface FastEthernet0/1
```

```
R3#
PIM(0): Changing DR for FastEthernet0/1, from 192.168.234.3 to 192.168.234.2  
%PIM-5-DRCHG: DR change from neighbor 192.168.234.3 to 192.168.234.2 on interface FastEthernet0/1
```

As you can see, it’s preemptive, it will take effect immediately. The designated router does something else besides forwarding the PIM join messages. The DR is also responsible for sending PIM register messages to the RP once a source starts sending packets to the multicast group address. Let’s send some packets to the multicast group address to see how it works:

<pre><code><strong>R4#ping 239.1.1.1 repeat 9999
</strong>
Type escape sequence to abort.
Sending 9999, 100-byte ICMP Echos to 239.1.1.1, timeout is 2 seconds:

Reply to request 0 from 192.168.234.4, 8 ms
</code></pre>

I’ll send some packets to the multicast group address on R4. This will trigger a PIM register message for the source to the RP. Take a look at R2:

```
R2#
Send v2 Register to 1.1.1.1 for 192.168.234.4, group 239.1.1.1
```

As the DR, it’s responsible for **registering the source at the rendezvous point**.

Last but not least, we have a failover mechanism. Unlike OSPF, there is no BDR (Backup Designated Router) in PIM. When the DR fails, other routers will see this because their PIM neighbor adjacency will go down. A new election will take place, and another router will become the DR.

This new DR already has the IGMP state for the required multicast groups because it also heard the IGMP membership reports from receivers on the segment. The only thing the new DR has to do is send a PIM join to the RP, and our traffic flow will continue.

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
interface GigabitEthernet0/2
 ip address 192.168.13.1 255.255.255.0
 ip pim sparse-mode
!
router ospf 1
 network 1.1.1.1 0.0.0.0 area 0
 network 192.168.12.0 0.0.0.255 area 0
 network 192.168.13.0 0.0.0.255 area 0
!
ip pim rp-address 1.1.1.1
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
 ip pim sparse-mode
!
interface GigabitEthernet0/2
 ip address 192.168.234.2 255.255.255.0
 ip pim dr-priority 100
 ip pim sparse-mode
!
router ospf 1
 network 192.168.12.0 0.0.0.255 area 0
 network 192.168.234.0 0.0.0.255 area 0
!
ip pim rp-address 1.1.1.1
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
 ip address 192.168.13.3 255.255.255.0
 ip pim sparse-mode
!
interface GigabitEthernet0/2
 ip address 192.168.234.3 255.255.255.0
 ip pim sparse-mode
!
router ospf 1
 network 192.168.13.0 0.0.0.255 area 0
 network 192.168.234.0 0.0.0.255 area 0
!
ip pim rp-address 1.1.1.1
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
 ip address 192.168.234.4 255.255.255.0
 ip igmp join-group 239.1.1.1
!
router ospf 1
 network 192.168.234.0 0.0.0.255 area 0
!
end
```
{% endtab %}
{% endtabs %}

I hope this has been helpful to you. If you have any more multicast questions, just leave a comment!
