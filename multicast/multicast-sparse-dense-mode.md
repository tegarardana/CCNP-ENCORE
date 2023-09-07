# Multicast Sparse-Dense Mode

Multicast PIM has three modes:

* [PIM sparse mode](https://networklessons.com/cisco/ccnp-encor-350-401/multicast-pim-sparse-mode): this is a “pull” model where we only forward multicast traffic when requested.
* [PIM dense mode](https://networklessons.com/cisco/ccnp-encor-350-401/multicast-pim-dense-mode): this is a “push” model where we flood multicast traffic everywhere and then prune it when it’s not needed.

The third one is **PIM sparse-dense mode** where we can use sparse _or_ dense mode for each multicast group. Why do you want to use this?\
\
When you use PIM sparse mode, our multicast routers need to know where the RP (Rendezvous Point) is in the network and which groups they serve. There are two methods:

* **Static**: configure the IP address of the RP on all multicast routers.
* **Dynamic**: use [Auto RP](https://networklessons.com/cisco/ccnp-encor-350-401/multicast-ip-pim-auto-rp) or [BSR](https://networklessons.com/cisco/ccnp-encor-350-401/multicast-pim-bootstrap-bsr).

When you use PIM sparse mode and Auto RP, the group-to-RP mapping is sent to the multicast 224.0.1.40 address. This is a classic chicken and egg problem…how do we receive traffic from 224.0.1.40 when we don’t know where the RP is? There are two methods to solve this:

* Use the **ip pim autorp listener** command.
* Use PIM sparse-dense mode.

The ip pim autorp listener command floods auto RP 224.0.1.39 and 224.0.1.40 multicast groups on sparse mode interfaces, allowing all routers to receive the group-to-RP mapping information.

PIM sparse-dense mode also allows us to flood the auto RP 224.0.1.39 and 224.0.1.40 multicast groups but in addition, it also **floods all multicast traffic that we don’t have an RP** for.

In this lesson, I’ll show you how PIM sparse-dense mode forwards some traffic with sparse mode and other traffic with dense mode.

### Configuration

Here is the topology we will use:

![Multicast Pim Sparse Dense Topology](https://networklessons.com/wp-content/uploads/2018/01/multicast-pim-sparse-dense-topology.png)

R1, R2, and R3 are multicast routers and we use Auto RP on R1 for both the mapping agent and RP role. S1 is used to source some multicast traffic, H1 is our receiver.

* Configurations
* R1
* R2
* R3
* S1
* H1

Want to take a look for yourself? Here you will find the startup configuration of each device.

First, we need to enable multicast routing globally on all routers:

<pre><code>R1, R2 &#x26; R3
<strong>(config)#ip multicast-routing
</strong></code></pre>

All interfaces that process multicast traffic need PIM sparse-dense mode enabled. Here’s R1:

<pre><code><strong>R1(config)#interface GigabitEthernet 0/1
</strong><strong>R1(config-if)#ip pim sparse-dense-mode 
</strong>
<strong>R1(config)#interface GigabitEthernet 0/2
</strong><strong>R1(config-if)#ip pim sparse-dense-mode     
</strong>
<strong>R1(config)#interface Loopback 0
</strong><strong>R1(config-if)#ip pim sparse-dense-mode
</strong></code></pre>

And R2:

<pre><code><strong>R2(config)#interface GigabitEthernet 0/1 
</strong><strong>R2(config-if)#ip pim sparse-dense-mode 
</strong>
<strong>R2(config)#interface GigabitEthernet 0/2
</strong><strong>R2(config-if)#ip pim sparse-dense-mode
</strong></code></pre>

Last but not least, R3:

<pre><code><strong>R3(config)#interface GigabitEthernet 0/1
</strong><strong>R3(config-if)#ip pim sparse-dense-mode
</strong>
<strong>R3(config)#interface GigabitEthernet 0/2
</strong><strong>R3(config-if)#ip pim sparse-dense-mode
</strong></code></pre>

We configure R1 so that it’s the Auto RP mapping agent and that it advertises itself as the RP:

<pre><code><strong>R1(config)#ip pim send-rp-announce loopback 0 scope 5
</strong><strong>R1(config)#ip pim send-rp-discovery loopback 0 scope 5
</strong></code></pre>

Everything is now in place. Let’s make sure we see PIM neighbors:

<pre><code><strong>R2#show ip pim neighbor
</strong>PIM Neighbor Table
Mode: B - Bidir Capable, DR - Designated Router, N - Default DR Priority,
      P - Proxy Capable, S - State Refresh Capable, G - GenID Capable,
      L - DR Load-balancing Capable
Neighbor          Interface                Uptime/Expires    Ver   DR
Address                                                            Prio/Mode
192.168.12.1      GigabitEthernet0/1       00:27:17/00:01:33 v2    1 / S P G
192.168.23.3      GigabitEthernet0/2       00:26:00/00:01:23 v2    1 / DR S P G
</code></pre>

#### Discovering the RP

Do all routers have group-to-RP mapping? Let’s find out:

<pre><code><strong>R2#show ip pim rp mapping 
</strong>PIM Group-to-RP Mappings

Group(s) 224.0.0.0/4
  RP 1.1.1.1 (?), v2v1
    Info source: 1.1.1.1 (?), elected via Auto-RP
         Uptime: 00:00:35, expires: 00:02:23
</code></pre>

<pre><code><strong>R3#show ip pim rp mapping 
</strong>PIM Group-to-RP Mappings

Group(s) 224.0.0.0/4
  RP 1.1.1.1 (?), v2v1
    Info source: 1.1.1.1 (?), elected via Auto-RP
         Uptime: 00:00:46, expires: 00:02:10
</code></pre>

Both R2 and R3 have learned the group-to-RP mapping. How did they learn this? Let’s find out:

<pre><code><strong>R1#show ip mroute 224.0.1.40
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

(*, 224.0.1.40), 00:07:15/stopped, RP 0.0.0.0, flags: DCL
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    Loopback0, Forward/Sparse-Dense, 00:02:31/stopped
    GigabitEthernet0/1, Forward/Sparse-Dense, 00:07:15/stopped

(1.1.1.1, 224.0.1.40), 00:01:36/00:02:22, flags: LT
  Incoming interface: Loopback0, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/1, Forward/Sparse-Dense, 00:01:36/stopped
</code></pre>

The multicast 224.0.1.40 address is used to share group-to-RP mapping information and as you can see above, this is flooded using dense mode.

Here’s the output on R2:

<pre><code><strong>R2#show ip mroute 224.0.1.40
</strong>
(*, 224.0.1.40), 00:05:46/stopped, RP 0.0.0.0, flags: DCL
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/2, Forward/Sparse-Dense, 00:04:28/stopped
    GigabitEthernet0/1, Forward/Sparse-Dense, 00:05:46/stopped

(1.1.1.1, 224.0.1.40), 00:02:03/00:02:56, flags: LT
  Incoming interface: GigabitEthernet0/1, RPF nbr 192.168.12.1
  Outgoing interface list:
    GigabitEthernet0/2, Forward/Sparse-Dense, 00:02:03/stopped
</code></pre>

Above we see that R2 has received traffic to 224.0.1.40 using dense mode. For R2, it doesn’t matter if we use sparse or dense mode since it is directly connected to R1. It would have received traffic to 224.1.0.40 anyway.

For R3, we do need dense mode since R2 has to flood the traffic it receives from R1 to R3. Here’s the output of R3:

<pre><code><strong>R3#show ip mroute 224.0.1.40
</strong>
(*, 224.0.1.40), 00:04:33/stopped, RP 0.0.0.0, flags: DCL
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/1, Forward/Sparse-Dense, 00:04:33/stopped

(1.1.1.1, 224.0.1.40), 00:02:08/00:02:51, flags: PLTX
  Incoming interface: GigabitEthernet0/1, RPF nbr 192.168.23.2
  Outgoing interface list: Null
</code></pre>

R3 also has received traffic to 224.0.1.40. This is looking good.

#### Traffic with sparse mode

Let’s try to send some multicast traffic. Remember our group-to-RP mapping?

<pre><code><strong>R3#show ip pim rp mapping | include Group
</strong>PIM Group-to-RP Mappings
Group(s) 224.0.0.0/4
</code></pre>

R1 is willing to be the RP for the entire 224.0.0.0/4 multicast range. This means that all groups are forwarded with PIM sparse mode.To test if this is true, I will let H1 join a multicast group:

<pre><code><strong>H1(config)#interface GigabitEthernet 0/1
</strong><strong>H1(config-if)#ip igmp join-group 239.1.1.1
</strong></code></pre>

Let’s send a ping from our source:

<pre><code><strong>S1#ping 239.1.1.1 repeat 1
</strong>Type escape sequence to abort.
Sending 1, 100-byte ICMP Echos to 239.1.1.1, timeout is 2 seconds:

Reply to request 0 from 192.168.3.101, 23 ms
</code></pre>

Let’s check the multicast routing table of R3:

<pre><code><strong>R3#show ip mroute 239.1.1.1
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

(*, 239.1.1.1), 00:08:10/stopped, RP 1.1.1.1, flags: SJC
  Incoming interface: GigabitEthernet0/1, RPF nbr 192.168.23.2
  Outgoing interface list:
    GigabitEthernet0/2, Forward/Sparse-Dense, 00:08:10/00:02:56

(192.168.1.101, 239.1.1.1), 00:00:46/00:02:13, flags: JT
  Incoming interface: GigabitEthernet0/1, RPF nbr 192.168.23.2
  Outgoing interface list:
    GigabitEthernet0/2, Forward/Sparse-Dense, 00:00:46/00:02:56
</code></pre>

Above, we see that we use sparse mode to forward traffic to the 239.1.1.1 address.

#### Traffic with dense mode

Besides auto RP traffic, what other traffic is forwarded with dense mode? To test this, we need to reconfigure R1 so that it’s not the RP for all multicast groups. Let’s change it so that it only wants to be the RP for the 239.1.1.1 address:

<pre><code><strong>R1(config)#ip access-list standard MULTICAST_GROUPS
</strong><strong>R1(config-std-nacl)#permit host 239.1.1.1
</strong></code></pre>

<pre><code><strong>R1(config)#ip pim send-rp-announce loopback 0 scope 5 group-list MULTICAST_GROUPS
</strong></code></pre>

Now you need to wait for the PIM group-to-rp mapping to expire or you can use clear ip pim rp-mapping to speed it up. Take a look at R3:

<pre><code><strong>R3#show ip pim rp mapping 
</strong>PIM Group-to-RP Mappings

Group(s) 239.1.1.1/32
  RP 1.1.1.1 (?), v2v1
    Info source: 1.1.1.1 (?), elected via Auto-RP
         Uptime: 00:00:09, expires: 00:02:47
</code></pre>

R1 is only willing to be the RP for 239.1.1.1 and we don’t know of any other RPs in our network. Let’s join another multicast group on H1, perhaps 239.3.3.3:

<pre><code><strong>H1(config)#interface GigabitEthernet 0/1
</strong><strong>H1(config-if)#ip igmp join-group 239.3.3.3
</strong></code></pre>

Let’s send some traffic:

<pre><code><strong>S1#ping 239.3.3.3 repeat 1
</strong>Type escape sequence to abort.
Sending 1, 100-byte ICMP Echos to 239.3.3.3, timeout is 2 seconds:

Reply to request 0 from 192.168.3.101, 84 ms
</code></pre>

Our ping is working. Why? Let’s find out:

<pre><code><strong>R3#show ip mroute 239.3.3.3
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

(*, 239.3.3.3), 00:01:04/stopped, RP 0.0.0.0, flags: DC
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet0/2, Forward/Sparse-Dense, 00:01:04/stopped
    GigabitEthernet0/1, Forward/Sparse-Dense, 00:01:04/stopped

(192.168.1.101, 239.3.3.3), 00:00:38/00:02:21, flags: T
  Incoming interface: GigabitEthernet0/1, RPF nbr 192.168.23.2
  Outgoing interface list:
    GigabitEthernet0/2, Forward/Sparse-Dense, 00:00:38/stopped
</code></pre>

Because there is no RP for 239.3.3.3, this traffic got forwarded with dense mode.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
no ip domain lookup
ip multicast-routing 
ip cef
!
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
 ip pim sparse-dense-mode
!
interface GigabitEthernet0/1
 ip address 192.168.12.1 255.255.255.0
 ip pim sparse-dense-mode
!
interface GigabitEthernet0/2
 ip address 192.168.1.254 255.255.255.0
 ip pim sparse-dense-mode
!
router ospf 1
 router-id 1.1.1.1
 network 1.1.1.1 0.0.0.0 area 0
 network 192.168.1.0 0.0.0.255 area 0
 network 192.168.12.0 0.0.0.255 area 0
!
ip pim send-rp-announce Loopback0 scope 5 group-list MULTICAST_GROUPS
ip pim send-rp-discovery Loopback0 scope 5
!
ip access-list standard MULTICAST_GROUPS
 permit 239.1.1.1
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
no ip domain lookup
ip multicast-routing 
ip cef
!
interface GigabitEthernet0/1
 ip address 192.168.12.2 255.255.255.0
 ip pim sparse-dense-mode
!
interface GigabitEthernet0/2
 ip address 192.168.23.2 255.255.255.0
 ip pim sparse-dense-mode
!
router ospf 1
 router-id 2.2.2.2
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
no ip domain lookup
ip multicast-routing 
ip cef
!
interface GigabitEthernet0/1
 ip address 192.168.23.3 255.255.255.0
 ip pim sparse-dense-mode
!
interface GigabitEthernet0/2
 ip address 192.168.3.254 255.255.255.0
 ip pim sparse-dense-mode
!
router ospf 1
 router-id 3.3.3.3
 network 192.168.3.0 0.0.0.255 area 0
 network 192.168.23.0 0.0.0.255 area 0
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
no ip cef
!
interface GigabitEthernet0/1
 ip address 192.168.3.101 255.255.255.0
 ip igmp join-group 239.3.3.3
 ip igmp join-group 239.1.1.1
!
ip default-gateway 192.168.3.254
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
no ip cef
!
interface GigabitEthernet0/1
 ip address 192.168.1.101 255.255.255.0
!
ip default-gateway 192.168.1.254
!
end
```
{% endtab %}
{% endtabs %}

### Conclusion

You have now learned how PIM sparse-dense mode works:

* When there is an RP, multicast traffic is forwarded using sparse mode.
* When there is no RP, multicast traffic is forwarded using dense mode.
* PIM sparse-dense mode can be used when you use auto RP so that traffic to 224.0.1.39 and 224.0.1.40 is flooded using dense mode. This allows all multicast routers to receive group-to-RP information.

I hope you enjoyed this lesson. If you have any questions feel free to leave a comment!

\
